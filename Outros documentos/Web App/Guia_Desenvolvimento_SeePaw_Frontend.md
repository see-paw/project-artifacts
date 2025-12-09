# Guia de Desenvolvimento - SeePaw Frontend

## Estado Atual do Projeto

### Implementado

#### 1. Navbar
- Componente de navegação principal
- Exportado em `src/components/index.ts`
- Integrado no `MainLayout`

#### 2. Página de Animais (`/animals`)
- Cards de animais com paginação
- TanStack Query integrado com React Router loaders
- Função de API separada na pasta `api/`

#### 3. Estrutura de Routing
```tsx
/ (root)
  ├── Home (index)
  ├── animals (com loader)
  └── animals/:animalId
```

### Importante: Padrões Estabelecidos

**Ao desenvolver novas funcionalidades:**
1. Consultar o código existente como referência
2. Seguir a mesma estrutura de pastas e nomenclatura
3. Reutilizar os estilos de `tokens.css` e `utilities.css`
4. Usar CSS Modules para estilos de componentes
5. Seguir o padrão de TanStack Query + React Router Loaders
6. Criar tipos TypeScript para todas as entidades

## Padrão: TanStack Query + React Router Loaders

### Como Funciona

Este padrão combina o melhor dos dois mundos:
- **React Router Loader**: carrega dados críticos antes do render
- **TanStack Query**: gestão de cache, refetch, invalidação

### Estrutura

```
src/
├── api/
│   └── animals.ts          # Funções de comunicação com backend
├── routes/
│   ├── loaders/
│   │   └── animal.ts       # Loader functions
│   └── routes.tsx          # Configuração de rotas
└── pages/
    └── Animals/
        └── Animals.tsx     # Componente da página
```

### Implementação Passo a Passo

#### 1. Criar Função de API (`src/api/animals.ts`)

```typescript
import apiClient from './client';

export interface Animal {
  id: string;
  name: string;
  species: string;
  age: number;
  imageUrl: string;
  description: string;
}

export interface PaginatedAnimals {
  data: Animal[];
  totalCount: number;
  pageNumber: number;
  pageSize: number;
  totalPages: number;
}

export const getAnimals = async (
  pageNumber: number = 1,
  pageSize: number = 12
): Promise<PaginatedAnimals> => {
  const response = await apiClient.get('/animals', {
    params: { pageNumber, pageSize }
  });
  return response.data;
};

export const getAnimalById = async (id: string): Promise<Animal> => {
  const response = await apiClient.get(`/animals/${id}`);
  return response.data;
};
```

#### 2. Criar Loader (`src/routes/loaders/animal.ts`)

```typescript
import { QueryClient } from '@tanstack/react-query';
import { LoaderFunctionArgs } from 'react-router-dom';
import { getAnimals } from '@/api/animals';

export const animalsQueryKey = (pageNumber: number, pageSize: number) => 
  ['animals', pageNumber, pageSize];

export const animalsLoader = (queryClient: QueryClient) => 
  async ({ request }: LoaderFunctionArgs) => {
    const url = new URL(request.url);
    const pageNumber = Number(url.searchParams.get('page') || '1');
    const pageSize = Number(url.searchParams.get('pageSize') || '12');

    const query = {
      queryKey: animalsQueryKey(pageNumber, pageSize),
      queryFn: () => getAnimals(pageNumber, pageSize),
    };

    return (
      queryClient.getQueryData(query.queryKey) ??
      await queryClient.fetchQuery(query)
    );
  };
```

#### 3. Configurar Router (`src/routes/routes.tsx`)

```typescript
import { createBrowserRouter } from 'react-router-dom';
import { QueryClient } from '@tanstack/react-query';
import { animalsLoader } from './loaders/animal';

const queryClient = new QueryClient();

const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        path: 'animals',
        element: <Animals />,
        loader: animalsLoader(queryClient)
      }
    ]
  }
]);
```

#### 4. Usar no Componente (`src/pages/Animals/Animals.tsx`)

```typescript
import { useQuery } from '@tanstack/react-query';
import { useSearchParams } from 'react-router-dom';
import { getAnimals } from '@/api/animals';
import { animalsQueryKey } from '@/routes/loaders/animal';

export default function Animals() {
  const [searchParams, setSearchParams] = useSearchParams();
  const pageNumber = Number(searchParams.get('page') || '1');
  const pageSize = Number(searchParams.get('pageSize') || '12');

  const { data, isLoading, error } = useQuery({
    queryKey: animalsQueryKey(pageNumber, pageSize),
    queryFn: () => getAnimals(pageNumber, pageSize),
  });

  const handlePageChange = (newPage: number) => {
    setSearchParams({ page: newPage.toString(), pageSize: pageSize.toString() });
  };

  if (isLoading) return <div>A carregar...</div>;
  if (error) return <div>Erro ao carregar animais</div>;

  return (
    <div>
      <div className="grid grid-cols-3 gap-4">
        {data.data.map(animal => (
          <AnimalCard key={animal.id} animal={animal} />
        ))}
      </div>
      <Pagination 
        currentPage={data.pageNumber}
        totalPages={data.totalPages}
        onPageChange={handlePageChange}
      />
    </div>
  );
}
```

## Padrões Estabelecidos

### 1. Estrutura de Ficheiros API

Cada entidade tem o seu ficheiro em `src/api/`:
```
api/
├── client.ts           # Configuração Axios
├── animals.ts          # Endpoints de animais
├── fosterings.ts       # Endpoints de fosterings
├── shelters.ts         # Endpoints de abrigos
└── auth.ts             # Autenticação
```

### 2. Nomenclatura de Query Keys

```typescript
// Padrão: [entidade, ...params]
const animalsQueryKey = (page: number, size: number) => ['animals', page, size];
const animalDetailQueryKey = (id: string) => ['animals', id];
const fosteringsQueryKey = (userId: string) => ['fosterings', userId];
```

### 3. Tipos TypeScript

Criar interfaces para todas as entidades e respostas da API:

```typescript
// src/types/animal.ts
export interface Animal {
  id: string;
  name: string;
  species: 'Dog' | 'Cat' | 'Other';
  breed?: string;
  age: number;
  gender: 'Male' | 'Female';
  size: 'Small' | 'Medium' | 'Large';
  status: 'Available' | 'Fostering' | 'Ownership' | 'Adopted';
  imageUrls: string[];
  description: string;
  medicalInfo?: string;
  shelterId: string;
  createdAt: string;
}

export interface CreateAnimalDto {
  name: string;
  species: 'Dog' | 'Cat' | 'Other';
  breed?: string;
  age: number;
  gender: 'Male' | 'Female';
  size: 'Small' | 'Medium' | 'Large';
  description: string;
  medicalInfo?: string;
}
```

### 4. Mutations com TanStack Query

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { createAnimal, CreateAnimalDto } from '@/api/animals';

export function useCreateAnimal() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: CreateAnimalDto) => createAnimal(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['animals'] });
    },
  });
}

// Uso no componente
function CreateAnimalForm() {
  const createMutation = useCreateAnimal();

  const handleSubmit = (data: CreateAnimalDto) => {
    createMutation.mutate(data, {
      onSuccess: () => {
        toast.success('Animal criado com sucesso!');
        navigate('/animals');
      },
      onError: (error) => {
        toast.error('Erro ao criar animal');
      }
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
    </form>
  );
}
```

## Estilização com CSS Modules

### Sistema de Design Tokens

O projeto usa um sistema de design tokens centralizado em `src/styles/`:

- **`tokens.css`**: Variáveis CSS (cores, espaçamentos, tipografia, shadows, etc.)
- **`utilities.css`**: Classes utilitárias (flex, grid, spacing, text)
- **`global.css`**: Reset e estilos globais

### Como Usar CSS Modules

Cada componente deve ter o seu CSS Module próprio:

```
components/
└── features/
    └── AnimalCard/
        ├── AnimalCard.tsx
        └── AnimalCard.module.css
```

**No componente TypeScript:**
```typescript
import styles from './AnimalCard.module.css';

export function AnimalCard({ animal }: AnimalCardProps) {
  return (
    <article className={styles.card}>
      <img src={animal.imageUrls[0]} alt={animal.name} />
      <div className={styles.content}>
        <h3>{animal.name}</h3>
      </div>
    </article>
  );
}
```

**No CSS Module, usar os tokens:**
```css
.card {
  background-color: var(--color-background-primary);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
  padding: var(--spacing-4);
  transition: transform var(--transition-fast);
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-md);
}

.content {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-2);
}
```

### Classes Utilitárias

Para layouts comuns, usar as classes de `utilities.css`:

```tsx
<div className="flex items-center gap-4">
  <div className="grid grid-cols-3 gap-6">
    {/* conteúdo */}
  </div>
</div>
```

### Combinação de Classes

```tsx
import styles from './Component.module.css';

// Combinar CSS Module com utilities
<div className={`${styles.customStyle} flex items-center`}>

// Combinar múltiplas classes do module
<div className={`${styles.card} ${isActive ? styles.active : ''}`}>
```

## Tratamento de Erros

### Nos Loaders

Os loaders devem tratar erros adequadamente:

```typescript
export const animalsLoader = (queryClient: QueryClient) => 
  async ({ request }: LoaderFunctionArgs) => {
    try {
      const url = new URL(request.url);
      const pageNumber = Number(url.searchParams.get('page') || '1');
      const pageSize = Number(url.searchParams.get('pageSize') || '12');

      const query = {
        queryKey: animalsQueryKey(pageNumber, pageSize),
        queryFn: () => getAnimals(pageNumber, pageSize),
      };

      return (
        queryClient.getQueryData(query.queryKey) ??
        await queryClient.fetchQuery(query)
      );
    } catch (error) {
      console.error('Error loading animals:', error);
      throw new Response('Erro ao carregar animais', { status: 500 });
    }
  };
```

### Página de Erro

Configurada no router como `errorElement`:

```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout/>,
    errorElement: <Error/>,
    children: [...]
  }
]);
```

### Nos Componentes

Usar os estados do TanStack Query:

```typescript
const { data, isLoading, error, isError } = useQuery({
  queryKey: animalsQueryKey(pageNumber, pageSize),
  queryFn: () => getAnimals(pageNumber, pageSize),
});

if (isLoading) return <div>A carregar...</div>;
if (isError) return <div>Erro: {error.message}</div>;
```

## Reutilização de Componentes

### Componente Pagination

Criar um componente de paginação reutilizável:

```typescript
// src/components/common/Pagination/Pagination.tsx
interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
}

export function Pagination({ currentPage, totalPages, onPageChange }: PaginationProps) {
  return (
    <nav className={styles.pagination}>
      <button 
        disabled={currentPage === 1}
        onClick={() => onPageChange(currentPage - 1)}
      >
        Anterior
      </button>
      
      <span>{currentPage} / {totalPages}</span>
      
      <button 
        disabled={currentPage === totalPages}
        onClick={() => onPageChange(currentPage + 1)}
      >
        Próxima
      </button>
    </nav>
  );
}
```

### Custom Hook: usePagination

Extrair lógica de paginação para um hook reutilizável:

```typescript
// src/hooks/usePagination.ts
import { useSearchParams } from 'react-router-dom';

export function usePagination(defaultPageSize: number = 12) {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const pageNumber = Number(searchParams.get('page') || '1');
  const pageSize = Number(searchParams.get('pageSize') || defaultPageSize);

  const setPage = (newPage: number) => {
    setSearchParams({ 
      page: newPage.toString(), 
      pageSize: pageSize.toString() 
    });
  };

  const setPageSize = (newPageSize: number) => {
    setSearchParams({ 
      page: '1', 
      pageSize: newPageSize.toString() 
    });
  };

  return {
    pageNumber,
    pageSize,
    setPage,
    setPageSize,
  };
}
```

**Uso no componente:**
```typescript
function Animals() {
  const { pageNumber, pageSize, setPage } = usePagination();

  const { data, isLoading, error } = useQuery({
    queryKey: animalsQueryKey(pageNumber, pageSize),
    queryFn: () => getAnimals(pageNumber, pageSize),
  });

  return (
    <div>
      {/* renderizar animais */}
      <Pagination 
        currentPage={data.pageNumber}
        totalPages={data.totalPages}
        onPageChange={setPage}
      />
    </div>
  );
}
```

## Configuração do Axios Client

O ficheiro `src/api/client.ts` configura o Axios com:
- Base URL da API (via variável de ambiente)
- Timeout configurado
- Interceptor para adicionar JWT token automaticamente

```typescript
import axios from 'axios';
import { useAuthStore } from '@/stores/authStore';

const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  timeout: 10000
});

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default apiClient;
```

**Variáveis de ambiente** (`.env.local`):
```
VITE_API_URL=http://localhost:5000/api
```

## Configuração do TanStack Query

No `src/main.tsx`, o QueryClient está configurado com:

```typescript
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,        // 5 minutos
      refetchOnWindowFocus: false,      // Não refetch ao focar janela
      retry: 1                          // Retry uma vez em caso de erro
    }
  }
});

// Envolver App com o Provider
<QueryClientProvider client={queryClient}>
  <App />
  <ReactQueryDevtools />  // DevTools apenas em desenvolvimento
</QueryClientProvider>
```

**Configurações importantes:**
- `staleTime`: Quanto tempo os dados são considerados "frescos"
- `refetchOnWindowFocus`: Refetch automático ao focar janela
- `retry`: Número de tentativas em caso de erro

## Estrutura de Pastas

Seguir rigorosamente a estrutura definida no documento Word:

```
src/
├── api/                # Configuração HTTP e endpoints
├── assets/             # Imagens, fontes, ícones
├── components/         # Componentes reutilizáveis
│   ├── common/         # Button, Input, Modal, Card
│   ├── layout/         # Navbar, Sidebar, Footer
│   └── features/       # Componentes específicos
├── hooks/              # Custom hooks
├── pages/              # Páginas da aplicação
├── routes/             # Configuração de rotas
│   └── loaders/        # Loader functions
├── stores/             # Zustand stores
├── styles/             # Estilos globais (tokens, utilities, global)
├── types/              # Tipos TypeScript
└── utils/              # Funções utilitárias
```

## Exportação de Componentes

Seguir o padrão estabelecido em `src/components/index.ts`:

```typescript
export { Navbar } from "./layout/Navbar/Navbar"
export type { NavbarProps } from "./layout/Navbar/Navbar"
```

À medida que novos componentes são criados, exportá-los aqui para facilitar imports:

```typescript
// Em vez de:
import { Navbar } from '@/components/layout/Navbar/Navbar';

// Usar:
import { Navbar } from '@/components';
```

## Paginação com URL State

```typescript
function Animals() {
  const [searchParams, setSearchParams] = useSearchParams();
  
  const pageNumber = Number(searchParams.get('page') || '1');
  const pageSize = Number(searchParams.get('pageSize') || '12');

  const handlePageChange = (newPage: number) => {
    setSearchParams({ 
      page: newPage.toString(), 
      pageSize: pageSize.toString() 
    });
  };
}
```

## Stack Tecnológica (Referência do Word)

Seguir rigorosamente a stack definida no documento técnico:

### Core
- **React 18+** com TypeScript
- **Vite** como build tool
- **React Router v6.4+** com Data API (loaders/actions)

### Estado
- **TanStack Query v5** para server state
- **Zustand** para client state (auth, UI)

### HTTP & API
- **Axios** com interceptors (JWT auth)

### Formulários
- **React Hook Form** para gestão de formulários
- **Zod** para validação de schemas

### Estilo
- **CSS Modules** (sem frameworks CSS)
- Design tokens em `tokens.css`
- Utilities em `utilities.css`

### Utilitários
- **date-fns** para manipulação de datas
- **react-hot-toast** para notificações

## Comandos Úteis

```bash
npm run dev              # Iniciar servidor desenvolvimento
npm run build            # Build produção
npm run preview          # Preview build
```

## Git Workflow

```bash
git checkout -b feature/nome-funcionalidade
git add .
git commit -m "feat: descrição da funcionalidade"
git push origin feature/nome-funcionalidade
```

## Recursos

- [React Router Docs](https://reactrouter.com/)
- [TanStack Query Docs](https://tanstack.com/query/latest)
- [React Hook Form](https://react-hook-form.com/)
- [Zod](https://zod.dev/)
- [Zustand](https://github.com/pmndrs/zustand)

---

**Nota**: Este guia complementa o documento Word principal (`SeePaw_Stack_Frontend_Completo.docx`). Consultar sempre os dois documentos em conjunto para ter a visão completa da stack e das decisões arquiteturais.

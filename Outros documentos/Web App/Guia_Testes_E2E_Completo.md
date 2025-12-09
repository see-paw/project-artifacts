# Guia Completo de Testes E2E - SeePaw

> **Arquitetura de Testes End-to-End com Playwright**  
> Estrutura modular, escalável e com suporte para API Mocking e Interception

---

## 📋 Índice

1. [Introdução e Filosofia](#1-introdução-e-filosofia)
2. [Arquitetura Geral](#2-arquitetura-geral)
3. [Estrutura de Diretórios](#3-estrutura-de-diretórios)
4. [Core Layer - Classes Base](#4-core-layer---classes-base)
5. [Page Object Model](#5-page-object-model)
6. [Component Objects](#6-component-objects)
7. [Page Manager](#7-page-manager)
8. [Fixtures Customizadas](#8-fixtures-customizadas)
9. [API Helpers](#9-api-helpers)
10. [Test Data](#10-test-data)
11. [Escrita de Testes](#11-escrita-de-testes)
12. [Configuração do Playwright](#12-configuração-do-playwright)
13. [Boas Práticas](#13-boas-práticas)
14. [Padrões de Testes](#14-padrões-de-testes)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. Introdução e Filosofia

### 1.1. Objetivos dos Testes E2E

Os testes End-to-End (E2E) do SeePaw têm como objetivo principal **validar fluxos completos da aplicação do ponto de vista do utilizador**, simulando interações reais com o browser e garantindo que:

- A UI renderiza corretamente
- As interações do utilizador funcionam como esperado
- Os dados são apresentados corretamente
- A navegação entre páginas funciona
- Estados de loading, erro e sucesso são geridos adequadamente
- A aplicação é responsiva e performante

### 1.2. Filosofia de Design

A estrutura de testes E2E do SeePaw foi construída seguindo princípios fundamentais:

#### Separação de Responsabilidades

Cada componente do sistema de testes tem uma responsabilidade única e bem definida:

- **Pages**: Encapsulam interações com páginas específicas
- **Components**: Encapsulam interações com componentes reutilizáveis
- **Core**: Fornece funcionalidades base partilhadas
- **Fixtures**: Gerenciam setup e teardown de recursos
- **Helpers**: Facilitam operações complexas (mocking, interception)
- **Test Data**: Centralizam dados de teste

#### Reutilização e DRY (Don't Repeat Yourself)

- Locators definidos uma única vez em Page/Component Objects
- Lógica de interação comum na classe `BasePage`
- Fixtures customizadas eliminam código duplicado nos testes
- Test data reutilizável entre múltiplos testes

#### Manutenibilidade

- Mudanças na UI requerem alterações apenas nos Page Objects
- Testes permanecem legíveis e focados no comportamento
- Estrutura modular facilita adição de novas páginas/componentes
- Nomenclatura consistente e auto-explicativa

#### Testabilidade

- Suporte para API mocking (testes rápidos e isolados)
- Suporte para API interception (testes realistas com modificações)
- Flexibilidade entre testes unitários de UI e testes de integração
- Facilita testes de edge cases e cenários de erro

### 1.3. Playwright vs Outras Ferramentas

**Por que Playwright?**

- **Multi-browser**: Chromium, Firefox, WebKit
- **Auto-waiting**: Espera automática por elementos estarem prontos
- **Network interception**: Suporte nativo para mocking e modificação de respostas
- **TypeScript first-class**: Tipagem completa
- **Parallelização**: Execução paralela de testes
- **Debugging**: Ferramentas excelentes (trace viewer, UI mode)
- **CI/CD ready**: Configuração simples para pipelines

---

## 2. Arquitetura Geral

### 2.1. Visão de Alto Nível

```
┌─────────────────────────────────────────────────────────────┐
│                         TESTES E2E                          │
│                      (animals.spec.ts)                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │      FIXTURES CUSTOMIZADAS         │
        │  - PageManager (pm)                │
        │  - ApiMockHelper (apiMock)         │
        │  - ApiInterceptHelper (apiIntercept)│
        └────────┬───────────────────────────┘
                 │
                 ▼
    ┌────────────────────────────────────────────┐
    │           PAGE MANAGER                     │
    │  - Centraliza Pages e Components           │
    │  - Gerencia navegação                      │
    └────┬───────────────────────┬───────────────┘
         │                       │
         ▼                       ▼
┌─────────────────┐    ┌──────────────────────┐
│   PAGE OBJECTS  │    │  COMPONENT OBJECTS   │
│  - AnimalsPage  │    │  - Pagination        │
│  - (futuras)    │    │  - Navbar            │
└────────┬────────┘    └──────────┬───────────┘
         │                        │
         └────────────┬───────────┘
                      ▼
              ┌──────────────┐
              │  BASE PAGE   │
              │  - Métodos   │
              │    comuns    │
              └──────────────┘
```

### 2.2. Camadas da Arquitetura

#### Camada de Testes (Tests Layer)

- **Localização**: `e2e/tests/`
- **Responsabilidade**: Definir cenários de teste e asserções
- **Características**: 
  - Usa fixtures customizadas
  - Foca no comportamento (what), não na implementação (how)
  - Legível e auto-documentada

#### Camada de Abstração (Abstraction Layer)

**Page Objects** (`e2e/pages/`):
- Representam páginas completas da aplicação
- Encapsulam locators e interações específicas da página
- Estendem `BasePage` para herdar funcionalidades comuns

**Component Objects** (`e2e/components/`):
- Representam componentes reutilizáveis da UI
- Podem ser usados em múltiplas páginas
- Também estendem `BasePage`

**Page Manager** (`e2e/page.manager.ts`):
- Centraliza acesso a todos os Page Objects e Components
- Gerencia navegação entre páginas
- Injetado automaticamente via fixture `pm`

#### Camada Core (Core Layer)

- **Localização**: `e2e/core/`
- **Responsabilidade**: Funcionalidades base partilhadas
- **Componentes**:
  - `BasePage`: Classe base com métodos de interação comuns
  - `ApiMockHelper`: Helper para mock completo de APIs
  - `ApiInterceptHelper`: Helper para interceptação e modificação de respostas

#### Camada de Fixtures (Fixtures Layer)

- **Localização**: `e2e/fixtures/`
- **Responsabilidade**: Setup e configuração de recursos de teste
- **Vantagem**: Injeção de dependências automática nos testes

#### Camada de Dados (Data Layer)

- **Localização**: `e2e/test-data/`
- **Responsabilidade**: Centralizar dados de teste mock
- **Organização**: Por feature/página

---

## 3. Estrutura de Diretórios

### 3.1. Árvore Completa

```
e2e/
├── components/                      # Component Objects
│   ├── navbar.component.ts          # Componente de navegação
│   └── PaginationComponent.ts       # Componente de paginação
│
├── core/                            # Funcionalidades base
│   ├── base.page.ts                 # Classe base para todos os Page/Component Objects
│   ├── ApiInterceptHelper.ts        # Helper para interceptação de API
│   └── ApiMockHelper.ts             # Helper para mock de API
│
├── fixtures/                        # Fixtures customizadas
│   └── base.fixture.ts              # Fixture base com pm, apiMock, apiIntercept
│
├── pages/                           # Page Objects
│   └── animals.page.ts              # Página de listagem de animais
│
├── test-data/                       # Dados de teste
│   └── AnimalsPage/
│       ├── animals-page1.ts         # Mock de animais para página 1
│       └── mockAnimalsPage1.ts      # Mock de página vazia
│
├── tests/                           # Testes E2E
│   └── animals.spec.ts              # Testes da página de animais
│
└── page.manager.ts                  # Gestor centralizado de Pages/Components
```

### 3.2. Convenções de Nomenclatura

| Tipo | Padrão                | Exemplo                   |
|------|-----------------------|---------------------------|
| Page Object | `[nome].page.ts`      | `animals.page.ts`         |
| Component Object | `[Nome].component.ts` | `pagination.component.ts` |
| Test File | `[nome].spec.ts`      | `animals.spec.ts`         |
| Test Data | `mock[Nome].ts`       | `mockAnimalsPage1.ts`     |
| Helper | `[Nome]Helper.ts`     | `ApiMockHelper.ts`        |
| Fixture | `[nome].fixture.ts`   | `base.fixture.ts`         |

---

## 4. Core Layer - Classes Base

### 4.1. BasePage

**Localização**: `e2e/core/base.page.ts`

A classe `BasePage` é a **fundação de toda a arquitetura de Page Objects**. Fornece métodos reutilizáveis para interações comuns com elementos da página.

#### Estrutura

```typescript
import {Locator, Page} from "@playwright/test";

export class BasePage {
    readonly page: Page;

    constructor(page: Page) {
        this.page = page
    }

    // Métodos de espera, interação, validação, etc.
}
```

#### Métodos Principais

##### Esperas (Waits)

```typescript
async waitForElementToBeVisible(locator: Locator, timeout: number = 5000) {
    await locator.waitFor({ state: 'visible', timeout });
}

async waitForElementToBeHidden(locator: Locator, timeout: number = 5000) {
    await locator.waitFor({ state: 'hidden', timeout });
}

async waitForNumberOfElements(locator: Locator, expectedCount: number, timeout: number = 5000) {
    await this.page.waitForFunction(
        ({ selector, count }) => {
            const elements = document.querySelectorAll(selector);
            return elements.length === count;
        },
        { selector: await locator.first().evaluate(el => {
                const testId = el.getAttribute('data-testid');
                return testId ? `[data-testid="${testId}"]` : el.className;
            }), count: expectedCount },
        { timeout }
    );
}
```

**Por que estas esperas são importantes?**

- `waitForElementToBeVisible`: Garante que o elemento está visível antes de interagir (evita `ElementNotVisibleError`)
- `waitForElementToBeHidden`: Útil para validar que spinners de loading desapareceram
- `waitForNumberOfElements`: Essencial para validar que uma lista foi completamente carregada

##### Validações

```typescript
async isElementVisible(locator: Locator): Promise<boolean> {
    try {
        await locator.waitFor({ state: 'visible', timeout: 1000 });
        return true;
    } catch {
        return false;
    }
}
```

**Uso**: Verificar se um elemento existe sem lançar exceção se não existir.

##### Interações com Formulários

```typescript
async fillFormField(locator: Locator, value: string) {
    await locator.clear();
    await locator.fill(value);
}

async selectDropdownOption(locator: Locator, value: string) {
    await locator.selectOption(value);
}
```

**Por que clear() antes de fill()?**

Para garantir que não há texto residual, especialmente importante em campos com validação que pode ter sido preenchida anteriormente.

##### Navegação

```typescript
async clickAndWaitForNavigation(locator: Locator, urlPattern: string) {
    await locator.click();
    await this.page.waitForURL(urlPattern);
}
```

**Uso**: Clicar num link e garantir que a navegação ocorreu antes de continuar.

##### Extração de Dados

```typescript
async getElementText(locator: Locator): Promise<string> {
    const text = await locator.textContent();
    return text ?? '';
}

async getElementAttribute(locator: Locator, attribute: string): Promise<string> {
    const attr = await locator.getAttribute(attribute);
    return attr ?? '';
}
```

**Por que retornar string vazia em vez de null?**

Evita `null` checks constantes nos testes, tornando o código mais limpo.

##### Gestão de Loading States

```typescript
async waitForLoadingToComplete(loadingSelector: string = '[data-testid="loading-spinner"]') {
    const loadingLocator = this.page.locator(loadingSelector);
    const isVisible = await this.isElementVisible(loadingLocator);

    if (isVisible) {
        await this.waitForElementToBeHidden(loadingLocator);
    }
}

async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle');
}
```

**Uso crítico**: Sempre esperar que o loading termine antes de fazer asserções sobre o conteúdo da página.

### 4.2. ApiMockHelper

**Localização**: `e2e/core/ApiMockHelper.ts`

O `ApiMockHelper` permite **substituir completamente chamadas à API** por dados mock, tornando os testes:

- **Mais rápidos**: Não há chamadas reais à API
- **Mais estáveis**: Não dependem da disponibilidade do backend
- **Mais previsíveis**: Dados controlados e consistentes
- **Isolados**: Testam apenas o frontend

#### Estrutura

```typescript
import { Page, Route } from '@playwright/test';

export class ApiMockHelper {
    constructor(private page: Page) {}

    // Métodos de mocking
}
```

#### Métodos

##### Mock Básico

```typescript
async mockApiCall<T>(urlPattern: string, mockData: T) {
    await this.page.route(urlPattern, async (route: Route) => {
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify(mockData)
        });
    });
}
```

**Como funciona?**

1. `page.route()` intercepta todas as requests que correspondem ao `urlPattern`
2. Quando uma request é interceptada, em vez de ir ao servidor, retorna `mockData`
3. O frontend recebe a resposta como se fosse real

**Exemplo de uso**:

```typescript
await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);
```

Qualquer chamada a `/api/animals` (com quaisquer query params) retornará `mockAnimalsPage1`.

##### Mock com Delay

```typescript
async mockWithDelay<T>(urlPattern: string, mockData: T, delayMs: number) {
    await this.page.route(urlPattern, async (route: Route) => {
        await new Promise(resolve => setTimeout(resolve, delayMs));
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify(mockData)
        });
    });
}
```

**Uso**: Simular loading states, testar skeleton screens, validar que spinners aparecem.

**Exemplo**:

```typescript
// Simula uma API lenta (1 segundo)
await apiMock.mockWithDelay('**/api/animals*', mockAnimalsPage1, 1000);

await pm.navigateToAnimals(1);

// Validar que loading spinner apareceu
const animalsPage = pm.getAnimalsPage();
const isLoadingVisible = await animalsPage.isLoadingSpinnerVisible();
expect(isLoadingVisible).toBe(true);

// Esperar carregar
await animalsPage.waitForAnimalsToLoad();
```

##### Mock de Erros

```typescript
async mockError(urlPattern: string, statusCode: number, errorMessage: string) {
    await this.page.route(urlPattern, async (route: Route) => {
        await route.fulfill({
            status: statusCode,
            contentType: 'application/json',
            body: JSON.stringify({ message: errorMessage })
        });
    });
}
```

**Uso**: Testar como a aplicação lida com erros (404, 500, etc.).

**Exemplo**:

```typescript
// Simular erro 500
await apiMock.mockError('**/api/animals*', 500, 'Internal Server Error');

await pm.navigateToAnimals(1);

// Validar que mensagem de erro aparece
const animalsPage = pm.getAnimalsPage();
const errorMessage = await animalsPage.getErrorMessage();
expect(errorMessage).toContain('Erro ao carregar');
```

##### Mock de Paginação

```typescript
async mockPaginatedAnimals<T>(pageNumber: number, mockData: T) {
    await this.page.route(`**/api/animals?pageNumber=${pageNumber}`, async (route: Route) => {
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify(mockData)
        });
    });
}
```

**Uso**: Mockar diferentes páginas de resultados.

**Exemplo**:

```typescript
// Mockar página 1
await apiMock.mockPaginatedAnimals(1, mockAnimalsPage1);

// Mockar página 2 com dados diferentes
await apiMock.mockPaginatedAnimals(2, mockAnimalsPage2);
```

### 4.3. ApiInterceptHelper

**Localização**: `e2e/core/ApiInterceptHelper.ts`

O `ApiInterceptHelper` permite **interceptar e modificar respostas reais da API**, mantendo a integração com o backend mas permitindo:

- Testar edge cases difíceis de reproduzir
- Modificar respostas para cenários específicos
- Validar tratamento de dados inesperados
- Capturar e validar respostas da API

#### Métodos

##### Interceptar e Modificar

```typescript
async interceptAndModify<T>(
    urlPattern: string,
    modifier: (response: T) => T
): Promise<void> {
    await this.page.route(urlPattern, async (route: Route) => {
        const response = await route.fetch();
        const responseBody = await response.json();
        const modifiedBody = modifier(responseBody);

        await route.fulfill({
            status: response.status(),
            headers: response.headers(),
            body: JSON.stringify(modifiedBody)
        });
    });
}
```

**Como funciona?**

1. Intercepta a request
2. Faz a chamada real ao servidor (`route.fetch()`)
3. Recebe a resposta real
4. Aplica a função `modifier` para alterar a resposta
5. Retorna a resposta modificada ao frontend

**Exemplo prático**:

```typescript
// Modificar o nome do primeiro animal na resposta real
await apiIntercept.interceptAndModify<PagedList<Animal>>(
    '**/api/animals',
    (response) => {
        if (response.items.length > 0) {
            response.items[0].name = 'Modified Animal Name';
        }
        return response;
    }
);

await pm.navigateToAnimals(1);

const animalsPage = pm.getAnimalsPage();
await animalsPage.waitForAnimalsToLoad();

const firstName = await animalsPage.getAnimalNameByIndex(0);
expect(firstName).toBe('Modified Animal Name');
```

**Casos de uso**:

- Testar como a UI lida com nomes muito longos
- Testar valores null/undefined em campos opcionais
- Simular dados inconsistentes
- Validar tratamento de tipos de dados inesperados

##### Capturar Resposta

```typescript
async captureResponse<T>(urlPattern: string): Promise<T> {
    return new Promise((resolve) => {
        this.page.on('response', async (response) => {
            if (response.url().includes(urlPattern)) {
                const data = await response.json();
                resolve(data);
            }
        });
    });
}
```

**Uso**: Capturar e validar que a API retornou os dados corretos.

##### Esperar e Capturar

```typescript
async waitAndCaptureResponse<T>(urlPattern: string): Promise<T> {
    const response = await this.page.waitForResponse(
        (resp) => resp.url().includes(urlPattern) && resp.status() === 200
    );
    return await response.json();
}
```

**Diferença de `captureResponse`**:

- `captureResponse`: Escuta passivamente, pode perder a resposta se já aconteceu
- `waitAndCaptureResponse`: Espera ativamente, garante que captura a resposta

##### Validar Status Code

```typescript
async waitForResponseWithStatus(urlPattern: string, expectedStatus: number): Promise<boolean> {
    try {
        await this.page.waitForResponse(
            (resp) => resp.url().includes(urlPattern) && resp.status() === expectedStatus,
            { timeout: 5000 }
        );
        return true;
    } catch {
        return false;
    }
}
```

**Uso**: Validar que uma chamada à API foi feita e teve o status esperado.

```typescript
const responseReceived = apiIntercept.waitForResponseWithStatus('**/api/animals', 200);

await pm.navigateToAnimals(1);

const wasReceived = await responseReceived;
expect(wasReceived).toBe(true);
```

---

## 5. Page Object Model

### 5.1. Conceito

O **Page Object Model** é um padrão de design onde **cada página da aplicação é representada por uma classe** que encapsula:

- Locators dos elementos da página
- Métodos para interagir com esses elementos
- Lógica de navegação específica da página

**Benefícios**:

- **Manutenibilidade**: Mudanças na UI só afetam o Page Object
- **Reutilização**: Métodos podem ser usados em múltiplos testes
- **Legibilidade**: Testes ficam mais descritivos e menos técnicos
- **Abstração**: Testes não precisam saber sobre locators ou DOM

### 5.2. AnimalsPage - Análise Completa

**Localização**: `e2e/pages/animals.page.ts`

#### Estrutura Básica

```typescript
import {Page} from "@playwright/test";
import {BasePage} from "../core/base.page";

export class AnimalsPage extends BasePage {

    constructor(page: Page) {
        super(page);
    }

    // Métodos de interação e validação
}
```

**Características**:

- Estende `BasePage` para herdar métodos comuns
- Recebe `Page` do Playwright no construtor
- Todos os métodos são `async` (interações com o browser são assíncronas)

#### Locators Inline vs Instance Variables

**Decisão de design**: Esta implementação usa **inline locators** em vez de propriedades de instância.

```typescript
// ❌ NÃO usado (instance variables)
readonly animalCard: Locator;

constructor(page: Page) {
    super(page);
    this.animalCard = page.locator('[data-testid="animal-card"]');
}

// ✅ USADO (inline locators)
async getAnimalCount(): Promise<number> {
    return await this.page.locator('[data-testid="animal-card"]').count();
}
```

**Por quê?**

- **Frescura**: Locators são sempre recalculados, capturando estado atual do DOM
- **Menos boilerplate**: Não precisa declarar propriedades
- **Melhor com elementos dinâmicos**: Ideal para listas que mudam
- **Mais próximo do Playwright moderno**: Alinha com as recomendações atuais

#### Métodos de Espera

```typescript
async waitForAnimalsToLoad() {
    const firstCard = this.page.locator('[data-testid="animal-card"]').first();
    await this.waitForElementToBeVisible(firstCard);
}
```

**Uso**: Sempre chamar este método após navegar para a página de animais.

```typescript
await pm.navigateToAnimals(1);

const animalsPage = pm.getAnimalsPage();
await animalsPage.waitForAnimalsToLoad(); // CRÍTICO

// Agora podemos fazer asserções
const count = await animalsPage.getAnimalCount();
```

#### Métodos de Contagem

```typescript
async getAnimalCount(): Promise<number> {
    return await this.page.locator('[data-testid="animal-card"]').count();
}
```

**Uso típico**:

```typescript
const count = await animalsPage.getAnimalCount();
expect(count).toBe(6);
```

#### Métodos de Seleção

```typescript
async selectAnimalByIndex(index: number) {
    await this.page.locator('[data-testid="animal-card"]').nth(index).click();
}

async selectAnimalByName(name: string) {
    await this.page.locator(`[data-testid="animal-name-link"]:has-text("${name}")`).first().click();
}
```

**Diferença**:

- `selectAnimalByIndex`: Seleção posicional (útil quando não importa qual animal)
- `selectAnimalByName`: Seleção semântica (quando queremos um animal específico)

**Exemplo**:

```typescript
// Clicar no primeiro animal (não importa qual)
await animalsPage.selectAnimalByIndex(0);

// Clicar em "Rex Mock" especificamente
await animalsPage.selectAnimalByName('Rex Mock');
```

#### Métodos de Extração de Dados

```typescript
async getFirstAnimalName(): Promise<string> {
    const text = await this.page.locator('[data-testid="animal-card"]').first()
        .locator('[data-testid="animal-name-link"]').textContent();
    return text ?? '';
}

async getAnimalNameByIndex(index: number): Promise<string> {
    const text = await this.page.locator('[data-testid="animal-card"]').nth(index)
        .locator('[data-testid="animal-name-link"]').textContent();
    return text ?? '';
}

async getAllAnimalNames(): Promise<string[]> {
    return await this.page.locator('[data-testid="animal-name-link"]').allTextContents();
}
```

**Padrão**: Sempre retornar string vazia em vez de `null` para simplificar os testes.

**Uso**:

```typescript
// Validar nome do primeiro animal
const firstName = await animalsPage.getFirstAnimalName();
expect(firstName).toBe('Rex Mock');

// Validar todos os nomes
const allNames = await animalsPage.getAllAnimalNames();
expect(allNames).toHaveLength(6);
expect(allNames).toContain('Luna Mock');
```

#### Métodos de Validação de Estado

```typescript
async isEmptyStateVisible(): Promise<boolean> {
    return await this.isElementVisible(this.page.locator('[data-testid="animal-list-empty"]'));
}

async isAnimalListVisible(): Promise<boolean> {
    return await this.isElementVisible(this.page.locator('[data-testid="animal-list"]'));
}

async isLoadingSpinnerVisible(): Promise<boolean> {
    return await this.isElementVisible(this.page.locator('[data-testid="loading-spinner"]'));
}
```

**Uso típico**:

```typescript
// Testar estado vazio
await apiMock.mockApiCall('**/api/animals*', mockAnimalsEmpty);
await pm.navigateToAnimals(1);

const isEmptyStateVisible = await animalsPage.isEmptyStateVisible();
expect(isEmptyStateVisible).toBe(true);

const message = await animalsPage.getNoResultsMessage();
expect(message).toContain('Nenhum animal encontrado');
```

#### Métodos de Interação Avançada

```typescript
async hoverAnimalCard(index: number) {
    await this.page.locator('[data-testid="animal-card"]').nth(index).hover();
}

async isAnimalImageLoaded(index: number): Promise<boolean> {
    const image = this.page.locator('[data-testid="animal-card"]').nth(index)
        .locator('[data-testid="animal-image"]');

    const naturalWidth = await image.evaluate((img: HTMLImageElement) => img.naturalWidth);
    return naturalWidth > 0;
}

async getGridColumnCount(): Promise<number> {
    const grid = this.page.locator('[data-testid="animals-grid"]');
    const gridTemplateColumns = await grid.evaluate((el) => {
        return window.getComputedStyle(el).gridTemplateColumns;
    });

    return gridTemplateColumns.split(' ').length;
}
```

**Casos de uso**:

- `hoverAnimalCard`: Testar efeitos hover (animações, tooltips)
- `isAnimalImageLoaded`: Validar que imagens carregaram corretamente
- `getGridColumnCount`: Testar responsividade (grid muda com viewport)

**Exemplo de teste de responsividade**:

```typescript
// Desktop: 4 colunas
await page.setViewportSize({ width: 1920, height: 1080 });
let columns = await animalsPage.getGridColumnCount();
expect(columns).toBe(4);

// Tablet: 2 colunas
await page.setViewportSize({ width: 768, height: 1024 });
columns = await animalsPage.getGridColumnCount();
expect(columns).toBe(2);

// Mobile: 1 coluna
await page.setViewportSize({ width: 375, height: 667 });
columns = await animalsPage.getGridColumnCount();
expect(columns).toBe(1);
```

---

## 6. Component Objects

### 6.1. Conceito

**Component Objects** são similares a Page Objects, mas representam **componentes reutilizáveis** que aparecem em múltiplas páginas.

**Diferença de Page Objects**:

- Page Objects = Página completa
- Component Objects = Componente dentro de uma ou mais páginas

**Exemplo**: O componente de paginação aparece em múltiplas páginas (animais, fosterings, etc.), logo é um Component Object.

### 6.2. PaginationComponent

**Localização**: `e2e/components/PaginationComponent.ts`

```typescript
import { Page } from '@playwright/test';
import {BasePage} from "../core/base.page";

class PaginationComponent extends BasePage {
    constructor(page: Page) {
        super(page);
    }
    
    // Métodos específicos de paginação
}
```

#### Métodos de Visibilidade

```typescript
async isPaginationVisible(): Promise<boolean> {
    return await this.isElementVisible(this.page.locator('[data-testid="pagination"]'));
}
```

**Uso**: Validar que a paginação aparece quando há múltiplas páginas.

#### Métodos de Navegação

```typescript
async goToNextPage() {
    await this.page.locator('[data-testid="pagination-next"]').click();
}

async goToPreviousPage() {
    await this.page.locator('[data-testid="pagination-previous"]').click();
}

async goToPage(pageNumber: number) {
    await this.page.locator(`[data-testid="pagination-page-${pageNumber}"]`).click();
}
```

**Exemplo de teste**:

```typescript
const pagination = pm.getPaginationComponent();

// Ir para próxima página
await pagination.goToNextPage();
await page.waitForURL('**/animals?page=2');

// Ir para página específica
await pagination.goToPage(3);
await page.waitForURL('**/animals?page=3');
```

#### Métodos de Estado

```typescript
async getCurrentPage(): Promise<number> {
    const activeButton = this.page.locator('[data-testid^="pagination-page-"][data-active="true"]');
    const testId = await activeButton.getAttribute('data-testid');

    if (!testId) return 1;

    const pageNumber = testId.replace('pagination-page-', '');
    return parseInt(pageNumber, 10);
}

async isNextButtonDisabled(): Promise<boolean> {
    return await this.page.locator('[data-testid="pagination-next"]').isDisabled();
}

async isPreviousButtonDisabled(): Promise<boolean> {
    return await this.page.locator('[data-testid="pagination-previous"]').isDisabled();
}
```

**Uso**: Validar estados de navegação.

```typescript
// Estamos na página 1
const currentPage = await pagination.getCurrentPage();
expect(currentPage).toBe(1);

// Botão "Anterior" está disabled
const isPrevDisabled = await pagination.isPreviousButtonDisabled();
expect(isPrevDisabled).toBe(true);

// Ir para última página
await pagination.goToPage(totalPages);

// Botão "Próxima" está disabled
const isNextDisabled = await pagination.isNextButtonDisabled();
expect(isNextDisabled).toBe(true);
```

#### Métodos de Análise

```typescript
async getVisiblePageNumbers(): Promise<number[]> {
    const buttons = this.page.locator('[data-testid^="pagination-page-"]');
    const count = await buttons.count();

    const pageNumbers: number[] = [];
    for (let i = 0; i < count; i++) {
        const testId = await buttons.nth(i).getAttribute('data-testid');
        if (testId) {
            const pageNumber = testId.replace('pagination-page-', '');
            pageNumbers.push(parseInt(pageNumber, 10));
        }
    }

    return pageNumbers;
}
```

**Uso**: Validar lógica de janela de paginação (mostra 5 páginas de cada vez).

```typescript
// Com 10 páginas totais, na página 5, deve mostrar: 3,4,5,6,7
await pagination.goToPage(5);
const visiblePages = await pagination.getVisiblePageNumbers();
expect(visiblePages).toEqual([3, 4, 5, 6, 7]);
```

#### Métodos Combinados

```typescript
async clickPageAndWaitForLoad(pageNumber: number) {
    await this.goToPage(pageNumber);
    await this.page.waitForLoadState('networkidle');
}

async verifyUrlContainsPage(expectedPage: number): Promise<boolean> {
    const url = this.page.url();
    return url.includes(`page=${expectedPage}`);
}
```

**Uso**: Simplificar testes que precisam de navegação + validação.

### 6.3. NavbarComponent

**Localização**: `e2e/components/navbar.component.ts`

```typescript
import {Locator, Page} from "@playwright/test";

export class NavbarComponent {
    readonly page: Page;
    readonly homeLink : Locator
    readonly animalsLink : Locator
    readonly favoritesLink : Locator
    readonly notificationsLink : Locator
    readonly profileLink : Locator

    constructor(page: Page) {
        this.page = page;
        this.homeLink = page.locator('nav').first().locator('a[href="/"]');
        this.animalsLink = page.locator('nav').first().locator('a[href="/animals"]');
        this.favoritesLink = page.locator('nav').first().locator('a[href="/favorites"]');
        this.notificationsLink = page.locator('nav').first().locator('a[href="/notifications"]');
        this.profileLink = page.locator('nav').first().locator('a[href="/user/profile"]');
    }

    async goToHome() {
        await this.homeLink.click();
    }

    async goToAnimals() {
        await this.animalsLink.click();
    }

    async getActiveLink(): Promise<Locator> {
        return this.page.locator('nav').first().locator('a[class*="Active"]');
    }
}
```

**Nota**: Este componente usa **instance variables** para locators porque:

- Os links da navbar são estáticos (não mudam dinamicamente)
- Melhor performance (locators calculados uma vez)
- Mais legível para componentes com muitos elementos fixos

**Uso**:

```typescript
const navbar = new NavbarComponent(page);

await navbar.goToAnimals();
await page.waitForURL('**/animals');

const activeLink = await navbar.getActiveLink();
expect(await activeLink.getAttribute('href')).toBe('/animals');
```

---

## 7. Page Manager

**Localização**: `e2e/page.manager.ts`

### 7.1. Conceito

O `PageManager` é o **ponto central de acesso** a todos os Page Objects e Component Objects. Funciona como uma **factory** que:

- Instancia Pages e Components
- Gerencia navegação
- É injetado automaticamente nos testes via fixture `pm`

### 7.2. Implementação

```typescript
import {AnimalsPage} from "./pages/animals.page";
import {Page} from "@playwright/test";
import PaginationComponent from "./components/PaginationComponent";

export class PageManager {
    private animalsPage: AnimalsPage;
    private paginationComponent: PaginationComponent;

    constructor(private page: Page) {
        this.animalsPage = new AnimalsPage(page);
        this.paginationComponent = new PaginationComponent(page);
    }

    getAnimalsPage(): AnimalsPage {
        return this.animalsPage;
    }

    getPaginationComponent(): PaginationComponent {
        return this.paginationComponent;
    }

    async navigateToAnimals(page: number = 1) {
        await this.page.goto(`/animals?page=${page}`);
    }

    async navigateToHome() {
        await this.page.goto('/');
    }
}
```

### 7.3. Benefícios

#### Centralização

Todos os Page/Component Objects são acessados através do `pm`:

```typescript
const animalsPage = pm.getAnimalsPage();
const pagination = pm.getPaginationComponent();
```

Em vez de:

```typescript
// ❌ Verboso e repetitivo
const animalsPage = new AnimalsPage(page);
const pagination = new PaginationComponent(page);
```

#### Navegação Centralizada

```typescript
async navigateToAnimals(page: number = 1) {
    await this.page.goto(`/animals?page=${page}`);
}
```

**Vantagens**:

- URL centralizada (se mudar, só alterar aqui)
- Navegação consistente em todos os testes
- Parâmetros default (página 1)

#### Expansibilidade

Quando adicionar uma nova página:

```typescript
// 1. Criar o Page Object
// e2e/pages/fosterings.page.ts
export class FosteringsPage extends BasePage { ... }

// 2. Adicionar ao PageManager
export class PageManager {
    private fosteringsPage: FosteringsPage;

    constructor(private page: Page) {
        // ...
        this.fosteringsPage = new FosteringsPage(page);
    }

    getFosteringsPage(): FosteringsPage {
        return this.fosteringsPage;
    }

    async navigateToFosterings() {
        await this.page.goto('/fosterings');
    }
}

// 3. Usar nos testes
const fosteringsPage = pm.getFosteringsPage();
```

---

## 8. Fixtures Customizadas

**Localização**: `e2e/fixtures/base.fixture.ts`

### 8.1. Conceito

Fixtures no Playwright são **mecanismos de injeção de dependências** que permitem:

- Setup automático de recursos antes dos testes
- Teardown automático após os testes
- Reutilização de código de setup
- Isolamento entre testes

### 8.2. Implementação

```typescript
import { test as base } from '@playwright/test';
import {PageManager} from "../page.manager";
import {ApiMockHelper} from "../core/ApiMockHelper";
import {ApiInterceptHelper} from "../core/ApiInterceptHelper";

type CustomFixtures = {
    pm: PageManager;
    apiMock: ApiMockHelper;
    apiIntercept: ApiInterceptHelper;
};

export const test = base.extend<CustomFixtures>({
    pm: async ({ page }, provide) => {
        const pageManager = new PageManager(page);
        await provide(pageManager);
    },

    apiMock: async ({ page }, provide) => {
        const apiMockHelper = new ApiMockHelper(page);
        await provide(apiMockHelper);
    },

    apiIntercept: async ({ page }, provide) => {
        const apiInterceptHelper = new ApiInterceptHelper(page);
        await provide(apiInterceptHelper);
    },
});

export { expect } from '@playwright/test';
```

### 8.3. Como Funciona

#### Definição de Tipos

```typescript
type CustomFixtures = {
    pm: PageManager;
    apiMock: ApiMockHelper;
    apiIntercept: ApiInterceptHelper;
};
```

Define os tipos das fixtures customizadas para TypeScript autocomplete.

#### Extensão do Test

```typescript
export const test = base.extend<CustomFixtures>({ ... });
```

Cria um novo objeto `test` que **estende** o `test` base do Playwright com as fixtures customizadas.

#### Fixture Factory

```typescript
pm: async ({ page }, provide) => {
    const pageManager = new PageManager(page);
    await provide(pageManager);
}
```

**Como funciona**:

1. Fixture recebe `page` (do Playwright base)
2. Cria instância de `PageManager` com esse `page`
3. `provide()` disponibiliza a instância para o teste

#### Uso nos Testes

```typescript
import { test, expect } from '../fixtures/base.fixture';

test('meu teste', async ({ pm, apiMock, page }) => {
    // pm, apiMock, apiIntercept são injetados automaticamente
    // page também está disponível (do Playwright base)
    
    await pm.navigateToAnimals(1);
    const animalsPage = pm.getAnimalsPage();
    // ...
});
```

**Benefícios**:

- Não precisa instanciar manualmente
- Sempre recebe instâncias novas (isolamento)
- Código de teste mais limpo
- Fácil adicionar novas fixtures

### 8.4. Expandindo Fixtures

**Exemplo**: Adicionar fixture de autenticação (futuro)

```typescript
type CustomFixtures = {
    pm: PageManager;
    apiMock: ApiMockHelper;
    apiIntercept: ApiInterceptHelper;
    authenticatedUser: { token: string; user: User }; // Nova fixture
};

export const test = base.extend<CustomFixtures>({
    // ... fixtures existentes

    authenticatedUser: async ({ page, apiMock }, provide) => {
        // Setup: fazer login
        const mockUser = { id: '1', name: 'Test User', role: 'User' };
        const mockToken = 'test-jwt-token';

        await apiMock.mockApiCall('**/api/auth/login', {
            token: mockToken,
            user: mockUser
        });

        await page.goto('/login');
        await page.fill('[data-testid="email"]', 'test@example.com');
        await page.fill('[data-testid="password"]', 'password');
        await page.click('[data-testid="login-button"]');

        await page.waitForURL('**/');

        await provide({ token: mockToken, user: mockUser });

        // Teardown (opcional)
        // Será executado automaticamente após o teste
    },
});
```

**Uso**:

```typescript
test('teste que requer autenticação', async ({ pm, authenticatedUser }) => {
    // Já estamos autenticados!
    console.log(authenticatedUser.user.name); // "Test User"
    
    await pm.navigateToAnimals(1);
    // ...
});
```

---

## 9. API Helpers

### 9.1. Quando Usar Mock vs Intercept

#### Use ApiMockHelper quando:

✅ Testar comportamento da UI isoladamente  
✅ Testes rápidos (não dependem do backend)  
✅ Testar estados de loading e erro  
✅ Testar com dados específicos e controlados  
✅ Backend não está disponível  
✅ Dados de teste difíceis de criar no backend  

**Exemplo**:

```typescript
// Testar estado vazio
await apiMock.mockApiCall('**/api/animals*', { items: [], totalCount: 0 });

// Testar erro 500
await apiMock.mockError('**/api/animals*', 500, 'Server Error');

// Testar loading state
await apiMock.mockWithDelay('**/api/animals*', mockData, 2000);
```

#### Use ApiInterceptHelper quando:

✅ Testar integração real com backend  
✅ Modificar casos edge específicos de dados reais  
✅ Validar tratamento de dados inesperados  
✅ Capturar e validar respostas da API  
✅ Testar com dados do backend mas com pequenas modificações  

**Exemplo**:

```typescript
// Modificar dados reais para testar edge case
await apiIntercept.interceptAndModify<PagedList<Animal>>(
    '**/api/animals',
    (response) => {
        // Nome muito longo (teste de truncamento)
        response.items[0].name = 'A'.repeat(200);
        // Breed null (teste de fallback)
        response.items[1].breed = null;
        return response;
    }
);
```

### 9.2. Padrões Avançados

#### Combinar Mock e Intercept

```typescript
test('teste combinado', async ({ pm, apiMock, apiIntercept }) => {
    // Mockar página 1
    await apiMock.mockApiCall('**/api/animals?pageNumber=1', mockPage1);

    // Interceptar e modificar página 2 (dados reais)
    await apiIntercept.interceptAndModify(
        '**/api/animals?pageNumber=2',
        (response) => {
            response.items = response.items.slice(0, 5); // Limitar a 5
            return response;
        }
    );

    // Navegar
    await pm.navigateToAnimals(1);
    
    // Página 1 usa dados mock
    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();
    
    let count = await animalsPage.getAnimalCount();
    expect(count).toBe(mockPage1.items.length);

    // Ir para página 2 (dados reais modificados)
    const pagination = pm.getPaginationComponent();
    await pagination.goToPage(2);
    await animalsPage.waitForAnimalsToLoad();

    count = await animalsPage.getAnimalCount();
    expect(count).toBe(5);
});
```

#### Mock Condicional

```typescript
// Mock diferente baseado em query params
await page.route('**/api/animals**', async (route) => {
    const url = new URL(route.request().url());
    const pageNumber = url.searchParams.get('pageNumber');

    if (pageNumber === '1') {
        await route.fulfill({
            status: 200,
            body: JSON.stringify(mockPage1)
        });
    } else if (pageNumber === '2') {
        await route.fulfill({
            status: 200,
            body: JSON.stringify(mockPage2)
        });
    } else {
        await route.fulfill({
            status: 404,
            body: JSON.stringify({ message: 'Not Found' })
        });
    }
});
```

---

## 10. Test Data

**Localização**: `e2e/test-data/`

### 10.1. Organização

```
test-data/
└── AnimalsPage/
    ├── animals-page1.ts         # Dados completos de animais
    └── mockAnimalsPage1.ts      # Casos especiais (vazio, etc.)
```

**Convenção**:

- Uma pasta por feature/página
- Nomes descritivos do cenário
- Export de constantes tipadas

### 10.2. Exemplo Completo

**Ficheiro**: `e2e/test-data/AnimalsPage/animals-page1.ts`

```typescript
export const mockAnimalsPage1 = {
    items: [
        {
            id: "mock-animal-001",
            name: "Rex Mock",
            species: "Dog",
            size: "Medium",
            sex: "Male",
            breed: {
                id: "breed-001",
                name: "Labrador Retriever",
                description: ""
            },
            animalState: "Available",
            colour: "Brown",
            birthDate: "2020-01-01",
            age: 4,
            description: "Energetic and loyal mock dog.",
            sterilized: true,
            features: "Friendly, active",
            cost: 0,
            shelterId: "shelter-mock-01",
            images: []
        },
        // ... mais animais
    ],
    currentPage: 1,
    pageSize: 20,
    totalPages: 1,
    totalCount: 6
};
```

**Boas práticas**:

- IDs com prefixo "mock-" para distinguir de dados reais
- Dados realistas mas identificáveis
- Cobrir diversos cenários (diferentes espécies, tamanhos, etc.)

### 10.3. Cenários Especiais

**Ficheiro**: `e2e/test-data/AnimalsPage/mockAnimalsPage1.ts`

```typescript
export const mockAnimalsEmpty = {
    items: [],
    currentPage: 1,
    pageSize: 1,
    totalPages: 1,
    totalCount: 0,
}
```

**Outros cenários úteis** (a criar):

```typescript
// Animal com nome muito longo
export const mockAnimalLongName = {
    items: [{
        id: "mock-long-name",
        name: "A".repeat(200),
        // ...
    }],
    // ...
};

// Animal sem imagens
export const mockAnimalNoImages = {
    items: [{
        id: "mock-no-images",
        images: [],
        // ...
    }],
    // ...
};

// Animal com breed null
export const mockAnimalNoBreed = {
    items: [{
        id: "mock-no-breed",
        breed: null,
        // ...
    }],
    // ...
};
```

### 10.4. Builders para Test Data (Futuro)

Para cenários mais complexos, considerar builders:

```typescript
class AnimalBuilder {
    private animal: Animal = {
        id: 'test-id',
        name: 'Test Animal',
        species: 'Dog',
        age: 1,
        // ... defaults
    };

    withId(id: string): this {
        this.animal.id = id;
        return this;
    }

    withName(name: string): this {
        this.animal.name = name;
        return this;
    }

    withNoBreed(): this {
        this.animal.breed = null;
        return this;
    }

    build(): Animal {
        return this.animal;
    }
}

// Uso
const animal = new AnimalBuilder()
    .withName('Custom Name')
    .withNoBreed()
    .build();
```

---

## 11. Escrita de Testes

**Localização**: `e2e/tests/`

### 11.1. Estrutura de um Teste

**Ficheiro**: `e2e/tests/animals.spec.ts`

```typescript
import { test, expect } from '../fixtures/base.fixture';
import {mockAnimalsPage1} from '../test-data/AnimalsPage/animals-page1';

test.describe('Animals List - With API Mocking', () => {

    test('should load page instantly with mocked data', async ({ pm, apiMock }) => {
        // Arrange
        await apiMock.mockApiCall('**/api/animals?**', mockAnimalsPage1);
        const startTime = Date.now();

        // Act
        await pm.navigateToAnimals(1);
        const animalsPage = pm.getAnimalsPage();
        await animalsPage.waitForAnimalsToLoad();

        // Assert
        const loadTime = Date.now() - startTime;
        expect(loadTime).toBeLessThan(2000);

        const count = await animalsPage.getAnimalCount();
        expect(count).toBe(mockAnimalsPage1.items.length);
    });
});
```

#### Anatomia

**1. Imports**:

```typescript
import { test, expect } from '../fixtures/base.fixture';
import {mockAnimalsPage1} from '../test-data/AnimalsPage/animals-page1';
```

- `test` e `expect` da fixture customizada (não do Playwright base)
- Test data importado

**2. Describe Block**:

```typescript
test.describe('Animals List - With API Mocking', () => {
    // Testes relacionados agrupados
});
```

**3. Test Case**:

```typescript
test('should load page instantly with mocked data', async ({ pm, apiMock }) => {
    // Arrange - Act - Assert
});
```

**Fixtures injetadas**: `pm`, `apiMock` (e outros conforme necessário)

### 11.2. Padrão AAA (Arrange-Act-Assert)

#### Arrange (Preparar)

```typescript
// Setup de mocks
await apiMock.mockApiCall('**/api/animals?**', mockAnimalsPage1);

// Dados de teste
const startTime = Date.now();
```

#### Act (Agir)

```typescript
// Ação do utilizador
await pm.navigateToAnimals(1);

const animalsPage = pm.getAnimalsPage();
await animalsPage.waitForAnimalsToLoad();
```

#### Assert (Verificar)

```typescript
// Validações
const loadTime = Date.now() - startTime;
expect(loadTime).toBeLessThan(2000);

const count = await animalsPage.getAnimalCount();
expect(count).toBe(mockAnimalsPage1.items.length);
```

### 11.3. Tipos de Testes

#### Testes com Mock Completo

**Uso**: Testar comportamento da UI isoladamente.

```typescript
test('should display mocked animal data correctly', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    const firstAnimalName = await animalsPage.getAnimalNameByIndex(0);
    expect(firstAnimalName).toBe(mockAnimalsPage1.items[0].name);

    const firstAnimalBreed = await animalsPage.getAnimalBreedByIndex(0);
    expect(firstAnimalBreed).toBe(mockAnimalsPage1.items[0].breed.name);
});
```

#### Testes com Intercept

**Uso**: Testar edge cases com dados reais modificados.

```typescript
test('should modify first animal in response', async ({ pm, apiIntercept }) => {
    await apiIntercept.interceptAndModify<PagedList<Animal>>(
        '**/api/animals',
        (response) => {
            if (response.items.length > 0) {
                response.items[0] = {
                    ...response.items[0],
                    name: 'Modified Animal Name',
                    breed: {
                        ...response.items[0].breed,
                        name: 'Modified Breed'
                    }
                };
            }
            return response;
        }
    );

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    const firstName = await animalsPage.getAnimalNameByIndex(0);
    expect(firstName).toBe('Modified Animal Name');

    const firstBreed = await animalsPage.getAnimalBreedByIndex(0);
    expect(firstBreed).toBe('Modified Breed');
});
```

#### Testes de Estado Vazio

```typescript
test('should handle empty state with mocked empty data', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsEmpty);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();

    const isEmptyStateVisible = await animalsPage.isEmptyStateVisible();
    expect(isEmptyStateVisible).toBe(true);

    const message = await animalsPage.getNoResultsMessage();
    expect(message).toContain('Nenhum animal encontrado');
});
```

#### Testes de Loading State

```typescript
test('should simulate loading state with delayed mock', async ({ pm, apiMock }) => {
    await apiMock.mockWithDelay('**/api/animals*', mockAnimalsPage1, 1000);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();

    // Validar que loading apareceu
    // (nota: pode ser muito rápido para capturar em teste real)

    await animalsPage.waitForAnimalsToLoad();

    const count = await animalsPage.getAnimalCount();
    expect(count).toBe(mockAnimalsPage1.items.length);
});
```

#### Testes de Erro

```typescript
test('should test error handling with mocked 500 error', async ({ pm, apiMock, page }) => {
    await apiMock.mockError('**/api/animals*', 500, 'Internal Server Error');

    await pm.navigateToAnimals(1);

    await page.waitForLoadState('networkidle');

    // Validar que erro é mostrado
    // (depende de como o erro é tratado no frontend)

    const currentUrl = page.url();
    expect(currentUrl).toContain('/animals');
});
```

#### Testes de Performance

```typescript
test('should handle large dataset efficiently', async ({ pm, apiMock }) => {
    const largeDataset = {
        items: Array(20).fill(null).map((_, i) => ({
            id: `animal-${i}`,
            name: `Animal ${i}`,
            age: Math.floor(Math.random() * 10) + 1,
            breed: { id: `breed-${i}`, name: `Breed ${i}` },
            images: []
        })),
        pageNumber: 1,
        totalPages: 10,
        totalCount: 200,
        pageSize: 20,
        hasPreviousPage: false,
        hasNextPage: true
    };

    await apiMock.mockApiCall('**/api/animals*', largeDataset);

    const startTime = Date.now();

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    const loadTime = Date.now() - startTime;

    expect(loadTime).toBeLessThan(3000);

    const count = await animalsPage.getAnimalCount();
    expect(count).toBe(20);
});
```

#### Testes de Paginação

```typescript
test('should test pagination with intercepted different page data', async ({ pm, apiIntercept, page }) => {
    await apiIntercept.interceptAndModify<PagedList<Animal>>(
        '**/api/animals?pageNumber=2',
        (response) => {
            response.items = response.items.map((item, index) => ({
                ...item,
                name: `Page 2 Animal ${index + 1}`
            }));
            return response;
        }
    );

    await pm.navigateToAnimals(1);

    const paginationComponent = pm.getPaginationComponent();
    const animalsPage = pm.getAnimalsPage();

    await paginationComponent.waitForPaginationToLoad();

    const totalPages = await paginationComponent.getTotalVisiblePages();

    if (totalPages >= 2) {
        await paginationComponent.goToPage(2);
        await page.waitForURL('**/animals?page=2');
        await animalsPage.waitForAnimalsToLoad();

        const firstName = await animalsPage.getAnimalNameByIndex(0);
        expect(firstName).toContain('Page 2 Animal');
    }
});
```

### 11.4. Boas Práticas de Nomenclatura

#### Describe Blocks

```typescript
// ✅ Bom - Agrupa testes relacionados
test.describe('Animals List - With API Mocking', () => { ... });
test.describe('Animals List - With API Interception', () => { ... });
test.describe('Animals List - Performance Testing', () => { ... });

// ❌ Evitar - Muito genérico
test.describe('Tests', () => { ... });
```

#### Test Names

```typescript
// ✅ Bom - Comportamento esperado claro
test('should load page instantly with mocked data', ...);
test('should display mocked animal data correctly', ...);
test('should handle empty state with mocked empty data', ...);

// ❌ Evitar - Não descreve comportamento
test('test 1', ...);
test('animals page', ...);
```

**Padrão**: `should [ação/comportamento] [contexto]`

---

## 12. Configuração do Playwright

**Localização**: `playwright.config.ts`

### 12.1. Configuração Completa

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.CI ? 'http://localhost:3000' : 'https://localhost:3000',
    trace: process.env.CI ? 'on' : "on-first-retry",
    headless: !!process.env.CI,
    launchOptions: {
      slowMo: process.env.CI ? 0 : 1000
    }
  },
  outputDir: 'test-results',
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
  ],
});
```

### 12.2. Explicação das Configurações

#### Test Directory

```typescript
testDir: './e2e'
```

Pasta raiz dos testes E2E.

#### Parallel Execution

```typescript
fullyParallel: true
```

Executa testes em paralelo para maior velocidade.

**Nota**: Cada teste deve ser independente (não compartilhar estado).

#### Forbid Only

```typescript
forbidOnly: !!process.env.CI
```

Em CI, falha se algum teste tiver `.only` (evita commits acidentais de testes focados).

#### Retries

```typescript
retries: process.env.CI ? 2 : 0
```

- **CI**: 2 retries (testes podem falhar por issues de rede/timing)
- **Local**: 0 retries (falhas devem ser investigadas imediatamente)

#### Workers

```typescript
workers: process.env.CI ? 1 : undefined
```

- **CI**: 1 worker (ambiente limitado)
- **Local**: Automático baseado em CPU

#### Reporter

```typescript
reporter: 'html'
```

Gera relatório HTML visual dos resultados.

**Outros reporters úteis**:

```typescript
reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/junit.xml' }],
    ['json', { outputFile: 'test-results/results.json' }]
]
```

#### Base URL

```typescript
baseURL: process.env.CI ? 'http://localhost:3000' : 'https://localhost:3000'
```

- **CI**: HTTP (sem certificados SSL)
- **Local**: HTTPS (desenvolvimento com certificados)

**Uso**: Permite usar URLs relativas nos testes.

```typescript
// Em vez de
await page.goto('https://localhost:3000/animals');

// Pode usar
await page.goto('/animals');
```

#### Trace

```typescript
trace: process.env.CI ? 'on' : "on-first-retry"
```

- **CI**: Sempre gerar trace (para debugging de falhas)
- **Local**: Só na primeira retry (economizar disco)

**Trace**: Gravação completa da execução do teste (screenshots, network, console, etc.).

#### Headless

```typescript
headless: !!process.env.CI
```

- **CI**: Headless (sem UI)
- **Local**: Com UI (ver o que está acontecendo)

#### Slow Motion

```typescript
launchOptions: {
    slowMo: process.env.CI ? 0 : 1000
}
```

- **CI**: Velocidade normal
- **Local**: 1 segundo de pausa entre ações (facilitar visualização)

#### Output Directory

```typescript
outputDir: 'test-results'
```

Pasta para screenshots, vídeos, traces.

#### Projects (Multi-browser)

```typescript
projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
]
```

Testa em múltiplos browsers.

**Executar browsers específicos**:

```bash
# Só Chromium
npx playwright test --project=chromium

# Só Firefox
npx playwright test --project=firefox

# Todos
npx playwright test
```

### 12.3. Configurações Adicionais Úteis

#### Timeouts

```typescript
use: {
    // Timeout de cada ação (click, fill, etc.)
    actionTimeout: 10000,
    
    // Timeout de navegação
    navigationTimeout: 30000,
}
```

#### Screenshot

```typescript
use: {
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Ou sempre
    screenshot: 'on',
}
```

#### Video

```typescript
use: {
    // Gravar vídeo só em falhas
    video: 'retain-on-failure',
    
    // Ou sempre
    video: 'on',
}
```

#### Viewport

```typescript
use: {
    // Tamanho padrão da janela
    viewport: { width: 1920, height: 1080 },
}
```

---

## 13. Boas Práticas

### 13.1. Princípios Gerais

#### 1. Testes Independentes

**❌ Errado**:

```typescript
let sharedData: Animal[];

test('test 1', async ({ pm }) => {
    sharedData = await getAnimals();
});

test('test 2', async ({ pm }) => {
    // Depende de test 1
    expect(sharedData).toBeDefined();
});
```

**✅ Correto**:

```typescript
test('test 1', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockData);
    const animalsPage = pm.getAnimalsPage();
    // Teste completo e isolado
});

test('test 2', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockData);
    const animalsPage = pm.getAnimalsPage();
    // Completamente independente
});
```

#### 2. Data-testid vs CSS Selectors

**❌ Evitar**:

```typescript
// Frágil - quebra se CSS mudar
const button = page.locator('.button.primary.large');
const name = page.locator('div > h2.animal-name');
```

**✅ Preferir**:

```typescript
// Robusto - explicitamente para testes
const button = page.locator('[data-testid="submit-button"]');
const name = page.locator('[data-testid="animal-name"]');
```

**Por quê?**

- Classes CSS podem mudar por motivos de estilo
- `data-testid` é explicitamente para testes
- Mudanças na estrutura HTML não quebram testes

#### 3. Esperas Explícitas

**❌ Evitar**:

```typescript
await page.goto('/animals');
await page.waitForTimeout(2000); // Espera fixa
const count = await page.locator('[data-testid="animal-card"]').count();
```

**✅ Preferir**:

```typescript
await page.goto('/animals');
const animalsPage = pm.getAnimalsPage();
await animalsPage.waitForAnimalsToLoad(); // Espera condicional
const count = await animalsPage.getAnimalCount();
```

**Por quê?**

- Esperas fixas são não-determinísticas
- Podem ser muito curtas (flaky) ou muito longas (lentas)
- Esperas condicionais são mais confiáveis

#### 4. Asserções Específicas

**❌ Evitar**:

```typescript
const name = await animalsPage.getAnimalNameByIndex(0);
expect(name).toBeTruthy(); // Muito genérico
```

**✅ Preferir**:

```typescript
const name = await animalsPage.getAnimalNameByIndex(0);
expect(name).toBe('Rex Mock'); // Específico e claro
```

#### 5. Um Conceito por Teste

**❌ Evitar**:

```typescript
test('test everything', async ({ pm, apiMock }) => {
    // Setup
    await apiMock.mockApiCall('**/api/animals*', mockData);
    await pm.navigateToAnimals(1);
    
    // Test loading
    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();
    
    // Test count
    const count = await animalsPage.getAnimalCount();
    expect(count).toBe(6);
    
    // Test names
    const names = await animalsPage.getAllAnimalNames();
    expect(names).toContain('Rex Mock');
    
    // Test pagination
    const pagination = pm.getPaginationComponent();
    await pagination.goToNextPage();
    
    // Test navigation
    await animalsPage.selectAnimalByIndex(0);
    
    // ... muitos mais testes
});
```

**✅ Preferir**:

```typescript
test('should display correct animal count', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockData);
    await pm.navigateToAnimals(1);
    
    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();
    
    const count = await animalsPage.getAnimalCount();
    expect(count).toBe(6);
});

test('should display correct animal names', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockData);
    await pm.navigateToAnimals(1);
    
    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();
    
    const names = await animalsPage.getAllAnimalNames();
    expect(names).toContain('Rex Mock');
});

test('should navigate to next page', async ({ pm, apiMock }) => {
    // ...
});
```

**Por quê?**

- Testes focados são mais fáceis de debugar
- Falha de um teste não obscurece outros problemas
- Melhor isolamento

### 13.2. Organização de Código

#### Agrupar Testes Relacionados

```typescript
test.describe('Animals List - Happy Path', () => {
    test('should load successfully', ...);
    test('should display animal cards', ...);
    test('should navigate between pages', ...);
});

test.describe('Animals List - Error Handling', () => {
    test('should handle 404 error', ...);
    test('should handle 500 error', ...);
    test('should handle network timeout', ...);
});

test.describe('Animals List - Edge Cases', () => {
    test('should handle empty list', ...);
    test('should handle very long names', ...);
    test('should handle missing images', ...);
});
```

#### Setup Compartilhado com beforeEach

```typescript
test.describe('Animals List Tests', () => {
    test.beforeEach(async ({ pm, apiMock }) => {
        // Setup comum a todos os testes deste grupo
        await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);
        await pm.navigateToAnimals(1);
    });

    test('should display correct count', async ({ pm }) => {
        const animalsPage = pm.getAnimalsPage();
        await animalsPage.waitForAnimalsToLoad();
        
        const count = await animalsPage.getAnimalCount();
        expect(count).toBe(6);
    });

    test('should display correct names', async ({ pm }) => {
        const animalsPage = pm.getAnimalsPage();
        await animalsPage.waitForAnimalsToLoad();
        
        const name = await animalsPage.getFirstAnimalName();
        expect(name).toBe('Rex Mock');
    });
});
```

### 13.3. Debugging

#### Playwright Inspector

```bash
# Executar testes com inspector
npx playwright test --debug

# Executar teste específico com inspector
npx playwright test animals.spec.ts --debug
```

#### Trace Viewer

```bash
# Executar testes e gerar trace
npx playwright test --trace on

# Abrir trace viewer
npx playwright show-trace trace.zip
```

#### Screenshots e Vídeos

```typescript
test('debug test', async ({ page }) => {
    await page.goto('/animals');
    
    // Screenshot manual
    await page.screenshot({ path: 'debug.png' });
    
    // Screenshot de elemento específico
    const card = page.locator('[data-testid="animal-card"]').first();
    await card.screenshot({ path: 'card.png' });
});
```

#### Console Logs

```typescript
test('debug with logs', async ({ page, pm }) => {
    // Capturar console do browser
    page.on('console', msg => console.log('Browser:', msg.text()));
    
    await pm.navigateToAnimals(1);
    
    const animalsPage = pm.getAnimalsPage();
    const count = await animalsPage.getAnimalCount();
    
    console.log('Animal count:', count);
});
```

---

## 14. Padrões de Testes

### 14.1. Testar Estados da UI

#### Loading State

```typescript
test('should show loading spinner', async ({ pm, apiMock }) => {
    // Mock com delay para capturar loading
    await apiMock.mockWithDelay('**/api/animals*', mockData, 2000);

    // Não esperar carregar
    const loadingPromise = pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    
    // Verificar que loading apareceu
    const isLoadingVisible = await animalsPage.isLoadingSpinnerVisible();
    expect(isLoadingVisible).toBe(true);

    // Esperar terminar
    await loadingPromise;
    await animalsPage.waitForAnimalsToLoad();

    // Verificar que loading sumiu
    const isLoadingHidden = await animalsPage.isLoadingSpinnerHidden();
    expect(isLoadingHidden).toBe(true);
});
```

#### Error State

```typescript
test('should show error message on 500', async ({ pm, apiMock }) => {
    await apiMock.mockError('**/api/animals*', 500, 'Server Error');

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    
    const errorMessage = await animalsPage.getErrorMessage();
    expect(errorMessage).toContain('Erro ao carregar');
});
```

#### Empty State

```typescript
test('should show empty state', async ({ pm, apiMock }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsEmpty);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    
    const isEmptyVisible = await animalsPage.isEmptyStateVisible();
    expect(isEmptyVisible).toBe(true);

    const message = await animalsPage.getNoResultsMessage();
    expect(message).toBe('Nenhum animal encontrado');
});
```

### 14.2. Testar Interações

#### Click e Navegação

```typescript
test('should navigate to animal detail', async ({ pm, apiMock, page }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    const animalId = mockAnimalsPage1.items[0].id;
    
    await animalsPage.selectAnimalByIndex(0);

    // Validar navegação
    await page.waitForURL(`**/animals/${animalId}`);
});
```

#### Hover Effects

```typescript
test('should show hover effects', async ({ pm, apiMock, page }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    const card = page.locator('[data-testid="animal-card"]').first();
    
    // Estado inicial
    const initialBoxShadow = await card.evaluate(el => 
        window.getComputedStyle(el).boxShadow
    );

    // Hover
    await animalsPage.hoverAnimalCard(0);

    // Estado após hover
    const hoverBoxShadow = await card.evaluate(el => 
        window.getComputedStyle(el).boxShadow
    );

    // Validar que mudou
    expect(hoverBoxShadow).not.toBe(initialBoxShadow);
});
```

### 14.3. Testar Responsividade

```typescript
test('should adapt grid to viewport size', async ({ pm, apiMock, page }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    // Desktop (4 colunas)
    await page.setViewportSize({ width: 1920, height: 1080 });
    let columns = await animalsPage.getGridColumnCount();
    expect(columns).toBe(4);

    // Tablet (2 colunas)
    await page.setViewportSize({ width: 768, height: 1024 });
    columns = await animalsPage.getGridColumnCount();
    expect(columns).toBe(2);

    // Mobile (1 coluna)
    await page.setViewportSize({ width: 375, height: 667 });
    columns = await animalsPage.getGridColumnCount();
    expect(columns).toBe(1);
});
```

### 14.4. Testar Acessibilidade

```typescript
test('should have proper ARIA labels', async ({ pm, apiMock, page }) => {
    await apiMock.mockApiCall('**/api/animals*', mockAnimalsPage1);

    await pm.navigateToAnimals(1);

    const animalsPage = pm.getAnimalsPage();
    await animalsPage.waitForAnimalsToLoad();

    // Validar aria-label
    const pagination = page.locator('[aria-label="Paginação"]');
    await expect(pagination).toBeVisible();

    const nextButton = page.locator('[aria-label="Próxima página"]');
    await expect(nextButton).toBeVisible();
});
```

---

## 15. Troubleshooting

### 15.1. Problemas Comuns

#### Testes Flaky

**Sintoma**: Testes passam e falham aleatoriamente.

**Causas comuns**:

1. **Esperas inadequadas**:

```typescript
// ❌ Problema
await page.goto('/animals');
const count = await page.locator('[data-testid="animal-card"]').count();

// ✅ Solução
await page.goto('/animals');
const animalsPage = pm.getAnimalsPage();
await animalsPage.waitForAnimalsToLoad();
const count = await animalsPage.getAnimalCount();
```

2. **Timeouts muito curtos**:

```typescript
// ❌ Problema
await element.waitFor({ state: 'visible', timeout: 100 });

// ✅ Solução
await element.waitFor({ state: 'visible', timeout: 5000 });
```

3. **Condições de corrida**:

```typescript
// ❌ Problema
await button.click();
await input.fill('text'); // Pode executar antes do click terminar

// ✅ Solução
await button.click();
await page.waitForLoadState('networkidle');
await input.fill('text');
```

#### Elementos Não Encontrados

**Sintoma**: `Error: Element not found`.

**Soluções**:

1. Verificar data-testid:

```typescript
// Ver HTML real
console.log(await page.content());

// Ou screenshot
await page.screenshot({ path: 'debug.png' });
```

2. Aguardar carregamento:

```typescript
await page.waitForSelector('[data-testid="animal-card"]');
```

3. Verificar visibilidade:

```typescript
const isVisible = await element.isVisible();
console.log('Element visible:', isVisible);
```

#### Mocks Não Funcionam

**Sintoma**: Testes fazem chamadas reais à API.

**Soluções**:

1. Mock antes de navegar:

```typescript
// ✅ Correto
await apiMock.mockApiCall('**/api/animals*', mockData);
await pm.navigateToAnimals(1);

// ❌ Errado (tarde demais)
await pm.navigateToAnimals(1);
await apiMock.mockApiCall('**/api/animals*', mockData);
```

2. Verificar URL pattern:

```typescript
// Verificar que pattern está correto
console.log('URL pattern:', '**/api/animals*');

// Ou usar pattern mais específico
await apiMock.mockApiCall('https://localhost:5001/api/animals', mockData);
```

3. Validar que mock foi chamado:

```typescript
let mockCalled = false;

await page.route('**/api/animals*', async (route) => {
    mockCalled = true;
    await route.fulfill({ 
        status: 200, 
        body: JSON.stringify(mockData) 
    });
});

await pm.navigateToAnimals(1);

console.log('Mock called:', mockCalled);
```

#### Timeouts em CI

**Sintoma**: Testes passam localmente mas falham em CI com timeout.

**Soluções**:

1. Aumentar timeouts:

```typescript
// playwright.config.ts
use: {
    actionTimeout: 30000, // 30s em vez de 10s
}
```

2. Esperar networkidle:

```typescript
await page.goto('/animals');
await page.waitForLoadState('networkidle');
```

3. Reduzir workers em CI:

```typescript
workers: process.env.CI ? 1 : undefined
```

### 15.2. Commands Úteis

```bash
# Executar todos os testes
npx playwright test

# Executar teste específico
npx playwright test animals.spec.ts

# Executar com UI
npx playwright test --ui

# Executar com debug
npx playwright test --debug

# Executar em modo headed
npx playwright test --headed

# Gerar relatório
npx playwright show-report

# Listar testes sem executar
npx playwright test --list

# Executar testes que falharam na última vez
npx playwright test --last-failed

# Executar em browser específico
npx playwright test --project=chromium
```

### 15.3. CI/CD Debugging

**GitHub Actions - Ver logs detalhados**:

1. Adicionar step para upload de artifacts:

```yaml
- name: Upload Playwright Report
  if: always()
  uses: actions/upload-artifact@v3
  with:
    name: playwright-report
    path: playwright-report/
```

2. Ativar trace em CI:

```typescript
// playwright.config.ts
use: {
    trace: process.env.CI ? 'on' : 'on-first-retry',
}
```

3. Ativar screenshots:

```typescript
use: {
    screenshot: 'only-on-failure',
}
```

---

## Conclusão

A arquitetura de testes E2E do SeePaw foi construída com foco em **modularidade**, **manutenibilidade** e **escalabilidade**. Seguindo os padrões e práticas descritos neste guia, conseguimos:

✅ **Testes independentes e isolados**  
✅ **Código reutilizável através de Page/Component Objects**  
✅ **Flexibilidade com Mocking e Interception**  
✅ **Facilidade de manutenção** (mudanças na UI só afetam Page Objects)  
✅ **Debugging eficiente** com ferramentas do Playwright  
✅ **Execução rápida** com parallelização e mocks  
✅ **Cobertura abrangente** de cenários (happy path, erros, edge cases)  

### Próximos Passos

1. **Expandir cobertura**: Adicionar testes para outras páginas (Fosterings, Ownership, etc.)
2. **Testes visuais**: Integrar screenshot comparison
3. **Performance testing**: Adicionar métricas de Web Vitals
4. **Acessibilidade**: Integrar Axe ou Pa11y
5. **Cross-browser**: Adicionar Safari/Edge aos projects

---

**Última atualização**: Novembro 2025  
**Versão**: 1.0  
**Autor**: Equipa SeePaw

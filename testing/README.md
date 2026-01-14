# Testes - Documentação

## Visão Geral

Esta documentação cobre estratégias e boas práticas de testes para o Delupo Dashboard.

⚠️ **Status Atual**: Testes não implementados. Esta documentação serve como guia para implementação futura.

## Estratégia de Testes

### Pirâmide de Testes

```
        /\
       /  \      E2E Tests (10%)
      /____\
     /      \    Integration Tests (30%)
    /________\
   /          \  Unit Tests (60%)
  /____________\
```

## Backend - Testes

### Ferramentas Sugeridas

- **Framework**: Jest ou Vitest
- **Mocks**: jest.mock() ou vi.mock()
- **API Testing**: Supertest
- **Database**: Test containers ou SQLite in-memory

### Instalação (Jest)

```bash
cd delupo/delupo-backend
npm install --save-dev jest @types/jest ts-jest supertest @types/supertest
```

### Configuração (jest.config.js)

```javascript
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
  ],
};
```

### Estrutura de Testes

```
delupo-backend/
├── src/
│   ├── config.ts
│   ├── config.test.ts
│   ├── vtex.ts
│   ├── vtex.test.ts
│   ├── order-processor.ts
│   └── order-processor.test.ts
└── __tests__/
    ├── integration/
    │   ├── api.test.ts
    │   └── ingest.test.ts
    └── e2e/
        └── full-flow.test.ts
```

### Testes Unitários

#### Exemplo: config.test.ts

```typescript
import { describe, it, expect, beforeEach } from '@jest/globals';

describe('Config', () => {
  beforeEach(() => {
    // Reset environment
    delete process.env.VTEX_ACCOUNT;
  });

  it('should throw error when required env is missing', () => {
    expect(() => {
      require('./config');
    }).toThrow('Missing required env: VTEX_ACCOUNT');
  });

  it('should use default values when optional env is missing', () => {
    process.env.VTEX_ACCOUNT = 'test';
    process.env.VTEXAPPKEY = 'key';
    process.env.VTEXTOKEN = 'token';
    
    const { config } = require('./config');
    
    expect(config.vtexBaseDomain).toBe('vtexcommercestable.com.br');
    expect(config.vtexConcurrency).toBe(5);
    expect(config.ingestDays).toBe(30);
  });
});
```

#### Exemplo: vtex.test.ts

```typescript
import { describe, it, expect, vi } from 'vitest';
import { vtexUrl, vtexFetch } from './vtex';

describe('VTEX Client', () => {
  describe('vtexUrl', () => {
    it('should build correct URL', () => {
      const url = vtexUrl({
        path: '/api/oms/pvt/orders',
        query: { page: 1, per_page: 50 }
      });
      
      expect(url).toContain('vtexcommercestable.com.br');
      expect(url).toContain('page=1');
      expect(url).toContain('per_page=50');
    });
  });

  describe('vtexFetch', () => {
    it('should fetch data with correct headers', async () => {
      global.fetch = vi.fn(() =>
        Promise.resolve({
          ok: true,
          json: () => Promise.resolve({ data: 'test' }),
        })
      ) as any;

      const data = await vtexFetch({ path: '/test' });

      expect(fetch).toHaveBeenCalledWith(
        expect.any(String),
        expect.objectContaining({
          headers: expect.objectContaining({
            'X-VTEX-API-AppKey': expect.any(String),
            'X-VTEX-API-AppToken': expect.any(String),
          })
        })
      );
    });

    it('should throw error on failed request', async () => {
      global.fetch = vi.fn(() =>
        Promise.resolve({
          ok: false,
          status: 404,
          statusText: 'Not Found',
          text: () => Promise.resolve('Resource not found'),
        })
      ) as any;

      await expect(vtexFetch({ path: '/test' }))
        .rejects
        .toThrow('VTEX 404 Not Found');
    });
  });
});
```

### Testes de Integração

#### Exemplo: api.test.ts

```typescript
import { describe, it, expect, beforeAll, afterAll } from '@jest/globals';
import Fastify from 'fastify';
import { app } from '../src/index';

describe('API Integration Tests', () => {
  let server: any;

  beforeAll(async () => {
    server = Fastify();
    // Register routes from app
    await server.ready();
  });

  afterAll(async () => {
    await server.close();
  });

  describe('GET /health', () => {
    it('should return ok status', async () => {
      const response = await server.inject({
        method: 'GET',
        url: '/health'
      });

      expect(response.statusCode).toBe(200);
      expect(JSON.parse(response.payload)).toEqual({ status: 'ok' });
    });
  });

  describe('GET /metrics/summary', () => {
    it('should return summary metrics', async () => {
      const response = await server.inject({
        method: 'GET',
        url: '/metrics/summary?start=2024-01-01&end=2024-01-31'
      });

      expect(response.statusCode).toBe(200);
      const data = JSON.parse(response.payload);
      
      expect(data).toHaveProperty('orders');
      expect(data).toHaveProperty('customers');
      expect(data).toHaveProperty('totalRevenue');
      expect(data).toHaveProperty('avgOrderValue');
    });

    it('should apply status filter', async () => {
      const response = await server.inject({
        method: 'GET',
        url: '/metrics/summary?status=invoiced'
      });

      expect(response.statusCode).toBe(200);
    });
  });
});
```

### Testes de Banco de Dados

#### Setup com Test Container

```typescript
import { PostgreSqlContainer } from '@testcontainers/postgresql';
import { PrismaClient } from '@prisma/client';

let container: PostgreSqlContainer;
let prisma: PrismaClient;

beforeAll(async () => {
  container = await new PostgreSqlContainer()
    .withDatabase('test')
    .withUsername('test')
    .withPassword('test')
    .start();

  const connectionString = container.getConnectionUri();
  
  prisma = new PrismaClient({
    datasources: {
      db: {
        url: connectionString
      }
    }
  });

  // Run migrations
  await prisma.$executeRawUnsafe('CREATE EXTENSION IF NOT EXISTS "uuid-ossp"');
  // ... run migrations
});

afterAll(async () => {
  await prisma.$disconnect();
  await container.stop();
});

it('should create order', async () => {
  const order = await prisma.order.create({
    data: {
      vtexOrderId: 'test-001',
      totalValue: 10000,
    }
  });

  expect(order.vtexOrderId).toBe('test-001');
  expect(order.totalValue).toBe(10000);
});
```

### Mocks

#### Mock de Cliente VTEX

```typescript
vi.mock('./vtex', () => ({
  vtexFetch: vi.fn(() => Promise.resolve({
    list: [
      { orderId: '001', status: 'invoiced' },
      { orderId: '002', status: 'shipped' },
    ],
    paging: { total: 2, pages: 1 }
  }))
}));
```

#### Mock de Prisma

```typescript
vi.mock('./db', () => ({
  prisma: {
    order: {
      findMany: vi.fn(() => Promise.resolve([])),
      create: vi.fn((data) => Promise.resolve({ id: '1', ...data })),
    }
  }
}));
```

### Cobertura de Testes

```bash
# Executar testes com cobertura
npm test -- --coverage

# Ver relatório HTML
open coverage/index.html
```

**Meta de cobertura**: 80%+

### Scripts package.json

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:integration": "jest --testPathPattern=integration",
    "test:e2e": "jest --testPathPattern=e2e"
  }
}
```

## Frontend - Testes

### Ferramentas Sugeridas

- **Framework**: Vitest
- **Testing Library**: @testing-library/react
- **E2E**: Playwright ou Cypress
- **Visual Regression**: Percy ou Chromatic

### Instalação

```bash
cd delupo/delupo-frontend
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom jsdom
```

### Configuração (vitest.config.js)

```javascript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    globals: true,
  },
});
```

### Setup (src/test/setup.ts)

```typescript
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

afterEach(() => {
  cleanup();
});
```

### Estrutura de Testes

```
delupo-frontend/
├── src/
│   ├── components/
│   │   ├── KpiCard.jsx
│   │   └── KpiCard.test.jsx
│   ├── lib/
│   │   ├── format.js
│   │   └── format.test.js
│   └── test/
│       ├── setup.ts
│       └── utils.tsx
└── e2e/
    └── dashboard.spec.ts
```

### Testes de Componentes

#### Exemplo: KpiCard.test.jsx

```javascript
import { render, screen } from '@testing-library/react';
import { describe, it, expect } from 'vitest';
import KpiCard from './KpiCard';
import { formatCurrency } from '../lib/format';

describe('KpiCard', () => {
  it('renders title and value', () => {
    render(
      <KpiCard 
        title="Total Revenue"
        value={123456}
        format={formatCurrency}
      />
    );

    expect(screen.getByText('Total Revenue')).toBeInTheDocument();
    expect(screen.getByText('R$ 1.234,56')).toBeInTheDocument();
  });

  it('renders subtitle when provided', () => {
    render(
      <KpiCard 
        title="Orders"
        value={100}
        subtitle="Last 30 days"
      />
    );

    expect(screen.getByText('Last 30 days')).toBeInTheDocument();
  });

  it('applies custom className', () => {
    const { container } = render(
      <KpiCard 
        title="Test"
        value={100}
        className="custom-class"
      />
    );

    expect(container.firstChild).toHaveClass('custom-class');
  });
});
```

#### Exemplo: DataTable.test.jsx

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import DataTable from './DataTable';

describe('DataTable', () => {
  const columns = [
    { header: 'Name', accessor: 'name' },
    { header: 'Price', accessor: 'price', formatter: (v) => `$${v}` },
  ];

  const data = [
    { name: 'Product A', price: 100 },
    { name: 'Product B', price: 200 },
  ];

  it('renders table headers', () => {
    render(<DataTable columns={columns} data={data} />);

    expect(screen.getByText('Name')).toBeInTheDocument();
    expect(screen.getByText('Price')).toBeInTheDocument();
  });

  it('renders table data', () => {
    render(<DataTable columns={columns} data={data} />);

    expect(screen.getByText('Product A')).toBeInTheDocument();
    expect(screen.getByText('$100')).toBeInTheDocument();
  });

  it('calls onRowClick when row is clicked', () => {
    const handleRowClick = vi.fn();
    render(
      <DataTable 
        columns={columns} 
        data={data} 
        onRowClick={handleRowClick}
      />
    );

    fireEvent.click(screen.getByText('Product A'));
    expect(handleRowClick).toHaveBeenCalledWith(data[0]);
  });
});
```

### Testes de Utilidades

#### Exemplo: format.test.js

```javascript
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatNumber, formatPercent } from './format';

describe('Format Utils', () => {
  describe('formatCurrency', () => {
    it('formats cents to currency', () => {
      expect(formatCurrency(10000)).toBe('R$ 100,00');
      expect(formatCurrency(123456)).toBe('R$ 1.234,56');
      expect(formatCurrency(0)).toBe('R$ 0,00');
    });
  });

  describe('formatNumber', () => {
    it('formats numbers with thousands separator', () => {
      expect(formatNumber(1000)).toBe('1.000');
      expect(formatNumber(1234567)).toBe('1.234.567');
      expect(formatNumber(0)).toBe('0');
    });
  });

  describe('formatPercent', () => {
    it('formats decimal to percentage', () => {
      expect(formatPercent(0.1234)).toBe('12,34%');
      expect(formatPercent(1)).toBe('100,00%');
      expect(formatPercent(0)).toBe('0,00%');
    });
  });
});
```

### Testes de Hooks

#### Exemplo: hooks.test.js

```javascript
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { useSummary } from './hooks';
import * as api from './api';

// Mock API
vi.mock('./api');

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
    },
  });
  
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

describe('useSummary', () => {
  it('fetches summary data', async () => {
    const mockData = {
      orders: 100,
      customers: 50,
      totalRevenue: 1000000,
    };

    api.getSummary.mockResolvedValue(mockData);

    const { result } = renderHook(
      () => useSummary({ start: '2024-01-01', end: '2024-01-31' }),
      { wrapper: createWrapper() }
    );

    await waitFor(() => expect(result.current.isSuccess).toBe(true));

    expect(result.current.data).toEqual(mockData);
  });
});
```

### Testes E2E com Playwright

#### Instalação

```bash
npm install --save-dev @playwright/test
npx playwright install
```

#### Configuração (playwright.config.ts)

```typescript
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  use: {
    baseURL: 'http://localhost:5173',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  webServer: {
    command: 'npm run dev',
    port: 5173,
    reuseExistingServer: !process.env.CI,
  },
});
```

#### Exemplo: dashboard.spec.ts

```typescript
import { test, expect } from '@playwright/test';

test.describe('Dashboard', () => {
  test('should display summary metrics', async ({ page }) => {
    await page.goto('/');

    // Wait for data to load
    await page.waitForSelector('[data-testid="orders-kpi"]');

    // Check KPIs are visible
    await expect(page.locator('[data-testid="orders-kpi"]')).toBeVisible();
    await expect(page.locator('[data-testid="revenue-kpi"]')).toBeVisible();
    await expect(page.locator('[data-testid="customers-kpi"]')).toBeVisible();
  });

  test('should filter by date range', async ({ page }) => {
    await page.goto('/');

    // Change date filters
    await page.fill('[name="start"]', '2024-01-01');
    await page.fill('[name="end"]', '2024-01-31');
    await page.click('[data-testid="apply-filters"]');

    // Wait for updated data
    await page.waitForLoadState('networkidle');

    // Verify data updated
    await expect(page.locator('[data-testid="date-range"]'))
      .toContainText('01/01/2024 - 31/01/2024');
  });

  test('should display chart', async ({ page }) => {
    await page.goto('/');

    // Wait for chart to render
    await page.waitForSelector('.recharts-wrapper');

    // Verify chart is visible
    await expect(page.locator('.recharts-wrapper')).toBeVisible();
  });
});
```

### Scripts package.json

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

## Continuous Integration

### GitHub Actions (.github/workflows/test.yml)

```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: test
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          
      - name: Install dependencies
        working-directory: ./delupo/delupo-backend
        run: npm ci
        
      - name: Run tests
        working-directory: ./delupo/delupo-backend
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
          
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          directory: ./delupo/delupo-backend/coverage

  frontend-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          
      - name: Install dependencies
        working-directory: ./delupo/delupo-frontend
        run: npm ci
        
      - name: Run tests
        working-directory: ./delupo/delupo-frontend
        run: npm test -- --coverage
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          directory: ./delupo/delupo-frontend/coverage

  e2e-tests:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          
      - name: Install Playwright
        working-directory: ./delupo/delupo-frontend
        run: |
          npm ci
          npx playwright install --with-deps
          
      - name: Run E2E tests
        working-directory: ./delupo/delupo-frontend
        run: npm run test:e2e
        
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: ./delupo/delupo-frontend/playwright-report
```

## Boas Práticas

### 1. Nomenclatura

```javascript
// ✅ Bom
describe('OrderProcessor', () => {
  describe('processOrder', () => {
    it('should create order when data is valid', () => {});
    it('should throw error when vtexOrderId is missing', () => {});
  });
});

// ❌ Evitar
describe('test', () => {
  it('works', () => {});
});
```

### 2. Arrange-Act-Assert

```javascript
it('should calculate total', () => {
  // Arrange
  const items = [
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 },
  ];

  // Act
  const total = calculateTotal(items);

  // Assert
  expect(total).toBe(250);
});
```

### 3. Testes Independentes

```javascript
// ✅ Cada teste limpa seu estado
beforeEach(() => {
  // Reset state
});

// ❌ Testes dependem um do outro
let sharedState;
it('test 1', () => {
  sharedState = 'value';
});
it('test 2', () => {
  expect(sharedState).toBe('value'); // Dependente!
});
```

### 4. Mocking Mínimo

```javascript
// ✅ Mock apenas dependências externas
vi.mock('./vtex');

// ❌ Mock excessivo
vi.mock('./utils');
vi.mock('./helpers');
vi.mock('./formatters');
```

## Referências

- [Vitest](https://vitest.dev/)
- [Jest](https://jestjs.io/)
- [Testing Library](https://testing-library.com/)
- [Playwright](https://playwright.dev/)
- [Test Containers](https://testcontainers.com/)

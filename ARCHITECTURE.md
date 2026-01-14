# Arquitetura do Sistema

## Visão Geral da Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                          VTEX Platform                          │
│                    (E-commerce Source Data)                     │
└────────────────────────┬────────────────────────────────────────┘
                         │ REST API
                         │ (Orders, Products, Customers)
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    DELUPO BACKEND (Node.js)                     │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Ingest Service │  │ API Service  │  │  Queue Processor │  │
│  │                 │  │              │  │                  │  │
│  │  - queue-orders │  │  - Fastify   │  │  - process-queue │  │
│  │  - ingest       │  │  - Metrics   │  │  - order-proc    │  │
│  │  - sync-master  │  │  - CORS      │  │  - retry logic   │  │
│  └────────┬────────┘  └──────┬───────┘  └────────┬─────────┘  │
│           │                  │                    │             │
│           └──────────────────┼────────────────────┘             │
│                              │                                  │
│                    ┌─────────▼─────────┐                        │
│                    │   Prisma ORM      │                        │
│                    └─────────┬─────────┘                        │
└──────────────────────────────┼──────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PostgreSQL Database                          │
│                                                                 │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐  │
│  │  Customer  │  │  Order   │  │ Product  │  │ OrderQueue  │  │
│  │  Address   │  │  Items   │  │   Sku    │  │ IngestionJob│  │
│  │            │  │ Payments │  │          │  │             │  │
│  │            │  │ Shipping │  │          │  │             │  │
│  └────────────┘  └──────────┘  └──────────┘  └─────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │ REST API (JSON)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                  DELUPO FRONTEND (React)                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                      App.jsx                             │  │
│  │                  (Main Application)                      │  │
│  └────────┬──────────────────────────┬──────────────────────┘  │
│           │                          │                          │
│  ┌────────▼─────────┐       ┌────────▼─────────┐              │
│  │   Components     │       │   Custom Hooks    │              │
│  │                  │       │                   │              │
│  │  - KpiCard       │       │  - useSummary     │              │
│  │  - ChartCard     │       │  - useOrders      │              │
│  │  - DataTable     │       │  - useProducts    │              │
│  │  - FiltersBar    │       │  - useCustomers   │              │
│  │  - Card          │       │  - useUtm         │              │
│  └──────────────────┘       └────────┬──────────┘              │
│                                      │                          │
│                             ┌────────▼──────────┐               │
│                             │  TanStack Query   │               │
│                             │  (Data Fetching)  │               │
│                             └────────┬──────────┘               │
│                                      │                          │
│                             ┌────────▼──────────┐               │
│                             │   API Client      │               │
│                             │   (lib/api.js)    │               │
│                             └───────────────────┘               │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Visualization (Recharts)                                │  │
│  │  - AreaChart, BarChart, PieChart, LineChart             │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Styling (Tailwind CSS)                                  │  │
│  │  - Responsive Design, Components, Utilities             │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Fluxo de Dados

### 1. Ingestão de Dados (VTEX → Database)

```
┌─────────┐
│  VTEX   │
└────┬────┘
     │
     │ 1. queue-orders.ts
     │    Lista pedidos e adiciona à fila
     ▼
┌─────────────┐
│ OrderQueue  │
│  (pending)  │
└────┬────────┘
     │
     │ 2. process-order-queue.ts
     │    Processa pedidos da fila
     ▼
┌──────────────────┐
│ order-processor  │
│  - Fetch details │
│  - Normalize     │
│  - Upsert        │
└────┬─────────────┘
     │
     │ 3. Salva no banco
     ▼
┌──────────────────┐
│   PostgreSQL     │
│  - Order         │
│  - OrderItem     │
│  - Customer      │
│  - etc.          │
└──────────────────┘
```

### 2. API de Métricas (Database → API → Frontend)

```
┌──────────────────┐
│   PostgreSQL     │
└────┬─────────────┘
     │
     │ 1. Prisma queries (SQL agregado)
     ▼
┌──────────────────┐
│   Fastify API    │
│  /metrics/...    │
└────┬─────────────┘
     │
     │ 2. JSON response
     ▼
┌──────────────────┐
│  TanStack Query  │
│  (Cache/Fetch)   │
└────┬─────────────┘
     │
     │ 3. React state
     ▼
┌──────────────────┐
│   Components     │
│  - Charts        │
│  - Tables        │
│  - KPIs          │
└──────────────────┘
```

## Componentes Principais

### Backend

#### 1. API Service (index.ts)
- **Responsabilidade**: Expor endpoints REST
- **Tecnologia**: Fastify
- **Endpoints**: 11 endpoints de métricas
- **Features**:
  - CORS configurado
  - Query builders para filtros
  - Agregações SQL otimizadas
  - Timezone handling

#### 2. Ingest Service (ingest.ts)
- **Responsabilidade**: Importar pedidos da VTEX
- **Modo**:
  - Normal: Últimos N dias
  - Rescan: Desde primeiro pedido
  - Backfill: Períodos específicos
- **Features**:
  - Controle de concorrência (p-limit)
  - Paginação automática
  - Skip de duplicados

#### 3. Queue Service (queue-orders.ts + process-order-queue.ts)
- **Responsabilidade**: Processamento assíncrono
- **Fluxo**:
  1. Queue: Adiciona pedidos à fila
  2. Process: Processa fila com retry
- **Features**:
  - Retry logic (3 tentativas)
  - Error tracking
  - Status management

#### 4. Order Processor (order-processor.ts)
- **Responsabilidade**: Processar pedido individual
- **Operações**:
  - Fetch de detalhes VTEX
  - Normalização de dados
  - Upsert de entidades relacionadas
- **Entidades processadas**:
  - Customer
  - Order
  - OrderItems
  - Payments
  - Shipping
  - Promotions
  - Addresses

### Frontend

#### 1. App Component (App.jsx)
- **Responsabilidade**: Container principal
- **Estado**:
  - Filtros por dashboard
  - Dashboard ativo
- **Dashboards**:
  - Orders
  - Clients
  - Products
  - UTM

#### 2. Custom Hooks (lib/hooks.js)
- **Responsabilidade**: Data fetching
- **Tecnologia**: TanStack Query
- **Features**:
  - Cache automático (5min)
  - Retry automático (3x)
  - Loading/Error states
  - Revalidação em background

#### 3. API Client (lib/api.js)
- **Responsabilidade**: Comunicação com backend
- **Funções**: 11 funções (uma por endpoint)
- **Features**:
  - URL building
  - Query params
  - Error handling

#### 4. Components
- **KpiCard**: Exibir métricas únicas
- **ChartCard**: Wrapper para gráficos
- **DataTable**: Tabelas com sorting
- **FiltersBar**: Filtros de período/status
- **SectionHeader**: Cabeçalhos de seção

## Padrões de Arquitetura

### Backend

#### Separation of Concerns
```
config.ts       → Configuração
db.ts           → Database connection
vtex.ts         → VTEX API client
index.ts        → API endpoints
ingest.ts       → Data ingestion
order-processor → Business logic
```

#### Repository Pattern (via Prisma)
```typescript
// Abstração de acesso a dados
prisma.order.create(...)
prisma.customer.findUnique(...)
prisma.$queryRaw`...`
```

#### Service Layer
```typescript
// Lógica de negócio isolada
async function upsertOrder(data) {
  // Validação
  // Transformação
  // Persistência
}
```

### Frontend

#### Component Composition
```jsx
<App>
  <FiltersBar />
  <SectionHeader />
  <KpiCard />
  <ChartCard>
    <AreaChart />
  </ChartCard>
  <DataTable />
</App>
```

#### Custom Hooks Pattern
```javascript
// Encapsular lógica de data fetching
const { data, isLoading, error } = useSummary(filters);
```

#### Presentational vs Container
```
Container: App.jsx (lógica + estado)
Presentational: Components (apenas UI)
```

## Segurança

### Backend
- ✅ Variáveis de ambiente para secrets
- ✅ CORS configurado
- ✅ Prisma (SQL injection protection)
- ⚠️ Sem autenticação (a implementar)
- ⚠️ Sem rate limiting (a implementar)

### Frontend
- ✅ Ambiente variables (VITE_*)
- ✅ CSP headers (via Nginx)
- ⚠️ Sem autenticação (a implementar)

## Performance

### Backend
- ✅ Queries SQL agregadas (vs N+1)
- ✅ Índices de banco
- ✅ Paginação de resultados
- ✅ Concorrência controlada (ingest)
- ⚠️ Sem cache (Redis a implementar)

### Frontend
- ✅ TanStack Query (cache)
- ✅ Code splitting (Vite)
- ✅ Lazy loading
- ✅ Debounce de filtros
- ✅ Memoização de cálculos

## Escalabilidade

### Horizontal Scaling
```
Load Balancer
    │
    ├─── Backend Instance 1
    ├─── Backend Instance 2
    └─── Backend Instance 3
         │
    PostgreSQL (Single)
```

### Vertical Scaling
- Aumentar recursos do servidor
- Aumentar pool de conexões PostgreSQL
- Otimizar queries

### Cache Layer (Futuro)
```
Frontend → API → Redis → PostgreSQL
```

## Monitoramento

### Logs
- Backend: Fastify logger
- Nginx: Access + Error logs
- Database: PostgreSQL logs

### Métricas (a implementar)
- Request rate
- Response time
- Error rate
- CPU/Memory usage
- Database connections

### Alertas (a implementar)
- API down
- High error rate
- Slow queries
- Disk space low

## Dependências Críticas

### Backend
- **Fastify**: Web framework
- **Prisma**: ORM e migrations
- **p-limit**: Controle de concorrência

### Frontend
- **React**: UI framework
- **Vite**: Build tool
- **TanStack Query**: Data fetching
- **Recharts**: Visualização
- **Tailwind**: Estilização

### Infrastructure
- **PostgreSQL**: Database
- **Docker**: Containerização
- **Nginx**: Reverse proxy (produção)

## Decisões Arquiteturais

### Por que Fastify?
- Performance superior ao Express
- Schema validation built-in
- Plugin architecture
- TypeScript support nativo

### Por que Prisma?
- Type-safe queries
- Migrations automáticas
- Schema como fonte de verdade
- Developer experience

### Por que TanStack Query?
- Cache automático
- Background refetching
- Deduplicação de requests
- Developer tools excelentes

### Por que Recharts?
- Componentes React nativos
- Customizável
- Responsivo
- Bem documentado

### Por que Tailwind?
- Produtividade
- Consistência de design
- Tree-shaking (CSS pequeno)
- Responsive built-in

## Limitações Conhecidas

1. **Sem autenticação**: API aberta (desenvolvimento)
2. **Sem rate limiting**: Vulnerável a abuse
3. **Sem cache**: Queries repetidas batem no banco
4. **Single database**: Ponto único de falha
5. **Sem testes**: Cobertura zero
6. **Sem logging estruturado**: Logs básicos apenas
7. **Sem monitoramento**: Sem métricas de produção

## Roadmap de Melhorias

### Curto Prazo
- [ ] Implementar autenticação (JWT)
- [ ] Adicionar rate limiting
- [ ] Implementar cache (Redis)
- [ ] Adicionar testes unitários

### Médio Prazo
- [ ] Implementar logs estruturados
- [ ] Adicionar monitoramento (Prometheus)
- [ ] Implementar CI/CD completo
- [ ] Adicionar documentação OpenAPI

### Longo Prazo
- [ ] Microservices architecture
- [ ] Message queue (RabbitMQ/SQS)
- [ ] Read replicas para database
- [ ] CDN para frontend
- [ ] Multi-tenancy

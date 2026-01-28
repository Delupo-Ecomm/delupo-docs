# Delupo Dashboard - Documentação Completa

## Índice

1. [Visão Geral](#visão-geral)
2. [Arquitetura](#arquitetura)
3. [Backend](./backend/README.md)
4. [Frontend](./frontend/README.md)
5. [Banco de Dados](./database/README.md)
6. [Comunicação API](./api/README.md)
7. [Testes](./testing/README.md)
8. [Deployment](./deployment/README.md)

## Visão Geral

O **Delupo Dashboard** é uma aplicação de analytics de e-commerce que consome dados da plataforma VTEX para fornecer insights detalhados sobre vendas, clientes, produtos e performance de campanhas.

### Objetivos

- Análise de receita, pedidos e ticket médio
- Performance por canal de vendas
- Análise de produtos e SKUs (top sellers, margem, descontos)
- Análise de clientes (novos vs recorrentes, LTV, cohort)
- Análise de campanhas (UTM source/medium/campaign)
- Logística (prazo, custo, transportadoras, SLA)
- Pagamentos (meio de pagamento, parcelas, status)

### Tecnologias Principais

**Backend:**
- Node.js 20+
- TypeScript
- Fastify (servidor HTTP)
- Prisma (ORM)
- PostgreSQL 15

**Frontend:**
- React 18
- Vite
- Tailwind CSS
- Recharts (gráficos)
- TanStack Query (data fetching)

## Arquitetura

```
┌─────────────┐
│    VTEX     │ ← Fonte de dados (e-commerce)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Backend   │ ← Ingestão + API REST
│  (Node.js)  │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  PostgreSQL │ ← Armazenamento
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Frontend   │ ← Dashboard Analytics
│   (React)   │
└─────────────┘
```

### Fluxo de Dados

1. **Ingestão**: Scripts backend consomem a API VTEX e armazenam dados no PostgreSQL
2. **API**: Backend expõe endpoints REST com métricas agregadas
3. **Visualização**: Frontend consome a API e exibe dashboards interativos

## Estrutura do Projeto

```
delupo-dash/
├── docker-compose.yml          # Configuração do banco de dados
├── docs/                       # Documentação (esta pasta)
│   ├── README.md
│   ├── backend/
│   ├── frontend/
│   ├── database/
│   ├── api/
│   ├── testing/
│   └── deployment/
└── delupo/
    ├── delupo-backend/        # API e serviços backend
    └── delupo-frontend/       # Interface web
```

## Quick Start

### 1. Pré-requisitos

- Node.js 20+
- Docker & Docker Compose
- npm ou yarn

### 2. Iniciar o Banco de Dados

```bash
docker compose up -d
```

### 3. Configurar Backend

```bash
cd delupo/delupo-backend
npm install
npm run db:generate
npm run db:migrate
```

Crie um arquivo `.env`:

```env
DATABASE_URL="postgresql://delupo:delupo@localhost:5433/delupo"
VTEX_ACCOUNT=sua-conta
VTEXAPPKEY=sua-app-key
VTEXTOKEN=seu-token
VTEX_BASE_DOMAIN=vtexcommercestable.com.br
SALES_CHANNEL=1
REPORT_TIMEZONE=America/Sao_Paulo
PORT=3000
```

### 4. Iniciar Backend

```bash
npm start
```

Backend estará disponível em: `http://localhost:3000`

### 5. Configurar Frontend

```bash
cd delupo/delupo-frontend
npm install
```

Crie um arquivo `.env`:

```env
VITE_API_BASE=http://localhost:3000
```

### 6. Iniciar Frontend

```bash
npm run dev
```

Frontend estará disponível em: `http://localhost:5173`

## Componentes Principais

### Backend

- **API REST**: Endpoints de métricas agregadas
- **Ingestão VTEX**: Scripts de importação de pedidos
- **Processamento**: Queue de pedidos e sincronização

### Frontend

- **Dashboards**: Pedidos, Clientes, Produtos, UTM
- **Filtros**: Período, status, canal de vendas
- **Gráficos**: Séries temporais, tabelas comparativas
- **Exportação**: CSV de dados brutos

## Variáveis de Ambiente

### Backend (.env)

| Variável | Descrição | Padrão |
|----------|-----------|--------|
| DATABASE_URL | String de conexão PostgreSQL | - |
| VTEX_ACCOUNT | Nome da conta VTEX | - |
| VTEXAPPKEY | App Key da VTEX | - |
| VTEXTOKEN | App Token da VTEX | - |
| VTEX_BASE_DOMAIN | Domínio base VTEX | vtexcommercestable.com.br |
| SALES_CHANNEL | Canal de vendas principal | 1 |
| REPORT_TIMEZONE | Timezone dos relatórios | America/Sao_Paulo |
| PORT | Porta do servidor | 3000 |

### Frontend (.env)

| Variável | Descrição | Padrão |
|----------|-----------|--------|
| VITE_API_BASE | URL base da API backend | http://localhost:3000 |

## Scripts Disponíveis

### Backend

```bash
npm start              # Iniciar servidor de produção
npm run build          # Compilar TypeScript
npm run db:generate    # Gerar cliente Prisma
npm run db:migrate     # Executar migrações
npm run ingest         # Importar pedidos da VTEX
npm run queue:orders   # Adicionar pedidos à fila
npm run process:orders # Processar fila de pedidos
npm run sync:masterdata # Sincronizar dados mestres
```

### Frontend

```bash
npm run dev      # Servidor de desenvolvimento
npm run build    # Build de produção
npm run preview  # Preview do build
npm run lint     # Verificar código
```

## Funcionalidades

### ✅ Implementadas

- Dashboard de pedidos com série temporal
- **Agrupamento temporal configurável** (dia, semana, mês)
- **Filtro por UTM source** com interface clicável
- Métricas de receita e ticket médio
- Top produtos por receita/quantidade
- Top clientes
- Análise de UTM campaigns
- Análise de frete e transportadoras
- Análise de meios de pagamento
- Análise de cohort e retenção
- Filtros por período e status
- Gráficos interativos

### 🚧 Roadmap

- Exportação de dados para CSV/Excel
- Alertas e notificações
- Dashboards customizáveis
- Comparação de períodos (MoM, YoY)
- Análise de margem e lucratividade
- Integração com outras plataformas
- Autenticação e multi-tenancy

## Convenções de Código

### Backend

- TypeScript strict mode
- ESM modules (type: "module")
- Prisma para queries
- Valores monetários em centavos (inteiros)
- Datas em UTC, conversão para timezone do relatório

### Frontend

- Componentes funcionais React
- Hooks para data fetching
- Tailwind para estilização
- Formatação de valores via helpers
- Estado global com React Context (quando necessário)

## Performance

### Backend

- Queries otimizadas com índices
- Agregações no banco de dados
- Caching de resultados (quando aplicável)
- Paginação de resultados grandes

### Frontend

- Code splitting
- Lazy loading de componentes
- Debounce em filtros
- React Query para cache automático

## Segurança

- Variáveis de ambiente para credenciais
- CORS configurado para domínios específicos
- Validação de inputs
- Prepared statements (Prisma)
- Rate limiting (recomendado)

## Suporte

Para questões e suporte:
- Documentação: `/docs`
- Issues: GitHub Issues
- Email: suporte@exemplo.com

## Licença

Proprietary - Todos os direitos reservados

# Índice da Documentação - Delupo Dashboard

## 📚 Documentação Completa

Bem-vindo à documentação completa do Delupo Dashboard. Esta documentação foi organizada para facilitar o entendimento do projeto, desde conceitos básicos até deployment em produção.

## 🚀 Começar Rápido

**Novo no projeto?** Comece aqui:

1. **[Guia de Início Rápido](./QUICK_START.md)** ⭐
   - Setup em 10 minutos
   - Comandos básicos
   - Verificação de instalação
   - Solução de problemas comuns

2. **[README Principal](./README.md)**
   - Visão geral do projeto
   - Tecnologias utilizadas
   - Funcionalidades
   - Variáveis de ambiente

## 🏗️ Arquitetura

**Entenda como o sistema funciona:**

3. **[Arquitetura do Sistema](./ARCHITECTURE.md)**
   - Diagramas de arquitetura
   - Fluxo de dados
   - Componentes principais
   - Padrões de design
   - Decisões arquiteturais
   - Limitações e roadmap

## 💻 Backend

**Documentação do servidor Node.js:**

4. **[Backend - Documentação](./backend/README.md)**
   - Estrutura de diretórios
   - Módulos principais
   - API REST (index.ts)
   - Ingestão de dados (ingest.ts)
   - Queue de processamento
   - Configuração e variáveis
   - Boas práticas

## 🎨 Frontend

**Documentação da interface React:**

5. **[Frontend - Documentação](./frontend/README.md)**
   - Componentes
   - Custom hooks
   - API client
   - Formatação de dados
   - Gráficos com Recharts
   - Estilização com Tailwind
   - Performance
   - Build e deploy

## 🗄️ Banco de Dados

**Tudo sobre o PostgreSQL:**

6. **[Banco de Dados - Documentação](./database/README.md)**
   - Schema completo
   - Modelos do Prisma
   - Relacionamentos
   - Migrações
   - Queries comuns
   - Índices e performance
   - Backup e restore
   - Manutenção

## 🔌 API

**Referência completa da API REST:**

7. **[API - Documentação](./api/README.md)**
   - Todos os endpoints
   - Parâmetros e filtros
   - Formatos de resposta
   - Códigos de status
   - Exemplos de uso
   - Valores monetários
   - Status VTEX

## 🧪 Testes

**Estratégias de testes (futuro):**

8. **[Testes - Documentação](./testing/README.md)**
   - Pirâmide de testes
   - Testes unitários
   - Testes de integração
   - Testes E2E
   - Coverage
   - CI/CD
   - Boas práticas

## 🚢 Deploy

**Colocar em produção:**

9. **[Deployment - Documentação](./deployment/README.md)**
   - Ambientes
   - Build de produção
   - Configuração Nginx
   - PM2
   - Docker
   - Kubernetes
   - CI/CD
   - Monitoramento
   - Backup
   - Troubleshooting

## 📖 Guias por Tarefa

### Desenvolvimento

| Tarefa | Documento |
|--------|-----------|
| Configurar ambiente local | [Quick Start](./QUICK_START.md) |
| Entender a arquitetura | [Architecture](./ARCHITECTURE.md) |
| Adicionar novo endpoint | [Backend](./backend/README.md), [API](./api/README.md) |
| Criar novo componente | [Frontend](./frontend/README.md) |
| Modificar banco de dados | [Database](./database/README.md) |
| Adicionar testes | [Testing](./testing/README.md) |

### Operações

| Tarefa | Documento |
|--------|-----------|
| Deploy em produção | [Deployment](./deployment/README.md) |
| Configurar monitoramento | [Deployment - Monitoring](./deployment/README.md#monitoring) |
| Fazer backup do banco | [Database - Backup](./database/README.md#backup-e-restore) |
| Troubleshooting | [Deployment - Troubleshooting](./deployment/README.md#troubleshooting) |
| Rollback de deploy | [Deployment - Rollback](./deployment/README.md#rollback) |

### Análise de Dados

| Tarefa | Documento |
|--------|-----------|
| Consultar API | [API](./api/README.md) |
| Queries SQL diretas | [Database - Queries](./database/README.md#queries-comuns) |
| Importar dados VTEX | [Backend - Ingest](./backend/README.md#5-ingestts) |
| Entender métricas | [API - Endpoints](./api/README.md#endpoints) |

## 🔍 Busca Rápida

### Variáveis de Ambiente

- Backend: [Backend - Configuração](./backend/README.md#configuração)
- Frontend: [Frontend - Configuração](./frontend/README.md#configuração)
- Produção: [Deployment - Backend](./deployment/README.md#variáveis-de-ambiente-envproduction)

### Comandos

- Desenvolvimento: [Quick Start](./QUICK_START.md#comandos-úteis)
- Produção: [Deployment](./deployment/README.md)
- Docker: [Quick Start - Docker](./QUICK_START.md#docker)

### Endpoints da API

| Endpoint | Descrição | Documento |
|----------|-----------|-----------|
| GET /health | Health check | [API](./api/README.md#health-check) |
| GET /metrics/summary | Resumo geral | [API](./api/README.md#resumo-de-métricas) |
| GET /metrics/orders | Série temporal | [API](./api/README.md#série-temporal-de-pedidos) |
| GET /metrics/products | Top produtos | [API](./api/README.md#top-produtos) |
| GET /metrics/customers | Top clientes | [API](./api/README.md#top-clientes) |
| GET /metrics/utm | Performance UTM | [API](./api/README.md#performance-utm) |
| GET /metrics/shipping | Análise de frete | [API](./api/README.md#análise-de-frete) |
| GET /metrics/payments | Meios de pagamento | [API](./api/README.md#análise-de-pagamentos) |
| GET /metrics/retention | Retenção | [API](./api/README.md#análise-de-retenção) |
| GET /metrics/cohort | Cohort | [API](./api/README.md#análise-de-cohort) |
| GET /metrics/new-vs-returning | Novos vs Recorrentes | [API](./api/README.md#novos-vs-recorrentes) |

### Modelos do Banco

| Modelo | Descrição | Documento |
|--------|-----------|-----------|
| Customer | Clientes | [Database - Customer](./database/README.md#customer-clientes) |
| Order | Pedidos | [Database - Order](./database/README.md#order-pedidos) |
| OrderItem | Itens do pedido | [Database - OrderItem](./database/README.md#orderitem-itens-do-pedido) |
| Product | Produtos | [Database - Product](./database/README.md#product-produtos) |
| Sku | SKUs | [Database - Sku](./database/README.md#sku-skus) |
| OrderPayment | Pagamentos | [Database - OrderPayment](./database/README.md#orderpayment-pagamentos) |
| OrderShipping | Entregas | [Database - OrderShipping](./database/README.md#ordershipping-entregas) |
| OrderQueue | Fila de pedidos | [Database - OrderQueue](./database/README.md#orderqueue-fila-de-processamento) |

### Componentes Frontend

| Componente | Descrição | Documento |
|------------|-----------|-----------|
| App.jsx | Container principal | [Frontend - App](./frontend/README.md#1-appjsx) |
| KpiCard | Card de KPI | [Frontend - KpiCard](./frontend/README.md#5-kpicardjsx) |
| ChartCard | Card de gráfico | [Frontend - ChartCard](./frontend/README.md#3-chartcardjsx) |
| DataTable | Tabela de dados | [Frontend - DataTable](./frontend/README.md#5-datatablejsx) |
| FiltersBar | Barra de filtros | [Frontend - FiltersBar](./frontend/README.md#6-filtersbarjsx) |

## 📊 Status do Projeto

### ✅ Implementado

- [x] Backend API REST
- [x] Ingestão de dados VTEX
- [x] Dashboard frontend
- [x] Métricas de vendas
- [x] Análise de clientes
- [x] Análise de produtos
- [x] Análise UTM
- [x] Docker Compose
- [x] Documentação completa

### 🚧 Em Progresso

- [ ] Testes unitários
- [ ] Testes E2E
- [ ] CI/CD pipeline

### 📅 Roadmap

- [ ] Autenticação
- [ ] Rate limiting
- [ ] Cache (Redis)
- [ ] Logs estruturados
- [ ] Monitoramento
- [ ] Exportação CSV
- [ ] Multi-tenancy

## 🆘 Suporte

### Problemas Comuns

1. **"Cannot connect to database"**
   - Solução: [Quick Start - Troubleshooting](./QUICK_START.md#solução-de-problemas)

2. **"Port already in use"**
   - Solução: [Quick Start - Troubleshooting](./QUICK_START.md#erro-port-already-in-use)

3. **"Missing required env"**
   - Solução: [Quick Start - Troubleshooting](./QUICK_START.md#erro-missing-required-env-vtex_account)

4. **Aplicação não inicia**
   - Solução: [Deployment - Troubleshooting](./deployment/README.md#aplicação-não-inicia)

### Recursos Adicionais

- 📖 Documentação Prisma: https://www.prisma.io/docs
- 📖 Documentação Fastify: https://www.fastify.io/
- 📖 Documentação React: https://react.dev/
- 📖 Documentação Tailwind: https://tailwindcss.com/
- 📖 Documentação Recharts: https://recharts.org/

## 📝 Convenções

### Commits

```
feat: adiciona novo endpoint de métricas
fix: corrige cálculo de ticket médio
docs: atualiza documentação da API
refactor: melhora estrutura de componentes
test: adiciona testes para order-processor
```

### Branches

```
main          → Produção
develop       → Desenvolvimento
feature/*     → Novas funcionalidades
bugfix/*      → Correções
hotfix/*      → Correções urgentes
docs/*        → Documentação
```

## 📄 Licença

Proprietary - Todos os direitos reservados

---

**Última atualização**: 13 de janeiro de 2026

**Versão da documentação**: 1.0.0

**Projeto**: Delupo Dashboard

**Equipe**: Inovaki

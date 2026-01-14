# Backend - Documentação

## Visão Geral

O backend do Delupo Dashboard é uma aplicação Node.js/TypeScript que serve dois propósitos principais:

1. **API REST**: Expõe endpoints com métricas agregadas para o frontend
2. **Ingestão de Dados**: Importa e processa pedidos da plataforma VTEX

## Tecnologias

- **Runtime**: Node.js 20+
- **Linguagem**: TypeScript 5.5+
- **Framework Web**: Fastify 4.x
- **ORM**: Prisma 5.x
- **Banco de Dados**: PostgreSQL 15+
- **Gerenciamento de Concorrência**: p-limit

## Estrutura de Diretórios

```
delupo-backend/
├── prisma/
│   ├── schema.prisma           # Schema do banco de dados
│   └── migrations/             # Migrações SQL
├── src/
│   ├── config.ts              # Configurações e variáveis de ambiente
│   ├── db.ts                  # Cliente Prisma
│   ├── index.ts               # Servidor API principal
│   ├── vtex.ts                # Cliente HTTP para API VTEX
│   ├── ingest.ts              # Script de ingestão de pedidos
│   ├── order-processor.ts     # Processador de pedidos individuais
│   ├── queue-orders.ts        # Adiciona pedidos à fila
│   ├── process-order-queue.ts # Processa fila de pedidos
│   ├── sync-masterdata.ts     # Sincronização de dados mestres
│   └── export-masked-emails.ts # Exportação de emails mascarados
├── package.json
├── tsconfig.json
└── .env                       # Variáveis de ambiente (não versionado)
```

## Configuração

### Variáveis de Ambiente (.env)

```env
# Banco de Dados
DATABASE_URL="postgresql://delupo:delupo@localhost:5433/delupo"

# VTEX API
VTEX_ACCOUNT=nome-da-conta
VTEXAPPKEY=vtexappkey-conta-XXXX
VTEXTOKEN=token-secreto-XXXX
VTEX_BASE_DOMAIN=vtexcommercestable.com.br

# Configurações de Ingestão
VTEX_CONCURRENCY=5           # Requisições paralelas à VTEX
VTEX_PER_PAGE=50             # Itens por página
INGEST_DAYS=30               # Dias a importar por padrão

# Configurações de Relatórios
SALES_CHANNEL=1              # Canal de vendas principal
REPORT_TIMEZONE=America/Sao_Paulo

# Servidor
PORT=3000
```

### Instalação

```bash
npm install
```

### Gerar Cliente Prisma

```bash
npm run db:generate
```

### Executar Migrações

```bash
npm run db:migrate
```

## Módulos Principais

### 1. config.ts

Gerencia variáveis de ambiente e configurações da aplicação.

```typescript
export const config = {
  vtexAccount: requireEnv("VTEX_ACCOUNT"),
  vtexAppKey: requireEnv("VTEXAPPKEY"),
  vtexToken: requireEnv("VTEXTOKEN"),
  vtexBaseDomain: process.env.VTEX_BASE_DOMAIN || "vtexcommercestable.com.br",
  vtexConcurrency: Number(process.env.VTEX_CONCURRENCY || "5"),
  vtexPerPage: Number(process.env.VTEX_PER_PAGE || "50"),
  ingestDays: Number(process.env.INGEST_DAYS || "30"),
};
```

### 2. db.ts

Inicializa e exporta o cliente Prisma.

```typescript
import { PrismaClient } from "@prisma/client";

export const prisma = new PrismaClient();
```

### 3. vtex.ts

Cliente HTTP para comunicação com a API VTEX.

**Funções principais:**
- `vtexUrl()`: Constrói URLs da API VTEX
- `vtexFetch()`: Executa requisições autenticadas

**Headers de autenticação:**
```typescript
{
  "X-VTEX-API-AppKey": config.vtexAppKey,
  "X-VTEX-API-AppToken": config.vtexToken,
}
```

### 4. index.ts

Servidor API Fastify com endpoints de métricas.

**Endpoints disponíveis:**

| Endpoint | Descrição |
|----------|-----------|
| `GET /health` | Health check |
| `GET /metrics/summary` | Resumo geral (pedidos, receita, clientes) |
| `GET /metrics/orders` | Série temporal de pedidos |
| `GET /metrics/products` | Top produtos por receita/quantidade |
| `GET /metrics/customers` | Top clientes e cohorts |
| `GET /metrics/retention` | Análise de retenção mensal |
| `GET /metrics/cohort` | Análise de cohort |
| `GET /metrics/new-vs-returning` | Clientes novos vs recorrentes |
| `GET /metrics/utm` | Performance de campanhas UTM |
| `GET /metrics/shipping` | Análise de frete e transportadoras |
| `GET /metrics/payments` | Análise de meios de pagamento |

**Parâmetros de query comuns:**
- `start`: Data inicial (YYYY-MM-DD)
- `end`: Data final (YYYY-MM-DD)
- `status`: Lista de status separados por vírgula
- `limit`: Limite de resultados (padrão 20, máximo 100)
- `sort`: Ordenação (quantity ou revenue)

**Funcionalidades:**
- Timezone automático para relatórios (REPORT_TIMEZONE)
- Filtro por canal de vendas (SALES_CHANNEL)
- Valores monetários em centavos
- Queries SQL otimizadas via Prisma

### 5. ingest.ts

Script de importação de pedidos da VTEX.

**Uso:**

```bash
# Importar últimos 30 dias
npm run ingest

# Importar últimos 60 dias
npm run ingest -- --days=60

# Rescan completo desde o pedido mais antigo
npm run ingest -- --rescan

# Backfill de períodos específicos
npm run ingest -- --backfill --days=90
```

**Fluxo:**
1. Lista pedidos da API VTEX (paginação)
2. Para cada pedido, busca detalhes completos
3. Processa e normaliza dados
4. Insere/atualiza no banco via `upsertOrder()`

**Características:**
- Controle de concorrência (p-limit)
- Skip de pedidos existentes (modo rescan/backfill)
- Tratamento de erros com retry
- Registro de progresso

### 6. order-processor.ts

Processa detalhes de um pedido individual.

**Funções principais:**
- `fetchOrderDetail()`: Busca detalhes do pedido na VTEX
- `upsertOrder()`: Insere ou atualiza pedido no banco
- `upsertCustomer()`: Processa dados do cliente
- `upsertAddress()`: Processa endereços
- `upsertOrderItems()`: Processa itens do pedido
- `upsertOrderPayments()`: Processa pagamentos
- `upsertOrderShipping()`: Processa informações de frete
- `upsertOrderPromotions()`: Processa promoções aplicadas

**Normalização de dados:**
- Extração de UTM parameters
- Conversão de valores monetários
- Parsing de datas
- Limpeza de campos nulos/vazios

### 7. queue-orders.ts

Adiciona pedidos à tabela OrderQueue para processamento posterior.

**Uso:**

```bash
npm run queue:orders
```

**Fluxo:**
1. Lista pedidos da VTEX
2. Insere IDs na tabela `OrderQueue` com status "pending"
3. Permite processamento assíncrono via `process-order-queue.ts`

### 8. process-order-queue.ts

Processa pedidos da fila (OrderQueue).

**Uso:**

```bash
npm run process:orders
```

**Fluxo:**
1. Busca pedidos com `processingStatus = "pending"`
2. Processa cada pedido via `order-processor.ts`
3. Atualiza status para "completed" ou "failed"
4. Incrementa contador de tentativas
5. Registra erros para debugging

**Controle de retry:**
- Máximo de 3 tentativas por pedido
- Erros são armazenados em `lastError`

### 9. sync-masterdata.ts

Sincroniza dados mestres (produtos, SKUs) da VTEX.

**Uso:**

```bash
npm run sync:masterdata
```

**Dados sincronizados:**
- Produtos (nome, marca, categoria)
- SKUs (nome, EAN, dimensões, imagens)
- Informações de estoque e disponibilidade

### 10. export-masked-emails.ts

Exporta lista de emails mascarados (LGPD compliance).

**Uso:**

```bash
npm run export:masked-emails
```

## API REST - Detalhes

### Timezone e Datas

Todas as queries consideram o timezone configurado em `REPORT_TIMEZONE` (padrão: America/Sao_Paulo).

```sql
(o."creationDate" AT TIME ZONE 'UTC' AT TIME ZONE 'America/Sao_Paulo')::date
```

### Valores Monetários

Todos os valores são armazenados e retornados em **centavos** (padrão VTEX):
- R$ 100,00 = 10000 (centavos)
- R$ 1.234,56 = 123456 (centavos)

O frontend é responsável por formatar e exibir os valores corretamente.

### Filtros de Status

Os endpoints aceitam múltiplos status separados por vírgula:

```
?status=invoiced,payment-approved
```

Status comuns VTEX:
- `payment-pending`
- `payment-approved`
- `payment-failed`
- `invoiced`
- `canceled`
- `shipped`
- `delivered`

### Canal de Vendas

Por padrão, apenas pedidos do canal principal (SALES_CHANNEL=1) são considerados nas métricas.

Para incluir todos os canais, remova o filtro ou configure SALES_CHANNEL="".

## Performance e Otimização

### Índices do Banco

O schema Prisma já define índices essenciais:
- `vtexOrderId` (unique)
- `customerId`
- `creationDate`
- `status`

### Queries Agregadas

Todas as métricas usam agregações SQL nativas (COUNT, SUM, AVG) para performance.

### Concorrência

A ingestão usa `p-limit` para controlar requisições paralelas à VTEX:

```typescript
const limit = pLimit(config.vtexConcurrency); // padrão: 5
```

### Paginação

Endpoints com muitos resultados suportam paginação via `limit`:

```
GET /metrics/products?limit=50
```

## Logs e Debugging

O Fastify está configurado com logger padrão:

```typescript
const app = Fastify({ logger: true });
```

Logs incluem:
- Requisições HTTP (método, URL, status, tempo)
- Erros de API
- Progresso de ingestão

### Níveis de Log

Para produção, configure o nível via variável de ambiente:

```env
LOG_LEVEL=warn
```

Níveis disponíveis: `trace`, `debug`, `info`, `warn`, `error`, `fatal`

## Tratamento de Erros

### API VTEX

Erros da VTEX são capturados e re-lançados com contexto:

```typescript
if (!res.ok) {
  const body = await res.text();
  throw new Error(`VTEX ${res.status} ${res.statusText} - ${body}`);
}
```

### Prisma

Erros de banco são automaticamente tratados pelo Prisma e Fastify.

### Ingestão

Erros durante ingestão são:
1. Logados no console
2. Armazenados em `OrderQueue.lastError`
3. Incrementam contador de tentativas
4. Permitem retry manual

## Boas Práticas

### 1. Use Transações

Para operações que modificam múltiplas tabelas:

```typescript
await prisma.$transaction([
  prisma.order.create(...),
  prisma.orderItem.createMany(...),
]);
```

### 2. Sempre Valide Inputs

```typescript
const limit = Math.min(Number(request.query.limit || 20), 100);
```

### 3. Use Prepared Statements

O Prisma automaticamente usa prepared statements, prevenindo SQL injection.

### 4. Trate Valores Null

```typescript
const totalRevenue = Number(totals?.totalValue ?? 0n);
```

### 5. Formate Datas Corretamente

```typescript
function formatDateInTimezone(date: Date, timeZone: string): string {
  return new Intl.DateTimeFormat("en-CA", {
    timeZone,
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  }).format(date);
}
```

## Próximos Passos

- Adicionar autenticação (JWT)
- Implementar rate limiting
- Adicionar caching (Redis)
- Melhorar documentação OpenAPI/Swagger
- Adicionar testes unitários e de integração
- Implementar WebSockets para updates em tempo real
- Criar jobs agendados (cron) para ingestão automática

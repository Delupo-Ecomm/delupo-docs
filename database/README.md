# Banco de Dados - Documentação

## Visão Geral

O Delupo Dashboard utiliza PostgreSQL 15 como banco de dados relacional, gerenciado pelo ORM Prisma.

## Tecnologias

- **SGBD**: PostgreSQL 15
- **ORM**: Prisma 5.x
- **Migrations**: Prisma Migrate
- **Containerização**: Docker & Docker Compose

## Configuração

### Docker Compose

```yaml
services:
  db:
    image: postgres:15
    container_name: inovaDash
    environment:
      POSTGRES_DB: delupo
      POSTGRES_USER: delupo
      POSTGRES_PASSWORD: delupo
    ports:
      - "5433:5432"
    volumes:
      - delupo-stats:/var/lib/postgresql/data

volumes:
  delupo-stats:
```

### Iniciar Banco de Dados

```bash
docker compose up -d
```

### String de Conexão

```
postgresql://delupo:delupo@localhost:5433/delupo
```

## Schema do Banco de Dados

### Diagrama ER (Simplificado)

```
Customer ──┐
           │
           ├──< Order >──┬──< OrderItem >──> Product/Sku
           │             │
           │             ├──< OrderPayment
           │             │
           │             ├──< OrderShipping >──> Address
           │             │
           │             └──< OrderPromotion
           │
Address ───┘

OrderQueue (fila de processamento)
IngestionJob (controle de ingestão)
```

## Modelos do Prisma

### Customer (Clientes)

Armazena informações de clientes.

```prisma
model Customer {
  id             String   @id @default(cuid())
  vtexCustomerId String?  @unique
  email          String?  @unique
  firstName      String?
  lastName       String?
  phone          String?
  document       String?
  documentType   String?
  isCorporate    Boolean  @default(false)
  corporateName  String?
  tradeName      String?
  stateInscr     String?
  birthDate      DateTime?
  gender         String?
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  orders         Order[]
}
```

**Índices:**
- `vtexCustomerId` (unique)
- `email` (unique)

**Relações:**
- Um cliente pode ter múltiplos pedidos

### Order (Pedidos)

Armazena informações principais de pedidos.

```prisma
model Order {
  id               String   @id @default(cuid())
  vtexOrderId      String   @unique
  vtexSequence     String?
  marketplaceOrderId String?
  status           String?
  statusDescription String?
  isCompleted      Boolean @default(false)
  creationDate     DateTime?
  lastChange       DateTime?
  totalValue       Int?
  itemsValue       Int?
  shippingValue    Int?
  discountsValue   Int?
  taxValue         Int?
  roundingValue    Int?
  salesChannel     String?
  seller           String?
  affiliateId      String?
  affiliateName    String?
  origin           String?
  source           String?
  device           String?
  userAgent        String?
  utmSource        String?
  utmMedium        String?
  utmCampaign      String?
  utmTerm          String?
  utmContent       String?
  utmiCp           String?
  utmiPart         String?
  currency         String?
  locale           String?
  raw              Json?
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt

  customerId       String?
  billingAddressId String?
  shippingAddressId String?

  customer         Customer? @relation(fields: [customerId], references: [id])
  billingAddress   Address?  @relation("BillingAddress", fields: [billingAddressId], references: [id])
  shippingAddress  Address?  @relation("ShippingAddress", fields: [shippingAddressId], references: [id])

  items            OrderItem[]
  payments         OrderPayment[]
  shipments        OrderShipping[]
  promotions       OrderPromotion[]
}
```

**Campos importantes:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| vtexOrderId | String | ID único do pedido na VTEX |
| status | String | Status atual do pedido |
| creationDate | DateTime | Data de criação do pedido |
| totalValue | Int | Valor total em centavos |
| itemsValue | Int | Valor dos itens em centavos |
| shippingValue | Int | Valor do frete em centavos |
| discountsValue | Int | Descontos aplicados em centavos |
| salesChannel | String | Canal de vendas (política comercial) |
| utmSource | String | Origem da campanha (Google, Facebook, etc) |
| utmMedium | String | Meio da campanha (cpc, email, etc) |
| utmCampaign | String | Nome da campanha |

**Índices:**
- `vtexOrderId` (unique)
- `customerId`
- `creationDate`
- `status`

**Valores monetários:**
Todos armazenados em **centavos** (inteiros):
- R$ 100,00 = 10000
- R$ 1.234,56 = 123456

### OrderItem (Itens do Pedido)

Armazena itens individuais de cada pedido.

```prisma
model OrderItem {
  id            String   @id @default(cuid())
  orderId       String
  uniqueItemId  String
  productId     String?
  skuId         String?
  seller        String?
  quantity      Int
  price         Int?
  listPrice     Int?
  sellingPrice  Int?
  manualPrice   Int?
  totalPrice    Int?
  totalDiscount Int?
  tax           Int?
  measurementUnit String?
  unitMultiplier Float?
  isGift        Boolean @default(false)
  isCustomized  Boolean @default(false)
  refId         String?
  skuRefId      String?

  order         Order   @relation(fields: [orderId], references: [id])
  product       Product? @relation(fields: [productId], references: [id])
  sku           Sku?     @relation(fields: [skuId], references: [id])

  @@unique([orderId, uniqueItemId])
}
```

**Índices:**
- `(orderId, uniqueItemId)` (unique compound)

### Product (Produtos)

Dados de produtos do catálogo.

```prisma
model Product {
  id            String   @id @default(cuid())
  vtexProductId String   @unique
  name          String?
  brand         String?
  categoryId    String?
  departmentId  String?
  isActive      Boolean  @default(true)
  isVisible     Boolean  @default(true)
  releaseDate   DateTime?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  skus          Sku[]
  items         OrderItem[]
}
```

### Sku (SKUs)

Variações de produtos (tamanho, cor, etc).

```prisma
model Sku {
  id           String   @id @default(cuid())
  vtexSkuId    String   @unique
  productId    String?
  name         String?
  refId        String?
  ean          String?
  manufacturerCode String?
  height       Float?
  width        Float?
  length       Float?
  weight       Float?
  imageUrl     String?
  isActive     Boolean  @default(true)
  isAvailable  Boolean  @default(true)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  product      Product? @relation(fields: [productId], references: [id])
  items        OrderItem[]
}
```

### Address (Endereços)

Endereços de cobrança e entrega.

```prisma
model Address {
  id           String   @id @default(cuid())
  type         String?
  street       String?
  number       String?
  complement   String?
  neighborhood String?
  city         String?
  state        String?
  postalCode   String?
  country      String?
  geoLat       Float?
  geoLng       Float?
  createdAt    DateTime @default(now())

  billingOrders   Order[]         @relation("BillingAddress")
  shippingOrders  Order[]         @relation("ShippingAddress")
  orderShippings  OrderShipping[]
}
```

### OrderPayment (Pagamentos)

Informações de pagamento dos pedidos.

```prisma
model OrderPayment {
  id            String   @id @default(cuid())
  orderId       String
  transactionId String?
  paymentId     String?
  paymentSystem String?
  paymentGroup  String?
  paymentName   String?
  installments  Int?
  value         Int?
  status        String?
  authorizationId String?
  tid           String?
  nsu           String?
  gateway       String?
  cardBin       String?
  cardLast4     String?
  cardHolder    String?

  order         Order @relation(fields: [orderId], references: [id])
}
```

**Campos importantes:**

| Campo | Descrição |
|-------|-----------|
| paymentGroup | Tipo de pagamento (creditCard, debitCard, boleto, etc) |
| paymentName | Nome do meio de pagamento |
| installments | Número de parcelas |
| value | Valor do pagamento em centavos |
| status | Status da transação |

### OrderShipping (Entregas)

Informações de frete e entrega.

```prisma
model OrderShipping {
  id               String   @id @default(cuid())
  orderId          String
  addressId        String?
  deliveryChannel  String?
  shippingSla      String?
  carrier          String?
  shippingEstimate String?
  shippingEstimateDate DateTime?
  shippingValue    Int?
  deliveryWindow   Json?
  pickupPointId    String?
  pickupFriendlyName String?
  isDelivered      Boolean @default(false)
  deliveredDate    DateTime?

  order            Order   @relation(fields: [orderId], references: [id])
  address          Address? @relation(fields: [addressId], references: [id])
}
```

### OrderPromotion (Promoções)

Promoções e descontos aplicados.

```prisma
model OrderPromotion {
  id           String   @id @default(cuid())
  orderId      String
  promotionId  String?
  name         String?
  description  String?
  value        Int?
  isCumulative Boolean?
  type         String?
  raw          Json?

  order        Order @relation(fields: [orderId], references: [id])
}
```

### OrderQueue (Fila de Processamento)

Controla a fila de pedidos a processar.

```prisma
model OrderQueue {
  id               String   @id @default(cuid())
  vtexOrderId      String   @unique
  vtexSequence     String?
  status           String?
  creationDate     DateTime?
  lastChange       DateTime?
  processingStatus String   @default("pending")
  attempts         Int      @default(0)
  lastError        String?
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt
}
```

**Status de processamento:**
- `pending`: Aguardando processamento
- `processing`: Em processamento
- `completed`: Processado com sucesso
- `failed`: Falha no processamento

### IngestionJob (Jobs de Ingestão)

Controla jobs de ingestão de dados.

```prisma
model IngestionJob {
  id          String   @id @default(cuid())
  jobType     String
  status      String
  cursor      String?
  retryCount  Int      @default(0)
  startedAt   DateTime?
  finishedAt  DateTime?
  lastError   String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

## Migrações

### Listar Migrações

```bash
cd delupo/delupo-backend
npx prisma migrate status
```

### Criar Nova Migração

```bash
npx prisma migrate dev --name nome_da_migracao
```

### Aplicar Migrações

```bash
npx prisma migrate deploy
```

### Resetar Banco (desenvolvimento)

```bash
npx prisma migrate reset
```

⚠️ **ATENÇÃO**: Isso apaga todos os dados!

## Queries Comuns

### Total de Pedidos por Status

```sql
SELECT status, COUNT(*) as total
FROM "Order"
GROUP BY status
ORDER BY total DESC;
```

### Receita por Dia

```sql
SELECT 
  DATE("creationDate") as dia,
  SUM("totalValue") as receita_total,
  COUNT(*) as total_pedidos
FROM "Order"
WHERE "creationDate" >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY DATE("creationDate")
ORDER BY dia DESC;
```

### Top 10 Produtos por Receita

```sql
SELECT 
  p.name as produto,
  SUM(oi.quantity) as quantidade,
  SUM(oi."totalPrice") as receita
FROM "OrderItem" oi
JOIN "Product" p ON p.id = oi."productId"
GROUP BY p.id, p.name
ORDER BY receita DESC
LIMIT 10;
```

### Clientes com Mais Pedidos

```sql
SELECT 
  c.email,
  c."firstName",
  c."lastName",
  COUNT(o.id) as total_pedidos,
  SUM(o."totalValue") as receita_total
FROM "Customer" c
JOIN "Order" o ON o."customerId" = c.id
GROUP BY c.id, c.email, c."firstName", c."lastName"
ORDER BY total_pedidos DESC
LIMIT 20;
```

### Pedidos com Múltiplos Pagamentos

```sql
SELECT 
  o."vtexOrderId",
  o."totalValue",
  COUNT(p.id) as total_pagamentos
FROM "Order" o
JOIN "OrderPayment" p ON p."orderId" = o.id
GROUP BY o.id, o."vtexOrderId", o."totalValue"
HAVING COUNT(p.id) > 1
ORDER BY total_pagamentos DESC;
```

## Índices e Performance

### Índices Automáticos (Prisma)

O Prisma cria automaticamente índices para:
- Chaves primárias (`@id`)
- Campos únicos (`@unique`)
- Chaves estrangeiras (relations)

### Índices Adicionais Recomendados

```sql
-- Melhorar queries por data
CREATE INDEX idx_order_creation_date ON "Order"("creationDate");

-- Melhorar queries por status
CREATE INDEX idx_order_status ON "Order"("status");

-- Melhorar queries por canal
CREATE INDEX idx_order_sales_channel ON "Order"("salesChannel");

-- Melhorar queries de UTM
CREATE INDEX idx_order_utm_source ON "Order"("utmSource");
CREATE INDEX idx_order_utm_campaign ON "Order"("utmCampaign");

-- Índice composto para queries comuns
CREATE INDEX idx_order_date_status ON "Order"("creationDate", "status");
```

### Análise de Queries

```sql
-- Ver plano de execução
EXPLAIN ANALYZE
SELECT * FROM "Order"
WHERE "creationDate" >= '2024-01-01'
  AND status = 'invoiced';
```

## Backup e Restore

### Backup Completo

```bash
docker exec inovaDash pg_dump -U delupo delupo > backup.sql
```

### Backup com Compressão

```bash
docker exec inovaDash pg_dump -U delupo delupo | gzip > backup.sql.gz
```

### Restore

```bash
cat backup.sql | docker exec -i inovaDash psql -U delupo delupo
```

### Backup Apenas Schema

```bash
docker exec inovaDash pg_dump -U delupo --schema-only delupo > schema.sql
```

## Manutenção

### Vacuum

```sql
VACUUM ANALYZE;
```

### Reindexar

```sql
REINDEX DATABASE delupo;
```

### Estatísticas de Tabelas

```sql
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Monitoramento

### Conexões Ativas

```sql
SELECT 
  datname,
  count(*) as connections
FROM pg_stat_activity
GROUP BY datname;
```

### Queries Lentas

```sql
SELECT 
  query,
  calls,
  total_time,
  mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

### Cache Hit Ratio

```sql
SELECT 
  sum(heap_blks_read) as heap_read,
  sum(heap_blks_hit) as heap_hit,
  sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) as ratio
FROM pg_statio_user_tables;
```

## Segurança

### Criar Usuário Read-Only

```sql
CREATE USER relatorio WITH PASSWORD 'senha_segura';
GRANT CONNECT ON DATABASE delupo TO relatorio;
GRANT USAGE ON SCHEMA public TO relatorio;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO relatorio;
```

### Revogar Permissões

```sql
REVOKE ALL PRIVILEGES ON DATABASE delupo FROM usuario;
```

## Troubleshooting

### Conexão Recusada

```bash
# Verificar se container está rodando
docker ps | grep inovaDash

# Ver logs do container
docker logs inovaDash

# Reiniciar container
docker restart inovaDash
```

### Tabela Não Existe

```bash
# Verificar migrações
cd delupo/delupo-backend
npx prisma migrate status

# Aplicar migrações pendentes
npx prisma migrate deploy
```

### Dados Corrompidos

```bash
# Resetar banco (CUIDADO!)
npx prisma migrate reset

# Reimportar dados
npm run ingest
```

## Referências

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)

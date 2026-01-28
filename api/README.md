# API - Documentação

## Visão Geral

A API REST do Delupo Dashboard fornece endpoints para consulta de métricas agregadas de e-commerce.

## Base URL

```
http://localhost:3000
```

## Autenticação

Atualmente a API não requer autenticação (desenvolvimento).

**Roadmap**: Implementar JWT ou API Keys.

## Headers Comuns

```http
Content-Type: application/json
Accept: application/json
```

## Parâmetros de Query Comuns

| Parâmetro | Tipo | Descrição | Padrão |
|-----------|------|-----------|--------|
| start | String (YYYY-MM-DD) | Data inicial do período | Hoje - 30 dias |
| end | String (YYYY-MM-DD) | Data final do período | Hoje |
| status | String | Status de pedidos (separados por vírgula) | Todos |
| limit | Integer | Limite de resultados | 20 |
| sort | String | Ordenação (quantity ou revenue) | revenue |
| groupBy | String | Agrupamento temporal (day, week, month) | day |
| utmSource | String | Filtrar por UTM source específico | - |

## Endpoints

### Health Check

Verifica se a API está respondendo.

**Request:**
```http
GET /health
```

**Response:**
```json
{
  "status": "ok"
}
```

**Status Codes:**
- `200 OK`: API funcionando

---

### Resumo de Métricas

Retorna resumo geral de pedidos e receita.

**Request:**
```http
GET /metrics/summary?start=2024-01-01&end=2024-01-31&status=invoiced
```

**Query Parameters:**
- `start` (opcional): Data inicial
- `end` (opcional): Data final
- `status` (opcional): Lista de status separados por vírgula

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "orders": 123,
  "customers": 85,
  "totalRevenue": 3456700,
  "avgOrderValue": 28103,
  "itemsValue": 3100000,
  "shippingValue": 280000,
  "discountsValue": 120000,
  "taxValue": 0
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| start | String | Data inicial aplicada |
| end | String | Data final aplicada |
| orders | Integer | Total de pedidos |
| customers | Integer | Total de clientes únicos |
| totalRevenue | Integer | Receita total em centavos |
| avgOrderValue | Integer | Ticket médio em centavos |
| itemsValue | Integer | Valor total de itens em centavos |
| shippingValue | Integer | Valor total de frete em centavos |
| discountsValue | Integer | Valor total de descontos em centavos |
| taxValue | Integer | Valor total de impostos em centavos |

**Status Codes:**
- `200 OK`: Sucesso

---

### Série Temporal de Pedidos

Retorna série temporal de pedidos e receita com agrupamento configurável.

**Request:**
```http
GET /metrics/orders?start=2024-01-01&end=2024-01-31&groupBy=day&utmSource=google
```

**Query Parameters:**
- `start` (opcional): Data inicial
- `end` (opcional): Data final
- `status` (opcional): Lista de status separados por vírgula
- `groupBy` (opcional): Agrupamento temporal - `day` (padrão), `week` ou `month`
- `utmSource` (opcional): Filtrar por UTM source específico

**Response (groupBy=day):**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "groupBy": "day",
  "series": [
    {
      "day": "2024-01-01",
      "orders": 12,
      "revenue": 345670
    },
    {
      "day": "2024-01-02",
      "orders": 18,
      "revenue": 512340
    }
  ]
}
```

**Response (groupBy=week):**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "groupBy": "week",
  "series": [
    {
      "week": "2024-01-01",
      "orders": 85,
      "revenue": 2456700
    },
    {
      "week": "2024-01-08",
      "orders": 92,
      "revenue": 2678400
    }
  ]
}
```

**Response (groupBy=month):**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "groupBy": "month",
  "series": [
    {
      "month": "2024-01-01",
      "orders": 350,
      "revenue": 10234500
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| day/week/month | String | Data no formato YYYY-MM-DD (início do período) |
| orders | Integer | Total de pedidos no período |
| revenue | Integer | Receita do período em centavos |
| groupBy | String | Tipo de agrupamento aplicado |

**Nota sobre UTM Source:**
Quando `utmSource` é especificado, apenas pedidos com aquele UTM source serão incluídos na série temporal, permitindo análise focada de campanhas específicas.

---

### Top Produtos

Retorna lista de produtos mais vendidos.

**Request:**
```http
GET /metrics/products?start=2024-01-01&end=2024-01-31&limit=20&sort=revenue
```

**Query Parameters:**
- `limit` (opcional): Número de produtos (máximo 100)
- `sort` (opcional): `revenue` ou `quantity`

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "items": [
    {
      "skuId": "abc123",
      "productId": "prod456",
      "skuName": "Tinta Branca 18L",
      "productName": "Tinta Premium",
      "quantity": 150,
      "revenue": 4500000
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| skuId | String | ID do SKU |
| productId | String | ID do produto |
| skuName | String | Nome do SKU |
| productName | String | Nome do produto |
| quantity | Integer | Quantidade vendida |
| revenue | Integer | Receita gerada em centavos |

---

### Top Clientes

Retorna clientes com maior receita e análise de cohorts.

**Request:**
```http
GET /metrics/customers?start=2024-01-01&end=2024-01-31&limit=20
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "cohorts": {
    "newCustomers": 45,
    "returningCustomers": 78
  },
  "topCustomers": [
    {
      "customerId": "cust123",
      "email": "cliente@example.com",
      "name": "João Silva",
      "orders": 8,
      "revenue": 1234560
    }
  ]
}
```

**Campos cohorts:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| newCustomers | Integer | Clientes novos no período |
| returningCustomers | Integer | Clientes recorrentes |

**Campos topCustomers:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| customerId | String | ID do cliente |
| email | String | Email do cliente |
| name | String | Nome completo |
| orders | Integer | Total de pedidos |
| revenue | Integer | Receita gerada em centavos |

---

### Análise de Retenção

Retorna métricas de retenção mensal.

**Request:**
```http
GET /metrics/retention?start=2024-01-01&end=2024-12-31
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-12-31",
  "items": [
    {
      "periodStart": "2024-01-01",
      "customers": 100,
      "previousCustomers": 95,
      "retainedCustomers": 70
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| periodStart | String | Início do período (mês) |
| customers | Integer | Clientes ativos no período |
| previousCustomers | Integer | Clientes do período anterior |
| retainedCustomers | Integer | Clientes retidos (compraram novamente) |

---

### Análise de Cohort

Retorna análise de cohort de clientes.

**Request:**
```http
GET /metrics/cohort?start=2024-01-01&end=2024-12-31
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-12-31",
  "months": ["2024-01-01", "2024-02-01", "2024-03-01"],
  "items": [
    {
      "cohortMonth": "2024-01-01",
      "orderMonth": "2024-01-01",
      "customers": 100
    },
    {
      "cohortMonth": "2024-01-01",
      "orderMonth": "2024-02-01",
      "customers": 45
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| cohortMonth | String | Mês da primeira compra |
| orderMonth | String | Mês do pedido |
| customers | Integer | Clientes que compraram neste mês |

---

### Novos vs Recorrentes

Retorna comparação de clientes novos e recorrentes por período.

**Request:**
```http
GET /metrics/new-vs-returning?start=2024-01-01&end=2024-12-31
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-12-31",
  "items": [
    {
      "periodStart": "2024-01-01",
      "newOrders": 50,
      "returningOrders": 30
    }
  ]
}
```

---

### Performance UTM

Retorna performance de campanhas por parâmetros UTM.

**Request:**
```http
GET /metrics/utm?start=2024-01-01&end=2024-01-31&limit=200
```

**Query Parameters:**
- `all` (opcional): `true` para retornar todos os resultados (sem limite)

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "items": [
    {
      "utmSource": "google",
      "utmMedium": "cpc",
      "utmCampaign": "verao2024",
      "orders": 45,
      "revenue": 1234560
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| utmSource | String | Origem (google, facebook, email, etc) |
| utmMedium | String | Meio (cpc, organic, email, etc) |
| utmCampaign | String | Nome da campanha |
| orders | Integer | Total de pedidos |
| revenue | Integer | Receita em centavos |

**Valores especiais:**
- `(none)`: Quando não há valor UTM

---

### Análise de Frete

Retorna estatísticas de frete e transportadoras.

**Request:**
```http
GET /metrics/shipping?start=2024-01-01&end=2024-01-31&limit=20
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "items": [
    {
      "carrier": "Correios",
      "deliveryChannel": "delivery",
      "shippingSla": "PAC",
      "shipments": 150,
      "revenue": 4500000,
      "shippingValue": 450000
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| carrier | String | Nome da transportadora |
| deliveryChannel | String | Canal de entrega (delivery, pickup-in-point) |
| shippingSla | String | SLA de entrega (PAC, SEDEX, etc) |
| shipments | Integer | Total de entregas |
| revenue | Integer | Receita dos pedidos em centavos |
| shippingValue | Integer | Valor total de frete em centavos |

---

### Análise de Pagamentos

Retorna estatísticas de meios de pagamento.

**Request:**
```http
GET /metrics/payments?start=2024-01-01&end=2024-01-31&limit=20
```

**Response:**
```json
{
  "start": "2024-01-01",
  "end": "2024-01-31",
  "items": [
    {
      "paymentGroup": "creditCard",
      "paymentName": "Visa",
      "payments": 85,
      "revenue": 2567800
    }
  ]
}
```

**Campos:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| paymentGroup | String | Grupo (creditCard, debitCard, boleto, pix) |
| paymentName | String | Nome específico (Visa, Mastercard, etc) |
| payments | Integer | Total de transações |
| revenue | Integer | Receita em centavos |

---

## Status de Pedidos VTEX

| Status | Descrição |
|--------|-----------|
| payment-pending | Pagamento pendente |
| payment-approved | Pagamento aprovado |
| payment-failed | Pagamento falhou |
| invoiced | Faturado |
| canceled | Cancelado |
| ready-for-handling | Pronto para manuseio |
| handling | Em separação |
| shipped | Enviado |
| delivered | Entregue |

## Filtros de Status

Múltiplos status podem ser combinados:

```http
GET /metrics/summary?status=invoiced,payment-approved,shipped
```

## Valores Monetários

**IMPORTANTE**: Todos os valores são retornados em **centavos**.

| Valor Real | API (centavos) |
|------------|----------------|
| R$ 1,00 | 100 |
| R$ 10,00 | 1000 |
| R$ 100,00 | 10000 |
| R$ 1.234,56 | 123456 |

### Conversão

**Backend para Frontend:**
```javascript
const reais = centavos / 100;
```

**Frontend para Backend:**
```javascript
const centavos = reais * 100;
```

## Timezone

Todas as datas são processadas no timezone configurado (padrão: `America/Sao_Paulo`).

A conversão é feita automaticamente pelo backend:

```sql
(o."creationDate" AT TIME ZONE 'UTC' AT TIME ZONE 'America/Sao_Paulo')::date
```

## Canal de Vendas

Por padrão, apenas pedidos do canal principal (`SALES_CHANNEL=1`) são incluídos.

Para modificar, ajuste a variável de ambiente no backend.

## CORS

O backend está configurado para aceitar requisições de `localhost`:

```javascript
await app.register(cors, {
  origin: [/^http:\/\/localhost:\d+$/],
});
```

**Produção**: Configure origins específicos.

## Rate Limiting

⚠️ **Não implementado** - Recomendado para produção.

**Implementação sugerida:**

```javascript
import rateLimit from "@fastify/rate-limit";

await app.register(rateLimit, {
  max: 100,
  timeWindow: '1 minute'
});
```

## Tratamento de Erros

### Formato de Erro

```json
{
  "statusCode": 400,
  "error": "Bad Request",
  "message": "Invalid date format"
}
```

### Códigos HTTP

| Código | Descrição |
|--------|-----------|
| 200 | Sucesso |
| 400 | Requisição inválida |
| 404 | Recurso não encontrado |
| 500 | Erro interno do servidor |

## Paginação

Endpoints com muitos resultados suportam o parâmetro `limit`:

```http
GET /metrics/products?limit=50
```

**Limites:**
- Padrão: 20
- Máximo: 100 (maioria dos endpoints)
- Máximo: 500 (endpoint UTM)

## Exemplos de Uso

### cURL

```bash
# Resumo dos últimos 7 dias
curl "http://localhost:3000/metrics/summary?start=2024-01-07&end=2024-01-14"

# Top 10 produtos por receita
curl "http://localhost:3000/metrics/products?limit=10&sort=revenue"

# Pedidos faturados de janeiro
curl "http://localhost:3000/metrics/orders?start=2024-01-01&end=2024-01-31&status=invoiced"
```

### JavaScript (Fetch)

```javascript
const response = await fetch(
  'http://localhost:3000/metrics/summary?start=2024-01-01&end=2024-01-31'
);
const data = await response.json();
console.log(data);
```

### JavaScript (Axios)

```javascript
const { data } = await axios.get('http://localhost:3000/metrics/summary', {
  params: {
    start: '2024-01-01',
    end: '2024-01-31',
    status: 'invoiced,shipped'
  }
});
```

### Python (Requests)

```python
import requests

response = requests.get(
    'http://localhost:3000/metrics/summary',
    params={
        'start': '2024-01-01',
        'end': '2024-01-31'
    }
)
data = response.json()
```

## Performance

### Cache

⚠️ **Não implementado** - Considere adicionar Redis.

### Otimizações Aplicadas

- ✅ Queries SQL otimizadas
- ✅ Agregações no banco de dados
- ✅ Índices em campos frequentemente consultados
- ✅ Limit em resultados grandes

### Métricas de Performance

Tempo médio de resposta por endpoint:

| Endpoint | Tempo Médio |
|----------|-------------|
| /health | < 5ms |
| /metrics/summary | 50-100ms |
| /metrics/orders | 100-200ms |
| /metrics/products | 150-300ms |
| /metrics/cohort | 300-500ms |

## Documentação OpenAPI/Swagger

⚠️ **Não implementado** - Roadmap futuro.

**Implementação sugerida:**

```javascript
import swagger from '@fastify/swagger';
import swaggerUi from '@fastify/swagger-ui';

await app.register(swagger, {
  openapi: {
    info: {
      title: 'Delupo Dashboard API',
      version: '1.0.0'
    }
  }
});

await app.register(swaggerUi, {
  routePrefix: '/docs'
});
```

## Versionamento

Atualmente na versão **v1** (implícita).

**Futuro**: Adicionar versionamento explícito:

```
GET /api/v1/metrics/summary
GET /api/v2/metrics/summary
```

## Segurança

### Checklist de Produção

- [ ] Implementar autenticação (JWT/API Keys)
- [ ] Configurar CORS para domínios específicos
- [ ] Adicionar rate limiting
- [ ] Validar e sanitizar todos os inputs
- [ ] Usar HTTPS
- [ ] Implementar logs de auditoria
- [ ] Adicionar monitoring e alertas
- [ ] Configurar firewall

## Referências

- [Fastify Documentation](https://www.fastify.io/)
- [REST API Best Practices](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)

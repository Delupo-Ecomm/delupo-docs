# Frontend - Documentação

## Visão Geral

O frontend do Delupo Dashboard é uma Single Page Application (SPA) construída com React que exibe dashboards interativos de analytics de e-commerce.

## Tecnologias

- **Framework**: React 18.3+
- **Build Tool**: Vite 5.4+
- **Estilização**: Tailwind CSS 3.4+
- **Gráficos**: Recharts 2.12+
- **Data Fetching**: TanStack Query 5.52+
- **Manipulação de Datas**: date-fns 3.6+
- **Utilitários**: clsx 2.1+

## Estrutura de Diretórios

```
delupo-frontend/
├── public/                    # Arquivos estáticos
├── src/
│   ├── App.jsx               # Componente principal
│   ├── main.jsx              # Entry point
│   ├── components/           # Componentes reutilizáveis
│   │   ├── Card.jsx
│   │   ├── ChartCard.jsx
│   │   ├── DataTable.jsx
│   │   ├── FiltersBar.jsx
│   │   ├── KpiCard.jsx
│   │   └── SectionHeader.jsx
│   ├── lib/                  # Bibliotecas e utilitários
│   │   ├── api.js           # Cliente da API
│   │   ├── format.js        # Funções de formatação
│   │   └── hooks.js         # Custom hooks
│   ├── pages/               # Páginas (futuro)
│   └── styles/
│       └── tailwind.css     # Estilos globais Tailwind
├── index.html
├── package.json
├── postcss.config.js
├── tailwind.config.js
├── vite.config.js
└── .env                     # Variáveis de ambiente (não versionado)
```

## Configuração

### Variáveis de Ambiente (.env)

```env
VITE_API_BASE=http://localhost:3000
```

### Instalação

```bash
npm install
```

### Executar em Desenvolvimento

```bash
npm run dev
```

Acesse: http://localhost:5173

### Build de Produção

```bash
npm run build
```

Build gerado em: `dist/`

### Preview do Build

```bash
npm run preview
```

## Componentes

### 1. App.jsx

Componente principal da aplicação. Gerencia:
- Estado global de filtros
- Navegação entre dashboards
- Renderização de seções

**Estado principal:**

```javascript
const [activeDash, setActiveDash] = useState("orders");
const [orderFilters, setOrderFilters] = useState({
  start: format(subDays(new Date(), 30), "yyyy-MM-dd"),
  end: format(new Date(), "yyyy-MM-dd"),
  status: ""
});
```

**Dashboards disponíveis:**
- `orders`: Dashboard de pedidos
- `clients`: Dashboard de clientes
- `products`: Dashboard de produtos
- `utm`: Dashboard de campanhas UTM

### 2. Card.jsx

Componente wrapper genérico para cards.

**Props:**
- `children`: Conteúdo do card
- `className`: Classes CSS adicionais

**Uso:**

```jsx
<Card>
  <h2>Título</h2>
  <p>Conteúdo</p>
</Card>
```

### 3. ChartCard.jsx

Card especializado para exibir gráficos.

**Props:**
- `title`: Título do gráfico
- `data`: Dados do gráfico
- `children`: Componente de gráfico (Recharts)

**Uso:**

```jsx
<ChartCard title="Vendas Diárias" data={salesData}>
  <AreaChart data={salesData}>
    <Area dataKey="value" />
  </AreaChart>
</ChartCard>
```

### 4. KpiCard.jsx

Card para exibir KPIs (Key Performance Indicators).

**Props:**
- `title`: Nome do KPI
- `value`: Valor principal
- `format`: Função de formatação (currency, number, percent)
- `subtitle`: Informação adicional

**Uso:**

```jsx
<KpiCard 
  title="Receita Total"
  value={1234567}
  format={formatCurrency}
  subtitle="Últimos 30 dias"
/>
```

### 5. DataTable.jsx

Tabela de dados com suporte a ordenação.

**Props:**
- `columns`: Array de definições de colunas
- `data`: Array de dados
- `onRowClick`: Callback ao clicar em linha

**Estrutura de coluna:**

```javascript
{
  header: "Nome",
  accessor: "name",
  formatter: (value) => value,
  sortable: true
}
```

**Uso:**

```jsx
<DataTable 
  columns={columns}
  data={products}
  onRowClick={(row) => console.log(row)}
/>
```

### 6. FiltersBar.jsx

Barra de filtros com campos de data e status.

**Props:**
- `filters`: Objeto com valores atuais
- `onFiltersChange`: Callback ao alterar filtros
- `statusOptions`: Array de opções de status

**Uso:**

```jsx
<FiltersBar 
  filters={orderFilters}
  onFiltersChange={setOrderFilters}
  statusOptions={[
    { value: "invoiced", label: "Faturado" },
    { value: "canceled", label: "Cancelado" }
  ]}
/>
```

### 7. SectionHeader.jsx

Cabeçalho de seção com título e descrição.

**Props:**
- `title`: Título da seção
- `description`: Descrição opcional
- `actions`: Botões/ações adicionais

**Uso:**

```jsx
<SectionHeader 
  title="Dashboard de Pedidos"
  description="Análise completa de pedidos e receita"
  actions={<button>Exportar</button>}
/>
```

## Biblioteca de Utilitários

### lib/api.js

Cliente da API backend. Todas as funções retornam Promises.

**Funções disponíveis:**

```javascript
// Health check
fetchJson("/health")

// Métricas resumidas
getSummary({ start, end, status })

// Série temporal de pedidos
getOrders({ start, end, status })

// Top produtos
getProducts({ start, end, limit, sort, status })

// Top clientes e cohorts
getCustomers({ start, end, limit, status })

// Performance UTM
getUtm({ start, end, limit, status, all })

// Análise de frete
getShipping({ start, end, limit, status })

// Análise de pagamentos
getPayments({ start, end, limit, status })

// Retenção mensal
getRetention({ start, end, status })

// Análise de cohort
getCohort({ start, end, status })

// Novos vs recorrentes
getNewVsReturning({ start, end, status })
```

**Tratamento de erros:**

```javascript
try {
  const data = await getSummary(filters);
} catch (error) {
  console.error("API error:", error);
  // Exibir mensagem ao usuário
}
```

### lib/format.js

Funções de formatação de valores.

**Funções:**

```javascript
// Formata valores monetários
formatCurrency(12345) // "R$ 123,45"

// Formata números com separadores
formatNumber(1234567) // "1.234.567"

// Formata porcentagens
formatPercent(0.1234) // "12,34%"

// Formata datas
formatDate(new Date()) // "13/01/2026"
```

**Implementação:**

```javascript
export const formatCurrency = (cents) => {
  const reais = cents / 100;
  return new Intl.NumberFormat("pt-BR", {
    style: "currency",
    currency: "BRL",
  }).format(reais);
};

export const formatNumber = (value) => {
  return new Intl.NumberFormat("pt-BR").format(value);
};

export const formatPercent = (value) => {
  return new Intl.NumberFormat("pt-BR", {
    style: "percent",
    minimumFractionDigits: 2,
  }).format(value);
};
```

### lib/hooks.js

Custom React hooks para data fetching com TanStack Query.

**Hooks disponíveis:**

```javascript
const { data, isLoading, error } = useSummary(filters);
const { data, isLoading, error } = useOrders(filters);
const { data, isLoading, error } = useProducts(filters);
const { data, isLoading, error } = useCustomers(filters);
const { data, isLoading, error } = useUtm(filters);
const { data, isLoading, error } = useShipping(filters);
const { data, isLoading, error } = usePayments(filters);
const { data, isLoading, error } = useRetention(filters);
const { data, isLoading, error } = useCohort(filters);
const { data, isLoading, error } = useNewVsReturning(filters);
```

**Configuração TanStack Query:**

```javascript
const { data, isLoading, error } = useQuery({
  queryKey: ["summary", filters],
  queryFn: () => getSummary(filters),
  staleTime: 5 * 60 * 1000, // 5 minutos
  retry: 3,
});
```

**Benefícios:**
- Cache automático
- Revalidação em background
- Deduplicação de requisições
- Estados de loading e erro

## Gráficos com Recharts

### AreaChart - Vendas ao longo do tempo

```jsx
<ResponsiveContainer width="100%" height={300}>
  <AreaChart data={salesData}>
    <CartesianGrid strokeDasharray="3 3" />
    <XAxis dataKey="date" />
    <YAxis />
    <Tooltip formatter={formatCurrency} />
    <Area 
      type="monotone" 
      dataKey="value" 
      stroke="#8884d8" 
      fill="#8884d8" 
    />
  </AreaChart>
</ResponsiveContainer>
```

### BarChart - Comparação de valores

```jsx
<BarChart data={productsData}>
  <CartesianGrid strokeDasharray="3 3" />
  <XAxis dataKey="name" />
  <YAxis />
  <Tooltip />
  <Bar dataKey="revenue" fill="#82ca9d" />
  <Bar dataKey="quantity" fill="#8884d8" />
</BarChart>
```

### PieChart - Distribuição

```jsx
<PieChart>
  <Pie
    data={utmData}
    dataKey="value"
    nameKey="source"
    cx="50%"
    cy="50%"
    outerRadius={80}
    fill="#8884d8"
    label
  />
  <Tooltip />
</PieChart>
```

## Estilização com Tailwind

### Classes Utilitárias Comuns

```jsx
// Layout
<div className="container mx-auto px-4">
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">

// Cards
<div className="bg-white rounded-lg shadow-md p-6">

// Tipografia
<h1 className="text-3xl font-bold text-gray-900">
<p className="text-sm text-gray-600">

// Botões
<button className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded">

// Estados
<div className="hover:bg-gray-100 transition-colors">
<div className="opacity-50 cursor-not-allowed">
```

### Customização (tailwind.config.js)

```javascript
export default {
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: {
    extend: {
      colors: {
        primary: "#1e40af",
        secondary: "#6366f1",
      },
    },
  },
  plugins: [],
};
```

## Normalização de Dados

O frontend normaliza dados recebidos da API para garantir consistência.

### normalizeSeries()

Normaliza diferentes formatos de séries temporais:

```javascript
const normalizeSeries = (payload) => {
  const candidates = [
    payload.daily,
    payload.series,
    payload.data,
    // ... outros formatos
  ];
  
  const series = candidates.find((item) => Array.isArray(item)) || [];
  
  return series.map((entry) => ({
    date: entry.date || entry.day,
    value: Number(entry.revenue || entry.total || entry.value),
  }));
};
```

### detectValueFormatter()

Detecta automaticamente o tipo de valor (monetário ou numérico):

```javascript
const detectValueFormatter = (payload) => {
  const raw = payload?.series?.[0] || {};
  const keys = Object.keys(raw);
  
  const looksLikeMoney = keys.some((key) =>
    ["revenue", "amount", "total", "sales"].includes(key)
  );
  
  return {
    label: looksLikeMoney ? "Receita diária" : "Pedidos diários",
    format: looksLikeMoney ? formatCurrency : formatNumber
  };
};
```

## Performance

### Code Splitting

```javascript
// Lazy loading de componentes
const Dashboard = lazy(() => import("./pages/Dashboard"));

<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

### Memoização

```javascript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(data);
}, [data]);

const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

### Debounce de Filtros

```javascript
const [search, setSearch] = useState("");
const debouncedSearch = useDebounce(search, 500);

useEffect(() => {
  fetchData(debouncedSearch);
}, [debouncedSearch]);
```

## Acessibilidade

### Boas Práticas

```jsx
// Labels semânticos
<label htmlFor="start-date">Data Inicial</label>
<input id="start-date" type="date" />

// Alt text em imagens
<img src="chart.png" alt="Gráfico de vendas mensais" />

// Navegação por teclado
<button 
  onClick={handleClick}
  onKeyPress={(e) => e.key === 'Enter' && handleClick()}
>

// ARIA labels
<button aria-label="Fechar modal">×</button>
```

## Testes (Futuro)

### Vitest + Testing Library

```javascript
import { render, screen } from "@testing-library/react";
import KpiCard from "./KpiCard";

test("renders KPI value correctly", () => {
  render(<KpiCard title="Revenue" value={12345} format={formatCurrency} />);
  expect(screen.getByText("R$ 123,45")).toBeInTheDocument();
});
```

## Build e Deploy

### Build de Produção

```bash
npm run build
```

Gera arquivos otimizados em `dist/`:
- Minificação
- Tree shaking
- Code splitting
- Hash de arquivos para cache

### Variáveis de Ambiente de Build

```env
VITE_API_BASE=https://api.producao.com
```

### Deploy Estático

O build pode ser servido por qualquer servidor estático:
- Nginx
- Apache
- Vercel
- Netlify
- AWS S3 + CloudFront

### Configuração Nginx

```nginx
server {
  listen 80;
  server_name dashboard.exemplo.com;
  root /var/www/delupo-frontend/dist;
  
  location / {
    try_files $uri $uri/ /index.html;
  }
  
  location /api {
    proxy_pass http://localhost:3000;
  }
}
```

## Próximos Passos

- Adicionar testes unitários e E2E
- Implementar roteamento (React Router)
- Adicionar autenticação
- Implementar tema escuro
- Adicionar exportação de dados (CSV, Excel)
- Melhorar responsividade mobile
- Adicionar mais gráficos e visualizações
- Implementar dashboards customizáveis
- Adicionar notificações push
- Internacionalização (i18n)

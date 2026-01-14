# Guia de Início Rápido

Este guia irá ajudá-lo a colocar o projeto Delupo Dashboard funcionando em minutos.

## Pré-requisitos

- ✅ Node.js 20 ou superior
- ✅ Docker e Docker Compose
- ✅ npm (vem com Node.js)

## Passo a Passo

### 1. Iniciar o Banco de Dados

```bash
# Na raiz do projeto
docker compose up -d
```

Aguarde alguns segundos até o PostgreSQL iniciar completamente.

### 2. Configurar Backend

```bash
cd delupo/delupo-backend

# Instalar dependências
npm install

# Gerar cliente Prisma
npm run db:generate

# Executar migrações
npm run db:migrate
```

### 3. Configurar Variáveis de Ambiente do Backend

Crie um arquivo `.env` em `delupo/delupo-backend/`:

```env
DATABASE_URL="postgresql://delupo:delupo@localhost:5433/delupo"
VTEX_ACCOUNT=sua-conta-vtex
VTEXAPPKEY=sua-app-key
VTEXTOKEN=seu-token
VTEX_BASE_DOMAIN=vtexcommercestable.com.br
SALES_CHANNEL=1
REPORT_TIMEZONE=America/Sao_Paulo
PORT=3000
```

> **Nota**: Peça as credenciais VTEX ao administrador do sistema.

### 4. Iniciar Backend

```bash
# Ainda em delupo/delupo-backend
npm start
```

✅ Backend rodando em: **http://localhost:3000**

### 5. Configurar Frontend

Abra um **novo terminal** e execute:

```bash
cd delupo/delupo-frontend

# Instalar dependências
npm install
```

### 6. Configurar Variáveis de Ambiente do Frontend

Crie um arquivo `.env` em `delupo/delupo-frontend/`:

```env
VITE_API_BASE=http://localhost:3000
```

### 7. Iniciar Frontend

```bash
# Ainda em delupo/delupo-frontend
npm run dev
```

✅ Frontend rodando em: **http://localhost:5173**

## Verificar Instalação

### 1. Testar API

```bash
curl http://localhost:3000/health
```

Deve retornar: `{"status":"ok"}`

### 2. Acessar Dashboard

Abra o navegador em: **http://localhost:5173**

Você deve ver o dashboard (pode estar vazio se não houver dados ainda).

## Importar Dados (Opcional)

Para importar pedidos da VTEX:

```bash
cd delupo/delupo-backend

# Importar últimos 30 dias
npm run ingest
```

Aguarde o processo finalizar. Pode levar alguns minutos dependendo do volume de pedidos.

## Comandos Úteis

### Backend

```bash
cd delupo/delupo-backend

npm start                  # Iniciar servidor
npm run db:migrate         # Executar migrações
npm run db:generate        # Gerar cliente Prisma
npm run ingest             # Importar pedidos da VTEX
```

### Frontend

```bash
cd delupo/delupo-frontend

npm run dev                # Servidor de desenvolvimento
npm run build              # Build de produção
npm run preview            # Preview do build
```

### Docker

```bash
# Na raiz do projeto

docker compose up -d       # Iniciar banco
docker compose down        # Parar banco
docker compose logs -f     # Ver logs
docker ps                  # Ver containers rodando
```

## Estrutura de URLs

| Serviço | URL |
|---------|-----|
| Frontend | http://localhost:5173 |
| Backend API | http://localhost:3000 |
| PostgreSQL | localhost:5433 |
| Health Check | http://localhost:3000/health |

## Solução de Problemas

### Erro: "Port already in use"

```bash
# Encontrar processo usando a porta
lsof -i :3000

# Matar processo
kill -9 <PID>
```

### Erro: "Cannot connect to database"

```bash
# Verificar se Docker está rodando
docker ps

# Reiniciar banco
docker compose restart
```

### Erro: "Missing required env: VTEX_ACCOUNT"

Certifique-se de criar o arquivo `.env` no backend com todas as variáveis necessárias.

### Frontend carregando sem dados

1. Verifique se o backend está rodando: `http://localhost:3000/health`
2. Verifique se há dados no banco
3. Execute o ingest: `npm run ingest`

## Próximos Passos

- 📖 Leia a [Documentação Completa](./README.md)
- 🔌 Explore os [Endpoints da API](./api/README.md)
- 🗃️ Entenda o [Schema do Banco](./database/README.md)
- 🚀 Veja o [Guia de Deploy](./deployment/README.md)

## Ajuda

Se encontrar problemas:

1. Verifique os logs do backend e frontend
2. Consulte a documentação específica na pasta `/docs`
3. Verifique as issues no repositório
4. Entre em contato com a equipe

---

**Tempo estimado de setup**: 10-15 minutos

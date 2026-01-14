# Deployment - Documentação

## Visão Geral

Guia para deployment do Delupo Dashboard em diferentes ambientes.

## Ambientes

### Desenvolvimento
- Backend: `http://localhost:3000`
- Frontend: `http://localhost:5173`
- Banco: `localhost:5433`

### Staging (Homologação)
- Backend: `https://api-staging.delupo.com`
- Frontend: `https://staging.delupo.com`
- Banco: RDS/Cloud SQL

### Produção
- Backend: `https://api.delupo.com`
- Frontend: `https://dashboard.delupo.com`
- Banco: RDS/Cloud SQL

## Pré-requisitos de Produção

### Infraestrutura
- [ ] Servidor Linux (Ubuntu 22.04 LTS recomendado)
- [ ] Node.js 20+ instalado
- [ ] PostgreSQL 15+ (RDS, Cloud SQL, ou próprio)
- [ ] Nginx ou outro reverse proxy
- [ ] SSL/TLS certificates (Let's Encrypt)
- [ ] Domain names configurados

### Segurança
- [ ] Firewall configurado
- [ ] Acesso SSH restrito
- [ ] Secrets management (AWS Secrets Manager, Vault)
- [ ] Backup automático do banco
- [ ] Monitoring e alertas

## Backend - Deployment

### Build de Produção

```bash
cd delupo/delupo-backend

# Instalar dependências
npm ci --production=false

# Compilar TypeScript
npm run build

# Gerar cliente Prisma
npm run db:generate
```

### Variáveis de Ambiente (.env.production)

```env
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=postgresql://user:pass@db-host:5432/delupo

# VTEX
VTEX_ACCOUNT=conta-producao
VTEXAPPKEY=vtexappkey-conta-XXXX
VTEXTOKEN=token-secreto-XXXX
VTEX_BASE_DOMAIN=vtexcommercestable.com.br

# Settings
SALES_CHANNEL=1
REPORT_TIMEZONE=America/Sao_Paulo
LOG_LEVEL=warn

# Concurrency
VTEX_CONCURRENCY=5
VTEX_PER_PAGE=50
```

### Executar Migrações

```bash
# No servidor de produção
npm run db:migrate
```

### Iniciar com PM2

#### Instalação do PM2

```bash
npm install -g pm2
```

#### Configuração (ecosystem.config.js)

```javascript
module.exports = {
  apps: [{
    name: 'delupo-backend',
    script: 'dist/index.js',
    instances: 2,
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'production',
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
    merge_logs: true,
    autorestart: true,
    max_memory_restart: '1G',
  }]
};
```

#### Comandos PM2

```bash
# Iniciar aplicação
pm2 start ecosystem.config.js

# Ver status
pm2 status

# Ver logs
pm2 logs delupo-backend

# Monitoramento
pm2 monit

# Restart
pm2 restart delupo-backend

# Stop
pm2 stop delupo-backend

# Startup automático
pm2 startup
pm2 save
```

### Configuração Nginx

```nginx
# /etc/nginx/sites-available/delupo-backend

upstream backend {
    least_conn;
    server localhost:3000;
    server localhost:3001;
}

server {
    listen 80;
    server_name api.delupo.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.delupo.com;

    ssl_certificate /etc/letsencrypt/live/api.delupo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.delupo.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;

    access_log /var/log/nginx/delupo-backend-access.log;
    error_log /var/log/nginx/delupo-backend-error.log;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=100r/s;
    limit_req zone=api burst=20 nodelay;
}
```

### Health Check

Configure um health check para monitoring:

```bash
# Cron job para verificar saúde da API
*/5 * * * * curl -f http://localhost:3000/health || systemctl restart delupo-backend
```

## Frontend - Deployment

### Build de Produção

```bash
cd delupo/delupo-frontend

# Instalar dependências
npm ci

# Build
VITE_API_BASE=https://api.delupo.com npm run build
```

Arquivos gerados em: `dist/`

### Deploy Estático

#### Opção 1: Nginx

```nginx
# /etc/nginx/sites-available/delupo-frontend

server {
    listen 80;
    server_name dashboard.delupo.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name dashboard.delupo.com;

    ssl_certificate /etc/letsencrypt/live/dashboard.delupo.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/dashboard.delupo.com/privkey.pem;

    root /var/www/delupo-frontend/dist;
    index index.html;

    access_log /var/log/nginx/delupo-frontend-access.log;
    error_log /var/log/nginx/delupo-frontend-error.log;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript 
               application/x-javascript application/xml+rss 
               application/javascript application/json;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # SPA routing
    location / {
        try_files $uri $uri/ /index.html;
        add_header Cache-Control "no-cache";
    }

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' https://api.delupo.com; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';" always;
}
```

#### Opção 2: Vercel

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
cd delupo/delupo-frontend
vercel --prod
```

**vercel.json:**

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "env": {
    "VITE_API_BASE": "https://api.delupo.com"
  },
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Frame-Options",
          "value": "SAMEORIGIN"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        }
      ]
    }
  ]
}
```

#### Opção 3: AWS S3 + CloudFront

```bash
# Build
npm run build

# Upload to S3
aws s3 sync dist/ s3://delupo-dashboard/ --delete

# Invalidate CloudFront cache
aws cloudfront create-invalidation \
  --distribution-id DISTRIBUTION_ID \
  --paths "/*"
```

**CloudFront Settings:**
- Origin: S3 bucket
- Default Root Object: `index.html`
- Error Pages: 404 → `/index.html` (for SPA routing)
- SSL Certificate: ACM certificate
- Compress Objects: Yes

## Banco de Dados - Deployment

### PostgreSQL Managed Services

#### AWS RDS

```bash
# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier delupo-prod \
  --db-instance-class db.t3.medium \
  --engine postgres \
  --engine-version 15.4 \
  --master-username delupo \
  --master-user-password SECURE_PASSWORD \
  --allocated-storage 100 \
  --vpc-security-group-ids sg-xxx \
  --db-subnet-group-name subnet-group \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00" \
  --multi-az \
  --storage-encrypted \
  --publicly-accessible false
```

#### Google Cloud SQL

```bash
# Create Cloud SQL instance
gcloud sql instances create delupo-prod \
  --database-version=POSTGRES_15 \
  --tier=db-custom-2-7680 \
  --region=us-east1 \
  --backup-start-time=03:00 \
  --retained-backups-count=7 \
  --storage-auto-increase \
  --availability-type=regional
```

### Backup Automático

```bash
#!/bin/bash
# /opt/scripts/backup-db.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/postgresql"
RETENTION_DAYS=30

# Criar backup
pg_dump -h db-host -U delupo -d delupo | \
  gzip > $BACKUP_DIR/delupo_$DATE.sql.gz

# Remover backups antigos
find $BACKUP_DIR -name "delupo_*.sql.gz" \
  -mtime +$RETENTION_DAYS -delete

# Upload para S3
aws s3 cp $BACKUP_DIR/delupo_$DATE.sql.gz \
  s3://delupo-backups/postgresql/
```

**Cron:**

```cron
# Backup diário às 3h
0 3 * * * /opt/scripts/backup-db.sh
```

### Restore de Backup

```bash
# Download do S3
aws s3 cp s3://delupo-backups/postgresql/delupo_20240113_030000.sql.gz .

# Descompactar e restaurar
gunzip delupo_20240113_030000.sql.gz
psql -h db-host -U delupo -d delupo < delupo_20240113_030000.sql
```

## Docker - Deployment

### Dockerfile (Backend)

```dockerfile
# delupo/delupo-backend/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
COPY prisma ./prisma/

RUN npm ci

COPY . .

RUN npm run build
RUN npm run db:generate

FROM node:20-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/prisma ./prisma

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

### Dockerfile (Frontend)

```dockerfile
# delupo/delupo-frontend/Dockerfile

FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

ARG VITE_API_BASE
ENV VITE_API_BASE=$VITE_API_BASE

RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose (Produção)

```yaml
# docker-compose.prod.yml

version: '3.8'

services:
  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./delupo/delupo-backend
      dockerfile: Dockerfile
    restart: always
    depends_on:
      db:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
      VTEX_ACCOUNT: ${VTEX_ACCOUNT}
      VTEXAPPKEY: ${VTEXAPPKEY}
      VTEXTOKEN: ${VTEXTOKEN}
      NODE_ENV: production
    networks:
      - backend
      - frontend
    healthcheck:
      test: ["CMD", "node", "-e", "require('http').get('http://localhost:3000/health')"]
      interval: 30s
      timeout: 10s
      retries: 3

  frontend:
    build:
      context: ./delupo/delupo-frontend
      dockerfile: Dockerfile
      args:
        VITE_API_BASE: ${VITE_API_BASE}
    restart: always
    depends_on:
      - backend
    ports:
      - "80:80"
    networks:
      - frontend

networks:
  backend:
  frontend:

volumes:
  postgres_data:
```

### Deploy com Docker

```bash
# Build e start
docker-compose -f docker-compose.prod.yml up -d

# Ver logs
docker-compose -f docker-compose.prod.yml logs -f

# Stop
docker-compose -f docker-compose.prod.yml down

# Update e redeploy
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d
```

## Kubernetes - Deployment

### Deployment (Backend)

```yaml
# k8s/backend-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: delupo-backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: delupo-backend
  template:
    metadata:
      labels:
        app: delupo-backend
    spec:
      containers:
      - name: backend
        image: delupo/backend:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: delupo-secrets
              key: database-url
        - name: VTEX_ACCOUNT
          valueFrom:
            secretKeyRef:
              name: delupo-secrets
              key: vtex-account
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: delupo-backend
spec:
  selector:
    app: delupo-backend
  ports:
  - port: 3000
    targetPort: 3000
  type: ClusterIP
```

### Ingress

```yaml
# k8s/ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: delupo-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  tls:
  - hosts:
    - api.delupo.com
    - dashboard.delupo.com
    secretName: delupo-tls
  rules:
  - host: api.delupo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: delupo-backend
            port:
              number: 3000
  - host: dashboard.delupo.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: delupo-frontend
            port:
              number: 80
```

## CI/CD

### GitHub Actions

```yaml
# .github/workflows/deploy.yml

name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy-backend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Build
        working-directory: ./delupo/delupo-backend
        run: |
          npm ci
          npm run build
          npm run db:generate
      
      - name: Deploy to server
        uses: appleboy/scp-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          source: "./delupo/delupo-backend/dist,./delupo/delupo-backend/node_modules"
          target: "/var/www/delupo-backend"
      
      - name: Restart PM2
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /var/www/delupo-backend
            pm2 restart delupo-backend

  deploy-frontend:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
      
      - name: Build
        working-directory: ./delupo/delupo-frontend
        run: |
          npm ci
          VITE_API_BASE=${{ secrets.API_BASE_URL }} npm run build
      
      - name: Deploy to S3
        uses: jakejarvis/s3-sync-action@master
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: delupo-dashboard
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: ./delupo/delupo-frontend/dist
      
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

## Monitoring

### Logs

```bash
# PM2 logs
pm2 logs delupo-backend --lines 100

# Nginx logs
tail -f /var/log/nginx/delupo-backend-access.log
tail -f /var/log/nginx/delupo-backend-error.log
```

### Métricas com Prometheus

```yaml
# prometheus.yml

scrape_configs:
  - job_name: 'delupo-backend'
    static_configs:
      - targets: ['localhost:3000']
```

### Alertas

Configure alertas para:
- [ ] CPU > 80%
- [ ] Memória > 90%
- [ ] Disco > 85%
- [ ] API response time > 1s
- [ ] Error rate > 5%
- [ ] Database connections > 80%

## Rollback

### PM2

```bash
# Manter versões anteriores
cp -r dist dist.backup

# Rollback
pm2 stop delupo-backend
rm -rf dist
mv dist.backup dist
pm2 start delupo-backend
```

### Docker

```bash
# Listar imagens
docker images

# Usar imagem anterior
docker-compose up -d backend:previous-tag
```

## Checklist de Deploy

### Pré-Deploy
- [ ] Testes passando
- [ ] Build sem erros
- [ ] Variáveis de ambiente configuradas
- [ ] Migrações de banco testadas
- [ ] Backup do banco realizado

### Deploy
- [ ] Deploy do backend
- [ ] Executar migrações
- [ ] Deploy do frontend
- [ ] Verificar health checks
- [ ] Testar endpoints principais

### Pós-Deploy
- [ ] Monitorar logs por 30min
- [ ] Verificar métricas
- [ ] Testar funcionalidades críticas
- [ ] Comunicar equipe

## Troubleshooting

### Aplicação não inicia

```bash
# Verificar logs
pm2 logs delupo-backend

# Verificar processos
pm2 status

# Verificar portas
netstat -tulpn | grep 3000
```

### Banco não conecta

```bash
# Testar conexão
psql -h db-host -U delupo -d delupo

# Verificar firewall
telnet db-host 5432
```

### Alto uso de memória

```bash
# Análise com PM2
pm2 monit

# Limitar memória
pm2 start --max-memory-restart 1G
```

## Referências

- [PM2 Documentation](https://pm2.keymetrics.io/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

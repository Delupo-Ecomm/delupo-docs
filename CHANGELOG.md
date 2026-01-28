# Changelog - Delupo Dashboard

## [Não versionado] - 2026-01-27

### ✨ Novas Funcionalidades

#### Agrupamento Temporal Configurável

- **Backend**: Adicionado parâmetro `groupBy` ao endpoint `/metrics/orders`
  - Opções: `day` (padrão), `week`, `month`
  - Permite agregar dados por dia, semana ou mês
  - Resposta inclui campo `groupBy` indicando o agrupamento aplicado

- **Frontend**: Implementados controles de agrupamento temporal
  - Botões para alternar entre visualização diária, semanal e mensal
  - Título do gráfico atualiza dinamicamente (ex: "Receita diária", "Receita semanal", "Receita mensal")
  - Estado persistido em `orderFilters.groupBy`

#### Filtro por UTM Source

- **Backend**: Adicionado parâmetro `utmSource` ao endpoint `/metrics/orders`
  - Filtra pedidos por uma fonte de UTM específica
  - Permite análise focada de campanhas individuais

- **Frontend**: UTMs agora são clicáveis
  - Card "UTM sources no periodo" com badges clicáveis
  - Clicar em uma UTM filtra o gráfico para mostrar apenas dados daquela fonte
  - Indicador visual mostra UTM selecionada (cor azul destacada)
  - Botão "Limpar filtro" para remover filtro ativo
  - Filtro também disponível no FiltersBar como botões

### 🔧 Melhorias

#### FiltersBar Component

- Adicionadas novas props:
  - `showGroupBy`: Exibe controles de agrupamento temporal
  - `utmSources`: Array de UTM sources para filtro
- Layout reorganizado em seções verticais para melhor usabilidade
- Melhor feedback visual para filtros ativos

#### Normalização de Dados

- Função `normalizeSeries` atualizada para suportar campos `week` e `month`
- Função `detectValueFormatter` agora considera o `groupBy` para gerar labels apropriados

### 📝 Documentação

#### Atualizações

- **docs/README.md**: 
  - Corrigida referência incorreta a "maxitintas"
  - Adicionadas novas funcionalidades à lista de implementadas
  
- **docs/api/README.md**: 
  - Documentados novos parâmetros `groupBy` e `utmSource`
  - Exemplos de response para cada tipo de agrupamento
  - Tabela de parâmetros atualizada

- **docs/frontend/README.md**:
  - Documentação atualizada do componente FiltersBar
  - Exemplos de uso das novas props
  - Descrição das funcionalidades de filtro UTM

- **docs/CHANGELOG.md**: 
  - Criado este arquivo para rastrear mudanças

### 🎯 Como Usar

#### Agrupamento Temporal

1. No dashboard de pedidos, use os botões "Dia", "Semana" ou "Mês" para alternar a visualização
2. O gráfico será atualizado automaticamente
3. O título refletirá o período selecionado

#### Filtro por UTM

1. No card "UTM sources no periodo", clique em qualquer badge de UTM
2. O gráfico mostrará apenas dados daquela fonte
3. Para remover o filtro:
   - Clique novamente no badge selecionado
   - Use o botão "Limpar filtro"
   - Clique em "Todos" no FiltersBar

### 🔄 Compatibilidade

- Backend: Totalmente retrocompatível - parâmetros opcionais
- Frontend: Novos filtros opcionais, não afetam dashboards existentes
- API: `groupBy` padrão é "day", mantendo comportamento anterior

### 📋 Arquivos Modificados

#### Backend
- `delupo/delupo-backend/src/index.ts`

#### Frontend
- `delupo/delupo-frontend/src/App.jsx`
- `delupo/delupo-frontend/src/components/FiltersBar.jsx`

#### Documentação
- `docs/README.md`
- `docs/api/README.md`
- `docs/frontend/README.md`
- `docs/CHANGELOG.md` (novo)

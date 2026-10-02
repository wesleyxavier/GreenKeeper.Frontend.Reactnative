## ADDED Requirements

### Requirement: Banco local com Drizzle e migrations
O app SHALL usar `expo-sqlite` com Drizzle ORM e aplicar as migrations pendentes na inicialização, antes de renderizar as abas.

#### Scenario: Primeiro boot
- **WHEN** o app inicia sem banco existente
- **THEN** as migrations rodam, o schema é criado e as abas são exibidas

#### Scenario: Falha de migration
- **WHEN** uma migration falha na inicialização
- **THEN** a tela de erro padrão é exibida, o erro é logado e o app não fecha

### Requirement: Colunas base de sincronização
Toda tabela de domínio SHALL ter `id`, `version`, `syncStatus` e `deletedAt` (soft delete), definidas por um helper de schema compartilhado.

#### Scenario: Registro novo
- **WHEN** uma entidade é criada localmente
- **THEN** `syncStatus` é `PendingSync`, `version` é 1 e `deletedAt` é nulo

### Requirement: Listagens reativas por live query
Listagens SHALL ser live queries do Drizzle que reexecutam quando qualquer tabela observada muda, inclusive por escrita feita fora da tela.

#### Scenario: Escrita externa atualiza a lista
- **WHEN** um registro é inserido no banco por um serviço enquanto a lista está montada
- **THEN** a lista exibe o novo registro sem recarga manual

#### Scenario: Query com join
- **WHEN** a lista usa join entre duas tabelas e uma delas muda
- **THEN** a lista é atualizada

### Requirement: Fallback web atrás da mesma interface
Na web, o banco SHALL ser entregue pelo adapter escolhido no spike (`expo-sqlite` WASM ou IndexedDB), atrás da mesma interface de repositório, sem alterar telas ou casos de uso.

#### Scenario: Mesmo teste de contrato nos dois adapters
- **WHEN** a suíte de contrato de repositório roda contra o adapter Android e o adapter web
- **THEN** ambos passam com o mesmo comportamento

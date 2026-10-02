## ADDED Requirements

### Requirement: Store Redux Toolkit único
O app SHALL ter um único store Redux Toolkit, criado no composition root, com listener middleware habilitado e um slice de exemplo de UI/sessão.

#### Scenario: Action chega ao reducer
- **WHEN** uma action do slice de exemplo é despachada
- **THEN** o estado reflete a mudança e o selector retorna o novo valor

### Requirement: Store sem entidades de domínio
O Store SHALL guardar somente estado de UI/sessão e eventos, nunca entidades de domínio nem listas delas; o SQLite SHALL ser a fonte de dados.

#### Scenario: Teste de reflexão sobre slices
- **WHEN** um teste inspeciona o estado inicial de todos os slices
- **THEN** nenhum valor é uma entidade do `domain` nem array de entidades

### Requirement: Listener middleware para eventos
O app SHALL permitir reações a actions via listener middleware, sem acoplar telas a quem emite o evento.

#### Scenario: Evento dispara reação
- **WHEN** uma action registrada em um listener é despachada
- **THEN** o efeito do listener executa uma vez

### Requirement: TanStack Query só para chamadas remotas
TanStack Query SHALL ser usado apenas para chamadas HTTP; dados locais SHALL vir de live queries do banco.

#### Scenario: Lista local não usa Query
- **WHEN** uma tela lista dados do banco local
- **THEN** a fonte é uma live query do Drizzle, não um `useQuery` de rede

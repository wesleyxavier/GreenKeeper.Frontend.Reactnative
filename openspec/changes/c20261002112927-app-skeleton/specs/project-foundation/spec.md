## ADDED Requirements

### Requirement: Projeto Expo com TypeScript strict
O projeto SHALL ser um app Expo SDK 56 (React Native 0.85) em TypeScript com `strict: true`, com ESLint e Prettier configurados e scripts `lint`, `typecheck` e `test`.

#### Scenario: Verificações estáticas passam
- **WHEN** o dev roda `npm run lint` e `npm run typecheck`
- **THEN** ambos terminam sem erros

### Requirement: Camadas com dependência unidirecional
O código SHALL ser organizado em `domain`, `application`, `infrastructure` e `ui`, onde `ui` depende de `application`, `application` depende de `domain`, `infrastructure` implementa interfaces de `application` e `domain` não importa nenhuma outra camada nem biblioteca de plataforma.

#### Scenario: Domain não importa de outras camadas
- **WHEN** um teste de arquitetura varre os imports de `src/domain`
- **THEN** nenhum import aponta para `application`, `infrastructure`, `ui`, `react`, `react-native` ou `expo-*`

#### Scenario: UI não importa infrastructure direto
- **WHEN** um teste de arquitetura varre os imports de `src/ui`
- **THEN** nenhum import aponta para `src/infrastructure`, exceto o composition root

### Requirement: Composition root com adapters por plataforma
O app SHALL montar as dependências em um único composition root, escolhendo o adapter de cada interface de plataforma (banco, storage seguro, preferências, logger, notificações) conforme o alvo (Android ou web).

#### Scenario: Adapter Android no nativo
- **WHEN** o app inicia no Android
- **THEN** o composition root entrega os adapters nativos (SQLite, SecureStore, MMKV, logger em arquivo)

#### Scenario: Adapter web no PWA
- **WHEN** o app inicia na web
- **THEN** o composition root entrega os adapters web e recursos sem suporte ficam explicitamente desligados

### Requirement: Testes unitários executáveis
O projeto SHALL ter Jest com `jest-expo` e React Native Testing Library configurados, com ao menos um teste de domain e um de componente.

#### Scenario: Suite passa
- **WHEN** o dev roda `npm test`
- **THEN** a suíte termina verde

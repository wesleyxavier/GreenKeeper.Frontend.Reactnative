## ADDED Requirements

### Requirement: Build do APK via Docker
O projeto SHALL gerar o APK de release por `docker compose run build`, usando imagem com Node, JDK 17 e Android SDK, sem exigir SDK no host, e gravar o APK em `artifacts/`.

#### Scenario: Build limpo
- **WHEN** o dev roda `docker compose run build` em clone limpo
- **THEN** um APK é gerado em `artifacts/` e o comando termina com código 0

### Requirement: E2E Android via Docker
O projeto SHALL rodar o E2E Android por `docker compose run e2e`, subindo emulador headless, instalando o APK e executando fluxos Maestro, com screenshots e logs em `artifacts/`.

#### Scenario: Smoke test Android
- **WHEN** o E2E roda com o APK gerado
- **THEN** o fluxo abre o app, vê os 5 ícones do footer, troca entre as abas, abre e fecha o sheet de Configurações, e passa

#### Scenario: Falha
- **WHEN** um passo do fluxo falha
- **THEN** screenshot e log ficam em `artifacts/` e o comando termina com código diferente de 0

### Requirement: E2E web via Docker
O projeto SHALL rodar o E2E do PWA por `docker compose run web-e2e` com Playwright, cobrindo as abas e a instalabilidade (manifest e service worker registrados).

#### Scenario: Smoke test web
- **WHEN** o E2E web roda contra a build do PWA
- **THEN** as 4 abas são navegáveis e o sheet abre e o service worker está ativo

### Requirement: Suporte a KVM documentado
O README SHALL documentar como habilitar aceleração KVM para o emulador (WSL2 com virtualização aninhada ou runner Linux) e como proceder se não houver KVM.

#### Scenario: Ambiente sem KVM
- **WHEN** o E2E Android roda sem `/dev/kvm`
- **THEN** o serviço falha cedo com mensagem clara apontando para a seção do README

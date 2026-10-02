## ADDED Requirements

### Requirement: Arquivos de ignore
O repo SHALL ter `.gitignore` cobrindo `node_modules/`, `android/build`, `.gradle`, `.expo/`, `dist/`, `artifacts/`, `*.apk`, `.env`, `local.properties` e `worktrees/`, e `.graphifyignore` excluindo dependências, build e binários.

#### Scenario: Artefatos não versionados
- **WHEN** o dev gera um APK e roda `git status`
- **THEN** `artifacts/`, `node_modules/` e `*.apk` não aparecem como não rastreados

### Requirement: README e instruções de agentes
O repo SHALL ter `README.md` (stack, comandos Docker, E2E, instalação do PWA, estrutura de pastas, KVM) e `CLAUDE.md`, `AGENTS.md` e `GEMINI.md` no mesmo padrão do repo MAUI, apontando para o README.

#### Scenario: Comandos documentados
- **WHEN** um dev novo segue o README
- **THEN** consegue buildar o APK, rodar os testes e rodar os E2E apenas com os comandos documentados

### Requirement: Knowledge graph inicial
O repo SHALL ter `graphify-out/` gerado e atualizado com `graphify update .` antes do PR.

#### Scenario: Grafo atualizado
- **WHEN** a change vai para PR
- **THEN** `graphify update .` foi executado e o resultado está commitado

### Requirement: Branches de trabalho
O repo SHALL ter `develop` criada a partir do commit inicial, e o trabalho SHALL ocorrer em worktree curto (`worktrees/skeleton`) na branch `feature/app-skeleton`.

#### Scenario: Worktree
- **WHEN** `git worktree list` é executado
- **THEN** mostra `worktrees/skeleton` na branch `feature/app-skeleton`

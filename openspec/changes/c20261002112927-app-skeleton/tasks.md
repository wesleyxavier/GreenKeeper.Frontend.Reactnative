# Tasks

> Regras: todo comando Node/Gradle/emulador/Playwright roda via `docker compose` (regra `docker-isolation`); commit, push e PR exigem **autorização explícita do usuário** a cada vez (regra `authorization`); o PR sai por `gh` (remote GitHub), com squash em `develop`.

## 0. Bootstrap e Branch

- [ ] 0.1 **[AUTORIZAÇÃO] Commit inicial e `develop`** (repo vazio, sem `develop`)
  - **Comando:** `git add openspec .claude && git commit -m "chore: bootstrap openspec" && git push -u origin main && git branch develop && git push -u origin develop`
  - **Contexto:** design.md Decisão 19 — a regra de worktree parte de `develop`.
  - **Done:** `git branch -a` mostra `main`, `develop` e os remotos.

- [ ] 0.2 **Criar worktree `feature/app-skeleton`** (skill `openspec-worktree`)
  - **Comando:** `git worktree add worktrees/skeleton -b feature/app-skeleton develop` (path curto de propósito: aapt2/OneDrive)
  - **Done:** `git worktree list` mostra `worktrees/skeleton` na branch `feature/app-skeleton`. Todas as tasks seguintes rodam dentro desse worktree.

---

## 1. Spikes (ponto de replanejamento)

> Se um spike falhar, **parar e revisar design.md** antes de seguir.

- [ ] 1.1 **Spike (a): APK com JDK 17 no Docker**
  - **O que fazer:** projeto Expo mínimo (`create-expo-app`, SDK 56) + `Dockerfile.build` provisório (Node conforme o SDK, JDK 17, Android cmdline-tools) + `expo prebuild --platform android` + `./gradlew assembleRelease`.
  - **Contexto:** design.md Decisão 16 e Open Questions (Node, compileSdk, JDK).
  - **Done:** APK gerado; registrar em design.md as versões reais de Node, `compileSdk`, `build-tools` e se o JDK 17 bastou.

- [ ] 1.2 **Spike (b): emulador + Maestro com KVM**
  - **O que fazer:** subir emulador x86_64 headless num container (WSL2 com `/dev/kvm`), instalar o APK do spike (a) e rodar um fluxo Maestro que abre o app.
  - **Contexto:** design.md Decisões 16 e 18; risco de KVM.
  - **Done:** Maestro abre o app e tira screenshot; se não houver KVM, registrar o plano B (runner Linux) em design.md.

- [ ] 1.3 **Spike (c): banco na web**
  - **O que fazer:** `expo-sqlite` + Drizzle rodando no `expo export --platform web` (OPFS/COOP-COEP); se instável, protótipo IndexedDB com a mesma interface.
  - **Contexto:** design.md Decisão 5.
  - **Done:** decisão registrada em design.md (adapter web escolhido) com um insert + select funcionando no Chromium.

- [ ] 1.4 **Spike (d): BottomSheet e Reanimated no PWA**
  - **O que fazer:** testar `@gorhom/bottom-sheet` no Android e na web; se a web falhar, protótipo do sheet próprio (`Modal` + `Animated`).
  - **Contexto:** design.md Decisão 9.
  - **Done:** decisão registrada em design.md; sheet abre e fecha nos dois alvos.

- [ ] 1.5 **Commit + push do grupo 1** (descartar código do spike, manter só as decisões em design.md)
  - **Comando:** `git add openspec && git commit -m "docs: registra decisoes dos spikes" && git push -u origin feature/app-skeleton`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 2. Fundação do repo

- [ ] 2.1 **Criar `.gitignore`**
  - **Conteúdo:** `node_modules/`, `.expo/`, `dist/`, `android/`, `ios/`, `.gradle/`, `local.properties`, `artifacts/`, `*.apk`, `.env`, `worktrees/`, `**/graphify-out/cache`, `.DS_Store`, `Thumbs.db`.
  - **Done:** `git status` não lista esses caminhos depois de gerados.

- [ ] 2.2 **Criar `.graphifyignore`**
  - **Conteúdo:** `node_modules`, `android`, `dist`, `.expo`, `artifacts`, `*.apk`, lockfiles e assets binários.
  - **Done:** arquivo existe e `graphify update .` ignora esses caminhos.

- [ ] 2.3 **Criar `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` na raiz**
  - **O que mudar:** mesmo padrão do repo MAUI (regra absoluta, graphify, "Infos em README.md").
  - **Done:** os 3 arquivos existem e apontam para o README.

- [ ] 2.4 **Criar `README.md` inicial**
  - **Conteúdo:** visão, stack, estrutura de pastas, comandos Docker (placeholders a preencher nos grupos 12 a 14), seção KVM/WSL2.
  - **Done:** README renderiza e lista os comandos previstos.

- [ ] 2.5 **Commit + push do grupo 2**
  - **Comando:** `git add . && git commit -m "chore: arquivos de repo (ignore, docs, instrucoes de agentes)" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 3. Projeto Expo, camadas e testes

- [ ] 3.1 **Criar o app Expo (SDK 56) com TypeScript strict e Expo Router**
  - **Comando:** `docker compose run --rm node npx create-expo-app@latest . --template` (serviço `node` provisório no compose) e `tsconfig` com `"strict": true` e aliases (`@/domain`, `@/application`, `@/infrastructure`, `@/ui`, `@/composition`).
  - **Contexto:** design.md Decisões 1 e 2.
  - **Done:** `docker compose run --rm node npm run typecheck` passa.

- [ ] 3.2 **ESLint + Prettier + `eslint-plugin-boundaries`**
  - **O que mudar:** regras de fronteira das camadas (domain sem imports externos; ui sem infrastructure fora de `composition`); regra proibindo cor literal fora de `ui/theme`.
  - **Contexto:** spec project-foundation; design.md Decisão 3.
  - **Done:** `npm run lint` passa; um import proibido de teste faz o lint falhar.

- [ ] 3.3 **Estrutura de camadas e composition root**
  - **O que mudar:** pastas `src/domain`, `src/application`, `src/infrastructure`, `src/ui`, `src/composition`; interfaces `Database`, `SecureStore`, `Preferences`, `Logger`, `Notifier`, `HttpClient` em `application/ports`; `composition/root.ts` e Context provider; seleção de adapter por `*.native.ts`/`*.web.ts`.
  - **Contexto:** design.md Decisões 3 e 4.
  - **Done:** o app abre com o composition root montando fakes; teste confirma a seleção por plataforma.

- [ ] 3.4 **Jest + `jest-expo` + RNTL e teste de arquitetura**
  - **O que mudar:** `jest.config`, `setupTests`, um teste de domain, um de componente e `architecture.test.ts` varrendo imports de `domain` e `ui`.
  - **Done:** `docker compose run --rm node npm test` fica verde.

- [ ] 3.5 **Commit + push do grupo 3**
  - **Comando:** `git add . && git commit -m "feat: projeto Expo, camadas e testes" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 4. Tema, ícones e shell de abas

- [ ] 4.1 **Tokens e `ThemeProvider` em `src/ui/theme/`**
  - **O que mudar:** converter os tokens `oklch` do mockup para hex (`accent`, `purple`, `pink`, `orange`, `red`, `muted`, `border`, `bg`, `fg`) e derivar o conjunto dark; `useTheme()` por `useColorScheme()`; espaçamento e raios (16/10/999).
  - **Contexto:** design.md Decisão 7; UC-02.
  - **Done:** teste unit do UC-02 passa (light, dark e troca em runtime).

- [ ] 4.2 **Ícones Lucide**
  - **O que mudar:** adicionar `lucide-react-native` e `react-native-svg`; componente `Icon` tipado com cor do tema.
  - **Contexto:** design.md Decisão 8.
  - **Done:** os 5 ícones do footer renderizam no Android e na web.

- [ ] 4.3 **Layout de abas e `BottomNav`**
  - **O que mudar:** `app/(tabs)/_layout.tsx` com Início, Tarefas, Plantas, Alertas e botão ⚙ (despacha `openSettings`); ativo em `accent`, Plantas em `pink`; `accessibilityLabel` e `testID` em cada ícone.
  - **Contexto:** ux-design.md; spec app-shell; UC-01.
  - **Done:** testes unit do UC-01 passam.

- [ ] 4.4 **Telas placeholder (`ScreenContainer`, `ScreenHeader`, `EmptyState`)**
  - **O que mudar:** 4 telas com título e "Em breve"; barra de status roxa.
  - **Done:** as 4 abas navegáveis, com tema aplicado.

- [ ] 4.5 **Commit + push do grupo 4**
  - **Comando:** `git add . && git commit -m "feat(ui): tema, icones e shell de abas" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 5. Estado (Redux Toolkit)

- [ ] 5.1 **Store, slices `ui`/`session` e listener middleware**
  - **O que mudar:** `configureStore` no composition root; slice `ui` (`settingsOpen`, `toast`) e `session` (placeholder); `listenerMiddleware` registrado.
  - **Contexto:** design.md Decisão 6; spec ui-state-store.
  - **Done:** testes de reducer e de listener passam.

- [ ] 5.2 **Teste de reflexão "sem entidades no Store"**
  - **O que mudar:** teste que inspeciona o estado inicial de todos os slices e falha se houver tipo de `domain` ou array de entidades.
  - **Done:** teste verde; um slice com entidade faz o teste falhar (validado localmente).

- [ ] 5.3 **TanStack Query provider (só rede)**
  - **O que mudar:** `QueryClientProvider` no root; regra documentada no README (dados locais por live query).
  - **Done:** provider monta sem afetar o shell.

- [ ] 5.4 **Commit + push do grupo 5**
  - **Comando:** `git add . && git commit -m "feat(state): Redux Toolkit e TanStack Query" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 6. BottomSheet e sheet de Configurações

- [ ] 6.1 **`BottomSheet` reutilizável (implementação decidida no spike 1.4)**
  - **O que mudar:** API `open/close/onClose`, handle, backdrop, gesto de arrastar, `BackHandler` no Android; estado aberto/fechado no slice `ui`.
  - **Contexto:** design.md Decisão 9; UC-03.
  - **Done:** testes unit do UC-03 passam.

- [ ] 6.2 **Sheet de Configurações**
  - **O que mudar:** 5 `SettingsOption` desabilitadas com selo "em breve" (Perfil, Sincronização, Casas, Áreas, Sobre o App) e botão "Fechar"; abre pelo ícone ⚙.
  - **Contexto:** ux-design.md (Sheet de Configurações).
  - **Done:** o sheet abre e fecha sem trocar a rota.

- [ ] 6.3 **Commit + push do grupo 6**
  - **Comando:** `git add . && git commit -m "feat(ui): BottomSheet e sheet de configuracoes" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 7. Logger, erros e toast

- [ ] 7.1 **Logger com `ConsoleSink` e `FileSink`**
  - **O que mudar:** interface `Logger` em `application/ports`; `ConsoleSink` (todos os alvos); `FileSink` Android (`expo-file-system`, rotação por tamanho/quantidade, máscara de segredos); web só console; falha de gravação nunca propaga.
  - **Contexto:** design.md Decisão 12; spec logging-and-error-handling.
  - **Done:** testes unit do UC-06 (log, rotação, máscara) passam.

- [ ] 7.2 **`ErrorBoundary` e tela de erro padrão**
  - **O que mudar:** `ErrorBoundary` na raiz, `ErrorScreen` com "Tentar de novo" (remonta a árvore); detalhe técnico só em build de teste.
  - **Done:** teste do componente que lança exibe a tela e chama `logger.error`.

- [ ] 7.3 **`ToastProvider`, `useToast()` e `notify()` global**
  - **O que mudar:** toast no root acima do Router; erro (`red`) e info (`accent`); 1 visível, ~4s; listener Redux que converte o evento de erro em toast.
  - **Contexto:** design.md Decisão 13.
  - **Done:** testes de toast (aparece, some, substitui) passam.

- [ ] 7.4 **Commit + push do grupo 7**
  - **Comando:** `git add . && git commit -m "feat: logger, ErrorBoundary e toast" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 8. Banco de dados

- [ ] 8.1 **Drizzle + `expo-sqlite`, helper `baseColumns` e primeira migration**
  - **O que mudar:** `schema/base.ts` (`id`, `version`, `syncStatus`, `deletedAt`), tabela de exemplo `exemplo` (só para o diagnóstico), `drizzle.config`, migration gerada por `drizzle-kit`.
  - **Contexto:** design.md Decisão 5; UC-04.
  - **Done:** teste unit do UC-04 (migrate em memória, no-op na segunda execução, erro tipado) passa.

- [ ] 8.2 **Migrations no boot com tratamento de falha**
  - **O que mudar:** `useMigrations`/`migrate` antes de renderizar as abas; falha vai para o `ErrorBoundary` e o log.
  - **Done:** falha forçada exibe a tela de erro sem fechar o app.

- [ ] 8.3 **Interface `Repository` e suíte de contrato**
  - **O que mudar:** repositório genérico (insert, update, soft delete, list) + suíte de contrato reutilizável por adapter.
  - **Done:** suíte verde contra o adapter SQLite (Jest) e contra o fake.

- [ ] 8.4 **Adapter web (decidido no spike 1.3)**
  - **O que mudar:** `expo-sqlite` WASM ou IndexedDB atrás da mesma interface; `*.web.ts`.
  - **Done:** a suíte de contrato passa contra o adapter web (Jest + jsdom ou Playwright).

- [ ] 8.5 **Live query e teste de reatividade**
  - **O que mudar:** hook `useLiveList` sobre `useLiveQuery`; testes do UC-05 (escrita externa, join, soft delete).
  - **Contexto:** spec local-database (reatividade).
  - **Done:** testes unit do UC-05 passam.

- [ ] 8.6 **Commit + push do grupo 8**
  - **Comando:** `git add . && git commit -m "feat(db): Drizzle, migrations e live queries" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 9. Rede e storage

- [ ] 9.1 **Interfaces e adapters de storage seguro e preferências**
  - **O que mudar:** `SecureStore` (expo-secure-store / web) e `Preferences` (MMKV / localStorage) com fakes para Jest.
  - **Contexto:** design.md Decisão 11.
  - **Done:** teste de salvar, ler e apagar passa nos adapters e nos fakes.

- [ ] 9.2 **`DeviceIdProvider`**
  - **O que mudar:** gera UUID uma vez e persiste em `Preferences`.
  - **Done:** teste confirma o mesmo id após "reabrir" (nova instância).

- [ ] 9.3 **`HttpClient` com `X-Device-Id`, timeout e erros tipados**
  - **O que mudar:** cliente `fetch` + `AbortController`; `NetworkError`, `TimeoutError`, `AuthError`, `HttpError`; URL por `EXPO_PUBLIC_API_URL`; `.env.example`.
  - **Contexto:** design.md Decisão 10; UC-07.
  - **Done:** testes unit do UC-07 passam (header igual entre chamadas, erro de rede, timeout, 401).

- [ ] 9.4 **Commit + push do grupo 9**
  - **Comando:** `git add . && git commit -m "feat(net): cliente HTTP, storage e device id" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 10. Notificações (base)

- [ ] 10.1 **Canal Android e `Notifier`**
  - **O que mudar:** `expo-notifications` cria o canal no boot sem pedir permissão; `NoopNotifier` na web.
  - **Contexto:** design.md Decisão 14.
  - **Done:** teste do adapter confirma o canal e nenhuma chamada de permissão.

- [ ] 10.2 **Background task no-op**
  - **O que mudar:** `expo-background-task` registra uma tarefa que só loga; desligada na web.
  - **Done:** teste confirma o registro no Android e nenhum registro na web.

- [ ] 10.3 **Commit + push do grupo 10**
  - **Comando:** `git add . && git commit -m "feat(notifications): canal e background task base" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 11. Diagnóstico (build de teste)

- [ ] 11.1 **Rota `/__diagnostico` condicional a `EXPO_PUBLIC_E2E=1`**
  - **O que mudar:** inserir registro de exemplo, lista por live query, chamar `/version`, "Forçar erro"; excluída do bundle de produção.
  - **Contexto:** ux-design.md (Diagnóstico); UC-05, UC-06, UC-07.
  - **Done:** com a flag a rota abre; sem a flag a rota não existe (teste de bundle ou de roteamento).

- [ ] 11.2 **`testID` estáveis nos elementos dos E2E**
  - **Done:** lista de `testID` documentada no README e usada pelos fluxos dos grupos 14 e 15.

- [ ] 11.3 **Commit + push do grupo 11**
  - **Comando:** `git add . && git commit -m "feat: tela de diagnostico para E2E" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 12. PWA

- [ ] 12.1 **Manifest e ícones**
  - **O que mudar:** `public/manifest.webmanifest` (nome PlantumHibis, 192/512 + maskable, `display: standalone`, `start_url`, cores do tema) e `<link>` no HTML.
  - **Contexto:** design.md Decisão 15; UC-08.
  - **Done:** teste unit valida os campos do manifest.

- [ ] 12.2 **Service worker (`workbox-build`)**
  - **O que mudar:** precache do shell + fallback offline, gerado após `expo export --platform web`, registrado só em produção na web.
  - **Done:** Lighthouse/Playwright confirmam SW ativo e `installable`.

- [ ] 12.3 **Commit + push do grupo 12**
  - **Comando:** `git add . && git commit -m "feat(pwa): manifest e service worker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 13. Docker: build do APK

- [ ] 13.1 **`Dockerfile.build` e serviço `build`**
  - **O que mudar:** imagem final (versões registradas no spike 1.1), `expo prebuild` + `gradlew assembleRelease`, keystore de debug, cache do Gradle em volume, APK em `artifacts/`.
  - **Comando:** `docker compose run --rm build`
  - **Contexto:** design.md Decisão 16; UC-09.
  - **Done:** `artifacts/*.apk` existe e `aapt dump badging` confere o pacote.

- [ ] 13.2 **Script de pós-build (`scripts/verify-apk.sh`)**
  - **Done:** sem APK retorna código ≠ 0 com mensagem clara (teste do UC-09).

- [ ] 13.3 **Serviço `test` (Jest) no compose**
  - **Comando:** `docker compose run --rm test`
  - **Done:** suíte verde via Docker.

- [ ] 13.4 **Commit + push do grupo 13**
  - **Comando:** `git add . && git commit -m "feat(docker): build do APK e testes via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 14. Docker: E2E Android (Maestro)

- [ ] 14.1 **`Dockerfile.e2e` e serviço `e2e` com pré-checagem de KVM**
  - **O que mudar:** emulador x86_64 headless + Maestro; `scripts/check-kvm.sh` falha cedo apontando para a seção do README.
  - **Contexto:** design.md Decisão 16; UC-10.
  - **Done:** sem `/dev/kvm` a mensagem aparece; com KVM o emulador sobe.

- [ ] 14.2 **Fluxos Maestro (`e2e/android/*.yaml`)**
  - **O que mudar:** smoke UC-01/02/03 (abas, tema dark, sheet e botão voltar), UC-04 (primeiro boot), UC-05/06/07 pelo build de teste (`EXPO_PUBLIC_E2E=1`) com `backend-mock` no compose.
  - **Comando:** `docker compose run --rm e2e`
  - **Done:** fluxos passam; screenshots e logs em `artifacts/`; falha forçada gera código ≠ 0 com artefatos; `adb run-as` lê o log do app (UC-06).

- [ ] 14.3 **Commit + push do grupo 14**
  - **Comando:** `git add . && git commit -m "feat(e2e): Maestro no Android via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 15. Docker: E2E web (Playwright)

- [ ] 15.1 **`Dockerfile.web` e serviço `web-e2e`**
  - **O que mudar:** build estática do PWA servida localmente + imagem oficial do Playwright.
  - **Comando:** `docker compose run --rm web-e2e`
  - **Done:** specs executam no Chromium.

- [ ] 15.2 **Specs Playwright (`e2e/web/*.spec.ts`)**
  - **O que mudar:** UC-01 (abas), UC-02 (`colorScheme: 'dark'`), UC-03 (sheet), UC-04 (primeira visita), UC-08 (SW ativo, manifest válido, recarregar offline).
  - **Done:** todas verdes; relatório e traces em `artifacts/`.

- [ ] 15.3 **Commit + push do grupo 15**
  - **Comando:** `git add . && git commit -m "feat(e2e): Playwright no PWA via Docker" && git push`
  - **Done:** `git log --oneline -1` mostra o commit.

---

## 16. Pré-PR

- [ ] 16.1 **Atualizar `README.md`**
  - **O que mudar:** comandos reais (`build`, `test`, `e2e`, `web-e2e`), instalação do PWA, `testID`, seção KVM/WSL2, estrutura final de pastas, gaps conhecidos.
  - **Done:** um dev novo executa só os comandos do README e obtém APK, testes e E2E.

- [ ] 16.2 **Validação completa via Docker**
  - **Comando:** `docker compose run --rm test && docker compose run --rm build && docker compose run --rm e2e && docker compose run --rm web-e2e`
  - **Done:** os 4 serviços passam; capturar screenshots do Android (light e dark) e do PWA para o PR.

- [ ] 16.3 **`graphify update .`**
  - **Contexto:** regra `graphify-update`.
  - **Done:** `graphify-out/` atualizado.

---

## 17. Commit final e PR

- [ ] 17.1 **Commit final + push**
  - **Comando:** `git add . && git commit -m "feat: esqueleto do app React Native (Android + PWA)" && git push`
  - **Done:** `git log --oneline -1` mostra o commit; árvore limpa.

- [ ] 17.2 **[AUTORIZAÇÃO] Criar Pull Request**
  - **Comando:** `gh pr create --base develop --head feature/app-skeleton --title "feat: esqueleto do app React Native (Android + PWA)"` (descrição ASCII-safe, com screenshots e checklist de specs).
  - **Contexto:** o remote é GitHub, então não usa MCP azure-repos.
  - **Done:** PR aberto com destino `develop`; aguardar aprovação e squash merge.

- [ ] 17.3 **Pós-merge**
  - **O que fazer:** sincronizar `develop`, mover a change para `openspec/changes/archive/`, remover o worktree (`git worktree remove worktrees/skeleton` + `git branch -d feature/app-skeleton`), e propor a change 2 (auth).
  - **Done:** `git worktree list` sem o worktree; change arquivada.

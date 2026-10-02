## Context

O frontend MAUI (`GreenKeeper.Frontend`) é Clean Architecture em C#, offline-first (SQLite/EF Core), com sync para o Backend FastAPI. Esta change cria o repo novo `GreenKeeper.Frontend.Reactnative` (vazio hoje) com a **base** do app em React Native/TypeScript, **sem nenhum módulo de negócio**. Alvos: **Android** (APK) e **PWA** (Expo Web) sobre a mesma base.

Restrições: Docker obrigatório para build e E2E (regra `docker-isolation`, sem SDK no host); Windows host sem KVM nativo; paths curtos para worktrees (aapt2/OneDrive); remote GitHub (não Azure DevOps); o MAUI e o Backend permanecem intactos; o mockup visual de referência está em `GreenKeeper.Frontend/design/mockups/screens/` (tokens em `oklch`).

Insumos: `concept.md`, `proposal.md`, 10 specs, `use-cases.md` (UC-01 a UC-10) e `ux-design.md`.

## Goals / Non-Goals

**Goals:**
- Base Expo/TS strict com camadas verificáveis por teste de arquitetura.
- Shell de UI: 4 abas + sheet, tema light/dark, ícones SVG, ErrorBoundary, toast.
- Banco local reativo (Drizzle + live query) com adapter web validado em spike.
- Redux Toolkit (UI/sessão/eventos) sem entidades; TanStack Query só para rede.
- Logger em arquivo + console; base de notificações e background task.
- PWA instalável e offline.
- Build do APK, E2E Android (Maestro) e E2E web (Playwright) 100% via Docker.
- Repo pronto: ignore files, README, instruções de agentes, graphify.

**Non-Goals:**
- Qualquer módulo de negócio (auth, casas, áreas, plantas, tarefas, alertas, perfil, sync, dashboard).
- Lógica de alerta e pedido de permissão de notificação.
- Deploy/hospedagem do PWA, iOS, assinatura de release, Play Store.
- Mudanças no Backend ou no repo MAUI.
- Toggle manual de tema.

## Decisions

1. **Expo SDK 56 com prebuild (CNG) e dev client; `android/` não é versionado.** O Gradle roda sobre o `android/` gerado por `expo prebuild` dentro do Docker, a partir de `app.json` e config plugins. Alternativas: bare RN com `android/` commitado (mais código nativo para manter, drift de template); Expo Go (não suporta módulos nativos como MMKV e expo-sqlite com a config necessária). CNG mantém o repo pequeno e o build reproduzível.

2. **Expo Router (file-based) com `(tabs)`.** Rotas: `/`, `/tarefas`, `/plantas`, `/alertas`; o ícone ⚙ é um botão customizado do tab bar que despacha `openSettings` no Store em vez de navegar. Rota escondida `/__diagnostico` existe só quando `EXPO_PUBLIC_E2E=1` (grupo de rotas condicional + exclusão no bundle de produção). Alternativa: React Navigation puro (mais boilerplate, sem deep link/web automáticos).

3. **Camadas e composition root manual.** `src/domain`, `src/application`, `src/infrastructure`, `src/ui`, mais `src/composition` (único lugar que importa de `infrastructure` para a UI). Dependências por interface (`Database`, `SecureStore`, `Preferences`, `Logger`, `Notifier`, `HttpClient`) injetadas via React Context + factory. A fronteira é imposta por `eslint-plugin-boundaries` (ou `dependency-cruiser`) e por um teste de arquitetura em Jest. Alternativas: container DI (tsyringe/inversify — decorators e reflect-metadata pesam no Hermes e na web); import direto de adapters (viola as specs).

4. **Seleção de adapter por plataforma via extensões de arquivo do Metro** (`*.native.ts` / `*.web.ts`) atrás da mesma interface, com um fake em memória para Jest. A seleção ocorre no composition root, não nas telas.

5. **Banco: `expo-sqlite` + Drizzle ORM, migrations geradas por `drizzle-kit` e aplicadas no boot (`useMigrations`/`migrate`).** Helper de schema `baseColumns` (`id` UUID, `version`, `syncStatus`, `deletedAt`). Listagens via `useLiveQuery` (usa o listener de mudança do expo-sqlite). Para joins, a query declara as tabelas observadas e há teste por tela. Alternativas: WatermelonDB (modelo próprio, lazy loading, menos alinhado ao SQL que o backend usa); OP-SQLite (mais rápido, mas foge do ecossistema Expo); MMKV para dados (sem relacional). **Web:** `expo-sqlite` WASM/OPFS é o primeiro candidato; o spike decide entre ele e **IndexedDB** (via Dexie ou wrapper próprio) atrás da interface `Repository`, validada por uma **suíte de contrato** executada contra os dois adapters.

6. **Estado: Redux Toolkit com `listenerMiddleware`; TanStack Query só para HTTP.** Slices iniciais: `ui` (sheet aberto, toast atual) e `session` (placeholder). Teste de reflexão garante que nenhum slice contenha entidades do `domain`. Eventos cross-cutting (erro → toast, futuramente sync concluído → reagendar) passam por listeners. Alternativa Zustand foi descartada pela decisão do usuário (alinhamento com o Fluxor do MAUI).

7. **Tema: tokens em TS (`src/ui/theme/tokens.ts`), derivados do mockup.** Os valores `oklch` do mockup são **convertidos uma vez para hex** e commitados (RN/Hermes não interpreta `oklch`); `useColorScheme()` escolhe light/dark e um `ThemeProvider` expõe `useTheme()`. Regra de lint proíbe literais de cor fora de `theme/`. Os tokens dark não existem no mockup; serão **derivados** (decisão de design, validada visualmente no smoke e revisável pelo usuário).

8. **Ícones: `lucide-react-native`** (SVG via `react-native-svg`, mesmo set do mockup), funcionando em Android e web, o que elimina o problema do SkiaSharp do MAUI.

9. **BottomSheet:** primeira opção `@gorhom/bottom-sheet` (depende de `react-native-reanimated` e `react-native-gesture-handler`). O suporte web do gorhom é parcial; o **spike valida** o sheet no PWA, e o fallback é um componente próprio baseado em `Modal` + `Animated`, atrás da mesma API (`open/close/onClose`). O botão voltar do Android fecha o sheet via `BackHandler`.

10. **Rede:** cliente baseado em `fetch` com timeout (AbortController), erros tipados (`NetworkError`, `TimeoutError`, `AuthError`, `HttpError`) e `X-Device-Id`. URL por `EXPO_PUBLIC_API_URL` (`.env` gitignored; `.env.example` versionado). O device id é um UUID gerado uma vez e guardado em `Preferences`. TanStack Query envolve o cliente nos casos de uso remotos. Axios foi descartado (peso e nenhuma necessidade).

11. **Storage:** token em `expo-secure-store` (Android); preferências em `react-native-mmkv`. Web: IndexedDB/`localStorage` atrás das mesmas interfaces, com a limitação de segurança documentada (token na web não tem o mesmo nível de proteção).

12. **Logger próprio** com níveis e destinos plugáveis: `ConsoleSink` (todos os alvos) e `FileSink` (Android, `expo-file-system` no diretório do app, rotação por tamanho e quantidade, máscara de segredos por lista de chaves e padrões de token). Legível via `adb run-as <pkg> cat files/logs/...`. Falha ao gravar o log nunca propaga. Alternativas: `react-native-logs` (dependência extra para pouco ganho); só console (perde o diagnóstico em device).

13. **Erros:** `ErrorBoundary` na raiz (acima do Router) + `ToastProvider`. Um `notify()` global (emitter) conecta qualquer camada ao toast sem importar React.

14. **Notificações:** `expo-notifications` cria o canal Android no boot **sem pedir permissão**; `expo-background-task` registra uma tarefa no-op que só loga; ambos desligados na web por adapter `NoopNotifier`. A permissão e a lógica de alerta ficam para a change 7.

15. **PWA:** Expo Web com `output: "single"`; `public/manifest.webmanifest` (nome PlantumHibis, ícones 192/512 e maskable, `display: standalone`, cores do tema) e service worker gerado com `workbox-build` (precache do shell + fallback offline), registrado só em produção na web. Alternativa: `vite-plugin-pwa` (exigiria outro bundler, fora do pipeline do Expo).

16. **Docker:**
    - `Dockerfile.build`: base Node LTS exigida pelo SDK 56 + **JDK 17** + Android command-line tools (platform e build-tools conforme o `compileSdk` do template); `expo prebuild --platform android --no-install` e `./gradlew assembleRelease`. O APK de release é **assinado com keystore de debug** (apenas para E2E; assinatura real fora de escopo). Cache do Gradle em volume nomeado.
    - `Dockerfile.e2e`: emulador Android x86_64 headless (imagem de sistema API 34) + Maestro CLI; exige `/dev/kvm` (pré-checagem falha cedo com mensagem do README); screenshots/logs em `artifacts/`.
    - `Dockerfile.web` + serviço `web-e2e`: build estática do PWA servida localmente + Playwright (imagem oficial Playwright) com `colorScheme` e contexto offline.
    - `docker-compose.yml` com serviços `build`, `test`, `e2e`, `web-e2e` e `backend-mock` (para UC-07, só no E2E).
    - Alternativa ao Docker puro no Windows: rodar o `docker compose` dentro do **WSL2** com virtualização aninhada habilitada; plano B é um runner Linux (CI). O README documenta os dois caminhos.

17. **Testabilidade:** `testID` estável em todos os elementos que os E2E usam (abas, sheet, botões do diagnóstico), compartilhados entre Maestro e Playwright; fluxos Maestro em `e2e/android/*.yaml`, specs Playwright em `e2e/web/*.spec.ts`.

18. **Spikes primeiro (Task 1, com ponto de replanejamento):** (a) build do APK com JDK 17 no Docker; (b) emulador + Maestro com KVM no WSL2; (c) `expo-sqlite` na web vs IndexedDB; (d) BottomSheet e Reanimated no PWA. Se um spike falhar, o design é revisado **antes** de construir sobre ele.

19. **Bootstrap do repo:** o repo está vazio; o commit inicial (`openspec/`, `.claude/`, `.gitignore`, README mínimo) vai para `main`, `develop` é criada a partir dele e o trabalho ocorre em `worktrees/skeleton` na branch `feature/app-skeleton`. Commit/push/PR exigem autorização explícita; o PR sai por `gh` (remote GitHub) com squash em `develop`.

## Risks / Trade-offs

- [Emulador Android em Docker exige KVM; Windows nativo não expõe] → Spike (b); WSL2 com nested virtualization; plano B runner Linux; pré-checagem com mensagem clara.
- [`expo-sqlite` web instável (COOP/COEP, OPFS)] → Spike (c); adapter IndexedDB atrás da interface com suíte de contrato nos dois.
- [Gorhom bottom-sheet fraco na web] → Spike (d); fallback próprio com a mesma API.
- [Tokens dark inexistentes no mockup] → derivados e sinalizados no PR; revisão visual do usuário nos screenshots dos E2E.
- [Live query com join não observa todas as tabelas] → declarar tabelas observadas + teste de reatividade por tela.
- [Reanimated/worklets com a nova arquitetura e Hermes V1] → fixar versões compatíveis com o SDK 56 (`expo install`); build do spike valida.
- [Esqueleto grande] → tasks granuladas com commit por grupo; spikes no início; cada módulo de negócio fica fora.
- [Drift entre o PWA e o Android] → mesma UI/código; adapters por interface; E2E nos dois alvos.
- [Segredos em log/ambiente] → máscara no logger; `.env` ignorado; sem chaves de terceiros no cliente.
- [Trade-off] Dois alvos (Android + PWA) dobram a superfície de teste; aceito por decisão do usuário, mitigado por interfaces e suíte de contrato.

## Migration Plan

Repo novo, sem migração de dados. Sequência: bootstrap do repo (commit inicial + `develop`) → worktree → spikes → fundação → UI shell → dados/estado/rede → logs/erros/notificações → PWA → Docker/E2E → docs/graphify → PR. **Rollback:** descartar a branch `feature/app-skeleton`; nada fora do repo novo é alterado.

## Resolved Questions (Decisões dos Spikes 1.1 a 1.4)

- **Spike 1.1 (Build Android & SDK 56):** Confirmado Node 22 LTS, Gradle Wrapper 9.3.1, Kotlin 2.0.21, `compileSdk: 35`, `targetSdk: 35`, `minSdk: 24`, `buildToolsVersion: 35.0.0`, `ndkVersion: 27.1.12297006`. JDK 17 bastou e é totalmente suportado pelo Gradle 9.3.1 e AGP.
- **Spike 1.2 (Emulador KVM no Docker):** `/dev/kvm` está ativo no WSL2 do host e validado no Docker via `--device /dev/kvm` (permissões crw-rw---- root kvm). O serviço de emulador pode usar KVM diretamente.
- **Spike 1.3 (Banco na Web):** Definido o adapter IndexedDB (`idb`/Dexie) atrás da interface unificada `Repository`/`Database` para o alvo web, evitando dependência de cabeçalhos de isolamento cross-origin (COOP/COEP) exigidos pelo SQLite WASM/OPFS. O alvo Android nativo utiliza `expo-sqlite` + Drizzle ORM. Ambos são validados pela mesma suíte de testes de contrato.
- **Spike 1.4 (BottomSheet no PWA e Android):** Definida estratégia de adapter por extensão Metro: `@gorhom/bottom-sheet` (com Reanimated e GestureHandler) em `BottomSheet.native.tsx`, e sheet acessível baseado em `Modal` + `Animated` em `BottomSheet.web.tsx`, ambos compartilhando a mesma interface tipada de props (`isOpen`, `onClose`, `title`, `children`).
- **Valores dos tokens dark:** Serão derivados a partir da paleta base dos tokens light (PlantumHibis) e validados nos testes e screenshots.
- **Imagem de emulador:** Imagem baseada em `sdkmanager` com API 34 x86_64 e script de inicialização headless com verificação fail-fast de `/dev/kvm`.

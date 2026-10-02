## Why

O frontend MAUI depende do toolchain .NET/MAUI (build Android lento, frágil em path longo no Windows) e tem E2E cujo input não roda no sandbox. Falta uma base reproduzível por Docker, com testes que rodem de verdade, em stack web-first que permita também um PWA. Este esqueleto é a fundação sobre a qual os módulos do app serão migrados, um por change, a partir do RN.

## What Changes

- Novo projeto **Expo SDK 56 / React Native 0.85 / TypeScript strict** no repo `GreenKeeper.Frontend.Reactnative`, com alvo nativo **Android** e alvo **PWA** (Expo Web) sobre a mesma base.
- Camadas `domain / application / infrastructure / ui` com composition root manual; adapters por plataforma (Android vs web) atrás de interfaces.
- **Expo Router** com 5 abas (Início, Tarefas, Plantas, Alertas, Configurações) como placeholders; só Configurações abre um bottom sheet reutilizável.
- Tema PlantumHibis (tokens, light/dark pelo sistema) e ícones SVG via `react-native-svg`.
- **Redux Toolkit** só para estado de UI/sessão/eventos (listener middleware); regra "sem entidades de domínio no Store" com teste de reflexão. TanStack Query só para chamadas remotas.
- **expo-sqlite + Drizzle** com migrations e tabela base (`SyncStatus`, `version`); listagens por live query; fallback IndexedDB na web atrás da mesma interface (validado em spike).
- Cliente HTTP base (URL configurável, header `X-Device-Id`); SecureStore/MMKV e equivalentes web atrás de interface.
- Logger em arquivo (Android) + console; ErrorBoundary + toast padrão de erro.
- Base de notificações: canal Android, provider global de toast, estrutura de background task (sem lógica de alerta e sem pedir permissão ainda).
- PWA: manifest, ícones, service worker, instalável no Chrome/Edge/Windows, abre offline.
- **Docker:** build do APK (JDK 17 + Android SDK), E2E Android (emulador headless + Maestro) e E2E web (Playwright), cada um com smoke test.
- Jest + React Native Testing Library; arquivos de repo (`.gitignore`, `.graphifyignore`, `README.md`, `CLAUDE.md`/`AGENTS.md`/`GEMINI.md`) e graphify inicial.
- Sem breaking changes: repo novo; o MAUI e o Backend ficam intactos.

## Capabilities

### New Capabilities
- `project-foundation`: projeto Expo/TS strict, lint/format, camadas Clean Architecture, composition root e adapters por plataforma.
- `app-shell`: Expo Router com 5 abas, tema PlantumHibis light/dark, ícones SVG, BottomSheet reutilizável e sheet de Configurações.
- `ui-state-store`: Redux Toolkit para estado de UI/sessão/eventos, listener middleware, regra de não guardar entidades no Store.
- `local-database`: expo-sqlite + Drizzle, migrations, tabela base, listagens por live query, fallback IndexedDB na web.
- `network-and-storage`: cliente HTTP base, config de URL, `X-Device-Id`, SecureStore/MMKV e equivalentes web.
- `logging-and-error-handling`: logger em arquivo + console, ErrorBoundary, toast padrão de erro.
- `notifications-foundation`: canal Android, provider global de toast, estrutura de background task.
- `pwa-shell`: manifest, ícones, service worker, instalação e abertura offline.
- `docker-build-and-e2e`: imagens de build do APK, E2E Android (Maestro + emulador) e E2E web (Playwright), com smoke tests.
- `repo-bootstrap`: `.gitignore`, `.graphifyignore`, README, arquivos de instrução dos agentes, graphify inicial.

### Modified Capabilities
<!-- Nenhuma: repo novo, sem specs existentes. -->

## Impact

- **Repo novo:** todo o código nasce em `GreenKeeper.Frontend.Reactnative` (GitHub `wesleyxavier/GreenKeeper.Frontend.Reactnative`); `develop` precisa ser criada a partir de um commit inicial (repo vazio).
- **Dependências novas (npm):** `expo`, `expo-router`, `expo-sqlite`, `drizzle-orm`, `@reduxjs/toolkit`, `@tanstack/react-query`, `react-native-svg`, `@gorhom/bottom-sheet`, `react-native-mmkv`, `expo-secure-store`, `expo-notifications`, `expo-background-task`, `jest`/`jest-expo`/RNTL; `@playwright/test`; Maestro (CLI, só na imagem Docker).
- **Infra:** imagens Docker (`Dockerfile.build`, `Dockerfile.e2e`, Playwright) e `docker-compose.yml`; nenhum SDK instalado no host.
- **Riscos principais:** emulador Android em Docker exige KVM (Windows nativo não expõe; provável WSL2 com virtualização aninhada); `expo-sqlite` na web pode exigir o fallback IndexedDB; JDK 17 a confirmar no template do SDK 56. Os três são validados em spikes no início das tasks.
- **Fora de escopo:** módulos de negócio (auth, casas, áreas, plantas, tarefas, alertas, perfil, sync, dashboard), sync com o Backend, geração de alertas, deploy do PWA, iOS, alterações no Backend/MAUI.

# GreenKeeper Frontend (React Native & PWA)

Frontend moderno, reproduzível e offline-first do GreenKeeper, construído sobre **Expo SDK 56**, **React Native 0.85** e **TypeScript Strict**, com suporte unificado a **Android nativo** e **PWA (Web)**.

---

## 🌿 Visão Geral

O `GreenKeeper.Frontend.Reactnative` substitui a dependência do toolchain MAUI por uma arquitetura desacoplada, orientada a testes e executável 100% via Docker (regra de isolamento).

- **Multiplataforma:** Código compartilhado para Android e PWA.
- **Offline-First:** Armazenamento local reativo com Drizzle ORM (`expo-sqlite` no Android e IndexedDB na Web).
- **Clean Architecture:** Camadas estritas (`domain`, `application`, `infrastructure`, `ui`, `composition`) validadas por lint de fronteiras e testes de arquitetura.
- **UI Shell:** Expo Router com navegação em abas, tema PlantumHibis (tokens light/dark), ícones Lucide SVG e BottomSheet reutilizável.
- **Observabilidade e Diagnóstico:** Logger estruturado em arquivo (Android) e console, ErrorBoundary global, sistema unificado de notificações/toasts e rota diagnóstica para E2E.

---

## 🛠️ Stack Tecnológica

| Componente | Tecnologia |
|---|---|
| Framework Base | Expo SDK 56 / React Native 0.85 |
| Linguagem | TypeScript (Strict Mode) |
| Roteamento | Expo Router (File-based routing) |
| Estado de UI | Redux Toolkit + Listener Middleware |
| Estado Remoto | TanStack Query |
| Banco Local (Android) | `expo-sqlite` + Drizzle ORM |
| Banco Local (Web) | IndexedDB Adapter via interface unificada |
| Componentes & Ícones | Lucide React Native (SVG), BottomSheet nativo/web |
| Testes Unitários/Integrados | Jest + `jest-expo` + React Native Testing Library |
| Testes E2E | Maestro (Android Headless) & Playwright (Web PWA) |
| Containerização | Docker & Docker Compose |

---

## 📁 Estrutura de Pastas

```text
.
├── src/
│   ├── domain/               # Entidades, value objects e regras de domínio puras
│   ├── application/          # Casos de uso, portas (interfaces) e DTOs
│   │   └── ports/            # Database, SecureStore, Preferences, Logger, Notifier, HttpClient
│   ├── infrastructure/       # Implementação concreta dos adapters
│   │   ├── database/         # Drizzle SQLite (native) e IndexedDB (web)
│   │   ├── storage/          # SecureStore / MMKV (native) e localStorage (web)
│   │   ├── logging/          # ConsoleSink e FileSink
│   │   └── network/          # HttpClient com X-Device-Id e timeout
│   ├── ui/                   # Componentes visuais, telas, abas e tema
│   │   ├── theme/            # Tokens PlantumHibis, ThemeProvider e hooks
│   │   └── components/       # ScreenContainer, BottomSheet, Header, Toast
│   └── composition/          # Composition Root (único ponto de injeção de dependências)
├── app/                      # Rotas do Expo Router ((tabs), __diagnostico, etc.)
├── e2e/                      # Suítes de testes End-to-End
│   ├── android/              # Fluxos Maestro (.yaml)
│   └── web/                  # Specs Playwright (.spec.ts)
├── openspec/                 # Especificações e tracking de changes (OpenSpec)
└── docker-compose.yml        # Orquestração dos serviços Docker
```

---

## 🐳 Comandos Docker

Todo o ciclo de build, testes e validações deve rodar via `docker compose` conforme a regra `docker-isolation`:

```bash
# Executar testes unitários e de arquitetura (Jest)
docker compose run --rm test

# Executar verificação de tipos e lint
docker compose run --rm node npm run typecheck
docker compose run --rm node npm run lint

# Gerar build do APK Android
docker compose run --rm build

# Executar testes E2E no Android (Maestro + Emulador Headless)
docker compose run --rm e2e

# Executar testes E2E na Web (Playwright PWA)
docker compose run --rm web-e2e
```

---

## 🖥️ Requisitos KVM e WSL2

Para a execução do serviço de emulador Android (`e2e`) em containers Docker no Windows:
1. É necessário utilizar o **WSL2** com virtualização aninhada habilitada.
2. O dispositivo `/dev/kvm` deve existir no WSL2 (`ls -la /dev/kvm`).
3. O Docker Desktop deve ter a integração WSL2 ativada e permissão para montar dispositivos (`--device /dev/kvm`).

Se o ambiente host não possuir `/dev/kvm`, o container de E2E acusará falha imediata e a validação do emulador deverá ser executada via runner Linux (CI).

## Visão

**Problema:** O frontend MAUI exige o toolchain .NET/MAUI (workload pesado, build Android lento e frágil em path longo no Windows) e tem E2E cujo input não roda no sandbox. Falta uma base Android reproduzível por Docker, com testes automatizados que rodem de verdade e uma stack web-first que permita também um PWA.
**Solução:** App GreenKeeper/PlantumHibis em React Native + TypeScript (Expo SDK 56), com alvo nativo **100% Android** e um segundo alvo **PWA** instalável (Chrome/Edge/Windows) sobre a mesma base de código. Paridade com o comportamento Android do MAUI, migrada por módulos em changes sequenciais. Esta change entrega só o **esqueleto**: projeto, camadas, tema, 5 abas vazias, banco local, estado, HTTP, logs, erros, notificações (base), Docker de build/E2E e PWA. Build do APK e E2E (emulador Android + Playwright web) rodam via Docker.
**Usuários-alvo:** Wesley (dev) nesta change; usuários de plantas em Android e no PWA a partir das changes de módulos.

## Mapa Funcional

```mermaid
mindmap
  root((GreenKeeper RN - Esqueleto))
    Projeto
      Expo SDK 56 + TypeScript strict
      Expo Router - 5 abas
      ESLint + Prettier
    Camadas
      domain - entidades e regras puras
      application - casos de uso e interfaces
      infrastructure - db, http, storage, plataforma
      ui - telas e componentes
    UI Base
      Tema PlantumHibis light e dark
      Tokens de cor, espaçamento, tipografia
      Ícones SVG
      BottomSheet reutilizável
      Footer 5 abas
        Início
        Tarefas
        Plantas
        Alertas
        Configurações
    Estado
      Redux Toolkit - UI, sessão, eventos
      Listener middleware
      TanStack Query - só chamadas remotas
    Dados e Reatividade
      expo-sqlite + Drizzle
      Fallback IndexedDB na web
      Migrations
      Tabela base com SyncStatus e version
      Listagens via live queries
    Rede
      Cliente HTTP base
      Config de URL do backend
      Header X-Device-Id
    Storage
      SecureStore - token
      MMKV - preferências
      Equivalentes web atrás de interface
    Notificações
      expo-notifications
      Canal Android e permissão
      Background task
      Provider global de toast
    Erros e Logs
      ErrorBoundary
      Toast padrão de erro
      Logger em arquivo + console
      Leitura via adb run-as
    Web PWA
      Expo Web
      Manifest + ícones
      Service worker - offline
      Instalável no Windows
    Infra Docker
      Imagem de build Android SDK - JDK 17
      Script assembleRelease - APK
      Imagem de E2E com emulador
      Smoke test Maestro
      Playwright - E2E web
    Qualidade
      Jest + RNTL
      Cobertura de domain
```

## Jornadas de Usuário

### Dev

```mermaid
flowchart TD
    A([Dev: clona o repo]) --> B[npm install]
    B --> C[docker compose run build: prebuild + gradle assembleRelease]
    C --> D[APK gerado em artifacts/]
    D --> E[docker compose run e2e: sobe emulador headless]
    E --> F[Maestro instala APK e roda smoke test]
    F --> G{Smoke passou?}
    G -->|Sim| H[npm test: Jest]
    G -->|Não| I[Lê log e screenshot em artifacts/]
    I --> C
    H --> J[docker compose run web-e2e: Playwright no PWA]
    J --> K([Dev: pronto pra implementar módulos])
```

### Usuário Android

```mermaid
flowchart TD
    A([Usuário: instala o APK]) --> B[Abre o app]
    B --> C[Splash PlantumHibis]
    C --> D[Migrations do SQLite rodam no 1º boot]
    D --> E{Falha na inicialização?}
    E -->|Sim| F[Tela de erro padrão + log em arquivo]
    E -->|Não| G[Aba Início]
    G --> H[Footer: Início, Tarefas, Plantas, Alertas, Configurações]
    H --> I[Troca de aba: tela vazia com título]
    H --> J[Toca Configurações: abre bottom sheet]
    J --> K[Fecha o sheet]
    G --> L{Sistema em dark mode?}
    L -->|Sim| M[Tema escuro]
    L -->|Não| N[Tema claro]
    I --> O{Erro em runtime?}
    O -->|Sim| P[Toast de erro + log em arquivo]
    O -->|Não| Q([App estável])
    K --> Q
```

### Usuário PWA

```mermaid
flowchart TD
    A([Usuário: abre a URL no Chrome/Edge]) --> B[Service worker registra, assets em cache]
    B --> C[Banco local inicializa]
    C --> D{Storage web OK?}
    D -->|Não| E[Tela de erro padrão + log no console]
    D -->|Sim| F[Aba Início]
    F --> G[Navegador oferece Instalar app]
    G --> H[Usuário instala: janela própria, ícone no Windows]
    H --> I[Abre o app instalado]
    I --> J[Troca de aba, abre sheet de Configurações]
    J --> K{Sem internet?}
    K -->|Sim| L[App abre do cache, dados do banco local]
    K -->|Não| M[App normal]
    L --> N([App funcional offline])
    M --> N
```

## Arquitetura Macro

```mermaid
graph LR
    subgraph Alvos
        AND([Android APK])
        WEB([PWA - Chrome/Edge/Windows])
    end

    subgraph App["GreenKeeper RN (Expo, TypeScript)"]
        UI[ui: Expo Router + tema + BottomSheet]
        ST[Estado: Redux Toolkit - UI, sessão, eventos]
        APP[application: casos de uso + interfaces]
        DOM[domain: entidades e regras puras]
        INF[infrastructure: adapters]
    end

    subgraph Adapters["Adapters por plataforma"]
        DB[(expo-sqlite + Drizzle)]
        DBW[(IndexedDB - fallback web)]
        SEC[SecureStore / web storage]
        PREF[MMKV / localStorage]
        LOG[Logger: arquivo + console]
        NOT[expo-notifications + background task]
        HTTP[Cliente HTTP + TanStack Query]
    end

    BE[Backend FastAPI - inalterado]
    EXT[APIs externas: Wikipedia, GBIF, CEP, Open-Meteo]

    AND --> UI
    WEB --> UI
    UI --> ST
    UI --> APP
    ST --> APP
    APP --> DOM
    APP --> INF
    INF --> DB
    INF -.fallback.-> DBW
    INF --> SEC
    INF --> PREF
    INF --> LOG
    INF --> NOT
    INF --> HTTP
    HTTP --> BE
    HTTP --> EXT

    subgraph Docker
        BLD[build: Node + JDK17 + Android SDK]
        E2EA[e2e Android: emulador + Maestro]
        E2EW[e2e web: Playwright]
    end
    BLD -.gera.-> AND
    E2EA -.testa.-> AND
    E2EW -.testa.-> WEB
```

## Funcionalidades por Área

| Funcionalidade | Ator | Prioridade | Complexidade |
|---|---|---|---|
| Projeto Expo + TS strict + lint/format | Dev | Alta | P |
| Camadas domain/application/infrastructure/ui + composition root | Dev | Alta | M |
| Expo Router com 5 abas e placeholders | Usuário | Alta | P |
| Tema PlantumHibis light/dark (tokens) | Usuário | Alta | M |
| Ícones SVG (react-native-svg) | Usuário | Média | P |
| BottomSheet reutilizável + sheet de Configurações | Usuário | Alta | M |
| Redux Toolkit (UI/sessão/eventos) + regra "sem entidades no Store" | Dev | Alta | M |
| expo-sqlite + Drizzle + migrations + tabela base | Dev | Alta | G |
| Listagens por live query (regra de reatividade) | Dev | Alta | M |
| Fallback IndexedDB atrás da interface de banco (web) | Dev | Alta | G |
| Cliente HTTP base + config de URL + X-Device-Id | Dev | Alta | M |
| SecureStore / MMKV e equivalentes web atrás de interface | Dev | Média | M |
| Logger em arquivo (Android) + console | Dev | Alta | M |
| ErrorBoundary + toast padrão de erro | Usuário | Alta | M |
| Base de notificações (canal, provider global, background task) | Dev | Média | M |
| PWA: manifest, ícones, service worker, instalável, offline | Usuário | Alta | G |
| Docker build do APK (JDK 17 + Android SDK) | Dev | Alta | G |
| Docker E2E Android (emulador + Maestro) com smoke test | Dev | Alta | G |
| Docker E2E web (Playwright) com smoke test | Dev | Média | M |
| Jest + RNTL + testes de domain/Store | Dev | Alta | M |
| .gitignore, .graphifyignore, README, CLAUDE/AGENTS/GEMINI, graphify | Dev | Alta | P |

## Viabilidade

- [x] A solução resolve o problema declarado
- [x] Não duplica funcionalidade já existente no sistema (repo novo; MAUI segue como referência)
- [x] Stack tecnológica disponível ou definida (Expo SDK 56, RN 0.85, expo-sqlite + Drizzle, Redux Toolkit, Maestro, Playwright, JDK 17)
- [x] Escopo é delimitável (esqueleto sem módulos de negócio; módulos em changes sequenciais)
- [ ] Emulador Android em Docker roda neste ambiente (exige KVM; Windows nativo não expõe — provável WSL2 com virtualização aninhada; validar no spike)
- [ ] `expo-sqlite` funciona na web (WASM/OPFS) ou fallback IndexedDB cobre (validar no spike)
- [ ] JDK 17 confirmado como o exigido pelo template do Expo SDK 56 (validar no build do spike)
- [ ] Estimativa de esforço aceitável (esqueleto grande por incluir Docker + PWA; reavaliar após spikes)

## Riscos Iniciais

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Emulador Android no Docker sem KVM no Windows | Alta | Alto | Spike na task 1; rodar Docker dentro do WSL2 com nested virtualization; plano B: runner Linux/CI |
| `expo-sqlite` na web instável (COOP/COEP, OPFS) | Média | Alto | Spike; interface de banco com adapter IndexedDB de fallback sem mudar telas |
| Paridade Android vs web diverge (SecureStore, câmera, GPS, notificação) | Alta | Médio | Interfaces em `infrastructure`; adapters separados; recursos não suportados desligados explicitamente na web |
| Live queries com join não observam todas as tabelas | Média | Médio | Teste de reatividade por tela; regra de spec; observar tabelas explicitamente |
| Time põe listas/entidades de domínio no Redux | Média | Médio | Regra na spec + teste de reflexão sobre os slices |
| Versão do JDK/Gradle incompatível com o template do SDK 56 | Baixa | Médio | JDK só na imagem Docker; troca é de uma linha |
| Esqueleto grande demais (Docker + PWA + base) | Média | Médio | Tasks granuladas com commit por grupo; spikes primeiro com ponto de replanejamento |

## Fora de Escopo

- Telas e regras de negócio: auth, casas, áreas, plantas, tarefas, alertas, perfil, sync, dashboard (changes 2 a 10).
- Sync com o Backend (só o cliente HTTP base entra).
- Lógica de geração de alertas e permissão de notificação (só a base de infraestrutura).
- Hospedagem/deploy do PWA (roda local e no Docker).
- iOS e builds nativos Windows.
- Alterações no Backend FastAPI e no repo MAUI.
- Toggle manual de tema (segue o sistema).
- Publicação na Play Store e assinatura de release.

## UC-01: Abrir o app e navegar pelas abas

**Ator:** Usuário (Android e PWA)
**Tipo:** `Frontend only`
**Pré-condição:** App instalado (APK) ou PWA aberto; banco inicializado.

### Fluxo Principal
1. Usuário abre o app.
2. O app exibe a aba **Início** com título e estado vazio.
3. O footer mostra 5 ícones, com Início ativo.
4. Usuário toca em Tarefas, Plantas e Alertas em sequência.
5. Cada toque exibe a tela correspondente com título e estado vazio, e marca a aba como ativa.

### Fluxos Alternativos
- **2a.** A inicialização falha → UC-06 (tela de erro padrão).
- **4a.** Toque na aba já ativa → nada muda, sem erro.
- **4b.** Toque no ícone de Configurações → abre o sheet sem trocar a rota (UC-03).

### Cenários de Teste — Frontend Unit
> Testa componente isolado. Jest + React Native Testing Library.
- **Given** o layout das abas renderizado
  **When** o componente monta
  **Then** existem 5 ícones na ordem Início, Tarefas, Plantas, Alertas, Configurações, com Início ativo
- **Given** o footer renderizado
  **When** o usuário toca em Plantas
  **Then** a aba Plantas fica ativa e a tela mostra o título "Plantas"
- **Given** a aba Início ativa
  **When** o usuário toca em Início de novo
  **Then** o estado da navegação não muda

### Cenários de Teste — Frontend E2E
> Maestro (Android, emulador Docker) e Playwright (PWA, Docker).
- **Given** o APK instalado no emulador
  **When** o app abre e o usuário toca em Tarefas, Plantas e Alertas
  **Then** cada título aparece na tela e o app não fecha
- **Given** o PWA aberto no Chromium
  **When** o usuário clica em Tarefas, Plantas e Alertas
  **Then** a rota muda e o título correspondente aparece

---

## UC-02: Aplicar tema light/dark do sistema

**Ator:** Usuário
**Tipo:** `Frontend only`
**Pré-condição:** App aberto.

### Fluxo Principal
1. O app lê o esquema de cor do sistema.
2. Aplica os tokens do tema correspondente em fundo, texto, footer e sheet.
3. Se o sistema mudar com o app aberto, o tema troca sem reiniciar.

### Fluxos Alternativos
- **1a.** Esquema indisponível → usa light.

### Cenários de Teste — Frontend Unit
- **Given** esquema do sistema `dark`
  **When** o provider de tema monta
  **Then** `useTheme()` retorna os tokens escuros
- **Given** esquema muda de light para dark
  **When** o provider reage à mudança
  **Then** os componentes usam os tokens escuros
- **Given** o código das telas
  **When** o teste de arquitetura varre os arquivos de `ui`
  **Then** nenhuma tela usa cor hex literal fora dos tokens

### Cenários de Teste — Frontend E2E
- **Given** o emulador em dark mode
  **When** o app abre
  **Then** o screenshot do Início tem fundo escuro e texto legível
- **Given** o Playwright com `colorScheme: 'dark'`
  **When** o PWA abre
  **Then** o tema escuro é aplicado

---

## UC-03: Abrir e fechar o sheet de Configurações

**Ator:** Usuário
**Tipo:** `Frontend only`
**Pré-condição:** App aberto em qualquer aba.

### Fluxo Principal
1. Usuário toca em Configurações.
2. O BottomSheet abre sobre a tela atual.
3. Usuário fecha por gesto, botão ou backdrop.
4. O sheet some e a aba de fundo continua a mesma.

### Fluxos Alternativos
- **3a.** Botão voltar do Android fecha o sheet, não o app.

### Cenários de Teste — Frontend Unit
- **Given** sheet fechado
  **When** `open()` é chamado
  **Then** o sheet fica visível
- **Given** sheet aberto
  **When** o usuário toca no backdrop
  **Then** o sheet fecha e `onClose` é chamado
- **Given** o estado de UI no Store
  **When** o sheet abre
  **Then** o slice guarda apenas o flag de visibilidade, sem entidades

### Cenários de Teste — Frontend E2E
- **Given** a aba Plantas ativa
  **When** o usuário toca em Configurações e depois no backdrop
  **Then** o título "Plantas" continua visível
- **Given** o sheet aberto no Android
  **When** o usuário aciona o botão voltar
  **Then** só o sheet fecha e o app continua aberto

---

## UC-04: Inicializar banco e rodar migrations

**Ator:** Usuário (indireto)
**Tipo:** `Frontend only`
**Pré-condição:** Primeiro boot ou atualização do app.

### Fluxo Principal
1. O app abre o banco.
2. Aplica as migrations pendentes.
3. Libera as abas.

### Fluxos Alternativos
- **1a.** Banco existente e atualizado → pula as migrations.
- **2a.** Migration falha → tela de erro padrão (UC-06) e log; as abas não carregam.

### Cenários de Teste — Frontend Unit
- **Given** banco vazio em memória
  **When** `migrate()` executa
  **Then** as tabelas base existem com `version`, `syncStatus` e `deletedAt`
- **Given** uma migration inválida
  **When** `migrate()` executa
  **Then** lança erro tipado e chama `logger.error`
- **Given** banco já migrado
  **When** `migrate()` executa de novo
  **Then** é no-op
- **Given** a suíte de contrato de repositório
  **When** roda nos adapters Android e web
  **Then** ambos têm o mesmo comportamento

### Cenários de Teste — Frontend E2E
- **Given** o app instalado sem dados
  **When** abre
  **Then** chega à aba Início sem erro
- **Given** o PWA na primeira visita
  **When** abre
  **Then** o banco inicializa e as abas aparecem

---

## UC-05: Lista reativa atualiza com escrita externa

**Ator:** Dev / sistema
**Tipo:** `Frontend only`
**Pré-condição:** Tela de exemplo (somente build de teste) com live query montada.

### Fluxo Principal
1. A tela lista registros de uma tabela de exemplo via live query.
2. Um serviço insere ou atualiza um registro (simula o sync).
3. A lista reflete a mudança sem recarga manual.

### Fluxos Alternativos
- **2a.** Registro com `deletedAt` preenchido → sai da lista.
- **2b.** Mudança em tabela do join → a lista atualiza.

### Cenários de Teste — Frontend Unit
- **Given** lista montada e vazia
  **When** um serviço insere um registro
  **Then** a lista renderiza 1 item sem `refetch`
- **Given** lista com join entre as tabelas A e B
  **When** um registro de B muda
  **Then** a lista reexecuta e reflete a mudança
- **Given** um item na lista
  **When** `deletedAt` é preenchido
  **Then** o item some
- **Given** o estado inicial de todos os slices
  **When** o teste de reflexão inspeciona
  **Then** nenhum valor é entidade de domínio nem array delas

### Cenários de Teste — Frontend E2E
- **Given** a tela de exemplo no build de teste
  **When** o app grava um registro em background
  **Then** o item aparece sem interação do usuário

---

## UC-06: Recuperar de erro (ErrorBoundary, toast e log)

**Ator:** Usuário
**Tipo:** `Frontend only`
**Pré-condição:** App em execução.

### Fluxo Principal
1. Ocorre um erro.
2. Se fatal de renderização: o ErrorBoundary mostra a tela de erro com "Tentar de novo". Se não fatal (ex.: rede): o evento de erro mostra um toast.
3. O erro é gravado no log (arquivo no Android, console na web).
4. "Tentar de novo" remonta a árvore.

### Fluxos Alternativos
- **3a.** Falha ao gravar o log → não derruba o app.
- **4a.** Erro persiste → mantém a tela de erro.

### Cenários de Teste — Frontend Unit
- **Given** um componente que lança
  **When** renderiza dentro do ErrorBoundary
  **Then** exibe a tela de erro e chama `logger.error`
- **Given** o evento de erro emitido
  **When** o provider de toast recebe
  **Then** o toast aparece e some após o timeout
- **Given** um log contendo token
  **When** o logger formata a linha
  **Then** o segredo é mascarado
- **Given** o arquivo de log acima do limite
  **When** uma nova linha é gravada
  **Then** o arquivo rotaciona e os antigos além do limite são removidos

### Cenários de Teste — Frontend E2E
- **Given** o build de teste com botão "forçar erro"
  **When** o usuário toca nele
  **Then** a tela de erro aparece e o app não fecha
- **Given** o erro forçado no Android
  **When** o teste lê o log via `adb run-as`
  **Then** o arquivo contém a linha de erro

---

## UC-07: Chamada HTTP com `X-Device-Id` e erro de rede

**Ator:** Sistema
**Tipo:** `Frontend only` (backend mockado)
**Pré-condição:** URL do backend configurada.

### Fluxo Principal
1. O cliente obtém o id de device persistido (gera na primeira vez).
2. Envia a requisição com `X-Device-Id`.
3. Retorna a resposta tipada.

### Fluxos Alternativos
- **2a.** Sem conexão → erro tipado de rede, convertido em toast.
- **2b.** Timeout → erro tipado de timeout.
- **2c.** 401 → erro tipado de autenticação (tratamento na change de auth).

### Cenários de Teste — Frontend Unit
- **Given** um servidor mockado
  **When** o cliente faz duas chamadas
  **Then** o header `X-Device-Id` está presente e igual nas duas
- **Given** falha de rede
  **When** o cliente chama
  **Then** lança `NetworkError`
- **Given** o app reaberto
  **When** o id de device é lido
  **Then** é o mesmo de antes
- **Given** o storage seguro
  **When** salva, lê e apaga um valor
  **Then** a leitura devolve o valor salvo e depois nulo

### Cenários de Teste — Frontend E2E
- **Given** um mock de backend em container no compose
  **When** a tela de diagnóstico (build de teste) chama `/version`
  **Then** exibe o resultado
- **Given** o mock parado
  **When** a mesma chamada é feita
  **Then** o toast de erro de rede aparece

---

## UC-08: Instalar e abrir o PWA offline

**Ator:** Usuário PWA
**Tipo:** `Frontend only`
**Pré-condição:** Primeira visita com rede.

### Fluxo Principal
1. Abre a URL; o service worker registra e cacheia os assets.
2. O navegador oferece instalar.
3. Usuário instala.
4. Sem rede, abre o app instalado a partir do cache.
5. Navega entre abas e usa o banco local.

### Fluxos Alternativos
- **1a.** Navegador sem suporte a service worker → funciona online, sem instalação.
- **4a.** Cache limpo → tela de erro com orientação de reconectar.

### Cenários de Teste — Frontend Unit
- **Given** o manifest
  **When** é validado
  **Then** tem nome, ícones 192 e 512, `display: standalone`, `start_url` e cores do tema
- **Given** o service worker
  **When** a lista de precache é gerada
  **Then** inclui os assets do shell

### Cenários de Teste — Frontend E2E
- **Given** a primeira visita (Playwright)
  **When** a página carrega
  **Then** o service worker fica ativo e o manifest é válido
- **Given** o contexto offline após a visita
  **When** a página recarrega
  **Then** a aba Início aparece e a troca de abas funciona

---

## UC-09: Dev builda o APK via Docker

**Ator:** Dev
**Tipo:** `Frontend only`
**Pré-condição:** Docker rodando; clone limpo; `npm install` feito.

### Fluxo Principal
1. Dev roda `docker compose run build`.
2. A imagem executa `expo prebuild` e `gradlew assembleRelease`.
3. O APK é gravado em `artifacts/`.

### Fluxos Alternativos
- **1a.** Sem Docker → mensagem apontando para o README.
- **2a.** Falha no Gradle → código ≠ 0 e log completo em `artifacts/`.

### Cenários de Teste — Frontend Unit
- **Given** o script de pós-build
  **When** não há APK em `artifacts/`
  **Then** termina com código ≠ 0 e mensagem clara
- **Given** o `app.json`
  **When** o script lê a versão e o pacote
  **Then** os valores batem com o esperado

### Cenários de Teste — Frontend E2E
- **Given** clone limpo
  **When** roda `docker compose run build`
  **Then** `artifacts/*.apk` existe, o pacote confere (`aapt dump badging`) e o código de saída é 0

---

## UC-10: Dev roda E2E Android e web via Docker

**Ator:** Dev
**Tipo:** `Frontend only`
**Pré-condição:** APK gerado (UC-09); KVM disponível para o Android.

### Fluxo Principal
1. `docker compose run e2e` sobe o emulador headless, instala o APK e roda o Maestro.
2. `docker compose run web-e2e` roda o Playwright no PWA.
3. Resultados, screenshots e logs ficam em `artifacts/`.

### Fluxos Alternativos
- **1a.** Sem `/dev/kvm` → falha cedo com mensagem apontando para o README.
- **1b.** Fluxo falha → screenshot + log em `artifacts/` e código ≠ 0.

### Cenários de Teste — Frontend Unit
- **Given** o script de pré-checagem
  **When** `/dev/kvm` não existe
  **Then** retorna erro com a mensagem do README

### Cenários de Teste — Frontend E2E
- **Given** os dois alvos disponíveis
  **When** os smokes de UC-01 a UC-03 rodam no Android e na web
  **Then** ambos passam e os artefatos são gerados

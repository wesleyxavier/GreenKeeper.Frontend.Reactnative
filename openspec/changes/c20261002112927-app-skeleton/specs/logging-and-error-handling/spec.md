## ADDED Requirements

### Requirement: Logger em arquivo e console
O app SHALL ter um logger com níveis (debug, info, warn, error) que escreve no console e, no Android, em arquivo rotativo no diretório do app, legível via `adb run-as`. Na web SHALL escrever só no console.

#### Scenario: Log no Android
- **WHEN** o app registra um erro
- **THEN** a linha aparece no console e no arquivo de log do app

#### Scenario: Rotação
- **WHEN** o arquivo de log passa do limite configurado
- **THEN** um novo arquivo é iniciado e os mais antigos além do limite são removidos

#### Scenario: Sem dados sensíveis
- **WHEN** uma requisição autenticada é logada
- **THEN** o token e demais segredos não aparecem no log

### Requirement: ErrorBoundary global
O app SHALL envolver a árvore de telas em um ErrorBoundary que exibe tela de erro padrão com opção de tentar de novo e loga o erro.

#### Scenario: Erro de renderização
- **WHEN** um componente lança durante a renderização
- **THEN** a tela de erro padrão é exibida, o erro é logado e o app não fecha

### Requirement: Toast padrão de erro
O app SHALL oferecer um toast global para erros não fatais, acionável por qualquer camada via evento, com mensagem amigável ao usuário.

#### Scenario: Erro não fatal
- **WHEN** uma chamada remota falha e o caso de uso emite o evento de erro
- **THEN** o toast é exibido sobre a tela atual e some após alguns segundos

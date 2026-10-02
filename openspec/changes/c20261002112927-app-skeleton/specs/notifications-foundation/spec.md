## ADDED Requirements

### Requirement: Canal de notificação Android
O app SHALL criar um canal de notificação Android na inicialização, sem solicitar a permissão de notificação ao usuário nesta etapa.

#### Scenario: Primeiro boot no Android
- **WHEN** o app inicia no Android
- **THEN** o canal existe nas configurações do sistema e nenhum pedido de permissão é exibido

### Requirement: Provider global de toast e notificação in-app
O app SHALL ter um provider no root que exibe notificações in-app independentemente da tela aberta.

#### Scenario: Evento com app aberto
- **WHEN** um evento de notificação in-app é emitido enquanto qualquer aba está aberta
- **THEN** o toast aparece sem o usuário abrir uma tela específica

### Requirement: Estrutura de background task
O app SHALL registrar uma tarefa de background vazia (no-op, logada) no Android, pronta para receber a geração de alertas em change futura, e SHALL desligá-la explicitamente na web.

#### Scenario: Registro no Android
- **WHEN** o app inicia no Android
- **THEN** a tarefa está registrada e sua execução gera uma linha de log

#### Scenario: Web
- **WHEN** o app inicia na web
- **THEN** nenhuma tarefa de background é registrada e nenhum erro é lançado

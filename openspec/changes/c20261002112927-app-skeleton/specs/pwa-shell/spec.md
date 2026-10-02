## ADDED Requirements

### Requirement: PWA instalável
A build web SHALL ter manifest (nome PlantumHibis, ícones, `display: standalone`, cores do tema) e service worker, atendendo aos critérios de instalação do Chrome e do Edge.

#### Scenario: Instalação
- **WHEN** o usuário abre o PWA no Chrome ou Edge
- **THEN** o navegador oferece "Instalar app" e, após instalar, o app abre em janela própria

### Requirement: Abertura offline
Depois da primeira visita, o PWA SHALL abrir sem rede a partir do cache do service worker e usar o banco local.

#### Scenario: Sem internet
- **WHEN** o usuário abre o PWA instalado sem conexão
- **THEN** a aba Início é exibida e a navegação entre abas funciona

### Requirement: Mesma UI e tema do Android
O PWA SHALL exibir o mesmo footer, as mesmas abas, o mesmo tema e o mesmo bottom sheet, seguindo `prefers-color-scheme`.

#### Scenario: Tema escuro no navegador
- **WHEN** o navegador está em preferência escura
- **THEN** o PWA usa os tokens do tema escuro

## ADDED Requirements

### Requirement: Navegação por 5 abas
O app SHALL usar Expo Router com um footer de 5 ícones Lucide (sem rótulo visível, com `accessibilityLabel`) na ordem Início, Tarefas, Plantas, Alertas e Configurações. Início, Tarefas, Plantas e Alertas são rotas com título e estado vazio ("Em breve"); o ícone de Configurações é um gatilho que abre o sheet, sem rota própria.

#### Scenario: Troca de abas
- **WHEN** o usuário toca em Tarefas, Plantas e Alertas
- **THEN** a tela correspondente é exibida com seu título e a aba fica marcada como ativa

#### Scenario: Configurações não é rota
- **WHEN** o usuário toca no ícone de Configurações
- **THEN** o sheet abre e a rota atual não muda

#### Scenario: Aba inicial
- **WHEN** o app abre
- **THEN** a aba Início é exibida

### Requirement: Tema PlantumHibis light e dark
O app SHALL aplicar tokens de cor, espaçamento e tipografia do tema PlantumHibis e seguir o esquema de cor do sistema (claro/escuro), sem cores hardcoded nas telas.

#### Scenario: Sistema em dark mode
- **WHEN** o sistema está em modo escuro
- **THEN** fundo e texto usam os tokens do tema escuro e o texto permanece legível

#### Scenario: Sistema em light mode
- **WHEN** o sistema está em modo claro
- **THEN** fundo e texto usam os tokens do tema claro

### Requirement: Ícones SVG
Os ícones SHALL ser renderizados via `react-native-svg` a partir de arquivos SVG do projeto, tanto no Android quanto na web.

#### Scenario: Ícones do footer
- **WHEN** o footer é exibido
- **THEN** cada aba mostra seu ícone SVG nos estados ativo e inativo

### Requirement: BottomSheet reutilizável
O app SHALL oferecer um componente BottomSheet reutilizável (abrir, fechar por gesto, botão, backdrop ou botão voltar do Android) e o ícone de Configurações SHALL abri-lo, listando Perfil, Sincronização, Casas, Áreas e Sobre o App desabilitados com selo "em breve".

#### Scenario: Abrir e fechar
- **WHEN** o usuário toca no ícone de Configurações e depois fecha o sheet
- **THEN** o sheet abre e fecha sem trocar a aba de fundo

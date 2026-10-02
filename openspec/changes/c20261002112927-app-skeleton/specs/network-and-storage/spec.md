## ADDED Requirements

### Requirement: Cliente HTTP base
O app SHALL ter um cliente HTTP com URL base configurável por ambiente, timeout, tratamento de erro padronizado e header `X-Device-Id` em toda chamada ao backend.

#### Scenario: Header de device
- **WHEN** o cliente faz uma chamada ao backend
- **THEN** a requisição inclui `X-Device-Id` com o identificador estável do device

#### Scenario: Erro de rede
- **WHEN** a chamada falha por falta de conexão
- **THEN** o cliente lança um erro tipado de rede, que a UI converte em toast

### Requirement: Identificador estável de device
O app SHALL gerar um id de device uma vez e persisti-lo, retornando o mesmo valor nas execuções seguintes.

#### Scenario: Reabrir o app
- **WHEN** o app é fechado e reaberto
- **THEN** o id de device é o mesmo

### Requirement: Storage seguro e preferências atrás de interface
O token SHALL ser guardado por uma interface de storage seguro (SecureStore no Android) e as preferências por outra (MMKV no Android), com equivalentes web, sem que `ui` ou `application` conheçam a implementação.

#### Scenario: Salvar e ler token
- **WHEN** um valor é salvo no storage seguro e lido em seguida
- **THEN** o valor lido é igual ao salvo, e após apagar a leitura retorna nulo

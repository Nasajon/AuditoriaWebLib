# AuditoriaWebLib

Biblioteca destinada a integração das aplicações web Nasajon com o módulo de AuditoriaWeb.

_Obs.: Esta biblioteca foi planejada para implementação nas diferentes linguagens web hoje utilizadas na empresa, notadamente PHP e Python._

## Principais features

* Registro de eventos diversos ocorridos no ambiente de aplicações WEB da Nasajon.
    * Por evento, deve-se entender principalmente a execução de transações MOPE (incluindo eventos CRUD). Mas, a biblioteca contém preventivas que permitam o registro de ações não diretamente relacionadas ao uso de um sistema específico (por exemplo, estatísticas de interesse funcional ou para marketing, como a quantidade de empresas usando determinada instituição financeira parceira, ou até a quantidade de usuários por módulo).
* Consulta de eventos realizados sobre um determinado objeto
    * Isto é, transações realizadas sobre uma determinada entidade. De modo que seja possível demonstrar um histórico de auditoria do objeto.
* Consulta de eventos (fitrados por tipo) realizados sobre qualquer objeto.
    * De modo que seja possível visualizar históricos por ação, como as exclusões de objetos de uma determinada entidade, por exemplo.

## Documentação das APIs

O uso do termo API neste contexto se refere ao sentido original do termo (Application Program Interface), e, por ser tratar de uma biblioteca (e não aplicação REST), o termo consiste portanto numa documentação de código (e não de endpoints HTTP).

Conforme dito à princípio, o objetivo é a implementação da iblioteca em mais de uma linguagem. Portanto, a presente documentação foi concebida antes da implementação em si, e será descrita de modo a não estar diretamente relacionada a nenhuma linguagem em particular.

### Classe EventoClient

Classe reponsável pela manipulação dos eventos, incluindo as operações de gravação, recuperação, etc.

#### Parâmetros do construtor

| Parâmetro | Tipo | Descrição |
| - | - | - |
| modulo | string | Identificador do módulo web utilizando a biblioteca. |
| chave_enfileriamento | string | Conteúdo da chave de acesso à API de enfileiramento (usada no enfileiramento dos eventos ocorridos, para posterior processamento e gravação eno banco da AuditoriaWeb). |

#### Atributos

| Atributo | Tipo | Visbilidade | Descrição |
| - | - | - | - |
| modulo | string | público | Identificador do módulo web utilizando a biblioteca. |
| chave_enfileriamento | string | privado |  Conteúdo da chave de acesso à API de enfileiramento. |

#### Métodos

##### record()

Registra um novo evento no módulo AuditoriaWEB, por meio de enfileiramento do mesmo.

###### Parâmetros

| Parâmetro | Tipo | Descrição | Obrigatório |
| - | - | - | - |
| tenant | int | Tenant da instalação onde ocorreu o evento. | Sim |
| codigo_mope | string | Código mope que identifica a transação referente ao evento realizado. | Sim |
| conta_nasajon | string | Conta do usuário responsável pela execução da transação (e-mail). | Sim |
| data_hora | datetime | Objeto contendo data e hora de ocorrência do evento. | Sim |
| sub_codigo<sup>1</sup> | string | Código adicional à transação MOPE, para especificar detalhes de uma transação. | Não |
| descricao | string | Texto livre descritivo do evento. | Não |
| object_old<sup>2</sup>  | json | Representação JSON da entidade relacionada, antes da execução do evento. | Não |
| object_new<sup>2</sup>  | json | Representação JSON da entidade relacionada, após a execução do evento. | Não |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) relacionado à transação. | Não |
| detalhes | json ou string | Valor adicional livre (para registro opcional de dados no evento). | Não |
| insert | boolean | Flag indicando se o objeto está sendo inserido na realização do evento. | Não |
| delete | boolean | Flag indicando se o objeto está sendo excluído na realização do evento. | Não |

1. _Este código está sendo inserido por dois motivos:_
   1. _Prevenção para necessidades ainda não previstas de relacionamentos 1 X N entre o código MOPE e os eventos que se desejam registrar._
   2. _Possibilidade de registro de eventos não diretamente relacionados ao uso do sistema (exemplo, atualizações de versão, ou registro de características dos clientes, como a quantidade de usuários, ou as instituições bancárias utilizadas - desde que haja necessidade funcionais, e consentimento por parte dos usuários)._
   3. _Sugere-se utilização do _codigo_mope_ "000.000" para eventos onde só se aplique o _sub_codigo_ (no entanto, este caso ainda precisa ser analisado com relação à pertinência, isto é, sobre se de fato ocorrerá). A princípio, isto deve ser evitado, mas foi concebido como preventiva para situações extremas no futuro._
2. _As propriedades ```object_old``` e ```object_new``` não são necessariamente gravadas no BD do AuditoriaWeb, antes, apenas as diferenças são registradas._
   1. _No caso em que se passe apenas o ```object_new```, este é ignorado (porque, na inserção de novos registros de uma entidade, não é desejada a replicação do dado presente na tabela original)._
   2. _Mas, no caso em que se passe apenas o ```object_old```, o mesmo é integralmente gravado (porque a deleção de uma entidade precisa gravar seu estado último)._
   3. _Por fim, se for desejado gravar sempre um dado, sem análise de diferenças, utilize o campo ```detalhes```._
###### Exceções

* **UnauthorizedException:** Erro de autenticação junto ao servidor de enfileiramento (causa provável: chave de comunicação, passada no construtor, inválida).
* **UnknownException:** Erro desconhecido durante enfileiramento do evento (ver mensagem da exceção, para identificação da causa).
###### Retorno

Método sem retorno.

##### list_by_object()

Lista os eventos registrados para um determinado objeto (permitindo a exibição de um histórico de auditoria sobre um determinado objeto ou registro de entidade).

###### Parâmetros

| Parâmetro | Tipo | Descrição | Obrigatório |
| - | - | - | - |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) alterado pela transação. | Sim |
| codigos_mope<sup>1</sup> | List[string] | Lista de códigos mope das transações desejadas. | Não |
| sub_codigos<sup>2</sup> | List[string] | Lista de códigos auxiliares para identificação dos eventos desejados. | Não |
| access_token<sup>3</sup> | string | Token de autenticação do usuário logado no módulo cliente. | Sim |

1. _Filtro opcional de códigos MOPE (para viabilizar um histórico de operações apenas CRUD, por exemplo)._
2. _Filtro opcional de códigos auxiliares, para viabilizar um histórico de operações fora da MOPE (ou com granularidade mais fina que a própria MOPE)._
3. _O acesso aos mecanismos da AuditoriaWeb também serão personificados em nome do mesmo usuário do módulo web cliente._

###### Exceções

* **UnauthorizedException:** Erro de autenticação junto ao servidor do módulo de Auditoria WEB (causa provável: access_token inválido).
* **AuthenticationException:** Falta de permissão para acesso ao histórico do objeto.
* **NotFoundException:** Objeto não encontrado nos registros de auditoria.
* **UnknownException:** Erro desconhecido durante recuperação do histórico (ver mensagem da exceção, para identificação da causa).

###### Retorno

Retorna uma lista dos eventos, em ordem cronológica descrescente, ocorridos para o objeto solicitado. Dados contidos em cada evento:

| Atributo | Tipo | Descrição | Not Null |
| - | - | - | - |
| modulo | string | Identificador do módulo web de origem do evento. | Sim |
| tenant | int | Tenant da instalação onde ocorreu o evento. | Sim |
| codigo_mope | string | Código mope de identificação da transação referente ao evento realizado. | Sim |
| conta_nasajon | string | Conta do usuário responsável pela execução da transação (e-mail). | Sim |
| data_hora | datetime | Objeto contendo data e hora de ocorrência do evento. | Sim |
| created_at | datetime | Objeto contendo data e hora de registro do evento (data e hora da inserção no BD de auditoria). | Sim |
| sub_codigo | string | Código adicional à transação MOPE, para especificar detalhes de uma transação. | Não |
| descricao | string | Texto livre descritivo do evento. | Não |
| object_diff<sup>1</sup> | json | Representação em JSON das alterações realizadas na entidade relacionada, por ocorrência do evento. | Não |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) relacionado à transação. | Não |
| detalhes | json ou string | Valor adicional livre (para registro opcional de dados no evento). | Não |
| insert | boolean | Flag indicando se o objeto foi inserido na realização do evento. | Não |
| delete | boolean | Flag indicando se o objeto foi excluído na realização do evento. | Não |
| symmetricds | boolean | Flag indicando que o evento foi realizado por intermédio do SymmetricDS (tendo origem nas aplicações Desktop). | Sim (default false) |

1. _Conterá toda a representação da entidade, para um caso de deleção do objeto (e será vazio, para um caso de inserção)._

##### list_by_code()

Lista os eventos registrados para determinados _codigo_mope_ (e adicionalmente sub_codigos), porém que tenham relação à qualquer objeto, e não a apenas um específico (permitindo a exibição de um histórico de auditoria sobre um determinado tipo de evento, independente do objeto de impacte).

_Obs.: Pode ser útil para auditoria de deleção de um tipo de entidade, por exemplo._

###### Parâmetros

| Parâmetro | Tipo | Descrição | Obrigatório |
| - | - | - | - |
| codigos_mope | List[string] | Lista de códigos mope das transações desejadas. | Sim |
| sub_codigos<sup>1</sup> | List[string] | Lista de códigos auxiliares para identificação dos eventos desejados. | Não |
| access_token<sup>2</sup> | string | Token de autenticação do usuário logado no módulo cliente. | Sim |

1. _Filtro opcional de códigos auxiliares, para viabilizar um histórico de operações "fora" da MOPE (ou com granularidade mais fina que a própria MOPE)._
2. _O acesso aos mecanismos da AuditoriaWeb também serão personificados em nome do mesmo usuário do módulo web cliente._

###### Exceções

* **UnauthorizedException:** Erro de autenticação junto ao servidor do módulo de Auditoria WEB (causa provável: ausência ou access_token inválido).
* **AuthenticationException:** Falta de permissão para acesso ao histórico desejado.
* **UnknownException:** Erro desconhecido durante recuperação do histórico (ver mensagem da exceção, para identificação da causa).

###### Retorno

Retorna uma lista dos eventos, em ordem cronológica descrescente, ocorridos para os códigos passados. Dados contidos em cada evento:

| Atributo | Tipo | Descrição | Not Null |
| - | - | - | - |
| modulo | string | Identificador do módulo web de origem do evento. | Sim |
| tenant | int | Tenant da instalação onde ocorreu o evento. | Sim |
| codigo_mope | string | Código mope da transação referente ao evento realizado. | Sim |
| conta_nasajon | string | Conta do usuário responsável pela execução da transação (e-mail). | Sim |
| data_hora | datetime | Objeto contendo data e hora de ocorrência do evento. | Sim |
| created_at | datetime | Objeto contendo data e hora de registro do evento (data e hora da inserção no BD de auditoria). | Sim |
| sub_codigo | string | Código adicional à transação MOPE, para especificar detalhes de uma transação. | Não |
| descricao | string | Texto livre descritivo do evento. | Não |
| object_diff<sup>1</sup> | json | Representação em JSON das alterações realizadas na entidade relacionada, por ocorrência do evento. | Não |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) relacionado à transação. | Não |
| detalhes | json ou string | Valor adicional livre (para registro opcional de dados no evento). | Não |
| insert | boolean | Flag indicando se o objeto foi inserido na realização do evento. | Não |
| delete | boolean | Flag indicando se o objeto foi excluído na realização do evento. | Não |
| symmetricds | boolean | Flag indicando que o evento foi realizado por intermédio do SymmetricDS (tendo origem nas aplicações Desktop). | Sim (default false) |

1. _Conterá toda a representação da entidade, para um caso de deleção do objeto (e será vazio, para um caso de inserção)._

##### retrieve_initial_state()

Reconstroi o estado inicial de um objeto, baseado no estado atual (ou no último estado, para um objeto já excluído), e nas diferenças realizadas ;por cada execução de evento.

_Obs.: Pode ser útil para detalhamento da auditoria de um objeto. Isto é, após a listagem dos eventos ocorridos sobre um objeto, se o usuário desejar visualizar os detalhes de inserção do mesmo, será necessário "calcular" o estado inical do objeto (já que, por questões de não replicaçao de dados, o estado inicial não é armazenado)._

###### Parâmetros

| Parâmetro | Tipo | Descrição | Obrigatório |
| - | - | - | - |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) desejado. | Sim |
| current_state | json | Representação JSON do estado atual do objeto. | Não<sup>1</sup> |
| access_token<sup>2</sup> | string | Token de autenticação do usuário logado no módulo cliente. | Sim |

1. _Para reconstrução do estado inicial de um objeto ainda não excluído, é necessário enviar o estado atual do mesmo (pois apenas as diferenças são armazenadas no BD de auditoria, não havendo um ponto de partida para a reconstrução). Mas, para um objeto já excluído, pode-se passar este parâmetro como nulo._
2. _O acesso aos mecanismos da AuditoriaWeb também serão personificados em nome do mesmo usuário do módulo web cliente._

###### Exceções

* **UnauthorizedException:** Erro de autenticação junto ao servidor do módulo de Auditoria WEB (causa provável: ausência ou access_token inválido).
* **AuthenticationException:** Falta de permissão para acesso ao histórico do objeto desejado.
* **MissingCurrentState:** Faltando envio do parâmetro _current_state_ (causa provável: o objeto ainda não foi excluído, e portanto só é possível reconstruir seu estado inicial recebendo o estado atual do mesmo).
* **NotFoundException:** Objeto não encontrado nos registros de auditoria.
* **UnknownException:** Erro desconhecido durante a reconstrução do estado inicial do objeto (ver mensagem da exceção, para identificação da causa).

###### Retorno

Retorna a representação em JSON do objeto em seu estado inicial (calculado), junto a uma flag, indicando se o estado é realmente o inicial, e outros dados.

_Obs.: Para objetos cuja inserção não tenha sido inicialmente registrada no banco de auditoria, o estado inicial retornado não se pode afirmar ser (com certeza) o estado realmente inicial (por falta de informações). Neste caso, a flag adicional ```estado_duvidoso``` será marcada como ```true```._

| Atributo | Tipo | Descrição | Not Null |
| - | - | - | - |
| initial_state | json | Representação em JSON do estado inicial do objeto. | Sim |
| estado_duvidoso | boolean | Flag indicando se estado inicial calculado é duvidoso (por falta de informações no banco de auditoria). | Sim (default false) |
| tenant | int | Tenant da instalação dona do objeto. | Sim |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) relacionado à transação. | Sim |
| modulo | string | Identificador do módulo web de origem do evento de inserção. | Não<sup>1</sup> |
| conta_nasajon | string | Conta do usuário responsável pela execução da transação (e-mail). | Não<sup>1</sup> |
| data_hora | datetime | Objeto contendo data e hora do evento de inseração do objeto. | Não<sup>1</sup> |
| descricao | string | Texto livre descritivo do evento de inserção. | Não<sup>1</sup> |

1. _Estas informações podem não estar preenchidas, caso a flag ```estado_duvidoso``` esteja marcada como ```true``` (pois seriam informações não disponíveis)._
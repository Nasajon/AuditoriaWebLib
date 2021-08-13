# AuditoriaWebLib

Biblioteca destinada a integração das aplicações web Nasajon, com o módulo de AuditoriaWeb.

_Obs.: Esta biblioteca foi planejada para implementação nas diferentes linguagens web, hoje utilizadas na empresa (notadamente PHP e Python)._

## Principais features

* Registro de eventos diversos ocorridos no ambiente de aplicações WEB da Nasajon
    * Por evento, deve-se entender a execução de transações MOPE (incluindo eventos CRUD), ou ações não diretamente relacionadas ao uso de um sistema específico (por exemplo, estatísticas de interesse funcional ou para marketing, como a quantidade de empresas usando determinada instituião financeira parceira, ou até quantidade de usuários por módulo).
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

#### Atributos

| Atributo | Tipo | Descrição |
| - | - | - |
| modulo | string | Identificador do módulo web utilizando a biblioteca. |

#### Métodos

##### registrar

Registra um novo evento no módulo AuditoriaWEB, por meio de enfileiramento do mesmo.

###### Parâmetros

| Parâmetro | Tipo | Descrição | Obrigatório |
| - | - | - | - |
| codigo_mope | string | Código mope da transação referente ao evento realizado. | Sim |
| conta_nasajon | string | Conta do usuário responsável pela execução da transação (e-mail). | Sim |
| data_hora | datetime | Objeto contendo data e hora de corrência do evento. | Sim |
| sub_codigo <sup>1</sup> | string | Código adicional à transação MOPE, para especificar detalhes de uma transação. | Não |
| descricao | string | Texto livre descritivo do evento. | Não |
| object_old | json | Representação JSON da entidade relacionada, antes da execução do evento. | Não |
| object_new | json | Representação JSON da entidade relacionada, após a execução do evento. | Não |
| object_id | uuid ou string | ID (normalmente uuid) do objeto (normalmente entidade) alterado pela transação. | Não |
| detalhes | json ou string | Valor adicional livre (para registro opcional de dados no evento). | Não |

1. _Este código está sendo inserido por dois motivos:_
   1. _Prevenção para necessidades ainda não previstas de relacionamentos 1 X N entre o código MOPE e os eventos que se desejam registrar._
   2. _Possibilidade de registro de eventos não diretamente relacionados ao uso do sistema (exemplo, atualizações de versão, ou registro de características dos clientes, como a quantidade de usuários, ou as instituições bancárias utilizadas - desde que haja necessidade funcionais, e consentimento por parte dos usuários)._

###### Exceções

* **UnauthorizedException:** Erro de autenticação junto ao servidor de enfileiramento (causa provável: ausência ou chave de comunicação incorreta).
* **QueuingException:** Erro durante enfileiramento do evento (ver mensagem da exceção, para identificação da causa).
###### Retorno

Método sem retorno.

# Analisando Arquiteturas de Sistemas de Software Populares - Facebook

## Requisitos Importantes

- Escalabilidade: Capacidade de suportar mais de 1 bilhão de usuários ativos, realizando bilhões de leituras e milhões de escritas por segundo.
- Desempenho: Reduzir a latência e otimizar a resposta do sistema para operações distribuídas geograficamente.
- Consistência: Manter a consistência dos dados em um sistema de cache distribuído e em várias localizações geográficas.
- Disponibilidade: Garantir alta disponibilidade e recuperação rápida de falhas.
- Manutenibilidade: Facilitar a manutenção e evolução do sistema ao longo do tempo.

## Decisões de Projeto

- Modelagem do Grafo Social:

  - Tabela de Objetos: Armazena os dados de nós, como usuários, postagens e comentários.
  - Tabela de Associações: Armazena as relações entre os nós, como amizades e likes.

  ![image](https://github.com/anacarv/Arquitetura-Meta-Facebook-/assets/38952487/9ab4b09e-d752-42f6-9bef-2f702f279003)
- Cache TAO: Um sistema de cache write-through distribuído que reduz a latência e a carga no banco de dados principal.
- Sharding do Banco de Dados: Particionamento lógico dos dados em shards, cada shard sendo hospedado em uma instância de banco de dados.
- MyRocks DB: Um motor de armazenamento baseado em LSM tree que substituiu o InnoDB para reduzir o uso de armazenamento e a latência. O MyRocks foi escolhido principalmente por:

  - Melhor Utilização do Armazenamento: Redução significativa no espaço de armazenamento necessário.
  - Desempenho Superior para Escritas: Otimização para operações de escrita intensiva.
  - Redução da Latência: Menor latência em comparação com o InnoDB, especialmente em workloads de escrita intensiva.

## Tecnologias Utilizadas

- MySQL/InnoDB: Utilizado inicialmente para armazenamento e replicação dos dados.
- GraphQL: para gerenciar dados e queries de forma eficiente.
- JavaScript/React: Para construção de interfaces de usuário.
- MyRocks DB: Motor de armazenamento baseado em LSM tree desenvolvido para otimizar o desempenho.
- Multifeed: Tecnologia utilizada para ranking no News Feed e Timeline.
- Thrift: Ferramenta de comunicação entre serviços.
- Memcached: Sistema de cache utilizado para melhorar o desempenho e reduzir a latência.

## Banco de Dados e Modelo de Dados

- Tabela de Objetos (Object Table):
  - id: Identificador único do objeto.
  - otype: Tipo do objeto (por exemplo, usuário, post, comentário).
  - data: Dados do objeto armazenados como pares chave-valor serializados.
  - Exemplo de entrada: 632 | “Post” | {“text”: “It was a great party! @ Bob”}
- Tabela de Associações (Association Table):
  - id1: Identificador do nó de origem.
  - id2: Identificador do nó de destino.
  - atype: Tipo da associação (por exemplo, comentário, like).
  - data: Dados da associação armazenados como pares chave-valor serializados.
  - Exemplo de entrada: 632 | 731 | Comment | null

## Backend Distribuído TAO:

- TAO expõe uma lista de APIs para consultar e alterar objetos e associações. Os servidores de aplicativos do Facebook se comunicam com o TAO em vez do banco de dados. As APIs podem ser classificadas em 2 categorias:
  - APIs de Objetos: Operações para alocar, recuperar, atualizar e deletar objetos.
  - APIs de Associações: Operações para adicionar, modificar e deletar associações, além de consultas específicas para associações.
- Cache: TAO atua como um cache write-through, armazenando agressivamente objetos e associações para reduzir a latência e a carga no banco de dados.

## Frontend:

- O frontend do Facebook utiliza principalmente JavaScript, com React sendo a biblioteca dominante para construção de interfaces de usuário. Além disso, o Facebook também adota GraphQL para gerenciar dados e queries de forma eficiente. A arquitetura é altamente modular, com componentes reutilizáveis sendo uma prática comum. Isso permite que novas funcionalidades sejam adicionadas e testadas de forma mais eficiente. Sendo projetado para funcionar em uma ampla gama de dispositivos e tamanhos de tela, garantindo uma experiência consistente do usuário em desktops, tablets e smartphones.
- Devido à escala massiva do Facebook, a performance e a acessibilidade são prioridade. Isso inclui técnicas avançadas de otimização de carga inicial, renderização de lado cliente (client-side rendering), estratégias de caching para minimizar o tempo de resposta e melhorar a experiência do usuário, conformidade com padrões de acessibilidade, testes rigorosos e melhorias contínuas na usabilidade.
- Como uma plataforma que lida com dados pessoais e transações sensíveis, a segurança é uma preocupação central no desenvolvimento. Medidas rigorosas são implementadas para proteger a integridade dos dados do usuário e prevenir vulnerabilidades.


## Arquitetura flux - Front End

A arquitetura Flux, desenvolvida pelo Facebook, é um padrão de design utilizado para gerenciar o fluxo de dados em aplicações JavaScript. Flux funciona em conjunto com o React, uma biblioteca JavaScript para construir interfaces de usuário. Vamos entender como funciona essa arquitetura:

### Componentes Principais do Flux

Ações (Actions):

* Ações são objetos simples que contêm informações sobre algo que aconteceu (um evento) na aplicação. Elas são despachadas para o Dispatcher.
* Cada ação geralmente tem um tipo (type), que é uma string constante que identifica a natureza da ação, e outros dados necessários para descrever a ação.

Dispatcher:

* O Dispatcher é um mecanismo central que gerencia todas as ações e as distribui para os stores apropriados.
* Atua como um hub central para o fluxo de dados, garantindo que todas as ações passem por ele antes de atingir os stores.

Stores:

* Stores contêm o estado da aplicação e a lógica para gerenciar esse estado. Eles são responsáveis por atualizar o estado em resposta às ações recebidas.
* Diferente dos modelos tradicionais, os stores não possuem métodos getters e setters típicos; em vez disso, expõem métodos que os componentes da aplicação podem usar para registrar listeners que respondem a mudanças no estado.

Views:

* As Views são componentes React que recebem dados dos stores e reagem às mudanças nesses dados.
* Elas podem despachar novas ações para o Dispatcher em resposta às interações do usuário.

### Fluxo de Dados Unidirecional

A principal característica do Flux é o fluxo de dados unidirecional. Aqui está um exemplo de como esse fluxo ocorre:

Usuário Interage com a View:

* O usuário realiza alguma ação (por exemplo, clica em um botão).
* A View despacha uma ação para o Dispatcher.

Ação é Despachada:

* A ação é um objeto que descreve o que aconteceu e é enviado ao Dispatcher.

Dispatcher Distribui a Ação:

* O Dispatcher envia essa ação para todos os stores registrados.

Stores Atualizam o Estado:

* Cada store que está interessado nessa ação irá atualizá-lo e alterar seu estado.
* Após atualizar, o store emite um evento de mudança.

Views Recebem Atualizações:

* As views registradas como listeners desses stores são notificadas sobre a mudança.
* As views obtêm o estado atualizado dos stores e se re-renderizam para refletir as mudanças.

### Vantagens do Flux

* Previsibilidade do Fluxo de Dados:

  * Com o fluxo de dados unidirecional, é mais fácil entender e rastrear como os dados fluem através da aplicação.
* Facilidade de Depuração:
* Como todas as ações passam pelo Dispatcher, é possível registrar e depurar facilmente todas as interações e mudanças de estado.
* Componentização e Reutilização:
* A separação clara entre as responsabilidades de ações, dispatcher, stores e views permite uma melhor organização do código e facilita a reutilização de componentes.

**

## Histórico de Mudanças no Projeto

- Uso Inicial do InnoDB: O motor de armazenamento InnoDB foi inicialmente utilizado, mas substituído pelo MyRocks DB para otimizar a performance.
  - Baseado em Log-Structured Merge (LSM) tree.
  - Reduziu o uso de armazenamento em 50% e melhorou a latência do banco de dados.
- Processo de Desnormalização: Um processo massivo de desnormalização foi implementado para suportar o modelo de ranking do Timeline.

## Padrões e Conceitos Envolvidos:

- Denormalização de Dados: Processo de transformar dados altamente normalizados em uma forma desnormalizada para melhorar a eficiência das consultas. A desnormalização envolve a transformação de dados altamente normalizados em uma forma menos normalizada para melhorar a eficiência das consultas e o desempenho geral do sistema. Isso geralmente significa armazenar dados redundantes em várias tabelas para evitar junções complexas durante as consultas.
- Sharding do Banco de Dados:
  - Técnica de particionamento dos objetos e associações em shards lógicos de dados para distribuir a carga entre várias instâncias de banco de dados.
  - As associações são sempre armazenadas no mesmo shard de seu id1 para otimizar as consultas.
- Cache Write-Through: Sistema de cache que escreve dados diretamente no cache ao mesmo tempo que os grava no banco de dados subjacente.
- Consistência Eventual: Modelo de consistência adotado para garantir que, eventualmente, todos os nós de um sistema distribuído terão a mesma visão dos dados.

## Referências:

- https://engineering.fb.com/2012/01/05/web/building-timeline-scaling-up-to-hold-your-life-story/
- https://medium.com/swlh/an-introduction-to-facebooks-system-architecture-47cfcf597101

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

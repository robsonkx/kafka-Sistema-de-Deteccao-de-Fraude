Objetivo:
Projetar um Sistema de Detecção de Fraude que analisa transações de cartão de crédito em tempo real e identifica atividades suspeitas. O sistema deve fornecer uma resposta imediata se a transação envolver um cartão, usuário ou site previamente identificado como fraudulento.

Para transações que não possuem histórico de fraude, o sistema utilizará um modelo de Machine Learning para identificar comportamentos suspeitos. O modelo irá prever quais transações são suspeitas e esse resultado será usado para alimentar a lista de bloqueio usada pelo módulo de resposta imediata, ajudando a prevenir futuras transações fraudulentas.

As transações são originadas nas maquininhas de pagamento, distribuídas pelos estabelecimentos comerciais ou via API, em caso de pagamento online. Pense em casos de uso como transações verdadeiras, transações de um usuário marcado como fraudulento, transações suspeitas, confirmação ou cancelamento de que a transação é fraudulenta etc.

Dados:

Uma transação é composta pelos campos: timestamp, transaction_id, user_id, card_id, site_id, value, location_id, country
Você terá acesso aos dados do usuário e dados de referência. Precisará determinar como armazená-los e para que usá-los:
Usuário: user_id, nome, endereço, email
Estabelecimento: site_id, nome, endereço, categoria de produtos (bens de consumo, viagens, restaurantes etc.)
Requisitos não funcionais:

Escalabilidade: O sistema deve suportar até 10 mil transações por segundo (TPS).
Disponibilidade: 99,9% de uptime.
Latência:
Resposta imediata:
P50: 1 segundo
P90: 5 segundos
Identificação de comportamentos suspeitos:
P50: 10 minutos
P90: 30 minutos
Armazenamento:
Todas as transações devem ser armazenadas por 180 dias.
Transações suspeitas também devem ser armazenadas.
O que você deve entregar:

Diagrama de Arquitetura: Apresente a arquitetura do sistema, mostrando os diversos módulos e como eles interagem.
  
Casos de Uso: Detalhe os casos de uso, explicando os fluxos de dados, como transações verdadeiras, fraudulentas, e falsos positivos.
  
Tecnologias: Identifique as tecnologias a serem usadas, explicando, por exemplo, a escolha do banco de dados (relacional, NoSQL, grafos, busca etc.).
  
Machine Learning: Detalhe a estratégia para coletar dados históricos e o fluxo de treinamento de modelos para detecção de transações suspeitas. Você NÃO precisa especificar qual modelo de ML será usado..

Evitar SPOF: Garanta que o sistema não tenha pontos únicos de falha (SPOF).

Monitoramento: Identifique que métricas você utilizará para definir se o sistema está saudável

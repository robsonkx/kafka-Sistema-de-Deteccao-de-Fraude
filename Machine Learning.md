Machine Learning
Coleta de dados históricos

Dados coletados de:

Kafka topics
fraud.alerts
fraud.confirmed
transactions.raw

Persistidos no Data Lake.

Feature Engineering

Criadas pelo pipeline:

Flink / Spark

Exemplos de features:

número de transações por minuto
valor médio por estabelecimento
distância entre compras
frequência de uso

Features armazenadas no:

Feature Store
Pipeline de Treinamento

Fluxo típico:

Data Lake
 → Feature extraction
 → Dataset de treinamento
 → treinamento ML
 → validação
 → deploy modelo

Treinamento pode ocorrer:

diário ou semanal
Deploy do Modelo

Modelos são expostos via:

TensorFlow Serving

ou microserviço de scoring.

Integração:

Fraud Check Service → ML Inference

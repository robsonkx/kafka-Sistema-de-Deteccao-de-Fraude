Origem e Coleta de Dados Históricos para Machine Learning
1. Origem dos Dados Históricos para ML

Os dados históricos utilizados para treinamento de modelos de Machine Learning de detecção de fraude são coletados principalmente de quatro fontes dentro da arquitetura:

Kafka — stream de eventos de transações

Data Lake — histórico bruto e datasets de treinamento

Bancos operacionais — Cassandra, PostgreSQL e Neo4j

Feedback humano — analistas de fraude

Fluxo simplificado
Transações → Kafka → Streaming Processing → Data Lake
                                  │
                                  ▼
                         Feature Engineering
                                  │
                                  ▼
                          Feature Store
                                  │
                                  ▼
                        Dataset de Treinamento ML
2. Kafka: Primeira Fonte de Dados

Dentro da arquitetura, Kafka é a origem dos eventos transacionais.

Principais tópicos:

transactions.raw
transactions.validated
fraud.alerts
fraud.confirmed
Exemplo de evento de transação
{
  "transaction_id": "tx123",
  "card_id": "card_987",
  "user_id": "user_456",
  "merchant_id": "mcdonalds_001",
  "amount": 120.50,
  "currency": "BRL",
  "timestamp": "2026-03-05T14:21:33",
  "location": {
      "lat": -23.55,
      "lon": -46.63
  },
  "device_id": "iphone_123"
}

Esses eventos:

são consumidos por Flink ou Spark Streaming

são gravados no Data Lake

alimentam pipelines de feature engineering

3. Data Lake: Principal Fonte Histórica

No diagrama de arquitetura aparece como:

❄️ COLD PATH
Data Lake (S3/GCS)
• Raw logs
• ML datasets
• Retenção de 7 anos

Este é o repositório principal de dados históricos usados para ML.

Estrutura típica do Data Lake
data-lake/
   transactions/
       year=2026/
           month=03/
               day=05/
                   part-000.parquet

   fraud_labels/
       year=2026/
           month=03/

   features/
       user_features/
       card_features/
Formatos de armazenamento comuns

Parquet

ORC

Delta Lake

Apache Iceberg

Motivos técnicos

compressão eficiente

leitura paralela distribuída

otimizado para analytics e ML

4. Feature Engineering (Streaming + Batch)

Os dados históricos não são usados diretamente pelos modelos.

Antes disso, são transformados em features de ML.

Ferramentas utilizadas:

Flink Streaming

Spark

Airflow pipelines

Exemplos de features geradas
Features de comportamento
tx_count_1h
tx_count_24h
avg_transaction_amount
Features estatísticas
amount_zscore
std_transaction_amount
Features geográficas
distance_last_transaction
country_change_rate
Features de dispositivo
new_device_flag
device_change_frequency
Exemplo de dataset de features
card_id | tx_count_1h | avg_amount | geo_distance | fraud_label
5. Feature Store (Online + Offline)

No diagrama aparece:

Feature Store (Feast / Tecton)
Online / Offline

A Feature Store mantém consistência entre treinamento e inferência.

Offline Store

Utilizada para treinar modelos ML.

Fonte de dados:

Data Lake

Exemplo:

training_dataset.parquet
Online Store

Utilizada para inferência em tempo real.

Fontes comuns:

Redis

Cassandra

DynamoDB

Exemplo:

card_123 → tx_count_last_1h = 4
6. Labels (Dados de Verdade)

Para que o modelo aprenda a detectar fraude, é necessário ter labels supervisionadas.

Essas labels vêm do feedback humano.

Fluxo presente no diagrama:

Analista de risco
      │
Console Web
      │
Decisão
      │
Confirmar Fraude / Falso Positivo

Quando o analista decide:

fraud_confirmed = 1
fraud_confirmed = 0

Eventos gerados:

fraud.confirmed
fraud.false_positive

Esses eventos seguem o fluxo:

Kafka → Data Lake → Dataset de treinamento
7. Construção do Dataset de Treino

Pipeline típico executado por:

Airflow

Spark Jobs

Fluxo
1. Ler transações do Data Lake
2. Juntar features
3. Juntar labels
4. Criar dataset ML
Exemplo de query
SELECT
   t.card_id,
   t.amount,
   f.tx_count_1h,
   f.geo_distance,
   f.device_change_rate,
   l.fraud_label
FROM transactions t
JOIN features f
ON t.card_id = f.card_id
JOIN labels l
ON t.transaction_id = l.transaction_id
Dataset final
card_id | amount | tx_count_1h | geo_distance | fraud_label
8. Treinamento do Modelo

Treinamento normalmente ocorre usando:

Spark ML

XGBoost

TensorFlow

PyTorch

Pipeline de treinamento
Data Lake
   ↓
Feature Dataset
   ↓
Training Job
   ↓
Model Artifact
   ↓
Model Registry
   ↓
TF Serving / ML Service
Exemplo de artefato de modelo
fraud_model_v14.xgboost
9. Atualização Contínua (Retraining)

Modelos de fraude não são estáticos.

Existe retraining contínuo.

Estratégias comuns
Batch retraining

Periodicidade:

diário ou semanal

Dataset utilizado:

últimos 90 dias de transações
Incremental learning

Treina com novos casos confirmados.

Eventos utilizados:

fraud.confirmed
fraud.false_positive
10. Resumo do Fluxo de Dados para ML

Fluxo completo:

POS / API
   │
   ▼
Kafka (transactions.raw)
   │
   ▼
Flink / Spark
   │
   ▼
Data Lake (histórico)
   │
   ▼
Feature Engineering
   │
   ▼
Feature Store
   │
   ▼
Dataset de Treinamento
   │
   ▼
Treinamento ML
   │
   ▼
Modelo
   │
   ▼
Inference Service
11. Onde o ML Busca os Dados Históricos

Em produção real, os dados utilizados pelo ML costumam vir de várias fontes:

Fonte	Tipo de dado
Kafka	eventos recentes
Data Lake	histórico completo
Cassandra	histórico operacional
PostgreSQL	dados mestre
Neo4j	relações de fraude
Redis	features em tempo real
Feedback humano	labels


# Origem dos Dados Históricos para ML

## 1. Fontes de Dados Históricos

Os dados históricos são coletados principalmente de 4 fontes dentro da arquitetura:

- **Kafka** (event stream de transações)
- **Data Lake** (histórico bruto e datasets de treino)
- **Bancos operacionais** (Cassandra / PostgreSQL / Neo4j)
- **Feedback humano** (analistas de fraude)

### Fluxo simplificado:
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

text

## 2. Kafka: Primeira Fonte de Dados

Dentro do seu diagrama, **Kafka é a origem do histórico de eventos**.

### Tópicos importantes:
- `transactions.raw`
- `transactions.validated`
- `fraud.alerts`
- `fraud.confirmed`

### Exemplo de evento:

Esses eventos:

são consumidos por Flink/Spark

são gravados no Data Lake

são usados para gerar features

3. Data Lake: Principal Fonte Histórica
No seu diagrama aparece:

text
❄️ COLD PATH
Data Lake (S3/GCS)
• Raw logs
• ML datasets
• 7 anos
Esse é o repositório principal de histórico para ML.

Estrutura típica:
text
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
Formatos geralmente usados:
Parquet

ORC

Delta Lake / Iceberg

Motivos:

compressão

leitura paralela

otimizado para ML e analytics

4. Feature Engineering (Streaming + Batch)
Os dados históricos não são usados diretamente.

Primeiro são transformados em features de ML.

Isso acontece via:

Flink Streaming

Spark

Airflow pipelines

Exemplos de features geradas
Features de comportamento:

tx_count_1h

tx_count_24h

avg_transaction_amount

Features estatísticas:

amount_zscore

std_transaction_amount

Features geográficas:

distance_last_transaction

country_change_rate

Features de device:

new_device_flag

device_change_frequency

Exemplo final:
card_id	tx_count_1h	avg_amount	geo_distance	fraud_label
5. Feature Store (Online + Offline)
No diagrama aparece:

text
Feature Store (Feast/Tecton)
Online / Offline
Ela mantém features consistentes entre treinamento e inferência.

Offline Store
Usada para treinar modelos.

Fonte: Data Lake

Exemplo: training_dataset.parquet

Online Store
Usada durante inferência em tempo real.

Fontes:

Redis

Cassandra

DynamoDB

Exemplo: card_123 → tx_count_last_1h = 4

6. Labels (Dados de Verdade)
Para ML aprender fraude, precisa de labels.

Essas labels vêm do Feedback Loop.

No seu diagrama:

text
Analista de risco
Console Web
Decisão
Confirmar Fraude / Falso Positivo
Quando o analista decide:

fraud_confirmed = 1

fraud_confirmed = 0

Isso gera eventos:

fraud.confirmed

fraud.false_positive

Esses eventos vão para:
Kafka → Data Lake → Dataset de treinamento

7. Construção do Dataset de Treino
Pipeline típico:
Airflow / Spark job

Fluxo:
Ler transações do Data Lake

Juntar features

Juntar labels

Criar dataset ML

Exemplo SQL:
sql
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
Dataset final:
card_id	amount	tx_count_1h	geo_distance	fraud_label
8. Treinamento do Modelo
O treinamento ocorre normalmente em:

Spark ML

XGBoost

TensorFlow

PyTorch

Pipeline:
text
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
Exemplo de artefato: fraud_model_v14.xgboost

9. Atualização Contínua (Retraining)
O modelo não é estático.

Existe retraining contínuo.

Estratégias comuns:
Batch retraining

todo dia / semana

Dataset usado: últimos 90 dias de transações

Incremental learning

treina com novos casos confirmados

Eventos usados: fraud.confirmed, fraud.false_positive

10. Resumo do Fluxo de Dados para ML
Fluxo completo:
text
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
11. Onde exatamente o ML "busca" os dados históricos
Em produção real, ML costuma buscar dados em:

Fonte	Tipo de dado
Kafka	eventos recentes
Data Lake	histórico completo
Cassandra	histórico operacional
PostgreSQL	dados mestre
Neo4j	relações de fraude
Redis	features em tempo real
Feedback humano	labels

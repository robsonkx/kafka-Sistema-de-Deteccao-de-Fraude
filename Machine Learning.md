
Markdown
Copy
Code
Preview
# 🧠 Origem dos Dados Históricos para ML

## 📊 Visão Geral das Fontes de Dados

Os dados históricos são coletados de **4 fontes principais** dentro da arquitetura:

| Fonte | Tipo de Dado |
|-------|-------------|
| **Kafka** | Event stream de transações |
| **Data Lake** | Histórico bruto e datasets de treino |
| **Bancos Operacionais** | Cassandra / PostgreSQL / Neo4j |
| **Feedback Humano** | Analistas de fraude |

---

## 🔄 Fluxo Simplificado
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
plain
Copy

---

## 1️⃣ Kafka: Primeira Fonte de Dados

Kafka é a origem do histórico de eventos no sistema.

### 📌 Tópicos Importantes

- `transactions.raw`
- `transactions.validated`
- `fraud.alerts`
- `fraud.confirmed`

### 📝 Exemplo de Evento

```json
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
⚡ Processamento dos Eventos
Consumidos por Flink/Spark
Gravados no Data Lake
Usados para gerar features
2️⃣ Data Lake: Principal Fonte Histórica
plain
Copy
❄️ COLD PATH
Data Lake (S3/GCS)
• Raw logs
• ML datasets
• 7 anos
Repositório principal de histórico para ML
📁 Estrutura Típica
plain
Copy
data-lake/
├── transactions/
│   └── year=2026/
│       └── month=03/
│           └── day=05/
│               └── part-000.parquet
├── fraud_labels/
│   └── year=2026/
│       └── month=03/
└── features/
    ├── user_features/
    └── card_features/
💾 Formatos Utilizados
Table
Formato	Uso
Parquet	Padrão principal
ORC	Alternativa otimizada
Delta Lake / Iceberg	Controle de versão
✅ Vantagens
Compressão eficiente
Leitura paralela
Otimizado para ML e analytics
3️⃣ Feature Engineering (Streaming + Batch)
Os dados históricos são transformados em features de ML via:
Flink Streaming
Spark
Airflow pipelines
🎯 Tipos de Features
Comportamentais
Table
Feature	Descrição
tx_count_1h	Contagem de transações na última hora
tx_count_24h	Contagem de transações nas últimas 24h
avg_transaction_amount	Valor médio das transações
Estatísticas
Table
Feature	Descrição
amount_zscore	Score Z do valor
std_transaction_amount	Desvio padrão dos valores
Geográficas
Table
Feature	Descrição
distance_last_transaction	Distância desde a última transação
country_change_rate	Taxa de mudança de país
Dispositivo
Table
Feature	Descrição
new_device_flag	Flag de novo dispositivo
device_change_frequency	Frequência de troca de dispositivo
📊 Exemplo Final
Table
card_id	tx_count_1h	avg_amount	geo_distance	fraud_label
...	...	...	...	...
4️⃣ Feature Store (Online + Offline)
plain
Copy
Feature Store (Feast/Tecton)
├── Online Store
└── Offline Store
Mantém features consistentes entre treinamento e inferência
🗄️ Offline Store
Uso: Treinar modelos
Fonte: Data Lake
Exemplo: training_dataset.parquet
⚡ Online Store
Uso: Inferência em tempo real
Fonte: Redis, Cassandra, DynamoDB
Exemplo: card_123 → tx_count_last_1h = 4
5️⃣ Labels (Dados de Verdade)
Para ML aprender fraude, precisamos de labels vindas do Feedback Loop:
plain
Copy
Analista de Risco → Console Web → Decisão
                                        │
                    ┌───────────────────┴───────────────────┐
                    ▼                                       ▼
           Confirmar Fraude                          Falso Positivo
           (fraud_confirmed = 1)                     (fraud_confirmed = 0)
📤 Eventos Gerados
fraud.confirmed
fraud.false_positive
🔄 Fluxo dos Labels
plain
Copy
Kafka → Data Lake → Dataset de Treinamento
6️⃣ Construção do Dataset de Treino
🛠️ Pipeline (Airflow / Spark Job)
sql
Copy
-- Fluxo de construção
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
📋 Dataset Final
Table
card_id	amount	tx_count_1h	geo_distance	fraud_label
...	...	...	...	...
7️⃣ Treinamento do Modelo
🧮 Frameworks Utilizados
Spark ML
XGBoost
TensorFlow
PyTorch
🔄 Pipeline de Treinamento
plain
Copy
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
📦 Exemplo de Artefato
plain
Copy
fraud_model_v14.xgboost
8️⃣ Atualização Contínua (Retraining)
O modelo não é estático - existe retraining contínuo
📅 Estratégias
Table
Tipo	Frequência	Dataset
Batch Retraining	Todo dia/semana	Últimos 90 dias de transações
Incremental Learning	Contínuo	Novos casos confirmados
📥 Eventos para Incremental Learning
fraud.confirmed
fraud.false_positive
🌊 Resumo do Fluxo Completo
plain
Copy
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
📍 Onde o ML Busca Dados em Produção
Table
Fonte	Tipo de Dado	Uso
Kafka	Eventos recentes	Streaming em tempo real
Data Lake	Histórico completo	Treinamento batch
Cassandra	Histórico operacional	Consultas rápidas
PostgreSQL	Dados mestre	Informações de referência
Neo4j	Relações de fraude	Análise de rede
Redis	Features em tempo real	Inferência online
Feedback Humano	Labels	Supervisão do modelo

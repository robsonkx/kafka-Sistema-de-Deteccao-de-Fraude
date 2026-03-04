## Tecnologias

### 1. Mensageria e Streaming

Table

| Tecnologia | Função             | Justificativa Técnica                                                                                                                       |
| :----------- | :--------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------- |
| **Apache Kafka**           | Backbone de eventos  | Throughput de 2M+ msgs/seg por cluster; persistência com replay capability; exactly-once semantics; particionamento nativo para paralelismo |
| **Schema Registry**           | Governança de dados | Evolução de schemas Avro com backward/forward compatibility; validação em runtime                                                        |
| **Kafka Streams**           | Processamento leve   | Stateful processing com rocksDB; windowed aggregations para features temporais; embedded processing (sem infra extra)                        |

**Configurações Críticas:**

- `transactions.raw`: 72 partições, RF\=3, min.insync.replicas\=2, retention\=7 dias
- `fraud.alerts`: 12 partições, RF\=3, compression\=snappy

### 2. Armazenamento em Memória (Hot Path)

Table

| Tecnologia | Uso                | Vantagens                                                                                                               |
| :----------- | :------------------- | :------------------------------------------------------------------------------------------------------------------------ |
| **Redis Cluster**           | Blacklists e cache | Sub-millisecond latency (\< 5ms); 1M+ ops/seg por nó; estruturas de dados especializadas; TTL nativo; auto-sharding |
| **RedisBloom**           | Bloom filters      | 10x mais eficiente em memória que Sets para membership testing; 1% false positive rate aceitável para pre-filtragem   |
| **RedisTimeSeries**           | Séries temporais  | Downsampling automático; aggregações por range; retenção configurável                                             |

**Estruturas de Dados Específicas:**

- `SET fraud:cards:{id}`: Cards confirmados como fraudulentos (TTL 24h renovável)
- `BF.RESERVE fraud:bloom:cards`: Pre-filtragem probabilística para economia de memória
- `GEOADD card:locations`: Indexação geospatial para detecção de velocidade impossível
- `TS.ADD user:spending`: Time-series de valores para cálculo de z-scores

### 3. Bancos de Dados Persistentes

Table

| Tecnologia | Caso de Uso             | Modelo de Consistência                                                                                                             |
| :----------- | :------------------------ | :------------------------------------------------------------------------------------------------------------------------------------ |
| **Apache Cassandra**           | Transações (180 dias) | Eventual consistency com LOCAL\_QUORUM; particionamento composto (site\_id, mês); TTL nativo; linear scalability para writes |
| **PostgreSQL + Patroni**           | Dados mestre            | ACID strict; replicação síncrona; failover automático via etcd; sharding por user\_id para escala                            |
| **Neo4j**           | Redes de fraude         | Graph nativo para detecção de ciclos (anéis de fraudadores); algoritmos de centralidade (PageRank, Betweenness)                  |


### 4. Processamento de Stream

Table

| Tecnologia | Função                          | Diferenciais                                                                                                                                     |
| :----------- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Apache Flink**           | Feature engineering em tempo real | Event time processing (lida com eventos atrasados); stateful operators com checkpointing; CEP (Complex Event Processing) para padrões de fraude |
| **Kafka Connect**           | Integração dados                | Sink connectors sem código para Cassandra/Postgres; exactly-once delivery; schema evolution                                                     |


### 5. Machine Learning Infrastructure

Table

| Componente | Tecnologia                         | Propósito                                                                                                |
| :----------- | :----------------------------------- | :---------------------------------------------------------------------------------------------------------- |
| **Feature Store**           | Feast / Tecton                     | Consistência entre treino (batch) e serving (online); feature versioning; compartilhamento entre equipes |
| **Model Serving**           | TensorFlow Serving / NVIDIA Triton | Inference de baixa latência (\< 100ms); batching dinâmico; model versioning e A/B testing            |
| **Training Pipeline**           | Kubeflow Pipelines / Airflow       | Orquestração de ETL, treinamento, validação, deploy; reproducibilidade; paralelismo                   |
| **Experiment Tracking**           | MLflow                             | Versionamento de modelos, parâmetros, métricas; model registry com stages (Staging/Production/Archived) |
| **Monitoring**           | Evidently AI / WhyLabs             | Data drift detection; feature importance tracking; performance degradation alerts                         |


## 4. Estratégia de Machine Learning

### 4.1 Arquitetura de Dados para ML

**Camadas do Data Lake:**

- **Bronze (Raw):**  Logs JSON de Kafka, eventos de auditoria, dumps de bancos. Retenção: 7 anos. Formato: JSON/Avro.
- **Silver (Cleaned):**  Dados deduplicados, validados (Great Expectations), enriquecidos com referências. Particionado por data. Formato: Parquet.
- **Gold (Features):**  Feature vectors prontos para treinamento, com labels confirmados. Balanceados (SMOTE para oversampling de fraudes). Formato: Parquet + Delta Lake.

- 

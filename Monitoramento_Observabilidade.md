## Monitoramento e Observabilidade

### 1. Pilares da Observabilidade

Table

| Pilar | Ferramenta                                  | Métricas Principais                                                                     |
| :------ | :-------------------------------------------- | :----------------------------------------------------------------------------------------- |
| **Métricas**      | Prometheus + Grafana                        | Latência percentiles, throughput, taxas de erro, cache hit ratio, ML accuracy           |
| **Logs**      | ELK Stack (Elasticsearch, Logstash, Kibana) | Logs estruturados JSON; correlação por trace\_id; retention 30 dias hot, 1 ano warm |
| **Traces**      | Jaeger (OpenTelemetry)                      | Distributed tracing end-to-end; análise de caminho crítico; detecção de bottlenecks  |
| **Alertas**      | PagerDuty + Opsgenie                        | Escalonamento por severidade; runbooks automatizados; post-mortem tracking               |

### 2. SLIs e SLOs (Service Level Indicators/Objectives)

Table

| SLI                           | SLO                        | Janela de Medição |
| :------------------------------ | :--------------------------- | :-------------------- |
| Latência Hot Path (P50)      | \< 100ms                | 1 minuto            |
| Latência Hot Path (P99)      | \< 500ms                | 1 minuto            |
| Disponibilidade Hot Path      | 99.99%                     | 30 dias             |
| Throughput                    | Suportar 10k TPS sustained | Contínuo           |
| Taxa de Falsos Positivos      | \< 1%                   | 7 dias              |
| Precisão do Modelo (AUC-ROC) | \> 0.90                 | 7 dias              |
| Tempo de Detecção ML (P90)  | \< 30 minutos           | 1 hora              |

### 3. Dashboards e Alertas

**Dashboards Grafana:**

1. **Executive:**  SLO compliance, valor de fraude evitado, taxa de bloqueios
2. **Operacional:**  Latência em tempo real, throughput por componente, filas (Kafka lag)
3. **ML Performance:**  Feature drift, prediction distribution, confusion matrix, ROC curve
4. **Infraestrutura:**  CPU/memória por pod, disk I/O, network throughput, GC pauses

### 4. Distributed Tracing

Cada transação propaga `trace_id` e `span_id` via headers HTTP (W3C Trace Context):


```plain
Trace: tx_abc123 (root)
├── span: api_gateway (12ms)
│   └── tags: http.method=POST, http.route=/transactions
├── span: kafka_produce (2ms)
│   └── tags: kafka.topic=transactions.raw, kafka.partition=42
├── span: fraud_check_service (45ms)
│   ├── span: redis_lookup (2ms)
│   │   ├── span: sis_member_card (0.5ms)
│   │   ├── span: sis_member_user (0.5ms)
│   │   └── span: geopos_check (1ms)
│   └── span: response_serialization (1ms)
├── span: kafka_produce_validated (2ms)
└── span: ml_pipeline_async (8min) [span separado, mesmo trace]
    ├── span: feature_extraction (2min)
    ├── span: model_inference (100ms)
    └── span: alert_generation (1min)
```

**Benefícios:**

- Identificação precisa de gargalos (ex: Redis lento em específico shard)
- Correlação de erros (ex: falha no ML traceada até feature específica)
- Análise de performance por caminho (hot path vs warm path)

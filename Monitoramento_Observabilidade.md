# Monitoramento e Observabilidade

Ferramentas:

Prometheus
Grafana
OpenTelemetry
ELK Stack
Jaeger


## Métricas de Infraestrutura

- latência da API
- uso de CPU
- uso de memória
- consumer lag do Kafka
- latência Redis
- throughput do sistema

---

## Métricas de Negócio

- taxa de fraude detectada
- taxa de falsos positivos
- transações por segundo (TPS)
- tempo médio de decisão
- volume de alertas de fraude


## Observabilidade de Transações

Cada transação possui **trace distribuído**.

Fluxo observado:

POS
 → API Gateway
 → Kafka
 → Fraud Service
 → Redis
 → Persistência


Ferramenta:

OpenTelemetry + Jaeger

Isso permite identificar:

- gargalos
- latência por serviço
- falhas em pipelines

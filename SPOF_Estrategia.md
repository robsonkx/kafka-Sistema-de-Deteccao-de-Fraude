## Eliminação de SPOF (Single Point of Failure)

### 1. Estratégias por Componente

Table

| Componente | Topologia                                                      | Mecanismo de Failover                                                                  | RTO          | RPO                              |
| :----------- | :--------------------------------------------------------------- | :--------------------------------------------------------------------------------------- | :------------- | :--------------------------------- |
| **API Gateway**           | 3+ instâncias em 3 AZs                                        | Load balancer health checks (10s); auto-remoção de nó unhealthy                     | 10s          | 0                                |
| **Fraud Check Service**           | Stateless, min 3 pods, HPA (Horizontal Pod Autoscaler)         | Kubernetes self-healing; readiness probes                                              | 30s          | 0                                |
| **Kafka**           | 6 brokers (3 por região), RF\=3, min.insync.replicas\=2 | Controller failover automático (KRaft mode); unclean.leader.election.enable\=false | 5s           | 0                                |
| **Redis**           | 6 nós (3 masters + 3 replicas), Cluster mode                  | Redis Cluster auto-failover; replica promotion em \< 5s                             | 5s           | \< 1s (replica async)         |
| **Cassandra**           | 9 nós (3 por AZ), RF\=3, LOCAL\_QUORUM                  | Gossip protocol; hinted handoff; read repair                                           | 0 (seamless) | 0                                |
| **PostgreSQL**           | Patroni + etcd (3 nós); 1 leader + 2 sync standbys            | Failover automático em \< 30s; virtual IP com keepalived                           | 30s          | 0 (sync replication)             |
| **Flink**           | Checkpointing para S3; savepoints automáticos                 | Restart from last checkpoint; stateful recovery                                        | 60s          | \< 1min (checkpoint interval) |

### 2. Detalhamento de Alta Disponibilidade

**Kafka Multi-Região:**

- MirrorMaker 2 replica tópicos críticos entre Região A e B (bidirecional)
- Consumer groups configurados para `isolation.level=read_committed`
- Producers com `acks=all` e `retries=Integer.MAX_VALUE`

**Redis Cluster:**

- Hash slots 0-16384 distribuídos entre 3 masters
- Cada master tem 1 replica em AZ diferente
- Client library (Redis-py-cluster) com failover automático e redirecionamento de slots

**PostgreSQL Patroni:**

- DCS (Distributed Configuration Store): etcd cluster de 3 nós
- Synchronous replication para 2 standbys garante RPO\=0
- Failover automático com STONITH (Shoot The Other Node In The Head) para evitar split-brain

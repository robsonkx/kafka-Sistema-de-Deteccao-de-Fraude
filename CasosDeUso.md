### Caso de Uso ###

### Caso de Uso 1: Transação Legítima (Happy Path)

**Atores:**  Cliente, POS/Website, Sistema de Fraude, Banco Emissor

**Fluxo Principal:**

1. Cliente realiza compra de R\$ 150,00 em supermercado às 14h30 (horário habitual)
2. Transação chega via API Gateway com autenticação mTLS e rate limiting
3. Evento publicado no tópico `transactions.raw` (Kafka) com partition key \= `card_id`
4. **Fraud Check Service** (escrito em Go, 3 réplicas) consome o evento
5. Consultas paralelas ao Redis Cluster:

   - `SISMEMBER fraud:cards:{card_id}` → 0 (não está na blacklist)
   - `SISMEMBER fraud:users:{user_id}` → 0
   - `SISMEMBER fraud:sites:{site_id}` → 0
   - `GEODIST card:locations {last_tx} {current_tx}` → 2km (razoável)
6. Nenhum match em blacklist → Resposta HTTP 200 em 45ms (P50)
7. Transação publicada em `transactions.validated` para processamento assíncrono
8. ML Pipeline (Flink) processa em background:

   - Calcula features de velocidade, padrão horário, valor relativo ao histórico
   - Score do modelo: 0.12 (baixo risco)
   - Nenhuma ação adicional necessária

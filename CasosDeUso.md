### Casos de Uso ###

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


### Caso de Uso 2: Bloqueio por Entidade Fraudulenta

**Atores:**  Cliente (comprometido), Sistema de Fraude, Analista de Risco

**Fluxo Principal:**

1. Tentativa de transação de R\$ 2.000,00 em site de eletrônicos às 3h da manhã
2. **Fraud Check Service** consulta Redis:

   - `SISMEMBER fraud:cards:1234-5678` → 1 (CARTÃO NA BLACKLIST!)
3. Resposta imediata HTTP 403 em 12ms
4. Evento publicado em `fraud.alerts` com severidade HIGH
5. Ações paralelas disparadas:

   - Notificação SMS para titular do cartão: "Tentativa de uso bloqueada. Cartão suspenso."
   - Email para equipe de segurança com detalhes da tentativa
   - Atualização de métricas em tempo real (Grafana)
   - Log de auditoria em ELK Stack

**Fluxo Alternativo (Confirmação de Fraude):**

1. Analista revisa o caso no Console Web
2. Confirma que tentativa era de fato fraudulenta
3. Sistema executa:

   - Atualiza TTL da entrada no Redis (renova por mais 24h)
   - Publica em `fraud.confirmed` para atualização do modelo ML
   - Cria nó no Neo4j vinculando cartão a rede de fraudadores conhecidos
   - Dispara trigger para re-treinamento incremental se necessário

### Caso de Uso 3: Detecção de Comportamento Suspeito (ML)

**Atores:**  Sistema ML, Analista de Risco, Cliente (vítima potencial)

**Fluxo de Detecção:**

1. Transação de R\$ 5.000,00 em loja de viagens online
2. Passa na blacklist (entidades não conhecidas)
3. Aprovada no hot path, mas marcada para análise ML
4. Flink Streaming calcula features em janela de 2 horas:

   - `tx_count_1h`: 5 transações (usual: 1)
   - `amount_zscore`: 4.5 (4.5 desvios acima da média)
   - `distinct_countries_24h`: 3 países diferentes
   - `time_anomaly`: Transação às 3h (fora do padrão do usuário)
5. Modelo XGBoost retorna score: **0.92** (threshold: 0.8)

**Ações Baseadas no Score:**

- **Score**  **>=**  **0.9:**  Congelamento preventivo da transação (se protocolo permitir), notificação urgente ao analista via PagerDuty, inclusão em fila de revisão prioritária
- **0.8**  **<=**  **Score**  **<**  **0.9:**  Transação aprovada mas entidades adicionadas a "watch list" temporária no Redis (TTL 4h), monitoramento intensificado

**Feedback Loop:**

- Analista confirma fraude → Atualiza Redis (blacklist permanente), alimenta dataset de treino positivo
- Analista nega fraude (falso positivo) → Remove da watch list, alimenta dataset negativo, ajusta threshold local para usuário


### Caso de Uso 4: Falso Positivo e Correção

**Contexto:**  Usuário legítimo em viagem internacional realiza compra atípica

**Fluxo:**

1. Transação em Paris (usuário normalmente em São Paulo)
2. ML score: 0.85 (médio-alto) devido à mudança súbita de localização
3. Transação aprovada (score \< 0.9), mas usuário adicionado a watch list
4. Próximas 3 transações também geram scores elevados
5. Usuário recebe notificação: "Detectamos uso atípico. Confirma viagem?"
6. Usuário confirma via app → Sistema:

   - Remove entradas da watch list
   - Atualiza feature `user_trusted` no Redis
   - Publica evento `fraud.false_positive`
   - Ajusta pesos do modelo para esse perfil de usuário (importance sampling)
   - Atualiza padrão de localização aceitável

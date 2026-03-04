## Estratégia de Machine Learning

### 1. Arquitetura de Dados para ML

**Camadas do Data Lake:**

- **Bronze (Raw):**  Logs JSON de Kafka, eventos de auditoria, dumps de bancos. Retenção: 7 anos. Formato: JSON/Avro.
- **Silver (Cleaned):**  Dados deduplicados, validados (Great Expectations), enriquecidos com referências. Particionado por data. Formato: Parquet.
- **Gold (Features):**  Feature vectors prontos para treinamento, com labels confirmados. Balanceados (SMOTE para oversampling de fraudes). Formato: Parquet + Delta Lake.

- ### 2. Pipeline de Treinamento

**Fluxo Semanal:**

1. **Extract:**  Spark job lê Gold layer dos últimos 6 meses (billions de rows)
2. **Validate:**  Great Expectations verifica schema, nulls, distribuições
3. **Transform:**  Feature Store materializa features online/offline; aplica SMOTE para balanceamento (fraud:legítima \= 1:5)
4. **Train:**  XGBoost/LightGBM com hyperparameter tuning (Optuna); cross-validation temporal (evita data leakage)
5. **Evaluate:**  Métricas em holdout set temporal (últimos 7 dias): AUC-ROC, Precision@Recall\=0.8, Calibration curve
6. **Validate:**  Testes de stress (latência \< 100ms p99), testes de adversarial robustness
7. **Deploy:**  Modelo versionado no MLflow; promoted para "Staging"; shadow mode por 24h; A/B test (10% traffic); full rollout

**Treinamento Incremental:**

- Modelo base retreinado semanalmente
- Online learning: SGD com learning rate decay para ajustes diários baseados em feedback de analistas
- Trigger automático de re-treinamento se:

  - Data drift \> 0.1 (KS test em features principais)
  - Performance drop \> 5% em janela de 24h
  - Novo padrão de fraude detectado (alerta de analistas)

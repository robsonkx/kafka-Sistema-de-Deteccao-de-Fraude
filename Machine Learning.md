A arquitetura é baseada em processamento de eventos em tempo real, com separação entre três caminhos principais de processamento:

Hot Path → decisão imediata (<1s)

Warm Path → processamento analítico e features (minutos)

Cold Path → armazenamento histórico e treinamento de ML

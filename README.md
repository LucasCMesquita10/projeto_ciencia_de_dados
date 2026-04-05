*Cyber Security Data Pipeline (Medallion Architecture)*
Este projeto implementa um pipeline de engenharia de dados focado em incidentes de segurança cibernética. O objetivo é transformar dados brutos em um dataset unificado, limpo e protegido contra Data Leakage, pronto para análises preditivas de impacto financeiro.

*Como Executar*
Clone o repositório.

Certifique-se de ter as bibliotecas instaladas: pandas, pyarrow, kagglehub, seaborn.

Execute o notebook principal. O script baixará automaticamente os dados do Kaggle e processará as camadas.

 Arquitetura do Projeto
O pipeline segue a Arquitetura Medalhão, garantindo a trilha de auditoria e a qualidade dos dados:

Raw (CSV): Dados brutos baixados diretamente da API do Kaggle.

Bronze (Parquet): Dados convertidos para formato colunar, enriquecidos com metadados (hash de integridade e timestamp de ingestão).

Silver (Parquet): Camada de limpeza e cruzamento. Aqui os dados de incidentes e custos são unidos, tipos são corrigidos e variáveis de "futuro" são removidas.

*Linhagem de Dados (Data Lineage)*
O fluxo de dados deste pipeline foi projetado para garantir a rastreabilidade total, desde a captura na fonte até a entrega do dataset final. O caminho percorrido pelos dados é dividido em três estágios lógicos:

Ingestão e Persistência (Raw):
O ciclo começa com a extração automatizada via API (kagglehub). Os arquivos originais em formato CSV são salvos na pasta data/raw. Esta camada é imutável: ela serve como nosso "ponto de verdade" e backup de segurança, garantindo que possamos reconstruir todo o ecossistema mesmo sem acesso à internet.

Padronização e Governança (Bronze):
Nesta etapa, os arquivos CSV são lidos e convertidos para o formato Parquet. Além da compressão de dados, cada linha recebe uma "impressão digital" (meta_row_hash) e um selo de tempo (meta_ingestion_at). Isso permite que saibamos exatamente quando cada informação entrou no sistema e garante que o dado não foi alterado indevidamente.

Refino e Inteligência de Negócio (Silver):
Aqui ocorre a "mágica" da engenharia. Cruzamos as tabelas de incidentes com as de impactos financeiros através de um merge por ID. Aplicamos regras de saneamento para tratar valores nulos e, o mais importante, realizamos a filtragem seletiva: removemos variáveis de data leakage (dados que só existiriam após o fato ocorrido). O resultado é o arquivo cyber_security_silver.parquet, um dataset único, otimizado e pronto para alimentar modelos de Machine Learning ou dashboards de decisão.

*Transformações Principais (Camada Prata)*
Diferente de uma simples limpeza, a nossa Camada Prata aplicou regras de negócio estratégicas:

Unificação (Merge): Cruzamento entre a base de incidentes e a base financeira via ID.

Tratamento de Nulos: Imputação de valor 0 para campos financeiros vazios (evitando erros de soma) e preservação de nulos em categorias para evitar viés.

Estratégia Anti-Leakage: Remoção de colunas como insurance_payout e notas de peritos. Essas informações só existem após o fechamento do incidente, portanto, não podem ser usadas para treinar um modelo que pretende prever o dano no primeiro dia.
# Cyber Security Data Pipeline (Medallion Architecture)

Este projeto implementa um pipeline de engenharia de dados focado em incidentes de segurança cibernética. O objetivo é transformar dados brutos em um dataset unificado, limpo e protegido contra *Data Leakage* (Vazamento de Dados), pronto para análises preditivas de impacto financeiro.

## 🚀 Passo a Passo para Execução

1. **Clone o repositório** para a sua máquina local.
2. **Instale as dependências:** certifique-se de ter as bibliotecas instaladas no seu ambiente virtual: `pandas`, `pyarrow`, `kagglehub` e `seaborn`.
3. **Execute o notebook principal:** O script baixará automaticamente os dados da API do Kaggle e executará o processamento completo.

## 🔄 Passo a Passo do Pipeline (Linhagem de Dados)

O fluxo de dados deste pipeline foi projetado para garantir a rastreabilidade total, seguindo a **Arquitetura Medallion**. O caminho percorrido pelos dados acontece em 3 passos:

*   **Passo 1: Ingestão e Persistência (Camada Raw)**
    A extração é feita de forma automatizada via API (`kagglehub`). Os arquivos originais em formato CSV são salvos na pasta `data/raw`. Esta camada é o nosso backup de segurança, garantindo que possamos reconstruir todo o ecossistema localmente a qualquer momento.

*   **Passo 2: Padronização e Rastreabilidade (Camada Bronze)**
    Os arquivos CSV são lidos e convertidos para o formato *Parquet*. Cada linha recebe uma "impressão digital" (hash gerado na coluna `meta_row_hash`) e um selo de tempo (`meta_ingestion_at`). Isso garante a rastreabilidade exata de quando a informação entrou no sistema e atesta a integridade do dado.

*   **Passo 3: Refino e Regras de Negócio (Camada Silver)**
    Cruzamos as tabelas de incidentes com as de impactos financeiros através de um `merge` por ID. Aplicamos regras de saneamento para tratar valores nulos e realizamos a filtragem de *Data Leakage*. O resultado é o arquivo `cyber_security_silver.parquet`, um dataset íntegro e preparado para o Machine Learning.

## 🛡️ Transformações Principais na Camada Prata

Diferente de uma simples limpeza de dados, aplicamos regras estratégicas para garantir a resiliência do modelo:

*   **Unificação:** Cruzamento sequencial entre as bases utilizando a chave primária do incidente.
*   **Tratamento de Nulos:** Imputação do valor `0` para campos financeiros específicos (indicando que não houve pagamento de resgate ou multa), preservando nulos em outras categorias para não enviesar o modelo com preenchimentos artificiais.
*   **Estratégia Anti-Leakage (Prevenção de Vazamento de Dados):** Remoção de colunas que representam o "futuro" (ex: horas fora do ar, variação de ações após 30 dias). Essas informações só existem após a conclusão do incidente, portanto, foram removidas para não viciar o modelo preditivo.
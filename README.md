# 📊 Previsão de Estoque com Amazon SageMaker Canvas

Este projeto tem como objetivo analisar uma tabela de estoque utilizando o **Amazon SageMaker Canvas**, uma ferramenta da AWS que permite a analistas de negócios criar modelos de Machine Learning (ML) e gerar previsões usando uma interface visual, sem precisar escrever código (*no-code/low-code*).

A ideia central é **extrair insights importantes para a predição de dados futuros**. A partir dessa análise, podemos tomar decisões que impactam diretamente o estoque, entendendo melhor a relação entre **preço, promoções, vendas e quantidade em estoque**.

---

## 🧠 Visão Geral do Projeto

- Problema: prever a **QUANTIDADE_ESTOQUE** de produtos com base em histórico de vendas, preço e promoções.  
- Ferramenta: **Amazon SageMaker Canvas**.  
- Abordagem: criação de um modelo de **time series forecasting** (série temporal) usando apenas interface visual.  
- Objetivo de negócio:
  - Melhorar a assertividade na reposição de estoque;
  - Reduzir rupturas (falta de produto);
  - Evitar excesso de estoque parado.

---

## 🗂️ Dataset Utilizado

Para este projeto, foi utilizado o arquivo:

- `dataset-1000-com-preco-promocional-e-renovacao-estoque.csv`

O fluxo foi:

1. Upload do arquivo CSV no portal do **SageMaker Canvas**;
2. Importação automática do dataset com o nome padrão  
   **“New dataset 2025-12-29 10:07:25 PM”**;
3. Renomeamos esse dataset para **“Historico-de-estoque”** para facilitar o entendimento e a rastreabilidade dentro do projeto.

A tela de seleção de dataset no Canvas ficou semelhante a:

[SelectDataset.JPG](https://raw.githubusercontent.com/raylanbf/lab-aws-sagemaker-canvas-estoque/refs/heads/main/src/SelectDataset.JPG)

Exemplo de colunas do dataset:

- `ID_PRODUTO` – identificador único de cada item;
- `QUANTIDADE_ESTOQUE` – quantidade em estoque (variável alvo);
- `PRECO` – preço do produto;
- `FLAG_PROMOCAO` – indicador binário de promoção (0/1);
- `DATA_EVENTO` – data do registro.

---

## 🧹 Preparação e Exploração dos Dados

Após a importação, os dados foram explorados na própria interface do Canvas, que permite visualizar:

- Distribuição de valores por coluna;
- Tipos de dados;
- Quantidade de valores ausentes;
- Frequência de cada categoria.

Durante essa etapa, também é possível ajustar tipos de dados (por exemplo, tratar `ID_PRODUTO` como texto/categórico).

A tela de visualização e tratamento das colunas aparece como:

[Tabelas.JPG](https://raw.githubusercontent.com/raylanbf/lab-aws-sagemaker-canvas-estoque/refs/heads/main/src/Tabelas.JPG)

Principais ações realizadas:

- Confirmação de que `ID_PRODUTO` seria utilizado como **identificador do item**;
- Verificação da distribuição de `QUANTIDADE_ESTOQUE`, `PRECO` e `FLAG_PROMOCAO`;
- Garantia de que `DATA_EVENTO` foi reconhecida como campo de data, necessário para o modelo de série temporal.

---

## 🏗️ Criação e Configuração do Modelo

Depois de preparar os dados, foi criado um modelo no Canvas:

- Tipo de modelo escolhido: **Predictive analysis**;
- Nome inicial gerado automaticamente:  
  **“New model 2025-12-29 10:11:21 PM”**;
- Nome final adotado: **“Modelo-de-Predicao”**.

Em seguida:

1. Selecionamos o dataset **“Historico-de-estoque”**;
2. Definimos a coluna **`QUANTIDADE_ESTOQUE`** como **target** (variável de predição);
3. O Canvas recomendou o tipo de modelo **Time series forecasting**, utilizando o histórico da série temporal para prever valores futuros.

A tela de configuração do modelo e seleção da coluna alvo ficou assim:

[PredicaoDeEstoque.JPG](https://raw.githubusercontent.com/raylanbf/lab-aws-sagemaker-canvas-estoque/refs/heads/main/src/PredicaoDeEstoque.JPG)

Nessa visão é possível observar:

- A coluna `QUANTIDADE_ESTOQUE` marcada como **Target**;
- As demais colunas (`PRECO`, `ID_PRODUTO`, `FLAG_PROMOCAO`, `DATA_EVENTO`) configuradas como **features**;
- O tipo de modelo definido como **Time series forecasting**.

---

## ⚙️ Treinamento do Modelo

Ao acessar a opção **“Configure model”**, foram realizadas as seguintes configurações:

- Definição de `ID_PRODUTO` como identificador único dos itens;
- Salvamento das configurações do modelo de série temporal.

Com tudo configurado, foi iniciado o treinamento usando a opção **Quick Build**.

### Por que Quick Build?

- **Tempo estimado:** entre **14 e 20 minutos**;
- Objetivo principal: experimentar a plataforma e compreender o fluxo completo de criação do modelo;
- Comparação:
  - **Quick Build:** mais rápido, ideal para experimentação;
  - **Standard Build:** mais demorado (2 a 4 horas), porém tende a gerar modelos ainda mais precisos.

---

## 📊 Resultados do Modelo

Após aproximadamente 20 minutos, o Quick Build foi concluído e o modelo apresentou as seguintes métricas:

[resultado.JPG](https://raw.githubusercontent.com/raylanbf/lab-aws-sagemaker-canvas-estoque/refs/heads/main/src/resultado.JPG)

Métricas principais:

- **avg. wQL:** `0.060`  
  - Mede o erro médio do modelo considerando diferentes quantis (cenários otimista, médio e pessimista).  
  - Quanto mais próximo de **0**, melhor.  
  - **Interpretação:** `0.060` é um valor muito bom, indicando **boa precisão geral** do modelo.

- **MAPE:** `0.148`  
  - Erro percentual médio das previsões.  
  - `0.148` = **14,8% de erro médio**.  
  - Interpretação prática:
    - Em média, a previsão erra cerca de **15% para mais ou para menos**.
    - Para previsão de estoque, esse valor é de **aceitável a bom**, dependendo da criticidade dos itens.

- **WAPE:** `0.100`  
  - Similar ao MAPE, mas pondera o erro pelos itens de maior volume.  
  - `0.100` = **10% de erro ponderado**.  
  - **Insight:** para os produtos mais relevantes (maior giro), o modelo está ainda **mais preciso**.

- **RMSE:** `5.765`  
  - Mede o erro médio em unidades reais da variável alvo.  
  - Interpretação prática:
    - O modelo erra, em média, cerca de **6 unidades de estoque**.  
    - Se os estoques trabalham com dezenas ou centenas de unidades, esse erro é relativamente **baixo**.

- **MASE:** `0.301`  
  - Compara o modelo com um modelo ingênuo (baseline) que, por exemplo, repetiria o último valor observado.  
  - Valores:
    - `< 1` → melhor que o modelo simples;
    - `= 1` → mesmo desempenho;
    - `> 1` → pior.  
  - **Interpretação:** `0.301` é **excelente**, mostrando que o modelo é **muito melhor** do que um “chute” baseado apenas no histórico imediato.

### Impacto das Colunas (Feature Importance)

Na aba de análise, o Canvas também mostra o impacto de cada coluna nas previsões. No caso deste modelo:

- **PRECO** aparece como a variável de maior impacto na previsão de estoque;
- **FLAG_PROMOCAO** também é considerada, mas com impacto menor.

Essa visão ajuda a entender a relação entre **preço, promoções e comportamento de estoque**, reforçando a importância de políticas comerciais bem definidas.

---

## 🔮 Predição de Estoque por Item

Com o modelo treinado, utilizamos a aba **Predict** para gerar previsões futuras, tanto em lote quanto individualmente.

Um dos testes realizados foi a previsão para o produto **ID 1017**, onde analisamos os diferentes quantis de previsão (P10, P50, P90) ao longo do tempo:

[Prod_1017.JPG](https://raw.githubusercontent.com/raylanbf/lab-aws-sagemaker-canvas-estoque/refs/heads/main/src/Prod_1017.JPG)

Nessa tela é possível observar:

- **Historical Demand** – linha com o histórico real do item;
- **P10** – cenário mais conservador (demanda menor);
- **P50** – cenário esperado (mediano);
- **P90** – cenário mais agressivo (maior demanda).

Essa visualização permite:

- Planejar o estoque considerando diferentes níveis de risco;
- Ajustar políticas de reposição com base em cenários conservador, médio e otimista;
- Identificar tendências de aumento ou queda de demanda ao longo do horizonte de previsão.

---

## 🧩 Insights Extraídos

Com base nas análises e métricas:

- O modelo apresenta **boa precisão global**, com **erro médio percentual em torno de 15%** e erro ponderado de **10%** para itens de maior volume;
- O **preço** é a principal variável de impacto, indicando uma **relação direta entre estratégia de precificação e nível de estoque**;
- A **promoção** (`FLAG_PROMOCAO`) também influencia, mas em menor grau, o que pode indicar:
  - Promoções pontuais;
  - Ou necessidade de mais dados históricos para capturar melhor esse efeito;
- O **MASE muito abaixo de 1** reforça que o modelo de Canvas é significativamente melhor do que simplesmente repetir o último valor de estoque.

Esses resultados permitem:

- Definir níveis mais realistas de **estoque mínimo** e **estoque de segurança**;
- Antecipar necessidade de reposição para itens com tendência de aumento de demanda;
- Reduzir custos com excesso de estoque parado, ao alinhar melhor previsões com decisões de compra.

---

## 🧪 Como Reproduzir o Projeto

1. Acessar o console da **AWS** e abrir o **Amazon SageMaker Canvas**;
2. Fazer upload do CSV `dataset-1000-com-preco-promocional-e-renovacao-estoque.csv`;
3. Renomear o dataset para **“Historico-de-estoque”**;
4. Criar um novo modelo e renomeá-lo para **“Modelo-de-Predicao”**;
5. Definir:
   - Target: `QUANTIDADE_ESTOQUE`;
   - Identificador de item: `ID_PRODUTO`;
   - Campo de data: `DATA_EVENTO`;
6. Escolher o tipo de modelo sugerido (**Time series forecasting**);
7. Rodar um **Quick Build**;
8. Analisar:
   - Métricas (avg.wQL, MAPE, WAPE, RMSE, MASE);
   - Importância das colunas;
   - Previsão por produto (ex.: item 1017).

---

## 🚀 Conclusão

Este projeto demonstra, de forma prática, como é possível:

- Utilizar o **Amazon SageMaker Canvas** para criar modelos de previsão de estoque **sem escrever código**;
- Obter métricas sólidas de desempenho, adequadas para apoiar decisões de negócio;
- Visualizar a influência de **preço e promoções** sobre o comportamento de estoque;
- Gerar previsões por produto e por cenário, apoiando um planejamento de estoque mais **estratégico e orientado a dados**.

A partir deste ponto, é possível evoluir o projeto:

- Testando o **Standard Build** para tentar melhorar ainda mais a precisão;
- Enriquecendo o dataset com novas variáveis (sazonalidade, calendário promocional, feriados, etc.);
- Integrando as previsões a dashboards ou sistemas de gestão de estoque.

---

## 👨‍💻 Autor

- **Raylan Bruno Fraga**  
- GitHub: [raylanbf](https://github.com/raylanbf)
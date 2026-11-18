# HorrorMoviesData

Projeto de **Data Analytics / Projeto Baseado em Problemas** (PUCPR, Sistemas de Informação, 5º período, 2º semestre de 2026).
Professora: Marina de Lara.

**Equipe:** Carla Ferraz de Araujo, Gabriella Tavares Pedroso, Luiz Gustavo Santarosa e Mateus Lima da Cruz.

O projeto trabalha com filmes de terror do The Movie Database (TMDB). O RA1 prepara a base de dados; o RA2 usa essa base preparada para demonstrar processamento de dados em streaming; o RA3 usa a mesma base para estatística descritiva, visualizações e um modelo de machine learning.

## Arquivos do repositório

| Arquivo | Etapa | Descrição |
|---|---|---|
| `horror_movies.csv` | RA1 | Dados brutos: 32.540 filmes × 21 atributos |
| `HorrorMovieAnalysis.ipynb` | RA1 e RA3 | Ingestão, diagnóstico de qualidade (Q1 a Q8), limpeza (L1 a L11), testes, exportação, respostas às perguntas P1 a P6 e, na seção final, a estatística descritiva do RA3 (Parte 1) |
| `horror_movies_limpo.csv` | RA1 | Base preparada: 32.404 filmes × 25 atributos (saída do RA1 e entrada do RA2 e do RA3) |
| `HorrorMovieNight.ipynb` | RA2 | Streaming com janela deslizante sobre vendas simuladas de ingressos na noite de Halloween |
| `RA3_2_visualizacoes.ipynb` | RA3 | Visualizações interpretadas de P1 a P6 (Parte 2) e o modelo de machine learning (Parte 3) |
| `requirements.txt` | RA1, RA2 e RA3 | Dependências do projeto |

## Ambiente

- Python 3.14.8 no ambiente local (kernel registrado no notebook do RA2). No Google Colab, a versão do Python é gerenciada pela plataforma.
- Bibliotecas: pandas, numpy, matplotlib, seaborn (RA1 e RA3), scikit-learn (RA3) e pyspark (RA2). As versões estão em `requirements.txt`.
- O RA2 também precisa de **Java** instalado (o Colab já traz). O notebook instala o `pyspark` sozinho se ele não existir.
- Conexão com a internet: os notebooks leem os CSVs pela URL raw deste repositório (branch `main`) quando o arquivo local não existe.

Para rodar localmente: pip install -r requirements.txt.

---

## RA1 — Ingestão e preparação dos dados

**Fonte:** *Horror Movies*, extraída do TMDB por Tanya Shapiro em outubro de 2022, publicada no TidyTuesday (semana 44 de 2022) e republicada no Kaggle por Evan Gower (licença CC0). Origem do código: https://github.com/tashapiro/horrormovies. Dados em https://www.kaggle.com/datasets/evangower/horror-movies.

**O que o notebook faz:**
1. Carrega `horror_movies.csv` (32.540 × 21) e o preserva em `df_original`, sem alterações.
2. Avalia a qualidade em oito verificações (Q1 a Q8): unicidade, ausências, zeros que significam "não informado", tipos, domínios e período, coerência entre campos, categorias e valores extremos.
3. Aplica a limpeza em onze decisões (L1 a L11) sobre uma cópia, com testes (`assert`) e uma tabela antes × depois.
4. Exporta `horror_movies_limpo.csv` (32.404 × 25).
5. Responde às seis perguntas analíticas (P1 a P6) com tabelas, gráficos e uma síntese final, usando a base limpa.

**Como reproduzir:**
1. Abrir `HorrorMovieAnalysis.ipynb` no Google Colab (ou em Jupyter local).
2. Executar todas as células em ordem. A célula de ambiente deve imprimir `✓ Ambiente preparado.` e a de ingestão `✓ Dados carregados com sucesso!` com shape 32540 × 21.
3. As células de teste devem imprimir `Todas as verificações passaram.`
4. A última etapa grava `horror_movies_limpo.csv` com 32.404 linhas × 25 colunas.
5. A seção "Respostas às perguntas" mostra as tabelas e gráficos de P1 a P6 e termina com a síntese das respostas.

**Limitações conhecidas:** 87,0% dos filmes sem orçamento e 95,4% sem receita válidos (a análise de retorno fica restrita a 1.010 filmes); 35,5% sem nota; 10 filmes marcados como lançados com data posterior à coleta; `popularity` reflete o momento da coleta (outubro de 2022). O detalhamento está na seção "Limitações e pendências" do documento do RA1.

---

## RA2 — Streaming com janela deslizante

**Objetivo:** aplicar Structured Streaming (Apache Spark) em uma situação em que os registros chegam aos poucos. Como a base do RA1 é estática, o notebook usa **dados simulados**: 8 vendas de ingressos de um cinema na noite de 31/10/2022, com horários e valores inventados. Os quatro filmes (*Smile*, *Halloween Ends*, *Terrifier 2* e *Barbarian*) existem na base limpa do RA1, e o notebook confere os IDs deles lendo `horror_movies_limpo.csv`.

**Janela temporal:** Sliding Window de 5 minutos com deslocamento de 2 minutos, somando `valor` para obter a receita do cinema em cada janela. O início da janela é incluído e o fim é excluído.

**O que o notebook faz:**
1. Define as vendas simuladas em três etapas (como se chegassem aos poucos) e calcula **à mão** o resultado esperado de cada etapa (gabarito).
2. Roda a agregação em um DataFrame comum e confere com o gabarito por `assert`.
3. Liga o Structured Streaming: cada etapa vira um arquivo JSON publicado numa pasta de entrada monitorada, e a consulta atualiza a tabela de resultados (modo `complete`, saída em memória, com checkpoint).
4. Confere o resultado de cada etapa do streaming com o mesmo gabarito e mostra as janelas novas e as alteradas.
5. Identifica o pico da noite (janela 20:06–20:11, R$ 240, puxada por *Barbarian*) e apaga as pastas temporárias ao final.

**Como reproduzir:**
1. Abrir `HorrorMovieNight.ipynb` no Google Colab (ou Jupyter local com Java e `pip install -r requirements.txt`).
2. Executar todas as células em ordem. Cada etapa deve imprimir `confere com o gabarito`.
3. Ao final, a célula de encerramento deve mostrar `Consulta ativa: False` e `Pastas temporárias existem: False`.

**Limitações:** os registros são simulados; o resultado fica em tabela em memória (adequada só para demonstração, não para produção); a consulta não usa watermark, porque o conjunto é pequeno. Como cada venda entra em 2 ou 3 janelas, a soma de todas as janelas (R$ 960) não é a receita real da noite (R$ 375).

---

## RA3 — Estatística, visualizações e machine learning

**Objetivo:** aprofundar a análise da base limpa do RA1 com estatística descritiva, revisar e interpretar os gráficos de P1 a P6, e treinar um modelo simples de machine learning sobre a mesma base.

### Parte 1 — Estatística descritiva (seção final de `HorrorMovieAnalysis.ipynb`)

Resume nota, duração, orçamento, receita, retorno e popularidade com média, mediana, desvio-padrão, coeficiente de variação, amplitude, intervalo interquartil, histogramas e boxplots (estes dois últimos em escala logarítmica para orçamento, receita, retorno e popularidade).

**Principais resultados:** nota (média 5,35 / mediana 5,40) e duração (88,5 / 89 min) têm distribuição equilibrada, quase sem diferença entre média e mediana. Orçamento, receita, retorno e popularidade são muito concentrados — no orçamento, por exemplo, a média (US$ 4,2 milhões) é 21 vezes a mediana (US$ 200 mil), porque poucos filmes com valores enormes puxam a média para cima. Por isso a mediana, não a média, é a medida usada para essas quatro variáveis no projeto todo (inclusive nas respostas de P2, P3, P5 e P6 do RA1).

### Parte 2 — Visualizações e interpretação (primeira metade de `RA3_2_visualizacoes.ipynb`)

Traz de volta os seis gráficos de P1 a P6 do RA1, com título, nomes dos eixos, unidades e legenda conferidos, e uma interpretação curta por gráfico. Não repete os cálculos do RA1 — usa a mesma base limpa e as mesmas regras (nota confiável com 10 votos ou mais; mediana para orçamento, receita, retorno e popularidade).

### Parte 3 — Machine learning (segunda metade de `RA3_2_visualizacoes.ipynb`)

**Problema:** classificar se um filme de terror vai ser bem avaliado pelo público (`vote_average` ≥ 6).

**Alvo e features:** alvo binário `bem_avaliado`, calculado só para os 8.477 filmes com nota confiável (10 votos ou mais) — 30,3% são bem avaliados. Features: duração, década, mês de lançamento, idioma, se o filme pertence a uma franquia e os gêneros combinados ao terror (23 colunas ao todo, sem vazamento de dados: nenhuma vem do alvo).

**Treino e avaliação:** divisão 80/20 estratificada (6.781 filmes de treino, 1.696 de teste). Modelo KNN, com `k` e limiar de decisão escolhidos por validação cruzada (5 blocos) buscando o melhor F1 — o resultado foi `k = 101` e limiar 0,20 (em vez do limiar padrão de 0,50), porque a classe "bem avaliado" é minoritária e um limiar mais baixo captura mais desses casos.

**Resultado no teste**, comparado a três referências (classe positiva = bem avaliado):

| Modelo | Acurácia | F1 |
|---|---|---|
| Chute: classe mais frequente | 0,70 | 0,00 |
| Chute: sorteio proporcional à distribuição | 0,57 | 0,29 |
| KNN como na aula (k = 5, limiar 0,50) | 0,72 | 0,42 |
| **KNN ajustado (k = 101, limiar 0,20)** | 0,67 | **0,57** |

O modelo ajustado tem acurácia um pouco menor que o chute da classe mais frequente e que o KNN da aula, mas isso sozinho não diz muito: o chute da classe mais frequente nunca prevê "bem avaliado" (F1 = 0), e o KNN da aula, com o limiar padrão, também erra a maioria dos filmes bem avaliados. O F1 — que equilibra encontrar os filmes bem avaliados (recall) e não errar demais ao prevê-los (precisão) — mostra que o modelo ajustado aprende algo real sobre o problema: recall de 71,7% (encontra 369 dos 515 filmes bem avaliados do teste) com precisão de 46,7%, um resultado modesto, mas bem acima dos chutes.

### Parte 4 — Representatividade e limitações da base

- **Origem e abrangência:** a base vem do TMDB, um catálogo colaborativo mantido por usuários, extraído em outubro de 2022. Ela não é uma amostra aleatória do cinema de terror: reflete o que foi cadastrado no TMDB até aquele momento, com viés para filmes em inglês e de produção americana/europeia (a participação de outros idiomas já caiu de 50% nos anos 1960 para menos de 30% a partir dos anos 2000 — ver P1).
- **Retrato de um instante:** `popularity` e `vote_average`/`vote_count` são medidos no momento da coleta. Um filme lançado pouco antes de outubro de 2022 teve menos tempo para acumular votos e popularidade do que um filme antigo, o que pode subestimar lançamentos recentes nessas variáveis.
- **Dados financeiros raros e enviesados:** só cerca de 3% dos filmes (1.010 de 32.404) têm orçamento e receita informados — a análise de retorno (P3, P6 e a estatística descritiva da Parte 1) vale só para esse subconjunto. Como filmes bem-sucedidos têm mais chance de ter a bilheteria registrada, esse subconjunto provavelmente superestima o retorno típico do terror de baixo orçamento.
- **Nota pouco confiável com poucos votos:** por isso o projeto usa `nota_confiavel` (10 votos ou mais) sempre que a nota entra na análise, o que reduz a base de 32.404 para 8.477 filmes nessas partes — outra fonte de viés, já que filmes mais obscuros ficam de fora.
- **Consequência para o modelo de machine learning:** o modelo da Parte 3 herda esses vieses — aprende a prever a nota de filmes com votação suficiente para ter uma nota confiável, majoritariamente em inglês e cadastrados no TMDB até 2022, não do cinema de terror como um todo.

### Conclusões do RA3

- Nota e duração são variáveis estáveis e bem comportadas; orçamento, receita, retorno e popularidade são fortemente concentrados em poucos filmes, por isso a mediana (não a média) é a medida certa para descrevê-las, confirmando a escolha já feita nas respostas do RA1.
- Os gráficos revisados na Parte 2 sustentam os mesmos achados do RA1: orçamento maior está fracamente associado a nota maior; filmes baratos têm retorno proporcional maior (não maior chance de se pagar); outubro concentra lançamentos sem ser mais popular; franquias faturam muito mais que avulsos sem serem muito melhor avaliadas.
- O modelo de machine learning da Parte 3 aprende um padrão real, mas limitado: F1 de 0,57 no teste, bem acima dos chutes simples (0,00 e 0,29) e do KNN sem ajuste de limiar (0,42), prevendo corretamente 72% dos filmes bem avaliados à custa de alguns falsos positivos. Duração, década, idioma, franquia e gênero carregam algum sinal sobre a avaliação do público, mas não o suficiente para uma previsão de alta confiança — o que é esperado, já que fatores que o TMDB não registra (roteiro, elenco, marketing, crítica) pesam na nota.
- **Limitação central do projeto:** quase todas as conclusões sobre dinheiro (orçamento, receita, retorno) valem só para os ~3% de filmes com esses dados informados, e as conclusões sobre nota valem só para os ~26% com votação suficiente. A base descreve bem o catálogo de terror do TMDB até outubro de 2022, majoritariamente em inglês, mas não permite afirmações seguras sobre o universo completo do gênero.

---

## Referências

- GOWER, Evan. *Horror Movies*. Kaggle, 2023. https://www.kaggle.com/datasets/evangower/horror-movies
- SHAPIRO, Tanya. *horrormovies*. GitHub. https://github.com/tashapiro/horrormovies
- THE MOVIE DATABASE (TMDB). https://www.themoviedb.org/

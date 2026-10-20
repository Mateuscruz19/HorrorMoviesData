# HorrorMoviesData

Projeto de **Data Analytics / Projeto Baseado em Problemas** (PUCPR, Sistemas de Informação, 5º período, 2º semestre de 2026).
Professora: Marina de Lara.

**Equipe:** Carla Ferraz de Araujo, Gabriella Tavares Pedroso, Luiz Gustavo Santarosa e Mateus Lima da Cruz.

O projeto trabalha com filmes de terror do The Movie Database (TMDB). O RA1 prepara a base de dados; o RA2 usa essa base preparada para demonstrar processamento de dados em streaming.

## Arquivos do repositório

| Arquivo | Etapa | Descrição |
|---|---|---|
| `horror_movies.csv` | RA1 | Dados brutos: 32.540 filmes × 21 atributos |
| `HorrorMovieAnalysis.ipynb` | RA1 | Ingestão, diagnóstico de qualidade (Q1 a Q8), limpeza (L1 a L11), testes e exportação |
| `horror_movies_limpo.csv` | RA1 | Base preparada: 32.404 filmes × 25 atributos (saída do RA1 e entrada do RA2) |
| `HorrorMovieNight.ipynb` | RA2 | Streaming com janela deslizante sobre vendas simuladas de ingressos na noite de Halloween |
| `requirements.txt` | RA1 e RA2 | Dependências do projeto |

## Ambiente

- Python 3.14.8 no ambiente local (kernel registrado no notebook do RA2). No Google Colab, a versão do Python é gerenciada pela plataforma.
- Bibliotecas: pandas, numpy, matplotlib, seaborn (RA1) e pyspark (RA2). As versões estão em `requirements.txt`.
- O RA2 também precisa de **Java** instalado (o Colab já traz). O notebook instala o `pyspark` sozinho se ele não existir.
- Conexão com a internet: os dois notebooks leem os CSVs pela URL raw deste repositório (branch `main`).

Para rodar localmente: pip install -r requirements.txt.

---

## RA1 — Ingestão e preparação dos dados

**Fonte:** *Horror Movies*, extraída do TMDB por Tanya Shapiro em outubro de 2022, publicada no TidyTuesday (semana 44 de 2022) e republicada no Kaggle por Evan Gower (licença CC0). Origem do código: https://github.com/tashapiro/horrormovies. Dados em https://www.kaggle.com/datasets/evangower/horror-movies.

**O que o notebook faz:**
1. Carrega `horror_movies.csv` (32.540 × 21) e o preserva em `df_original`, sem alterações.
2. Avalia a qualidade em oito verificações (Q1 a Q8): unicidade, ausências, zeros que significam "não informado", tipos, domínios e período, coerência entre campos, categorias e valores extremos.
3. Aplica a limpeza em onze decisões (L1 a L11) sobre uma cópia, com testes (`assert`) e uma tabela antes × depois.
4. Exporta `horror_movies_limpo.csv` (32.404 × 25).

**Como reproduzir:**
1. Abrir `HorrorMovieAnalysis.ipynb` no Google Colab (ou em Jupyter local).
2. Executar todas as células em ordem. A célula de ambiente deve imprimir `✓ Ambiente preparado.` e a de ingestão `✓ Dados carregados com sucesso!` com shape 32540 × 21.
3. As células de teste devem imprimir `Todas as verificações passaram.`
4. A última etapa grava `horror_movies_limpo.csv` com 32.404 linhas × 25 colunas.

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

## Referências

- GOWER, Evan. *Horror Movies*. Kaggle, 2023. https://www.kaggle.com/datasets/evangower/horror-movies
- SHAPIRO, Tanya. *horrormovies*. GitHub. https://github.com/tashapiro/horrormovies
- THE MOVIE DATABASE (TMDB). https://www.themoviedb.org/

# Mini-Projeto — Análise Exploratória da Base Varejo

**Aluna:** Aline Góes Dos Santos
**Turma:** T8 — Análise de Dados com Python (SCTEC) · Módulo 1 · Semana 07

## Objetivo
Realizar uma Análise Exploratória de Dados (AED) de uma base de vendas de varejo: diagnosticar a qualidade dos dados, tratar apenas o que for comprovadamente problemático (com justificativa), analisar clientes, compras e categorias de produto e exportar uma base tratada (`varejo_limpo.csv`) com conclusões compreensíveis para quem não conhece Python.

## Estrutura do repositório
| Arquivo | Conteúdo |
|---|---|
| `Miniprojeto_AnaliseVarejo_AlineGoesDosSantos_T8.ipynb` | Notebook comentado com todo o processo (leitura → diagnóstico → limpeza → análise → exportação) |
| `Base_Varejo.csv.zip` | Base original, compactada (contém `Base Varejo.csv`) — **CSV de entrada** |
| `varejo_limpo.csv.zip` | Base tratada `varejo_limpo.csv`, gerada pelo notebook (`index=False`), compactada |
| `README.md` | Este documento |

> **Por que o `varejo_limpo.csv` está compactado?** O arquivo gerado tem cerca de 43 MB, e o envio pelo site do GitHub aceita no máximo 25 MB por arquivo. O `.zip` contém exatamente o CSV gerado pelo notebook; basta descompactar. Executando o notebook, o `varejo_limpo.csv` é gerado novamente a partir da base original.

## Origem da base
- **Base Varejo**, publicada no Kaggle por *namespaiva*: https://www.kaggle.com/datasets/namespaiva/base-varejo
- Versão utilizada: arquivo `Base Varejo.csv` baixado em 24/10/2026 (data interna do arquivo: 03/04/2023), 830.000 linhas, separador `;`.

## Unidade da análise
**Cada linha é um item (produto) de uma compra.** São 830.000 itens, em 18.471 compras (`CO_ID`), de 1.000 clientes (`CL_ID`), entre 04/01/2019 e 08/12/2022. Contar linhas = contar itens; clientes e compras são contados com `nunique()`. A base **não tem valor (R$) nem quantidade**, portanto não permite calcular faturamento.

## Colunas utilizadas
| Coluna | Significado |
|---|---|
| `DATA` | Data da compra (dd/mm/aaaa no original; convertida para data) |
| `CO_ID` | Identificador da compra (cupom) |
| `CL_ID` | Identificador do cliente |
| `CL_GENERO` | Gênero do cliente (F/M) |
| `CL_EC` | Estado civil, codificado de 1 a 5 (categoria, não é medida) |
| `CL_FHL` | Número de filhos do cliente (0 a 4) — coluna numérica analisada |
| `CL_SEG` | Segmento do cliente (A/B/C) |
| `PR_ID` | Identificador do produto |
| `PR_CAT` | Categoria do produto |
| `PR_NOME` | Nome do produto |

## Diagnóstico de qualidade
| Verificação | Resultado |
|---|---|
| Valores ausentes | 0 nas 10 colunas reais; 4 colunas `Unnamed` 100% vazias (sobras do `;` no fim das linhas) |
| Textos vazios / só espaços / espaços nas bordas | 0 |
| Ausência disfarçada `#N/D` | 3.650 linhas (`PR_CAT` e `PR_NOME`, produto `PR_ID` 107) |
| Grafia inconsistente | 7.320 linhas com `REFRIGERANTE LIMaO` |
| Mesmo produto com 2 códigos | 109 dos 118 nomes de produto |
| Datas | 0 inválidas, mas armazenadas como texto |
| Linhas idênticas | 96.553 (11,63%), todas dentro da mesma compra |

## Decisões de limpeza
| Problema | Decisão | Motivo |
|---|---|---|
| 4 colunas vazias | Removidas | Não contêm nenhum dado |
| `REFRIGERANTE LIMaO` | Padronizado para maiúsculas (`REFRIGERANTE LIMAO`), com remoção de espaços nas bordas em 4 colunas de texto | Mesma informação escrita de forma diferente |
| `#N/D` | Substituído por `NAO IDENTIFICADO`; linhas mantidas | É ausência disfarçada. Preencher com um produto seria inventar dado; remover apagaria itens de compras válidas |
| `DATA` em texto | Convertida para data | Ordenação e filtros por período corretos |
| Produto com 2 códigos | Mantido | Pode ser marca/tamanho diferente; não há como confirmar sem o cadastro |
| 96.553 linhas idênticas | **Mantidas** | Duplicata não confirmada: sem coluna de quantidade, a repetição representa mais de uma unidade do mesmo produto na compra. Remover reduziria o volume de itens em 11,63% |

**Registros:** 830.000 antes → 830.000 depois (colunas: 14 → 10).

## Funções criadas
| Função | Parâmetro → Retorno | Uso |
|---|---|---|
| `padronizar_texto(serie)` | coluna de texto → coluna padronizada | Remove espaços e coloca em maiúsculas |
| `tratar_base(df_bruto, remover_duplicatas=False)` | base bruta → base tratada | Função de tratamento (todas as decisões acima) |
| `diagnostico_qualidade(base)` | base → tabela de indicadores | Comparação antes × depois |
| `resumo_numerico(serie)` | coluna numérica → contagem, média, mínimo e máximo | Função de resumo (filhos por cliente e itens por compra) |

## Análises
- **Estatísticas de `CL_FHL` (unidade: cliente, uma linha por cliente):** 1.000 valores válidos, média 1,14, mínimo 0, máximo 4.
- **Filtro:** itens da categoria `ACESSORIOS` (chupeta e mordedor): 14.557 itens, em 9.224 compras.
- **Agrupamento 1:** quantidade de itens por categoria e participação (%).
- **Agrupamento 2:** por segmento (A/B/C) — clientes distintos, compras distintas, itens, compras por cliente e itens por compra.

## Conclusões
1. **A base registra itens, não clientes:** 830.000 itens em 18.471 compras de 1.000 clientes — em média **44,9 itens por compra** e **18,5 compras por cliente**.
2. **Qualidade boa, com 2 problemas reais:** 3.650 itens com produto `#N/D` (0,44%) e 7.320 com grafia inconsistente (0,88%); nenhum nulo ou texto vazio nas colunas com dados.
3. **Alimentos dominam o volume:** `ALIMENTOS` responde por 52,4% dos itens; `HIGIENE` e `LIMPEZA` somam mais 36,3%.
4. **Segmento B é o maior público** (64,5% dos clientes e 63,9% dos itens), mas os três segmentos compram com frequência (17,8 a 19,0 compras por cliente) e volume (44,8 a 45,4 itens por compra) muito parecidos.
5. **52,7% dos clientes não têm filhos** (média de 1,14 filho por cliente) e, mesmo assim, ficam com 52,5% dos itens de chupeta e mordedor — exatamente a mesma fatia que têm do total de itens.

## Recomendação
Antes de segmentar campanhas por número de filhos, **validar o cadastro de clientes** (`CL_FHL`), já que ele não explica a compra de produtos infantis. Em paralelo, corrigir na origem o produto `PR_ID 107` (`#N/D`) e investigar os 109 produtos com dois códigos.

## Limitação
A base **não tem valor (R$) nem quantidade** dos itens: não é possível falar em faturamento ou ticket médio, nem confirmar se as 96.553 linhas idênticas são unidades repetidas ou erro de registro.

## Explicação do processo
A **leitura** ocorre na Sprint 1, com `pd.read_csv("Base_Varejo.csv.zip", sep=";")`. A **transformação** acontece na função `tratar_base()` (Sprints 2 e 3), que trabalha sobre uma cópia da base e gera o `df_limpo`. A **gravação** ocorre na Sprint 5, com `df_limpo.to_csv("varejo_limpo.csv", index=False)`. Uma decisão de limpeza que afetou diretamente a qualidade da análise foi **manter as 96.553 linhas repetidas**: se tivessem sido removidas como "duplicatas", o volume de itens cairia 11,63% e todas as contagens por categoria e por segmento ficariam subestimadas, sem nenhuma prova de que se tratava de erro. Da mesma forma, rotular o `#N/D` como `NAO IDENTIFICADO` manteve esses itens nas contagens e deixou a ausência visível na análise por categoria (0,44% dos itens).

## Como executar
**Google Colab**
1. Abra o notebook no Colab (*Arquivo → Fazer upload de notebook*).
2. No menu lateral (ícone de pasta), envie o arquivo `Base_Varejo.csv.zip` deste repositório. Não é preciso descompactar.
3. Clique em *Ambiente de execução → Executar tudo*. O `varejo_limpo.csv` será gerado na pasta de arquivos do Colab.

**No computador (Jupyter / VS Code)**
1. Baixe o repositório e deixe o notebook e o `Base_Varejo.csv.zip` na mesma pasta.
2. Instale o pandas (`pip install pandas`).
3. Execute o notebook do início ao fim; ele grava `varejo_limpo.csv` na mesma pasta.

## Referências
- Material da disciplina: *Revisão orientada do mini-projeto — Módulo 1, Semana 07* (SCTEC).
- Documentação do pandas: https://pandas.pydata.org/docs/
- Base Varejo (Kaggle): https://www.kaggle.com/datasets/namespaiva/base-varejo

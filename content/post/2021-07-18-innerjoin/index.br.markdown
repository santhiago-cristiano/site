---
title: "Como juntar conjuntos de dados no R. Parte II: dplyr::inner_join()"
author: Santhiago Cristiano
date: '2021-07-20'
slug: []
categories:
  - R
tags:
  - dplyr
  - joins
  - R
  - inner join
type: ''
subtitle: ''
image: ''
---


Dando continuidade a série de posts sobre mesclagem de dados no R, hoje falarei sobre a função `inner_join()` do pacote `{dplyr}`. Na [primeira parte](https://santhiago-cristiano.github.io/site/post/2021-07-15-leftjoin/), demonstrei como podemos mesclar dois ou mais conjuntos de dados usando a função `left_join()`. Seguindo em frente, utilizarei os mesmos conjuntos de dados do primeiro post, que podem ser baixados [clicando aqui](https://github.com/santhiago-cristiano/site-scripts/tree/main/2021/2021-07-16-leftjoin/dados-mesclados). 


# Pacotes e importação dos dados

Todo esse processo inicial é semelhante ao do primeiro post: carregamos os pacotes que serão usados e em seguida importamos os dados para o R. Como já descrevi esse processo com mais detalhes na [parte I](https://santhiago-cristiano.github.io/site/post/2021-07-15-leftjoin/), serei breve dessa vez para não se tornar repetitivo.


```r
library(readr)
library(dplyr)
library(purrr)
```

O `{readr}` serve para fazermos a leitura e importação dos dados para o R; o `{dplyr}` serve para manejarmos os dados;
e o `{purrr}` tem funções que simplificam o processo de loop e será útil entre outras coisas para importar todos os conjuntos de dados de uma única vez.


```r
arquivos_csv <- list.files(
  path = "./dados/",
  pattern = "*.csv",
  full.names = TRUE
)

arquivos_csv
```

```
## [1] "./dados/2010-leftjoin.csv" "./dados/2011-leftjoin.csv"
## [3] "./dados/2012-leftjoin.csv"
```

```r
mortes_infantis <- map(
  arquivos_csv,
  read_csv2,
  locale(encoding = "latin1"),
  col_names = TRUE,
  col_types = "ccccdi"
) %>% 
  set_names(2010:2012)

mortes_infantis
```

```
## $`2010`
## # A tibble: 2,386 x 6
##    cod_ibge regiao uf    municipio             total   ano
##    <chr>    <chr>  <chr> <chr>                 <dbl> <int>
##  1 1100015  Norte  RO    Alta Floresta D'Oeste     5  2010
##  2 1100023  Norte  RO    Ariquemes                29  2010
##  3 1100049  Norte  RO    Cacoal                   14  2010
##  4 1100064  Norte  RO    Colorado do Oeste         2  2010
##  5 1100072  Norte  RO    Corumbiara                1  2010
##  6 1100080  Norte  RO    Costa Marques             1  2010
##  7 1100098  Norte  RO    Espigão D'Oeste           3  2010
##  8 1100106  Norte  RO    Guajará-Mirim            13  2010
##  9 1100114  Norte  RO    Jaru                      8  2010
## 10 1100122  Norte  RO    Ji-Paraná                31  2010
## # ... with 2,376 more rows
## 
## $`2011`
## # A tibble: 2,384 x 6
##    cod_ibge regiao uf    municipio             total   ano
##    <chr>    <chr>  <chr> <chr>                 <dbl> <int>
##  1 1100015  Norte  RO    Alta Floresta D'Oeste     3  2011
##  2 1100023  Norte  RO    Ariquemes                15  2011
##  3 1100049  Norte  RO    Cacoal                   14  2011
##  4 1100056  Norte  RO    Cerejeiras                2  2011
##  5 1100072  Norte  RO    Corumbiara                1  2011
##  6 1100080  Norte  RO    Costa Marques             1  2011
##  7 1100098  Norte  RO    Espigão D'Oeste           6  2011
##  8 1100106  Norte  RO    Guajará-Mirim             5  2011
##  9 1100114  Norte  RO    Jaru                      2  2011
## 10 1100122  Norte  RO    Ji-Paraná                10  2011
## # ... with 2,374 more rows
## 
## $`2012`
## # A tibble: 2,313 x 6
##    cod_ibge regiao uf    municipio             total   ano
##    <chr>    <chr>  <chr> <chr>                 <dbl> <int>
##  1 1100015  Norte  RO    Alta Floresta D'Oeste     1  2012
##  2 1100023  Norte  RO    Ariquemes                14  2012
##  3 1100049  Norte  RO    Cacoal                   12  2012
##  4 1100056  Norte  RO    Cerejeiras                3  2012
##  5 1100064  Norte  RO    Colorado do Oeste         2  2012
##  6 1100080  Norte  RO    Costa Marques             2  2012
##  7 1100098  Norte  RO    Espigão D'Oeste           1  2012
##  8 1100106  Norte  RO    Guajará-Mirim             3  2012
##  9 1100114  Norte  RO    Jaru                      6  2012
## 10 1100122  Norte  RO    Ji-Paraná                10  2012
## # ... with 2,303 more rows
```

A função `list.files()` lista todos os arquivos csv contidos na pasta `"dados/"` e nos retorna um vetor, já a função `purrr::map()` itera sobre cada elemento desse vetor e aplica a função `readr::read_csv2()` em cada um deles nos retornando uma lista de data frames. 


# dplyr::inner_join()

A função `inner_join()` possui os mesmos argumentos do `left_join()`:

`inner_join(x, y, by = NULL, copy = FALSE, suffix = c(".x", ".y"), ...)`

A diferença entre as duas é que, enquanto `left_join()` nos retorna todas as linhas de `x`, todas as colunas de `x` e `y` e preenche com `NA` as linhas em `x` que não possuem uma correspondente em `y`, `inner_join()` retorna **somente as linhas de `x` onde existem valores correpondentes em `y`**, ou seja, teremos como retorno um novo conjunto de dados em que estarão presentes somente as linhas que são comuns entre ambos. Além disso, `inner_join()` também retorna todas as colunas de `x` e `y`.

Novamente, `x` e `y` correspondem aos conjuntos de dados que desejamos unir, enquanto `by` recebe o nome da coluna (ou colunas, pode ser mais de uma) que servirá como chave para unir os dados. `by = "nome da coluna"` é usado quando o nome da coluna é o mesmo em ambos os conjuntos de dados e `by = c("nome da coluna em x" = "nome da coluna em y")` quando os nomes das colunas são diferentes. Como sempre, exemplos podem nos ajudar a entender melhor como as coisas acontecem.

## Exemplo

Admita que tenhamos dois conjuntos de dados com o desempenho dos alunos do ensino fundamental de determinada escola do Brasil em dois anos diferentes: 2019 e 2020. E desejamos unir esses dados trazendo somente os alunos que estão presentes nos dois anos para verificar se houve uma melhora ou piora no desempenho de cada um deles de um ano para o outro.


```r
alunos_2019 <- tibble::tribble(
  ~ matricula,   ~ nome,     ~ media,
  10,            "João",     7.1,
  11,            "José",     8.5,
  12,            "Maria",    5.2
)

alunos_2020 <- tibble::tribble(
  ~ matricula,    ~ nome,       ~ media,
  9,              "Joana",      8.0,
  10,             "João",       9.0,     
  11,             "José",       5.4,
  12,             "Maria",      8.7,
  13,             "Ana",        10.0,
  14,             "Francisco",  4.3
)
```

Podemos utilizar a coluna com o número da matrícula como chave para unir os dados passando-a para o argumento `by`:


```r
alunos_2019 %>%
  inner_join(alunos_2020, by = "matricula")
```

```
## # A tibble: 3 x 5
##   matricula nome.x media.x nome.y media.y
##       <dbl> <chr>    <dbl> <chr>    <dbl>
## 1        10 João       7.1 João       9  
## 2        11 José       8.5 José       5.4
## 3        12 Maria      5.2 Maria      8.7
```

Repare que, como as colunas `nome` e `media` existem nas duas tabelas, é acrescentado automaticamente um sufixo em cada uma delas para evitar nomes de colunas repetidos. Nesse caso, `.x` é o sufixo referente as colunas do data frame `alunos_2019` e `.y` é o sufixo referente as colunas de `alunos_2020`. É possível alterar esses nomes passando os sufixos que desejamos acrescentar para o argumento `suffix`. Dessa forma:


```r
alunos_2019 %>%
  inner_join(alunos_2020, by = "matricula", suffix = c("_2019", "_2020"))
```

```
## # A tibble: 3 x 5
##   matricula nome_2019 media_2019 nome_2020 media_2020
##       <dbl> <chr>          <dbl> <chr>          <dbl>
## 1        10 João             7.1 João             9  
## 2        11 José             8.5 José             5.4
## 3        12 Maria            5.2 Maria            8.7
```

Note que a ordem que os sufixos são passados importa! Como os dados referentes ao ano de 2019 vem primeiro, passamos primeiro o sufixo de 2019 e depois o de 2020. Se fosse ao contrário, seria assim:


```r
alunos_2020 %>%
  inner_join(alunos_2019, by = "matricula", suffix = c("_2020", "_2019"))
```

```
## # A tibble: 3 x 5
##   matricula nome_2020 media_2020 nome_2019 media_2019
##       <dbl> <chr>          <dbl> <chr>          <dbl>
## 1        10 João             9   João             7.1
## 2        11 José             5.4 José             8.5
## 3        12 Maria            8.7 Maria            5.2
```

# Juntando os conjuntos de dados

Feito essa explicação, agora vamos juntar os conjuntos de dados que foram importados lá no ínicio mantendo somente os municípios que estão presentes em todos os anos (2010, 2011 e 2012). Como chave para unir os dados utilizarei a coluna `cod_ibge`, que possui o código IBGE de 7 digítos para cada município.

Uma das maneiras de unir nossos dados é assim:


```r
mortes_infantis_inner <- mortes_infantis[["2010"]] %>%
  inner_join(
    mortes_infantis[["2011"]],
    by = "cod_ibge",
    suffix = c("_2010", "_2011")
  ) %>%
  inner_join(mortes_infantis[["2012"]], by = "cod_ibge")
```

Ok. Parece bom. Mas e se pudéssemos simplificar isso deixando nosso código mais limpo e legível? Com `purrr::reduce()` isso é possível!


```r
mortes_infantis_inner <- reduce(
  mortes_infantis,
  inner_join,
  by = "cod_ibge",
  suffix = c("_2010", "_2011")
)
```

Para a função `reduce()` passamos a lista que contém os data frames que queremos juntar e a função `inner_join()` com os seus devidos argumentos. Simples assim! É incrível o que conseguimos fazer utilizando o `{purrr}`.

Agora, temos um novo conjunto de dados que contém somente os municípios que estavam presentes em todos os anos:


```r
glimpse(mortes_infantis_inner)
```

```
## Rows: 1,447
## Columns: 16
## $ cod_ibge       <chr> "1100015", "1100023", "1100049", "1100080", "1100098", ~
## $ regiao_2010    <chr> "Norte", "Norte", "Norte", "Norte", "Norte", "Norte", "~
## $ uf_2010        <chr> "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "~
## $ municipio_2010 <chr> "Alta Floresta D'Oeste", "Ariquemes", "Cacoal", "Costa ~
## $ total_2010     <dbl> 5, 29, 14, 1, 3, 13, 8, 31, 3, 7, 307, 2, 2, 25, 2, 1, ~
## $ ano_2010       <int> 2010, 2010, 2010, 2010, 2010, 2010, 2010, 2010, 2010, 2~
## $ regiao_2011    <chr> "Norte", "Norte", "Norte", "Norte", "Norte", "Norte", "~
## $ uf_2011        <chr> "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "~
## $ municipio_2011 <chr> "Alta Floresta D'Oeste", "Ariquemes", "Cacoal", "Costa ~
## $ total_2011     <dbl> 3, 15, 14, 1, 6, 5, 2, 10, 6, 6, 254, 1, 4, 21, 3, 1, 7~
## $ ano_2011       <int> 2011, 2011, 2011, 2011, 2011, 2011, 2011, 2011, 2011, 2~
## $ regiao         <chr> "Norte", "Norte", "Norte", "Norte", "Norte", "Norte", "~
## $ uf             <chr> "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "~
## $ municipio      <chr> "Alta Floresta D'Oeste", "Ariquemes", "Cacoal", "Costa ~
## $ total          <dbl> 1, 14, 12, 2, 1, 3, 6, 10, 3, 3, 270, 1, 3, 26, 2, 2, 7~
## $ ano            <int> 2012, 2012, 2012, 2012, 2012, 2012, 2012, 2012, 2012, 2~
```

Como esperado, foi acrescentado o sufixo `_2010` para as colunas do ano 2010 e `_2011` para as colunas do ano 2011. As últimas 5 colunas sem sufixo, são referentes ao ano de 2012. Note que agora temos diversas colunas com região, UF e município, no entanto, como sei que o conteúdo é o mesmo para todas elas, podemos jogar fora essas colunas repetidas:


```r
mortes_infantis_inner <- mortes_infantis_inner %>%
  select(
    !(contains(c("regiao", "uf", "municipio", "ano")) &
      ends_with(c("_2010", "_2011"))),
    -ano
  ) %>%
  rename(total_2012 = total) %>%
  relocate(c(regiao, uf, municipio), .after = cod_ibge)
```

Para remover todas as colunas desejadas, peço para a função `select()` do `{dplyr}` selecionar somente as colunas que **simultaneamente** não contenham em seus nomes as palavras `"regiao"`, `"uf"`, `"municipio"` ou `"ano"` **e** termine com os sufixos `"_2010"` ou `"_2011"`. Após isso, apenas renomeio e reposiciono as outras colunas com `dplyr::rename()` e `dplyr::relocate()`, respectivamente.


```r
glimpse(mortes_infantis_inner)
```

```
## Rows: 1,447
## Columns: 7
## $ cod_ibge   <chr> "1100015", "1100023", "1100049", "1100080", "1100098", "110~
## $ regiao     <chr> "Norte", "Norte", "Norte", "Norte", "Norte", "Norte", "Nort~
## $ uf         <chr> "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO",~
## $ municipio  <chr> "Alta Floresta D'Oeste", "Ariquemes", "Cacoal", "Costa Marq~
## $ total_2010 <dbl> 5, 29, 14, 1, 3, 13, 8, 31, 3, 7, 307, 2, 2, 25, 2, 1, 5, 4~
## $ total_2011 <dbl> 3, 15, 14, 1, 6, 5, 2, 10, 6, 6, 254, 1, 4, 21, 3, 1, 7, 39~
## $ total_2012 <dbl> 1, 14, 12, 2, 1, 3, 6, 10, 3, 3, 270, 1, 3, 26, 2, 2, 7, 37~
```

Concluído todo o processo, agora é só exportar os dados:


```r
write.csv2(
  mortes_infantis_inner,
  file = "./dados-inner/mortes_infantis_inner.csv",
  row.names = FALSE
)
```

É isso. Finalizo por aqui. O código completo desse post está disponível no meu [GitHub](https://github.com/santhiago-cristiano/site-scripts/tree/main/2021/2021-07-20-innerjoin).

Até a próxima 👋. 

# Saiba Mais

- https://readr.tidyverse.org/

- https://dplyr.tidyverse.org/

- https://dplyr.tidyverse.org/reference/join.html

- https://purrr.tidyverse.org/

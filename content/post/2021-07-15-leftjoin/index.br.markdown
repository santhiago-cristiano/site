---
title: "Como juntar conjuntos de dados no R. Parte I: dplyr::left_join()"
author: Santhiago Cristiano
date: '2021-07-16'
slug: []
categories:
  - R
tags:
  - joins
  - dplyr
  - left join
  - R
type: ''
subtitle: ''
image: ''
---

Neste primeiro post do site, demonstrarei como podemos juntar dois ou mais conjuntos de dados no R por meio de uma ou mais colunas em comum entre eles (chaves) utilizando a função `left_join()` do pacote `{dplyr}`.


# Base de dados

Antes de tudo, precisamos obter nossos dados. Os dados para este post foram obtidos no DATASUS e correpondem aos óbitos infantis nos anos de 2010, 2011 e 2012. Os dados originais sem nenhuma modificação podem ser obtidos [aqui](https://datasus.saude.gov.br/mortalidade-desde-1996-pela-cid-10). Os dados que utilizarei como exemplo já passaram por uma breve limpeza e podem ser baixados no meu [GitHub](https://github.com/santhiago-cristiano/site-scripts/tree/main/2021/2021-07-16-leftjoin/dados).  

# Pacotes e criação do projeto

Inicialmente, recomendo a criação de um projeto. Se você não sabe o que é, como funciona, como criar ou por qual motivo você deveria passar a utilizar projetos no R, recomendo a leitura [deste capítulo do livro Zen do R](https://curso-r.github.io/zen-do-r/rproj-dir.html) escrito pelo pessoal da [Curso-R](https://curso-r.com/).

Dito isso, vamos carregar os pacotes que serão utilizados:


```r
library(readr)
library(dplyr)
library(abjData)
library(purrr)
library(glue)
```

Se ainda não possui esses pacotes instalados, você pode instalar todos de uma só vez rodando o comando abaixo:


```r
install.packages(c("readr", "dplyr", "abjData", "purrr", "glue"))
```

# Importando os dados para o R

Para importar os dados para o R, primeiramente utilizamos a função `list.files()`, que nos retorna um vetor com o caminho de todos os arquivos contidos em um determinado diretório, nesse caso, arquivos do tipo csv.


```r
# lista todos os arquivos csv contidos na minha pasta "dados/" 

arquivos_csv <- list.files(
  path = "./dados/",
  pattern = "*.csv",
  full.names = TRUE
)

arquivos_csv
```

```
## [1] "./dados/2010-mortes-infantis.csv" "./dados/2011-mortes-infantis.csv"
## [3] "./dados/2012-mortes-infantis.csv"
```

Em seguida, podemos ler todos esses arquivos de uma tacada só utilizando a função `read_csv2()` em conjunto com a `map()` dos pacotes `{readr}` e `{purrr}`, respectivamente. `read_csv2()` nos permite ler arquivos csv separados por `;` e o `map()` vai simplesmente aplicar essa função em cada um dos três elementos do vetor `arquivos_csv` e nos retornar uma lista de data frames.  


```r
# lendo todos os arquivos e armazenando em uma lista

morte_infantil <- map(
  arquivos_csv,
  read_csv2,
  locale(encoding = "latin1"),
  col_names = TRUE,
  col_types = "ccdi"
)

# atribuindo o ano como "nome" para cada um dos data frames da lista

names(morte_infantil) <- 2010:2012
```

Como argumentos para `read_csv2()` passamos o encoding dos nossos arquivos csv dentro da função `locale()`, que neste caso é `latin1`. O `latin1` é apenas um dos diversos encodings existentes, para saber mais, [clique aqui](https://en.wikipedia.org/wiki/Character_encoding). Além do encoding, passamos `col_names = TRUE`, para que a primeira linha dos nossos arquivos sejam reconhecidas como os nomes das colunas. Podemos ainda definir explicitamente o tipo de cada coluna com `col_types`. Nos dados temos 4 colunas, então, a primeira e a segunda coluna serão do tipo `c = character`, a terceira do tipo `d = double` e a última do tipo `i = integer`.


```r
# visualizando a lista
morte_infantil
```

```
## $`2010`
## # A tibble: 2,386 x 4
##    codigo municipio             total   ano
##    <chr>  <chr>                 <dbl> <int>
##  1 110001 Alta Floresta D'Oeste     5  2010
##  2 110002 Ariquemes                29  2010
##  3 110004 Cacoal                   14  2010
##  4 110006 Colorado do Oeste         2  2010
##  5 110007 Corumbiara                1  2010
##  6 110008 Costa Marques             1  2010
##  7 110009 Espigão D'Oeste           3  2010
##  8 110010 Guajará-Mirim            13  2010
##  9 110011 Jaru                      8  2010
## 10 110012 Ji-Paraná                31  2010
## # ... with 2,376 more rows
## 
## $`2011`
## # A tibble: 2,384 x 4
##    codigo municipio             total   ano
##    <chr>  <chr>                 <dbl> <int>
##  1 110001 Alta Floresta D'Oeste     3  2011
##  2 110002 Ariquemes                15  2011
##  3 110004 Cacoal                   14  2011
##  4 110005 Cerejeiras                2  2011
##  5 110007 Corumbiara                1  2011
##  6 110008 Costa Marques             1  2011
##  7 110009 Espigão D'Oeste           6  2011
##  8 110010 Guajará-Mirim             5  2011
##  9 110011 Jaru                      2  2011
## 10 110012 Ji-Paraná                10  2011
## # ... with 2,374 more rows
## 
## $`2012`
## # A tibble: 2,313 x 4
##    codigo municipio             total   ano
##    <chr>  <chr>                 <dbl> <int>
##  1 110001 Alta Floresta D'Oeste     1  2012
##  2 110002 Ariquemes                14  2012
##  3 110004 Cacoal                   12  2012
##  4 110005 Cerejeiras                3  2012
##  5 110006 Colorado do Oeste         2  2012
##  6 110008 Costa Marques             2  2012
##  7 110009 Espigão D'Oeste           1  2012
##  8 110010 Guajará-Mirim             3  2012
##  9 110011 Jaru                      6  2012
## 10 110012 Ji-Paraná                10  2012
## # ... with 2,303 more rows
```


Como pode ser visto, além do total de mortes infantis e do ano, os dados do DATASUS contém apenas os nomes dos municípios e os códigos de 6 dígitos referentes a cada um deles. Seria interessante também adicionarmos as regiões e os estados e substituirmos o código de 6 dígitos pelo código de 7 dígitos do IBGE. Para isso, importaremos a base de dados `muni`, do pacote `{abjData}`, que contém as colunas que desejamos acrescentar.


```r
glimpse(abjData::muni)
```

```
## Rows: 5,572
## Columns: 16
## $ muni_id       <chr> "1100015", "1100023", "1100031", "1100049", "1100056", "~
## $ muni_id_6     <chr> "110001", "110002", "110003", "110004", "110005", "11000~
## $ muni_nm       <chr> "Alta Floresta D'oeste", "Ariquemes", "Cabixi", "Cacoal"~
## $ muni_nm_clean <chr> "ALTA FLORESTA DOESTE", "ARIQUEMES", "CABIXI", "CACOAL",~
## $ uf_nm         <chr> "Rondônia", "Rondônia", "Rondônia", "Rondônia", "Rondôni~
## $ uf_sigla      <chr> "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "RO", "R~
## $ uf_id         <chr> "11", "11", "11", "11", "11", "11", "11", "11", "11", "1~
## $ regiao_nm     <chr> "Norte", "Norte", "Norte", "Norte", "Norte", "Norte", "N~
## $ tse_id        <chr> "310", "78", "450", "94", "272", "230", "655", "213", "2~
## $ rf_id         <chr> "33", "7", "37", "9", "27", "23", "981", "21", "25", "1"~
## $ bcb_id        <chr> "43036", "9393", "44626", "10746", "42219", "41210", "46~
## $ existia_1991  <int> 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0,~
## $ existia_2000  <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,~
## $ existia_2010  <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,~
## $ lon           <dbl> -62.27467, -62.95718, -60.63986, -61.32475, -61.26095, -~
## $ lat           <dbl> -12.470169, -9.951900, -13.474894, -11.301234, -13.20351~
```

Essa base de dados completa possui muitas colunas, mas não precisamos de todas elas. Para selecionar somente as colunas que desejamos, podemos utilizar a função `select()` do `{dplyr}`. Aqui também utilizo o [operador pipe (%>%)](https://www.datacamp.com/community/tutorials/pipe-r-tutorial) do `{magrittr}`.


```r
# carregando os dados auxiliares

auxiliar <- abjData::muni %>%
  select(muni_id, muni_id_6, uf_sigla, regiao_nm) %>%
  arrange(muni_id_6)
```

Dentro do `select()`, passamos como argumento os nomes das colunas que queremos selecionar:

- `muni_id`: código de 7 dígitos;
- `muni_id_6`: código de 6 dígitos que servirá como chave para fazermos o join com nossa base de dados do DATASUS;
- `uf_sigla`e `regiao_nm`: siglas dos estados e nomes das regiões, respectivamente.

O `arrange()` está apenas ordenando os dados em ordem crescente de acordo com o código de 6 dígitos.

# dplyr::left_join()

A função `left_join()` possui os seguintes argumentos:

`left_join(x, y, by = NULL, copy = FALSE, suffix = c(".x", ".y"), ...)`

Para o caso em questão, precisamos somente dos três primeiros. `x` e `y`correspondem aos dois conjuntos de dados que desejamos juntar. Para o `by`, passamos o nome da coluna (ou colunas) que usaremos como chave para mesclar os dados. Quando o nome da coluna é o mesmo em ambos, usamos somente `by = "nome da coluna"`, já quando os nomes das colunas são diferentes, usamos `by = c("nome da coluna em x" = "nome da coluna em y")`.

O `left_join()` irá retornar todas as linhas de `x` e todas as colunas de `x` e `y`. As linhas em `x` que não possuem uma correspondente em `y` são preenchidas automaticamente com `NA`. Talvez tudo isso tenha ficado um pouco confuso, mas com exemplos pode ficar mais claro.

## Exemplo 1

Admita que tenhamos dois data frames ([tibbles](https://tibble.tidyverse.org/)), que chamarei de `tbl_1` e `tbl_2`:


```r
tbl_1 <- tibble::tribble(
  ~cpf, ~nome, ~idade,
  11111111111, "Maria", 25,
  22222222222, "José" , 40,
  33333333333, "Fran" , 23,
  44444444444, "Jõao" , 30,
)
tbl_1
```

```
## # A tibble: 4 x 3
##           cpf nome  idade
##         <dbl> <chr> <dbl>
## 1 11111111111 Maria    25
## 2 22222222222 José     40
## 3 33333333333 Fran     23
## 4 44444444444 Jõao     30
```

```r
tbl_2 <- tibble::tribble(
  ~cpf, ~cidade, ~estado,
  11111111111, "São Paulo"     , "SP",
  22222222222, "Mossoró"       , "RN",
  33333333333, "Natal"         , "RN",
  44444444444, "Rio de Janeiro", "RJ",
)
tbl_2
```

```
## # A tibble: 4 x 3
##           cpf cidade         estado
##         <dbl> <chr>          <chr> 
## 1 11111111111 São Paulo      SP    
## 2 22222222222 Mossoró        RN    
## 3 33333333333 Natal          RN    
## 4 44444444444 Rio de Janeiro RJ
```

Vamos supor que o primeiro contém o CPF, nome e idade de clientes da empresa R & Python SA (pouco criativo, eu admito), enquanto o segundo contém o CPF, a cidade e o estado onde cada um desses clientes moram. Agora suponha que desejamos reunir todas as informações dos clientes em um único data frame, acrescentando as colunas de cidade e estado da `tbl_2` na `tbl_1`. Como faríamos isso usando o `left_join()`?

Note que a coluna `cpf` é comum em ambas as tabelas, portanto, podemos usá-la como chave passando-a para o argumento `by`, dessa forma:


```r
left_join(x = tbl_1, y = tbl_2, by = "cpf")
```

```
## # A tibble: 4 x 5
##           cpf nome  idade cidade         estado
##         <dbl> <chr> <dbl> <chr>          <chr> 
## 1 11111111111 Maria    25 São Paulo      SP    
## 2 22222222222 José     40 Mossoró        RN    
## 3 33333333333 Fran     23 Natal          RN    
## 4 44444444444 Jõao     30 Rio de Janeiro RJ
```

Alternativamente, o mesmo resultado pode ser obtido utilizando o pipe `%>%`:


```r
tbl_1 %>% 
  left_join(tbl_2, by = "cpf")
```

```
## # A tibble: 4 x 5
##           cpf nome  idade cidade         estado
##         <dbl> <chr> <dbl> <chr>          <chr> 
## 1 11111111111 Maria    25 São Paulo      SP    
## 2 22222222222 José     40 Mossoró        RN    
## 3 33333333333 Fran     23 Natal          RN    
## 4 44444444444 Jõao     30 Rio de Janeiro RJ
```

Nesse exemplo, cada linha da coluna `cpf` da `tbl_1` possui uma linha idêntica na coluna `cpf` da `tbl_2`, por esse motivo, não foi retornado nenhum valor `NA`. 

## Exemplo 2

Neste outro exemplo, farei algumas mudanças nos dados:


```r
tbl_1 <- tibble::tribble(
  ~cpf, ~nome, ~idade,
  11111111111, "Maria", 25,
  22222222222, "José" , 40,
  33333333333, "Fran" , 23,
  44444444444, "Jõao" , 30,
  55555555555, "Messi", 34
)
tbl_1
```

```
## # A tibble: 5 x 3
##           cpf nome  idade
##         <dbl> <chr> <dbl>
## 1 11111111111 Maria    25
## 2 22222222222 José     40
## 3 33333333333 Fran     23
## 4 44444444444 Jõao     30
## 5 55555555555 Messi    34
```

```r
tbl_2 <- tibble::tribble(
  ~cpf_cliente, ~cidade, ~estado,
  11111111111, "São Paulo"     , "SP",
  22222222222, "Mossoró"       , "RN",
  33333333333, "Natal"         , "RN",
  44444444444, "Rio de Janeiro", "RJ",
  66666666666, "Recife"        , "PE"  
)
tbl_2
```

```
## # A tibble: 5 x 3
##   cpf_cliente cidade         estado
##         <dbl> <chr>          <chr> 
## 1 11111111111 São Paulo      SP    
## 2 22222222222 Mossoró        RN    
## 3 33333333333 Natal          RN    
## 4 44444444444 Rio de Janeiro RJ    
## 5 66666666666 Recife         PE
```

Observe agora que a coluna `cpf` da `tbl_1` está com uma linha a mais e que esta não possui uma correspondente no segundo data frame. Além disso, o nome da coluna com o CPF dos clientes está diferente na `tbl_2`. Neste caso, o join ficaria assim:


```r
tbl_1 %>% 
  left_join(tbl_2, by = c("cpf" = "cpf_cliente"))
```

```
## # A tibble: 5 x 5
##           cpf nome  idade cidade         estado
##         <dbl> <chr> <dbl> <chr>          <chr> 
## 1 11111111111 Maria    25 São Paulo      SP    
## 2 22222222222 José     40 Mossoró        RN    
## 3 33333333333 Fran     23 Natal          RN    
## 4 44444444444 Jõao     30 Rio de Janeiro RJ    
## 5 55555555555 Messi    34 <NA>           <NA>
```

Por fim, vale ressaltar que a ordem que passamos as tabelas no `left_join()` é importante. O resultado seria diferente se fizéssemos assim:


```r
tbl_2 %>% 
  left_join(tbl_1, by = c("cpf_cliente" = "cpf"))
```

```
## # A tibble: 5 x 5
##   cpf_cliente cidade         estado nome  idade
##         <dbl> <chr>          <chr>  <chr> <dbl>
## 1 11111111111 São Paulo      SP     Maria    25
## 2 22222222222 Mossoró        RN     José     40
## 3 33333333333 Natal          RN     Fran     23
## 4 44444444444 Rio de Janeiro RJ     Jõao     30
## 5 66666666666 Recife         PE     <NA>     NA
```

No primeiro caso, todas as linhas da `tbl_1` são retornadas, já no segundo, todas as linhas da `tbl_2` é que são retornadas. 

# Juntando os conjuntos de dados

Agora que importamos todos os dados que precisamos e sabemos como funciona o `left_join()`, podemos enfim juntar nossos dados do DATASUS com os da nossa base auxiliar. Neste caso, como coluna chave para a mesclagem utilizaremos `by = c("codigo" = "muni_id_6")`.

Uma das maneiras de fazer o left_join em nossos dados é assim:


```r
# 2010
morte_infantil[["2010"]] <- morte_infantil[["2010"]] %>% 
  left_join(auxiliar, by = c("codigo" = "muni_id_6"))

# 2011
morte_infantil[["2011"]] <- morte_infantil[["2011"]] %>% 
  left_join(auxiliar, by = c("codigo" = "muni_id_6"))

# 2012
morte_infantil[["2012"]] <- morte_infantil[["2012"]] %>% 
  left_join(auxiliar, by = c("codigo" = "muni_id_6"))
```

No entanto, há uma outra forma de se fazer (que considero bem melhor) usando o `map()` do `{purrr}`. Para tanto, criarei uma função que terá como argumento um data frame que chamarei de `x` (pode ser qualquer nome). Vou chamar essa função de `mesclar_dados` (pode ser qualquer nome) e desejo que ela faça o seguinte:

- Join nos conjuntos de dados (`dplyr::left_join()`); 
- Após o join, remova a coluna `codigo`, já que não precisamos mais dela (`dplyr::select()`); 
- Renomeie as colunas `muni_id`, `uf_sigla` e `regiao_nm` (`dplyr::rename()`);
- Reposicione as colunas renomeadas para antes da coluna `municipio` (`dplyr::relocate()`).


```r
mesclar_dados <- function(x) {
  x %>%
    left_join(auxiliar, by = c("codigo" = "muni_id_6")) %>%
    select(-codigo) %>%
    rename(cod_ibge = muni_id, uf = uf_sigla, regiao = regiao_nm) %>%
    relocate(c(cod_ibge, regiao, uf), .before = municipio)
}
```

Criada a função, agora podemos simplesmente passar a lista `morte_infantil` e a função `mesclar_dados` como argumentos para o `map()` e ver a mágica acontecer:


```r
morte_infantil_mesclado <- map(morte_infantil, mesclar_dados)

morte_infantil_mesclado
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

Pronto! Simples assim! O `map()` aplicou a função em cada um dos data frames contidos na lista e nos retornou uma outra lista com os data frames já mesclados. Agora temos as colunas `cod_ibge`, `regiao` e `uf` em todos os anos.

Por fim, se quisermos exportar os dados em um arquivo csv para cada ano, podemos usar mais uma incrível função do `{purrr}`: a `walk()`. `walk()` funciona de forma semelhante ao `map()`, passamos como argumento a lista e a função que desejamos aplicar nela, neste caso, a `write.csv2()`: 
 

```r
# com o pipe

morte_infantil_mesclado %>%
  names(.) %>%
  walk(~ write.csv2(
    morte_infantil_mesclado[[.]],
    file = glue("./dados-mesclados/{.}-mesclado.csv"),
    row.names = FALSE
  ))

# sem o pipe

walk(
  names(morte_infantil_mesclado),
  ~ write.csv2(
    morte_infantil_mesclado[[.]],
    file = glue("./dados-mesclados/{.}-mesclado.csv"),
    row.names = FALSE
  )
)
```

O caminho que passei no argumento `file` é relativo ao meu projeto e pode ser diferente no seu caso. Se for, é só substituir pelo caminho da pasta onde deseja salvar os arquivos exportados no seu computador. 

Se desejar "empilhar" todos os anos e exportar tudo em um único arquivo csv, use:


```r
map_dfr(morte_infantil, mesclar_dados) %>%
  write.csv2(file = "./dados-mesclados/mortes_infantis_mesclado.csv",
             row.names = FALSE)
```

Bom, por hoje só. O código completo dessa postagem está disponível no meu [GitHub](https://github.com/santhiago-cristiano/site-scripts/tree/main/2021/2021-07-16-leftjoin). Na segunda parte darei continuidade falando sobre o `inner_join()`, que nos permite mesclar dois ou mais conjuntos de dados a partir de uma ou mais colunas chave mantendo somente as linhas que é comum entre todos eles.

Até a próxima 😄.

# Saiba Mais

- https://readr.tidyverse.org/

- https://dplyr.tidyverse.org/

- https://dplyr.tidyverse.org/reference/join.html

- https://github.com/abjur/abjData

- https://purrr.tidyverse.org/

- https://glue.tidyverse.org/reference/glue.html

- https://tibble.tidyverse.org/

- https://magrittr.tidyverse.org/

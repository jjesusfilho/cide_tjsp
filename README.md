Tutorial para baixar processos de primeira instância do Tribunal de Justiça de São Paulo usando R.
================

Introdução
----------

Este tutorial mostra como criar uma sequência de números de processos judiciais de primeira instância de São Paulo e em seguida iniciar o download das informações processuais.

Para tanto, é necessário instalar e carregar alguns pacotes to CRAN e outros que estão somente no Github.

Pacotes necessários
===================

``` r
install.packages("devtools")
devtools::install_github("courtsbr/JurisMiner")
devtools::install_github("courtsbr/esaj")
devtools::install_github("jjesusfilho/tjsp")
```

``` r
library(JurisMiner)
library(tjsp)
library(fs)
library(esaj)
library(knitr)
```

\`\`\`

Sequência
---------

A função `cnj_sequencia` cria uma sequência de números. Devemos indicar o número inicial, o número final, o ano, o nível, no caso será sempre 8 para indicar que é estadual, o número da UF, para indicar que é o Estado de São Paulo e número do distribuidor. Você pode acessar essas informações nesses links: [Resolução 65/2008 CNJ](http://www.cnj.jus.br/images/stories/docs_cnj/resolucao/rescnj_65.pdf) e [Códigos das unidades judiciárias](http://www.cnj.jus.br/images/programas/numeracao-unica/tribunais-estaduais/foros-1.xls)

Abaixo criamos uma sequência dos 100 possíveis primeiros números recebidos pelos processos distribuídos. Muitos números, na verdade, não estão vinculados a processos algum por erro, porque foram destinados a outro procedimento ou porque são cartas precatórias.

``` r
sequencia<-cnj_sequencial(inicio=1,fim=100,ano=2016,nivel=8,uf=26,distribuidor=0050)

diretorio<-dir_create("data-raw/cpopg")
head(sequencia)
#> [1] "00000017320168260050" "00000025820168260050" "00000034320168260050"
#> [4] "00000042820168260050" "00000051320168260050" "00000069520168260050"
```

Baixandos as informações
========================

O procedimentos abaixo baixa os htmls com as informações processuais. Em seguida, verificamos o tamanho de cada arquivo. Se for em torno de 85K, significa que a página está vazia.

``` r
download_cpopg(sequencia,diretorio) 

kable(informacoes)
```

``` r
informacoes<-file_info(dir_ls(diretorio))

head(knitr::kable(informacoes[c(1,3)]),20)
#>  [1] "path                                                                                      size"
#>  [2] "------------------------------------------------------------------------------------  --------"
#>  [3] "data-raw/cpopg/20181126_00000017320168260050.html                                       82.81K"
#>  [4] "data-raw/cpopg/20181126_00000025820168260050.html                                      194.36K"
#>  [5] "data-raw/cpopg/20181126_00000034320168260050.html                                      177.31K"
#>  [6] "data-raw/cpopg/20181126_00000042820168260050.html                                      114.38K"
#>  [7] "data-raw/cpopg/20181126_00000051320168260050.html                                        85.3K"
#>  [8] "data-raw/cpopg/20181126_00000069520168260050.html                                        85.3K"
#>  [9] "data-raw/cpopg/20181126_00000078020168260050.html                                      220.18K"
#> [10] "data-raw/cpopg/20181126_00000086520168260050.html                                        85.3K"
#> [11] "data-raw/cpopg/20181126_00000095020168260050.html                                       186.3K"
#> [12] "data-raw/cpopg/20181126_00000103520168260050.html                                      163.76K"
#> [13] "data-raw/cpopg/20181126_00000112020168260050.html                                      211.46K"
#> [14] "data-raw/cpopg/20181126_00000120520168260050.html                                      177.02K"
#> [15] "data-raw/cpopg/20181126_00000138720168260050.html                                        85.3K"
#> [16] "data-raw/cpopg/20181126_00000147220168260050.html                                        85.3K"
#> [17] "data-raw/cpopg/20181126_00000155720168260050.html                                      126.11K"
#> [18] "data-raw/cpopg/20181126_00000164220168260050.html                                        85.3K"
#> [19] "data-raw/cpopg/20181126_00000172720168260050.html                                        85.3K"
#> [20] "data-raw/cpopg/20181126_00000181220168260050.html                                      143.71K"
```

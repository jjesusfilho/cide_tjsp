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
library(tidyverse)
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

Abaixo criamos uma sequência dos 101 possíveis primeiros números recebidos pelos processos distribuídos. Muitos números, na verdade, não estão vinculados a processos algum por erro, porque foram destinados a outro procedimento ou porque são cartas precatórias.

``` r
sequencia<-cnj_sequencial(inicio=1,fim=100,ano=2016,nivel=8,uf=26,distribuidor=0050)

diretorio<-dir_create("data-raw/cpopg")
head(sequencia)
#> [1] "00000017320168260050" "00000025820168260050" "00000034320168260050"
#> [4] "00000042820168260050" "00000051320168260050" "00000069520168260050"
```

Baixandos as informações
========================

O procedimentos abaixo baixa os htmls com as informações processuais. Em seguida, verificamos o tamanho de cada arquivo. Se o tamanho for em torno de 85K, significa que a página está vazia.

``` r
download_cpopg(sequencia,diretorio) 

kable(informacoes)
```

Abaixo as primeiras 10 linhas dos 101 supostos processos baixados. Como se pode verificar, há muitos números que não correspondem a nenhum processo.

``` r
informacoes<-file_info(dir_ls(diretorio))

knitr::kable(head(informacoes[c(1,3)],10))
```

| path                                               |    size|
|:---------------------------------------------------|-------:|
| data-raw/cpopg/20181126\_00000017320168260050.html |   82.8K|
| data-raw/cpopg/20181126\_00000025820168260050.html |  194.4K|
| data-raw/cpopg/20181126\_00000034320168260050.html |  177.3K|
| data-raw/cpopg/20181126\_00000042820168260050.html |  114.4K|
| data-raw/cpopg/20181126\_00000051320168260050.html |   85.3K|
| data-raw/cpopg/20181126\_00000069520168260050.html |   85.3K|
| data-raw/cpopg/20181126\_00000078020168260050.html |  220.2K|
| data-raw/cpopg/20181126\_00000086520168260050.html |   85.3K|
| data-raw/cpopg/20181126\_00000095020168260050.html |  186.3K|
| data-raw/cpopg/20181126\_00000103520168260050.html |  163.8K|

Inicialmente, excluímos as páginas vazias. O procedimento abaixo exclui essas páginas.

``` r
remover<-informacoes %>% 
         filter(size <= as_fs_bytes("86K")) %>% 
         pull("path")

file_delete(remover)
```

Como se pode verificar, dos 101 htmls baixados, 56 foram aproveitados. Mostraremos somente as primeiras 10 linhas.

``` r
informacoes<-file_info(dir_ls(diretorio))

glimpse(informacoes[1:10,c(1,3)])
#> Observations: 10
#> Variables: 2
#> $ path <fs::path> "data-raw/cpopg/20181126_00000017320168260050.html",...
#> $ size <fs::bytes> 82.8K, 194.4K, 177.3K, 114.4K, 85.3K, 85.3K, 220.2K...
```

Extração dos dados.
-------------------

Para visualizar um os htmls, você pode usar a seguinte função: `rstudioapi::viewer(informacoes$path[1])`. As funções abaixo extraem as informações procesuais de cada html.

``` r
ler_dados_cpopg(diretorio) %>% 
spread(variavel,valor) %>% 
kable()
#> 
 Progress: ───────────────────────────────────────────────────────────────────────────────────── 100%
```

|      processo| Área     | Assunto                                | Classe                                   | Controle    | Distribuição     | Juiz                                              | Local Físico  | Outros assuntos           | Outros números                                                                  | Processo                                     |
|-------------:|:---------|:---------------------------------------|:-----------------------------------------|:------------|:-----------------|:--------------------------------------------------|:--------------|:--------------------------|:--------------------------------------------------------------------------------|:---------------------------------------------|
|  2.582017e+13| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000192 | 12/02/2016 às 12 | CRISTINA ESCHER                                   | NA            | NA                        | NA                                                                              | 0000002-58.2016.8.26.0050                    |
|  3.432017e+13| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000258 | 19/02/2016 às 12 | Roseane Cristina de Aguiar Almeida                | NA            | NA                        | NA                                                                              | 0000003-43.2016.8.26.0050 Extinto            |
|  4.282017e+13| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000460 | 19/01/2016 às 12 | Paulo de Abreu Lorenzino                          | 07/11/2018 00 | NA                        | NA                                                                              | 0000004-28.2016.8.26.0050 Extinto            |
|  7.802017e+13| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000068 | 18/01/2016 às 16 | Sonia Nazaré Fernandes Fraga                      | NA            | NA                        | 0013239-64.2016.8.26.0502                                                       | 0000007-80.2016.8.26.0050 Suspenso           |
|  9.502017e+13| Criminal | Estelionato                            | Ação Penal - Procedimento Ordinário      | 2016/000408 | 11/03/2016 às 12 | Fernanda Galizia Noriega                          | NA            | NA                        | NA                                                                              | 0000009-50.2016.8.26.0050                    |
|  1.035202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000143 | 04/02/2016 às 11 | Carlos José Zulian                                | NA            | NA                        | NA                                                                              | 0000010-35.2016.8.26.0050                    |
|  1.120202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000168 | 02/02/2016 às 10 | Klaus Marouelli Arroyo                            | NA            | NA                        | NA                                                                              | 0000011-20.2016.8.26.0050 Extinto            |
|  1.205202e+14| Criminal | Crimes de Trânsito                     | Ação Penal - Procedimento Ordinário      | 2017/001225 | 14/07/2017 às 16 | Fernanda Galizia Noriega                          | NA            | NA                        | NA                                                                              | 0000012-05.2016.8.26.0050                    |
|  1.557202e+14| Criminal | Lesão Corporal                         | Termo Circunstanciado                    | 2016/000247 | 14/01/2016 às 12 | Paulo de Abreu Lorenzino                          | 05/08/2016 00 | NA                        | NA                                                                              | 0000015-57.2016.8.26.0050 Extinto            |
|  1.812202e+14| Criminal | Furto                                  | Ação Penal - Procedimento Ordinário      | 2016/000189 | 10/02/2016 às 15 | FERNANDO OLIVEIRA CAMARGO                         | NA            | NA                        | NA                                                                              | 0000018-12.2016.8.26.0050                    |
|  1.994202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000110 | 27/01/2016 às 11 | Érica Aparecida Ribeiro Lopes e Navarro Rodrigues | NA            | NA                        | NA                                                                              | 0000019-94.2016.8.26.0050                    |
|  2.079202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000157 | 03/02/2016 às 15 | Cecilia Pinheiro da Fonseca                       | NA            | NA                        | 0003487-11.2017.8.26.0154                                                       | 0000020-79.2016.8.26.0050                    |
|  2.164202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000097 | 21/01/2016 às 12 | Giovana Furtado de Oliveira                       | NA            | NA                        | 0022538-90.2016.8.26.0041, 0022584-79.2016.8.26.0041, 0022602-03.2016.8.26.0041 | 0000021-64.2016.8.26.0050                    |
|  2.334202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000335 | 04/03/2016 às 15 | Luciana Piovesan                                  | NA            | NA                        | NA                                                                              | 0000023-34.2016.8.26.0050                    |
|  2.504202e+14| Criminal | Extorsão                               | Ação Penal - Procedimento Ordinário      | 2016/000165 | 03/02/2016 às 13 | EVA LOBO CHAIB DIAS JORGE                         | NA            | NA                        | NA                                                                              | 0000025-04.2016.8.26.0050                    |
|  2.686202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000078 | 21/01/2016 às 13 | CRISTINA ESCHER                                   | NA            | NA                        | 0012054-16.2016.8.26.0041                                                       | 0000026-86.2016.8.26.0050                    |
|  2.856202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Ação Penal - Procedimento Ordinário      | 2016/000136 | 01/02/2016 às 10 | Eduardo Pereira Santos Junior                     | NA            | NA                        | 0012773-02.2018.8.26.0502                                                       | 0000028-56.2016.8.26.0050                    |
|  2.941202e+14| Criminal | Furto                                  | Ação Penal - Procedimento Ordinário      | 2016/001538 | 08/09/2016 às 16 | Luciane Jabur Mouchaloite Figueiredo              | NA            | NA                        | NA                                                                              | 0000029-41.2016.8.26.0050                    |
|  3.026202e+14| Criminal | Furto                                  | Ação Penal - Procedimento Ordinário      | 2016/000133 | 27/01/2016 às 16 | Benedito Roberto Garcia Pozzer                    | NA            | NA                        | NA                                                                              | 0000030-26.2016.8.26.0050 Extinto            |
|  3.111202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000101 | 20/01/2016 às 09 | Cynthia Torres Cristofaro                         | NA            | NA                        | NA                                                                              | 0000031-11.2016.8.26.0050 Em grau de recurso |
|  4.762202e+14| Criminal | Furto                                  | Ação Penal - Procedimento Ordinário      | 2016/000377 | 15/03/2016 às 18 | Adriana Costa                                     | NA            | NA                        | NA                                                                              | 0000047-62.2016.8.26.0050 Extinto            |
|  5.102202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000342 | 15/01/2016 às 16 | José Zoéga Coelho                                 | 12/04/2016 00 | NA                        | NA                                                                              | 0000051-02.2016.8.26.0050 Extinto            |
|  5.284202e+14| Criminal | Desacato                               | Termo Circunstanciado                    | 2016/000445 | 19/01/2016 às 12 | José Zoéga Coelho                                 | 08/03/2016 00 | NA                        | NA                                                                              | 0000052-84.2016.8.26.0050 Extinto            |
|  5.454202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000448 | 19/01/2016 às 12 | Paulo de Abreu Lorenzino                          | 15/04/2016 00 | NA                        | NA                                                                              | 0000054-54.2016.8.26.0050 Extinto            |
|  5.624202e+14| Criminal | Lesão Corporal                         | Termo Circunstanciado                    | 2016/000368 | 15/01/2016 às 17 | José Zoéga Coelho                                 | 23/08/2018 00 | NA                        | NA                                                                              | 0000056-24.2016.8.26.0050 Extinto            |
|  5.891202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000369 | 15/01/2016 às 17 | José Zoéga Coelho                                 | 20/04/2016 00 | NA                        | NA                                                                              | 0000058-91.2016.8.26.0050 Extinto            |
|  5.976202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000346 | 15/01/2016 às 16 | Paulo de Abreu Lorenzino                          | 29/04/2016 00 | NA                        | NA                                                                              | 0000059-76.2016.8.26.0050 Extinto            |
|  6.061202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000105 | 22/01/2016 às 17 | Renata William Rached Catelli                     | NA            | NA                        | 0022686-04.2016.8.26.0041                                                       | 0000060-61.2016.8.26.0050                    |
|  6.146202e+14| Criminal | Lesão Corporal                         | Termo Circunstanciado                    | 2016/000371 | 15/01/2016 às 17 | José Zoéga Coelho                                 | NA            | NA                        | NA                                                                              | 0000061-46.2016.8.26.0050                    |
|  6.231202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000348 | 15/01/2016 às 16 | Paulo de Abreu Lorenzino                          | 15/04/2016 00 | NA                        | NA                                                                              | 0000062-31.2016.8.26.0050 Extinto            |
|  6.316202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000347 | 15/01/2016 às 16 | José Zoéga Coelho                                 | 19/10/2018 00 | NA                        | NA                                                                              | 0000063-16.2016.8.26.0050 Extinto            |
|  6.498202e+14| Criminal | Resistência                            | Termo Circunstanciado                    | 2016/000262 | 14/01/2016 às 13 | Paulo de Abreu Lorenzino                          | 29/10/2018 00 | NA                        | NA                                                                              | 0000064-98.2016.8.26.0050 Extinto            |
|  6.668202e+14| Criminal | Furto Qualificado                      | Ação Penal - Procedimento Ordinário      | 2016/000169 | 02/02/2016 às 12 | Cynthia Torres Cristofaro                         | NA            | NA                        | 20/2016                                                                         | 0000066-68.2016.8.26.0050                    |
|  6.753202e+14| Criminal | Crimes contra as Marcas                | Termo Circunstanciado                    | 2016/000367 | 15/01/2016 às 17 | Paulo de Abreu Lorenzino                          | 06/02/2018 00 | NA                        | NA                                                                              | 0000067-53.2016.8.26.0050 Extinto            |
|  6.838202e+14| Criminal | Crimes contra a Propriedade Industrial | Termo Circunstanciado                    | 2016/000350 | 15/01/2016 às 16 | José Zoéga Coelho                                 | 14/05/2018 00 | NA                        | NA                                                                              | 0000068-38.2016.8.26.0050 Extinto            |
|  6.923202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000362 | 04/03/2016 às 18 | CLAUDIO JULIANO FILHO                             | NA            | NA                        | NA                                                                              | 0000069-23.2016.8.26.0050                    |
|  7.190202e+14| Criminal | Crimes Previstos no Estatuto do Idoso  | Ação Penal - Procedimento Sumaríssimo    | 2017/000390 | 07/03/2017 às 18 | ALESSANDRA TEIXEIRA MIGUEL                        | NA            | Contravenções Penais,Leve | NA                                                                              | 0000071-90.2016.8.26.0050                    |
|  7.445202e+14| Criminal | Furto                                  | Inquérito Policial                       | 2016/000111 | 27/01/2016 às 15 | VICENTE LUIZ ADUA                                 | NA            | NA                        | NA                                                                              | 0000074-45.2016.8.26.0050                    |
|  7.530202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000152 | 29/01/2016 às 16 | Luciane Jabur Mouchaloite Figueiredo              | NA            | NA                        | 0008457-14.2016.8.26.0502                                                       | 0000075-30.2016.8.26.0050                    |
|  7.615202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000106 | 21/01/2016 às 11 | José Fernandes Freitas Neto                       | NA            | NA                        | 0023544-35.2016.8.26.0041                                                       | 0000076-15.2016.8.26.0050                    |
|  7.797202e+14| Criminal | Furto Qualificado                      | Ação Penal - Procedimento Ordinário      | 2016/000114 | 22/01/2016 às 11 | Klaus Marouelli Arroyo                            | NA            | NA                        | 0104865-94.2018.8.26.0050                                                       | 0000077-97.2016.8.26.0050                    |
|  7.882202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000053 | 15/01/2016 às 13 | Helio Narvaez                                     | NA            | NA                        | 0016287-56.2016.8.26.0041                                                       | 0000078-82.2016.8.26.0050                    |
|  7.967202e+14| Criminal | Furto Qualificado                      | Ação Penal - Procedimento Ordinário      | 2017/000339 | 20/02/2017 às 18 | Antonio Carlos de Campos Machado Junior           | NA            | NA                        | NA                                                                              | 0000079-67.2016.8.26.0050                    |
|  8.137202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000134 | 28/01/2016 às 11 | FABIOLA OLIVEIRA SILVA                            | NA            | NA                        | NA                                                                              | 0000081-37.2016.8.26.0050                    |
|  8.307202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000138 | 29/01/2016 às 10 | Marcos Vieira de Morais                           | NA            | NA                        | NA                                                                              | 0000083-07.2016.8.26.0050                    |
|  8.489202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000065 | 18/01/2016 às 15 | Wendell Lopes Barbosa de Souza                    | NA            | NA                        | 0007873-69.2016.8.26.0041                                                       | 0000084-89.2016.8.26.0050 Suspenso           |
|  8.659202e+14| Criminal | Roubo                                  | Ação Penal - Procedimento Ordinário      | 2016/000148 | 29/01/2016 às 16 | FERNANDO OLIVEIRA CAMARGO                         | NA            | NA                        | 0016574-91.2016.8.26.0502                                                       | 0000086-59.2016.8.26.0050                    |
|  8.744202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Ação Penal - Procedimento Ordinário      | 2016/000230 | 19/02/2016 às 09 | Marcello Ovidio Lopes Guimarães                   | NA            | NA                        | NA                                                                              | 0000087-44.2016.8.26.0050 Suspenso           |
|  8.829202e+14| Criminal | Receptação                             | Ação Penal - Procedimento Ordinário      | 2016/000140 | 01/02/2016 às 14 | CRISTINA ESCHER                                   | NA            | NA                        | NA                                                                              | 0000088-29.2016.8.26.0050                    |
|  9.096202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000233 | 16/02/2016 às 12 | Benedito Roberto Garcia Pozzer                    | NA            | NA                        | 0009665-87.2018.8.26.0041                                                       | 0000090-96.2016.8.26.0050                    |
|  9.266202e+14| Criminal | Furto                                  | Ação Penal - Procedimento Ordinário      | 2016/000127 | 27/01/2016 às 17 | FERNANDO OLIVEIRA CAMARGO                         | NA            | NA                        | NA                                                                              | 0000092-66.2016.8.26.0050                    |
|  9.351202e+14| Criminal | Tráfico de Drogas e Condutas Afins     | Procedimento Especial da Lei Antitóxicos | 2016/000080 | 22/01/2016 às 11 | Vanessa Strenger                                  | NA            | NA                        | 0008281-60.2016.8.26.0041                                                       | 0000093-51.2016.8.26.0050 Suspenso           |
|  9.606202e+14| Criminal | Lesão Corporal                         | Termo Circunstanciado                    | 2016/000307 | 15/01/2016 às 13 | Paulo de Abreu Lorenzino                          | 18/04/2016 00 | NA                        | NA                                                                              | 0000096-06.2016.8.26.0050 Extinto            |
|  9.873202e+14| Criminal | Contravenções Penais                   | Termo Circunstanciado                    | 2016/000306 | 15/01/2016 às 13 | José Zoéga Coelho                                 | 16/01/2017 00 | NA                        | NA                                                                              | 0000098-73.2016.8.26.0050 Extinto            |
|  9.958202e+14| Criminal | Posse de Drogas para Consumo Pessoal   | Termo Circunstanciado                    | 2016/000380 | 15/01/2016 às 18 | Paulo de Abreu Lorenzino                          | 03/12/2016 00 | NA                        | NA                                                                              | 0000099-58.2016.8.26.0050 Extinto            |
|  1.004320e+15| Criminal | Contravenções Penais                   | Termo Circunstanciado                    | 2016/000308 | 15/01/2016 às 13 | Paulo de Abreu Lorenzino                          | 01/04/2016 00 | NA                        | NA                                                                              | 0000100-43.2016.8.26.0050 Extinto            |

Abaixo, podemos visualizar as partes. Igualmente, mostraremos somente as primeiras 10 linhas.

``` r
ler_partes(diretorio) %>% 
.[1:10,] %>% 
kable()
```

| processo                  | parte\_nome                               | parte |
|:--------------------------|:------------------------------------------|:------|
| 0000003-43.2016.8.26.0050 | Justiça Pública                           | Autor |
| 0000003-43.2016.8.26.0050 | Defensoria Pública do Estado de São Paulo | Def   |
| 0000004-28.2016.8.26.0050 | Justiça Pública                           | Autor |
| 0000010-35.2016.8.26.0050 | Justiça Pública                           | Autor |
| 0000010-35.2016.8.26.0050 | ALEX VILAS BOAS                           | Réu   |
| 0000011-20.2016.8.26.0050 | Justiça Pública                           | Autor |
| 0000011-20.2016.8.26.0050 | Defensoria Pública do Estado de São Paulo | Def   |
| 0000015-57.2016.8.26.0050 | Justiça Pública                           | Autor |
| 0000015-57.2016.8.26.0050 | ANTONIO CARLOS DOS SANTOS MALICIA         | Autor |
| 0000015-57.2016.8.26.0050 | Justiça Pública                           | Autor |

Por fim, podemos visualizar o andamento processual. Como o número de linhas é muito grande, iremos imprimir somente as 20 primeiras linhas do primeiro processo.

``` r
ler_movimentacao_cposg(diretorio) %>% 
.[1:20,2:3] %>% 
kable()
```

| data       | movimentacao                  |
|:-----------|:------------------------------|
| 26/09/2018 | Folha de Antecedentes Juntada |
| 08/06/2018 | Suspensão do Prazo            |

            Prazo referente ao usuário foi alterado para 15/08/2018 devido à alteração da tabela de feriados                                  

04/06/2018 Suspensão do Prazo

            Prazo referente ao usuário foi alterado para 13/08/2018 devido à alteração da tabela de feriados                                  

17/02/2018 Suspensão do Prazo

            Prazo referente ao usuário foi alterado para 07/08/2018 devido à alteração da tabela de feriados                                  

23/10/2017 Processo Desarquivado Com Reabertura
02/10/2017 Despacho

            Procedam as anotações necessárias.Fls. 254: Aguarde-se no arquivo o comparecimento do acusado.                                     

29/08/2017 Documento Juntado
29/08/2017 Certidão de Cartório Expedida

            Certidão - Genérica                                                                                           

29/08/2017 Conclusos para Decisão
24/07/2017 Mensagem Eletrônica (e-mail) Juntada
10/07/2017 Contramandado de Prisão Expedido

            Mandado nº: 050.2017/124180-2 Situação: Emitido em 07/07/2017 17:32:08 Local: Cartório da 4ª Vara Criminal 

10/07/2017 Ofício Urgente Expedido

            Ofício - Prestação de Informações em Agravo de Instrumento-Habeas Corpus-Mandado de Segurança                       

10/07/2017 Despacho

            Cumpra-se a liminar.Expeça-se contramandado de prisão, certificando-se que o acusado não se encontra preso por este Juízo.         

07/07/2017 Pedido de Informações Juntado
07/07/2017 Conclusos para Decisão
05/07/2017 Ofício Expedido

            Ofício - IIRGD - Decisão - Crime                                                                                            

05/07/2017 Réu revel citado por edital

            Determinada suspensão do processo e o curso do prazo prescricional pelo artigo 366 do CPP                                

05/07/2017 Certidão de Remessa da Intimação Para o Portal Eletrônico Expedida

            Certidão - Remessa da Intimação para o Portal Eletrônico                 

05/07/2017 Ato Ordinatório - Não Publicável

            Vista à Defensoria Pública.                                                                                

05/07/2017 Certidão de Remessa da Intimação Para o Portal Eletrônico Expedida

            Certidão - Remessa da Intimação para o Portal Eletrônico

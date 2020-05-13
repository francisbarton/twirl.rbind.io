---
title: Latest learnings
author: Francis Barton
date: '2020-04-06'
categories:
  - 'twirl'
slug: latest-learnings
menu: main
editor_options:
  chunk_output_type: inline
df_print: kable
output:
  html_document:
    keep_md: yes
---



### This week in R I learned that ...

Passing two vectors to `str_replace_all` as the `pattern` and `replacement` arguments does not work.
Not even if you use `purrr::map2`.

You have to pass them as a named list, where the pattern is the name and the replacement is the vector.

Let's have a look...



```r
library(jsonlite)
library(janitor)
```

```
## 
## Attaching package: 'janitor'
```

```
## The following objects are masked from 'package:stats':
## 
##     chisq.test, fisher.test
```

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(stringr)
library(purrr)
```

```
## 
## Attaching package: 'purrr'
```

```
## The following object is masked from 'package:jsonlite':
## 
##     flatten
```

```r
# import LA - LAD - NUTS3 lookup table
la_nuts_lookup_json <- "https://opendata.arcgis.com/datasets/9b4c94e915c844adb11e15a4b1e1294d_0.geojson"

# get sample data

la_nuts_lookup <- fromJSON(la_nuts_lookup_json) %>% 
  pluck("features", "properties") %>% 
  clean_names()

df_sample <- la_nuts_lookup %>% 
  select(1:2, 5:6) %>% 
  dplyr::filter(str_detect(nuts318nm, "^(Bristol|Bath|Herefordshire|Kingston|Kensington|Glo|York)"))

head(df_sample)
```

```
##     lad18cd                      lad18nm nuts318cd
## 1 E06000019     Herefordshire, County of     UKG11
## 2 E06000023             Bristol, City of     UKK11
## 3 E06000024               North Somerset     UKK12
## 4 E06000025        South Gloucestershire     UKK12
## 5 E06000022 Bath and North East Somerset     UKK12
## 6 E07000082                       Stroud     UKK13
##                                                                nuts318nm
## 1                                               Herefordshire, County of
## 2                                                       Bristol, City of
## 3 Bath and North East Somerset, North Somerset and South Gloucestershire
## 4 Bath and North East Somerset, North Somerset and South Gloucestershire
## 5 Bath and North East Somerset, North Somerset and South Gloucestershire
## 6                                                        Gloucestershire
```

```r
nuts_patterns <- df_sample %>% 
  pull(nuts318nm) %>% 
  unique() %>% 
  str_subset(., "^(Bristol|Kingston|Bath|Hereford|Kensington)")

nuts_repl <- c("Herefordshire", "Bristol", "BANES/N Somerset/S Glos", "Kensington & Chelsea and H'smith & Fulham", "Kingston upon Hull")
```


```r
df_sample %>% 
  str_replace_all(nuts_patterns, nuts_repl)
```

```
## Warning in stri_replace_all_regex(string, pattern,
## fix_replacement(replacement), : argument is not an atomic vector; coercing
```

```
## Warning in stri_replace_all_regex(string, pattern,
## fix_replacement(replacement), : longer object length is not a multiple of
## shorter object length
```

```
## [1] "c(\"E06000019\", \"E06000023\", \"E06000024\", \"E06000025\", \"E06000022\", \"E07000082\", \"E07000079\", \"E07000078\", \"E07000081\", \"E07000083\", \"E07000080\", \"E09000013\", \"E09000020\", \"E06000010\", \"E06000014\")"                                                                                                                                                                                                                                                                                                                                
## [2] "c(\"Herefordshire, County of\", \"Bristol\", \"North Somerset\", \"South Gloucestershire\", \"Bath and North East Somerset\", \"Stroud\", \"Cotswold\", \"Cheltenham\", \"Gloucester\", \"Tewkesbury\", \"Forest of Dean\", \"Hammersmith and Fulham\", \"Kensington and Chelsea\", \"Kingston upon Hull, City of\", \"York\")"                                                                                                                                                                                                                                    
## [3] "c(\"UKG11\", \"UKK11\", \"UKK12\", \"UKK12\", \"UKK12\", \"UKK13\", \"UKK13\", \"UKK13\", \"UKK13\", \"UKK13\", \"UKK13\", \"UKI33\", \"UKI33\", \"UKE11\", \"UKE21\")"                                                                                                                                                                                                                                                                                                                                                                                            
## [4] "c(\"Herefordshire, County of\", \"Bristol, City of\", \"Bath and North East Somerset, North Somerset and South Gloucestershire\", \"Bath and North East Somerset, North Somerset and South Gloucestershire\", \"Bath and North East Somerset, North Somerset and South Gloucestershire\", \"Gloucestershire\", \"Gloucestershire\", \"Gloucestershire\", \"Gloucestershire\", \"Gloucestershire\", \"Gloucestershire\", \"Kensington & Chelsea and H'smith & Fulham\", \"Kensington & Chelsea and H'smith & Fulham\", \"Kingston upon Hull, City of\", \n\"York\")"
## [5] "c(\"E06000019\", \"E06000023\", \"E06000024\", \"E06000025\", \"E06000022\", \"E07000082\", \"E07000079\", \"E07000078\", \"E07000081\", \"E07000083\", \"E07000080\", \"E09000013\", \"E09000020\", \"E06000010\", \"E06000014\")"
```

This error message:

> In `stri_replace_all_regex(string, pattern, fix_replacement(replacement)`,  :
  longer object length is not a multiple of shorter object length
  
really confused me, because I was certain that my two vectors nuts_patterns and nuts_repl were the same length.


```r
df_sample %>% 
  mutate_at(vars(nuts318nm), ~ str_replace_all(., nuts_patterns, nuts_repl))
```

```
##      lad18cd                      lad18nm nuts318cd
## 1  E06000019     Herefordshire, County of     UKG11
## 2  E06000023             Bristol, City of     UKK11
## 3  E06000024               North Somerset     UKK12
## 4  E06000025        South Gloucestershire     UKK12
## 5  E06000022 Bath and North East Somerset     UKK12
## 6  E07000082                       Stroud     UKK13
## 7  E07000079                     Cotswold     UKK13
## 8  E07000078                   Cheltenham     UKK13
## 9  E07000081                   Gloucester     UKK13
## 10 E07000083                   Tewkesbury     UKK13
## 11 E07000080               Forest of Dean     UKK13
## 12 E09000013       Hammersmith and Fulham     UKI33
## 13 E09000020       Kensington and Chelsea     UKI33
## 14 E06000010  Kingston upon Hull, City of     UKE11
## 15 E06000014                         York     UKE21
##                                                                 nuts318nm
## 1                                                           Herefordshire
## 2                                                                 Bristol
## 3                                                 BANES/N Somerset/S Glos
## 4  Bath and North East Somerset, North Somerset and South Gloucestershire
## 5  Bath and North East Somerset, North Somerset and South Gloucestershire
## 6                                                         Gloucestershire
## 7                                                         Gloucestershire
## 8                                                         Gloucestershire
## 9                                                         Gloucestershire
## 10                                                        Gloucestershire
## 11                                                        Gloucestershire
## 12                          Kensington & Chelsea and Hammersmith & Fulham
## 13                          Kensington & Chelsea and Hammersmith & Fulham
## 14                                            Kingston upon Hull, City of
## 15                                                                   York
```

It didn't even work with `map2`!:


```r
# df_sample %>% 
  # mutate_at(vars(nuts318nm), ~ map2(nuts_patterns, nuts_repl, ~ str_replace_all(., .x, .y)))
```

Then a bit of searching led me to a post which mentioned [this in the stringr reference](https://stringr.tidyverse.org/reference/str_replace.html):


```
# If you want to apply multiple patterns and replacements to the same
# string, pass a named vector to pattern.
fruits %>%
  str_c(collapse = "---") %>%
  str_replace_all(c("one" = "1", "two" = "2", "three" = "3"))
#> [1] "1 apple---2 pears---3 bananas"
```

Which meant I just had to do this:


```r
names(nuts_repl) <- nuts_patterns

df_sample %>% 
  mutate_at(vars(nuts318nm), ~ str_replace_all(., nuts_repl))
```

```
##      lad18cd                      lad18nm nuts318cd
## 1  E06000019     Herefordshire, County of     UKG11
## 2  E06000023             Bristol, City of     UKK11
## 3  E06000024               North Somerset     UKK12
## 4  E06000025        South Gloucestershire     UKK12
## 5  E06000022 Bath and North East Somerset     UKK12
## 6  E07000082                       Stroud     UKK13
## 7  E07000079                     Cotswold     UKK13
## 8  E07000078                   Cheltenham     UKK13
## 9  E07000081                   Gloucester     UKK13
## 10 E07000083                   Tewkesbury     UKK13
## 11 E07000080               Forest of Dean     UKK13
## 12 E09000013       Hammersmith and Fulham     UKI33
## 13 E09000020       Kensington and Chelsea     UKI33
## 14 E06000010  Kingston upon Hull, City of     UKE11
## 15 E06000014                         York     UKE21
##                                    nuts318nm
## 1                              Herefordshire
## 2                                    Bristol
## 3                    BANES/N Somerset/S Glos
## 4                    BANES/N Somerset/S Glos
## 5                    BANES/N Somerset/S Glos
## 6                            Gloucestershire
## 7                            Gloucestershire
## 8                            Gloucestershire
## 9                            Gloucestershire
## 10                           Gloucestershire
## 11                           Gloucestershire
## 12 Kensington & Chelsea and H'smith & Fulham
## 13 Kensington & Chelsea and H'smith & Fulham
## 14                        Kingston upon Hull
## 15                                      York
```







# Data manipulation


```r
library(knitr)
library(tidyverse)
#> Loading tidyverse: ggplot2
#> Loading tidyverse: tibble
#> Loading tidyverse: tidyr
#> Loading tidyverse: readr
#> Loading tidyverse: purrr
#> Loading tidyverse: dplyr
#> Conflicts with tidy packages ----------------------------------------------
#> filter(): dplyr, stats
#> lag():    dplyr, stats
library(PKPDmisc)
```

## DATA IMPORT


Objectives:

* Import datasets and documents
* Perform basic data manipulation upon importing the data.


```r
pk_data <- read_csv("../data/pk_data.csv")
#> Parsed with column specification:
#> cols(
#>   ID = col_integer(),
#>   TIME = col_double(),
#>   DV = col_character(),
#>   AMT = col_integer(),
#>   DOSE = col_integer(),
#>   FORM = col_character(),
#>   SEX = col_character(),
#>   WT = col_double(),
#>   AGE = col_integer(),
#>   RACE = col_character()
#> )
```


```r
head(pk_data)
#> # A tibble: 6 x 10
#>      ID  TIME     DV   AMT  DOSE  FORM    SEX    WT   AGE     RACE
#>   <int> <dbl>  <chr> <int> <int> <chr>  <chr> <dbl> <int>    <chr>
#> 1     1  0.00   <NA>   100   100    IV Female  56.8    28 Hispanic
#> 2     1  0.25 1273.5    NA   100    IV Female  56.8    28 Hispanic
#> 3     1  0.50 995.38    NA   100    IV Female  56.8    28 Hispanic
#> 4     1  1.00 1254.7    NA   100    IV Female  56.8    28 Hispanic
#> 5     1  2.00 1037.6    NA   100    IV Female  56.8    28 Hispanic
#> 6     1  3.00 1135.4    NA   100    IV Female  56.8    28 Hispanic
```


## DATA MANIPULATION

The goals of this section:

* Use data manipulation tools to prepare the dataset for analysis


1.  Rename "DV" column as "COBS"


```r
pk_data_cobs <- pk_data %>% rename(COBS = DV)
```


2. Perform the following tasks:

a. Ensure that the following columns are numeric and not text: TIME, COBS, WT, AGE, AMT and DOSEs
    

```r
glimpse(pk_data_cobs)
#> Observations: 1,200
#> Variables: 10
#> $ ID   <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2, 2, 2, ...
#> $ TIME <dbl> 0.00, 0.25, 0.50, 1.00, 2.00, 3.00, 4.00, 6.00, 8.00, 12....
#> $ COBS <chr> NA, "1273.5", "995.38", "1254.7", "1037.6", "1135.4", "10...
#> $ AMT  <int> 100, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, 100, NA,...
#> $ DOSE <int> 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 100, 10...
#> $ FORM <chr> "IV", "IV", "IV", "IV", "IV", "IV", "IV", "IV", "IV", "IV...
#> $ SEX  <chr> "Female", "Female", "Female", "Female", "Female", "Female...
#> $ WT   <dbl> 56.8, 56.8, 56.8, 56.8, 56.8, 56.8, 56.8, 56.8, 56.8, 56....
#> $ AGE  <int> 28, 28, 28, 28, 28, 28, 28, 28, 28, 28, 28, 28, 30, 30, 3...
#> $ RACE <chr> "Hispanic", "Hispanic", "Hispanic", "Hispanic", "Hispanic...
```


```r
unique_non_numerics(pk_data_cobs$COBS)
#> [1] "BQL"
```


b. Create a new column called BQLFLAG which takes a value of 
`0` if there is a numerical value in CObs and `1` if there is "BQL" in COBS.
    

```r
pk_data_cobs <- pk_data_cobs %>% 
  mutate(BQLFLAG = ifelse(is.na(COBS), 0, 
                          ifelse(COBS == "BQL", 1, 0)),
        NONNUMERICS = ifelse(COBS %in% unique_non_numerics(COBS), 1, 0),
        COBS = as_numeric(COBS))
#> Warning in as_numeric(COBS): NAs introduced by coercion
```

    c. Create a new column called "GENDER" where:
        i. Female = 0
        ii. Male = 1 
    d. Create a new column called RACEN where:
        i. Caucasian = 0
        ii. Asian = 1
        iii. Black = 2
        iv. Hispanic = 3


```r
pk_data_cobs <- pk_data_cobs %>%
  mutate(
      GENDER = factor(SEX, 
                      levels = c(0, 1), 
                      labels = c("Female", "Male")),
      RACEN = ifelse(RACE == "Caucasian", 0,
                       ifelse(RACE == "Asian", 1,
                              ifelse(RACE == "Black",2,
                                     ifelse(RACE == "Hispanic", 3, -99))))) 
```

f. Create a new column called "IDF" - unique subject ID as combination of formulation and ID 


```r
pk_data_cobs <- pk_data_cobs %>% mutate(IDF = paste(ID, FORM, sep = "-"))
```

g. Remove the following columns
    i. SEX
    ii. RACE
   

```r
pk_data_output <- pk_data_cobs %>% 
    select(-SEX, -RACE)
```


```r
head(pk_data_output) %>% kable()
```



 ID   TIME   COBS   AMT   DOSE  FORM      WT   AGE   BQLFLAG   NONNUMERICS  GENDER    RACEN  IDF  
---  -----  -----  ----  -----  -----  -----  ----  --------  ------------  -------  ------  -----
  1   0.00     NA   100    100  IV      56.8    28         0             0  NA            3  1-IV 
  1   0.25   1274    NA    100  IV      56.8    28         0             0  NA            3  1-IV 
  1   0.50    995    NA    100  IV      56.8    28         0             0  NA            3  1-IV 
  1   1.00   1255    NA    100  IV      56.8    28         0             0  NA            3  1-IV 
  1   2.00   1038    NA    100  IV      56.8    28         0             0  NA            3  1-IV 
  1   3.00   1135    NA    100  IV      56.8    28         0             0  NA            3  1-IV 

Save the above modifications as a new csv file


```r
write_csv(pk_data_output, "../data/pk_data_output.csv")
```

## Descriptive Statistics

1. show a summary for all demographic columns


```r
# single row per id
sid_pk_data <- pk_data_cobs %>%
  distinct(ID, .keep_all = TRUE) 

sid_pk_data %>%
  select(WT, AGE, RACE, SEX) %>%
  mutate(RACE = as.factor(RACE),
         SEX = as.factor(SEX)) %>%
  summary
#>        WT            AGE              RACE        SEX    
#>  Min.   :52.3   Min.   :20.0   Asian    : 8   Female:28  
#>  1st Qu.:58.5   1st Qu.:31.0   Black    :12   Male  :22  
#>  Median :64.0   Median :39.5   Caucasian:17              
#>  Mean   :64.1   Mean   :38.5   Hispanic :13              
#>  3rd Qu.:68.8   3rd Qu.:48.0                             
#>  Max.   :80.9   Max.   :59.0
```

2. Count the number of subjects in each "Race" category


```r
# fastest
sid_pk_data %>% count(RACE) %>% kable()
```



RACE          n
----------  ---
Asian         8
Black        12
Caucasian    17
Hispanic     13


```r
# more manual
sid_pk_data %>%
  group_by(RACE) %>%
  tally

# most manual but have more control over column names
sid_pk_data %>%
  group_by(RACE) %>%
  summarize(num_per_race = n()) %>%
  kable()
```


3. calculate the min, mean, and max values for WT, AGE:
    a. by Sex


```r
sid_pk_data %>%
  group_by(SEX) %>%
  summarize(
    WT_min = min(WT),
    WT_mean = mean(WT),
    WT_max = max(WT),
    AGE_min = min(AGE),
    AGE_mean = mean(AGE),
    AGE_max = max(AGE)
  ) %>%
  kable()
```



SEX       WT_min   WT_mean   WT_max   AGE_min   AGE_mean   AGE_max
-------  -------  --------  -------  --------  ---------  --------
Female      52.3      59.5     69.0        20       37.0        51
Male        64.3      70.0     80.9        28       40.5        59

there are also targeted verbs in the form `<verb>_at` that can specify what
columns to act on, and which functions to run


```r
sid_pk_data %>%
  group_by(SEX) %>%
  summarize_at(vars(WT, AGE), funs(min, mean, max)) %>%
  kable()
```



SEX       WT_min   AGE_min   WT_mean   AGE_mean   WT_max   AGE_max
-------  -------  --------  --------  ---------  -------  --------
Female      52.3        20      59.5       37.0     69.0        51
Male        64.3        28      70.0       40.5     80.9        59

    
4. What is the Average numbers samples(observations) per individual in this dataset. 
Hint: make sure you are *only* counting samples, rows with AMT values are not considered observations.


```r
# observations are those with NA AMT values
pk_data_cobs %>%
  filter(is.na(AMT)) %>%
  group_by(ID) %>%
  summarize(num_obs = n()) %>%
  # ungroup so no longer calculating by grouping variable ID
  ungroup %>%
  summarize(average_obs = mean(num_obs)) %>% kable()
```



 average_obs
------------
          22


5. Calculate the Mean, 5th, and 95th percentile concentration at each time point for each formulation and dose level. hint: you can use `?quantile` to calculate various quantiles


```r
pk_data_cobs %>%
    mutate(COBS = as.numeric(COBS)) %>%
    filter(!is.na(COBS)) %>%
    group_by(TIME, FORM, DOSE) %>%
    summarize(q05 = quantile(COBS, 0.05),
              q50 = quantile(COBS, 0.5),
              q95 = quantile(COBS, 0.95)) %>%
  arrange(FORM, DOSE, TIME) %>% 
  head %>% kable()
```



 TIME  FORM    DOSE    q05    q50    q95
-----  -----  -----  -----  -----  -----
 0.25  IV       100    823   1716   2751
 0.50  IV       100   1000   1537   2815
 1.00  IV       100   1071   1423   2629
 2.00  IV       100    750   1216   1980
 3.00  IV       100    789   1059   1803
 4.00  IV       100    561    909   1278


```r
devtools::session_info()
#> Session info -------------------------------------------------------------
#>  setting  value                       
#>  version  R version 3.4.0 (2017-04-21)
#>  system   x86_64, mingw32             
#>  ui       RTerm                       
#>  language (EN)                        
#>  collate  English_United States.1252  
#>  tz       Europe/Prague               
#>  date     2017-06-05
#> Packages -----------------------------------------------------------------
#>  package    * version  date       source                            
#>  assertthat   0.2.0    2017-04-11 CRAN (R 3.4.0)                    
#>  backports    1.1.0    2017-05-22 CRAN (R 3.4.0)                    
#>  base       * 3.4.0    2017-04-21 local                             
#>  bindr        0.1      2016-11-13 CRAN (R 3.4.0)                    
#>  bindrcpp   * 0.1      2016-12-11 CRAN (R 3.4.0)                    
#>  bookdown     0.4      2017-05-20 CRAN (R 3.4.0)                    
#>  broom        0.4.2    2017-02-13 CRAN (R 3.4.0)                    
#>  cellranger   1.1.0    2016-07-27 CRAN (R 3.4.0)                    
#>  codetools    0.2-15   2016-10-05 CRAN (R 3.4.0)                    
#>  colorspace   1.3-2    2016-12-14 CRAN (R 3.4.0)                    
#>  compiler     3.4.0    2017-04-21 local                             
#>  datasets   * 3.4.0    2017-04-21 local                             
#>  devtools     1.13.1   2017-05-13 CRAN (R 3.4.0)                    
#>  digest       0.6.12   2017-01-27 CRAN (R 3.4.0)                    
#>  dplyr      * 0.6.0    2017-06-02 Github (tidyverse/dplyr@b064c4b)  
#>  evaluate     0.10     2016-10-11 CRAN (R 3.4.0)                    
#>  forcats      0.2.0    2017-01-23 CRAN (R 3.4.0)                    
#>  foreign      0.8-67   2016-09-13 CRAN (R 3.4.0)                    
#>  ggplot2    * 2.2.1    2016-12-30 CRAN (R 3.4.0)                    
#>  glue         1.0.0    2017-04-17 CRAN (R 3.4.0)                    
#>  graphics   * 3.4.0    2017-04-21 local                             
#>  grDevices  * 3.4.0    2017-04-21 local                             
#>  grid         3.4.0    2017-04-21 local                             
#>  gtable       0.2.0    2016-02-26 CRAN (R 3.4.0)                    
#>  haven        1.0.0    2016-09-23 CRAN (R 3.4.0)                    
#>  highr        0.6      2016-05-09 CRAN (R 3.4.0)                    
#>  hms          0.3      2016-11-22 CRAN (R 3.4.0)                    
#>  htmltools    0.3.6    2017-04-28 CRAN (R 3.4.0)                    
#>  httr         1.2.1    2016-07-03 CRAN (R 3.4.0)                    
#>  jsonlite     1.5      2017-06-01 CRAN (R 3.4.0)                    
#>  knitr      * 1.16     2017-05-18 CRAN (R 3.4.0)                    
#>  lattice      0.20-35  2017-03-25 CRAN (R 3.4.0)                    
#>  lazyeval     0.2.0    2016-06-12 CRAN (R 3.4.0)                    
#>  lubridate    1.6.0    2016-09-13 CRAN (R 3.4.0)                    
#>  magrittr     1.5      2014-11-22 CRAN (R 3.4.0)                    
#>  memoise      1.1.0    2017-04-21 CRAN (R 3.4.0)                    
#>  methods      3.4.0    2017-04-21 local                             
#>  mnormt       1.5-5    2016-10-15 CRAN (R 3.4.0)                    
#>  modelr       0.1.0    2016-08-31 CRAN (R 3.4.0)                    
#>  munsell      0.4.3    2016-02-13 CRAN (R 3.4.0)                    
#>  nlme         3.1-131  2017-02-06 CRAN (R 3.4.0)                    
#>  parallel     3.4.0    2017-04-21 local                             
#>  PKPDmisc   * 1.0.0    2017-06-02 Github (dpastoor/PKPDmisc@23e1f49)
#>  plyr         1.8.4    2016-06-08 CRAN (R 3.4.0)                    
#>  psych        1.7.5    2017-05-03 CRAN (R 3.4.0)                    
#>  purrr      * 0.2.2.2  2017-05-11 CRAN (R 3.4.0)                    
#>  R6           2.2.1    2017-05-10 CRAN (R 3.4.0)                    
#>  Rcpp         0.12.11  2017-05-22 CRAN (R 3.4.0)                    
#>  readr      * 1.1.1    2017-05-16 CRAN (R 3.4.0)                    
#>  readxl       1.0.0    2017-04-18 CRAN (R 3.4.0)                    
#>  reshape2     1.4.2    2016-10-22 CRAN (R 3.4.0)                    
#>  rlang        0.1.1    2017-05-18 CRAN (R 3.4.0)                    
#>  rmarkdown    1.5.9000 2017-06-03 Github (rstudio/rmarkdown@ea515ef)
#>  rprojroot    1.2      2017-01-16 CRAN (R 3.4.0)                    
#>  rvest        0.3.2    2016-06-17 CRAN (R 3.4.0)                    
#>  scales       0.4.1    2016-11-09 CRAN (R 3.4.0)                    
#>  stats      * 3.4.0    2017-04-21 local                             
#>  stringi      1.1.5    2017-04-07 CRAN (R 3.4.0)                    
#>  stringr      1.2.0    2017-02-18 CRAN (R 3.4.0)                    
#>  tibble     * 1.3.3    2017-05-28 CRAN (R 3.4.0)                    
#>  tidyr      * 0.6.3    2017-05-15 CRAN (R 3.4.0)                    
#>  tidyverse  * 1.1.1    2017-01-27 CRAN (R 3.4.0)                    
#>  tools        3.4.0    2017-04-21 local                             
#>  utils      * 3.4.0    2017-04-21 local                             
#>  withr        1.0.2    2016-06-20 CRAN (R 3.4.0)                    
#>  xml2         1.1.1    2017-01-24 CRAN (R 3.4.0)                    
#>  yaml         2.1.14   2016-11-12 CRAN (R 3.4.0)
```


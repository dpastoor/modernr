
# dplyr data manipulation



```r
library(PKPDmisc)
library(knitr)
library(tidyverse)
```

Objectives:

* Import datasets and documents
* Perform basic data manipulation upon importing the data.

### Task-I

Use the .csv files `demog`, `IV`, and `Oral` provided into the data object folder. 

1. Read in all three csv files  and give them descriptive names (not data1, data2, data3)


```r
demog <- read_csv("../data/demog.csv")
#> Parsed with column specification:
#> cols(
#>   ID = col_integer(),
#>   SEX = col_character(),
#>   WT = col_double(),
#>   AGE = col_integer(),
#>   RACE = col_character()
#> )
iv_data <- read_csv("../data/IV.csv")
#> Parsed with column specification:
#> cols(
#>   ID = col_integer(),
#>   TIME = col_double(),
#>   DV = col_character(),
#>   AMT = col_integer(),
#>   DOSE = col_integer()
#> )
oral_data <- read_csv("../data/ORAL.csv")
#> Parsed with column specification:
#> cols(
#>   ID = col_integer(),
#>   TIME = col_double(),
#>   DV = col_character(),
#>   AMT = col_integer(),
#>   DOSE = col_integer()
#> )
```

## DATA MANIPULATION
The goals of this section:

* Use data manipulation tools to prepare the dataset for analysis

### Task-II
1.  Rename "DV" column as "COBS"


```r
iv_data <- iv_data %>% rename(COBS = DV)
oral_data <- oral_data %>% rename(COBS = DV)
```

2. Add a Formulation column and label IV/Oral for each dataset


```r
iv_data <- iv_data %>% mutate(FORM = "IV")
oral_data <- oral_data %>% mutate(FORM = "ORAL")
```

3. Appropriately merge the demographics dataset into the IV and Oral dataset
4. Create one integrated dataset with both IV and Oral data.


```r
combined_data <- bind_rows(iv_data, oral_data)

## check to see if any ids not in the other
anti_join(combined_data, demog)
#> Joining, by = "ID"
#> # A tibble: 0 × 6
#> # ... with 6 variables: ID <int>, TIME <dbl>, COBS <chr>, AMT <int>,
#> #   DOSE <int>, FORM <chr>
anti_join(demog, combined_data)
#> Joining, by = "ID"
#> # A tibble: 2 × 5
#>      ID    SEX    WT   AGE      RACE
#>   <int>  <chr> <dbl> <int>     <chr>
#> 1    51   Male    60    28 Caucasian
#> 2    52 Female    70    33     Asian
```

Two individuals do not have any concentration-time data


```r
all_data <- left_join(combined_data, demog)
#> Joining, by = "ID"
```


5. Perform the following tasks:
    a. Ensure that the following columns are numeric and not text: TIME, COBS, WT, AGE, AMT and DOSEs


```r
all_data %>% select(TIME, COBS, WT, AGE, AMT, DOSE) %>% str
#> Classes 'tbl_df', 'tbl' and 'data.frame':	1200 obs. of  6 variables:
#>  $ TIME: num  0 0.25 0.5 1 2 3 4 6 8 12 ...
#>  $ COBS: chr  NA "1273.5" "995.38" "1254.7" ...
#>  $ WT  : num  56.8 56.8 56.8 56.8 56.8 56.8 56.8 56.8 56.8 56.8 ...
#>  $ AGE : int  28 28 28 28 28 28 28 28 28 28 ...
#>  $ AMT : int  100 NA NA NA NA NA NA NA NA NA ...
#>  $ DOSE: int  100 100 100 100 100 100 100 100 100 100 ...
```

COBS is a character column, therefore want to find out what character values exist


```r
# check what character values are present
unique_non_numerics(all_data$COBS)
#> [1] "BQL"
```

    b. Change the following:
    c. Create a new column called BQLFLAG which takes a value of "0" if there is a numerical value in CObs and "1" if there is "BQL" in CObs.
    

```r
# if don't manually specify to handle NA COBS, will also get NA values for BQLFLAG
all_data <- all_data %>% mutate(BQLFLAG = ifelse(is.na(COBS), 0, 
                                                 ifelse(COBS == "BQL", 1, 0)),
                                COBS = as_numeric(COBS))
#> Warning in as_numeric(c(NA, "1273.5", "995.38", "1254.7", "1037.6",
#> "1135.4", : NAs introduced by coercion
```


```r
all_data %>% head %>% kable
```



 ID   TIME   COBS   AMT   DOSE  FORM   SEX         WT   AGE  RACE        BQLFLAG
---  -----  -----  ----  -----  -----  -------  -----  ----  ---------  --------
  1   0.00     NA   100    100  IV     Female    56.8    28  Hispanic          0
  1   0.25   1274    NA    100  IV     Female    56.8    28  Hispanic          0
  1   0.50    995    NA    100  IV     Female    56.8    28  Hispanic          0
  1   1.00   1255    NA    100  IV     Female    56.8    28  Hispanic          0
  1   2.00   1038    NA    100  IV     Female    56.8    28  Hispanic          0
  1   3.00   1135    NA    100  IV     Female    56.8    28  Hispanic          0

```r
all_data %>% filter(BQLFLAG ==1) %>% kable
```



 ID   TIME   COBS   AMT   DOSE  FORM   SEX       WT   AGE  RACE     BQLFLAG
---  -----  -----  ----  -----  -----  -----  -----  ----  ------  --------
 20     24     NA    NA    100  IV     Male    80.9    31  Asian          1
 20     24     NA    NA    100  ORAL   Male    80.9    31  Asian          1

    d. Filter the dataset such that you remove all rows where BQLFLAG=1
        i. WT from lb to kg 
        iv. CObs from μg/mL to μg/L


```r
f_all_data <- all_data %>% filter(BQLFLAG != 1)
f_all_data_adjunits <- f_all_data %>% mutate(WT = WT/2.2,
                                             COBS = COBS*1000)
```


```r
f_all_data_adjunits %>% head %>% kable
```



 ID   TIME      COBS   AMT   DOSE  FORM   SEX         WT   AGE  RACE        BQLFLAG
---  -----  --------  ----  -----  -----  -------  -----  ----  ---------  --------
  1   0.00        NA   100    100  IV     Female    25.8    28  Hispanic          0
  1   0.25   1273500    NA    100  IV     Female    25.8    28  Hispanic          0
  1   0.50    995380    NA    100  IV     Female    25.8    28  Hispanic          0
  1   1.00   1254700    NA    100  IV     Female    25.8    28  Hispanic          0
  1   2.00   1037600    NA    100  IV     Female    25.8    28  Hispanic          0
  1   3.00   1135400    NA    100  IV     Female    25.8    28  Hispanic          0

    e. Create a new column called "GENDER" where:
        i. Female = 0
        ii. Male = 1 
    f. Create a new column called RACEN where:
        i. Caucasian = 0
        ii. Asian = 1
        iii. Black = 2
        iv. Hispanic = 3
    g. Create a new column called "LOGCOBS" where CObs is in the log scale
    h. Create a new column called "USUBJID" - unique subject ID as combination of formulation and ID (hint check out `?interaction`)
    
    i. Remove the following columns
    i. SEX
    ii. RACE


```r
final_data <- f_all_data_adjunits %>% mutate(
  GENDER = ifelse(SEX == "Female", 0, 1),
  RACEN = as.numeric(factor(RACE, levels = c("Caucasian", "Asian", "Black", "Hispanic"))),
  LOGCOBS = log(COBS),
  USUBJID = interaction(ID, FORM)
) %>% select(-SEX, -RACE)
```



6. Save the above modifications as a new csv file


```r
write_csv(final_data, "iv_oral_alldat.csv", na = ".")
```

## Descriptive Statistics

Objectives

* How to make summaries of the data using descriptive statistics and other data manipulation tools (dplyr, base R functions etc)

### Task III


1. show a summary for all demographic columns


```r
final_data <- final_data %>% 
  mutate(GENDER = as.factor(GENDER),
         RACEN = as.factor(RACEN))
uid_final_data <- final_data %>% distinct(ID, .keep_all = TRUE)

uid_final_data %>% 
  select(WT, AGE, GENDER, RACEN) %>%
 summary %>% kable
```

           WT            AGE       GENDER   RACEN 
---  -------------  -------------  -------  ------
     Min.   :23.8   Min.   :20.0   0:28     1:17  
     1st Qu.:26.6   1st Qu.:31.0   1:22     2: 8  
     Median :29.1   Median :39.5   NA       3:12  
     Mean   :29.1   Mean   :38.5   NA       4:13  
     3rd Qu.:31.3   3rd Qu.:48.0   NA       NA    
     Max.   :36.8   Max.   :59.0   NA       NA    


2. Count the number of males/females in the dataset

```r
# be careful only 1 row per id if calculating this way
uid_final_data %>% nrow
#> [1] 50
# or
n_distinct(uid_final_data$ID)
#> [1] 50
```


3. Count the number of subjects in each "Race" category


```r
uid_final_data %>%  
  group_by(RACEN) %>% 
  tally
#> # A tibble: 4 × 2
#>    RACEN     n
#>   <fctr> <int>
#> 1      1    17
#> 2      2     8
#> 3      3    12
#> 4      4    13
```

4. calculate the min, mean, and max values for WT, AGE:
    a. by Gender

```r
uid_final_data %>% 
  select(GENDER, WT, AGE) %>%
  group_by(GENDER) %>% 
  summarize_all(funs(min, mean, max)) %>% 
  kable
```



GENDER    WT_min   AGE_min   WT_mean   AGE_mean   WT_max   AGE_max
-------  -------  --------  --------  ---------  -------  --------
0           23.8        20      27.0       37.0     31.4        51
1           29.2        28      31.8       40.5     36.8        59

    b. by Race
    

```r
uid_final_data %>% select(RACEN, WT, AGE) %>%
  group_by(RACEN) %>% 
  summarize_all(funs(min, mean, max)) %>% 
  kable
```



RACEN    WT_min   AGE_min   WT_mean   AGE_mean   WT_max   AGE_max
------  -------  --------  --------  ---------  -------  --------
1          23.8        20      28.3       40.1     35.5        51
2          24.1        22      29.4       36.1     36.8        50
3          23.9        26      29.1       36.0     35.0        51
4          25.8        22      30.0       40.2     33.7        59

5. What is the Average numbers samples(observations) per individual in this dataset. Hint: make sure you are *only* counting samples, not necessarily all rows are observations!


```r
# don't want dosing observations
final_data %>% filter(is.na(AMT)) %>% group_by(ID) %>% 
  summarize(num_obs = n()) %>%
  summarize(avg_samples = mean(num_obs))
#> # A tibble: 1 × 1
#>   avg_samples
#>         <dbl>
#> 1          22
```


6. Calculate the Mean, 5th, and 95th percentile concentration at each time point for each formulation and dose level. hint: you can use `?quantile` to calculate various quantiles


```r
final_data %>%
  group_by(TIME) %>% 
  s_quantiles(COBS, probs = c(0.05, 0.5, 0.95)) %>% 
  kable
```



  TIME   COBS_q5   COBS_q50   COBS_q95
------  --------  ---------  ---------
  0.00        NA         NA         NA
  0.25    179528    1013450    6299400
  0.50    315901    1339500    6196680
  1.00    516881    1602900    4941020
  2.00    661580    1556600    4623085
  3.00    609477    1407150    4218805
  4.00    538884    1237250    3752430
  6.00    350257     882890    2881720
  8.00    170944     736590    2139750
 12.00     86539     372920    1449365
 16.00     28623     198495     987036
 24.00      3748      81368     550874



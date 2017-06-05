
## Diagnostic Plots

1) read in the csv datasets:

* EtaCov_gathered
* Residuals
* Theta


```r
library(PKPDmisc)
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
```


```r
resid <- read_phx("../data/Residuals.csv")
theta <- read_phx("../data/Theta.csv")
etacov_gathered <- read_phx("../data/EtaCov_gathered.csv")
```

2) From the Theta table, create a reasonable quality output table of the results.
Hint, use knitr::kable, in combination with results='asis' in the chunk settings

requires names:


```r
theta %>% 
  select(-one_of(c("Scenario", "Var. Inf. factor"))) %>% 
  kable(digits = 2)
```



Parameter    Estimate  Units    Stderr      CV%   2.5% CI   97.5% CI
----------  ---------  ------  -------  -------  --------  ---------
tvKa             0.39  1/hr       0.02     4.05      0.36       0.42
tvV              2.94             0.05     1.83      2.83       3.04
tvCl             0.08             0.00     1.80      0.08       0.08
dVdWT            1.00             0.00     0.00      1.00       1.00
dCldAGE         -0.87             0.11   -12.26     -1.09      -0.66
stdev0           0.10             0.00     2.80      0.09       0.10


* clean up columns
* clean up column names
* units

3) Create a CWRES vs Time plot with loess fits for the central tendency and the spread (hint abs() is your friend for the spread)


```r
gg_cwres_tad <- function(df) {
df %>%
  ggplot(aes(x = TAD, y = CWRES)) + geom_point() +
  stat_smooth(method = "loess", se=F, color = "red") +
  stat_smooth(data = df %>%
                mutate(CWRES = abs(CWRES)), 
              se = F, color = "blue") +
  stat_smooth(data = df %>%
                mutate(CWRES = -abs(CWRES)), 
              se = F, color = "blue") +
    theme_bw() +
    base_theme()
}
```




```r
gg_cwres_tad(resid)
#> `geom_smooth()` using method = 'loess'
#> `geom_smooth()` using method = 'loess'
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-6-1.png" width="672" />





4) update the CWRES vs Time plot to flag anything with CWRES > 2.5 as a red value

```r

resid %>% 
  mutate(HIGHCWRES = ifelse(abs(CWRES) > 2.5, 1, 0)) %>%
    ggplot(aes(x = TAD, y = CWRES)) +
  geom_point(aes(color = factor(HIGHCWRES))) +
  scale_color_manual(values = c("black", "red"), name = "Outlier", labels = c("not outlier", "outlier")) +
  stat_smooth(method = "loess") +
  stat_smooth(data = resid %>%
                mutate(CWRES = abs(CWRES)), 
              method="loess", color = "red", se = F) +
  stat_smooth(data = resid %>%
                mutate(CWRES = -abs(CWRES)), 
              method="loess", color = "red", se = F) 
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-7-1.png" width="672" />

5) print a table of key information for all points with CWRES > 2.5


```r
resid %>% 
  mutate(HIGHCWRES = ifelse(abs(CWRES) > 2.5, 1, 0)) %>%
  filter(HIGHCWRES ==1) %>% select(ID, IVAR, TAD, IPRED, DV) %>% kable(digits = 2)
```



 ID   IVAR   TAD   IPRED      DV
---  -----  ----  ------  ------
  4    364    28   28.93   18.62
  4    400    64   11.92   13.73
  5     48     0    7.26    8.12
  9    352    16   39.54   27.48
 36      3     3   23.60   17.10
 36    364    28   18.01   22.57


6) Plot individual IPRED and DV vs time


```r
split_resid <- resid %>% filter(TADSeq ==1) %>% mutate(IDBINS = ids_per_plot(ID, 9)) %>% split(.[["IDBINS"]])

p <- function(df) {
  df %>%
  ggplot(aes(x = TAD, y = IPRED, group= TADSeq)) +
  geom_line() + facet_wrap(~ID) + theme_bw() +
    geom_point(aes(x = TAD, y = DV))+
    labs(list(x = "Time after Dose, hrs",
              y = "Individual Predicted and Observed")) 
}
split_resid %>% map(p)
#> $`1`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-1.png" width="672" />

```
#> 
#> $`2`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-2.png" width="672" />

```
#> 
#> $`3`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-3.png" width="672" />

```
#> 
#> $`4`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-4.png" width="672" />

```
#> 
#> $`5`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-5.png" width="672" />

```
#> 
#> $`6`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-9-6.png" width="672" />

As a reminder, map works like lapply, it applies the same function to each element in the list. In this case, it is taking split_resid (which is the residual dataframe split by 9 ids per group) and then applies the plot function to each set of 9.


6b) add the population prediction as a dashed blue line


```r

p <- function(df) {
  df %>%
  ggplot(aes(x = TAD, y = IPRED, group= TADSeq)) +
  geom_line() + facet_wrap(~ID) + theme_bw() +
    geom_point(aes(x = TAD, y = DV))+labs(list(x = "Time after Dose, hrs", y = "Individual Predicted and Observed")) +
    geom_line(aes(x = TAD, y = PRED, group = TADSeq), color = "blue")
}

split_resid %>% map(p)
#> $`1`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-1.png" width="672" />

```
#> 
#> $`2`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-2.png" width="672" />

```
#> 
#> $`3`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-3.png" width="672" />

```
#> 
#> $`4`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-4.png" width="672" />

```
#> 
#> $`5`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-5.png" width="672" />

```
#> 
#> $`6`
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-10-6.png" width="672" />

7) With EtaCov_final create histograms of all the eta distributions


```r

p_etas<- etacov_gathered %>%
  ggplot(aes(x = VALUE, group = ETA)) + 
  geom_histogram(fill = "white", color = "black") + 
  facet_wrap(~ETA, scales = "free") + base_theme()

p_etas
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-11-1.png" width="672" />

add a mean value for each eta overlaid on the above plot

```r
mean_eta <- etacov_gathered %>% 
    group_by(ETA) %>%
  summarize(meanEta = mean(VALUE))

p_etas + 
  geom_vline(data = mean_eta, aes(xintercept = meanEta), size = 1.5, color = "red")
#> `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-12-1.png" width="672" />

8) Create Eta vs Covariate plots for each covariate and all etas


```r
etacov_gathered %>%
    ggplot(aes(x = WT, y = VALUE, group = ETA)) + 
  geom_point() + facet_wrap(~ETA, scales = "free") + 
  stat_smooth(method = "loess", color = "blue", se = F, size = 1.3) + 
  base_theme()
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-13-1.png" width="672" />

```r

etacov_gathered %>%
    ggplot(aes(x = AGE, y = VALUE, group = ETA)) + 
  geom_point() + facet_wrap(~ETA, scales = "free") +
  stat_smooth(method = "loess", color = "blue", se = F, size = 1.3) + base_theme()
```

<img src="diagnostic-plots_legacy_files/figure-html/unnamed-chunk-13-2.png" width="672" />

Note in the plot above, the choice of facet_wrap was arbitrary, and potentially a cleaner looking plot can be created with facet_grid, especially for labels, my suggestion is to try both.

Hint: since there is so much duplicated, this would be a good opportunity to turn that into a function that you pass in the covariate to plot for `x`.

9) add loess fits to the eta cov plots

done in above plots

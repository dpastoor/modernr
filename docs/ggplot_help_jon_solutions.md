
# advanced ggplot customizations

Help! Your colleague Jon has come to you for help. He is just starting to use ggplot and is having trouble. Thankfully, he has gotten started on making the necessary plots, and has a good idea what he wants. Your job, should you choose to accept it, is to help finish off the plots Jon has started. 


Jon has been kind enough to provide you with a zipped R project. You can unzip the project and click on the .Rproj to open up the project to get you started. 




```r
library("dplyr")
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
library("ggplot2")
library("knitr")
library("PKPDdatasets")
library("PKPDmisc")
               
opts_chunk$set(cache=T, fig.width=9)
```

The data Jon is working with conventiently comes from the `dapa_iv_oral` dataset in the `PKPDdatasets` package.

Jon's first attempt to make a concentration time plot for each ID looks funny. 


```r
oral_data <- dapa_IV_oral %>% filter(FORMULATION == "ORAL")
```


```r
kable(head(oral_data))
```



 ID   TIME    TAD   COBS   AMT_IV   AMT_ORAL   OCC   AGE   WEIGHT  GENDER   FORMULATION 
---  -----  -----  -----  -------  ---------  ----  ----  -------  -------  ------------
  1    168   0.00    0.0        0       5000     2    44     70.5  0        ORAL        
  1    168   0.05   13.7        0          0     2    44     70.5  0        ORAL        
  1    168   0.35   62.3        0          0     2    44     70.5  0        ORAL        
  1    168   0.50   67.9        0          0     2    44     70.5  0        ORAL        
  1    169   0.75   66.3        0          0     2    44     70.5  0        ORAL        
  1    169   1.00   86.3        0          0     2    44     70.5  0        ORAL        


```r
ggplot(oral_data, aes(x = TAD, y = COBS, group = ID, color = OCC)) + geom_line() + 
  facet_wrap(~ID)
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-4-1.png" width="672" />

You will need to help him adjust:

* fix lines (hint - check out `interaction`)
* fix line color to be discrete
* rename axes
* change legend name
* adjust scale
* adjust axis labels and numbers for text color and size
* adjust the output width so the x-axis numbers don't overlap

### to get a final plot that looks like this:


```r
ggplot(oral_data, aes(x = TAD, y = COBS, 
                      group = interaction(ID, OCC), 
                      color = factor(OCC))) + 
  geom_line(size = 1.05) + 
  facet_wrap(~ID) + base_theme() +
  xlab("Time After Dose, hours") +
  ylab("Concentration, ug/mL") +
  scale_color_discrete(name="Occasion") + scale_y_log10()
#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-5-1.png" width="672" />



Jon now wants to get a general feel for the covariate weight, and thus wants to color by weight.


```r
ggplot(oral_data, aes(x = TAD, y = COBS, 
                      group = ID)) + 
  geom_line(size = 1.05) + 
  facet_wrap(~OCC) + base_theme() +
  xlab("Time After Dose, hours") +
  ylab("Concentration, ug/mL") +
 scale_y_log10()
#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-6-1.png" width="672" />

He needs your help

* fixing the facet strips to be better labeled
* add the color to weight
* getting the plots to be row-wise rather than side-by-side



### so it will look like this:


```r
occ_labels <- list('1' = "5 mg IV", 
                   '2'= "5 mg",
                   '3' = "10 mg",
                   '4' = "25 mg")
occ_labeller <- function(variable,value){
  return(occ_labels[value])
}

ct_colWT <- ggplot(oral_data, aes(x = TAD, y = COBS, 
                      group = interaction(ID, OCC), 
                      color = WEIGHT)) + 
  geom_line(size = 1.05) + 
 base_theme() +
  xlab("Time After Dose, hours") +
  ylab("Concentration, ug/mL") +
 scale_y_log10() 
ct_colWT + facet_grid(OCC~., labeller=occ_labeller)+ theme(strip.text = element_text(size = 16, color="black"))
#> Warning: The labeller API has been updated. Labellers taking `variable`and
#> `value` arguments are now deprecated. See labellers documentation.
#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-7-1.png" width="672" />


But just in case also wants to see the old side-by-side view as well. 

He needs your help

* change facetting
* move legend to be below the plot

### so it looks like this:


```r
ct_colWT + facet_grid(.~OCC, labeller=occ_labeller)+ 
  theme(strip.text = element_text(size = 16, color="black")) + theme(legend.position="bottom")
#> Warning: The labeller API has been updated. Labellers taking `variable`and
#> `value` arguments are now deprecated. See labellers documentation.
#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-8-1.png" width="672" />


Jon decided to look at the 5 mg dose. He needs help figuring out how to add mean lines. He wants to show that the general trend for males and females is similar and so would like to overlay the geometric mean profile for males and females on the concentration-time plot below.


```r
oral_data_occ2 <- oral_data %>% filter(OCC==2)

# calculate geometric mean here
```

He did a couple calculations by hand so you can check that the values are the same.

```r
mean_occ2 <- oral_data %>% filter(OCC==2) %>%
  group_by(GENDER, TAD) %>% summarize(meanCONC = round(exp(mean(log(COBS))),3))
head(mean_occ2, n = 3)
#> Source: local data frame [3 x 3]
#> Groups: GENDER [1]
#> 
#>   GENDER   TAD meanCONC
#>   <fctr> <dbl>    <dbl>
#> 1      0  0.00      0.0
#> 2      0  0.05     11.0
#> 3      0  0.35     47.6
tail(mean_occ2, n = 3)
#> Source: local data frame [3 x 3]
#> Groups: GENDER [1]
#> 
#>   GENDER   TAD meanCONC
#>   <fctr> <dbl>    <dbl>
#> 1      1    16    1.409
#> 2      1    20    1.081
#> 3      1    24    0.833
```

He's gotten started on the plot but can't figure out how to overlay the profiles.


```r
ggplot(oral_data_occ2, aes(x = TAD, y = COBS, 
                      group = ID)) + 
  geom_line(size = 1.05)+ base_theme() +
  xlab("Time After Dose, hours") +
  ylab("Concentration, ug/mL") + scale_y_log10()
#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-11-1.png" width="672" />

To get the final result he asks you to:

* calculate the geometric mean values for males and females
* overlay the results and color by Gender
* update the legend with the name 'Gender' and Male/Female Labels
* move the legend to be in the top right corner inside the plot
* add another break in the y axis for 50

### So it looks like this:


```r
ggplot(oral_data_occ2, aes(x = TAD, y = COBS, 
                      group = ID)) + 
  geom_line(size = 1.05)+ base_theme() +
  xlab("Time After Dose, hours") +
  ylab("Concentration, ug/mL") +
  scale_color_discrete(name="Gender", labels= c("Male", "Female")) + 
    scale_y_log10(breaks = c(1, 10 , 50, 100)) +
  geom_line(data = mean_occ2, 
            aes(x = TAD, y = meanCONC, group = GENDER, color = GENDER), size = 1.5)+ 
    theme(legend.justification=c(1,1), legend.position=c(1,1))
#> Warning: Transformation introduced infinite values in continuous y-axis

#> Warning: Transformation introduced infinite values in continuous y-axis
```

<img src="ggplot_help_jon_solutions_files/figure-html/unnamed-chunk-12-1.png" width="672" />




```r
devtools::session_info()
#> Session info --------------------------------------------------------------
#>  setting  value                       
#>  version  R version 3.3.2 (2016-10-31)
#>  system   x86_64, mingw32             
#>  ui       RTerm                       
#>  language (EN)                        
#>  collate  English_United States.1252  
#>  tz       America/New_York            
#>  date     2016-11-22
#> Packages ------------------------------------------------------------------
#>  package      * version    date      
#>  assertthat     0.1        2013-12-06
#>  bookdown       0.2        2016-11-12
#>  codetools      0.2-15     2016-10-05
#>  colorspace     1.2-7      2016-10-11
#>  DBI            0.5-1      2016-09-10
#>  devtools       1.12.0     2016-06-24
#>  digest         0.6.10     2016-08-02
#>  dplyr        * 0.5.0      2016-06-24
#>  evaluate       0.10       2016-10-11
#>  ggplot2      * 2.1.0.9001 2016-11-07
#>  gtable         0.2.0      2016-02-26
#>  highr          0.6        2016-05-09
#>  htmltools      0.3.5      2016-03-21
#>  httpuv         1.3.3      2015-08-04
#>  knitr        * 1.15       2016-11-09
#>  labeling       0.3        2014-08-23
#>  lazyeval       0.2.0      2016-06-12
#>  magrittr       1.5        2014-11-22
#>  memoise        1.0.0      2016-01-29
#>  mime           0.5        2016-07-07
#>  miniUI         0.1.1      2016-01-15
#>  munsell        0.4.3      2016-02-13
#>  PKPDdatasets * 0.1.0      2016-11-02
#>  PKPDmisc     * 0.4.4.9000 2016-11-02
#>  plyr           1.8.4      2016-06-08
#>  R6             2.2.0      2016-10-05
#>  Rcpp           0.12.7     2016-09-05
#>  reshape2       1.4.2      2016-10-22
#>  rmarkdown      1.1        2016-10-16
#>  scales         0.4.0.9003 2016-11-07
#>  shiny          0.14.2     2016-11-01
#>  stringi        1.1.2      2016-10-01
#>  stringr        1.1.0      2016-08-19
#>  tibble         1.2        2016-08-26
#>  withr          1.0.2      2016-06-20
#>  xtable         1.8-2      2016-02-05
#>  yaml           2.1.13     2014-06-12
#>  source                                
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  Github (hadley/ggplot2@70c3d69)       
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  Github (dpastoor/PKPDdatasets@52880fa)
#>  Github (dpastoor/PKPDmisc@beae2a6)    
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  Github (hadley/scales@d58d83a)        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)                        
#>  CRAN (R 3.3.2)
```


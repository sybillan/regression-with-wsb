GME Regression Model
================

``` r
#Using Statlearning to draw regression model of how the users of subreddit WallStreetBets drove price of GME in 2021
#Inital Setup

library(dplyr) # for filter and join functions
library(readr) # for read_csv
library(tidyverse) #for str_which

#Loading WallStreetBets file
dataset1 <- read_csv("reddit_wsb.csv",col_types = cols(.default = "c"))

#Loading GME Market data from Jan to April 2021
GME<-read.table(unz("archive.zip","GME.csv"), header=T,quote="\"",sep=",")
```

WSB data for feature ‘title’ has errors in punctuation including “,”,
“;”, “.”, “/”, “", and also, notoriously,”\|". Using gsub, eliminating
delimiters from the title column. Thanks to <MLane@Kaggle> for cleaning
the WSB data.

``` r
dataset1$title = gsub(pattern = "\\,",".",dataset1$title)
dataset1$body  = gsub(pattern = "\\,",".",dataset1$body)
wsb<-dataset1
```

Regression Fit Against frequency of daily posts that mention
GME/GameStop

``` r
#Formatting Dates on both GME and WSB file
GME$Date<-as.Date(GME$Date,"%Y-%m-%d")  
wsb$X<-as.Date(wsb$timestamp,"%m/%d/%Y")

#Filter for keywords 'GameStop' and 'GME' using index
m<-str_which(wsb$title,"GME")
m<-append(m,str_which(wsb$title,"GameStop"))
wsb_gamestop<-wsb[c(m),]

#Deriving frequency table for GameStop/GME Mentions against Date
freq<-data.frame(table(wsb_gamestop$X))
freq$Var1<-as.Date(freq$Var1,"%Y-%m-%d")
colnames(freq)<-c("Date","Freq")

#Joining with the GME Stock Index file
GME<-dplyr::left_join(GME,freq,by="Date")

GME$Freq[is.na(GME$Freq)] <- 0

#Regression modeling of GME High Price against frequency of GME mentions on WallStreetBets 
summary(lm(High~Freq,data=GME))
```

    ## 
    ## Call:
    ## lm(formula = High ~ Freq, data = GME)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -125.39  -89.67   15.95   47.95  315.18 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 143.47337    9.94733  14.423  < 2e-16 ***
    ## Freq          0.09700    0.02976   3.259  0.00162 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 89.03 on 83 degrees of freedom
    ## Multiple R-squared:  0.1135, Adjusted R-squared:  0.1028 
    ## F-statistic: 10.62 on 1 and 83 DF,  p-value: 0.001621

``` r
plot(GME$Freq, GME$High)

abline(lm(High~Freq,data=GME))
```

![](Regression-Modeling-with-WSB-and-GME_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

Null hypothesis can be rejected, since p is significant at 0.00162.
There is a causal relationship between predictor and dependent variable.

Regression Fit Against Musk’s tweet that mention GME/GameStop

``` r
#Assessing Impact of Musk's solitary tweet on 26th Jan on GME Price Movement
GME$Musk<-0
GME$Musk[GME$Date=="2021-01-26"]<-1

summary(lm(High~Freq+Musk,data=GME))
```

    ## 
    ## Call:
    ## lm(formula = High ~ Freq + Musk, data = GME)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -125.31  -89.59   16.03   48.03  315.25 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 143.39086   10.07053  14.239  < 2e-16 ***
    ## Freq          0.09706    0.02995   3.240  0.00173 ** 
    ## Musk          6.60914   90.13356   0.073  0.94173    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 89.57 on 82 degrees of freedom
    ## Multiple R-squared:  0.1135, Adjusted R-squared:  0.09189 
    ## F-statistic:  5.25 on 2 and 82 DF,  p-value: 0.007155

There is no relationship to be inferred between GME’s High Price with
Musk’s tweet. As observed, p is not significant.

``` r
GME$Change<-((GME$Close-GME$Open)/GME$Open)*100
summary(lm(Change~Freq+Musk,data=GME))
```

    ## 
    ## Call:
    ## lm(formula = Change ~ Freq + Musk, data = GME)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -38.945  -6.854  -1.112   2.987 105.106 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  0.323111   2.161132   0.150 0.881518    
    ## Freq        -0.008440   0.006428  -1.313 0.192862    
    ## Musk        66.772643  19.342636   3.452 0.000882 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 19.22 on 82 degrees of freedom
    ## Multiple R-squared:  0.1449, Adjusted R-squared:  0.124 
    ## F-statistic: 6.948 on 2 and 82 DF,  p-value: 0.001632

``` r
plot(GME$Musk,GME$Change)
abline(lm(Change~Musk,data=GME))
```

![](Regression-Modeling-with-WSB-and-GME_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

Regression Fit Against Musk’s tweet that mention GME/GameStop

``` r
GME$Musk<-0
GME$Musk[GME$Date=="2021-01-26"]<-1

summary(lm(High~Freq+Musk,data=GME))
```

    ## 
    ## Call:
    ## lm(formula = High ~ Freq + Musk, data = GME)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -125.31  -89.59   16.03   48.03  315.25 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept) 143.39086   10.07053  14.239  < 2e-16 ***
    ## Freq          0.09706    0.02995   3.240  0.00173 ** 
    ## Musk          6.60914   90.13356   0.073  0.94173    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 89.57 on 82 degrees of freedom
    ## Multiple R-squared:  0.1135, Adjusted R-squared:  0.09189 
    ## F-statistic:  5.25 on 2 and 82 DF,  p-value: 0.007155

``` r
GME$Change<-((GME$Close-GME$Open)/GME$Open)*100

summary(lm(Change~Freq+Musk,data=GME))
```

    ## 
    ## Call:
    ## lm(formula = Change ~ Freq + Musk, data = GME)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -38.945  -6.854  -1.112   2.987 105.106 
    ## 
    ## Coefficients:
    ##              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)  0.323111   2.161132   0.150 0.881518    
    ## Freq        -0.008440   0.006428  -1.313 0.192862    
    ## Musk        66.772643  19.342636   3.452 0.000882 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 19.22 on 82 degrees of freedom
    ## Multiple R-squared:  0.1449, Adjusted R-squared:  0.124 
    ## F-statistic: 6.948 on 2 and 82 DF,  p-value: 0.001632

``` r
plot(GME$Musk,GME$Change)
abline(lm(Change~Musk,data=GME))
```

![](Regression-Modeling-with-WSB-and-GME_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

While GME’s High Price is not responsive to Musk’s tweeting, the %
change in price is, with p-value at 0.000882 and also observable using
correlation function. Change \~ Musk @ 0.36 positive.

``` r
correlation_matrix<-cor(GME[,-1])
correlation_matrix
```

    ##                  Open         High         Low       Close   Adj.Close
    ## Open       1.00000000  0.957683501  0.94232418  0.95972746  0.95972746
    ## High       0.95768350  1.000000000  0.85564267  0.92747845  0.92747845
    ## Low        0.94232418  0.855642672  1.00000000  0.96458336  0.96458336
    ## Close      0.95972746  0.927478453  0.96458336  1.00000000  1.00000000
    ## Adj.Close  0.95972746  0.927478453  0.96458336  1.00000000  1.00000000
    ## Volume    -0.10554424  0.031328562 -0.24298382 -0.09619792 -0.09619792
    ## Freq       0.33239192  0.336829136  0.18798522  0.26051733  0.26051733
    ## Musk      -0.06224279 -0.001461854 -0.06048403  0.02369134  0.02369134
    ## Change    -0.26739381 -0.179575288 -0.14587728 -0.04265070 -0.04265070
    ##                Volume        Freq         Musk     Change
    ## Open      -0.10554424  0.33239192 -0.062242793 -0.2673938
    ## High       0.03132856  0.33682914 -0.001461854 -0.1795753
    ## Low       -0.24298382  0.18798522 -0.060484029 -0.1458773
    ## Close     -0.09619792  0.26051733  0.023691340 -0.0426507
    ## Adj.Close -0.09619792  0.26051733  0.023691340 -0.0426507
    ## Volume     1.00000000  0.09886669  0.377004216  0.3129711
    ## Freq       0.09886669  1.00000000 -0.026966685 -0.1436342
    ## Musk       0.37700422 -0.02696668  1.000000000  0.3562654
    ## Change     0.31297107 -0.14363420  0.356265412  1.0000000
Group Project
================

``` r
applications_project <- applications_project %>%
  drop_na(application_number) %>%
  filter(application_number != "08531853")
```

``` r
#creating a new dataset of issued and abandoned applications

length_df <- applications_project %>%
  filter(disposal_type != "PEND") %>%
  drop_na(disposal_type)

#transforming by creating app length variable, cleaning and subtracting 60 days in cases where application was abandoned due to lack of response (to approximate the final patent office decision date)

length_df <- length_df %>%
  mutate(app_length = if_else(disposal_type == "ISS", interval(filing_date, patent_issue_date) %/% days(1),interval(filing_date, abandon_date) %/% days(1))) %>%
  mutate(wg = (floor(examiner_art_unit / 10) * 10)) %>%
  filter(app_length >= 0)%>%
  filter(app_length <= 6385) %>%
  mutate(app_length = if_else(appl_status_code == 161 & app_length > 60, app_length - 60, app_length))%>%
  drop_na(app_length) %>%
  mutate(year_submitted = year(filing_date))

#removing outliers of average processing time

avg_time <- length_df %>%
  drop_na(examiner_id) %>%
  drop_na(gender) %>%
  group_by(examiner_id) %>%
  summarize(avg_app_length = mean(app_length),
            )

is_outlier <- function(x) {
  q <- quantile(x, probs = c(0.25, 0.75))
  iqr <- q[2] - q[1]
  lower_bound <- q[1] - (1.5 * iqr)
  upper_bound <- q[2] + (1.5 * iqr)
  x < lower_bound | x > upper_bound
}

filtered <- avg_time %>%
  filter(!is_outlier(avg_app_length))

length_df <- length_df %>%
  drop_na(gender) %>%
  drop_na(examiner_id) %>%
  semi_join(filtered, by = "examiner_id")
```

``` r
#average time taken per app for gender
length_df %>%
  group_by(gender) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = gender, y = y, fill=gender)) +
  geom_col(position = "dodge")+
  labs(x = "Gender", y = "Average days for application completion", fill = "Gender") +
  ggtitle("Average application length by gender") +
  theme_minimal()+
  scale_fill_manual(values = c("male" = "lightblue", "female" = "pink"))
```

![](Final-Project_files/figure-gfm/visualizing-1.png)<!-- -->

``` r
#average time taken per app for race
length_df %>%
  group_by(race) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = as.factor(race), y = y,fill=race)) +
  geom_col(position = "dodge")+
  labs(x = "Technology Centre", y = "Average days for application completion", fill = "Race") +
  ggtitle("Average processing times by race") +
  theme_minimal()
```

![](Final-Project_files/figure-gfm/visualizing-2.png)<!-- -->

``` r
#average application times by TC
length_df %>%
  group_by(tc) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = as.factor(tc), y = y, fill=as.factor(tc))) +
  geom_col()+
  labs(x = "Technology Centre", y = "Average days for application completion", fill = "TC") +
  ggtitle("Average processing times by TC") +
  scale_fill_manual(values = c("1600" = "green", "1700" = "purple", "2100"="darkblue", "2400"="red"))+
  theme_minimal()
```

![](Final-Project_files/figure-gfm/visualizing-3.png)<!-- -->

``` r
#distribution of app length based on final decision (issued or abandoned)  
length_df %>%
  ggplot(aes(y = app_length, fill = disposal_type)) +
  geom_boxplot(outlier.shape = NA) +
  facet_wrap(~ disposal_type, nrow = 1) +
  coord_cartesian(ylim = c(0, 3000)) +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))+
  labs(y = "Application length") +
  ggtitle("Application length by final decision")+
  theme(axis.text.x = element_blank())
```

![](Final-Project_files/figure-gfm/visualizing-4.png)<!-- -->

``` r
#gender distribution
length_df %>%
  group_by(gender, tc) %>%
  summarize(n = n_distinct(examiner_id)) %>%
  ggplot(aes(x = as.factor(tc), y = n, fill=gender)) +
  geom_col(position = "dodge")+
  labs(x = "TC", y = "Count", fill = "Gender") +
  ggtitle("Gender Distribution across TC") +
  theme_minimal()+
  scale_fill_manual(values = c("male" = "lightblue", "female" = "pink"))
```

    ## `summarise()` has grouped output by 'gender'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-1.png)<!-- -->

``` r
#racial distribution across TC
length_df %>%
  group_by(race, tc) %>%
  summarize(n = n_distinct(examiner_id)) %>%
  ggplot(aes(x = as.factor(tc), y = n, fill=race)) +
  geom_col(position = "dodge")+
  labs(x = "TC", y = "Count", fill = "Race") +
  ggtitle("Racial distributions across TCs") +
  theme_minimal()
```

    ## `summarise()` has grouped output by 'race'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-2.png)<!-- -->

``` r
#total distribution across TC
length_df %>%
  group_by(tc) %>%
  summarize(n = n_distinct(examiner_id)) %>%
  ggplot(aes(x = as.factor(tc), y = n, fill=as.factor(tc))) +
  geom_col(position = "dodge")+
  labs(x = "TC", y = "Count", fill = "TC") +
  ggtitle("Distribution of employees across TCs") +
  theme_minimal()+
  scale_fill_manual(values = c("1600" = "green", "1700" = "purple", "2100"="darkblue", "2400"="red"))
```

![](Final-Project_files/figure-gfm/structural-differences-3.png)<!-- -->

``` r
#distribution of decision type by gender
length_df %>%
  group_by(gender, disposal_type) %>%
  summarize(count = n()) %>%
  ggplot(aes(x = gender, y = count, fill=disposal_type)) +
  geom_col(position = "fill")+
  labs(x = "Gender", y = "Decision proportion", fill = "Disposal Type") +
  ggtitle("Distribution of decision type by gender") +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'gender'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-4.png)<!-- -->

``` r
#distribution of decision type by race
length_df %>%
  group_by(race, disposal_type) %>%
  summarize(count = n()) %>%
  ggplot(aes(x = race, y = count, fill=disposal_type)) +
  geom_col(position = "fill")+
  labs(x = "Race", y = "Decision proportion", fill = "Disposal Type") +
  ggtitle("Distribution of decision type by race") +
  theme_minimal() +
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'race'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-5.png)<!-- -->

``` r
#distribution of decision type by TC
length_df %>%
  group_by(tc, disposal_type) %>%
  summarize(count = n()) %>%
  ggplot(aes(x = as.factor(tc), y = count, fill=disposal_type)) +
  geom_col(position = "fill")+
  labs(x = "TC", y = "Decision proportion", fill = "Disposal Type") +
  ggtitle("Distribution of decision type by TC") +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'tc'. You can override using the `.groups`
    ## argument.

![](Final-Project_files/figure-gfm/structural-differences-6.png)<!-- -->

``` r
#average processing time based on decision for each gender
length_df %>%
  group_by(gender, disposal_type) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = gender, y = y, fill=disposal_type)) +
  geom_col(position = "dodge")+
  labs(x = "Gender", y = "Decision time", fill = "Disposal Type") +
  ggtitle("Average processing time for each decision type by gender ") +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'gender'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-7.png)<!-- -->

``` r
#average processing time based on decision for each race
length_df %>%
  group_by(race, disposal_type) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = race, y = y, fill=disposal_type)) +
  geom_col(position = "dodge")+
  labs(x = "Race", y = "Decision time", fill = "Disposal Type") +
  ggtitle("Average processing time for each decision type by race") +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'race'. You can override using the
    ## `.groups` argument.

![](Final-Project_files/figure-gfm/structural-differences-8.png)<!-- -->

``` r
#average processing time based on decision for each tc
length_df %>%
  group_by(tc, disposal_type) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = as.factor(tc), y = y, fill=disposal_type)) +
  geom_col(position = "dodge")+
  labs(x = "TC", y = "Decision time", fill = "Disposal Type") +
  ggtitle("Average processing time for each decision type by TC") +
  theme_minimal()+
  scale_fill_manual(values = c("ISS" = "darkgreen", "ABN" = "darkred"))
```

    ## `summarise()` has grouped output by 'tc'. You can override using the `.groups`
    ## argument.

![](Final-Project_files/figure-gfm/structural-differences-9.png)<!-- -->

``` r
#creating a regression dataframe
average_length <- length_df %>%
  group_by(examiner_id, disposal_type) %>%
  summarize(
    gender= first(gender),
    race = first(race),
    tc = first(tc),
    avg_length = mean(app_length),
    tenure = first(tenure_days)
  )


#regression with preprocessed set
model_avg <- lm(data = average_length, avg_length ~ as.factor(tc) + as.factor(gender) + as.factor(race) + as.factor(disposal_type))

model_avg_wt <- lm(data = average_length, avg_length ~ as.factor(tc) + as.factor(gender) + as.factor(race) + as.factor(disposal_type) + tenure)

library(stargazer)

summary(model_avg)
```

    ## 
    ## Call:
    ## lm(formula = avg_length ~ as.factor(tc) + as.factor(gender) + 
    ##     as.factor(race) + as.factor(disposal_type), data = average_length)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1128.91  -200.48    -3.61   197.89  1942.25 
    ## 
    ## Coefficients:
    ##                             Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  980.221     10.510  93.266  < 2e-16 ***
    ## as.factor(tc)1700             26.227      9.199   2.851  0.00437 ** 
    ## as.factor(tc)2100            199.471      9.334  21.371  < 2e-16 ***
    ## as.factor(tc)2400            197.734     10.278  19.239  < 2e-16 ***
    ## as.factor(gender)male        -20.584      7.045  -2.922  0.00349 ** 
    ## as.factor(race)black         -28.593     18.191  -1.572  0.11603    
    ## as.factor(race)Hispanic      -27.258     16.661  -1.636  0.10187    
    ## as.factor(race)other         296.641    149.171   1.989  0.04678 *  
    ## as.factor(race)white         -11.095      7.590  -1.462  0.14385    
    ## as.factor(disposal_type)ISS  187.205      6.292  29.752  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 298 on 8963 degrees of freedom
    ## Multiple R-squared:  0.1621, Adjusted R-squared:  0.1612 
    ## F-statistic: 192.6 on 9 and 8963 DF,  p-value: < 2.2e-16

``` r
summary(model_avg_wt)
```

    ## 
    ## Call:
    ## lm(formula = avg_length ~ as.factor(tc) + as.factor(gender) + 
    ##     as.factor(race) + as.factor(disposal_type) + tenure, data = average_length)
    ## 
    ## Residuals:
    ##      Min       1Q   Median       3Q      Max 
    ## -1231.72  -182.35     4.29   183.86  2021.51 
    ## 
    ## Coefficients:
    ##                               Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                 657.540777  14.238532  46.180  < 2e-16 ***
    ## as.factor(tc)1700            58.877639   8.816004   6.678 2.56e-11 ***
    ## as.factor(tc)2100           242.681487   8.984766  27.010  < 2e-16 ***
    ## as.factor(tc)2400           284.440525  10.148500  28.028  < 2e-16 ***
    ## as.factor(gender)male       -18.186065   6.703079  -2.713  0.00668 ** 
    ## as.factor(race)black        -32.660094  17.316907  -1.886  0.05932 .  
    ## as.factor(race)Hispanic      -1.927813  15.835941  -0.122  0.90311    
    ## as.factor(race)other        244.715560 141.567122   1.729  0.08391 .  
    ## as.factor(race)white         -9.298622   7.217386  -1.288  0.19765    
    ## as.factor(disposal_type)ISS 184.433792   5.985583  30.813  < 2e-16 ***
    ## tenure                        0.060273   0.001902  31.693  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 282.8 on 8920 degrees of freedom
    ##   (42 observations deleted due to missingness)
    ## Multiple R-squared:  0.247,  Adjusted R-squared:  0.2462 
    ## F-statistic: 292.7 on 10 and 8920 DF,  p-value: < 2.2e-16

``` r
tidy(model_avg)
```

    ## # A tibble: 10 × 5
    ##    term                        estimate std.error statistic   p.value
    ##    <chr>                          <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 (Intercept)                    980.      10.5      93.3  0        
    ##  2 as.factor(tc)1700               26.2      9.20      2.85 4.37e-  3
    ##  3 as.factor(tc)2100              199.       9.33     21.4  7.10e- 99
    ##  4 as.factor(tc)2400              198.      10.3      19.2  7.37e- 81
    ##  5 as.factor(gender)male          -20.6      7.05     -2.92 3.49e-  3
    ##  6 as.factor(race)black           -28.6     18.2      -1.57 1.16e-  1
    ##  7 as.factor(race)Hispanic        -27.3     16.7      -1.64 1.02e-  1
    ##  8 as.factor(race)other           297.     149.        1.99 4.68e-  2
    ##  9 as.factor(race)white           -11.1      7.59     -1.46 1.44e-  1
    ## 10 as.factor(disposal_type)ISS    187.       6.29     29.8  1.37e-185

``` r
tidy(model_avg_wt)
```

    ## # A tibble: 11 × 5
    ##    term                        estimate std.error statistic   p.value
    ##    <chr>                          <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 (Intercept)                 658.      14.2        46.2   0        
    ##  2 as.factor(tc)1700            58.9      8.82        6.68  2.56e- 11
    ##  3 as.factor(tc)2100           243.       8.98       27.0   1.63e-154
    ##  4 as.factor(tc)2400           284.      10.1        28.0   9.72e-166
    ##  5 as.factor(gender)male       -18.2      6.70       -2.71  6.68e-  3
    ##  6 as.factor(race)black        -32.7     17.3        -1.89  5.93e-  2
    ##  7 as.factor(race)Hispanic      -1.93    15.8        -0.122 9.03e-  1
    ##  8 as.factor(race)other        245.     142.          1.73  8.39e-  2
    ##  9 as.factor(race)white         -9.30     7.22       -1.29  1.98e-  1
    ## 10 as.factor(disposal_type)ISS 184.       5.99       30.8   3.29e-198
    ## 11 tenure                        0.0603   0.00190    31.7   5.53e-209

``` r
stargazer(model_avg, type="text")
```

    ## 
    ## =======================================================
    ##                                 Dependent variable:    
    ##                             ---------------------------
    ##                                     avg_length         
    ## -------------------------------------------------------
    ## as.factor(tc)1700                    26.227***         
    ##                                       (9.199)          
    ##                                                        
    ## as.factor(tc)2100                   199.471***         
    ##                                       (9.334)          
    ##                                                        
    ## as.factor(tc)2400                   197.734***         
    ##                                      (10.278)          
    ##                                                        
    ## as.factor(gender)male               -20.584***         
    ##                                       (7.045)          
    ##                                                        
    ## as.factor(race)black                  -28.593          
    ##                                      (18.191)          
    ##                                                        
    ## as.factor(race)Hispanic               -27.258          
    ##                                      (16.661)          
    ##                                                        
    ## as.factor(race)other                 296.641**         
    ##                                      (149.171)         
    ##                                                        
    ## as.factor(race)white                  -11.095          
    ##                                       (7.590)          
    ##                                                        
    ## as.factor(disposal_type)ISS         187.205***         
    ##                                       (6.292)          
    ##                                                        
    ## Constant                            980.221***         
    ##                                      (10.510)          
    ##                                                        
    ## -------------------------------------------------------
    ## Observations                           8,973           
    ## R2                                     0.162           
    ## Adjusted R2                            0.161           
    ## Residual Std. Error             297.980 (df = 8963)    
    ## F Statistic                  192.597*** (df = 9; 8963) 
    ## =======================================================
    ## Note:                       *p<0.1; **p<0.05; ***p<0.01

``` r
stargazer(model_avg_wt, type="text")
```

    ## 
    ## =======================================================
    ##                                 Dependent variable:    
    ##                             ---------------------------
    ##                                     avg_length         
    ## -------------------------------------------------------
    ## as.factor(tc)1700                    58.878***         
    ##                                       (8.816)          
    ##                                                        
    ## as.factor(tc)2100                   242.681***         
    ##                                       (8.985)          
    ##                                                        
    ## as.factor(tc)2400                   284.441***         
    ##                                      (10.148)          
    ##                                                        
    ## as.factor(gender)male               -18.186***         
    ##                                       (6.703)          
    ##                                                        
    ## as.factor(race)black                 -32.660*          
    ##                                      (17.317)          
    ##                                                        
    ## as.factor(race)Hispanic               -1.928           
    ##                                      (15.836)          
    ##                                                        
    ## as.factor(race)other                 244.716*          
    ##                                      (141.567)         
    ##                                                        
    ## as.factor(race)white                  -9.299           
    ##                                       (7.217)          
    ##                                                        
    ## as.factor(disposal_type)ISS         184.434***         
    ##                                       (5.986)          
    ##                                                        
    ## tenure                               0.060***          
    ##                                       (0.002)          
    ##                                                        
    ## Constant                            657.541***         
    ##                                      (14.239)          
    ##                                                        
    ## -------------------------------------------------------
    ## Observations                           8,931           
    ## R2                                     0.247           
    ## Adjusted R2                            0.246           
    ## Residual Std. Error             282.772 (df = 8920)    
    ## F Statistic                 292.671*** (df = 10; 8920) 
    ## =======================================================
    ## Note:                       *p<0.1; **p<0.05; ***p<0.01

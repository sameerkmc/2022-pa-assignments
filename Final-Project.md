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
  geom_col(position = "dodge", color="black")+
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
  geom_col(position = "dodge", color="black")+
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
  geom_col(color="black")+
  labs(x = "Technology Centre", y = "Average days for application completion", fill = "TC") +
  ggtitle("Average processing times by TC") +
  scale_fill_manual(values = c("1600" = "green", "1700" = "purple", "2100"="darkblue", "2400"="red"))+
  theme_minimal()
```

![](Final-Project_files/figure-gfm/visualizing-3.png)<!-- -->

``` r
gender_avg <- length_df %>%
  group_by(gender) %>%
  summarize(y = mean(app_length))

race_avg <- length_df %>%
  group_by(race) %>%
  summarize(y = mean(app_length))

tc_avg <- length_df %>%
  group_by(tc) %>%
  summarize(y = mean(app_length))

  
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
  geom_col(position = "dodge", color="black")+
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
  geom_col(position = "dodge", color="black")+
  labs(x = "Technology Centre", y = "Count", fill = "Race") +
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
  geom_col(position = "dodge", color="black")+
  labs(x = "Technology Centre", y = "Count", fill = "TC") +
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
  labs(x = "Technology Centre", y = "Decision proportion", fill = "Disposal Type") +
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
  geom_col(position = "dodge", color="black")+
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
  geom_col(position = "dodge", color="black")+
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
  geom_col(position = "dodge", color="black")+
  labs(x = "Technology Centre", y = "Decision time", fill = "Disposal Type") +
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
yoe_df <- length_df %>%
  drop_na(earliest_date) %>%
  mutate(yoe = floor(interval(earliest_date, filing_date, ) / dyears(1)))

yoe_df %>%
  group_by(yoe) %>%
  summarize(y = mean(app_length)) %>%
  ggplot(aes(x = as.factor(yoe), y = y)) +
  geom_col(fill = "lightgreen", color="black") +
  labs(x = "Years of Experience", y = "Average decision time") +
  ggtitle("Average processing time by years of experience") +
  theme_minimal()
```

![](Final-Project_files/figure-gfm/years-of-experience-1.png)<!-- -->

``` r
yoe_reg <- yoe_df %>%
  group_by(examiner_id, disposal_type,yoe) %>%
  summarize(
    gender= first(gender),
    race = first(race),
    tc = first(tc),
    avg_length = mean(app_length),
    tenure = first(tenure_days)
  )
```

    ## `summarise()` has grouped output by 'examiner_id', 'disposal_type'. You can
    ## override using the `.groups` argument.

``` r
model_yoe_af <- lm(data = yoe_reg, avg_length ~ as.factor(tc) + as.factor(gender) + as.factor(race) + as.factor(disposal_type) + as.factor(yoe))

model_yoe <- lm(data = yoe_reg, avg_length ~ as.factor(tc) + as.factor(gender) + as.factor(race) + as.factor(disposal_type) + yoe)

summary(model_yoe_af)
```

    ## 
    ## Call:
    ## lm(formula = avg_length ~ as.factor(tc) + as.factor(gender) + 
    ##     as.factor(race) + as.factor(disposal_type) + as.factor(yoe), 
    ##     data = yoe_reg)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1561.1  -304.3   -21.1   249.3  4405.4 
    ## 
    ## Coefficients:
    ##                              Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)                  1636.491      8.115 201.664  < 2e-16 ***
    ## as.factor(tc)1700            -134.207      5.070 -26.471  < 2e-16 ***
    ## as.factor(tc)2100              16.705      5.464   3.057  0.00224 ** 
    ## as.factor(tc)2400             119.079      5.694  20.915  < 2e-16 ***
    ## as.factor(gender)male         -28.613      4.027  -7.105 1.22e-12 ***
    ## as.factor(race)black          -15.420     10.225  -1.508  0.13155    
    ## as.factor(race)Hispanic       -55.167     10.180  -5.419 6.00e-08 ***
    ## as.factor(race)other          270.898     68.906   3.931 8.45e-05 ***
    ## as.factor(race)white          -21.582      4.363  -4.946 7.58e-07 ***
    ## as.factor(disposal_type)ISS   213.079      3.642  58.507  < 2e-16 ***
    ## as.factor(yoe)1              -184.084      8.347 -22.053  < 2e-16 ***
    ## as.factor(yoe)2              -238.848      8.416 -28.380  < 2e-16 ***
    ## as.factor(yoe)3              -287.523      8.527 -33.718  < 2e-16 ***
    ## as.factor(yoe)4              -350.058      8.642 -40.505  < 2e-16 ***
    ## as.factor(yoe)5              -413.822      8.795 -47.052  < 2e-16 ***
    ## as.factor(yoe)6              -482.328      8.947 -53.911  < 2e-16 ***
    ## as.factor(yoe)7              -553.394      9.083 -60.923  < 2e-16 ***
    ## as.factor(yoe)8              -646.786      9.255 -69.889  < 2e-16 ***
    ## as.factor(yoe)9              -733.248      9.502 -77.168  < 2e-16 ***
    ## as.factor(yoe)10             -826.663      9.825 -84.140  < 2e-16 ***
    ## as.factor(yoe)11             -911.939     10.290 -88.625  < 2e-16 ***
    ## as.factor(yoe)12             -969.157     10.912 -88.813  < 2e-16 ***
    ## as.factor(yoe)13            -1010.576     11.681 -86.513  < 2e-16 ***
    ## as.factor(yoe)14            -1109.840     12.724 -87.222  < 2e-16 ***
    ## as.factor(yoe)15            -1258.234     14.897 -84.463  < 2e-16 ***
    ## as.factor(yoe)16            -1443.482     21.416 -67.402  < 2e-16 ***
    ## as.factor(yoe)17            -1644.337    228.192  -7.206 5.82e-13 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 510 on 78637 degrees of freedom
    ## Multiple R-squared:  0.3259, Adjusted R-squared:  0.3257 
    ## F-statistic:  1462 on 26 and 78637 DF,  p-value: < 2.2e-16

``` r
summary(model_yoe)
```

    ## 
    ## Call:
    ## lm(formula = avg_length ~ as.factor(tc) + as.factor(gender) + 
    ##     as.factor(race) + as.factor(disposal_type) + yoe, data = yoe_reg)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -1529.5  -304.9   -21.1   250.5  4454.1 
    ## 
    ## Coefficients:
    ##                              Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)                 1588.9916     6.4950  244.647  < 2e-16 ***
    ## as.factor(tc)1700           -134.4209     5.0801  -26.460  < 2e-16 ***
    ## as.factor(tc)2100             17.2722     5.4758    3.154  0.00161 ** 
    ## as.factor(tc)2400            120.9493     5.7021   21.212  < 2e-16 ***
    ## as.factor(gender)male        -28.4814     4.0359   -7.057 1.72e-12 ***
    ## as.factor(race)black         -15.3010    10.2476   -1.493  0.13541    
    ## as.factor(race)Hispanic      -55.0975    10.2018   -5.401 6.65e-08 ***
    ## as.factor(race)other         268.9962    69.0561    3.895 9.81e-05 ***
    ## as.factor(race)white         -21.5611     4.3727   -4.931 8.20e-07 ***
    ## as.factor(disposal_type)ISS  211.8978     3.6469   58.104  < 2e-16 ***
    ## yoe                          -76.8524     0.4314 -178.158  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 511.2 on 78653 degrees of freedom
    ## Multiple R-squared:  0.3228, Adjusted R-squared:  0.3228 
    ## F-statistic:  3750 on 10 and 78653 DF,  p-value: < 2.2e-16

``` r
tidy(model_yoe_af)
```

    ## # A tibble: 27 × 5
    ##    term                        estimate std.error statistic   p.value
    ##    <chr>                          <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 (Intercept)                   1636.       8.11    202.   0        
    ##  2 as.factor(tc)1700             -134.       5.07    -26.5  9.96e-154
    ##  3 as.factor(tc)2100               16.7      5.46      3.06 2.24e-  3
    ##  4 as.factor(tc)2400              119.       5.69     20.9  7.20e- 97
    ##  5 as.factor(gender)male          -28.6      4.03     -7.10 1.22e- 12
    ##  6 as.factor(race)black           -15.4     10.2      -1.51 1.32e-  1
    ##  7 as.factor(race)Hispanic        -55.2     10.2      -5.42 6.00e-  8
    ##  8 as.factor(race)other           271.      68.9       3.93 8.45e-  5
    ##  9 as.factor(race)white           -21.6      4.36     -4.95 7.58e-  7
    ## 10 as.factor(disposal_type)ISS    213.       3.64     58.5  0        
    ## # ℹ 17 more rows

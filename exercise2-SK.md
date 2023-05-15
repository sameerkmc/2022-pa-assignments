Exercise 2
================

### Sameer Kamal

## Load data

Load the following data: + applications from `app_data_sample.parquet` +
edges from `edges_sample.csv`

## Get gender for examiners

We’ll get gender based on the first name of the examiner, which is
recorded in the field `examiner_name_first`. We’ll use library `gender`
for that, relying on a modified version of their own
[example](https://cran.r-project.org/web/packages/gender/vignettes/predicting-gender.html).

Note that there are over 2 million records in the applications table –
that’s because there are many records for each examiner, as many as the
number of applications that examiner worked on during this time frame.
Our first step therefore is to get all *unique* names in a separate list
`examiner_names`. We will then guess gender for each one and will join
this table back to the original dataset. So, let’s get names without
repetition:

Now let’s use function `gender()` as shown in the example for the
package to attach a gender and probability to each name and put the
results into the table `examiner_names_gender`

Finally, let’s join that table back to our original applications data
and discard the temporary tables we have just created to reduce clutter
in our environment.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb) max used  (Mb)
    ## Ncells  4441152 237.2    7872827 420.5         NA  4459825 238.2
    ## Vcells 49378847 376.8   92843553 708.4      16384 79694514 608.1

## Guess the examiner’s race

We’ll now use package `wru` to estimate likely race of an examiner. Just
like with gender, we’ll get a list of unique names first, only now we
are using surnames.

We’ll follow the instructions for the package outlined here
<https://github.com/kosukeimai/wru>.

    ## Warning: Unknown or uninitialised column: `state`.

    ## Proceeding with last name predictions...

    ## ℹ All local files already up-to-date!

    ## 701 (18.4%) individuals' last names were not matched.

As you can see, we get probabilities across five broad US Census
categories: white, black, Hispanic, Asian and other. (Some of you may
correctly point out that Hispanic is not a race category in the US
Census, but these are the limitations of this package.)

Our final step here is to pick the race category that has the highest
probability for each last name and then join the table back to the main
applications table. See this example for comparing values across
columns: <https://www.tidyverse.org/blog/2020/04/dplyr-1-0-0-rowwise/>.
And this one for `case_when()` function:
<https://dplyr.tidyverse.org/reference/case_when.html>.

Let’s join the data back to the applications table.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb) max used  (Mb)
    ## Ncells  4626995 247.2    7872827 420.5         NA  6915089 369.4
    ## Vcells 51723863 394.7   92843553 708.4      16384 91832951 700.7

## Examiner’s tenure

To figure out the timespan for which we observe each examiner in the
applications data, let’s find the first and the last observed date for
each examiner. We’ll first get examiner IDs and application dates in a
separate table, for ease of manipulation. We’ll keep examiner ID (the
field `examiner_id`), and earliest and latest dates for each application
(`filing_date` and `appl_status_date` respectively). We’ll use functions
in package `lubridate` to work with date and time values.

The dates look inconsistent in terms of formatting. Let’s make them
consistent. We’ll create new variables `start_date` and `end_date`.

Let’s now identify the earliest and the latest date for each examiner
and calculate the difference in days, which is their tenure in the
organization.

Joining back to the applications data.

    ##            used  (Mb) gc trigger  (Mb) limit (Mb)  max used  (Mb)
    ## Ncells  4635636 247.6    7872827 420.5         NA   7872827 420.5
    ## Vcells 57798981 441.0  111492263 850.7      16384 111197021 848.4

## Visualizing gender, race and tenure distributions

![](exercise2-SK_files/figure-gfm/plotting-1.png)<!-- -->![](exercise2-SK_files/figure-gfm/plotting-2.png)<!-- -->

    ## Warning: Removed 25863 rows containing non-finite values (`stat_bin()`).

![](exercise2-SK_files/figure-gfm/plotting-3.png)<!-- -->

## Visualizing distribution across Technology Centres

![](exercise2-SK_files/figure-gfm/plotting%202-1.png)<!-- -->

## Visualizing distribution across work groups

![](exercise2-SK_files/figure-gfm/plotting%203-1.png)<!-- -->

## Correlating gender and race with tenure (excluding Technology Centre)

    ## 
    ## Call:
    ## lm(formula = tenure_days ~ 1 + factor(gender) + factor(race), 
    ##     data = applications)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -5569.1  -564.8   542.2   796.2  1260.8 
    ## 
    ## Coefficients:
    ##                      Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)          5575.113      2.057 2710.04   <2e-16 ***
    ## factor(gender)male   -131.280      1.776  -73.90   <2e-16 ***
    ## factor(race)black      94.490      4.846   19.50   <2e-16 ***
    ## factor(race)Hispanic -358.624      5.078  -70.62   <2e-16 ***
    ## factor(race)other     321.607     31.351   10.26   <2e-16 ***
    ## factor(race)white      75.966      1.968   38.61   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1090 on 1700642 degrees of freedom
    ##   (317829 observations deleted due to missingness)
    ## Multiple R-squared:  0.008426,   Adjusted R-squared:  0.008423 
    ## F-statistic:  2890 on 5 and 1700642 DF,  p-value: < 2.2e-16

## Correlating gender and race with tenure and including Technology Centre

    ## 
    ## Call:
    ## lm(formula = tenure_days ~ 1 + factor(gender) + factor(race) + 
    ##     factor(tc), data = applications)
    ## 
    ## Residuals:
    ##     Min      1Q  Median      3Q     Max 
    ## -5598.1  -586.0   462.1   779.3  1665.5 
    ## 
    ## Coefficients:
    ##                      Estimate Std. Error  t value Pr(>|t|)    
    ## (Intercept)          5887.872      2.430 2423.443  < 2e-16 ***
    ## factor(gender)male    -17.159      1.781   -9.635  < 2e-16 ***
    ## factor(race)black     107.638      4.722   22.793  < 2e-16 ***
    ## factor(race)Hispanic -398.776      4.950  -80.567  < 2e-16 ***
    ## factor(race)other     219.608     30.554    7.187 6.61e-13 ***
    ## factor(race)white     -38.761      1.980  -19.574  < 2e-16 ***
    ## factor(tc)1700       -285.382      2.092 -136.400  < 2e-16 ***
    ## factor(tc)2100       -348.374      2.514 -138.547  < 2e-16 ***
    ## factor(tc)2400       -812.399      2.694 -301.518  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## Residual standard error: 1062 on 1700639 degrees of freedom
    ##   (317829 observations deleted due to missingness)
    ## Multiple R-squared:  0.05884,    Adjusted R-squared:  0.05884 
    ## F-statistic: 1.329e+04 on 8 and 1700639 DF,  p-value: < 2.2e-16

## Findings

Based on the results above, we see that race and gender are
statistically significant predictors of tenure at the patent office. For
example, we see that men, on average, have a lower average tenure, and
the same could be said of Hispanics when compared to other ethnicities.
Looking at the distributions, however, we see that sample sizes vary
considerably between genders and ethnicities. In addition, the low
R-squared values demonstrate the model is not a good fit - there are
either missing predictors or other predictors that better explain the
model variation.

When we add technology centres to the model, we see a significant change
in the coefficient for gender and an overall improvement in the adjusted
R-squared. The change in adjusted R-squared shows that adding the new
variables has genuinely improved the model fit, while the considerable
change in the gender coefficient show that predictions previously
attributed to gender may have more to do with differences in technology
centres. It seems likely that there are significant differences in
gender ratios in different technology centres.

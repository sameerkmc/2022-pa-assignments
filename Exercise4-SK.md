Exercise 4
================

### Setting up the data

``` r
applications <- applications %>%
  mutate(wg = (floor(examiner_art_unit / 10) * 10)) %>%
  mutate(appl_status_date = dmy_hms(appl_status_date)) %>%
  mutate(year = year(appl_status_date)) %>%
  filter(year <= 2017) %>%
  drop_na(examiner_id)
```

``` r
examiners <- applications %>%
  group_by(examiner_id, examiner_art_unit, year) %>%
  summarise( 
    gender = first(gender),
    race = first(race),
    tc = first(tc),
    wg = first(wg)
    )
```

    ## `summarise()` has grouped output by 'examiner_id', 'examiner_art_unit'. You can
    ## override using the `.groups` argument.

### Selecting TC

``` r
tc1600 <- examiners %>%
  filter(tc == 1600) %>%
  drop_na(gender)
```

### Visualizing

    ## `summarise()` has grouped output by 'year'. You can override using the
    ## `.groups` argument.

![](Exercise4-SK_files/figure-gfm/plotting%20tc-1.png)<!-- -->

    ## `summarise()` has grouped output by 'year', 'gender'. You can override using
    ## the `.groups` argument.

![](Exercise4-SK_files/figure-gfm/plotting%20wg-1.png)<!-- -->

    ## `summarise()` has grouped output by 'year', 'gender'. You can override using
    ## the `.groups` argument.

![](Exercise4-SK_files/figure-gfm/plotting%20wg-2.png)<!-- -->

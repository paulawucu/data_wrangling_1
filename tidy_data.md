tidy_data
================
Paula Wu
9/28/2021

# pivot_longer

``` r
pulse_df = 
  haven::read_sas("./data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names()

pulse_df
```

    ## # A tibble: 1,087 × 7
    ##       id   age sex    bdi_score_bl bdi_score_01m bdi_score_06m bdi_score_12m
    ##    <dbl> <dbl> <chr>         <dbl>         <dbl>         <dbl>         <dbl>
    ##  1 10003  48.0 male              7             1             2             0
    ##  2 10015  72.5 male              6            NA            NA            NA
    ##  3 10022  58.5 male             14             3             8            NA
    ##  4 10026  72.7 male             20             6            18            16
    ##  5 10035  60.4 male              4             0             1             2
    ##  6 10050  84.7 male              2            10            12             8
    ##  7 10078  31.3 male              4             0            NA            NA
    ##  8 10088  56.9 male              5            NA             0             2
    ##  9 10091  76.0 male              0             3             4             0
    ## 10 10092  74.2 female           10             2            11             6
    ## # … with 1,077 more rows

Let’s try to pivot: pivot longer

``` r
# visit will be the type of visit, and dbi will store the values
pulse_tidy_df = 
  pulse_df %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    values_to = "bdi",
    names_prefix = "bdi_score_"
  ) %>% 
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)
  )
pulse_tidy_df
```

    ## # A tibble: 4,348 × 5
    ##       id   age sex   visit   bdi
    ##    <dbl> <dbl> <chr> <fct> <dbl>
    ##  1 10003  48.0 male  00m       7
    ##  2 10003  48.0 male  01m       1
    ##  3 10003  48.0 male  06m       2
    ##  4 10003  48.0 male  12m       0
    ##  5 10015  72.5 male  00m       6
    ##  6 10015  72.5 male  01m      NA
    ##  7 10015  72.5 male  06m      NA
    ##  8 10015  72.5 male  12m      NA
    ##  9 10022  58.5 male  00m      14
    ## 10 10022  58.5 male  01m       3
    ## # … with 4,338 more rows

don’t use gather(), use pivot_longer()

# pivot wider

make up a result data table

``` r
analysis_df = 
  tibble(
    group = c("treatment", "treatment", "control", "control"),
    time = c("a", "b", "c", "d"),
    group_mean = c(4, 8, 3, 6)
  )
analysis_df %>% 
  pivot_wider(
    names_from = "time",
    values_from = "group_mean"
  ) %>% 
  knitr::kable()
```

| group     |   a |   b |   c |   d |
|:----------|----:|----:|----:|----:|
| treatment |   4 |   8 |  NA |  NA |
| control   |  NA |  NA |   3 |   6 |

Don’t use spread(), use pivot_wider() # bind rows import the LotR movie
words stuff

``` r
fellowship_df = 
  read_excel("./data/LotR_Words.xlsx", range = "B3:D6") %>% 
  mutate(movie = "fellowship_rings")  # add an extra column "movie"
two_towers_df = 
  read_excel("./data/LotR_Words.xlsx", range = "F3:H6") %>% 
  mutate(movie = "tow_towers") 
return_df = 
  read_excel("./data/LotR_Words.xlsx", range = "J3:L6") %>% 
  mutate(movie = "return_of_the_king") 
lotr_df = 
  bind_rows(fellowship_df, two_towers_df, return_df) %>%  #stack all on top of each other
  janitor::clean_names() %>% 
  pivot_longer(
    female:male,
    names_to = "sex",
    values_to = "words"
  ) %>% 
  relocate(movie, .after = race) # just relocate movie after race column
```

# joins

Look at RAS data

``` r
litters_df = 
  read_csv("data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  separate(group, into = c("dose", "day_of_tx"), 3) %>% # 3 letters and split
  relocate(litter_number) %>% 
  mutate(dose = str_to_lower(dose))
```

    ## Rows: 49 Columns: 8

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df = 
  read_csv("data/FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex, `1` = "male", `2` = "female")) # recode, putting backticks is a way to force R to recognize the character
```

    ## Rows: 313 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Let’s join this up!

``` r
# left join to pups
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number") %>% # join by litter_number
  relocate(litter_number, dose, day_of_tx)
# fas_df %>% view
```

Homework 05: Factor and figure management
================
Stephen Chignell
October 19, 2018

-   [Overview](#overview)
-   [Part 1: Factor management](#part-1-factor-management)
-   [Exercise using gapminder](#exercise-using-gapminder)
    -   [Load libraries](#load-libraries)
    -   [Look at the data](#look-at-the-data)
    -   [Drop Oceania](#drop-oceania)
    -   [Reorder the levels](#reorder-the-levels)

Overview
--------

**Goals:**

-   Reorder a factor in a principled way based on the data and demonstrate the effect in arranged data and in figures.
-   Write some data to file and load it back into R.
-   Improve a figure (or make one from scratch), using new knowledge, e.g., control the color scheme, use factor levels, smoother mechanics.
-   Make a plotly visual.
-   Implement visualization design principles.

Part 1: Factor management
-------------------------

**Overview of factors**

Factors are “truly categorical” variables. They are vectors that:

-   have character entries on the surface (i.e. category/class name)
-   have integers underneath (i.e. code for computer to keep track)
-   have different levels (i.e., number of unique categories/classes)

Exercise using gapminder
------------------------

We will use the gapminder dataset to explore working with factors in the following ways:

-   Drop factor / levels
-   Reorder levels based on knowledge from data

#### Load libraries

``` r
suppressPackageStartupMessages(library(tidyr))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(ggplot2))
suppressPackageStartupMessages(library(knitr))
suppressPackageStartupMessages(library(forcats))
suppressPackageStartupMessages(library(gapminder))
```

#### Look at the data

``` r
str(gapminder)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    1704 obs. of  6 variables:
    ##  $ country  : Factor w/ 142 levels "Afghanistan",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ continent: Factor w/ 5 levels "Africa","Americas",..: 3 3 3 3 3 3 3 3 3 3 ...
    ##  $ year     : int  1952 1957 1962 1967 1972 1977 1982 1987 1992 1997 ...
    ##  $ lifeExp  : num  28.8 30.3 32 34 36.1 ...
    ##  $ pop      : int  8425333 9240934 10267083 11537966 13079460 14880372 12881816 13867957 16317921 22227415 ...
    ##  $ gdpPercap: num  779 821 853 836 740 ...

This confirms that we have two factors, `country` (142 levels), and `continent` (5 levels).

#### Drop Oceania

Suppose we wanted to remove observations in the continent of **Oceania** from the dataset.

To start, we should review the names of the level using the `nlevels` function:

``` r
levels(gapminder$continent)
```

    ## [1] "Africa"   "Americas" "Asia"     "Europe"   "Oceania"

Next we filter the data. We can remove unused levels at the same time by piping the filter results into the `droplevels()` function:

``` r
gap_sans_OCE <- gapminder %>% 
  filter(continent != "Oceania") %>% 
  droplevels()
```

Now let's confirm there are no longer any "Oceania" rows:

``` r
NROW(gap_sans_OCE %>% 
  filter(continent == "Oceania"))
```

    ## [1] 0

Looks good! We can also check how many rows were dropped:

``` r
# Subtract to find difference
NROW(gapminder) - NROW(gap_sans_OCE)
```

    ## [1] 24

We have now confirmed that all "Oceania" rows (n=24) have been removed from the original dataset. What about the levels?

``` r
levels(gap_sans_OCE$continent)
```

    ## [1] "Africa"   "Americas" "Asia"     "Europe"

Great! We have successfully removed the unused levels.

#### Reorder the levels

If we were to create a plot of the factor `continent`

``` r
cont <- gap_sans_OCE$continent
```

We see that it is plotted in the order of the factor levels:

``` r
qplot(cont)
```

![](hw05-schignel_files/figure-markdown_github/unnamed-chunk-3-1.png)

Suppose we wanted to plot life expectancy in Asia.

``` r
gap_asia <- gapminder %>% 
  filter(continent == "Asia") %>% 
  droplevels()
```

``` r
gap_asia %>% 
  group_by(country) %>% 
  summarize(maxlifeExp = max(lifeExp))
```

    ## # A tibble: 33 x 2
    ##    country          maxlifeExp
    ##    <fct>                 <dbl>
    ##  1 Afghanistan            43.8
    ##  2 Bahrain                75.6
    ##  3 Bangladesh             64.1
    ##  4 Cambodia               59.7
    ##  5 China                  73.0
    ##  6 Hong Kong, China       82.2
    ##  7 India                  64.7
    ##  8 Indonesia              70.6
    ##  9 Iran                   71.0
    ## 10 Iraq                   65.0
    ## # ... with 23 more rows

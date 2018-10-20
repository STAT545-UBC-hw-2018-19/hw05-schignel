Homework 05: Factor and figure management
================
Stephen Chignell
October 19, 2018

-   [Overview](#overview)
-   [Part 1: Factor management](#part-1-factor-management)
    -   [Load libraries](#load-libraries)
    -   [Look at the data](#look-at-the-data)
    -   [Drop Oceania](#drop-oceania)
    -   [Reorder the levels of `country` or `continent`](#reorder-the-levels-of-country-or-continent)
-   [Part 2: File I/O](#part-2-file-io)
    -   [Filter and arrange data](#filter-and-arrange-data)
    -   [Write to .CSV](#write-to-.csv)
    -   [Re-import from .csv](#re-import-from-.csv)
-   [Part 3: Visualization design](#part-3-visualization-design)
    -   [Old Graph](#old-graph)
    -   [New Graph](#new-graph)
    -   [Export to plotly](#export-to-plotly)
    -   [Reflections](#reflections)
-   [Part 4: Writing figures to file](#part-4-writing-figures-to-file)

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
suppressPackageStartupMessages(library(plotly))
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

#### Reorder the levels of `country` or `continent`

Suppose we wanted to plot mean life expectancy for each continent:

``` r
mean_pop <- gapminder %>%
  group_by(continent) %>% 
  summarize(mean.pop = mean(pop)) 
knitr::kable(mean_pop)
```

| continent |  mean.pop|
|:----------|---------:|
| Africa    |   9916003|
| Americas  |  24504795|
| Asia      |  77038722|
| Europe    |  17169765|
| Oceania   |   8874672|

Let's plot the reordered data:

``` r
ggplot(mean_pop, aes(continent, mean.pop)) + 
  geom_point()
```

![](hw05-schignel_files/figure-markdown_github/plot%20mean_pop-1.png)

By default, the levels are ordered alphabetically, which does not help to highlight any patterns in the data.

We can change these to ascending order using the `fct_reorder` function:

``` r
mean_pop %>%
  mutate(continent = fct_reorder(continent, mean.pop)) %>% 
  ggplot(aes(continent, mean.pop)) + 
  geom_point()
```

![](hw05-schignel_files/figure-markdown_github/mean_pop%20ordered-1.png)

Part 2: File I/O
----------------

Let's practice exporting and importing data. Say we wanted to look at life expectancies in American countries with a population over 5,000,000 for the year 2007:

#### Filter and arrange data

``` r
America_pop_2007 <- gapminder %>%
  filter(continent == "Americas", year == 2007, pop > 5000000) %>%
  arrange(pop) # arrange in ascending order
```

Let's view it as a table:

``` r
knitr::kable(America_pop_2007)
```

| country            | continent |  year|  lifeExp|        pop|  gdpPercap|
|:-------------------|:----------|-----:|--------:|----------:|----------:|
| Nicaragua          | Americas  |  2007|   72.899|    5675356|   2749.321|
| Paraguay           | Americas  |  2007|   71.752|    6667147|   4172.838|
| El Salvador        | Americas  |  2007|   71.878|    6939688|   5728.354|
| Honduras           | Americas  |  2007|   70.198|    7483763|   3548.331|
| Haiti              | Americas  |  2007|   60.916|    8502814|   1201.637|
| Bolivia            | Americas  |  2007|   65.554|    9119152|   3822.137|
| Dominican Republic | Americas  |  2007|   72.235|    9319622|   6025.375|
| Cuba               | Americas  |  2007|   78.273|   11416987|   8948.103|
| Guatemala          | Americas  |  2007|   70.259|   12572928|   5186.050|
| Ecuador            | Americas  |  2007|   74.994|   13755680|   6873.262|
| Chile              | Americas  |  2007|   78.553|   16284741|  13171.639|
| Venezuela          | Americas  |  2007|   73.747|   26084662|  11415.806|
| Peru               | Americas  |  2007|   71.421|   28674757|   7408.906|
| Canada             | Americas  |  2007|   80.653|   33390141|  36319.235|
| Argentina          | Americas  |  2007|   75.320|   40301927|  12779.380|
| Colombia           | Americas  |  2007|   72.889|   44227550|   7006.580|
| Mexico             | Americas  |  2007|   76.195|  108700891|  11977.575|
| Brazil             | Americas  |  2007|   72.390|  190010647|   9065.801|
| United States      | Americas  |  2007|   78.242|  301139947|  42951.653|

#### Write to .CSV

Let's practice exporting to a comma separated values (.csv) file:

``` r
# set row.names to FALSE to avoide creation of extra ID column
write.csv(America_pop_2007, file = "America_pop_2007_5M.csv", row.names = FALSE) 
```

We can see that the .csv was successfully exported to the project folder and the data are correct in Microsoft Excel.

Does its structure survive if we re-import it into R?

#### Re-import from .csv

``` r
America_pop_2007_READ <- read.csv("America_pop_2007_5M.csv")
knitr::kable(America_pop_2007_READ)
```

| country               | continent    |     year|     lifeExp|           pop|                                 gdpPercap|
|:----------------------|:-------------|--------:|-----------:|-------------:|-----------------------------------------:|
| Nicaragua             | Americas     |     2007|      72.899|       5675356|                                  2749.321|
| Paraguay              | Americas     |     2007|      71.752|       6667147|                                  4172.838|
| El Salvador           | Americas     |     2007|      71.878|       6939688|                                  5728.354|
| Honduras              | Americas     |     2007|      70.198|       7483763|                                  3548.331|
| Haiti                 | Americas     |     2007|      60.916|       8502814|                                  1201.637|
| Bolivia               | Americas     |     2007|      65.554|       9119152|                                  3822.137|
| Dominican Republic    | Americas     |     2007|      72.235|       9319622|                                  6025.375|
| Cuba                  | Americas     |     2007|      78.273|      11416987|                                  8948.103|
| Guatemala             | Americas     |     2007|      70.259|      12572928|                                  5186.050|
| Ecuador               | Americas     |     2007|      74.994|      13755680|                                  6873.262|
| Chile                 | Americas     |     2007|      78.553|      16284741|                                 13171.639|
| Venezuela             | Americas     |     2007|      73.747|      26084662|                                 11415.806|
| Peru                  | Americas     |     2007|      71.421|      28674757|                                  7408.906|
| Canada                | Americas     |     2007|      80.653|      33390141|                                 36319.235|
| Argentina             | Americas     |     2007|      75.320|      40301927|                                 12779.380|
| Colombia              | Americas     |     2007|      72.889|      44227550|                                  7006.580|
| Mexico                | Americas     |     2007|      76.195|     108700891|                                 11977.575|
| Brazil                | Americas     |     2007|      72.390|     190010647|                                  9065.801|
| United States         | Americas     |     2007|      78.242|     301139947|                                 42951.653|
| Great! The data are s | till arrange |  d by in|  trecing po|  pulation, so|  it surved the write-out/read-in process.|

**Note**: To write or read a .csv outside of the project folder, you need to enter in the full file path.

Part 3: Visualization design
----------------------------

In this section I reproduce an old figure and give it some new life through the techniques I've learned in the last couple of weeks of lecture.

#### Old Graph

Below is the code and graph ("as is") for a figure I was very proud of from Homework 02:

``` r
old_graph <- select(gapminder, gdpPercap, lifeExp, pop, year, continent) %>% #subsetting data
  filter(year > 1957, continent=="Europe"|continent=="Africa") %>% #filter by criteria
  ggplot(aes(lifeExp, gdpPercap))+ #piped to ggplot
  geom_point(aes(color=continent, size=pop, alpha=0.1)) + # add aesthetics 
  xlab("Life Expectancy")+
  ylab("GDP per Capita")
old_graph
```

![](hw05-schignel_files/figure-markdown_github/old%20graph-1.png)

#### New Graph

``` r
new_graph <- select(gapminder, gdpPercap, lifeExp, pop, year, continent) %>% 
  filter(year > 1957, continent == "Europe"|continent == "Africa") %>%   ggplot(aes(lifeExp, gdpPercap)) +
  geom_point(aes(color = continent, size = pop, alpha = 0.1)) +
  labs(title = "Life expectancy for each continent", 
       x = "Life Expectancy", 
       y = "GDP per Capita") + 
  theme_bw()+
  theme(axis.text.x  = element_text(vjust=0.6, size=10),
        axis.text = element_text(size = 10)) +
  scale_colour_brewer(palette = "Set1") + # use brewer scheme
  geom_smooth(method="lm") #add a linear regression line
new_graph
```

![](hw05-schignel_files/figure-markdown_github/new%20graph-1.png)

#### Export to plotly

``` r
# commented to avoid creating destroying .md 
#new_graph_plotly <- new_graph
#ggplotly(new_graph_plotly) 
```

#### Reflections

One of the first changes I made was to remove the unnecessary comments, as these did not provide extra information and make the code difficult to read, violating the recommendations by Hadley Wickham in the [tidyverse style guide](http://style.tidyverse.org/). Also in line with the style guide, I added spaces before and after each of the "=" and "+" in order to improve legibility.

I also added a linear regression line, which is particularly nice to include on the plotly plot, as it allows you to identify the predicted values from the model interactively. I could see this--along with the fact that plotly provide access to the specific attribute information for each point--being extremely useful for exploring and presenting data in the future.

Part 4: Writing figures to file
-------------------------------

With `ggsave()` it's easy to export a figure to as a file on your local computer:

``` r
ggsave("new_graph.png", new_graph)
```

    ## Saving 7 x 5 in image

We also may want to adjust the dimensions and resolution:

``` r
ggsave("new_graph_5x4.png", width = 5, height = 4, dpi = 150)
```

We can also save to vector format with .pdf

``` r
ggsave("new_graph.pdf", new_graph)
```

    ## Saving 7 x 5 in image

The ggsave( ) function will, by default, save the most recent plot in R. To save a different image, we have to make this explicit in the ggsave code. For example, if we wanted to save the old graph (see above), we would make the following changes to the code:

``` r
ggsave("old_graph.png", plot = old_graph)
```

    ## Saving 7 x 5 in image

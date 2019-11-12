---
layout: single
title: Exploring Switzerland Official Data
excerpt: Introducing my new R package {BFS} and making easily a Swiss map.
permalink: /swiss-data/
date: 2019-11-07 10:00:00 -0600  
author: lgnbhl
tags: rstats dataviz plotly
header:
  og_image: /images/swiss-data_map-1.png
---

![](/images/swiss-data_map-1.png)

The [Swiss Federal Statistical
Office](https://www.bfs.admin.ch/bfs/en/home.html), or BFS from
“Bundesamt für Statistik” in German, provides a rich public database. As
Swiss citizen and R enthousiast, I wanted to easily access its datasets
directly from R. So I created the [BFS package](https://felixluginbuhl.com/BFS/).

In this article, I will show how to use my `BFS` package to easily
search and download datasets from the Swiss Federal Statistical Office.
We will then quickly explore a dataset and plot a map of Swiss
municipalities, the lowest level of administrative division in
Switzerland.

As always the code is fully reproducible, so you can get it 
from my [Github](https://github.com/lgnbhl/blogposts/tree/master/swiss-data) account
or on my online [RStudio Cloud](https://rstudio.cloud/project/673254) session.

Getting the data
----------------

To use my `BFS` package, we should begin by downloading information
related to all available datasets of the Swiss Federal Statistical
Office Catalogue. We can get the BFS metadata in German (“de”), French
(“fr”), Italian (“it”) and English (“en”)[^1].

``` r
# install/load needed R packages
if (!require("pacman")) install.packages("pacman")
pacman::p_load(tidyverse, scales, colorspace, plotly, RSwissMaps, BFS)

# setting light theme
theme_set(theme_light())

# Get BFS metadata in German
meta_de <- bfs_get_metadata(language = "de")
meta_de
```

    ## # A tibble: 676 x 5
    ##    title          observation_period   published url           url_px      
    ##    <chr>          <chr>                <chr>     <chr>         <chr>       
    ##  1 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  2 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  3 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  4 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  5 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  6 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  7 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  8 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  9 Hotellerie: A… Dargestellter Zeitr… 04.11.20… https://www.… https://www…
    ## 10 Hotellerie: A… Dargestellter Zeitr… 04.11.20… https://www.… https://www…
    ## # … with 666 more rows

We currently have access to 676 BFS datasets.

The challenge today is to plot a detailed map of Switzerland containing
all 2’212 Swiss municipalites, or “gemeinde” in German. How many current
BFS datasets contain the word “gemeinde” in their title? Let’s find out
using the `bfs_search()` function.

``` r
meta_de_gemeinde <- bfs_search("gemeinde", data = meta_de)
meta_de_gemeinde
```

    ## # A tibble: 34 x 5
    ##    title          observation_period   published url           url_px      
    ##    <chr>          <chr>                <chr>     <chr>         <chr>       
    ##  1 Ausländische … Dargestellter Zeitr… 07.11.20… https://www.… https://www…
    ##  2 Hotellerie: A… Dargestellter Zeitr… 04.11.20… https://www.… https://www…
    ##  3 Hotellerie: A… Dargestellter Zeitr… 04.11.20… https://www.… https://www…
    ##  4 Nationalratsw… Dargestellter Zeitr… 19.09.20… https://www.… https://www…
    ##  5 Arbeitsstätte… Dargestellter Zeitr… 22.08.20… https://www.… https://www…
    ##  6 Kinoinfrastru… Dargestellter Zeitr… 14.03.20… https://www.… https://www…
    ##  7 Volksabstimmu… Dargestellter Zeitr… 04.03.20… https://www.… https://www…
    ##  8 Öffentliche B… Dargestellter Zeitr… 20.12.20… https://www.… https://www…
    ##  9 Gründung, Sch… Dargestellter Zeitr… 10.12.20… https://www.… https://www…
    ## 10 Neu gegründet… Dargestellter Zeitr… 05.10.20… https://www.… https://www…
    ## # … with 24 more rows

We got 34 different datasets related to Swiss municipalities. I am
interested by the one related to the cross-border worker (“Ausländische
Grenzgänger/innen” in German) at the first row. Let’s download it using
the `bfs_get_dataset()` function.

``` r
# browseURL(meta_de_gemeinde$url[1]) # open related webpage
meta_de_gemeinde$title[1] # print title
```

    ## [1] "Ausländische Grenzgänger/innen nach Geschlecht und Arbeitsgemeinde"

``` r
meta_de_gemeinde1 <- bfs_search("Ausländische Grenzgänger/innen nach Geschlecht und Arbeitsgemeinde", 
                                meta_de_gemeinde)

data_bfs <- bfs_get_dataset(meta_de_gemeinde1$url_px[1])
data_bfs
```

    ## # A tibble: 638,970 x 4
    ##    quartal arbeitsgemeinde geschlecht           value
    ##    <fct>   <fct>           <fct>                <dbl>
    ##  1 1996Q1  Schweiz         Geschlecht - Total 143526.
    ##  2 1996Q2  Schweiz         Geschlecht - Total 142744.
    ##  3 1996Q3  Schweiz         Geschlecht - Total 141538.
    ##  4 1996Q4  Schweiz         Geschlecht - Total 139210.
    ##  5 1997Q1  Schweiz         Geschlecht - Total 137687.
    ##  6 1997Q2  Schweiz         Geschlecht - Total 135740.
    ##  7 1997Q3  Schweiz         Geschlecht - Total 136010.
    ##  8 1997Q4  Schweiz         Geschlecht - Total 134580.
    ##  9 1998Q1  Schweiz         Geschlecht - Total 134379.
    ## 10 1998Q2  Schweiz         Geschlecht - Total 134552.
    ## # … with 638,960 more rows

Note that the developing version of `BFS` leverage the new `pins`
package to save all the downloaded Swiss datasets in the same cache
folder, accessible using the `bfs_open_dir()` function.

That’s all for my new `BFS` package.

Exploring the data
------------------

Using the [Tidyverse](https://www.tidyverse.org/) workflow, we can now
performe a quick exploratory data analysis.

Let’s begin with a glimpse at the data.

``` r
glimpse(data_bfs)
```

    ## Observations: 638,970
    ## Variables: 4
    ## $ quartal         <fct> 1996Q1, 1996Q2, 1996Q3, 1996Q4, 1997Q1, 1997Q2, …
    ## $ arbeitsgemeinde <fct> Schweiz, Schweiz, Schweiz, Schweiz, Schweiz, Sch…
    ## $ geschlecht      <fct> Geschlecht - Total, Geschlecht - Total, Geschlec…
    ## $ value           <dbl> 143526.0, 142744.5, 141537.9, 139210.4, 137686.8…

The dataset contains information about the number of cross-border workers
by quarter (`quartal`), Swiss municipality (`arbeitsgemeinde`) and
gender (`geschlecht`).

Notice that `value` is a pondered value: each worker get a weighted
point between 0-1 according to the number of hours of works he/she is
doing (see more
[here](https://www.bfs.admin.ch/bfs/de/home/statistiken/arbeit-erwerb/erhebungen/ggs.html)).
It is therefore more appropriate to speak about “cross-border work” as
the value of two men working half time in a Swiss municipality is equal
to a full time cross-border working woman.

I am curious to learn more about the gender ratio of cross-border work
by municipality and its evolution over the years. Let’s build a new
`gender_ratio` variable.

``` r
data_bfs_ratio <- data_bfs %>%
  tidyr::pivot_wider(names_from = "geschlecht", values_from = "value") %>%
  rename(quarter = quartal, 
         municipality = arbeitsgemeinde,
         man = Mann, woman = Frau, 
         gender_total = `Geschlecht - Total`) %>%
  mutate(municipality = str_remove_all(municipality, "\\.|^\\- ")) %>% # cleaning
  mutate(gender_ratio = man / gender_total * 100) %>%
  arrange(desc(quarter))

data_bfs_ratio
```

    ## # A tibble: 212,990 x 6
    ##    quarter municipality      gender_total        man     woman gender_ratio
    ##    <fct>   <chr>                    <dbl>      <dbl>     <dbl>        <dbl>
    ##  1 2019Q3  Schweiz              325291.   209406.      1.16e+5         64.4
    ##  2 2019Q3  Zürich                10342.     7557.      2.78e+3         73.1
    ##  3 2019Q3  Aeugst am Albis           0         0       0.             NaN  
    ##  4 2019Q3  Affoltern am Alb…        19.3      14.5     4.75e+0         75.4
    ##  5 2019Q3  Bonstetten                1.74      0.874   8.62e-1         50.4
    ##  6 2019Q3  Hausen am Albis           0         0       0.             NaN  
    ##  7 2019Q3  Hedingen                 42.1      41.5     5.99e-1         98.6
    ##  8 2019Q3  Kappel am Albis           0         0       0.             NaN  
    ##  9 2019Q3  Knonau                    0         0       0.             NaN  
    ## 10 2019Q3  Maschwanden               0         0       0.             NaN  
    ## # … with 212,980 more rows

We see that the gender ratio of cross-border workers for the 2nd quarter
of 2019 in Switzerland is 64.2% (but 73% in Zürich).

Does it mean we have strong cantonal gender disparities in terms of
cross-border work?

``` r
# Create table to join later to bfs_data
# ref: https://en.wikipedia.org/wiki/Data_codes_for_Switzerland#Cantons
cantons <- tibble::tribble(
  ~canton, ~code, ~id_can,
  "Aargau", "AG", 19,
  "Appenzell Innerrhoden", "AI", 15,
  "Appenzell Ausserrhoden", "AR", 16,
  "Bern", "BE", 2,
  "Basel-Landschaft", "BL", 13,
  "Basel-Stadt", "BS", 12,
  "Fribourg", "FR", 10,
  "Genève", "GE", 25,
  "Glarus", "GL", 8,
  "Graubünden", "GR", 18,
  "Jura", "JU", 26,
  "Luzern", "LU", 3,
  "Neuchâtel", "NE", 24,
  "Nidwalden", "NW", 7,
  "Obwalden", "OW", 6,
  "St Gallen", "SG", 17,
  "Schaffhausen", "SH", 14,
  "Solothurn", "SO", 11,
  "Schwyz", "SZ", 5,
  "Thurgau", "TG", 20,
  "Ticino", "TI", 21,
  "Uri", "UR", 4,
  "Vaud", "VD", 22,
  "Valais", "VS", 23,
  "Zug", "ZG", 9,
  "Zürich", "ZH", 1,
)

data_bfs_ratio_annualized <- data_bfs_ratio %>%
  mutate(year = str_extract(quarter, "^.{4}"),
         year = as.numeric(year)) %>%
  group_by(municipality, year) %>%
  summarise(gender_ratio_annualized = mean(gender_ratio, na.rm = TRUE)) %>%
  ungroup() %>%
  mutate(gender_per = gender_ratio_annualized/100)

data_bfs_ratio_annualized %>%
  inner_join(cantons, by = c("municipality" = "canton")) %>% # join table
  ggplot(aes(x = year, y = gender_per, color = code)) +
  geom_line() +
  geom_line(data = filter(data_bfs_ratio_annualized, municipality == "Schweiz"),
             color = "red2", linetype = "dashed", size = 1) +
  scale_y_continuous(label = percent) +
  annotate("text", x = 2016, y = 0.54, size = 3.5, label = "National gender ratio") +
  geom_segment(aes(x = 2016, y = 0.551, xend = 2016, yend = 0.635), 
               color = "black", size = 0.2, 
               arrow = arrow(length = unit(0.2, "cm"))) +
  labs(title = "Proportion of Men in Cross-Border Workforce",
       subtitle = "Switzerland, 1995-2019",
       color = "Canton",
       x = "", y = "",
       caption = "quarter annualized - Data source: BFS")
```

![](/images/swiss-data_cantonal_gender_ratio-1.png)

It looks like we have different gender ratio levels according to the
Swiss canton. However, it is hard to see clearly and to categorise the
cantons by group.

Let’s make a time serie clustering to get the categories. I will reuse
some code of the excellent
[blogpost](https://www.brodrigues.co/blog/2019-10-12-cluster_ts/) of
Bruno Rodrigues to perform a time-series k-means clustering.

``` r
set.seed(1111)

# Only since 2007 as missing values before for some cantons
data_bfs_wide <- data_bfs_ratio_annualized %>%
  inner_join(cantons, by = c("municipality" = "canton")) %>% # join table
  filter(year > 2006) %>%
  select(municipality, year, gender_ratio_annualized) %>%
  pivot_wider(names_from = year, values_from = gender_ratio_annualized)

wss <- map_dbl(1:6, ~{kmeans(select(data_bfs_wide, -municipality), .)$tot.withinss})

elbow_df <- as.data.frame(cbind("n_clust" = 1:6, "wss" = wss))

ggplot(elbow_df) +
  geom_line(aes(y = wss, x = n_clust))
```

![](/images/swiss-data_n_clust-1.png)

The optimal number of categories seems to be four. Let’s cluser our
times series in four different groups.

``` r
clusters <- kmeans(select(data_bfs_wide, -municipality), centers = 4)

gg_plot <- data_bfs_wide %>% 
  mutate(cluster = clusters$cluster) %>%
  pivot_longer(cols = c(-municipality, -cluster), 
               names_to = "year", 
               values_to = "gender ratio") %>%
  mutate(cluster = as.factor(cluster)) %>%
  rename(canton = municipality) %>%
  ggplot() +
  geom_line(aes(y = `gender ratio`, x = year, 
                group = canton, colour = cluster), 
            show.legend = FALSE) +
  facet_wrap(~cluster, nrow = 1) +
  scale_color_brewer(palette = "Set2") +
  scale_x_discrete(breaks = seq(2007, 2019, by = 3)) +
  guides(color = FALSE) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  labs(title = "Proportion of Men in Cross-Border Workforce in Swiss Cantons",
       x = "")

plotly::ggplotly(gg_plot) %>%
  hide_legend()
```

<iframe seamless src="/images/swiss-data_CantonalGenderRatioCrossBording.html" width="100%" height="500" frameborder="0"></iframe>

Put your mouse over the interactive plot above. You can discover the
canton’s name and related gender ratio for each time serie of the four
clusters. Note that we have only kept data from 2007 as we had missing
values for the previous years in some cantons.

That’s all for the exploration of the cantonal level. What about the
municipal level?

Mapping time!
-------------

Let’s plot the gender ratio of cross-bording work by Swiss municipality
for the last year available, i.e. 2019.

With the great `RSwissMaps` package of David Zumbach, it is possible to
create a map of Swizterland with only a few lines of code.

``` r
# the BFS id of all municipalities are inside the RSwissMaps
# the data inside RSwissMaps is taken from year 2016
bfs_id_mun <- RSwissMaps::mun.template(year = 2016)

data_bfs_2018 <- data_bfs_ratio_annualized %>%
  left_join(bfs_id_mun, by = c("municipality" = "name")) %>%
  filter(year == 2019)

mun.plot(data_bfs_2018$bfs_nr, 
         data_bfs_2018$gender_per, 
         year = 2016) +
  scale_fill_viridis_c(labels = percent, direction = -1) +
  theme(legend.position = "right") +
  labs(title = "Proportion of Men in Cross-Border Workforce in Switzerland",
       subtitle = "More women in green-yellow, 2019",
       fill = "",
       caption = "Quarterly annualized - Data Source: BFS")
```

![](/images/swiss-data_map-1.png)

Data shows that central Swiss municipalites also have some cross-bording
workers. If cross-bording work is mainly done by men, a few Swiss
municipalities have more women working as cross-bording worker.

Let me know what you think about my new `BFS` package and feel free to
contribute or make a pull request
[here](https://github.com/lgnbhl/BFS/issues).

[^1]: English and Italian have less datasets available.

{% capture notice-text %}

* For updates of recent blog posts, follow me on [Twitter](https://twitter.com/FelixLuginbuhl).
* For reproducing my data analysis, go on my [Github page](https://github.com/lgnbhl/blogposts).
* Curious about what I can do for your organisation? Have a look at [my services](https://felixluginbuhl.com).
{% endcapture %}

<div class="notice--primary">
  <h4>Thanks for reading!</h4>
  {{ notice-text | markdownify }}
</div>

<!-- Begin Mailchimp Signup Form -->
<link href="//cdn-images.mailchimp.com/embedcode/horizontal-slim-10_7.css" rel="stylesheet" type="text/css">
<style type="text/css">
	#mc_embed_signup{background:#transparent; clear:left; width:100%;}
	/* Add your own Mailchimp form style overrides in your site stylesheet or in this style block.
	   We recommend moving this block and the preceding CSS link to the HEAD of your HTML file. */
</style>
<div id="mc_embed_signup">
<form action="https://protonmail.us20.list-manage.com/subscribe/post?u=76318d6fc4f4bf252f3716c14&amp;id=bf5bf4210c" method="post" id="mc-embedded-subscribe-form" name="mc-embedded-subscribe-form" class="validate" target="_blank" novalidate>
    <div id="mc_embed_signup_scroll">
	<label for="mce-EMAIL">Like my articles? Subscribe via e-mail.</label>
	<input type="email" value="" name="EMAIL" class="email" id="mce-EMAIL" placeholder="E-mail address" required>
    <!-- real people should not fill this in and expect good things - do not remove this or risk form bot signups-->
    <div style="position: absolute; left: -5000px;" aria-hidden="true"><input type="text" name="b_76318d6fc4f4bf252f3716c14_bf5bf4210c" tabindex="-1" value=""></div>
    <div class="clear"><input type="submit" value="Subscribe" name="subscribe" id="mc-embedded-subscribe" class="button"></div>
    </div>
</form>
</div>
<!--End mc_embed_signup-->

---
layout: single
title: Most Recurring Word on each Country's Wikipedia Page
excerpt: Scraping 193 Wikipedia pages and creating an interactive Leaflet map.
permalink: /leaflet-map/
date: 2019-12-16 10:00:00 -0600  
author: lgnbhl
tags: rstats dataviz map
header:
  og_image: /images/screenshot_worldmap_leaflet.png
---

![](/images/screenshot_worldmap_leaflet.png)

A friend recently offered me a fun book: *Brilliant maps. An atlas for
curious minds*. One of the maps, called *Most recurring word on each
country’s English Wikipedia page*, particularly caught my attention.
This world map shows the most frequent word of each country’s English
Wikipedia page. An old version of this infographic is available
online:

![](https://mk0brilliantmaptxoqs.kinstacdn.com/wp-content/uploads/wikipedia-recurring.png)

The authors specify in a footnote that they excluded “country”, linking
words, demonyms and “government”. Surprisingly the most recurring word
of the English Wikipedia page are often either revealing or funny. For
example, the most recurring word of the English Wikipedia page of the
USA is, according to the infographic, “war”. For China, it would be
“dynasty”, Australia “new” and Russia “soviet”.

These results can quite easily be checked using R programming. Let’s do
this\!

## Scraping 193 Wikipedia pages

Firstly, we need a list of all Wikipedia pages we want to scrap. The
[Wikipedia
page](https://en.wikipedia.org/w/index.php?title=Member_states_of_the_United_Nations&oldid=926737595)
of the 193 United Nations states membres will do the trick.

``` r
library(tidyverse)
library(rvest)

wikilink_permalink <- "https://en.wikipedia.org/w/index.php?title=Member_states_of_the_United_Nations&oldid=926737595"

# Extract wikitable
wikitable <- wikilink_permalink %>%
  read_html() %>%
  html_nodes("table") %>%
  .[[2]]

# Get wikilinks of all UN members
wikilinks <- wikitable %>%
  html_nodes("a") %>%
  html_attr("href") %>%
  keep(startsWith(., "/wiki/")) %>%
  discard(endsWith(., "and_the_United_Nations")) %>%
  str_remove("/wiki/Viet_Nam") %>% # duplicate with "/wiki/Vietnam"
  str_remove("/wiki/The_Gambia") %>% # duplicate with "/wiki/Gambia"
  str_remove("/wiki/United_Nations_Assistance_Mission_in_Afghanistan") %>%
  str_remove("/wiki/United_Nations_Assistance_Mission_in_Afghanistan") %>%
  str_remove("/wiki/Costa_Rica_in_the_United_Nations") %>%
  str_remove("/wiki/Flag") %>%
  str_replace("S%C3%A3o_Tom%C3%A9_and_Pr%C3%ADncipe", "Sao_Tome_and_Principe") %>%
  na_if("") %>%
  unique() %>%
  na.omit() %>%
  paste0("https://en.wikipedia.org", .)

table <- wikitable %>%
  html_table() %>%
  as_tibble() %>%
  mutate(
    wikilinks = wikilinks,
    page_name = str_remove(wikilinks, ".*/")
  ) %>%
  janitor::clean_names() %>%
  mutate_all(funs(str_remove_all(., "\\[note [0-9]+]|\\[[0-9]+]"))) %>%
  select(member_state = member_state_7_13_14, wikilinks, page_name)

table
```

    ## # A tibble: 193 x 3
    ##    member_state       wikilinks                                 page_name       
    ##    <chr>              <chr>                                     <chr>           
    ##  1 Afghanistan        https://en.wikipedia.org/wiki/Afghanistan Afghanistan     
    ##  2 Albania            https://en.wikipedia.org/wiki/Albania     Albania         
    ##  3 Algeria            https://en.wikipedia.org/wiki/Algeria     Algeria         
    ##  4 Andorra            https://en.wikipedia.org/wiki/Andorra     Andorra         
    ##  5 Angola             https://en.wikipedia.org/wiki/Angola      Angola          
    ##  6 Antigua and Barbu… https://en.wikipedia.org/wiki/Antigua_an… Antigua_and_Bar…
    ##  7 Argentina          https://en.wikipedia.org/wiki/Argentina   Argentina       
    ##  8 Armenia            https://en.wikipedia.org/wiki/Armenia     Armenia         
    ##  9 Australia          https://en.wikipedia.org/wiki/Australia   Australia       
    ## 10 Austria            https://en.wikipedia.org/wiki/Austria     Austria         
    ## # … with 183 more rows

Now let’s use the official Wikipedia API available through the
{WikipediR} R package to smoothly download all the related Wikipedia
pages and add them into our dataset.

``` r
library(WikipediR)
library(purrr)
library(tidyr)

table_txt <- table %>%
  mutate(txt = map(page_name, ~ WikipediR::page_content("en", "wikipedia",
    page_name = .x
  )$parse$text$`*`[[1]])) %>%
  mutate(
    txt = map(txt, unlist),
    txt = map(txt, read_html),
    txt = map(txt, html_text)
  ) %>%
  unnest_longer(txt)

# write_csv(table_txt, "data/table_txt.csv")
glimpse(table_txt)
```

    ## Observations: 193
    ## Variables: 4
    ## $ member_state <chr> "Afghanistan", "Albania", "Algeria", "Andorra", "Angola"…
    ## $ wikilinks    <chr> "https://en.wikipedia.org/wiki/Afghanistan", "https://en…
    ## $ page_name    <chr> "Afghanistan", "Albania", "Algeria", "Andorra", "Angola"…
    ## $ txt          <chr> "A landlocked south-central Asian country\nFor the Japan…

Nice\! We have all the data we need.

## Getting the most recurring words

We want now to clean the data and extract the most recurring word of
each Wikipedia page. We will follow the methodology by removing
“country”, linking words, demonyms and “government”.

Let’s get the [demonyms of the
countries](https://en.wikipedia.org/wiki/List_of_adjectival_and_demonymic_forms_for_countries_and_nations).

``` r
wiki_demonyms <- "https://en.wikipedia.org/wiki/List_of_adjectival_and_demonymic_forms_for_countries_and_nations"

table_demonyms <- wiki_demonyms %>%
  read_html() %>%
  html_nodes("table") %>%
  .[[1]] %>%
  html_table() %>%
  as_tibble() %>%
  mutate_all(funs(str_remove_all(., "\\[.]|[(.)]"))) %>%
  mutate_all(funs(str_to_lower)) %>%
  janitor::clean_names()

table_demonyms <- table_demonyms %>%
  mutate(
    adjectivals = str_trim(adjectivals),
    demonyms = str_trim(demonyms),
    adjectivals_unnested = str_split(adjectivals, ", | or | "),
    demonyms_unnested = str_split(demonyms, ", | or |/| ")
  ) %>%
  unnest_longer(adjectivals_unnested) %>%
  unnest_longer(demonyms_unnested)

table_demonyms
```

    ## # A tibble: 726 x 5
    ##    country_entity_n… adjectivals   demonyms   adjectivals_unne… demonyms_unnest…
    ##    <chr>             <chr>         <chr>      <chr>             <chr>           
    ##  1 abkhazia          abkhaz, abkh… abkhazians abkhaz            abkhazians      
    ##  2 abkhazia          abkhaz, abkh… abkhazians abkhazian         abkhazians      
    ##  3 afghanistan       afghan        afghans    afghan            afghans         
    ##  4 åland islands     åland island  åland isl… åland             åland           
    ##  5 åland islands     åland island  åland isl… åland             islanders       
    ##  6 åland islands     åland island  åland isl… island            åland           
    ##  7 åland islands     åland island  åland isl… island            islanders       
    ##  8 albania           albanian      albanians  albanian          albanians       
    ##  9 algeria           algerian      algerians  algerian          algerians       
    ## 10 american samoa    american sam… american … american          american        
    ## # … with 716 more rows

Now we can clean the dataset following the methodology of the authors,
i.e. removing “country”, linking words, demonyms and “government”.

``` r
library(tidytext)

table_tidy <- table_txt %>%
  tidytext::unnest_tokens(word, txt) %>%
  anti_join(tidytext::get_stopwords("en"), by = "word") %>%
  anti_join(table_txt %>%
    mutate(word = str_to_lower(member_state)), by = "word") %>%
  anti_join(table_txt %>%
    unnest_tokens(word, member_state), by = "word") %>%
  filter(!word %in% table_demonyms$country_entity_name) %>%
  filter(!word %in% paste0(table_demonyms$country_entity_name, "'s")) %>%
  filter(!word %in% table_demonyms$adjectivals_unnested) %>%
  filter(!word %in% table_demonyms$demonyms_unnested) %>%
  filter(!word %in% c("country", "government"))

table_tidy
```

    ## # A tibble: 2,326,428 x 4
    ##    member_state wikilinks                               page_name  word         
    ##    <chr>        <chr>                                   <chr>      <chr>        
    ##  1 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… landlocked   
    ##  2 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… asian        
    ##  3 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… manga        
    ##  4 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… see          
    ##  5 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… afghanis     
    ##  6 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… tan          
    ##  7 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… afghanistan.…
    ##  8 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… parser       
    ##  9 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… output       
    ## 10 Afghanistan  https://en.wikipedia.org/wiki/Afghanis… Afghanist… nobold       
    ## # … with 2,326,418 more rows

The results shows that additional stopwords should be added, such as
removing numbers, month names, and other additional words. I don’t think
we can really call this cleaning step “cheating”, considering the result
of this data analysis. Let’s do this.

``` r
table_tidy2 <- table_tidy %>%
  filter(!word %in% str_to_lower(month.name)) %>%
  filter(!word %in% c(0:2030)) %>%
  filter(!word %in% letters) %>%
  filter(!word %in% c(
    "u.s", "uk", "uae", "drc", "de", "en", "also",
    "retrieved", "archived", "original", "pdf", "edit", "isbn", "p", "pp",
    "output", "redirect", "page", "parser", "mw", "wayback", "main",
    "st", "al", "la", "per", "percent", "cent", "05", "cs1", "one", "two",
    "perú", "d'andorra", "china's", "syria", "brasil", "citation"
  ))
```

Let’s have a look at our most recurring words.

``` r
top1words <- table_tidy2 %>%
  count(member_state, word, sort = TRUE) %>%
  group_by(member_state) %>%
  top_n(1) %>%
  distinct(member_state, .keep_all = TRUE) %>%
  arrange(desc(member_state, n))

top1words
```

    ## # A tibble: 193 x 3
    ## # Groups:   member_state [193]
    ##    member_state                                         word           n
    ##    <chr>                                                <chr>      <int>
    ##  1 Zimbabwe                                             news          67
    ##  2 Zambia                                               rhodesia      40
    ##  3 Yemen                                                sana'a        87
    ##  4 Viet Nam                                             journal       95
    ##  5 Vanuatu                                              pacific       30
    ##  6 Uzbekistan                                           soviet        47
    ##  7 Uruguay                                              montevideo    51
    ##  8 United States of America                             war          131
    ##  9 United Republic of Tanzania                          zanzibar      58
    ## 10 United Kingdom of Great Britain and Northern Ireland world        127
    ## # … with 183 more rows

Yes, the most recurring word of the United States of America’s Wikipedia
page is “war” indeed.

But to which extend the word “war” is more recurring than other words?

``` r
table_tidy2 %>%
  filter(member_state == "United States of America") %>%
  count(word, sort = TRUE)
```

    ## # A tibble: 6,883 x 2
    ##    word           n
    ##    <chr>      <int>
    ##  1 war          131
    ##  2 world        130
    ##  3 history      106
    ##  4 federal      102
    ##  5 population   100
    ##  6 press         97
    ##  7 york          77
    ##  8 national      76
    ##  9 university    75
    ## 10 first         63
    ## # … with 6,873 more rows

Just one more occurence than the word “world”.

Let’s have a closer look at all the top 10 recurring words by country.

``` r
top10words <- table_tidy2 %>%
  count(member_state, word, sort = T) %>%
  group_by(member_state) %>%
  top_n(10) %>%
  arrange(member_state)

DT::datatable(
  top10words,
  options = list(pageLength = 10, dom = "ftpi"),
  rownames = FALSE,
  caption = "Top 10 recurring words on each country's English Wikipedia page"
)
```

<iframe seamless src="/images/DT_top10_2.html" width="100%" height="600" frameborder="0"></iframe>

What are the most recurring words in all our tops?

``` r
table_tidy2 %>%
  count(member_state, word) %>%
  group_by(member_state) %>%
  top_n(10) %>%
  ungroup() %>%
  count(word, sort = TRUE)
```

    ## # A tibble: 645 x 2
    ##    word              n
    ##    <chr>         <int>
    ##  1 world           174
    ##  2 population      138
    ##  3 national        126
    ##  4 international    76
    ##  5 first            53
    ##  6 war              53
    ##  7 president        51
    ##  8 century          35
    ##  9 article          33
    ## 10 university       32
    ## # … with 635 more rows

The words “world”, “population” and “national” seem good candidates to
be removed as stopwords. Let’s try to remove them\!

``` r
top1words_new <- table_tidy2 %>%
  filter(!word %in% c("world", "world's", "population", "national")) %>%
  count(member_state, word, sort = TRUE) %>%
  group_by(member_state) %>%
  top_n(1) %>%
  ungroup() %>%
  distinct(member_state, .keep_all = TRUE) %>%
  arrange(desc(member_state))

top1words_new %>%
  count(word, sort = TRUE)
```

    ## # A tibble: 133 x 2
    ##    word          n
    ##    <chr>     <int>
    ##  1 caribbean     9
    ##  2 pacific       9
    ##  3 soviet        8
    ##  4 president     7
    ##  5 war           7
    ##  6 century       6
    ##  7 europe        4
    ##  8 federal       3
    ##  9 military      3
    ## 10 atoll         2
    ## # … with 123 more rows

Seven countries have “war” as the most recurring word. Which are they?

``` r
top1words_new %>%
  filter(word == "war") %>%
  arrange(desc(n))
```

    ## # A tibble: 7 x 3
    ##   member_state                     word      n
    ##   <chr>                            <chr> <int>
    ## 1 Afghanistan                      war     138
    ## 2 United States of America         war     131
    ## 3 Syrian Arab Republic             war      68
    ## 4 Spain                            war      60
    ## 5 Democratic Republic of the Congo war      55
    ## 6 Libya                            war      46
    ## 7 Paraguay                         war      39

Quite interesting\!

Finally, let’s make our own map based on this new dataset.

## An interactive world map

As it is a blog article, let’s make our world map interactive.

``` r
library(countrycode)
library(leaflet)
library(rnaturalearth)
library(rnaturalearthdata)
library(htmltools)

world <- ne_countries(scale = "small", returnclass = "sf") %>%
  filter(continent != "Antarctica") %>%
  mutate(
    name = recode(name, "Greenland" = "Denmark"),
    iso_a3 = recode(iso_a3, "GRL" = "DNK")
  ) %>%
  select(name, iso_a3, geometry)

top1words_new$iso3c <- countrycode(top1words_new$member_state, "country.name", "iso3c")

world <- world %>%
  inner_join(top1words_new, by = c("iso_a3" = "iso3c"))

labels <- sprintf(
  "<strong>%s</strong><br/>%s",
  world$name, world$word
) %>% lapply(htmltools::HTML)

# reference: https://stackoverflow.com/a/52226825
tag.map.title <- tags$style(HTML("
  .leaflet-control.map-title { 
    transform: translate(-50%,20%);
    position: fixed !important;
    left: 50%;
    text-align: center;
    padding-left: 10px; 
    padding-right: 10px; 
    background: rgba(255,255,255,0.75);
    font-weight: bold;
    font-size: 22px;
  }
"))

title <- tags$div(
  tag.map.title, HTML("Most Recurring Word on each Country's Wikipedia Page")
)

leaflet(world) %>%
  addPolygons(
    weight = 1,
    fillOpacity = 0.3,
    highlight = highlightOptions(
      weight = 2,
      fillOpacity = 1
    ),
    label = labels,
    labelOptions = labelOptions(
      textsize = "15px"
    )
  ) %>%
  addControl(title, position = "topleft", className = "map-title")
```

<iframe seamless src="/images/worldmap_leaflet2.html" width="100%" height="400" frameborder="0"></iframe>

Hover your mouse on the world map and discover the most recurring words
of each country’s English Wikipedia page. Yes, they are quite different
from the [old
version](https://mk0brilliantmaptxoqs.kinstacdn.com/wp-content/uploads/wikipedia-recurring.png)
of *Brilliant Maps*.


{% capture notice-text %}

* For updates of recent blog posts, follow me on [Twitter](https://twitter.com/FelixLuginbuhl).
* For reproducing my data analysis, go on my [Github page](https://github.com/lgnbhl/blogposts).
* Curious about what I can do for your organisation? Have a look [my services](https://felixluginbuhl.com).
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

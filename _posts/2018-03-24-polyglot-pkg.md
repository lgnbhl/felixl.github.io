---
layout: single
title: My Package {polyglot} Is Now on CRAN
excerpt: Using the R console as an interactive environment to learn French, Spanish or anything you want.
permalink: /polyglot-pkg/
date: 2018-03-24 10:00:00 -0600  
author: lgnbhl
tags: rstats package
header:
  og_image: /images/screenshot_polyglot_1.png
---

A few months ago, I wrote my first package called {learner}. I finally took the time to put it on CRAN. Following the advice of a CRAN team member, I changed its name. I had to agree: it sounded like a machine learning package. So now the {learner} package is dead, long life to {[polyglot](https://CRAN.R-project.org/package=polyglot)}!

With {polyglot}, you can learn any two columns dataset right in the R console:

![](/images/screenshot_polyglot_1.png)

## How to use {polyglot}

Let's imagine you want to know if you can remember some basic Japanese expressions[^1]. You would search online a two columns table and save it into the {polyglot} directory.

[^1]: For my part, all I know in Japanese is "arigato", "konnichiwa" and "moshi moshi".

``` r
library(tidyverse)
library(rvest)
library(polyglot) # install.packages("polyglot")

url1 <- "https://www.omniglot.com/language/phrases/japanese.php" 

japanese <- url1 %>%
  read_html() %>%
  html_table() %>%
  .[[1]] %>%
  write_csv(path = paste0("", system.file("extdata/", package =
  "polyglot"), "Japanese_54_Basic_Expressions.csv"))

as_tibble(japanese)
```

    ## # A tibble: 54 x 2
    ##                     English
    ##                       <chr>
    ##  1                  Welcome
    ##  2 Hello (General greeting)
    ##  3         Hello (on phone)
    ##  4             How are you?
    ##  5  Reply to 'How are you?'
    ##  6         Long time no see
    ##  7        What's your name?
    ##  8           My name is ...
    ##  9      Where are you from?
    ## 10             I'm from ...
    ## # ... with 44 more rows, and 1 more variables: `日本語 (Japanese)` <chr>

Or maybe you would like to see if you know most of the capitals in the world.

``` r
url2 <- "https://en.wikipedia.org/wiki/List_of_national_capitals_in_alphabetical_order"

capitals <- url2 %>%
  read_html() %>%
  html_node(".wikitable") %>%
  html_table(fill = TRUE) %>%
  select(Country, City, Notes) %>%
  arrange(Country) %>%
  write_csv(path = paste0("", system.file("extdata/", package =
  "polyglot"), "List_Capital_Cities.csv"))

as_tibble(capitals)
```

    ## # A tibble: 243 x 3
    ##                  Country                City
    ##                    <chr>               <chr>
    ##  1              Abkhazia             Sukhumi
    ##  2           Afghanistan               Kabul
    ##  3 Akrotiri and Dhekelia Episkopi Cantonment
    ##  4               Albania              Tirana
    ##  5               Algeria             Algiers
    ##  6        American Samoa           Pago Pago
    ##  7               Andorra    Andorra la Vella
    ##  8                Angola              Luanda
    ##  9              Anguilla          The Valley
    ## 10   Antigua and Barbuda          St. John's
    ## # ... with 233 more rows, and 1 more variables: Notes <chr>

You would just have to run the `learn` function to learn some Japanese vocabulary.

``` r
library(polyglot)
learn() # launch the interactive learning environment
```

![](/images/screenshot_polyglot_2.png)

Before launching the interactive learning environment, be sure to have the [appropriate encoding](https://stat.ethz.ch/R-manual/R-devel/library/base/html/locales.html) and that your 2 or 3 columns dataset is a [CSV file](https://en.wikipedia.org/wiki/Comma-separated_values).

If your R console cannot read the Japanese characters, try the following:

``` r
# Make R reading Japanese
# Sys.setlocale("LC_ALL", "Japanese")   # for Windows
# Sys.setlocale("LC_ALL", "ja_JP")       # for macOS 
# Sys.setlocale("LC_ALL", "ja_JP.utf8")  # for Modern Linux etc. 
```

Happy learning!

## Post-scriptum

You can also try the [development version](https://github.com/lgnbhl/polyglot) of the package, which implements a simplified [spaced repetition](https://en.wikipedia.org/wiki/Spaced_repetition) learning algorithm.

![](/images/polyglot_dev.gif)

``` r
# install.packages("devtools")
devtools::install_github("lgnbhl/polyglot")
library(polyglot)
learn()
```

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

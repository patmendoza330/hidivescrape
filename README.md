3/12/2022

# Collecting Anime listings from various platforms

I currently have a tableau dashboard located
[here](https://public.tableau.com/app/profile/patrick.mendoza5877/viz/WhatAnimetoWatchNextMyAnimeList/Dashboard).
This dashboard is an exploratory view of top ranked and popular anime
titles based primarily on data from <http://myanimelist.net>. The
purpose of this dashboard is to assist the end-user in locating their
next anime to watch. However, what is lacking from this tool is the
streaming platform that currently houses the titles.

To add that information into the dashboard, I will need to either
locate, or scrape the anime titles that are housed on various platforms.

The most popular platforms for viewing anime titles are:

-   Crunchyroll
-   [Funimation](https://github.com/patmendoza330/funimationtitles)
-   Netflix
-   Hulu
-   Amazon Prime
-   HIDIVE

This list is not all-inclusive and in no particular order.

## First things first - setting some variables

Set your working directory to wherever you’d like in the
`WORKINDIRECTORY` section.

``` r
wd1 = WORKINGDIRECTORY
setwd(wd1)
```

## Install and load any libraries

The `tidyverse` contains the `rvest` library which will be used to
scrape the data, while `dplyr` and `tidyr` will be used to
manipulate/transform data.

``` r
install.packages(c("tidyverse","textutils"))
```

Next, we want to load three libraries:

``` r
library(tidyverse)
library(rvest)
library(textutils)
```

Although rvest is a component of the tidyverse, it doesn’t automatically
load with the library call `tidyverse`, as a result, you’ll need to load
it separately. The library `textutils` will allow you to decode html
special characters into their expected values.

Lets start with HIDIVE:

## Scraping HIDIVE Anime TV Listings

This is one of the large streaming platforms that I will be scraping as
an add-on to my anime suggestion tableau dashboard. Thankfully, HIDIVE
has a listing of all of their titles on a webpage at:
<https://www.hidive.com/tv>.

### Scraping the HIDIVE Title List

``` r
HIDIVE <- read_html('https://www.hidive.com/tv')

title_bucket <- HIDIVE %>%
    html_element(xpath = '/html/body/div[1]/div')
```

The code above isolates the bucket where I want to pull of my
information from.

When reviewing the HTML code on the page, it appears that the titles for
the anime can be found between `H2` headers, images can be found within
`default-img` divs, and date information can be found within `cell`
divs.

Lets grab that data:

``` r
anime_title <- title_bucket %>%
  html_elements('h2') %>%
  html_text(trim = TRUE)

anime_premiere <- title_bucket %>%
  html_elements('div.cell') %>%
  html_attr('data-premieredt')

anime_release <- title_bucket %>%
  html_elements('div.cell') %>%
  html_attr('data-releasedt')

anime_nextair <- title_bucket %>%
  html_elements('div.cell') %>%
  html_attr('data-nextairdate')

anime_img <-  title_bucket %>%
  html_elements('div.default-img') %>%
  html_elements('img') %>%
  html_attr('data-src')
```

Now, there are also badges on each of the tiles for the anime that
contain whether or not the anime is exclusive to HIDIVE or is dubbed. I
also want to grab that information.

However, there aren’t consistent `div` containers that will allow me to
align the badge attributes with the title. For that reason, I will
select all of the overall buckets for the titles that are housed within
`cell` divs, then run a `for` loop to see if there are badges within
those buckets.

``` r
anime_cell <- title_bucket %>%
  html_elements('div.cell')

anime_badge <- c()

for (item in anime_cell){
   rowVal <- item %>%
     html_elements('div.top-badge') %>%
     html_text(trim = TRUE)
   if (identical(rowVal, character(0))){
     rowVal <- NA
   }
   anime_badge <- append(anime_badge, rowVal)
}
```

Lets check the length of all the items and see if we can throw them
together

``` r
length(anime_title)
```

    ## [1] 436

``` r
length(anime_premiere)
```

    ## [1] 436

``` r
length(anime_release)
```

    ## [1] 436

``` r
length(anime_nextair)
```

    ## [1] 436

``` r
length(anime_img)
```

    ## [1] 436

``` r
length(anime_badge)
```

    ## [1] 436

They are all the same length, so I can combine and export them in the
next step

### Exporting

Lets make it a dataframe and export the table

``` r
hidive_data <- data.frame(anime_title = anime_title, anime_premiere = anime_premiere, anime_release = anime_release, anime_nextair = anime_nextair, anime_img = anime_img, anime_badge = anime_badge)
knitr::kable(head(hidive_data))
```

| anime\_title                                 | anime\_premiere       | anime\_release        | anime\_nextair | anime\_img                                                                           | anime\_badge |
|:---------------------------------------------|:----------------------|:----------------------|:---------------|:-------------------------------------------------------------------------------------|:-------------|
| 100 Sleeping Princes & the Kingdom of Dreams | 7/5/2018 12:00:00 AM  | 7/5/2018 12:00:00 AM  |                | //static.hidive.com/titles/OSP/OSP\_01\_MASTER\_300x169.jpg                          | Exclusive    |
| A Little Snow Fairy Sugar                    | 10/2/2001 12:00:00 AM | 10/2/2001 12:00:00 AM |                | //static.hidive.com/titles/LSF/LSF\_01\_MASTER\_300x169.jpg                          | Dubbed       |
| Action Heroine Cheer Fruits                  | 7/6/2017 12:00:00 AM  | 7/6/2017 5:00:00 PM   |                | //static.hidive.com/titles/ACF/action-heroine-cheer-fruits\_ACF\_MASTER\_300x169.jpg | NA           |
| After the Rain                               | 1/12/2018 12:00:00 AM | 3/31/2021 1:00:00 PM  |                | //static.hidive.com/titles/ATR/after-the-rain\_ATR\_01\_MASTER\_300x169.jpg          | Dubbed       |
| Ahiru no Sora                                | 10/2/2019 12:00:00 AM | 9/30/2019 5:00:00 PM  |                | //static.hidive.com/titles/ANS/ahiru-no-sora\_ANS\_01\_MASTER\_300x169\_01.jpg       | Dubbed       |
| Akame ga Kill!                               | 7/6/2014 12:00:00 AM  | 7/6/2014 12:00:00 PM  |                | //static.hidive.com/titles/AGK/AGK\_MASTER\_300x169.jpg                              | Dubbed       |

``` r
write.csv(hidive_data, 'hidivetitles.csv', row.names = FALSE) 
```

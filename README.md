
<!-- README.md is generated from README.Rmd. Please edit that file -->
rtweet <img src="man/figures/logo.png" align="right" />
=======================================================

[![Build Status](https://travis-ci.org/mkearney/rtweet.svg?branch=master)](https://travis-ci.org/mkearney/rtweet) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/rtweet)](https://CRAN.R-project.org/package=rtweet) ![Downloads](https://cranlogs.r-pkg.org/badges/rtweet) ![Downloads](https://cranlogs.r-pkg.org/badges/grand-total/rtweet) <!-- [![Travis-CI Build Status](https://travis-ci.org/mkearney/rtweet.svg?branch=master)](https://travis-ci.org/mkearney/rtweet) --> <!-- [![Coverage Status](https://img.shields.io/codecov/c/github/mkearney/rtweet/master.svg)](https://codecov.io/gh/mkearney/rtweet) --> [![Rdoc](http://www.rdocumentation.org/badges/version/rtweet)](http://www.rdocumentation.org/packages/rtweet)

R client for accessing Twitter's REST and stream APIs.

Check out the [rtweet package documentation website](http://rtweet.info).

Install
-------

To get the current released version from CRAN:

``` r
## install rtweet from CRAN
install.packages("rtweet")

## load rtweet package
library(rtweet)
```

To get the current development version from Github:

``` r
## install devtools package if it's not already
if (!requireNamespace("devtools", quietly = TRUE)) {
  install.packages("devtools")
}

## install dev version of rtweet from github
devtools::install_github("mkearney/rtweet")

## load rtweet package
library(rtweet)
```

Getting started
---------------

#### (NEW) **Get up and running with rtweet in no time**

-   NO MORE CREATING TWITTER APPS
-   NO MORE DEALING WITH API/CONSUMER/SECRET/ACCESS KEYS
-   NO MORE COMPLICATED INSTRUCTIONS

All you need is a **Twitter account** + **rtweet** and you're up and running!

#### Easy API authorization

-   The first time you make an API request function---e.g., `search_tweets()`, `stream_tweets()`, `get_followers()`---a browser window will open.
-   Log in to your Twitter account.
-   Agree/authorize the rtweet application.

And that's it! You're ready to start collecting and analyzing Twitter data!

Package features
----------------

### Search tweets

Search for 5,000 (non-retweeted) tweets containing the rstats hashtag.

``` r
## search for 5000 tweets using the rstats hashtag
rt <- search_tweets(
  "#rstats", n = 5000, include_rts = FALSE
)
> rt
# A tibble: 2,478 x 42
            status_id          created_at            user_id screen_name
                <chr>              <dttm>              <chr>       <chr>
 1 928811894000861185 2017-11-10 02:29:46           29916355    rensa_co
 2 928808649836912641 2017-11-10 02:16:53         4246465853     bfgray3
 3 928808308647096320 2017-11-10 02:15:31           10162032 lenteaberta
 4 928805640016990209 2017-11-10 02:04:55          144592995   Rbloggers
 5 928805625009721344 2017-11-10 02:04:52 924760121913339904     ezmanip
 6 928804415896805376 2017-11-10 02:00:03           21930409   gdbassett
 7 928801913612759040 2017-11-10 01:50:07          144592995   Rbloggers
 8 928791611693195264 2017-11-10 01:09:10          517921400 LeahAWasser
 9 928790088238034944 2017-11-10 01:03:07         3288057307   lucyleeow
10 928789068623765504 2017-11-10 00:59:04         3288057307   lucyleeow
# ... with 2,468 more rows, and 38 more variables: text <chr>, source <chr>,
#   reply_to_status_id <chr>, reply_to_user_id <chr>,
#   reply_to_screen_name <chr>, is_quote <lgl>, is_retweet <lgl>,
#   favorite_count <int>, retweet_count <int>, hashtags <list>, symbols <list>,
#   urls_url <list>, urls_t.co <list>, urls_expanded_url <list>,
#   media_url <list>, media_t.co <list>, media_expanded_url <list>,
#   media_type <list>, ext_media_url <list>, ext_media_t.co <list>,
#   ext_media_expanded_url <list>, ext_media_type <lgl>,
#   mentions_user_id <list>, mentions_screen_name <list>, lang <chr>,
#   quoted_status_id <chr>, quoted_text <chr>, retweet_status_id <chr>,
#   retweet_text <chr>, place_url <chr>, place_name <chr>,
#   place_full_name <chr>, place_type <chr>, country <chr>, country_code <chr>,
#   geo_coords <list>, coords_coords <list>, bbox_coords <list>
```

Quickly visualize frequency of tweets over time using `ts_plot()`.

``` r
## plot time series of tweets
ts_plot(rt, "hours") +
  ggplot2::theme_minimal() +
  ggplot2::theme(plot.title = ggplot2::element_text(face = "bold")) +
  ggplot2::labs(
    x = NULL, y = NULL,
    title = "Twitter statuses containing \"rstats\" hashtag over time",
    subtitle = "Tweets aggregated and counted in 1-hour intervals",
    caption = "\nSource: Data collected from Twitter's REST API via rtweet"
  )
```

![](example-rstatsts.png)

Twitter rate limits cap the number of search results returned to 18,000 every 15 minutes. To request more than that, simply set `retryonratelimit = TRUE` and rtweet will wait for rate limit resets for you.

``` r
## search for 250,000 tweets containing the word "data"
rt <- search_tweets(
  "data", n = 250000, retryonratelimit = TRUE
)
```

Search by geo-location---for example, find 10,000 tweets in the English language sent from the United States.

``` r
## search for 10,000 tweets sent from the US
rt <- search_tweets(
  "lang:en", geocode = lookup_coords("usa"), n = 10000
)

## create lat/lng variables using all available tweet and profile geo-location data
rt <- lat_lng(rt)

## plot state boundaries
par(mar = c(0, 0, 0, 0))
maps::map("state", lwd = .25)

## plot lat and lng points onto state map
with(rt, points(lng, lat, pch = 20, cex = .75, col = rgb(0, .3, .7, .75)))
```

![](example-statemap.png)

### Stream tweets

Randomly sample (approximately 1%) from the live stream of all tweets.

``` r
## random sample for 30 seconds (default)
rt <- stream_tweets("")
```

Stream all geo enabled tweets from London for 60 seconds.

``` r
## stream tweets from london for 60 seconds
rt <- stream_tweets(lookup_coords("london, uk"), timeout = 60)
```

Stream all tweets mentioning @realDonaldTrump or Trump for a week.

``` r
## stream london tweets for a week (60 secs x 60 mins * 24 hours *  7 days)
stream_tweets(
  "realdonaldtrump,trump",
  timeout = 60 * 60 * 24 * 7,
  file_name = "tweetsabouttrump.json",
  parse = FALSE
)

## read in the data as a tidy tbl data frame
djt <- parse_stream("tweetsabouttrump.json")
```

### Get friends

Retrieve a list of all the accounts a **user follows**.

``` r
## get user IDs of accounts followed by CNN
cnn_fds <- get_friends("cnn")
> cnn_fds
# A tibble: 1,115 x 2
    user    user_id
   <chr>      <chr>
 1   cnn   14085040
 2   cnn   58524428
 3   cnn 1630896181
 4   cnn   52920970
 5   cnn   43197255
 6   cnn  466788677
 7   cnn 3282859598
 8   cnn  247832362
 9   cnn  326255267
10   cnn  274714964
# ... with 1,105 more rows

## lookup data on those accounts
cnn_fds_data <- lookup_users(cnn_fds$user_id)
```

### Get followers

Retrieve a list of the **accounts following** a user.

``` r
## get user IDs of accounts following CNN
cnn_flw <- get_followers("cnn", n = 75000)

## lookup data on those accounts
cnn_flw_data <- lookup_users(cnn_flw$user_id)
```

Or if you really want ALL of their followers:

``` r
## how many total follows does cnn have?
cnn <- lookup_users("cnn")
cnn$followers_count
> [1] 38349142

## get them all (this would take a little over 5 days)
cnn_flw <- get_followers("cnn", n = 38349142, retryonratelimit = TRUE)
```

### Get timelines

Get the most recent 3,200 tweets from cnn, BBCWorld, and foxnews

``` r
## get user IDs of accounts followed by CNN
tmls <- get_timelines(c("cnn", "BBCWorld", "foxnews"), n = 3200)

## plot the frequency of tweets for each user over time
tmls %>%
  dplyr::filter(created_at > "2017-10-23") %>%
  dplyr::group_by(screen_name) %>%
  ts_plot("days", trim = 1L) +
  ggplot2::theme_minimal() +
  ggplot2::ggsave("example-tmls.png", width = 8, height = 6, units = "in")
```

![](example-tmls.png)

### Get favorites

Get the 3,000 most recently favorited statuses by JK Rowling.

``` r
jkr <- get_favorites("jk_rowling", n = 3000)
```

### Search users

Search for 1,000 users with the rstats hashtag in their profile bios.

``` r
## search for users with #rstats in their profiles
usrs <- search_users("#rstats", n = 1000)
> usrs
# A tibble: 655 x 20
              user_id              name    screen_name
                <chr>             <chr>          <chr>
 1          342250615          rOpenSci       rOpenSci
 2           12640312       Daniel Pett        DEJPett
 3         2170413740   Baseball with R BaseballRstats
 4          295344317   One R Tip a Day       RLangTip
 5         3075907260 Positive Residual      presidual
 6           46245868    David Robinson           drob
 7            5685812         boB Rudis       hrbrmstr
 8 824037040996098049   LIRR Statistics      LIRRstats
 9         3230388598      Mara Averick      dataandme
10            4515151     Pablo Barberá      p_barbera
# ... with 645 more rows, and 17 more variables: location <chr>,
#   description <chr>, url <chr>, protected <lgl>, followers_count <int>,
#   friends_count <int>, listed_count <int>, statuses_count <int>,
#   favourites_count <int>, account_created_at <dttm>, verified <lgl>,
#   profile_url <chr>, profile_expanded_url <chr>, account_lang <chr>,
#   profile_banner_url <chr>, profile_background_url <chr>,
#   profile_image_url <chr>
```

### Get trends

Discover what's currently trending in San Francisco.

``` r
sf <- get_trends("san francisco")
> sf
# A tibble: 50 x 9
                  trend                                                url
 *                <chr>                                              <chr>
 1             #SEAvsAZ             http://twitter.com/search?q=%23SEAvsAZ
 2            Roy Moore        http://twitter.com/search?q=%22Roy+Moore%22
 3            Star Wars        http://twitter.com/search?q=%22Star+Wars%22
 4             Louis CK         http://twitter.com/search?q=%22Louis+CK%22
 5 #BandTogetherBayArea http://twitter.com/search?q=%23BandTogetherBayArea
 6         #DreamActNow         http://twitter.com/search?q=%23DreamActNow
 7                #DF17                http://twitter.com/search?q=%23DF17
 8            #DealBook            http://twitter.com/search?q=%23DealBook
 9           Jeff Green       http://twitter.com/search?q=%22Jeff+Green%22
10       John Hillerman   http://twitter.com/search?q=%22John+Hillerman%22
# ... with 40 more rows, and 7 more variables: promoted_content <lgl>,
#   query <chr>, tweet_volume <int>, place <chr>, woeid <int>, as_of <dttm>,
#   created_at <dttm>
```

For posting (tweeting from R console) or read DM permissions
------------------------------------------------------------

-   If you'd like to post Twitter statuses, follow or unfollow accounts, and/or read your direct messages, you'll need to create your own Twitter app.
-   To create your own Twitter app, follow the instructions in the authorization vignette on [obtaining and using access tokens](http://rtweet.info/articles/auth.html).

``` r
## authorizing API access
vignette("auth", package = "rtweet")
```

Other vignettes
---------------

[Quick overview of rtweet package](http://rtweet.info/articles/intro.html)

``` r
## quick overview of rtweet functions
vignette("intro", package = "rtweet")
```

[Live streaming tweets data](http://rtweet.info/articles/stream.html)

``` r
## working with the stream
vignette("stream", package = "rtweet")
```

Contact
-------

Communicating with Twitter's APIs relies on an internet connection, which can sometimes be inconsistent. With that said, if you encounter an obvious bug for which there is not already an active [issue](https://github.com/mkearney/rtweet/issues), please [create a new issue](https://github.com/mkearney/rtweet/issues/new) with all code used (preferably a reproducible example) on Github.

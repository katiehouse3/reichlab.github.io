Intro to using the delphi-epidata API to access infectious disease data
================
Nick Reich
2016-09-01

This week I attended a workshop at the CDC about last year's [FluSight challenge](https://predict.phiresearchlab.org/flu/index.html), a competition that scores weekly real-time predictions about the course of the influenza season. They are planning another round this year and are hoping to increase the number of teams particiating. Stay tuned to [this site](https://predict.phiresearchlab.org/flu/index.html) for more info.

At the workshop, I learned about [DELPHI's](http://delphi.midas.cs.cmu.edu/) real-time epidemiological [data API](https://github.com/undefx/delphi-epidata). The API is linked to various data sources on influenza and dengue, including US CDC flu data, Google Flu Trends, and Wikipedia data. There is [some documentation](https://github.com/undefx/delphi-epidata#the-api) and [minimal examples](https://github.com/undefx/delphi-epidata#code-samples), and this post documents a more robust and complete example for using the API via R. I'll note that the CDC's influenza data, can also be accessed via the `cdcfluview` R package, which I'm not going to discuss here and I will focus here on accessing some of the other data sources.

Setup
-----

Let's start by loading the R script containing the relevant methods needed to access the API.

``` r
source("https://raw.githubusercontent.com/undefx/delphi-epidata/master/code/delphi_epidata.R")
```

Then, load in two packages that we will use to tidy and plot the data.

``` r
library(MMWRweek)
library(ggplot2)
```

Dengue data from Taiwan CDC
---------------------------

Here is some code that pulls data from [Taiwan's NIDSS](http://nidss.cdc.gov.tw/en/), specifically asking for nationwide data, and data from the central region. A complete list of [regions](https://github.com/undefx/delphi-epidata/blob/master/labels/nidss_regions.txt) and [locations](https://github.com/undefx/delphi-epidata/blob/master/labels/nidss_locations.txt) are available. Also, I've specified a range of weeks from the first week of 2010 (`201001`) to the last week of 2016 (`201653`).

``` r
res <- Epidata$nidss.dengue(locations = list('nationwide', 'central'), 
                            epiweeks = list(Epidata$range(201001, 201653)))
```

The above command should pull data down into your current session, but it will be a little bit 'list-y', so here is some code I wrote to clean it up and make it a bit more of a workable dataset in R.

``` r
df <- data.frame(matrix(unlist(res$epidata), nrow=length(res$epidata), byrow=T))
colnames(df) <- names(res$epidata[[1]])[!is.null(res$epidata[[1]])]
df$count <- as.numeric(as.character(df$count))
df$year <- as.numeric(substr(df$epiweek, 0, 4))
df$week <- as.numeric(substr(df$epiweek, 5, 6))
df$date <- MMWRweek2Date(MMWRyear = df$year, MMWRweek = df$week)
```

Note the use of the `MMWRweek2Date()` function that gives us a date column in our data frame.

And here is a plot of the resulting data. Pretty noisy, but interesting to look at nonetheless.

``` r
ggplot(df, aes(x=date, y=count, color=location)) + geom_point()
```

![](epidata-api-sandbox_files/figure-markdown_github/plot-1.png)

Wikipedia data
--------------

Let's try loading some of the Wikipedia data on influenza and other related terms. The article. I think this reflects the number of hits on pages of certain articles, although I'm not sure.

``` r
res <- Epidata$wiki(articles=list("influenza", "common_cold", "cough"),
                    epiweeks=list(Epidata$range(201101, 201553)))
df <- data.frame(matrix(unlist(res$epidata), nrow=length(res$epidata), byrow=T))
colnames(df) <- names(res$epidata[[1]])[!is.null(res$epidata[[1]])]
df$count <- as.numeric(as.character(df$count))
df$year <- as.numeric(substr(df$epiweek, 0, 4))
df$week <- as.numeric(substr(df$epiweek, 5, 6))
df$date <- MMWRweek2Date(MMWRyear = df$year, MMWRweek = df$week)
ggplot(df, aes(x=date, y=count, color=article)) + 
  geom_point() + 
  geom_smooth(span=.1, se=FALSE)
```

![](epidata-api-sandbox_files/figure-markdown_github/unnamed-chunk-3-1.png)

Happy data exploring!

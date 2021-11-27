---
title: Exploring Montreal's Urban Water Demand
author: ''
date: '2021-11-26'
slug: hi-hugo
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-11-26T12:47:20-05:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In this post, we will explore Montreal's urban water demand (UWD) between the end of February, 1999 and the end of August, 2010. In total, there are 4179 observations/records in our data set. The time scale of the data set is daily. Our data is stored as a `tsibble` called `montreal`.








```r
autoplot(montreal) +
  ggtitle("Urban water demand for Montreal, Quebec, Canada")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" width="672" />

Montreal's urban water demand exhibits strong seasonality on the monthly scale. Regardless of year, average monthly UWD rises and falls in a predictable fashion. This may be observed in the following 2 plots.


```r
montreal %>%
  gg_season(UWD, labels = "none", period = "year") +
  # theme(legend.position = "none") +
  labs(y = "UWD (Mega-Litres)",
       title = "Seasonal Plot: Montreal Urban Water Demand")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-5-1.png" width="672" />


```r
montreal %>% 
  mutate(Month = yearmonth(Date)) %>% 
  index_by(Month) %>% 
  summarise(UWD_month = mean(UWD)) %>% 
  gg_subseries(UWD_month, period = "year") +
  labs(
    y = "UWD (Mega-Litres)",
    title = "Montreal Urban Water Demand"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-6-1.png" width="672" />

In addition to seasonal effects, there seems to be a slight decrease in Montreal's water demand on weekends. The following plot shows this.


```r
montreal %>% gg_subseries(UWD, period = "week") +
  labs(
    y = "UWD (Mega-Litres)",
    title = "Montreal Urban Water Demand"
  )
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

Let's take a look at correlations between input features and UWD. Our data set `montreal` has 3 input features: `Tmax`, `R` and `API95`. `Tmax` is maximum daily air temperature, `R` is total daily rainfall, and `API95` is the daily antecedent precipitation index. [Click here](https://tonyladson.wordpress.com/tag/antecedent-precipitation-index/) to learn more about this feature.


```r
montreal %>%
  GGally::ggpairs(columns = 2:5, lower = list(continuous = GGally::wrap('points', alpha = 0.2)))
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

it is evident that maximum daily temperature has a significant positive correlation with UWD. Let's zoom in on the scatter plot between maximum daily temperature and UWD:


```r
montreal %>%
  ggplot(aes(x = Tmax, y = UWD)) +
  geom_point(alpha = 0.3) +
  labs(x = "Maximum Temperature (degrees Celsius)",
       y = "Urban Water Demand (ML)",
       title = "Scatterplot of Maximum Daily Temperature and UWD")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-9-1.png" width="672" />

it appears that there is a nonlinear relationship between maximum daily temperature and UWD.

We can also look at auto-correlations by plotting UWD against different lagged versions of itself:


```r
montreal %>%
  gg_lag(UWD, geom = "point", lags = 1) +
  labs(x = "lag(UWD, 1)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

the plot above shows that if yesterday's UWD was high, then today's UWD will probably be high too. We can see powerful correlations for longer lags as well:


```r
montreal %>%
  gg_lag(UWD, geom = "point") +
  labs(x = "lag(UWD, k)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-11-1.png" width="672" />

We even see autocorrelation at the monthly scale:


```r
montreal %>%
  mutate(Month = yearmonth(Date)) %>%
  index_by(Month) %>%
  summarise(UWD = mean(UWD)) %>%
  gg_lag(UWD, geom = "point") +
  labs(x = "lag(UWD, k)")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-12-1.png" width="672" />

Note that ACF plots give the same information as the lagged scatter plots above. For instance, at the daily scale, the ACF plot for UWD looks as follows:


```r
montreal %>%
  ACF(UWD, lag_max = 48) %>%
  autoplot() +
  labs(title="Montreal UWD ACF Plot")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-13-1.png" width="672" />

Lastly, it appears that Montreal's UWD has been trending down over the years, which could be caused (in part) by newer toilets, appliances and other water consuming items replacing older, more water intensive products. This applies to both residential and industrial settings.


```r
montreal %>% 
  mutate(Month = yearmonth(Date)) %>% 
  index_by(Month) %>% 
  summarise(UWD_month = mean(UWD)) %>% 
  model(
    STL(UWD_month ~ trend() + season(), robust = TRUE)
    ) %>%
  components() %>%
  autoplot()
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-14-1.png" width="672" />


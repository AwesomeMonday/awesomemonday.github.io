---
layout: post
title: "Are We Reading More Books Nowadays"
subtitle: "TidyTuesday Week 19 NY Times Best-sellers analysis using R"
date: 2022-05-11 02:45:13 +0500
background: '/img/posts/NYT-bestsells/nytbest.jpg'
---

Hello, this is Hao.

For this week's [TidyTuesday](https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-05-10){:target="_blank"} dataset, we are looking at the New York Times Best-sellers list for the past 80 years. 
I want to see if people are reading more books nowadays.

load in libraries
```r

library(tidyverse)
library(scales)
library(ggpubr)
library(showtext)
library(ggthemes)
library(viridis)

```

load in data
```r
rm(list = ls())
nyt_titles <- readr::read_tsv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-05-10/nyt_titles.tsv')
nyt_full <- readr::read_tsv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-05-10/nyt_full.tsv')

```
load in fonts

```r
font_add_google(name = "Macondo", family = "Macondo")
font_add_google(name = "Montserrat", family = "Montserrat")
showtext_auto()
```
I will only use the titles data set for my analysis
```r

nyt_titles %>% 
  View()

nyt_titles %>% 
  count(year, sort = TRUE)

# number of books made to the chart over the decades

nyt_decades <- nyt_titles %>% 
  filter(year != 2020) %>% 
  mutate(decades = (year %/% 10) * 10) %>% 
  group_by(decades) %>% 
  summarise(total_books = n())

# plot
nyt_decades %>% 
  ggplot(aes(decades, total_books, fill = total_books)) +
  geom_col(show.legend = F) +
  coord_polar(start = 0)+
  theme_minimal() +
  scale_x_continuous(breaks = seq(1930, 2010, 10))+
  labs(
    title = "Number of NYT Best-selling Books by Decades",
    caption = "TidyTuesday | wk19 NY Times Best-seller List \nVisualization by Hao Li"
  ) +
  theme(
    plot.title = element_text(size = 20, face = "bold", family = "Macondo", hjust = .5, vjust = -4),
    plot.caption = element_text(family = "Montserrat", color = "black", size = 10, vjust = 29),
    axis.title = element_blank(),
    axis.text.x = element_text(size = 15),
    axis.text.y = element_blank(),
    panel.grid.major  = element_blank(),
    panel.background = element_rect(fill = "lightblue", color = "lightblue")
  ) +
  scale_fill_viridis(discrete = FALSE)
  
  ```

![nyt-bestsellers](/img/posts/NYT-bestsells/NYT-bestseller.png)

Analysis: as we can see from the plot above, starting from the 1990 decade to 2010 decade, the number of titles on the best-sellers chart are more than doubled than the previous decades. 
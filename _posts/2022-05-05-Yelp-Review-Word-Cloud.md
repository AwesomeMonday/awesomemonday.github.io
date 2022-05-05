---
layout: post
title: "What do people say about a restaurant they like. (part 2. visualization)"
subtitle: "Creating a WrodCloud with R"
date: 2022-05-05 10:45:13 +0500
background: '/img/posts/wordcloud/good-review.jpg'
---
Hello, this is Hao.

Word cloud doesn't provide much of statistical meaning itself. But it is great for text comparisons and it's very fun to look at. So today, I will make a wordcloud about what the customers say about one of the top rated Asian restaurants in the New York city area.

This is for exploratory purposes only, the reason I picked this restaurant is simply because the amount of reviews is large enough to be statistical significant, but not too large to be picked up by anti scraping bot from the website, in order to block my IP address.

This is the second part of the web scraping project I am working on, for details about the first part on how to scrape website, please click [here](https://awesomemonday.github.io/2022/05/03/NYC-Asian-Restaurant.html){:target = "_blank"}.

I will use the these 2 R packages for this, if you don't have them installed, you can run the following code.
```r
install.packages("wordcloud2")
install.packages("tm")
```
load in libraries
```r
library(tidyverse)
library(wordcloud2)
library(tm)
library(rvest)
library(lubridate)
```


There are a total of 48 pages of reviews, I will also need the date of the review, the rating, and the reviewer's states for future analysis.

```r
rm(list = ls())

date_all <-  c()
rating_all <- c()
user_location_all <- c()
review_text_all <- c()

for (page_num in seq(from = 0, to = (48*10)-10, by = 10)){
  url <- paste0("https://www.yelp.com/biz/blue-willow-%E5%A4%9C%E6%9D%A5%E6%B9%98-new-york-2?start=", page_num)
  
  page <- read_html(url)

  date <- page %>% 
    html_nodes(".margin-b1-5__09f24__NHcQi .css-chan6m") %>% 
    html_text2()

  user_location <- page %>% 
    html_nodes(".arrange-unit-fill__09f24__CUubG .border-color--default__09f24__NPAKY .border-color--default__09f24__NPAKY .border-color--default__09f24__NPAKY .border-color--default__09f24__NPAKY .css-qgunke") %>% 
    html_text2()
  
  rating <- page %>% 
    html_elements(xpath = "//div[contains(@aria-label, 'rating')]") %>% 
    html_attr("aria-label") %>% 
    str_remove_all(" star rating") %>% 
    as.numeric() %>% 
    tail(-1)
  
  review_text <- page %>% 
    html_nodes(".css-qgunke .raw__09f24__T4Ezm") %>% 
    html_text2()
  
  date_all <- append(date_all, date)
  rating_all <- append(rating_all, rating)
  user_location_all <- append(user_location_all, user_location)
  review_text_all <- append(review_text_all, review_text)
  
  print(paste("page", (page_num+10)/10))
  print("taking a break")
  
  Sys.sleep(5)
  
}

str(date_all)
str(review_text_all)
str(user_location_all)
str(rating_all)

reviews <- tibble("date" = head(date_all, 453),
                  "rating" = head(rating_all, 453),
                  "user_location" = head(user_location_all, 453),
                  "review_text" = head(review_text_all, 453))

View(reviews)

# change "date" to proper date format
reviews <- reviews %>% 
  mutate(date = mdy(date)) 

# remove the city info from "user_location", only keep state
reviews <- reviews %>% 
  mutate(user_location = str_sub(user_location, -2)) %>% 
  mutate(user_location = toupper(user_location))


# output the csv
write_csv(reviews, file = "C://Users//haoli//Desktop//restaurant_review.csv")

```

Let's create the wordcloud!
```r
review_corpus <- Corpus(VectorSource(reviews$review_text))

review_corpus <- review_corpus %>% 
  tm_map(removeNumbers) %>% 
  tm_map(removePunctuation) %>% 
  tm_map(stripWhitespace) %>% 
  tm_map(content_transformer(tolower)) %>% 
  tm_map(removeWords, stopwords("english")) %>% 
  tm_map(removeWords, stopwords("SMART")) %>% 
  tm_map(removeWords, c("blue", "willow", "good", "great", "restaurant", "dish", "chinese", "place", "ordered"))

tdm <- TermDocumentMatrix(review_corpus) %>% 
  as.matrix()

words <- sort(rowSums(tdm), decreasing = TRUE)

df <- data.frame(word = names(words), freq = words)

wordcloud2(df)
```

Here it is, now we can see what people say the most about this restaurant that they like!

![word cloud](/img/posts/wordcloud/wordcloud.png)

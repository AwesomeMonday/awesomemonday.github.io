---
layout: post
title: "What are the 5 most popular categories of a social media company"
subtitle: "A Accenture Data Analytics Project."
date: 2022-04-25 23:45:13 +0500
background: '/img/posts/Accenture-Social/social-background.png'
---

Hello, this is Hao. 

As I was looking for more projects to practice, I came across with this virtual experience programe on [theforage.com](https://www.theforage.com/){:target="_blank"}. The project is hosted by Accenture North America, and it's called Navigating Numbers (full details can be found [here](https://www.theforage.com/virtual-internships/prototype/hzmoNKtzvAzXsEqx8/Data-Analytics-Virtual-Experience?ref=9W3J5sbu3KKhpJDGr){:target="_blank"}). The goal for this project is to be "equipped with data fundamentals and an understanding of what a career in data analytics could look like".

The project is devided into 4 modules: project understanding, data cleaning and modeling, data visualization and storytelling, present to the client. Immediately, the flow from understand the business tasks, clean and process the data, to create visualization to gain business insights, all sounded very similar to the Google Data Analytics courses, but with one big different, in order to complete the course and receive the certificate, a video presentation must be recorded and submitted. 

Let's get started.




load in the libraries

```{r}
library(tidyverse)
library(Hmisc)
library(lubridate)
library(janitor)
```

load in the data sets

```{r}
rm(list = ls())
content <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/Content%20(1).csv")

location <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/Location%20(1).csv")

profile <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/Profile%20(1).csv")

reaction <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/Reactions%20(1).csv")

reaction_types <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/ReactionTypes%20(1).csv")

session <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/Session%20(1).csv")

user <- read_csv("https://cdn.theforage.com/vinternships/companyassets/T6kdcdKSTfg2aotxT/User%20(1).csv")
```

1. content data set
log: removed unnecessary columns, and clean col names; cleaned category names;
this data set will join with other sets on content_id and user_id

```{r}
# use describe function to see if there are any missing values
describe(content)

content <- content %>% 
  select(-"URL", -...1) %>% 
  clean_names()

glimpse(content)


# some category has quotes on them and some don't, remove marks and change to lower case to keep consistency

content <- content %>% 
  mutate(category = gsub('[\"]', '', category)) %>% 
  mutate(category = tolower(category)) %>% 
  mutate(category = fct_recode(category, "public_speaking" = "public speaking"),
         category = fct_recode(category, "healthy_eating" = "healthy eating"))

# rename some of the column names to prepare for joining datasets
content <- content %>% 
  rename(content_type = type)

# initial exploration of the 'content' dataset
content %>% 
  count(category, sort = TRUE) %>% 
  View()

# top 5 categories by content category
content %>% 
  mutate(category = fct_lump(category, 5)) %>% 
  group_by(category) %>% 
  summarise(total = n()) %>% 
  arrange(desc(total))

# distribution of categories
content %>% 
  group_by(category) %>% 
  summarise(n = n()) %>% 
  mutate(category = fct_reorder(category, n),) %>% 
  ggplot(aes(category, n, fill = category)) +
  geom_col(show.legend = FALSE) +
  coord_flip() +
  labs(title = "rank on category by the number of posts", y = 'number of posts', x = '')


# distribution of content_type 
content %>% 
  group_by(content_type) %>% 
  summarise(n = n()) %>% 
  mutate(content_type = fct_reorder(content_type, n)) %>% 
  ggplot(aes(content_type, n, fill = content_type)) +
  geom_col(show.legend = FALSE) +
  coord_flip()+
  labs(title = 'rank on content_type by the number of posts', y = 'number of posts', x = '')
```

2.location data set
log: removed unnecessary columns and cleaned col names; extracted zip code and state name from address then dropped address column
this set will join with user data set on user_id to view the state distribution on map

```{r}
describe(location)

location <- location %>% 
  select(-...1) %>% 
  clean_names()

#extract state and zip code from address then drop address column
location <- location %>% 
  mutate(zip_code = str_sub(address, -5),
         state = str_sub(address, -8, -7)) %>% 
  select(-address)

# initial exploration of the 2 new columns
location %>% 
  count(state, sort = TRUE) %>% 
  View()
location %>% 
  count(zip_code, sort = TRUE)
glimpse(location)
```

3. profile
log: separated interests into individual rows, removed marks and corrected spelling error, and dropped age column
this set will NOT be used to join with other datasets for the final csv

```{r}
describe(profile)

profile <- profile %>% 
  select(-'...1', -'Age')

profile <- profile %>% 
  separate_rows(Interests, sep = ',') %>% 
  mutate(Interests = str_replace(Interests, "\\'", "")) %>%   
  mutate(Interests = str_replace(Interests, "\\[", "")) %>%   
  mutate(Interests = str_replace(Interests, "\\]", "")) %>%   
  mutate(Interests = str_replace(Interests, "\\'", "")) %>%   
  mutate(Interests = str_replace(Interests, " ", "")) %>% 
  clean_names() 

# recode some of the misspelled names
profile <- profile %>% 
  mutate(interests = fct_recode(interests, "public_speaking" = "public speaking"),
         interests = fct_recode(interests, "public_speaking" = "publicspeaking"),
         interests = fct_recode(interests, "healthy_eating" = "healthy eating"),
         interests = fct_recode(interests, "healthy_eating" = "healthyeating")) 

profile %>% 
  count(interests, sort = TRUE)
```

4. reaction
log: removed unnecessary columns and cleaned col names; extracted time info for future analysis; 
dropped the user_id column since it refers the user that interacted with the content, not the user who created it; 
this set will join with reaction_types on type to get the scores of each content thru group by

```{r}
describe(reaction) 

# looking into datetime column, extracting the month, day, weekday and hour info for initial exploration in R
reaction <- reaction %>% 
  mutate(Month = month(Datetime, label = TRUE),
         Day = day(Datetime),
         Weekday = wday(Datetime, label = TRUE),
         Hour = hour(Datetime)) %>% 
  clean_names() %>%
  select(-(x1), -(user_id)) 
  
reaction <- reaction %>% 
  rename(reaction_type = type) 

# reaction type distribution
reaction %>% 
  filter(!is.na(reaction_type)) %>% 
  group_by(reaction_type) %>% 
  summarise(total = n()) %>% 
  mutate(reaction_type = fct_reorder(reaction_type, total)) %>% 
  ggplot(aes(total, reaction_type, fill = reaction_type))+
  geom_col(show.legend = F) +
  labs(title = 'reaction type distribution', x = 'number of reactions', y = '')
 
# initial exploration on the time variables
reaction %>% 
  group_by(month) %>% 
  summarise(total = n()) %>% 
  ggplot(aes(month, total, group = 1))+
  geom_line()+
  expand_limits(y = 0)+
  labs(title = 'reactoin count by month', x = '', y = '')

reaction %>% 
  group_by(weekday) %>% 
  summarise(total = n()) %>% 
  ggplot(aes(weekday, total, group = 1))+
  geom_line()+
  expand_limits(y = 0)+
  labs(title = 'reaction count by weekday', x = '', y = '')

reaction %>% 
  group_by(hour) %>% 
  summarise(total = n()) %>% 
  ggplot(aes(hour, total, group = 1))+
  geom_line()+
  expand_limits(y = 0)+
  labs(title = 'reaction count by hour', x = '', y = '')
```

5. reaction_types
log: removed unnecessary columns and cleaned col names; 
this set will join reaction data set and use the score to rank the top 5 categories that is in the task

```{r}
reaction_types <- reaction_types %>% 
  select(-"...1")

reaction_types <- reaction_types %>% 
  clean_names() %>%
  rename(reaction_type = type)

# what kind of reaction_types have high scores
reaction_types %>% 
  arrange(desc(score))

# sentiment distribution
reaction_types %>%
  count(sentiment, sort = TRUE)
```

6. session
log: removed unnecessary columns and cleaned names; 
this data set can provide us some interesting insights on device usage, but it's not ralevant to the analysis this time

```{r}
session <- session %>% 
  select(-...1) %>% 
  clean_names()
```

7. user data set
log: removed unnecessary columns and cleaned column names
we don't need the user info for our business task

```{r}
user <- user %>% 
  select(-...1) %>% 
  clean_names()
```

since the business task is to find out the top 5 most popular categories, it should be based on the content reactions, but the count of how many posts
of content in each category. I will use the score to calculate the ranks. 

```{r}
# first let's join the tables

buzz_full <- location %>% 
  right_join(content, by = 'user_id') %>% 
  left_join(reaction, by = 'content_id') %>% 
  left_join(reaction_types, by = 'reaction_type')

# export the dataset for future analysis
# write.csv(buzz_full, "C:\\Users\\haoli\\Desktop\\buzz_full.csv")

# then we can use the aggregated score to rank the categories
buzz_full %>% 
  group_by(category) %>% 
  summarise(total_score = sum(score, na.rm = T)) %>% 
  arrange(desc(total_score))

# the top 5 categories are: animals, science, healthy_eating, technology, food
social_buzz_top5 <- buzz_full %>% 
  filter(category %in% c("animals", "science", "healthy_eating", "technology", "food"))
# write.csv(social_buzz_top5, "C:\\Users\\haoli\\Desktop\\social_buzz_top5.csv")
```


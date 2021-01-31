---
title: "Kenya Population and Housing Census"
summary: Total population by county from 2019 census
authors:
- admin
date: "2019-11-05"
categories: ["R"]
tags: 
- rmarkdown
- books
- tidyverse
- networks
---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = FALSE,
                      warning = FALSE,
                      message = FALSE,
                      fig.height = 21,
                      fig.width = 24)

```



```{r load_libraries}


library(plyr)
library(dplyr)
library(ggplot2)
library(ggthemes)
library(reshape2)
library(gdata)
library(stringr)
library(lubridate)
library(readxl)


```

```{r import_data}


data_kphc <- read_xlsx("~/Documents/RProjects/PMSite/data/county_popdata.xlsx",
                        sheet = "tidy_data")


# compute proportion change in population

data_kphc <- data_kphc %>%
  mutate(population_change = round((population_2019 - population_2009)/population_2009*100, 1),
         households_change = round((households_2019 - households_2009)/households_2009*100, 1)) %>%
  mutate(county_name = str_to_sentence(county_name))

data_pop <- data_kphc %>%
  dplyr::select(starts_with("county"),
                ends_with("change"),
                ends_with("2009"),
                ends_with("2019")) 

data_pop <- reshape(data_pop, 
                      varying=c(names(data_pop)[6:ncol(data_pop)]), 
                      direction="long", 
                      idvar=1:3, 
                      sep="_",
                      timevar="census_year")

row.names(data_pop) <- NULL

data_pop <- data_pop %>%
  group_by(census_year) %>%
  mutate(country_pop = sum(population)) %>%
  mutate(county_proportion = round(population/country_pop*100,1))

# ggplot(data_pop,
#        aes(x = reorder(county_name,
#                        county_code),
#            y = county_proportion)) +
#   geom_bar(stat = "Identity", 
#            aes(fill = factor(census_year)),
#            position = "dodge") +
#   geom_line(aes(x = county_name,
#                 y = population_change),
#             group = 1) +
#   theme_bw(base_size = 30, base_family = "sans") +
#   coord_flip()


ggplot(data_kphc,
       aes(x = reorder(county_name,
                       population_change),
           y = population_change)) +
  geom_line(group = 1, size = 1.5) +
  geom_hline(yintercept = c(0, 25, 50, 75)) +
  theme_pander(base_size = 25, 
            base_family = "sans") +
  theme(plot.caption = element_text(size = 15, face = "bold")) +
  scale_y_continuous(breaks = c(0, 25, 50, 75, 100)) +
  coord_flip() +
  labs(x = "County\n",
       y = "\nProportion (%) change",
       caption = "PM\n Data source: KPHC") +
  ggtitle("Proportion (%) change in population between 2009 and 2019")


# ggsave("~/Documents/RProjects/PMSite/figures/County_population_change.png",
#        height = 15,
#        width = 21)

data_change <- data_kphc %>%
  dplyr::select(starts_with("county"),
                #prop_change = population_change,
                ends_with("change")) %>%
  melt(id.vars = 1:3,
       variable.name = "category",
       value.name = "change") %>%
  mutate(category = str_replace(category,
                                pattern = "_",
                                replacement = " ")) %>%
  mutate(category = str_to_sentence(category))

ggplot(data_change,
       aes(x = reorder(county_name,
                       change),
           y = change)) +
  geom_line(aes(color = factor(category),
                group = category), 
                size = 1.5) +
  geom_hline(yintercept = c(0, 25, 50, 75)) +
  theme_pander(base_size = 25, 
            base_family = "sans") +
  theme(plot.caption = element_text(size = 15, face = "bold"),
        plot.title = element_text(size = 18),
        legend.position = "bottom") +
  scale_y_continuous(breaks = c(0, 25, 50, 75, 100)) +
  scale_color_wsj() +
  coord_flip() +
  labs(x = "County\n",
       y = "\nProportion (%) change",
       color = "",
       caption = "PM\n Data source: KPHC") +
  ggtitle("Proportion (%) change in population size and number of households between 2009 and 2019")


# ggsave("~/Documents/RProjects/PMSite/figures/County_population_hh_change.png",
#        height = 18,
#        width = 21)

```

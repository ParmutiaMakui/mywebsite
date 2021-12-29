---
title: "Health Facilities in Kenya"
summary: A look into the published list of public health facilities in Kenya
authors:
- admin
date: 2020-07-19
categories: ["Other"]
tags: 
- rmarkdown
- health
- kenya
- devolution
---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = FALSE,
                      warning = FALSE,
                      message = FALSE)

```

```{r Load libraries}


library(tidyverse)
library(ggthemes)
library(reshape2)
library(gdata)
library(readxl)
library(knitr)
library(kableExtra)
library(DT)
library(patchwork)
library(ggpubr)

```

```{r import_data}

# health facility levels
data_hf_level <- read_xlsx("~/Documents/RProjects/PMSite/data/healthfacilities/healthfacilities.xlsx",
                        sheet = "levels")

# health facility list
data_hf_list <- read_xlsx("~/Documents/RProjects/PMSite/data/healthfacilities/healthfacilities.xlsx",
                        sheet = "complete_table")

```


On 4th February, 2020 the Government of Kenya published a list of health facilities within the country. The  Kenya Gazette issue is available [**here**](http://kenyalaw.org/kenya_gazette/gazette/volume/MjA4Mw--/Vol.CXXII-No.24/). The notice gives details on levels/categories of health facilities, the type and description of facilities under each level as well as the services that would be expected to be offered by such a facility.  The table below gives a summary of the various levels and type of facilities under each.


## Health facilities categories

```{r hf_level}

data_hf_level[is.na(data_hf_level)] <- ""

knitr::kable(data_hf_level,
             format = "html",
      digits = 2,
      caption = "Table 1. Health Facility Categorization in Kenya") %>%
      kable_styling(bootstrap_options = "striped", 
                full_width = F,
                position = "center",
                font_size = "11")

```
  

The second schedule of the notice published the list of facility by county giving the name, level and ownership. From the notice, a list of **9064** health facilities are listed. Besides **2** cases being empty, some duplicates were also found which were excluded from the summaries below. 


## Total number of health facilities

After exclusion of identified duplicates and empty rows, a total of **9023** health facilities was identified. County Governments owned 50.6% of all the health facilities, 44.6% of the health facilities are private owned. Over 90% of the facilities were within Level 2 and Level 3B.

```{r hf_summary1, fig.height=9, fig.width=24}

duplicates_id <- c(223, 7818, 
                   628, 2527, 1346, 914, 567, 
                   234, 570, 572, 573, 964, 
                   574, 575, 455, 577, 578,
                   579, 580, 4312, 583, 6900,
                   5177, 585, 2594, 2600, 588,
                   589, 645, 590, 591, 594, 
                   6133, 595, 4913, 6572, 598,
                   6221, 409, 601, 602)

data_hf_list[] <- lapply(data_hf_list, str_trim)
# data_hf_list[] <- lapply(data_hf_list, str_to_sentence)

data_hf_list <- data_hf_list %>%
  mutate(ownership = if_else(ownership == "Public" |
                               ownership == "Government Of Kenya" |
                               ownership == "Government" |
                               ownership == "None", 
                             "County Government",
                      if_else(ownership == "Faith Based", 
                              "Faith Based Organization", ownership))) %>%
  mutate(ownership = str_to_title(ownership))

duplicate_cases <- data_hf_list %>%
  filter(number %in% duplicates_id)

data_hf_list <- data_hf_list %>%
  filter(!number %in% duplicates_id)

summary_ownership <- data_hf_list %>%
  group_by(ownership) %>%
  summarise(Number = n()) %>%
  mutate(Total_num = sum(Number),
         Proportion = round(Number/Total_num*100, 2))

summary_level <- data_hf_list %>%
  group_by(level) %>%
  summarise(Number = n()) %>%
  mutate(Total_num = sum(Number),
         Proportion = round(Number/Total_num*100, 2))

summary_ownershiplevel <- data_hf_list %>%
  group_by(ownership, level) %>%
  summarise(Number = n()) %>%
  mutate(Total_num = sum(Number),
         Proportion = round(Number/Total_num*100, 2))

plot_ownership <- summary_ownership %>%
  ggplot(aes(x = fct_reorder(ownership, Number),
             y = Proportion)) +
  geom_col() +
  geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            hjust = -.4,
            size = 8,
            fontface = "bold",
            position = position_dodge(width = 1)) +
    theme_bw(base_size = 30, base_family = "sans") +
    theme(legend.position = "bottom") +
  coord_flip() +
  ylim(NA, 70) +
  labs(x = "",
       y = "Proportion",
       title = "Ownership of health facilities")


plot_level <- summary_level %>%
  ggplot(aes(x = fct_reorder(level, Number),
             y = Proportion)) +
  geom_col() +
  geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            hjust = -.4,
            size = 8,
            fontface = "bold",
            position = position_dodge(width = 1)) +
    theme_bw(base_size = 30, base_family = "sans") +
    theme(legend.position = "bottom") +
  coord_flip() +
  ylim(NA, 90) +
  labs(x = "",
       y = "Proportion",
       title = "Category of health facilities")

plot_ownership | plot_level

```

```{r hf_summary2, fig.height=18, fig.width=24}

summary_ownershiplevel %>%
  ggplot(aes(x = level,
             y = Proportion,
             fill = level)) +
  geom_col(position = position_dodge(width = 1)) +
  geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            hjust = -.2,
            size = 6,
            fontface = "bold",
            position = position_dodge(width = 1)) +
    theme_bw(base_size = 30, base_family = "sans") +
    theme(legend.position = "none") +
  scale_fill_discrete() +
  facet_wrap(~ownership, ncol = 3) +
  coord_flip() +
  ylim(NA, 100) +
  labs(x = "",
       y = "Proportion",
       title = "Health facility ownership by category")
  

```


## Health facilities by county

Table 2 shows top 10 counties by the total number of health facilities. **Nairobi County** has the highest number of health facilities (**745**) which is equivalent to **8.3%** of the total number of health facilities. Table 3 shows 10 counties with the lowest number of health facilities. **Lamu County** had the lowest number with only 23 health facilities which is less than **1%** of the total number of health facilities within the country. 


```{r hf_summary3}

summary_county <- data_hf_list %>%
  group_by(county) %>%
  summarise(Number = n()) %>%
  mutate(Total_num = sum(Number),
         Proportion = round(Number/Total_num*100, 2))

top10 <- summary_county %>%
  arrange(desc(Number)) %>%
  slice_head(n = 10) %>%
  select(-Total_num)


knitr::kable(top10,
             format = "html",
      digits = 2,
      caption = "Table 2. Counties with highest number of hospitals") %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                position = "float_left",
                font_size = "11")


bot10 <- summary_county %>%
  arrange(desc(Number)) %>%
  slice_tail(n = 10) %>%
  select(-Total_num)


knitr::kable(bot10,
             format = "html",
      digits = 2,
      caption = "Table 3. Counties with lowest number of hospitals") %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                position = "right",
                font_size = "11")

```


```{r hf_summary4, fig.height=18, fig.width=15}

ggdotchart(summary_county, 
           x = "county", 
           y = "Number",
           # color = "cyl",                                # Color by groups
           # palette = c("#00AFBB", "#E7B800", "#FC4E07"), # Custom color palette
           sorting = "descending",              # Sort value in descending order
           add = "segments",                # Add segments from y = 0 to dots
           rotate = TRUE,                                # Rotate vertically
           # group = "cyl",                                # Order by groups
           dot.size = 10,                                 # Large dot size
           label = round(summary_county$Proportion, 1),  # Add mpg values as dot labels
           font.label = list(color = "white", 
                             size = 9, 
                             vjust = 0.5),               # Adjust label parameters
           ggtheme = theme_pubr(),    # ggplot2 theme
           title = "Total number of hospitals by county"
           )
  

```

## Health facilities by county, ownership and level


```{r hf_summary5}

summary_countyol <- data_hf_list %>%
  group_by(county, ownership, level) %>%
  summarise(Number = n()) %>%
  mutate(Total_num = sum(Number),
         Proportion = round(Number/Total_num*100, 2))

datatable(summary_countyol,
          caption = "Tabla 4. Ownership and category of health facility by county")

```


## Disclaimer 

> The summaries and graphs shown here are based on the data within the Gazette notice above.  
> I have not looked at the overlap or difference between the data used here and data found in: [Kenya Master Health Facility List](http://kmhfl.health.go.ke/#/home)



---
title: "Add linear equation to a plot"
summary: Below is a way of adding an equation to a plot
authors:
- admin
date: 2018-08-16
output: 
  html_document:
    highlight: tango
    theme: "journal"
    code_folding: hide
categories: ["R"]
tags: 
- rmarkdown
- plot
- regression
- tidyverse
draft: false
---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE,
                      message = FALSE,
                      warning = FALSE,
                      fig.height = 15,
                      fig.width = 18,
                      fig.align = "center")

```

In this post, I am going to show different ways of adding a linear equation to a plot. I will use 
the **diamonds** dataset that is within the **ggplot2** package.
The dataset has nearly _54,000_ observations and _10_ variables as shown below.

```{r Load libraries}

library(ggplot2)
library(ggpmisc)
library(ggthemes)
library(foreign)
suppressMessages(library(dplyr))
library(reshape2)
library(psych)

```

There are many ways of loading data but in this case, I choose to do as shown below. Below we see that
the structure of the _diamonds_ dataset - using the _str_ function.
```{r Load data}

diamonds <- ggplot2::diamonds

dim(diamonds)
str(diamonds)

```


## Explore dataset

A good start for summarizing a dataset (especially for datasets with few number of columns), is the 
use of the base _summary_ function as below. The result is a summary of each column in the dataset -
frequency for categories within a _categorical_ variable and summary statistics for _numeric_ variables. 

```{r explore data}

#general summary of all variables
summary(diamonds)


```

An interesting way of summarizing many numeric variables by one categorical variables is shown below.
I use the _describeBy_ function from the **_psych_** package. The table below shows the first 15 cases
for 10 columns

```{r summary of numeric variables by carat}

#summary of all numeric variables by carat
summary_num <- diamonds %>%
  dplyr::select(-c(color, clarity)) %>%
  melt(id.vars = 1:2,
       variable.name = "variable_name",
       value.name = "variable_value")

summary_la <- summary_num %>%
  filter(!is.na(variable_value))

summarystats_num <- describeBy(summary_num$variable_value,
                               list(summary_la$cut, summary_la$variable_name),
                               mat = T)


row.names(summarystats_num) <- NULL

knitr::kable(summarystats_num[1:15, 1:10])

```

Besides computation of numerical data summaries, use of data visualizations is a very great way 
to explore data. A plot showing bivariate distributions betweeen price and carat is shown below for 
each level of _cut_.

> Note there are no prior assumptions made about the distribution of the variables.

```{r plot 1}

#plot of diamond price by carat grouped by cut
formula_lm <- y ~ x

ggplot(diamonds,
       aes(x = price,
           y = carat,
           color = cut)) +
    geom_point() +
    geom_smooth(method = "lm", size = 3, se = T, formula = formula_lm) +
    stat_poly_eq(aes(label = paste(..eq.label.., ..rr.label.., sep = "~~~")), 
               label.x.npc = "right", label.y.npc = 0.15,
               formula = formula_lm, parse = TRUE, size = 10) +
    theme_bw(base_size = 30, base_family = "sans") +
  theme(axis.text = element_text(face = "bold"),
        legend.text = element_text(face = "bold"),
        legend.title = element_text(face = "bold"),
        axis.title = element_text(face = "bold"),
        strip.text = element_text(face = "bold"),
        legend.position = "bottom") +
  scale_color_stata() +
  #facet_wrap(~cut) +
  labs(x = "Price",
       y = "Carat\n",
       color = "Cut")


```


The plot below shows a scatterplot of _price_ and _carat_ with the linear regression line plotted 
through the origin.

```{r plot 2}
#with the line starting at the origin
formula_lm <- y ~ 0 + x

ggplot(diamonds,
       aes(x = price,
           y = carat,
           color = cut)) +
    geom_point() +
    geom_smooth(method = "lm", size = 3, se = T, formula = formula_lm) +
    stat_poly_eq(aes(label = paste(..eq.label.., ..rr.label.., sep = "~~~")), 
               label.x.npc = "right", label.y.npc = 0.15,
               formula = formula_lm, parse = TRUE, size = 10) +
    theme_bw(base_size = 30, base_family = "sans") +
  theme(axis.text = element_text(face = "bold"),
        legend.text = element_text(face = "bold"),
        legend.title = element_text(face = "bold"),
        axis.title = element_text(face = "bold"),
        strip.text = element_text(face = "bold"),
        legend.position = "bottom") +
  scale_color_stata() +
  #facet_wrap(~cut) +
  labs(x = "Price",
       y = "Carat\n",
       color = "Cut")


```

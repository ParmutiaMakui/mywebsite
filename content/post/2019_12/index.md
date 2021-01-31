---
title: "Data Manipulation in R with dplyr"
subtitle: "ICRAFuseR Beginner Series"
summary: Introduction to **dplyr** 
authors:
- admin
date: "2019-12-05"
output: 
  slidy_presentation:
    highlight: pygments
    icremental: true
    fig.align: center
    footer: "ICRAFuseR 2019"
tags:
- rmarkdown
- dplyr
- tidyverse
- training
---


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE,
                      fig.height = 12,
                      fig.width = 12,
                      warning = FALSE)
```



```{r Load libraries}

suppressPackageStartupMessages(library(tidyverse))
library(readxl)
library(ggthemes)

```


```{r Load data}

# import dataset 
data_kphc <- read_xlsx("~/Documents/RProjects/PMSite/data/county_popdata.xlsx",
                        sheet = "tidy_data")

# change data frame to tbl
data_kphc <- tbl_df(data_kphc)

```



# Introduction

- Data manipulation is a way(s) of modifying a dataset 
- Is the process of making data more organized.
- This involves ways of **selecting, inserting, deleting, sorting** and **summarising** data.
- Common packages used in data manipulation in R are:
    - **dplyr** 
    - **data.table**
- Each of packages has its own pros and cons.
- We will focus on [**dplyr**](https://dplyr.tidyverse.org/) in this tutorial


# Using dplyr for data manipulation

- dplyr is part of the [**tidyverse**](https://www.tidyverse.org/)
- dplyr has 5 main verbs used for the common data manipulation tasks.
    - **select** - use to select one or more columns
    - **filter** - used to select rows/cases based on a particular criteria
    - **arrange** - sort data based on one or more columns
    - **mutate** - used to compute new columns
    - **summarise** - used to compute data summaries base on particular column(s)
- Renaming variables
- Combining variables and observations
- Combining datasets 


# Manipulating variables: dplyr::select()

- dplyr::select() helps in creating subsets with specific variables or exclude various variables
- It is used to select one or more columns from a data frame.
- Below are several examples using **select** function from dplyr

```{r dplyr select example 1, echo=TRUE}

# option one
# select variables by writing their names explicitly
data_sub <- select(data_kphc,
                   county_name, county_region, hhsize_2019)

# option two
# select variables using by giving a range between two variables
data_sub <- select(data_kphc,
       county_code:county_region)

# option three 
# use of helper functions within select
# starts_with()
# ends_with()
# contains()
data_sub <- select(data_kphc,
                   starts_with("county"),
                   starts_with("hhsize"))

# display results
data_sub

# select variables that contain county in their names
select(data_kphc,
       contains("county"))

```

# 

- You can also use **select** to remove variables from a data set as shown below
- Below were remove **voter_turnout** and **households_2009** variables from the data

```{r dplyr select example 2, echo=TRUE}

# remove voter_turnout from the dataset
data_sub <- select(data_kphc,
                   -voter_turnout)

# names of columns in the new dataset
names(data_sub)

# remover voter_turnout and households_2009 from the dataset
data_sub <- select(data_kphc, 
                   -c(voter_turnout, households_2009))

# names of columns in the new dataset
names(data_sub)

# display the first 3 cases fo the results
data_sub[1:3,]

```


# Manipulating observations: dplyr::filter()

- It is used with logical statements to select cases that meet a given criteria/condition
- Below we select data for **Nairobi county**

```{r dplyr::filter example, echo=TRUE}

# filter all data for NAIROBI CITY
data_sub <- filter(data_kphc,
                   county_name=="NAIROBI CITY")

# number of rows and columns in NAIROBI CITY data
dim(data_sub)

```


# Arrange cases: dplyr::arrange()

- Used to order rows of data
- Can be used with one column or multiple columns
- Below we use **arrange** to sort the data set based on **county_region** column

```{r dplyr::arrange example, echo=TRUE}

# using one column to arrange/sort data
data_sub <- arrange(data_kphc, county_region)

# using multiple columns to arrange/sort data
data_sub <- arrange(data_kphc, 
                    county_region, population_2019)

```

# Compute new variables: dplyr::mutate()

- It is used to compute or add new columns to a data frame
- Below we use **mutate** to add a new variable to the dataset

```{r dplyr::mutate example, echo=TRUE}

# compute proportion change in population and households sizes between 2009 and 2019
data_kphc <- mutate(data_kphc,
                    population_change = round((population_2019 - population_2009)/population_2009*100, 1),
                    households_change = round((households_2019 - households_2009)/households_2009*100, 1))

# summary of population change
summary(data_kphc$population_change)

# summary of households change
summary(data_kphc$households_change)

```

# Summarise observations: dplyr::summarise()

- Used to compute summary statistics from the data
- Computed below is the average **population_change** value.
- This is mostly used with **group by** function when creating summaries.

```{r dplyr::summarise example, echo=TRUE}

# summarise population change 
# compute average population change for all counties
popchange_summary <- summarise(data_kphc,
                               avg_popchng = mean(population_change))

# display result
popchange_summary

popchange_region <- data_kphc %>%
  group_by(county_region) %>%
  summarise(avg_popchng = mean(population_change))

```

# Renaming variables: dplyr::rename()

- dplyr::rename() function is used to rename variables

```{r renaming, echo=TRUE}

# subset of data with variables that start with county
data_sub <- select(data_kphc,
                   starts_with("county"))

# names of selected variables
names(data_sub)

# rename county_name variable
# new name in the LHS and the old name in the RHS
# variable names maybe in quotes or not
data_sub <- rename(data_sub,
                   county = county_name)

# variables names after rename
names(data_sub)


```

# Combining all or some operations/steps

- Using the **"piping"** operator, we can make our code more readable and be able to combine
several operations in one step.
- The **"piping"** operator can be pronounced as **then**
- Below is a simple example


```{r piping_operator_1, echo=TRUE}

# compute average population change by region
popchange_summary <- data_kphc %>%
  group_by(county_region) %>%
  summarise(avg_pc = round(mean(population_change), 1)) %>%
  arrange(desc(avg_pc)) 

# display the first 5 cases
head(popchange_summary, 5)


```


# 

```{r piping_operator_2, echo=TRUE}

# combining several steps (even plotting) using the pipe operator
data_kphc %>%
  mutate(county_name = str_to_sentence(county_name)) %>%
ggplot(aes(x = reorder(county_name,
                       population_change),
           y = population_change)) +
  geom_line(group = 1, size = 1.5) +
  geom_hline(yintercept = c(0, 25, 50, 75)) +
  theme_pander(base_size = 15, 
            base_family = "sans") +
  theme(plot.caption = element_text(size = 15, face = "bold")) +
  scale_y_continuous(breaks = c(0, 25, 50, 75, 100)) +
  coord_flip() +
  labs(x = "County\n",
       y = "\nProportion (%) change",
       caption = "PM\n Data source: KPHC") +
  ggtitle("Proportion (%) change in population between 2009 and 2019")


```


# Combining variables

- dplyr::bind_cols() is used to combine variables
- The order of cases should be cross-checked before doing this

```{r combining_variables, echo=TRUE}

# subset of data with the columns on county variables
data_county <- select(data_kphc,
                      county_code:county_region)

# number of rows and columns in data_county
dim(data_county)

# variable names in data_county
names(data_county)

# subset of data with variables on population
data_population <- select(data_kphc,
                          starts_with("pop"))

# number of rows and columns in data_population
dim(data_population)

# variable names in data_population
names(data_population)

# combine columns from data_county and data_population
data_cols <- bind_cols(data_county,
                       data_population)

# number of rows and columns in data_cols
dim(data_cols)

# variable names in data_cols
names(data_cols)

```


# Combining observations

- dplyr::bind_rows() is used to combine observations
- The names of variables should match!!

```{r combining_observations, echo=TRUE}

# subset of coast region counties data
data_coast <- filter(data_kphc,
                     county_region == "Coast")

# number of rows and colums in data_coast
dim(data_coast)

# variable names in data_coast
# names(data_coast)

# subset of central region counties data
data_central <- filter(data_kphc,
                  county_region == "Central")

# number of rows and colums in data_central
dim(data_central)

# variable names in data_central
# names(data_central)

# bind cases from both datasets
data_cc <- bind_rows(data_coast,
                     data_central)

# number of rows and colums in data_cc
dim(data_cc)

# variable names in data_cc
# names(data_cc)

```


# Combining datasets

- This done using **mutating joins**
- Enables you to combine two tables 

```{r mj_data, echo=TRUE}

# variables with county description
data_x <- data_kphc %>%
  select(contains("county"))

# number of rows and columns in data_x
dim(data_x)

# data on 2019 population  and household size variables
# this data is for only coast and rift valley regions
data_y <- data_kphc %>%
  filter(county_region == "Coast" |
           county_region == "Rift Valley") %>%
  select(county_code,
         ends_with("19"))

# number of rows and columns in data_y
dim(data_y)


```


## dplyr::left_join()

```{r left_join, echo=TRUE}

# left_join()
# enables you to combine matching values from table y to table x
# option one
data_xy <- left_join(data_x, data_y, 
                     by = "county_code")

# option two
data_xy <- data_x %>%
  left_join(data_y,
            by = "county_code")

```

## dplyr::right_join()

```{r right_join, echo=TRUE}

# right_join()
# enables you to combine matching values from table x to table y
# only maintains values in tably y
# option one
data_xy <- data_x %>%
  right_join(data_y,
            by = "county_code")

# option two
data_xy <- right_join(data_x, data_y, 
                     by = "county_code")

```


## dplyr::inner_join()

```{r inner_join, echo=TRUE}

# inner_join()
# maintains only matching values from both x and y tables
data_xy <- data_x %>%
  inner_join(data_y,
            by = "county_code")

data_xy <- inner_join(data_x, data_y, 
                     by = "county_code")

```


## dplyr::full_join()

```{r full_join, echo=TRUE}

# inner_join()
# maintains all values/cases from both tables
# option one
data_xy <- data_x %>%
  full_join(data_y,
            by = "county_code")

# option two
data_xy <- full_join(data_x, data_y, 
                     by = "county_code")

```

# General advice and detailed info

- Practice, practice and practice some more
- [tidyverse website](https://www.tidyverse.org/packages/)
- [dplyr website](https://dplyr.tidyverse.org/)
- Google is your friend  

![](keepatit.jpeg)


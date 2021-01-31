---
title: "Manipulating data in R"
authors:
- admin
date: 2016-11-20
categories: ["R"]
tags: ["rmarkdown", "data manipulation", "tidyverse"]
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE)
```

```{r Load libraries}


suppressWarnings(suppressPackageStartupMessages(library(dplyr)))

suppressWarnings(suppressPackageStartupMessages(library(foreign)))

```

```{r Load data}

sldata <- read.dta("~/Documents/RProjects/RTrainings/Data/SL_Computed_Indicators_Subset_Data_2016-11-11.dta")

```


# Data manipulation

- This involves ways of **selecting, inserting, deleting, sorting** and 
**summarising** data.
- Is the process of making data more organized.
- Common packages used in data manipulation in R are:
    - **dplyr** 
    - **data.table**
- Each of packages has its own pros and cons.
- We will focus on **dplyr** in this tutorial

# Using dplyr for data manipulation

- dplyr has 5 main verbs used for the common data manipulation tasks.
    - **select** - use to select one or more columns
    - **filter** - used to select rows/cases based on a particular criteria
    - **arrange** - sort data based on one or more columns
    - **mutate** - used to compute new columns
    - **summarise** - used to compute data summaries base on particular column(s)


# dplyr::select()

- It is used to select one or more columns from a data frame.
- Below is an example

```{r dplyr select example 1, echo=TRUE}

data_sub <- select(sldata,
                   starts_with("new_SLID"),
                   starts_with("CID"),
                   starts_with("Landscape"),
                   starts_with("new_SID"),
                   starts_with("HDDS"),
                   starts_with("HDA_index"))

data_sub[1:3,]

```

# dplyr::select() continued
- You can also use **select** to remove variables from a data set as shown below
- Below were remove **HDA index** variable from the data

```{r dplyr select example 2, echo=TRUE}

data_sub <- select(data_sub, -HDA_index)

data_sub[1:3,]

```


# dplyr::filter()
- It is used with logical statements to select cases that meet the criteria given
- Below we select all cases from **West Africa** landscape.

```{r dplyr::filter example, echo=TRUE}

data_sub <- filter(sldata,
                   Landscape=="West Africa")

dim(data_sub)

```


# dplyr::arrange()

- Used to order rows of data
- Can be used with one column or multiple columns
- Below we will arrange the data set based on **Country** column, followed by the 
**Landscape** column

```{r dplyr::arrange example, echo=TRUE}

sldata <- arrange(sldata, CID, Landscape)

```

# dplyr::mutate()
- It is used to compute or add new columns to a data frame
- Below we will compute a new variable **HHibi** using variables **Farm size Ha** and
**HH size** already in the data frame.

```{r dplyr::mutate example, echo=TRUE}

sldata <- mutate(sldata, 
                 HHibi = (Farm_size_Ha/(HH_size * 365))*100)

summary(sldata$HHibi)

```

# dplyr::summarise()
- Used to compute summary statistics from the data
- Computed below is the average **HDDS** value.
- This is mostly used with **group_by** function when creating summaries.

```{r dplyr::summarise example, echo=TRUE}

hdds_summary <- summarise(sldata,
                          avg_hdds = mean(HDDS))

hdds_summary[,]

```


# Combining all or some operations
- Using the **"piping"** operator, we can make our code more readable and be able to combine
several operations in one step.
- The **"piping"** operator can be pronounced as **then**
- Below is a simple example


```{r piping operator example, echo=TRUE}

hdds_summary <- sldata %>%
  group_by(CID) %>%
  summarise(avg_hdds = round(mean(HDDS), 1)) %>%
  arrange(desc(avg_hdds))

hdds_summary[3:5,]


```



# General advice

- Practice, practice and practice some more
- Google is your friend
- Know how to reproduce your **error**


---
title: "MPESA Costs: Exploratory analysis"
summary: Exploring the transaction costs of MPESA
authors:
- admin
date: "2019-10-01"
lastmod: "2020-12-31T00:00:00Z"
categories: ["R"]
tags: 
- rmarkdown
- data manipulation
- tidyverse
- m-pesa
draft: false
---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = TRUE,
                      warning = FALSE,
                      message = FALSE)

```

In this blog post I'm going to show how I processed data from my [**M-Pesa**](https://www.safaricom.co.ke/personal/m-pesa) monthly statements from January 2016 to December 2019. M-Pesa is a mobile phone-based money transfer, financing and microfinancing service, launched in 2007 by [**Safaricom Limited**](https://www.safaricom.co.ke/). A user of the service is notified of transfer or receipt of money via text messages. 

A users monthly transaction history using the platform can be shared upon subscription to receipt of monthly statements which follow a particular format and send to the user's email. The statement has a summary statement of the total amount of money transfered, received, payment for services etc besides a detailed statement of each transaction within the calender month.

The statement are shared in encrypted PDF documents where the user has to use his/her ID to access. In order to create **Excel** sheets from the tabular summaries within the PDF documents, I saved each of the statements as a new PDF document without the encryption. I then used [**Tabular**](http://tabula.technology/) to extract each of the two summary statements to Excel (xlsx) workbooks for each month which are further processed as shown in the code sections below.

## Load libraries

Below I import the required packages that enable data processing and visualization.

```{r Load libraries}

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

## Load data

All the extracted Excel files are all stored in the same folder whose path is indicated below. In order to import all the files for each month, I use two list objects; 

 - **statement_summary**: will be a list of summary statements   
 - **statement_detailed**: will be a list of detailed statements    

Each of the holds the specific monthly statement for each month. While importing the data files for the **detailed statements** I specify the column types for the 7 columns within each of the statements.

```{r data_importing}

# folder path with the extracted xlsx files
fpath <- "~/Documents/PersonalDocuments/MPESA/Data/"

# list all files of the format (.xlsx) in the specified folder
lfiles <- list.files(path = fpath, 
                     pattern = "*.xlsx")

# create empty list to hold the files from both statements for all months
statement_summary <- list()
statement_detailed <- list()

for(file in lfiles)
  {
  perpos <- which(strsplit(file, "")[[1]]==".")
  assign(
    gsub(" ","",substr(file, 1, perpos-1)), 
    
    # load data files for the first list - overall summary statement
    statement_summary[[file]] <- read_xlsx(paste(fpath, 
                                                 file, 
                                                 sep = ""),
              sheet = "summary_statement"),
    
    # load data files for the second list - detailed statement
    statement_detailed[[file]] <- read_xlsx(paste(fpath, 
                                                  file, 
                                                  sep = ""),
              sheet = "detailed_statement",
              col_types = c("text",
                            "date",
                            "text",
                            "text",
                            "numeric",
                            "numeric",
                            "numeric"))
    )
    
}


```

## Data processing  

In the section below I combine all the monthly statements to individual files. I use **ldply** function from the [**plyr**](https://www.jstatsoft.org/article/view/v040i01) package which takes in a list(s) and returns a dataframe in this case. Next I work on the column names to harmonize them for ease of use/reference. I like column/variable names within my datasets in lower case and use of underscores (_) rather than a fullstop(.) between variable names that have more than one word.

Next, within the **detailed statements** data, I compute the columns below using the **mutate** function of the [**dplyr**](https://dplyr.tidyverse.org/) package:  

 - **Transaction completion date and time (completion_time)**: - converted in a proper date element with year, month, date, hours, minutes and seconds using the **ymd_hms** function from the [**lubridate**](https://lubridate.tidyverse.org/) package.  
 - **Transaction year (tran_year)**: - This is extracted from the formated transaction date-time column.  
 - **Transaction month (tran_month)**:- This is also extracted from the formated transaction date-time column. This is then converted to a factor with the names of calender months.

In the final part of this code section, I create a **character vector (charges)** with the names of the transaction categories which I'm interested in. The three categories represent the different types of transaction charges depending on the type of transaction:  
 - **Withdrawal Charge**: Cost incured when withdrawing money from the various options where a user can withdraw funds from their **M-Pesa** account.  
 - **Pay Bill Charge**: Cost incured when making merchant payments or transfer or receipt of funds to/from entities such as banks.  
 - **Customer Transfer of Funds Charge**: Cost incured when sending funds to other **M-Pesa** users or non-users.  

```{r data_processing}

# combine data files
df_ds <- plyr::ldply(statement_detailed)
df_ss <- plyr::ldply(statement_summary)


# name columns for summary data
names(df_ss) <- str_replace(names(df_ss),
                            pattern = " ",
                            replacement = "_")

names(df_ss) <- str_to_lower(names(df_ss))

df_ss <- df_ss %>%
  rename("file_id" = ".id")

# name columns for detailed data
names(df_ds) <- str_replace(names(df_ds),
                            pattern = " ",
                            replacement = "_")

names(df_ds) <- str_to_lower(names(df_ds))

df_ds <- df_ds %>%
  rename("file_id" = ".id",
         "receipt_no" = "receipt_no.")

# remove extra column headers within data
df_ds <- df_ds %>%
  filter(details != "Details") %>%
  mutate(completion_time = ymd_hms(completion_time)) %>%
  mutate(tran_year = year(completion_time),
         tran_month = month(completion_time)) %>%
  mutate(tran_month = factor(tran_month,
                             levels = 1:12,
                             labels = month.name))

# vector of charge categories
charges <- c("Withdrawal Charge", 
             "Pay Bill Charge",
             "Customer Transfer of Funds Charge")

```


# Exploratory data analysis

## Proportion of charges by year

The graph below shows the proportion of charges for each category to the total charges for each year. Its notable that **withdrawal charges** have been decreasing over time. Withdrawal charges started off at 37% in 2016 compared to 29.8% in 2018 and accounting for 20.9% of the total charges within 2019. This is mostly due to most entities adopting mobile payments options. 

```{r charges_summary_year, fig.height=15, fig.width=18}

#summary of charges by year
summary_charges_year <- df_ds %>%
  mutate(withdrawn = ifelse(withdrawn < 0, withdrawn*-1, withdrawn)) %>%
  filter(details %in% charges) %>%
  mutate(details = if_else(details == "Customer Transfer of Funds Charge", 1,
                           if_else(details == "Withdrawal Charge", 2, 3))) %>%
  mutate(details = factor(details,
                          levels = 1:3,
                          labels = c("Customer Transfer of Funds Charge",
                                     "Withdrawal Charge",
                                     "Pay Bill Charge"))) %>%
  group_by(tran_year, details) %>%
  summarise(amount = sum(withdrawn)) %>%
  mutate(Total_amount = sum(amount),
         Proportion = round(amount/Total_amount*100, 1))


ggplot(summary_charges_year,
       aes(x = tran_year,
           y = Proportion,
           fill = details)) +
    geom_bar(stat = "Identity", 
             position = position_dodge()) +
    geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            vjust = -.4,
            size = 5,
            fontface = "bold",
            position = position_dodge(width = 1)) +
    theme_bw(base_size = 25, base_family = "sans") +
    theme(legend.position = "bottom") +
    scale_fill_economist() +
    ylim(NA, 80) +
    labs(x = "",
         y = "Proportion (%)\n",
         fill = "Charges:")

```


## Trend of charges by month

The trend of the proportion of charges within each year is presented below. Across the four years, transfer of funds charges account for most of the charges within each year. 

```{r charges_graphs_a, fig.height=21, fig.width=28}

#summary of charges by year and month
summary_charges_month <- df_ds %>%
  mutate(withdrawn = ifelse(withdrawn < 0, withdrawn*-1, withdrawn)) %>%
  filter(details %in% charges) %>%
  mutate(details = if_else(details == "Customer Transfer of Funds Charge", 1,
                           if_else(details == "Withdrawal Charge", 2, 3))) %>%
  mutate(details = factor(details,
                          levels = 1:3,
                          labels = c("Customer Transfer of Funds Charge",
                                     "Withdrawal Charge",
                                     "Pay Bill Charge"))) %>%
  group_by(tran_year, tran_month, details) %>%
  summarise(amount = sum(withdrawn)) %>%
  mutate(Total_amount = sum(amount),
         Proportion = round(amount/Total_amount*100, 2))

ggplot(summary_charges_month,
       aes(x = tran_month,
           y = Proportion)) +
    geom_path(aes(colour = details, 
                  group = details), 
              size = 1.8) +
    theme_bw(base_size = 25, base_family = "sans") +
    theme(legend.position = "bottom",
          axis.text.x = element_text(angle = 90)) +
    scale_colour_economist() +
    facet_wrap(~tran_year) +
    ylim(NA, 80) +
    labs(x = "",
         y = "Proportion (%)\n",
         colour = "Charges:")


```

The graph below offers a different perspective of viewing the same data presented above. The trend of each charge category is compared across each year.

```{r charges_graph_b, fig.height=21, fig.width=24}

ggplot(summary_charges_month,
       aes(x = factor(tran_month),
           y = Proportion)) +
    geom_path(aes(colour = factor(tran_year), 
                  group = factor(tran_year)), 
              size = 1.8) +
    theme_bw(base_size = 25, base_family = "sans") +
    theme(legend.position = "right") +
    scale_colour_economist() +
    facet_wrap(~details, ncol = 1) +
    ylim(NA, 80) +
    labs(x = "",
         y = "Proportion (%)\n",
         colour = "Year")


```


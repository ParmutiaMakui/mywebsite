---
title: "The Nobel Memorial Prize in Economic Science: Network of doctoral advisors"
summary: A social network view of doctoral advisors and winners of the Nobel prize in Economic Science.
authors:
- admin
output:
  html_document:
    code_folding: hide
    highligh: tango
    theme: spacelab
    toc: yes
    toc_float: yes
date: 2018-10-05
categories: ["R"]
tags: 
- rmarkdown
- networks
- sna
- nobel prize
---

```{r setup, include=FALSE}

knitr::opts_chunk$set(echo = FALSE,
                      message = FALSE,
                      warning = FALSE,
                      fig.height = 16,
                      fig.width = 24)
```


```{r Functions}

import_library <- function(lib_name){
    suppressWarnings(suppressMessages(require(lib_name, 
                                              character.only = TRUE)))
}


```


```{r Load libraries}

import_library("foreign")
import_library("dplyr")
import_library("rvest")
import_library("splitstackshape")
import_library("stringr")
import_library("ggnet")
import_library("ggraph")
import_library("ggplot2")
import_library("ggthemes")
import_library("igraph")


```


## Introduction

Every year, the Royal Swedish Academy of Sciences awards **The Nobel Memorial Prize in Economic Sciences**, officially known as
**The Sveriges Riksbank Prize in Economic Sciences in Memory of Alfred Nobel** to researchers in the field of economic sciences.
**Ragnar Frisch and Jan Tinbergen** were the first recipients of the prize in **1969**. 

The award is presented in Stockholm at the annual Nobel Awards ceremony on December 10. As of this year, 2018, 81 individuals have 
received the prize. The analysis below explores the network between recipients of the award and their doctoral advisors starting with the data reported [here.](https://en.wikipedia.org/wiki/List_of_Nobel_Memorial_Prize_laureates_in_Economics)


### Getting data

The list of the award recipients was first accessed from [Wikipedia](https://en.wikipedia.org/wiki/List_of_Nobel_Memorial_Prize_laureates_in_Economics) using **rvest** package in **R**. 
From the above page, the **Wikipedia** page of each recipient was accessed to get details such as **doctoral advisor, influences, doctoral students and influencers**. The analysis below uses the data for **doctoral advisors** for each recipient, if such information is available. At the time of this analysis the doctoral advisors for 16 laurates was not available hence excluded from the network analysis.  

```{r Load data}

#load link to the wikipedia page with the Nobel Prize in Economics
econ_wiki <- "https://en.wikipedia.org/wiki/List_of_Nobel_Memorial_Prize_laureates_in_Economics"

#extract tables
econ_tab <- read_html(econ_wiki) %>%
  html_nodes("table") %>%
  html_table(fill = T)

#data with the details of economics nobel laureates
econ_lau <- econ_tab[[1]]

write.csv(econ_lau,
          "~/Documents/RProjects/PMSite/data/Economics_laurates.csv",
          row.names = F)

#data of nobel laureates
econ_net <- read.csv("~/Documents/RProjects/PMSite/data/Nobel_laureates_dadvisors.csv")

```

## Exploring the data

### Number of recipients by year

The graph below gives the summary of the number of recipients for each year. 
```{r recipients_year}

summary <- econ_net %>%
  group_by(Year) %>%
  summarise(Number_rec = n())

ggplot(summary,
       aes(x = Year,
           y = Number_rec)) +
  geom_bar(stat = "Identity") +
  theme_economist(base_size = 30, base_family = "sans") +
  theme(axis.title = element_text()) +
  ylim(0, NA) +
  labs(x = "\nYear",
       y = "Number of recipients\n")

```

> Half of the time **50%**, only one recipient is awarded the prize as presented below.

```{r recipientsnum_year}
summary <- econ_net %>%
  group_by(Year) %>%
  summarise(Number_rec = n()) %>%
  group_by(Number_rec) %>%
  summarise(Number_year = n()) %>%
  mutate(Total_year = sum(Number_year),
         Proportion = round(Number_year/Total_year*100,1)) %>%
  ungroup()

ggplot(summary,
       aes(x = Number_rec,
           y = Number_year)) +
  geom_bar(stat = "Identity") +
  geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            vjust = -.3,
            fontface = "bold",
            size = 7,
            #vjust = -.2,
            position = position_dodge(width = 1)) +
  theme_economist(base_size = 30, base_family = "sans") +
  theme(axis.title = element_text()) +
  ylim(0, NA) +
  labs(x = "\nNumber of recipients",
       y = "Number of years\n")


```


### Number of recipients by country

A summary of the number of recipients by country is presented below. Most of the recipients, **59.3%** are 
citizens of the United States with **7.4%** of the recipients being United Kingdom citizens. Several of the recipients are citizens have dual citizenship.
```{r recipients_country}

summary <- econ_net %>%
  group_by(Country) %>%
  summarise(Number_year = n()) %>%
  mutate(Total_year = sum(Number_year),
         Proportion = round(Number_year/Total_year*100,1),
         Country = as.character(Country)) %>%
  arrange(desc(Number_year)) %>%
  ungroup()

ggplot(summary,
       aes(x = reorder(Country, Number_year),
           y = Number_year)) +
  geom_bar(stat = "Identity") +
  geom_text(aes(label = paste(Proportion, "%", sep = "")), 
            hjust = -.3,
            fontface = "bold",
            size = 7,
            #vjust = -.2,
            position = position_dodge(width = 1)) +
  theme_economist(base_size = 30, base_family = "sans") +
  theme(axis.title = element_text()) +
  ylim(0, 60) +
  labs(x = "Country\n",
       y = "Number of recipients\n") +
  coord_flip()

### Number of recipients by institution

```

## Network of recipients and doctoral advisors

The data was organized in two files, **links and attributes** data. The **links data** file contained two columns
of the laurates and their doctoral advisors. A link is defined from the laurate to the doctoral adivisor. A sample of the first 5 cases
of the links table is as below. The **attributes data** file has details about all the individuals in the network. The key variable defined in this data file is a distinction of whether an individual is a nobel laurate or not. 
```{r process_data}

names(econ_net) <- str_replace(names(econ_net),
                               pattern = "_",
                               replacement = "")

econ_net <- cSplit(econ_net, "Doctoraladvisor", sep = ";")

#create id for each winner
#create time variable
econ_net <- plyr::ddply(econ_net,
                         plyr::.(Year),
                         mutate,
                         lid = 1:length(Year))

econ_net <- econ_net %>%
  mutate(lid = paste(Year, lid, sep = "")) %>%
  dplyr::select(lid, Year, Laureate, 
                starts_with("Doctoral"))

econ_net <- reshape(econ_net, 
                      varying=c(names(econ_net)[4:ncol(econ_net)]), 
                      direction="long", 
                      idvar=1:3, 
                      sep="_",
                      timevar = "rid")

row.names(econ_net) <- NULL

econ_net <- econ_net %>%
  filter(!is.na(Doctoraladvisor))


```



```{r network data}

econ_net <- econ_net %>%
  mutate(Laureate = as.character(Laureate),
         Doctoraladvisor = as.character(Doctoraladvisor),
         Laureate = str_trim(Laureate),
         Doctoraladvisor = str_trim(Doctoraladvisor)) %>%
  mutate(Doctoraladvisor = ifelse(Doctoraladvisor == "Thomas Schelling", "Thomas C. Schelling", 
                                  ifelse(Doctoraladvisor == "Eugene Fama", "Eugene F. Fama", Doctoraladvisor))) 


data_ties <- econ_net %>%
  dplyr::select(Laureate, Doctoraladvisor)

data_nodes_from <- econ_net %>%
  dplyr::select(Laureate) %>%
  data.frame()

data_nodes_to <- econ_net %>%
  dplyr::select(Doctoraladvisor) %>%
  data.frame()

names(data_nodes_from) <- c("name")
names(data_nodes_to) <- c("name")
  
data_nodes <- rbind(data_nodes_from,
                    data_nodes_to)

data_nodes <- data_nodes %>%
  mutate(name = str_trim(name),
         name = as.character(name)) %>%
  arrange(name)

data_nodes <- data_nodes %>%
  distinct(name)

data_lau <- econ_net %>%
  dplyr::select(Laureate) %>%
  mutate(Group = rep("Laureate")) %>%
  mutate(Laureate = str_trim(Laureate),
         Laureate = as.character(Laureate)) %>%
  arrange(Laureate)

data_nodes <- merge(data_nodes,
                    data_lau,
                    by.x = "name",
                    by.y = "Laureate",
                    all.x = T)

data_nodes <- data_nodes %>%
  distinct(name, Group) %>%
  mutate(Group = ifelse(is.na(Group), "Non-Laureate", Group))

knitr::kable(data_ties[1:5, c(1:2)])

```


### Complete network

The graph below shows the interaction between Nobel laurates in economic sciences with their doctoral advisors. Recipients of the award are colored in **red** while non-recipients are colored in **blue**. The arrows point to the doctoral adivor of the recipient. From the graph, it is notable that some doctoral students of earlier recipients also were awarded the nobel prize. Key among the doctoral advisors whose doctoral students also won the award later on are; **Wassily Leontief**,  **Robert Solow** and **Kenneth Arrow**.
```{r complete_graph, fig.height=18, fig.width=24}


graph_adv <- graph.data.frame(data_ties,
                              vertices = data_nodes,
                              directed = T)


# ggraph(graph_adv, 
#        layout = 'igraph',
#        algorith = 'nicely') + 
#     geom_edge_link(arrow = arrow(length = unit(4, 'mm')), 
#                    end_cap = circle(4, 'mm')#, 
#                    #aes(width = snvalue)
#                    ) +  
#     geom_node_point(aes(colour = Group), size = 7) + 
#     geom_node_text(aes(label = name), nudge_y = .3, size = 4) +
#     #theme_graph(base_size = 45, base_family = "sans") +
#     theme_bw(base_size = 30, base_family = "sans") +
#     theme(axis.ticks = element_blank(),
#           axis.title = element_blank(),
#           axis.text = element_blank(),
#           #panel.border = element_blank(),
#           legend.position = "bottom") +
#     guides(color = guide_legend(nrow = 2,
#                                 size = 3),
#            width = guide_legend(nrow = 3)) +
#     labs(colour = "Category",
#          title = "Network of Economics Laureates and their Doctoral advisors")

graph_adv_net <- intergraph::asNetwork(graph_adv)

ggnet2(graph_adv_net,
      node.color = "Group",
      palette = "Set1",
      node.size = 7,
      legend.size = 15,
      label = T,
      arrow.size = 12,
      arrow.gap = 0.025)

ggsave("~/Documents/RProjects/PMSite/Nobel_laureates_dadvisors.png",
       height = 18,
       width = 24)


```

### Largest connected component

The largest connected component of the complete network above was extracted and presented below. 18 of the 25 (**72%**), individuals in the connected component are all recipients of the award!! From **Wassiley Leontief**, there are two generations of recipients where some of his doctoral students (**Robert Solow, Paul Samuelson and Thomas C Schelling**) won the award and so did some of their students.
```{r largest_comp, fig.height=18, fig.width=24}

#largest component

graph_combs <- decompose.graph(graph_adv)

V(graph_adv)$label <- seq(vcount(graph_adv))
graphs_combs <- decompose.graph(graph_adv)

#extract the largest connected component
largest <- which.max(sapply(graphs_combs, vcount))

# plot(graphs_combs[[largest]], 
#      layout=layout.fruchterman.reingold)

graph_com_lar <- graphs_combs[[largest]]

ggnet2(graph_com_lar,
      node.color = "Group",
      palette = "Set1",
      node.size = 15,
      label = T,
      legend.size = 15,
      label.size = 8,
      arrow.size = 15,
      arrow.gap = 0.025)

ggsave("~/Documents/RProjects/PMSite/Nobel_laureates_dadvisors_largest_comb.png",
       height = 18,
       width = 24)


```



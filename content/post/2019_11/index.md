---
title: 'The Art of Thinking Clearly Chapters: Network Perspective'
summary: Looking at links between the chapters discussed in The Art of Think Clearly by Rolf Dobelli
authors:
- admin
date: '2019-11-01'
tags:
- rmarkdown
- books
- tidyverse
- networks
categories: R
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
import_library("readxl")
library(kableExtra)

```


Recently I had the pleasure of reading [**The Art of Thinking Clearly**](https://www.amazon.com/Art-Thinking-Clearly-Rolf-Dobelli/dp/0062219693) by Rolf Dobelli. This book discusses 99 "thinking errors". It is a great list of related thinking errors that we commit on a routine basis. The author discusses one cognitive error per chapter and gives a list of related chapters at the end of it.  

I put together the list of chapters and data on related chapters. Below, I explore the two files treating them as  linked/network data. The chapters list forms attributes data while data on relations between chapters forms links data. 


```{r import_data}

tatc_nodes <- read_xlsx("~/Documents/RProjects/PMSite/data/tatc.xlsx",
                        sheet = "chapters")

tatc_ties <- read_xlsx("~/Documents/RProjects/PMSite/data/tatc.xlsx",
                        sheet = "chapters_links")

```


```{r create_graph}

tatc_graph <- graph_from_data_frame(tatc_ties,
                                    vertices = tatc_nodes,
                                    directed = F)


tatc_cm <- data.frame(links_tot = degree(tatc_graph),
                      bet = betweenness(tatc_graph)) 

tatc_cm <- tatc_cm %>%
  mutate(chapter_id = row.names(tatc_cm)) %>%
  mutate(chapter_id = as.numeric(chapter_id))

tatc_nodes <- tatc_nodes %>%
  left_join(tatc_cm,
            by = "chapter_id")

```

Below is the top 10 chapters according to the number of total links recorded. Chapter 13, **Story bias** and Chapter 45, **Self-serving bias** are recorded as chapters with the highest number of links to other chapters. **Story bias** is the aspect where we try to "fit" or create an understanding of issues/events afterwards. Using words of the author, **Self-serving bias** may simply be explained as below:

> ".. we attribute success to ourselves and failures to external factors."


```{r top_10}

top10 <- tatc_nodes %>%
  arrange(desc(links_tot)) %>%
  slice_head(n = 10) %>%
  dplyr::select(-bet)


knitr::kable(top10,
             format = "html",
      digits = 2,
      caption = "Table 1. Top 10 chapters") %>%
  kable_styling(bootstrap_options = "striped", 
                full_width = F,
                position = "center",
                font_size = "11")


```

The graph below shows links between chapters. Points show chapters while the arrows/links are present when there is a link between two chapters. Points are sized according to the number of links reported for the respective point. Chapters with the largest point size are as presented in **Table 1** above. Generally, the graph follows the pattern of complex pattern rather than centralized or decentralized pattern. Theoretically, one may trace to any other chapter from any given point.


```{r tatc_graph}


# plot graph
ggraph(tatc_graph,
       layout = 'igraph',
       algorith = 'nicely') +
    geom_edge_link(arrow = arrow(length = unit(4, 'mm')),
                   end_cap = circle(4, 'mm')#,
                   #aes(width = snvalue)
                   ) +
    geom_node_point(colour = "red", size = degree(tatc_graph)) +
    geom_node_text(aes(label = name), nudge_y = .0, size = 7) +
    # theme_graph(base_size = 45, base_family = "sans") +
    theme_bw(base_size = 30, base_family = "sans") +
    theme(axis.ticks = element_blank(),
          axis.title = element_blank(),
          axis.text = element_blank(),
          #panel.border = element_blank(),
          legend.position = "bottom",
          plot.caption = element_text(size = 15, face = "bold")) +
    guides(color = guide_legend(nrow = 2,
                                size = 3),
           width = guide_legend(nrow = 3)) +
    labs(colour = "Category",
         title = "The Art of Thinking Clearly: Network of chapters",
         footnote = "admit",
         caption = "PM\n Data source: TATC Book")


```

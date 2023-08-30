---
title: 'Milestone project'
output: 
  #pdf_document: default
  html_document:
    keep_md: yes
always_allow_html: yes
---


Importing the required libraries

```r
library(stringi)
library(kableExtra)
library(dplyr)
library(ngram)
library(plotly)
```
Importing the data

```r
blogs <- readLines("./final/en_US/en_US.blogs.txt")
news <- readLines("./final/en_US/en_US.news.txt", skipNul = T, warn = F)
twitter <- readLines("./final/en_US/en_US.twitter.txt", skipNul = T, warn = F)
```
Showing some general information about the files

```r
gen_stats <- data.frame(Size = sapply(list(blogs, news, twitter), function(x) {format(object.size(x), "MB")}),
                                    t(rbind(sapply(list(blogs, news, twitter), stri_stats_general))), 
                                    t(rbind(sapply(list(blogs, news, twitter), stri_stats_latex))))

rownames(gen_stats) <- c("Blogs", "News", "Twitter")

kable(gen_stats, 'html') %>% kable_styling(full_width = F)
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> Size </th>
   <th style="text-align:right;"> Lines </th>
   <th style="text-align:right;"> LinesNEmpty </th>
   <th style="text-align:right;"> Chars </th>
   <th style="text-align:right;"> CharsNWhite </th>
   <th style="text-align:right;"> CharsWord </th>
   <th style="text-align:right;"> CharsCmdEnvir </th>
   <th style="text-align:right;"> CharsWhite </th>
   <th style="text-align:right;"> Words </th>
   <th style="text-align:right;"> Cmds </th>
   <th style="text-align:right;"> Envirs </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Blogs </td>
   <td style="text-align:left;"> 255.4 Mb </td>
   <td style="text-align:right;"> 899288 </td>
   <td style="text-align:right;"> 899288 </td>
   <td style="text-align:right;"> 206824382 </td>
   <td style="text-align:right;"> 170389539 </td>
   <td style="text-align:right;"> 162464653 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 42636700 </td>
   <td style="text-align:right;"> 37570839 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> News </td>
   <td style="text-align:left;"> 19.8 Mb </td>
   <td style="text-align:right;"> 77259 </td>
   <td style="text-align:right;"> 77259 </td>
   <td style="text-align:right;"> 15639408 </td>
   <td style="text-align:right;"> 13072698 </td>
   <td style="text-align:right;"> 12476453 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 3096618 </td>
   <td style="text-align:right;"> 2651432 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Twitter </td>
   <td style="text-align:left;"> 319 Mb </td>
   <td style="text-align:right;"> 2360148 </td>
   <td style="text-align:right;"> 2360148 </td>
   <td style="text-align:right;"> 162096241 </td>
   <td style="text-align:right;"> 134082806 </td>
   <td style="text-align:right;"> 125570778 </td>
   <td style="text-align:right;"> 3032 </td>
   <td style="text-align:right;"> 35958529 </td>
   <td style="text-align:right;"> 30451170 </td>
   <td style="text-align:right;"> 963 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>
Getting a sample from the data

```r
set.seed(123456)
n <- 1000
blog_sample <- sample(blogs, n)
news_sample <- sample(news, n)
twitter_sample <- sample(twitter, n)
sample_data <- c(blog_sample, news_sample, twitter_sample)
```
Showing some general information about the sample

```r
sample_stats <- data.frame(Size = sapply(list(blog_sample, news_sample, twitter_sample, sample_data), function(x) {format(object.size(x), "MB")}),
                            t(rbind(sapply(list(blog_sample, news_sample, twitter_sample, sample_data), stri_stats_general))), 
                            t(rbind(sapply(list(blog_sample, news_sample, twitter_sample, sample_data), stri_stats_latex))))
rownames(sample_stats) <- c("blog_sample", "news_sample", "twitter_sample", "sample_data")

kable(sample_stats, 'html') %>% kable_styling(full_width = F)
```

<table class="table" style="width: auto !important; margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> Size </th>
   <th style="text-align:right;"> Lines </th>
   <th style="text-align:right;"> LinesNEmpty </th>
   <th style="text-align:right;"> Chars </th>
   <th style="text-align:right;"> CharsNWhite </th>
   <th style="text-align:right;"> CharsWord </th>
   <th style="text-align:right;"> CharsCmdEnvir </th>
   <th style="text-align:right;"> CharsWhite </th>
   <th style="text-align:right;"> Words </th>
   <th style="text-align:right;"> Cmds </th>
   <th style="text-align:right;"> Envirs </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> blog_sample </td>
   <td style="text-align:left;"> 0.3 Mb </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 221129 </td>
   <td style="text-align:right;"> 182246 </td>
   <td style="text-align:right;"> 173542 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 45536 </td>
   <td style="text-align:right;"> 40075 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> news_sample </td>
   <td style="text-align:left;"> 0.3 Mb </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 197940 </td>
   <td style="text-align:right;"> 165341 </td>
   <td style="text-align:right;"> 158442 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 39318 </td>
   <td style="text-align:right;"> 33969 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> twitter_sample </td>
   <td style="text-align:left;"> 0.1 Mb </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 1000 </td>
   <td style="text-align:right;"> 66986 </td>
   <td style="text-align:right;"> 55358 </td>
   <td style="text-align:right;"> 51903 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 14848 </td>
   <td style="text-align:right;"> 12685 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sample_data </td>
   <td style="text-align:left;"> 0.7 Mb </td>
   <td style="text-align:right;"> 3000 </td>
   <td style="text-align:right;"> 3000 </td>
   <td style="text-align:right;"> 486055 </td>
   <td style="text-align:right;"> 402945 </td>
   <td style="text-align:right;"> 383887 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 99702 </td>
   <td style="text-align:right;"> 86729 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>
Removing all the punctuation marks

```r
sample_data <- lapply(sample_data, preprocess, remove.punct = T)
sample_data <- paste(sample_data, collapse = "")
```
Getting the first 4 n-grams for the sample

```r
unigram <- ngram(sample_data, n = 1) %>% get.phrasetable
bigrams <- ngram(sample_data, n = 2) %>% get.phrasetable
trigram <- ngram(sample_data, n = 3) %>% get.phrasetable
fourgram <- ngram(sample_data, n = 4) %>% get.phrasetable
```
Plotting bar charts for the first 4 n-grams

```r
plot_ly(unigram[1:20,], x = ~ngrams, y = ~freq, color = I("dodgerblue3")) %>%
  layout(xaxis = list(categoryorder = "total descending"))
```

```{=html}
<div class="plotly html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-a8c5ca27fd5529ee8edc" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-a8c5ca27fd5529ee8edc">{"x":{"visdat":{"6647abf6a1d":["function () ","plotlyVisDat"]},"cur_data":"6647abf6a1d","attrs":{"6647abf6a1d":{"x":{},"y":{},"color":["dodgerblue3"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20]}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"xaxis":{"domain":[0,1],"automargin":true,"categoryorder":"total descending","title":"ngrams","type":"category","categoryarray":["a ","and ","as ","at ","be ","for ","have ","i ","in ","is ","it ","my ","of ","on ","that ","the ","to ","was ","with ","you "]},"yaxis":{"domain":[0,1],"automargin":true,"title":"freq"},"hovermode":"closest","showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":["the ","to ","and ","a ","of ","in ","i ","that ","is ","for ","it ","on ","with ","you ","was ","my ","at ","as ","be ","have "],"y":[4156,2284,2132,2044,1855,1339,1029,911,862,845,663,629,603,595,575,461,450,429,428,405],"type":"bar","marker":{"color":"rgba(24,116,205,1)","line":{"color":"rgba(24,116,205,1)"}},"textfont":{"color":"rgba(24,116,205,1)"},"error_y":{"color":"rgba(24,116,205,1)"},"error_x":{"color":"rgba(24,116,205,1)"},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```

```r
plot_ly(bigrams[1:20,], x = ~ngrams, y = ~freq, color = I("darkgoldenrod2")) %>%
  layout(xaxis = list(categoryorder = "total descending"))
```

```{=html}
<div class="plotly html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-05a9b903079904d26dcd" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-05a9b903079904d26dcd">{"x":{"visdat":{"6644ab57812":["function () ","plotlyVisDat"]},"cur_data":"6644ab57812","attrs":{"6644ab57812":{"x":{},"y":{},"color":["darkgoldenrod2"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20]}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"xaxis":{"domain":[0,1],"automargin":true,"categoryorder":"total descending","title":"ngrams","type":"category","categoryarray":["and i ","and the ","at the ","for a ","for the ","from the ","going to ","i was ","in a ","in the ","is a ","it was ","of a ","of the ","on the ","to be ","to the ","will be ","with a ","with the "]},"yaxis":{"domain":[0,1],"automargin":true,"title":"freq"},"hovermode":"closest","showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":["of the ","in the ","to the ","for the ","on the ","to be ","in a ","at the ","and the ","with the ","from the ","for a ","is a ","of a ","it was ","with a ","i was ","will be ","going to ","and i "],"y":[399,365,189,170,153,128,118,117,113,87,85,84,83,82,78,74,72,65,65,64],"type":"bar","marker":{"color":"rgba(238,173,14,1)","line":{"color":"rgba(238,173,14,1)"}},"textfont":{"color":"rgba(238,173,14,1)"},"error_y":{"color":"rgba(238,173,14,1)"},"error_x":{"color":"rgba(238,173,14,1)"},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```

```r
plot_ly(trigram[1:20,], x = ~ngrams, y = ~freq, color = I("coral2")) %>%
  layout(xaxis = list(categoryorder = "total descending"))
```

```{=html}
<div class="plotly html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-562e7f7eabb96392a437" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-562e7f7eabb96392a437">{"x":{"visdat":{"664427d5e9c":["function () ","plotlyVisDat"]},"cur_data":"664427d5e9c","attrs":{"664427d5e9c":{"x":{},"y":{},"color":["coral2"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20]}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"xaxis":{"domain":[0,1],"automargin":true,"categoryorder":"total descending","title":"ngrams","type":"category","categoryarray":["a couple of ","a lot of ","according to the ","as well as ","be able to ","for the first ","going to be ","is one of ","it was a ","one of the ","out of the ","part of the ","some of the ","the end of ","the fact that ","the first time ","to be a ","to go to ","when i was ","you want to "]},"yaxis":{"domain":[0,1],"automargin":true,"title":"freq"},"hovermode":"closest","showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":["a lot of ","one of the ","to be a ","be able to ","as well as ","going to be ","a couple of ","according to the ","out of the ","some of the ","the end of ","the fact that ","part of the ","the first time ","it was a ","for the first ","to go to ","you want to ","is one of ","when i was "],"y":[35,30,19,17,16,15,15,14,14,12,12,11,11,11,11,11,11,10,10,10],"type":"bar","marker":{"color":"rgba(238,106,80,1)","line":{"color":"rgba(238,106,80,1)"}},"textfont":{"color":"rgba(238,106,80,1)"},"error_y":{"color":"rgba(238,106,80,1)"},"error_x":{"color":"rgba(238,106,80,1)"},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```

```r
plot_ly(fourgram[1:20,], x = ~ngrams, y = ~freq, color = I("navajowhite2")) %>%
  layout(xaxis = list(categoryorder = "total descending"))
```

```{=html}
<div class="plotly html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-402e04731b5b644caed7" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-402e04731b5b644caed7">{"x":{"visdat":{"66466e44ca":["function () ","plotlyVisDat"]},"cur_data":"66466e44ca","attrs":{"66466e44ca":{"x":{},"y":{},"color":["navajowhite2"],"alpha_stroke":1,"sizes":[10,100],"spans":[1,20]}},"layout":{"margin":{"b":40,"l":60,"t":25,"r":10},"xaxis":{"domain":[0,1],"automargin":true,"categoryorder":"total descending","title":"ngrams","type":"category","categoryarray":["as well as a ","at the end of ","dont know how to ","for the first time ","going to be a ","hunter matt hunter matt ","in the midst of ","is going to be ","is one of the ","matt hunter matt hunter ","oem acura integra wheels ","one of the most ","the back of the ","the end of the ","the oem acura integra ","to go to the ","to the point where ","what you want to ","when it comes to ","will be able to "]},"yaxis":{"domain":[0,1],"automargin":true,"title":"freq"},"hovermode":"closest","showlegend":false},"source":"A","config":{"modeBarButtonsToAdd":["hoverclosest","hovercompare"],"showSendToCloud":false},"data":[{"x":["matt hunter matt hunter ","one of the most ","for the first time ","hunter matt hunter matt ","when it comes to ","at the end of ","is one of the ","the end of the ","the oem acura integra ","is going to be ","as well as a ","to go to the ","will be able to ","dont know how to ","going to be a ","in the midst of ","oem acura integra wheels ","the back of the ","to the point where ","what you want to "],"y":[8,8,8,8,6,5,5,5,4,4,4,4,4,4,4,4,4,3,3,3],"type":"bar","marker":{"color":"rgba(238,207,161,1)","line":{"color":"rgba(238,207,161,1)"}},"textfont":{"color":"rgba(238,207,161,1)"},"error_y":{"color":"rgba(238,207,161,1)"},"error_x":{"color":"rgba(238,207,161,1)"},"xaxis":"x","yaxis":"y","frame":null}],"highlight":{"on":"plotly_click","persistent":false,"dynamic":false,"selectize":false,"opacityDim":0.20000000000000001,"selected":{"opacity":1},"debounce":0},"shinyEvents":["plotly_hover","plotly_click","plotly_selected","plotly_relayout","plotly_brushed","plotly_brushing","plotly_clickannotation","plotly_doubleclick","plotly_deselect","plotly_afterplot","plotly_sunburstclick"],"base_url":"https://plot.ly"},"evals":[],"jsHooks":[]}</script>
```

## Conclusion
After visualizing the tables for the sample datas we can 
deduce that the most common n-gram has a very low frequency, this is,
even the number of times a 1-gram is written in the data is small compared to the number of words.

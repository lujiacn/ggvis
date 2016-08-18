# ggvis

[![Build Status](https://travis-ci.org/rstudio/ggvis.png?branch=master)](https://travis-ci.org/rstudio/ggvis)

The goal of ggvis is to make it easy to describe interactive web graphics in
R. It combines:

* a grammar of graphics from [ggplot2](http://github.com/hadley/ggplot2),
  
* reactive programming from [shiny](http://github.com/rstudio/shiny), and

* data transformation pipelines from [dplyr](http://github.com/hadley/dplyr).

ggvis graphics are rendered with [vega](https://github.com/trifacta/vega), so you can generate both raster graphics with [HTML5 canvas](http://diveintohtml5.info/canvas.html) and vector graphics with
[svg](http://en.wikipedia.org/wiki/Scalable_Vector_Graphics). ggvis is less flexible than raw [d3](http://d3js.org/) or vega, but is much more succinct and is tailored to the needs of exploratory data analysis.

If you find a bug, please file a minimal reproducible example at http://github.com/rstudio/ggvis/issues. If you're not sure if something is a bug, you'd like to discuss new features or have any other questions about ggvis, please join us on the mailing list: https://groups.google.com/group/ggvis.

## Installation 

Install the latest release version from CRAN with:

```R
install.packages("ggvis")
```

Install the latest development version with:

```R
# install.packages("devtools")
devtools::install_github("hadley/lazyeval", build_vignettes = FALSE)
devtools::install_github("hadley/dplyr", build_vignettes = FALSE)
devtools::install_github("rstudio/ggvis", build_vignettes = FALSE)
```

## Getting started

You construct a visualisation by piping pieces together with `%>%`. The pipeline starts with a data set, flows into `ggvis()` to specify default visual properties, then layers on some visual elements:

```R
mtcars %>% ggvis(~mpg, ~wt) %>% layer_points()
```

The vignettes, available from http://ggvis.rstudio.com/, provide many more details. Start with the introduction, then work your way through the more advanced topics. Also check out the
various demos in the `demo/` directory. See the basics in `demo/scatterplot.r`
then check out the the coolest demos, `demo/interactive.r` and `demo/tourr.r`.

## Branch vega-2.x (github.com/lujiacn/ggvis) 
### Upgraded vega to version 2.6

### Enable tooltip on static page 
Added <mark>tooltip</mark> keywords. Example:
```r
devtools::install_github("lujiacn/ggvis", ref="vega-2.x")
library(ggvis)
library(dplyr)

mtcars %>% 
      mutate(tooltip = paste0("MPG: ", mpg, "</br>", "CYL: ", cyl)) %>%
      ggvis(~mpg, ~cyl) %>% 
      layer_points(fill := "steelblue", tooltip := ~tooltip)
```

### Pie chart

```
library(ggvis)
library(dplyr)
#-----sample data
data <- list(cat=c("A","B","C"), v = c(10,30,20))
data <- as.data.frame(data)
sum_n <- sum(data$v)
data_len <- nrow(data)

# calculate percentage, cat and v as variable
data <- data %>% mutate(percent=round(v/sum_n, 2)) %>%
  mutate(start_angle=NA, end_angle=NA) %>%
  mutate(tooltips=paste0(v, ", ", percent, "%"))

# calcuate start, end, mid angle
dt <- data %>% 
  mutate(cumsum = cumsum(percent)) %>%
  mutate(start_angle=ifelse(row_number()==1, 0, lag(cumsum))) %>%
  mutate(end_angle=cumsum) %>%
  mutate(mid_angle = start_angle + (end_angle-start_angle)/2)

canv_size <- 100
 dt %>% ggvis(x=~0, y=~0) %>% 
  scale_numeric("x",domain=c(-10,10)) %>%
  scale_numeric("y",domain=c(-10,10)) %>%
  layer_arcs(innerRadius:=0, outerRadius:=canv_size, 
             fill = ~cat, 
             stroke := "white",
             strokeWidth := 1,
             tooltip := ~tooltips,
             startAngle=~start_angle, endAngle=~end_angle) %>%
  layer_text( theta = prop("theta", ~mid_angle),
    radius := canv_size+10, text := ~cat,
    align := "center", baseline := "middle") %>%
  set_options(height=canv_size)
```

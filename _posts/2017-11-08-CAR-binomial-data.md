---
layout: post
title: "Simulating spatial data"
description: ""
category: [Teaching]
tags: [spatial,data,simulation,STAT615]
---
{% include JB/setup %}

The posts generates some spatial data on a lattice which can be used to evaluate
areal models or point-referenced models on a lattice.
The code is a modified version of the code in `?CARBayes::S.CARleroux`.


{% highlight r %}
library("MASS")
library("dplyr")
library("ggplot2")

set.seed(20171108)
sessionInfo()
{% endhighlight %}



{% highlight text %}
## R version 3.4.0 (2017-04-21)
## Platform: x86_64-redhat-linux-gnu (64-bit)
## Running under: Red Hat Enterprise Linux Workstation 7.3 (Maipo)
## 
## Matrix products: default
## BLAS/LAPACK: /usr/lib64/R/lib/libRblas.so
## 
## locale:
##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
##  [3] LC_TIME=en_US.UTF-8        LC_COLLATE=en_US.UTF-8    
##  [5] LC_MONETARY=en_US.UTF-8    LC_MESSAGES=en_US.UTF-8   
##  [7] LC_PAPER=en_US.UTF-8       LC_NAME=C                 
##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
## [11] LC_MEASUREMENT=en_US.UTF-8 LC_IDENTIFICATION=C       
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] knitr_1.17    bindrcpp_0.2  ggplot2_2.2.1 dplyr_0.7.4   CARBayes_5.0 
## [6] Rcpp_0.12.13  MASS_7.3-47  
## 
## loaded via a namespace (and not attached):
##  [1] spdep_0.6-13       CARBayesdata_2.0   plyr_1.8.4        
##  [4] compiler_3.4.0     bindr_0.1          LearnBayes_2.15   
##  [7] shapefiles_0.7     tools_3.4.0        digest_0.6.12     
## [10] boot_1.3-20        dotCall64_0.9-04   evaluate_0.10.1   
## [13] gtable_0.2.0       tibble_1.3.4       nlme_3.1-131      
## [16] lattice_0.20-35    pkgconfig_2.0.1    rlang_0.1.2       
## [19] Matrix_1.2-10      expm_0.999-2       SparseM_1.77      
## [22] spam_2.1-1         coda_0.19-1        stringr_1.2.0     
## [25] gtools_3.5.0       MatrixModels_0.4-1 grid_3.4.0        
## [28] glue_1.1.1         R6_2.2.2           foreign_0.8-69    
## [31] sp_1.2-5           gdata_2.18.0       deldir_0.1-14     
## [34] magrittr_1.5       scales_0.4.1       matrixcalc_1.0-3  
## [37] mcmc_0.9-5         gmodels_2.16.2     splines_3.4.0     
## [40] assertthat_0.2.0   colorspace_1.3-2   labeling_0.3      
## [43] quantreg_5.33      stringi_1.1.5      MCMCpack_1.4-0    
## [46] lazyeval_0.2.0     munsell_0.4.3      truncnorm_1.0-7
{% endhighlight %}

Construct spatial lattice.


{% highlight r %}
Grid <- expand.grid(x.easting = 1:10, x.northing = 1:10)
n <- nrow(Grid)
{% endhighlight %}

Build a neighborhood structure based on 4 nearest neighbors, 
i.e. cardinal directions.


{% highlight r %}
distance <- as.matrix(dist(Grid))

# Proximity matrix
W              <- matrix(0, nrow = n, ncol = n)
W[distance==1] <- 1       	
{% endhighlight %}

Simulate data


{% highlight r %}
# Explanatory variables and coefficients
x1 <- rnorm(n) %>% round(2)
x2 <- rnorm(n) %>% round(2)

# Spatial field
omega <- MASS::mvrnorm(n     = 1, 
                       mu    = rep(0,n), 
                       Sigma = 0.4 * exp(-0.1 * distance))

eta <- x1 + x2 + omega

d <- Grid %>% 
  mutate(Y_normal = rnorm(n, eta, sd = 0.1) %>% round(2),
         Y_pois   = rpois(n, exp(eta)),
         trials   = 10,
         Y_binom  = rbinom(n = n, size = trials, prob = 1/(1+exp(-eta))),
         x1       = x1,
         x2       = x2)
{% endhighlight %}

Normal data


{% highlight r %}
ggplot(d, aes(x = x.easting, y = x.northing, fill = Y_normal)) + 
  geom_raster() +
  theme_bw()
{% endhighlight %}

![center](/../figs/2017-11-08-CAR-binomial-data/normal_data-1.png)

Poisson data


{% highlight r %}
ggplot(d, aes(x = x.easting, y = x.northing, fill = Y_pois)) + 
  geom_raster() +
  theme_bw()
{% endhighlight %}

![center](/../figs/2017-11-08-CAR-binomial-data/poisson_data-1.png)

Binomial data


{% highlight r %}
ggplot(d, aes(x = x.easting, y = x.northing, fill = Y_binom/trials)) + 
  geom_raster() +
  theme_bw()
{% endhighlight %}

![center](/../figs/2017-11-08-CAR-binomial-data/binomial_data-1.png)

For use in future posts.


{% highlight r %}
dput(d)
{% endhighlight %}



{% highlight text %}
structure(list(x.easting = c(1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 
9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 
4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 
9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 
4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 
9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 
4L, 5L, 6L, 7L, 8L, 9L, 10L, 1L, 2L, 3L, 4L, 5L, 6L, 7L, 8L, 
9L, 10L), x.northing = c(1L, 1L, 1L, 1L, 1L, 1L, 1L, 1L, 1L, 
1L, 2L, 2L, 2L, 2L, 2L, 2L, 2L, 2L, 2L, 2L, 3L, 3L, 3L, 3L, 3L, 
3L, 3L, 3L, 3L, 3L, 4L, 4L, 4L, 4L, 4L, 4L, 4L, 4L, 4L, 4L, 5L, 
5L, 5L, 5L, 5L, 5L, 5L, 5L, 5L, 5L, 6L, 6L, 6L, 6L, 6L, 6L, 6L, 
6L, 6L, 6L, 7L, 7L, 7L, 7L, 7L, 7L, 7L, 7L, 7L, 7L, 8L, 8L, 8L, 
8L, 8L, 8L, 8L, 8L, 8L, 8L, 9L, 9L, 9L, 9L, 9L, 9L, 9L, 9L, 9L, 
9L, 10L, 10L, 10L, 10L, 10L, 10L, 10L, 10L, 10L, 10L), Y_normal = c(-1.37, 
1.81, 1.62, 0.65, 1.05, 0.4, -0.03, 0.76, -0.36, 0.86, 2.07, 
-0.05, 1.48, 1.63, 0.59, 0.36, 1.02, 1.44, 0.07, 2.65, 3.62, 
-1.79, 0.75, -1.87, 2.35, 1.13, -0.66, -0.17, 1.9, 0.57, 0.49, 
3.21, 0.38, 3.17, 0.16, 1.03, 1.02, 0.58, -1.32, 2.16, 0.68, 
-1.15, 0.14, 0.94, -0.92, 2.23, 1.9, -0.14, 1.13, 0.22, 0.7, 
3.49, 1.22, 2.31, -0.86, 0.38, 2.77, 1.87, 0.45, 0.95, 1.36, 
1.25, -0.7, 1.39, 1.26, -1.29, 0.51, -0.72, 0.31, 0.43, 0.35, 
-2.18, 1.72, 1.85, -0.3, 2.69, -0.02, 0.4, 0.59, 1.38, 0.13, 
3.08, 1.7, -0.41, -3.63, 2.22, 0.24, -0.24, 1.89, 0.88, 0.2, 
2.99, 0.76, 1.99, -0.19, 0.97, 2.93, -3.29, -0.41, 0.11), Y_pois = c(1L, 
5L, 5L, 2L, 1L, 2L, 0L, 2L, 0L, 2L, 10L, 0L, 5L, 3L, 4L, 2L, 
1L, 1L, 3L, 6L, 24L, 1L, 1L, 0L, 13L, 1L, 1L, 0L, 10L, 2L, 3L, 
18L, 0L, 25L, 0L, 8L, 6L, 4L, 0L, 5L, 1L, 0L, 2L, 4L, 0L, 10L, 
8L, 1L, 7L, 1L, 2L, 40L, 2L, 9L, 0L, 4L, 12L, 6L, 3L, 1L, 7L, 
3L, 0L, 6L, 6L, 1L, 3L, 0L, 0L, 0L, 1L, 0L, 4L, 4L, 1L, 14L, 
3L, 0L, 4L, 7L, 1L, 27L, 5L, 3L, 0L, 8L, 0L, 0L, 10L, 0L, 2L, 
23L, 2L, 6L, 1L, 3L, 16L, 0L, 0L, 1L), trials = c(10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 10, 
10), Y_binom = c(2L, 10L, 9L, 7L, 6L, 3L, 6L, 9L, 4L, 8L, 9L, 
8L, 7L, 9L, 8L, 7L, 7L, 6L, 4L, 9L, 10L, 3L, 6L, 0L, 10L, 8L, 
5L, 4L, 10L, 7L, 5L, 10L, 5L, 10L, 3L, 7L, 8L, 5L, 2L, 9L, 7L, 
2L, 5L, 7L, 1L, 10L, 10L, 6L, 8L, 8L, 8L, 10L, 7L, 10L, 4L, 4L, 
10L, 10L, 3L, 7L, 8L, 6L, 3L, 9L, 6L, 0L, 8L, 4L, 5L, 7L, 5L, 
0L, 10L, 9L, 3L, 10L, 4L, 6L, 7L, 8L, 7L, 9L, 10L, 3L, 0L, 10L, 
4L, 4L, 8L, 7L, 5L, 10L, 6L, 10L, 4L, 10L, 10L, 1L, 4L, 6L), 
    x1 = c(-2.01, 0.55, 0.1, 0.08, -0.54, 0.35, -0.7, 0.13, -0.45, 
    0.7, 0.31, 0.5, 0.44, 0.43, -0.46, -0.38, -0.53, -0.73, -0.58, 
    1.35, 2.22, -1.7, 0.32, -2.57, 2.47, 0.41, -1.41, 0.11, 1.51, 
    -1.05, 0.73, 1.18, 0.52, 1.42, -1.33, -0.94, 0.95, -0.77, 
    -1.03, 0.1, -0.27, -1.44, -0.45, 0.52, -0.5, 0.78, 0.66, 
    0.49, -0.16, -1.3, 1.76, 2.8, 0.87, 0.48, -0.18, 0.39, 0.04, 
    0.49, 0.06, 0.74, 0.28, 0.15, -0.22, 0.57, -0.39, -0.74, 
    -0.37, -1.34, -1.67, 0.81, 0.12, -0.69, 1.31, 0.77, -1.38, 
    1.19, -2.05, 0.96, 1.14, 0.89, 0.53, 2.39, 1.32, 0.95, -0.79, 
    -0.02, -0.08, -1.38, 0.35, 0.43, -0.14, 2.06, -1, 0.42, -0.38, 
    0.38, 0.93, -1.97, -0.5, 0.76), x2 = c(0, 0.38, 0.79, -0.14, 
    1.13, -0.84, 0.09, -0.09, -0.68, -1.12, 0.97, -1.2, 0.15, 
    0.4, 0.06, 0.01, 0.71, 1.38, -0.29, -0.06, 0.45, -0.54, -0.15, 
    -0.13, -0.89, 0.27, -0.09, -1.15, -0.45, 0.56, -0.72, 1.71, 
    -0.86, 1.02, 0.91, 1.65, -0.77, 0.61, -1.15, 1.04, 0.21, 
    -0.22, 0.3, -0.38, -0.48, 1.11, 0.62, -1.29, 0.41, 0.67, 
    -1.33, 0.45, -0.46, 1.33, -1.02, -0.74, 2.28, 1.11, 0.1, 
    -0.46, 0.72, 0.61, -0.91, 0.23, 0.99, -1.24, 0.53, 0.02, 
    1.71, -1.18, 0.07, -1.72, -0.35, 0.31, 0.4, 0.76, 1.22, -0.94, 
    -1.11, -0.07, -0.57, 0.45, -0.14, -1.85, -3.41, 1.4, -0.03, 
    0.64, 0.79, -0.36, -0.34, 0.39, 1.44, 0.97, -0.36, -0.19, 
    1.94, -1.56, -0.56, -1.41)), class = "data.frame", .Names = c("x.easting", 
"x.northing", "Y_normal", "Y_pois", "trials", "Y_binom", "x1", 
"x2"), row.names = c(NA, -100L))
{% endhighlight %}
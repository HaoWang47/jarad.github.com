---
layout: post
title: "Intrinsic CAR using CARBayes"
description: ""
category: [Teaching]
tags: [areal,spatial,STAT615]
---
{% include JB/setup %}


{% highlight r %}
library("CARBayes")

set.seed(20171109)
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
##  [1] spdep_0.6-13       CARBayesdata_2.0   highr_0.6         
##  [4] plyr_1.8.4         compiler_3.4.0     bindr_0.1         
##  [7] LearnBayes_2.15    shapefiles_0.7     tools_3.4.0       
## [10] digest_0.6.12      boot_1.3-20        dotCall64_0.9-04  
## [13] evaluate_0.10.1    gtable_0.2.0       tibble_1.3.4      
## [16] nlme_3.1-131       lattice_0.20-35    pkgconfig_2.0.1   
## [19] rlang_0.1.2        Matrix_1.2-10      expm_0.999-2      
## [22] SparseM_1.77       spam_2.1-1         coda_0.19-1       
## [25] stringr_1.2.0      gtools_3.5.0       MatrixModels_0.4-1
## [28] grid_3.4.0         glue_1.1.1         R6_2.2.2          
## [31] foreign_0.8-69     sp_1.2-5           gdata_2.18.0      
## [34] deldir_0.1-14      magrittr_1.5       scales_0.4.1      
## [37] matrixcalc_1.0-3   mcmc_0.9-5         gmodels_2.16.2    
## [40] splines_3.4.0      assertthat_0.2.0   colorspace_1.3-2  
## [43] labeling_0.3       quantreg_5.33      stringi_1.1.5     
## [46] MCMCpack_1.4-0     lazyeval_0.2.0     munsell_0.4.3     
## [49] truncnorm_1.0-7
{% endhighlight %}

Using the data from [this post](http://www.jarad.me/teaching/2017/11/08/CAR-binomial-data), 
we will utilize the intrinsic CAR prior to account for spatial association.



The code to read in the data is suppressed but the head of the data looks like


{% highlight r %}
head(d)
{% endhighlight %}



{% highlight text %}
##   x.easting x.northing Y_normal Y_pois trials Y_binom    x1    x2
## 1         1          1    -1.37      1     10       2 -2.01  0.00
## 2         2          1     1.81      5     10      10  0.55  0.38
## 3         3          1     1.62      5     10       9  0.10  0.79
## 4         4          1     0.65      2     10       7  0.08 -0.14
## 5         5          1     1.05      1     10       6 -0.54  1.13
## 6         6          1     0.40      2     10       3  0.35 -0.84
{% endhighlight %}

Build a neighborhood structure based on 4 nearest neighbors, 
i.e. cardinal directions.


{% highlight r %}
distance <- as.matrix(dist(d[,c("x.easting","x.northing")]))

# Proximity matrix
W              <- matrix(0, nrow = n, ncol = n)
W[distance==1] <- 1       	
{% endhighlight %}


{% highlight r %}
image(W)
{% endhighlight %}

![center](/../figs/2017-11-09-intrinsic-CAR-using-CARBayes/proximity_matrix-1.png)

## Normal model


{% highlight r %}
system.time(
  mn <- S.CARleroux(formula  = Y_normal ~ x1 + x2, 
                   family   = "gaussian", 
                   data     = d,
                   W        = W, 
                   fix.rho  = TRUE, rho = 1, # intrinsic CAR
                   burnin   = 20000, 
                   n.sample = 100000,
                   verbose  = FALSE)
)
{% endhighlight %}



{% highlight text %}
##    user  system elapsed 
##  29.257   0.129  29.380
{% endhighlight %}


{% highlight r %}
mn
{% endhighlight %}



{% highlight text %}
## 
## #################
## #### Model fitted
## #################
## Likelihood model - Gaussian (identity link function) 
## Random effects model - Leroux CAR
## Regression equation - Y_normal ~ x1 + x2
## Number of missing observations - 0
## 
## ############
## #### Results
## ############
## Posterior quantities and DIC
## 
##             Median   2.5%  97.5% n.sample % accept n.effective Geweke.diag
## (Intercept) 0.6222 0.5953 0.6488    80000      100     75928.7        -0.4
## x1          1.0162 0.9802 1.0530    80000      100     19371.2         0.7
## x2          0.9819 0.9423 1.0209    80000      100     19202.8        -1.2
## nu2         0.0168 0.0042 0.0341    80000      100      2079.1        -0.6
## tau2        0.0621 0.0245 0.1194    80000      100      2127.2         0.7
## rho         1.0000 1.0000 1.0000       NA       NA          NA          NA
## 
## DIC =  -63.54406       p.d =  59.70351       Percent deviance explained =  99.62
{% endhighlight %}



## Binomial model


{% highlight r %}
system.time(
  mb <- S.CARleroux(formula  = Y_binom ~ x1 + x2, 
                   family   = "binomial", 
                   data     = d,
                   trials   = d$trials,
                   W        = W, 
                   fix.rho  = TRUE, rho = 1, # intrinsic CAR
                   burnin   = 20000, 
                   n.sample = 100000,
                   verbose  = FALSE)
)
{% endhighlight %}



{% highlight text %}
##    user  system elapsed 
##  40.648   0.088  40.732
{% endhighlight %}


{% highlight r %}
mb
{% endhighlight %}



{% highlight text %}
## 
## #################
## #### Model fitted
## #################
## Likelihood model - Binomial (logit link function) 
## Random effects model - Leroux CAR
## Regression equation - Y_binom ~ x1 + x2
## Number of missing observations - 0
## 
## ############
## #### Results
## ############
## Posterior quantities and DIC
## 
##             Median   2.5%  97.5% n.sample % accept n.effective Geweke.diag
## (Intercept) 0.7036 0.5462 0.8648    80000     46.6     28172.6        -1.3
## x1          1.2821 1.0842 1.4967    80000     46.6     12485.4        -2.4
## x2          1.0849 0.8831 1.3002    80000     46.6     16405.8        -2.0
## tau2        0.0183 0.0030 0.1684    80000    100.0       360.6        -0.7
## rho         1.0000 1.0000 1.0000       NA       NA          NA          NA
## 
## DIC =  309.0988       p.d =  4.419879       Percent deviance explained =  51.44
{% endhighlight %}




## Poisson model


{% highlight r %}
system.time(
  mp <- S.CARleroux(formula  = Y_pois ~ x1 + x2, 
                   family   = "poisson", 
                   data     = d,
                   W        = W, 
                   fix.rho  = TRUE, rho = 1, # intrinsic CAR
                   burnin   = 20000, 
                   n.sample = 100000,
                   verbose  = FALSE)
)
{% endhighlight %}



{% highlight text %}
##    user  system elapsed 
##  31.887   0.020  31.903
{% endhighlight %}


{% highlight r %}
mp
{% endhighlight %}



{% highlight text %}
## 
## #################
## #### Model fitted
## #################
## Likelihood model - Poisson (log link function) 
## Random effects model - Leroux CAR
## Regression equation - Y_pois ~ x1 + x2
## Number of missing observations - 0
## 
## ############
## #### Results
## ############
## Posterior quantities and DIC
## 
##             Median   2.5%  97.5% n.sample % accept n.effective Geweke.diag
## (Intercept) 0.7773 0.6201 0.9248    80000     45.9      4998.7        -1.6
## x1          0.9260 0.8312 1.0236    80000     45.9      5518.6         2.7
## x2          0.7736 0.6561 0.8923    80000     45.9      7710.8         0.7
## tau2        0.0097 0.0025 0.0595    80000    100.0       334.0         0.7
## rho         1.0000 1.0000 1.0000       NA       NA          NA          NA
## 
## DIC =  374.7862       p.d =  5.034638       Percent deviance explained =  61.12
{% endhighlight %}
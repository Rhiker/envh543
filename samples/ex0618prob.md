# 1D Monte Carlo simulation of microbial exposure
Jane Pouzou  
License: CC BY-SA 4.0  

This document offers a 1D Monte Carlo probabilistic solution in R for the 
daily microbial exposure from drinking water consumption, swimming in surface 
water and shellfish consumption for 
[Example 6.18](https://books.google.com/books?id=ejTKAwAAQBAJ&lpg=PT265&ots=6q9TSELe3B&pg=PT265#v=onepage&q&f=false) from pages 215-216 of:

[Quantitative Microbial Risk Assessment, 2nd Edition](http://www.wiley.com/WileyCDA/WileyTitle/productCd-1118145291,subjectCd-CH20.html) 
by Charles N. Haas, Joan B. Rose, and Charles P. Gerba. (Wiley, 2014).


```r
# A 1D Monte Carlo simulation for the daily microbial exposure from drinking
# water consumption, swimming in surface water, and shellfish consumption for 
# Example 6.18 from Quantitative Microbial Risk Assessment, 2nd Edition by 
# Charles N. Haas, Joan B. Rose, and Charles P. Gerba. (Wiley, 2014).

# Copyright (c) Jane Pouzou
# License: CC BY-SA 4.0 - https://creativecommons.org/licenses/by-sa/4.0/

# Define additional variables provided in the example for three exposure types.
#
# Shellfish consumption
shell.viral.load <- 1
shell.cons <- 9e-4 * 150  # 9e-4 Person-Days per year * 150 g per occurrence
# Drinking water consumption
dw.viral.load <- 0.001
# Surface water consumption while swimming
sw.viral.load <- 0.01
sw.daily.IR <- 50         # Ingestion rate in mL of surface water
sw.frequency <- 7         # Exposure frequency of 7 swims per year

# Generate 5000 random values from a log-normal distribution to estimate 
# exposure from consumption of drinking water (ml/day). Divide by 1000 mL/L 
# to get consumption in liters/day.  Values for meanlog and sdlog are from the 
# QMRA textbook (Haas, 2014), page 216, Table 6.30.
set.seed(1)
water.cons.L <- rlnorm(5000, meanlog = 7.49, sdlog = 0.407) / 1000

# Plot the kernal density curve of the generated values just as a check.
plot(density(water.cons.L))
```

![](ex0618prob_files/figure-html/unnamed-chunk-1-1.png)

```r
# Sample 5000 times from a discrete distribution of swim duration with 
# assigned probabilities of each outcome. These values are hypothetical and
# are not found in the text, but are defined here to provide an example of
# sampling from a discrete distribution.
set.seed(1)
swim.duration <- sample(x = c(0.5, 1, 2, 2.6), 5000, replace = TRUE, 
                        prob = c(0.1, 0.1, 0.2, 0.6))

# Create a simple histogram of our distribution as a check.
hist(swim.duration)
```

![](ex0618prob_files/figure-html/unnamed-chunk-1-2.png)

```r
# Define a function to calculate microbial exposure risk.
Risk.fcn <- function(shell.vl, shell.cons, water.cons.L, dw.vl, sw.vl, 
                     sw.daily.IR, sw.duration, sw.frequency) {
    ((shell.vl * shell.cons) + (water.cons.L * dw.vl) + 
         ((sw.vl * (sw.daily.IR * sw.duration * sw.frequency)) / 365 / 1000))
}

# Run the simulation 5000 times.
daily.dose <- sapply(1:5000, 
                     function(j) Risk.fcn(water.cons.L = water.cons.L[j], 
                                          sw.duration = swim.duration[j], 
                                          shell.vl = shell.viral.load, 
                                          dw.vl = dw.viral.load, 
                                          shell.cons = shell.cons, 
                                          sw.vl = sw.viral.load, 
                                          sw.daily.IR = sw.daily.IR, 
                                          sw.frequency = sw.frequency))

# Plot the results of the simulation.
plot(density(daily.dose))
```

![](ex0618prob_files/figure-html/unnamed-chunk-1-3.png)
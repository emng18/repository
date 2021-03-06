Bootstrap + Permutation Test
================
Elliot Ng
November 2, 2019

Using/loading the Cars93 dataset:

``` r
library(MASS)
```

    ## Warning: package 'MASS' was built under R version 3.5.3

``` r
data("Cars93")
```

Histogram, QQplot and summary statistics:

``` r
summary(Cars93$Max.Price)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##     7.9    14.7    19.6    21.9    25.3    80.0

``` r
sd(Cars93$Max.Price)
```

    ## [1] 11.03046

``` r
par(mfrow=c(1,2))


hist(Cars93$Max.Price)
qqnorm(Cars93$Max.Price)
qqline(Cars93$Max.Price)
```

![](bootstrap_permutation_github_files/figure-markdown_github/unnamed-chunk-2-1.png)

The shape of the histogram is skewed right. This is confirmed by the normal qq plot, as there are many deviations from normality on the right side of the plot.

We can use the bootstrap method to estimate a parameter (i.e standard deviation) of given data. We keep taking random samples of the data (with replacement), compute standard deviation, and store it in a vector. Bootstrap method using 5000 replicates (for the standard deviation of Max.Price):

``` r
set.seed(9999)
n <- length(Cars93$Max.Price); n
```

    ## [1] 93

``` r
## [1] 93
replicates <- rep(0, 5000)
for(i in 1:5000) {
boot_samp <- sample(Cars93$Max.Price, size = n, replace = TRUE)
replicates[i] <- sd(boot_samp)
}
# bootstrap distribution
par(mfrow=c(1,2))
hist(replicates, xlab="Replicates of sample standard deviation", main='')
abline(v=sd(Cars93$Max.Price), col="cyan", lwd=1.5)
qqnorm(replicates)
qqline(replicates)
```

![](bootstrap_permutation_github_files/figure-markdown_github/unnamed-chunk-3-1.png)

95% bootstrap percentile confidence interval for the population standard deviation:

``` r
quantile(replicates, c(0.025, 0.975))
```

    ##      2.5%     97.5% 
    ##  8.058214 14.390267

We are 95% confident that the population standard deviation of max price for cars (on sale in the USA in 1993) is between 8.058 and 14.39 thousands of dollars.

Comparing to a 95% confidence interval for the population standard deviation using the formula-based method, with the critical values from the chi-square distribution:

``` r
s=sd(Cars93$Max.Price)
n=length(Cars93$Max.Price)

ci_upper <- sqrt((n-1)*s^2  / (qchisq(.025,df=92)))
ci_lower <- sqrt((n-1)*s^2  / (qchisq(1-.025,df=92)))
round(c(ci_lower, ci_upper), 2)
```

    ## [1]  9.64 12.89

The two intervals in c and d are only somewhat close -- there is about a 10%/16% difference between their upper/lower bounds. This leads us to believe that the two intervals do not quite agree with each other.

The bootstrap method is more preferrable with this set of data, because the formula-based method should only be used when the sample comes from a normal population distribution (regardless of sample size n). From the histogram of the sample data, we know that the sample data is not normally distributed (it is skewed right). With no information on whether the population (max car price for 1993 USA cars) is normal or not, the condition for the formula-based method may not have been met. With the bootstrap method, we do not need to make assumptions on normality.

Permuatation test comparison with T-test:

side-by-side box plot of MPG.city versus Origin:

``` r
boxplot(MPG.city ~ Origin, data=Cars93, ylab="MPG (city)")
```

![](bootstrap_permutation_github_files/figure-markdown_github/unnamed-chunk-6-1.png)

``` r
a=subset(Cars93,Origin=="USA")
#hist(a$MPG.city)
```

We Use t.test() to test whether the average city miles per gallon (mpg) of cars manufactured by USA companies is significantly different than cars manufactured by non-USA companies. The hypotheses are:

Ho: The mean city mpg of cars manufactured by USA companies is the same as that of cars manufactured by non-USA companies (u1=u2, or u1-u2 = 0)

HA: The mean city mpg of cars manufactured by USA companies is different than that of cars manufactured by non-USA companies (u1=/=u2, or u1-u2=/=0)

Assumptions:

Data for each group is approximately normal: This is somewhat satisfied by looking at the boxplots above. The USA group is somewhat normal, whereas the non-USA group is right-skewed. However, since n&gt;=30 for both groups, CLT applies. Therefore, the normality condition is satisfied.

Random/Independent: The data from both groups came from a random sample, therefore they are random and independent.

``` r
USA<- subset(Cars93,Origin=="USA")
nonUSA <- subset(Cars93,Origin=="non-USA")

#mean(USA$MPG.city)
#sd(USA$MPG.city)
#mean(nonUSA$MPG.city)
#sd(nonUSA$MPG.city)
t.test(USA$MPG.city, nonUSA$MPG.city)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  USA$MPG.city and nonUSA$MPG.city
    ## t = -2.5296, df = 71.024, p-value = 0.01364
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -5.2008385 -0.6158282
    ## sample estimates:
    ## mean of x mean of y 
    ##  20.95833  23.86667

From the t-test, we get a p-value of 0.013, which is statistically significant (at alpha = 0.05 or 0.1). Therefore, we reject Ho in favor of HA. We have evidence that suggests that the mean city mpg of cars manufactured by USA companies is different than that of cars manufactured by non-USA companies.

Permutation test: We can use the permutation test to test whether the average city miles per gallon (mpg) of cars manufactured by USA companies is significantly different than cars manufactured by non-USA

Null/Alternative Hypotheses:

Ho: The mean city mpg of cars manufactured by USA companies is the same as that of cars manufactured by non-USA companies (u1=u2, or u1-u2 = 0)

HA: The mean city mpg of cars manufactured by USA companies is different than that of cars manufactured by non-USA companies (u1=/=u2, or u1-u2=/=0)

``` r
ccars93 <- na.omit(Cars93[, c("MPG.city", "Origin")])
origin=ccars93$Origin
citympg=ccars93$MPG.city
```

``` r
set.seed(999)
results <- c()
for(i in 1:10000) {
# permute treatment labels
origin0 <- sample(origin)

xbar_esc0 <- mean(citympg[origin0 == "USA"])

xbar_ctrl0 <- mean(citympg[origin0 == "non-USA"])
# take difference in means after permuting labels
results[i] = xbar_esc0 - xbar_ctrl0
}
```

Permutation Distribution (ablines are set at the difference of means of USA and non-USA MPG in cars):

``` r
hist(results, xlab="Difference in sample means", main="Permutation distribution")
# compare with observed difference in means
obsv_result <- mean(citympg[origin == "USA"]) - mean(citympg[origin == "non-USA"])
abline(v=obsv_result, col="blue", lwd=2)
abline(v=-obsv_result, col="blue", lwd=2)
```

![](bootstrap_permutation_github_files/figure-markdown_github/unnamed-chunk-10-1.png)

P-value calculation: We can calculate the p-value by computing the probabilities below the first abline and above the second abline.

``` r
pval <- sum(results < obsv_result ) / 10000;
pval2 <- sum(results > -obsv_result ) / 10000;
psum=pval+pval2; psum
```

    ## [1] 0.0107

We get a p-value of 0.0107, which is close to the 0.0136 from the t-test. Since 0.0107 is less than 0.05 (standard confidence level), we reject Ho in favor of HA. We have evidence that suggests that the mean city mpg of cars manufactured by USA companies is different than that of cars manufactured by non-USA companies (same conclusion as the t-test).

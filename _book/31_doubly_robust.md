# Doubly Robust Estimator

Without random assignment, we can either use

|                    | Inverse propensity weighting estimator                                                                                                                                                                                                    | linear regression                                                                                     |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| Intuition          | (based on a logistic regression of the likelihood of receiving the treatment). In words, we try to reweight each observation based on its probability of receiving the treatment to make your sample as close to a randomized experiment. | linear regression with some variables and treatment assignment variable to model the outcome directly |
| Leverage           | Logistic Regression                                                                                                                                                                                                                       | Linear Regression                                                                                     |
| When does it fail? | When the propensity score model is incorrectly specified (e.g., missing variables)                                                                                                                                                        | When the outcome model is incorrectly specified (e.g., missing variables)                             |

But in both cases if you miss some variables (i.e., omitted variable bias) will bias your estimates.

The solution is to combine both approaches to form doubly robust estimator.

In this case you only need one of them to work, not both (known as the doubly robustness property)

To read a more technical introduction, see [@funk2011] and [@lunceford2017]

Extension include:

-   Panel data: [@arkhangelsky2021]

-   Instrumental variable: [@okui2012]

-   Dif-n-dif: [@santanna2020]

## Regression Adjustments

With the unconfoundedness assumption (i.e., potential outcomes given some characteristics are independent of the treatment)

$$
[\{Y_i(0)), Y_i(1)\} \perp T_i] | X_i
$$

then the [Average Treatment Effects] will be

$$
\begin{aligned}
\tau &= E[Y_i(1) - Y_i(0)] \\
&= E[E[Y_i(1) |X_i - E[Y_i(0)|X_i]] \\
&= E[E[Y_i |X_i, T= 1] - E[Y_i|X_i, T_1 = 0]] && \text{unconfoundedness}\\
&= E[\mu_1(X_i) - \mu_0(X_i)]
\end{aligned}
$$

With **linearity assumption**, we can use OLS to estimate $\mu_1(X_i)$ and $\mu_0(X_i)$ by

$$
Y_i = \mathbf{\beta}_0 X_i \text{ subset } T_i = 0 \\
Y_i = \mathbf{\beta}_1 X_i \text{ subset } T_i = 1
$$

and the estimate of $\tau$ becomes

$$
\begin{aligned}
\hat{\tau} &= \frac{1}{n} \sum_{i=1}^n (\hat{\mu}_1 (X_i) - \hat{\mu}_0(X_i)) \\
&= (\hat{\beta}_1 - \hat{\beta}_0 ) \bar{X}
\end{aligned}
$$

where $\bar{X} = \frac{\sum_{i=1}^n X_i}{n}$ (including an intercept)

Alternatively, to avoid the **linearity assumption**, we can use non-parametric approach (i.e., machine learning) to estimate $\mu_1(X_i)$ and $\mu_0(X_i)$

**Simulation**

Create functions to generate linear, nonlinear and experimental (random) data


```r
# linear setting
simu_ob_linear_df <- function(n){
    x1 = rnorm(n)
    x2 = rnorm(n)
    xb = 0.5 * x1 + 0.5 * x2 
    # probability of treatment is dependent on x1 and x2
    prob_treatment <- exp(xb) / (1 + exp(xb))
    treatment <- rbinom(n, 1, prob_treatment)
    
    outcome <- 10 + x1 + x2 + 5 * treatment + rnorm(n)
    data.frame(x1, x2, prob_treatment,  treatment, outcome)
}

# experimental setting (random treatment assignment)
simu_exp_df <- function(n){
    x1 = rnorm(n)
    x2 = rnorm(n)
    xb = 0.5 * x1 + 0.5 * x2 
    # probability of treatment is not dependent on x1 and x2
    treatment <- rbinom(n, 1, 0.5) # prob treatment = random = 0.5
    
    outcome <- 10 + x1 + x2 + 5 * treatment + rnorm(n)
    data.frame(x1, x2, treatment, outcome)
}

# non-linear setting
simu_exp_nonlinear_df <- function(n){
    x1 = rnorm(n)
    x2 = rnorm(n)
    xb = 0.5 * x1 + 0.5 * x2 
    # probability of treatment is  dependent on x1 and x2 nonlinearly
    prob_treatment <- (1 + sin(xb))/2
    treatment <- rbinom(n, 1, prob_treatment)
    
    
    outcome <- 10 + x1^3 + x2^2/2 + 5 * treatment + rnorm(n)
    data.frame(x1, x2, treatment, outcome)
}

set.seed(1)
linear_df <- simu_ob_linear_df(100)
nonlinear_df <- simu_exp_nonlinear_df(100)
```

See OLS result as an example


```r
library(sandwich)
ols.fit = lm(outcome ~ treatment + x1 + x2, data = linear_df)
summary(ols.fit)
#> 
#> Call:
#> lm(formula = outcome ~ treatment + x1 + x2, data = linear_df)
#> 
#> Residuals:
#>      Min       1Q   Median       3Q      Max 
#> -2.52037 -0.62298  0.02196  0.63963  2.40354 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)  10.1763     0.1378  73.838  < 2e-16 ***
#> treatment     4.8630     0.2013  24.160  < 2e-16 ***
#> x1            0.9524     0.1101   8.649 1.18e-13 ***
#> x2            1.1411     0.1050  10.870  < 2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 0.9789 on 96 degrees of freedom
#> Multiple R-squared:  0.9101,	Adjusted R-squared:  0.9073 
#> F-statistic: 323.8 on 3 and 96 DF,  p-value: < 2.2e-16

treatment_est <- coef(ols.fit)["treatment"]
treatment_se <- sqrt(sandwich::vcovHC(ols.fit)["treatment","treatment"])
print(paste0("95% CI:", round(treatment_est, 4)," +/ -", round(1.96 * treatment_se, 4)))
#> [1] "95% CI:4.863 +/ -0.3968"
```

Create dataframes to store results


```r
n_list = c(100, 500, 1000, 2000)
plot_df_linear_ols    <- data.frame(n_list) %>% mutate(tau = NA, lower = NA, upper = NA)
plot_df_linear_ml     <- data.frame(n_list) %>% mutate(tau = NA, lower = NA, upper = NA)
plot_df_nonlinear_ols <- data.frame(n_list) %>% mutate(tau = NA, lower = NA, upper = NA)
plot_df_nonlinear_ml  <- data.frame(n_list) %>% mutate(tau = NA, lower = NA, upper = NA)
```

Check [Bootstrap] section for estimating confidence intervals


```r
library(grf)

# create function to get tau (resemble function in the bootstrap section)
get_tau <- function(data1,i) {
    data = data1[i,]
    
    rf.0 = regression_forest(
        X = data %>% filter(treatment == 0) %>% select(x1, x2) %>% as.matrix(),
        Y = data %>% filter(treatment == 0) %>% select(outcome) %>% as_vector()
    )
    mu.hat.0 <- predict(rf.0, data %>% select(x1, x2))$predictions
    mu.hat.0["treatment" == 0] <- predict(rf.0)$predictions
    
    rf.1 = regression_forest(
        X = data %>% filter(treatment == 1) %>% select(x1, x2) %>% as.matrix(),
        Y = data %>% filter(treatment == 1) %>% select(outcome) %>% as_vector()
    )
    mu.hat.1 <- predict(rf.1, data %>% select(x1, x2))$predictions
    mu.hat.1["treatment" == 1] <- predict(rf.1)$predictions
    
    tau.hat.rf <- mean(mu.hat.1 - mu.hat.0)
    return(tau.hat.rf)
}
```


```r
library(boot)
set.seed(1)
# linear setting using ML
for (i in 1:nrow(plot_df_linear_ml)){
    linear_df <- simu_ob_linear_df(plot_df_linear_ml$n_list[i])
    
    bootstat <- boot(linear_df, 
                 get_tau,    # function to calculate the statistic of interest
                 R = 200)    # number of replicates
    plot_df_linear_ml$tau[i] = bootstat$t0
    plot_df_linear_ml$lower[i] = boot.ci(boot.out = bootstat, type = "norm")$normal[2]
    plot_df_linear_ml$upper[i] = boot.ci(boot.out = bootstat, type = "norm")$normal[3]
    # print(i)
}

# nonlinear setting using ML
for (i in 1:nrow(plot_df_nonlinear_ml)){
    nonlinear_df <- simu_exp_nonlinear_df(plot_df_nonlinear_ml$n_list[i])
    
    bootstat <- boot(nonlinear_df, 
                 get_tau,    # function to calculate the statistic of interest
                 R = 200)    # number of replicates
    plot_df_nonlinear_ml$tau[i] = bootstat$t0
    plot_df_nonlinear_ml$lower[i] = boot.ci(boot.out = bootstat, type = "norm")$normal[2]
    plot_df_nonlinear_ml$upper[i] = boot.ci(boot.out = bootstat, type = "norm")$normal[3]
    # print(i)
}

# linear setting using OLS
for (i in 1:nrow(plot_df_linear_ols)) {
    linear_df <- simu_ob_linear_df(plot_df_linear_ols$n_list[i])
    
    ols.fit = lm(outcome ~ treatment + x1 + x2, data = linear_df)
    
    plot_df_linear_ols$tau[i] = coef(ols.fit)["treatment"]
    plot_df_linear_ols$lower[i] = coef(ols.fit)["treatment"] - 1.96 * sqrt(sandwich::vcovHC(ols.fit)["treatment", "treatment"])
    plot_df_linear_ols$upper[i] = coef(ols.fit)["treatment"] + 1.96 * sqrt(sandwich::vcovHC(ols.fit)["treatment", "treatment"])
    # print(i)
}

# non-linear setting using OLS
for (i in 1:nrow(plot_df_nonlinear_ols)){
    nonlinear_df <- simu_exp_nonlinear_df(plot_df_nonlinear_ols$n_list[i])
    
    ols.fit = lm(outcome ~ treatment + x1 + x2, data = nonlinear_df)
    
    plot_df_nonlinear_ols$tau[i] = coef(ols.fit)["treatment"]
    plot_df_nonlinear_ols$lower[i] = coef(ols.fit)["treatment"] - 1.96 * sqrt(sandwich::vcovHC(ols.fit)["treatment", "treatment"])
    plot_df_nonlinear_ols$upper[i] = coef(ols.fit)["treatment"] + 1.96 * sqrt(sandwich::vcovHC(ols.fit)["treatment", "treatment"])
    # print(i)
}
```

Plot


```r
library(ggplot2)
par(mfrow=c(1,2))
# Evaluating OLS
ggplot(plot_df_linear_ols %>% mutate(n_list = stringr::str_pad(as.character(n_list), 4, pad = "0")), aes(n_list, tau)) + 
    geom_point() + 
    geom_errorbar(aes(ymin = lower, ymax = upper)) +
    geom_hline(yintercept = 5) +
    ggtitle("linear setting")
```

<img src="31_doubly_robust_files/figure-html/unnamed-chunk-6-1.png" width="90%" style="display: block; margin: auto;" />

```r


ggplot(plot_df_nonlinear_ols %>% mutate(n_list = stringr::str_pad(as.character(n_list), 4, pad = "0")), aes(n_list, tau)) + 
    geom_point() + 
    geom_errorbar(aes(ymin = lower, ymax = upper)) +
    geom_hline(yintercept = 5) +
    ggtitle("non-linear setting")
```

<img src="31_doubly_robust_files/figure-html/unnamed-chunk-6-2.png" width="90%" style="display: block; margin: auto;" />

-   Using OLS regression under the linear setting, the treatment effect is always unbiased and approaches the true treatment effect as the sample size increases.

-   Using OLS regression under the nonlinear setting, the treatment effect is biased and as $n \to \infty$, it approaches a biased estimate.


```r
# Evaluating ML
par(mfrow=c(1,2))
ggplot(plot_df_linear_ml %>% mutate(n_list = stringr::str_pad(as.character(n_list), 4, pad = "0")), aes(n_list, tau)) + 
    geom_point() + 
    geom_errorbar(aes(ymin = lower, ymax = upper)) +
    geom_hline(yintercept = 5) +
    ggtitle("linear setting")
```

<img src="31_doubly_robust_files/figure-html/unnamed-chunk-7-1.png" width="90%" style="display: block; margin: auto;" />

```r

ggplot(plot_df_nonlinear_ml %>% mutate(n_list = stringr::str_pad(as.character(n_list), 4, pad = "0")), aes(n_list, tau)) + 
    geom_point() + 
    geom_errorbar(aes(ymin = lower, ymax = upper)) +
    geom_hline(yintercept = 5) +
    ggtitle("non-linear setting")
```

<img src="31_doubly_robust_files/figure-html/unnamed-chunk-7-2.png" width="90%" style="display: block; margin: auto;" />

-   Using ML, as $n \to \infty$ under both linear and non-linear settings, your estimate converges to the true treatment effect

-   But ML under the linear setting, in a finite sample case, you don't have efficiency as in OLS. And for non-linear case, it performs much better but only at large sample sizes.

In summary,

-   With OLS, you are only good when your assumption is valid ($\hat{\mu}_1(X_i), \hat{\mu}_0(X_i)$ are linear)

-   With ML, you never quite wrong or quite right, you can always expect that as $n \to \infty$, you can have the right answer.

## Inverse Propensity Weighting

The goal is that each unit has the same probability of receiving the treatment (i.e., 50%).

-   Units that are likely to receive the treatment are weighted down

-   Unit that are unlikely to receive the treatment are weighted up

By definition, the propensity score is

$$
e(X) = P(T_i = 1 |X_i = x)
$$

which measures the probability of being treated conditional on $X_i$

Under random assignment, the propensity score is constant $e(X) = e_0 \in (0,1)$. Hence, the variability of the propensity can be considered a measure of how far we are from random assignment.

Similar to [Regression Adjustments], we still need the unconfoundedness assumption

$$
[\{Y_i(0)), Y_i(1)\} \perp T_1] | X_i
$$

However, this time, the ATE is formulated differently

$$
\begin{aligned}
\tau &= E[Y_i(1) - Y_i(0)] \\
&= E[E[Y_i(1) |X_i ] - E[Y_i(0) |X_i]]\\
&= E \left[ \frac{E[T_i |X_i ] E[Y_i(1) |X_i]}{e(X_i)} - \frac{E[1 - T_i |X_i] E[Y_i(0) |X_i]}{1 - e(X_i)} \right] \\
&= E \left[ \frac{E[ T_i Y_i(1) |X_i]}{e(X_i)} - \frac{E[(1-T_i) Y_i(0)|X_i]}{1 - e(X_i)} \right] && \text{unconfoundedness} \\
&= E\left[\frac{T_i Y_i}{e(X_i)} - \frac{(1-T_i) Y_i}{1 - e(X_i)} \right] && \text{consistency of potential outcomes}
\end{aligned}
$$

which means that the last equality can give an unbiased estimator of the [ATE][Average Treatment Effects]

$$
\hat{\tau} = \frac{1}{n} \sum_{i = 1}^n \left( \frac{T_i Y_i}{e(X_i)} - \frac{(1 - T_i) Y_i}{1 - e(X_i)} \right)
$$

However, this estimator is unbiased (i.e., $E(\hat{\tau}) = \tau$) only when we have the correct propensity score $e(.)$

But since in almost all cases, we do not know the data generating process (i.e., assignment process), we have to use an estimate of the propensity score $\hat{e}(.)$ from either:

-   Parametric approach: logistic regression

-   Non-parametric approach: machine learning

But both approaches would not be ideal, because the error from using the estimated propensity score would overwhelm the sampling error of the $\hat{\tau}$

Moreover, if you have the wrong model for the propensity score estimate, it would bias your treatment effect estimate.


```r
linear_df <- simu_ob_linear_df(1000)
nonlinear_df <- simu_exp_nonlinear_df(1000)
```

Linear setting


```r
prop_mod <- glm(treatment ~  x1 + x2, data = linear_df, family = 'binomial')
linear_df <- linear_df %>% 
    mutate(propensity = predict(prop_mod, type = "response"),
           weight = 1/ propensity * treatment + 1 / (1- propensity) * (1 - treatment))
linear_df %>% 
    group_by(treatment) %>% 
    summarize(weighted_outcome= weighted.mean(outcome, weight))
#> # A tibble: 2 x 2
#>   treatment weighted_outcome
#>       <int>            <dbl>
#> 1         0             10.0
#> 2         1             15.1

summary(lm(outcome ~ x1 + x2 + treatment, data = linear_df, weights = as.vector(linear_df$weight)))
#> 
#> Call:
#> lm(formula = outcome ~ x1 + x2 + treatment, data = linear_df, 
#>     weights = as.vector(linear_df$weight))
#> 
#> Weighted Residuals:
#>     Min      1Q  Median      3Q     Max 
#> -4.7444 -0.9197 -0.0181  0.8544  6.0037 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)    
#> (Intercept)  9.99164    0.04504  221.84   <2e-16 ***
#> x1           0.98957    0.03269   30.27   <2e-16 ***
#> x2           1.06540    0.03088   34.51   <2e-16 ***
#> treatment    5.02141    0.06389   78.60   <2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 1.428 on 996 degrees of freedom
#> Multiple R-squared:  0.8941,	Adjusted R-squared:  0.8938 
#> F-statistic:  2803 on 3 and 996 DF,  p-value: < 2.2e-16
```

Non-linear setting


```r
prop_mod <- glm(treatment ~  x1 + x2, data = nonlinear_df, family = 'binomial')
nonlinear_df <- nonlinear_df %>% 
    mutate(propensity = predict(prop_mod, type = "response"),
           weight = 1/ propensity * treatment + 1 / (1- propensity) * (1 - treatment))
nonlinear_df %>% 
    group_by(treatment) %>% 
    summarize(weighted_outcome= weighted.mean(outcome, weight))
#> # A tibble: 2 x 2
#>   treatment weighted_outcome
#>       <int>            <dbl>
#> 1         0             10.2
#> 2         1             15.8

summary(lm(outcome ~ x1 + x2 + treatment, data = nonlinear_df, weights = as.vector(nonlinear_df$weight)))
#> 
#> Call:
#> lm(formula = outcome ~ x1 + x2 + treatment, data = nonlinear_df, 
#>     weights = as.vector(nonlinear_df$weight))
#> 
#> Weighted Residuals:
#>      Min       1Q   Median       3Q      Max 
#> -19.3724  -1.5572   0.2344   1.5983  13.8459 
#> 
#> Coefficients:
#>             Estimate Std. Error t value Pr(>|t|)    
#> (Intercept) 10.30332    0.08899 115.775   <2e-16 ***
#> x1           2.49504    0.06534  38.183   <2e-16 ***
#> x2          -0.05043    0.06685  -0.754    0.451    
#> treatment    5.33245    0.12632  42.214   <2e-16 ***
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.784 on 996 degrees of freedom
#> Multiple R-squared:  0.7746,	Adjusted R-squared:  0.7739 
#> F-statistic:  1141 on 3 and 996 DF,  p-value: < 2.2e-16
```

## Augmented Inverse Propensity Weighting

To leverage the best of both worlds ([Regression Adjustments] and [Inverse Propensity Weighting]), we have the [Augmented Inverse Propensity Weighting] (known as the doubly robust estimator):

$$
\hat{\tau} = \frac{1}{n} \sum_{n=1}^n \left(\hat{\mu}_1 (X_i) - \hat{\mu}_0(X_i) \\ 
+ \frac{T_i}{\hat{e}(X_i)} (Y_i - \hat{\mu}_1 (X_i)) - \frac{1 - T_i }{1 - \hat{e}(X_i)} (Y_i - \hat{\mu}_0 (X_i)) \right)
$$

where the first part is [Regression Adjustments] and the second part is the [Inverse Propensity Weighting] estimator applied to the residuals, where it debiases the direct estimate.

When implementing this estimator, we assume

1.  Unconfoundedness
2.  ML method is reasonable accurate (i.e., good prediction)
3.  Overlap: $e(X) \in (0,1)$ (i.e., the propensity scores are bounded for all possible value of $X$). You can check your estimated $\hat{e}(X) \in (0,1)$ (e.g., histogram)


```r
# data
linear_df <- simu_ob_linear_df(100)
nonlinear_df <- simu_exp_nonlinear_df(100)

# package
library(grf)

# linear case
cf <- causal_forest(linear_df %>% select(x1, x2), linear_df$outcome, linear_df$treatment)
ate.est <- average_treatment_effect(cf)
ate.est
#>  estimate   std.err 
#> 5.1683733 0.2391832

# non-linear case
cf <- causal_forest(nonlinear_df %>% select(x1, x2), nonlinear_df$outcome, nonlinear_df$treatment)
ate.est <- average_treatment_effect(cf, target.sample = "overlap")
ate.est
#>  estimate   std.err 
#> 5.4827965 0.3732939
```

We can see in both cases, the estimates contain the true treatment effect
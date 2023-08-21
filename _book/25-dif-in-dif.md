# Difference-in-differences

[List of packages](https://github.com/lnsongxf/DiD-1)

Examples in marketing

-   [@liaukonyte2015television]: TV ad on online shopping
-   [@akca2020value]: aggregators for airlines business effect
-   [@pattabhiramaiah2019paywalls]: paywall affects readership
-   [@wang2018border]: political ad source and message tone on vote shares and turnout using discontinuities in the level of political ads at the borders
-   [@datta2018changing]: streaming service on total music consumption using timing of users adoption of a music streaming service
-   [@janakiraman2018effect]: data breach announcement affect customer spending using timing of data breach and variation whether customer info was breached in that event
-   [@lim2020competitive]: nutritional labels on nutritional quality for other brands in a category using variation in timing of adoption of nutritional labels across categories
-   [@guo2020let]: payment disclosure laws effect on physician prescription behavior using Timing of the Massachusetts open payment law as the exogenous shock
-   [@israeli2018online]: digital monitoring and enforcement on violations using enforcement of min ad price policies
-   [@ramani2019effects]: firms respond to foreign direct investment liberalization using India's reform in 1991.
-   [@he2022market]: using Amazon policy change to examine the causal impact of fake reviews on sales, average ratings.
-   [@peukert2022regulatory]: using European General data protection Regulation, examine the impact of policy change on website usage.

Show the mechanism via

-   Mediation analysis: see [@habel2021variable]

-   Moderation analysis: see [@goldfarb2011online]

## Simple Dif-n-dif

-   A tool developed intuitively to study "natural experiment", but its uses are much broader.

-   [Fixed Effects Estimator] is the foundation for DID

-   Why is dif-in-dif attractive? Identification strategy: Inter-temporal variation between groups

    -   **Cross-sectional estimator** helps avoid omitted (unobserved) **common trends**

    -   **Time-series estimator** helps overcome omitted (unobserved) **cross-sectional differences**

Consider

-   $D_i = 1$ treatment group

-   $D_i = 0$ control group

-   $T= 1$ After the treatment

-   $T =0$ Before the treatment

|                   | After (T = 1)          | Before (T = 0)       |
|-------------------|------------------------|----------------------|
| Treated $D_i =1$  | $E[Y_{1i}(1)|D_i = 1]$ | $E[Y_{0i}(0)|D)i=1]$ |
| Control $D_i = 0$ | $E[Y_{0i}(1) |D_i =0]$ | $E[Y_{0i}(0)|D_i=0]$ |

missing $E[Y_{0i}(1)|D=1]$

**The Average Treatment Effect on Treated (ATT)**

$$
\begin{aligned}
E[Y_1(1) - Y_0(1)|D=1] &= \{E[Y(1)|D=1] - E[Y(1)|D=0] \} \\
&- \{E[Y(0)|D=1] - E[Y(0)|D=0] \}
\end{aligned}
$$

More elaboration:

-   For the treatment group, we isolate the difference between being treated and not being treated. If the untreated group would have been affected in a different way, the DiD design and estimate would tell us nothing.
-   Alternatively, because we can't observe treatment variation in the control group, we can't say anything about the treatment effect on this group.

**Extension**

1.  **More than 2 groups** (multiple treatments and multiple controls), and more than 2 period (pre and post)

$$
Y_{igt} = \alpha_g + \gamma_t + \beta I_{gt} + \delta X_{igt} + \epsilon_{igt}
$$

where

-   $\alpha_g$ is the group-specific fixed effect

-   $\gamma_t$ = time specific fixed effect

-   $\beta$ = dif-in-dif effect

-   $I_{gt}$ = interaction terms (n treatment indicators x n post-treatment dummies) (capture effect heterogeneity over time)

This specification is the "two-way fixed effects DiD" - **TWFE** (i.e., 2 sets of fixed effects: group + time).

-   However, if you have [Staggered Dif-n-dif] (i.e., treatment is applied at different times to different groups). TWFE is really bad.

2.  **Long-term Effects**

To examine the dynamic treatment effects (that are not under rollout/staggered design), we can create a centered time variable,

+------------------------+------------------------------------------------+
| Centered Time Variable | Period                                         |
+========================+================================================+
| ...                    |                                                |
+------------------------+------------------------------------------------+
| $t = -1$               | 2 periods before treatment period              |
+------------------------+------------------------------------------------+
| $t = 0$                | Last period right before treatment period      |
|                        |                                                |
|                        | Remember to use this period as reference group |
+------------------------+------------------------------------------------+
| $t = 1$                | Treatment period                               |
+------------------------+------------------------------------------------+
| ...                    |                                                |
+------------------------+------------------------------------------------+

By interacting this factor variable, we can examine the dynamic effect of treatment (i.e., whether it's fading or intensifying)

$$
\begin{aligned}
Y &= \alpha_0 + \alpha_1 Group + \alpha_2 Time  \\
&+ \beta_{-T_1} Treatment+  \beta_{-(T_1 -1)} Treatment + \dots +  \beta_{-1} Treatment \\
&+ \beta_1 + \dots + \beta_{T_2} Treatment
\end{aligned}
$$

where

-   $\beta_0$ is used as the reference group (i.e., drop from the model)

-   $T_1$ is the pre-treatment period

-   $T_2$ is the post-treatment period

With more variables (i.e., interaction terms), coefficients estimates can be less precise (i.e., higher SE).

3.  DiD on the relationship, not levels. Technically, we can apply DiD research design not only on variables, but also on coefficients estimates of some other regression models with before and after a policy is implemented.

Goal:

1.  Pre-treatment coefficients should be non-significant $\beta_{-T_1}, \dots, \beta_{-1} = 0$ (similar to the [Placebo Test])
2.  Post-treatment coefficients are expected to be significant $\beta_1, \dots, \beta_{T_2} \neq0$
    -   You can now examine the trend in post-treatment coefficients (i.e., increasing or decreasing)


```r
library(tidyverse)
library(fixest)

od <- causaldata::organ_donations %>%
    
    # Treatment variable
    mutate(California = State == 'California') %>%
    # centered time variable
    mutate(center_time = as.factor(Quarter_Num - 3))  
# where 3 is the reference period precedes the treatment period

class(od$California)
#> [1] "logical"
class(od$State)
#> [1] "character"

cali <- feols(Rate ~ i(center_time, California, ref = 0) |
                  State + center_time,
              data = od)

etable(cali)
#>                                              cali
#> Dependent Var.:                              Rate
#>                                                  
#> California x center_time = -2    -0.0029 (0.0051)
#> California x center_time = -1   0.0063** (0.0023)
#> California x center_time = 1  -0.0216*** (0.0050)
#> California x center_time = 2  -0.0203*** (0.0045)
#> California x center_time = 3    -0.0222* (0.0100)
#> Fixed-Effects:                -------------------
#> State                                         Yes
#> center_time                                   Yes
#> _____________________________ ___________________
#> S.E.: Clustered                         by: State
#> Observations                                  162
#> R2                                        0.97934
#> Within R2                                 0.00979
#> ---
#> Signif. codes: 0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

iplot(cali, pt.join = T)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-1-1.png" width="90%" style="display: block; margin: auto;" />

```r
coefplot(cali)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-1-2.png" width="90%" style="display: block; margin: auto;" />

Notes:

-   [Matching Methods]

    -   Match treatment and control based on pre-treatment observables

    -   Modify SEs appropriately [@heckman1997matching]. It's might be easier to just use the [Doubly Robust DiD] [@sant2020doubly] where you just need either matching or regression to work in order to identify your treatment effect

    -   Whereas the group fixed effects control for the group time-invariant effects, it does not control for selection bias (i.e., certain groups are more likely to be treated than others). Hence, with these backdoor open (i.e., selection bias) between (1) propensity to be treated and (2) dynamics evolution of the outcome post-treatment, matching can potential close these backdoor.

    -   Be careful when matching time-varying covariates because you might encounter "regression to the mean" problem, where pre-treatment periods can have an unusually bad or good time (that is out of the ordinary), then the post-treatment period outcome can just be an artifact of the regression to the mean [@daw2018matching]. This problem is not of concern to time-invariant variables.

    -   Matching and DiD can use pre-treatment outcomes to correct for selection bias. From real world data and simulation, [@chabe2015analysis] found that matching generally underestimates the average causal effect and gets closer to the true effect with more number of pre-treatment outcomes. When selection bias is symmetric around the treatment date, DID is still consistent when implemented symmetrically (i.e., the same number of period before and after treatment). In cases where selection bias is asymmetric, the MC simulations show that Symmetric DiD still performs better than Matching.

-   It's always good to show results with and without controls because

    -   If the controls are fixed within group or within time, then those should be absorbed under those fixed effects

    -   If the controls are dynamic across group and across, then your parallel trends assumption is not plausible.

-   SEs are typically clustered within groups, but this approach can make our SEs too small, that leads to overconfidence in our estimates. Hence, @bertrand2004much suggest either

    1.  aggregating data to just one pre-treatment and one post-treatment period per group

    2.  using cluster bootstrapped SEs.

### Examples

#### Example from [Princeton](https://www.princeton.edu/~otorres/DID101R.pdf)


```r
library(foreign)
mydata = read.dta("http://dss.princeton.edu/training/Panel101.dta") %>%
    # create a dummy variable to indicate the time when the treatment started
    mutate(time = ifelse(year >= 1994, 1, 0)) %>%
    # create a dummy variable to identify the treatment group
    mutate(treated = ifelse(country == "E" |
                                country == "F" | country == "G" ,
                            1,
                            0)) %>%
    # create an interaction between time and treated
    mutate(did = time * treated)
```

estimate the DID estimator


```r
didreg = lm(y ~ treated + time + did, data = mydata)
summary(didreg)
#> 
#> Call:
#> lm(formula = y ~ treated + time + did, data = mydata)
#> 
#> Residuals:
#>        Min         1Q     Median         3Q        Max 
#> -9.768e+09 -1.623e+09  1.167e+08  1.393e+09  6.807e+09 
#> 
#> Coefficients:
#>               Estimate Std. Error t value Pr(>|t|)  
#> (Intercept)  3.581e+08  7.382e+08   0.485   0.6292  
#> treated      1.776e+09  1.128e+09   1.575   0.1200  
#> time         2.289e+09  9.530e+08   2.402   0.0191 *
#> did         -2.520e+09  1.456e+09  -1.731   0.0882 .
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
#> 
#> Residual standard error: 2.953e+09 on 66 degrees of freedom
#> Multiple R-squared:  0.08273,	Adjusted R-squared:  0.04104 
#> F-statistic: 1.984 on 3 and 66 DF,  p-value: 0.1249
```

The `did` coefficient is the differences-in-differences estimator. Treat has a negative effect

#### Example by @card1993minimum

found that increase in minimum wage increases employment

Experimental Setting:

-   New Jersey (treatment) increased minimum wage

-   Penn (control) did not increase minimum wage

|           |     | After | Before |                   |
|-----------|-----|-------|--------|-------------------|
| Treatment | NJ  | A     | B      | A - B             |
| Control   | PA  | C     | D      | C - D             |
|           |     | A - C | B - D  | (A - B) - (C - D) |

where

-   A - B = treatment effect + effect of time (additive)

-   C - D = effect of time

-   (A - B) - (C - D) = dif-n-dif

**The identifying assumptions**:

-   Can't have **switchers**

-   PA is the control group

    -   is a good counter factual

    -   is what NJ would look like if they hadn't had the treatment

$$
Y_{jt} = \beta_0 + NJ_j \beta_1 + POST_t \beta_2 + (NJ_j \times POST_t)\beta_3+ X_{jt}\beta_4 + \epsilon_{jt}
$$

where

-   $j$ = restaurant

-   $NJ$ = dummy where 1 = NJ, and 0 = PA

-   $POST$ = dummy where 1 = post, and 0 = pre

Notes:

-   We don't need $\beta_4$ in our model to have unbiased $\beta_3$, but including it would give our coefficients efficiency

-   If we use $\Delta Y_{jt}$ as the dependent variable, we don't need $POST_t \beta_2$ anymore

-   Alternative model specification is that the authors use NJ high wage restaurant as control group (still choose those that are close to the border)

-   The reason why they can't control for everything (PA + NJ high wage) is because it's hard to interpret the causal treatment

-   Dif-n-dif utilizes similarity in pretrend of the dependent variables. However, this is neither a necessary nor sufficient for the identifying assumption.

    -   It's not sufficient because they can have multiple treatments (technically, you could include more control, but your treatment can't interact)

    -   It's not necessary because trends can be parallel after treatment

-   However, we can't never be certain; we just try to find evidence consistent with our theory so that dif-n-dif can work.

-   Notice that we don't need before treatment the **levels of the dependent variable** to be the same (e.g., same wage average in both NJ and PA), dif-n-dif only needs **pre-trend (i.e., slope)** to be the same for the two groups.

#### Example by @butcher2014effects

Theory:

-   Highest achieving students are usually in hard science. Why?

    -   Hard to give students students the benefit of doubt for hard science

    -   How unpleasant and how easy to get a job. Degrees with lower market value typically want to make you feel more pleasant

Under OLS

$$
E_{ij} = \beta_0 + X_i \beta_1 + G_j \beta_2 + \epsilon_{ij}
$$

where

-   $X_i$ = student attributes

-   $\beta_2$ = causal estimate (from grade change)

-   $E_{ij}$ = Did you choose to enroll in major $j$

-   $G_j$ = grade given in major $j$

Examine $\hat{\beta}_2$

-   Negative bias: Endogenous response because department with lower enrollment rate will give better grade

-   Positive bias: hard science is already having best students (i.e., ability), so if they don't their grades can be even lower

Under dif-n-dif

$$
Y_{idt} = \beta_0 + POST_t \beta_1 + Treat_d \beta_2 + (POST_t \times Treat_d)\beta_3 + X_{idt} + \epsilon_{idt}
$$

where

-   $Y_{idt}$ = grade average

+--------------+-----------------------------------+----------+----------+-------------+
|              | Intercept                         | Treat    | Post     | Treat\*Post |
+==============+===================================+==========+==========+=============+
| Treat Pre    | 1                                 | 1        | 0        | 0           |
+--------------+-----------------------------------+----------+----------+-------------+
| Treat Post   | 1                                 | 1        | 1        | 1           |
+--------------+-----------------------------------+----------+----------+-------------+
| Control Pre  | 1                                 | 0        | 0        | 0           |
+--------------+-----------------------------------+----------+----------+-------------+
| Control Post | 1                                 | 0        | 1        | 0           |
+--------------+-----------------------------------+----------+----------+-------------+
|              | Average for pre-control $\beta_0$ |          |          |             |
+--------------+-----------------------------------+----------+----------+-------------+

A more general specification of the dif-n-dif is that

$$
Y_{idt} = \alpha_0 + (POST_t \times Treat_d) \alpha_1 + \theta_d + \delta_t + X_{idt} + u_{idt}
$$

where

-   $(\theta_d + \delta_t)$ richer , more df than $Treat_d \beta_2 + Post_t \beta_1$ (because fixed effects subsume Post and treat)

-   $\alpha_1$ should be equivalent to $\beta_3$ (if your model assumptions are correct)

Under causal inference, $R^2$ is not so important.

### Doubly Robust DiD

Also known as the locally efficient doubly robust DiD [@sant2020doubly]

[Code example by the authors](https://psantanna.com/DRDID/index.html)

The package (not method) is rather limited application:

-   Use OLS (cannot handle `glm`)

-   Canonical DiD only (cannot handle DDD).


```r
library(DRDID)
data("nsw_long")
eval_lalonde_cps <-
    subset(nsw_long, nsw_long$treated == 0 | nsw_long$sample == 2)
head(eval_lalonde_cps)
#>   id year treated age educ black married nodegree dwincl      re74 hisp
#> 1  1 1975      NA  42   16     0       1        0     NA     0.000    0
#> 2  1 1978      NA  42   16     0       1        0     NA     0.000    0
#> 3  2 1975      NA  20   13     0       0        0     NA  2366.794    0
#> 4  2 1978      NA  20   13     0       0        0     NA  2366.794    0
#> 5  3 1975      NA  37   12     0       1        0     NA 25862.322    0
#> 6  3 1978      NA  37   12     0       1        0     NA 25862.322    0
#>   early_ra sample experimental         re
#> 1       NA      2            0     0.0000
#> 2       NA      2            0   100.4854
#> 3       NA      2            0  3317.4678
#> 4       NA      2            0  4793.7451
#> 5       NA      2            0 22781.8555
#> 6       NA      2            0 25564.6699


# locally efficient doubly robust DiD Estimators for the ATT
out <-
    drdid(
        yname = "re",
        tname = "year",
        idname = "id",
        dname = "experimental",
        xformla = ~ age + educ + black + married + nodegree + hisp + re74,
        data = eval_lalonde_cps,
        panel = TRUE
    )
summary(out)
#>  Call:
#> drdid(yname = "re", tname = "year", idname = "id", dname = "experimental", 
#>     xformla = ~age + educ + black + married + nodegree + hisp + 
#>         re74, data = eval_lalonde_cps, panel = TRUE)
#> ------------------------------------------------------------------
#>  Further improved locally efficient DR DID estimator for the ATT:
#>  
#>    ATT     Std. Error  t value    Pr(>|t|)  [95% Conf. Interval] 
#> -901.2703   393.6247   -2.2897     0.022    -1672.7747  -129.766 
#> ------------------------------------------------------------------
#>  Estimator based on panel data.
#>  Outcome regression est. method: weighted least squares.
#>  Propensity score est. method: inverse prob. tilting.
#>  Analytical standard error.
#> ------------------------------------------------------------------
#>  See Sant'Anna and Zhao (2020) for details.



# Improved locally efficient doubly robust DiD estimator 
# for the ATT, with panel data
# drdid_imp_panel()

# Locally efficient doubly robust DiD estimator for the ATT, 
# with panel data
# drdid_panel()

# Locally efficient doubly robust DiD estimator for the ATT, 
# with repeated cross-section data
# drdid_rc()

# Improved locally efficient doubly robust DiD estimator for the ATT, 
# with repeated cross-section data
# drdid_imp_rc()
```

## Two-way Fixed-effects

A generalization of the dif-n-dif model is the two-way fixed-effects models where you have multiple groups and time effects. But this is not a designed-based, non-parametric causal estimator [@imai2021use]

When applying TWFE to multiple groups and multiple periods, the supposedly causal coefficient is the weighted average of all two-group/two-period DiD estimators in the data where some of the weights can be negative. More specifically, the weights are proportional to group sizes and treatment indicator's variation in each pair, where units in the middle of the panel have the highest weight.

The canonical/standard TWFE only works when

-   Effects are homogeneous across units and across time periods (i.e., no dynamic changes in the effects of treatment). See [@goodman2021difference; @de2020two; @sun2021estimating; @borusyak2021revisiting] for details.

    -   Have to argue why treatment heterogeneity is not a problem (e.g., plot treatment timing and decompose treatment coefficient using [Goodman-Bacon Decomposition]) know the percentage of observation are never treated (because as the never-treated group increases, the bias of TWFE decreases, with 80% sample to be never-treated, bias is negligible). The problem is worsen when you have long-run effects.

    -   Need to manually drop two relative time periods if everyone is eventually treated (to avoid multicollinearity). Programs might do this randomly and if it chooses to drop a post-treatment period, it will create biases. The choice usually -1, and -2 periods.

    -   Treatment heterogeneity can come in because (1) it might take some time for a treatment to have measurable changes in outcomes or (2) for each period after treatment, the effect can be different (phase in or increasing effects).

-   2 time periods.

Within this setting, TWFE works because, using the baseline (e.g., control units where their treatment status is unchanged across time periods), the comparison can be

-   Good for

    -   Newly treated units vs. control

    -   Newly treated units vs not-yet treated

-   Bad for

    -   Newly treated vs. already treated (because already treated cannot serve as the potential outcome for the newly treated).

Note: Notation for this section is consistent with [@arkhangelsky2021double]

$$
Y_{it} = \alpha_i + \lambda_t + \tau W_{it} + \beta X_{it} + \epsilon_{it}
$$

where

-   $Y_{it}$ is the outcome

-   $\alpha_i$ is the unit FE

-   $\lambda_t$ is the time FE

-   $\tau$ is the causal effect of treatment

-   $W_{it}$ is the treatment indicator

-   $X_{it}$ are covariates

When $T = 2$, the TWFE is the traditional DiD model

Under the following assumption, $\hat{\tau}_{OLS}$ is unbiased:

1.  homogeneous treatment effect
2.  parallel trends assumptions
3.  linear additive effects [@imai2021use]

Remedies for TWFE's shortcomings

-   [@goodman2021difference]: diagnostic robustness tests of the TWFE DiD and identify influential observations to the DiD estimate ([Goodman-Bacon Decomposition])

-   [@callaway2021difference]: 2-step estimation with a bootstrap procedure that can account for autocorrelation and clustering,

    -   the parameters of interest are the group-time average treatment effects, where each group is defined by when it was first treated ([Multiple periods and variation in treatment timing])

    -   Comparing post-treatment outcomes fo groups treated in a period against a similar group that is never treated (using matching).

    -   Treatment status cannot switch (once treated, stay treated for the rest of the panel)

    -   Package: `did`

-   [@sun2021estimating]: a specialization of [@callaway2021difference] in the event-study context.

    -   They include lags and leads in their design

    -   have cohort-specific estimates (similar to group-time estimates in [@callaway2021difference]

    -   They propose the "interaction-weighted" estimator.

    -   Package: `fixest`

-   [@imai2021use]

    -   Different from [@callaway2021difference] because they allow units to switch in and out of treatment.

    -   Based on matching methods, to have weighted TWFE

    -   Package: `wfe` and `PanelMatch`

-   [@gardner2022two]: two-stage DiD

    -   `did2s`

-   [@arkhangelsky2021double]: see below

To be robust against

1.  time- and unit-varying effects

We can use the reshaped inverse probability weighting (RIPW)- TWFE estimator

With the following assumptions:

-   SUTVA

-   Binary treatment: $\mathbf{W}_i = (W_{i1}, \dots, W_{it})$ where $\mathbf{W}_i \sim \mathbf{\pi}_i$ generalized propensity score (i.e., each person treatment likelihood follow $\pi$ regardless of the period)

Then, the unit-time specific effect is $\tau_{it} = Y_{it}(1) - Y_{it}(0)$

Then the Doubly Average Treatment Effect (DATE) is

$$
\tau(\xi) = \sum_{T=1}^T \xi_t \left(\frac{1}{n} \sum_{i = 1}^n \tau_{it} \right)
$$

where

-   $\frac{1}{n} \sum_{i = 1}^n \tau_{it}$ is the unweighted effect of treatment across units (i.e., time-specific ATE).

-   $\xi = (\xi_1, \dots, \xi_t)$ are user-specific weights for each time period.

-   This estimand is called DATE because it's weighted (averaged) across both time and units.

A special case of DATE is when both time and unit-weights are equal

$$
\tau_{eq} = \frac{1}{nT} \sum_{t=1}^T \sum_{i = 1}^n \tau_{it} 
$$

Borrowing the idea of inverse propensity-weighted least squares estimator in the cross-sectional case that we reweight the objective function via the treatment assignment mechanism:

$$
\hat{\tau} \triangleq \arg \min_{\tau} \sum_{i = 1}^n (Y_i -\mu - W_i \tau)^2 \frac{1}{\pi_i (W_i)}
$$

where

-   the first term is the least squares objective

-   the second term is the propensity score

In the panel data case, the IPW estimator will be

$$
\hat{\tau}_{IPW} \triangleq \arg \min_{\tau} \sum_{i = 1}^n \sum_{t =1}^T (Y_{i t}-\alpha_i - \lambda_t - W_{it} \tau)^2 \frac{1}{\pi_i (W_i)}
$$

Then, to have DATE that users can specify the structure of time weight, we use reshaped IPW estimator [@arkhangelsky2021double]

$$
\hat{\tau}_{RIPW} (\Pi) \triangleq \arg \min_{\tau} \sum_{i = 1}^n \sum_{t =1}^T (Y_{i t}-\alpha_i - \lambda_t - W_{it} \tau)^2 \frac{\Pi(W_i)}{\pi_i (W_i)}
$$

where it's a function of a data-independent distribution $\Pi$ that depends on the support of the treatment path $\mathbb{S} = \cup_i Supp(W_i)$

This generalization can transform to

-   IPW-TWFE estimator when $\Pi \sim Unif(\mathbb{S})$

-   randomized experiment when $\Pi = \pi_i$

To choose $\Pi$, we don't need to data, we just need possible assignments in your setting.

-   For most practical problems (DiD, staggered, transient), we have closed form solutions

-   For generic solver, we can use nonlinear programming (e..g, BFGS algorithm)

As argued in [@imai2021use] that TWFE is not a non-parametric approach, it can be subjected to incorrect model assumption (i.e., model dependence).

-   Hence, they advocate for matching methods for time-series cross-sectional data [@imai2021use]

-   Use `wfe` and `PanelMatch` to apply their paper.

This package is based on [@somaini2016algorithm]


```r
# dataset
library(bacondecomp)
df <- bacondecomp::castle
```


```r
# devtools::install_github("paulosomaini/xtreg2way")

library(xtreg2way)
# output <- xtreg2way(y,
#                     data.frame(x1, x2),
#                     iid,
#                     tid,
#                     w,
#                     noise = "1",
#                     se = "1")

# equilvalently
output <- xtreg2way(l_homicide ~ post,
                    df,
                    iid = df$state, # group id
                    tid = df$year, # time id
                    # w, # vector of weight
                    se = "1")
output$betaHat
#>                  [,1]
#> l_homicide 0.08181162
output$aVarHat
#>             [,1]
#> [1,] 0.003396724

# to save time, you can use your structure in the 
# last output for a new set of variables
# output2 <- xtreg2way(y, x1, struc=output$struc)
```

Standard errors estimation options

+----------------------+---------------------------------------------------------------------------------------------+
| Set                  | Estimation                                                                                  |
+======================+=============================================================================================+
| `se = "0"`           | Assume homoskedasticity and no within group correlation or serial correlation               |
+----------------------+---------------------------------------------------------------------------------------------+
| `se = "1"` (default) | robust to heteroskadasticity and serial correlation [@arellano1987computing]                |
+----------------------+---------------------------------------------------------------------------------------------+
| `se = "2"`           | robust to heteroskedasticity, but assumes no correlation within group or serial correlation |
+----------------------+---------------------------------------------------------------------------------------------+
| `se = "11"`          | Aerllano SE with df correction performed by Stata xtreg [@somaini2021twfem]                 |
+----------------------+---------------------------------------------------------------------------------------------+

Alternatively, you can also do it manually or with the `plm` package, but you have to be careful with how the SEs are estimated


```r
library(multiwayvcov) # get vcov matrix 
library(lmtest) # robust SEs estimation

# manual
output3 <- lm(l_homicide ~ post + factor(state) + factor(year),
              data = df)

# get variance-covariance matrix
vcov_tw <- multiwayvcov::cluster.vcov(output3,
                        cbind(df$state, df$year),
                        use_white = F,
                        df_correction = F)

# get coefficients
coeftest(output3, vcov_tw)[2,] 
#>   Estimate Std. Error    t value   Pr(>|t|) 
#> 0.08181162 0.05671410 1.44252696 0.14979397
```


```r
# using the plm package
library(plm)

output4 <- plm(l_homicide ~ post, 
               data = df, 
               index = c("state", "year"), 
               model = "within", 
               effect = "twoways")

# get coefficients
coeftest(output4, vcov = vcovHC, type = "HC1")
#> 
#> t test of coefficients:
#> 
#>      Estimate Std. Error t value Pr(>|t|)
#> post 0.081812   0.057748  1.4167   0.1572
```

As you can see, differences stem from SE estimation, not the coefficient estimate.

### Multiple periods and variation in treatment timing

This is an extension of the DiD framework to settings where you have

-   more than 2 time periods

-   different treatment timing

When treatment effects are heterogeneous across time or units, the standard [Two-way Fixed-effects] is inappropriate.

Notation is consistent with `did` [package](https://cran.r-project.org/web/packages/did/vignettes/multi-period-did.html) [@callaway2021difference]

-   $Y_{it}(0)$ is the potential outcome for unit $i$

-   $Y_{it}(g)$ is the potential outcome for unit $i$ in time period $t$ if it's treated in period $g$

-   $Y_{it}$ is the observed outcome for unit $i$ in time period $t$

$$
Y_{it} = 
\begin{cases}
Y_{it} = Y_{it}(0) & \forall i \in \text{never-treated group} \\
Y_{it} = 1\{G_i > t\} Y_{it}(0) +  1\{G_i \le t \}Y_{it}(G_i) & \forall i \in \text{other groups}
\end{cases}
$$

-   $G_i$ is the time period when $i$ is treated

-   $C_i$ is a dummy when $i$ belongs to the **never-treated** group

-   $D_{it}$ is a dummy for whether $i$ is treated in period $t$

**Assumptions**:

-   Staggered treatment adoption: once treated, a unit cannot be untreated (revert)

-   Parallel trends assumptions (conditional on covariates):

    -   Based on never-treated units: $E[Y_t(0)- Y_{t-1}(0)|G= g] = E[Y_t(0) - Y_{t-1}(0)|C=1]$

        -   Without treatment, the average potential outcomes for group $g$ equals the average potential outcomes for the never-treated group (i.e., control group), which means that we have (1) enough data on the never-treated group (2) the control group is similar to the eventually treated group.

    -   Based on not-yet treated units: $E[Y_t(0) - Y_{t-1}(0)|G = g] = E[Y_t(0) - Y_{t-1}(0)|D_s = 0, G \neq g]$

        -   Not-yet treated units by time $s$ ( $s \ge t$) can be used as comparison groups to calculate the average treatment effects for the group first treated in time $g$

        -   Additional assumption: pre-treatment trends across groups [@marcus2021role]

-   Random sampling

-   Irreversibility of treatment (once treated, cannot be untreated)

-   Overlap (the treatment propensity $e \in [0,1]$)

Group-Time ATE

-   This is the equivalent of the average treatment effect in the standard case (2 groups, 2 periods) under multiple time periods.

$$
ATT(g,t) = E[Y_t(g) - Y_t(0) |G = g]
$$

which is the average treatment effect for group $g$ in period $t$

-   Identification: When the parallel trends assumption based on

    -   Never-treated units: $ATT(g,t) = E[Y_t - Y_{g-1} |G = g] - E[Y_t - Y_{g-1}|C=1] \forall t \ge g$

    -   Not-yet-treated units: $ATT(g,t) = E[Y_t - Y_{g-1}|G= g] - E[Y_t - Y_{g-1}|D_t = 0, G \neq g] \forall t \ge g$

-   Identification: when the parallel trends assumption only holds conditional on covariates and based on

    -   Never-treated units: $ATT(g,t) = E[Y_t - Y_{g-1} |X, G = g] - E[Y_t - Y_{g-1}|X, C=1] \forall t \ge g$

    -   Not-yet-treated units: $ATT(g,t) = E[Y_t - Y_{g-1}|X, G= g] - E[Y_t - Y_{g-1}|X, D_t = 0, G \neq g] \forall t \ge g$

    -   This is plausible when you have suspected selection bias that can be corrected by using covariates (i.e., very much similar to matching methods to have plausible parallel trends).

Possible parameters of interest are:

1.  Average treatment effect per group

$$
\theta_S(g) = \frac{1}{\tau - g + 1} \sum_{t = 2}^\tau \mathbb{1} \{ \le t \} ATT(g,t)
$$

2.  Average treatment effect across groups (that were treated) (similar to average treatment effect on the treated in the canonical case)

$$
\theta_S^O := \sum_{g=2}^\tau \theta_S(g) P(G=g)
$$

3.  Average treatment effect dynamics (i.e., average treatment effect for groups that have been exposed to the treatment for $e$ time periods):

$$
\theta_D(e) := \sum_{g=2}^\tau \mathbb{1} \{g + e \le \tau \}ATT(g,g + e) P(G = g|G + e \le \tau)
$$

4.  Average treatment effect in period $t$ for all groups that have treated by period $t$)

$$
\theta_C(t) = \sum_{g=2}^\tau \mathbb{1}\{g \le t\} ATT(g,t) P(G = g|g \le t)
$$

5.  Average treatment effect by calendar time

$$
\theta_C = \frac{1}{\tau-1}\sum_{t=2}^\tau \theta_C(t)
$$

### Staggered Dif-n-dif

-   When subjects are treated at different point in time (variation in treatment timing across units), we have to use staggered DiD (also known as DiD event study or dynamic DiD).
-   For design where a treatment is applied and units are exposed to this treatment at all time afterward, see [@athey2022design]

Basic design [@stevenson2006bargaining]

$$
\begin{aligned}
Y_{it} &= \sum_k \beta_k Treatment_{it}^k + \sum_i \eta_i  State_i \\
&+ \sum_t \lambda_t Year_t + Controls_{it} + \epsilon_{it}
\end{aligned}
$$

where

-   $Treatment_{it}^k$ is a series of dummy variables equal to 1 if state $i$ is treated $k$ years ago in period $t$

-   SE is usually clustered at the group level (occasionally time level).

-   To avoid collinearity, the period right before treatment is usually chosen to drop.

In this setting, we try to show that the treatment and control groups are not statistically different (i.e., the coefficient estimates before treatment are not different from 0) to show pre-treatment parallel trends.

However, this two-way fixed effects design has been criticized by [@sun2021estimating; @callaway2021difference; @goodman2021difference]. When researchers include leads and lags of the treatment to see the long-term effects of the treatment, these leads and lags can be biased by effects from other periods, and pre-trends can falsely arise due to treatment effects heterogeneity.

Applying the new proposed method, finance and accounting researchers find that in many cases, the causal estimates turn out to be null [@baker2022much].

Robustness Check

-   The **triple-difference strategy** involves examining the interaction between the **treatment variable** and **the probability of being affected by the program**, and the group-level participation rate. The identification assumption is that there are no differential trends between high and low participation groups in early versus late implementing countries.

**Assumptions**

-   **Rollout Exogeneity** (i.e., exogeneity of treatment adoption): if the treatment is randomly implemented over time (i.e., unrelated to variables that could also affect our dependent variables)

    -   Evidence: Regress adoption on pre-treatment variables. And if you find evidence of correlation, include linear trends interacted with pre-treatment variables [@hoynes2009consumption]

-   **No confounding events**

-   **Exclusion restrictions**

    -   ***No-anticipation assumption***: future treatment time do not affect current outcomes

    -   ***Invariance-to-history assumption***: the time a unit under treatment does not affect the outcome (i.e., the time exposed does not matter, just whether exposed or not). This presents causal effect of early or late adoption on the outcome.

-   And all the assumptions in listed in the [Multiple periods and variation in treatment timing]

-   Auxiliary assumptions:

    -   Constant treatment effects across units

    -   Constant treatment effect over time

    -   Random sampling

    -   Effect Additivity

Remedies for staggered DiD:

-   Each treated cohort is compared to appropriate controls (not-yet-treated, never-treated)

    -   [@goodman2021difference]

    -   [@callaway2021difference] consistent for average ATT. more complicated but also more flexible than [@sun2021estimating]

        -   [@sun2021estimating] (a special case of [@callaway2021difference])

    -   [@de2020two]

    -   [@borusyak2017revisiting]

-   Stacked Regression (biased but simple):

    -   [@gormley2011growing]

    -   [@cengiz2019effect]

    -   [@deshpande2019screened]

#### Stacked DID

Notations following [these slides](https://scholarworks.iu.edu/dspace/bitstream/handle/2022/26875/2021-10-22_wim_wing_did_slides.pdf?sequence=1&isAllowed=y)

$$
Y_{it} = \beta_{FE} D_{it} + A_i + B_t + \epsilon_{it}
$$

where

-   $A_i$ is the group fixed effects

-   $B_t$ is the period fixed effects

Steps

1.  Choose [Event Window]
2.  Enumerate [Sub-experiments]
3.  Define [Inclusion Criteria]
4.  [Stack Data]
5.  Specify [Estimating Equation]

##### Event Window

Let

-   $\kappa_a$ be the length of the pre-event window

-   $\kappa_b$ be the length of the post-event window

By setting a common event window for the analysis, we essentially exclude all those events that do not meet this criteria.

##### Sub-experiments

Let $T_1$ be the earliest period in the dataset

$T_T$ be the last period in the dataset

Then, the collection of all policy adoption periods that are under our event window is

$$
\Omega_A = \{ A_i |T_1 + \kappa_a \le A_i \le T_T - \kappa_b\}
$$

where these events exist

-   at least $\kappa_a$ periods after the earliest period

-   at least $\kappa_b$ periods before the last period

Let $d = 1, \dots, D$ be the index column of the sub-experiments in $\Omega_A$

and $\omega_d$ be the event date of the d-th sub-experiment (e.g., $\omega_1$ = adoption date of the 1st experiment)

##### Inclusion Criteria

1.  Valid treated Units
    -   Within sub-experiment $d$, all treated units have the same adoption date

    -   This makes sure a unit can only serve as a treated unit in only 1 sub-experiment
2.  Clean controls
    -   Only units satisfying $A_i >\omega_d + \kappa_b$ are included as controls in sub-experiment d

    -   This ensures controls are only

        -   never treated units

        -   units that are treated in far future

    -   But a unit can be control unit in multiple sub-experiments (need to correct SE)
3.  Valid Time Periods
    -   All observations within sub-experiment d are from time periods within the sub-experiment's event window

    -   This ensures in sub-experiment d, only observations satisfying $\omega_d - \kappa_a \le t \le \omega_d + \kappa_b$ are included


```r
library(did)
library(tidyverse)
library(fixest)

data(base_stagg)



# first make the stacked datasets
# get the treatment cohorts
cohorts <- base_stagg %>%
    select(year_treated) %>%
    # exclude never-treated group
    filter(year_treated != 10000) %>%
    unique() %>%
    pull()

# make formula to create the sub-datasets
getdata <- function(j, window) {
    #keep what we need
    base_stagg %>%
        # keep treated units and all units not treated within -5 to 5
        # keep treated units and all units not treated within -window to window
        filter(year_treated == j | year_treated > j + window) %>%
        # keep just year -window to window
        filter(year >= j - window & year <= j + window) %>%
        # create an indicator for the dataset
        mutate(df = j)
}

# get data stacked
stacked_data <- map_df(cohorts, ~ getdata(., window = 5)) %>%
    mutate(rel_year = if_else(df == year_treated, time_to_treatment, NA_real_)) %>%
    fastDummies::dummy_cols("rel_year", ignore_na = TRUE) %>%
    mutate(across(starts_with("rel_year_"), ~ replace_na(., 0)))

# get stacked value
stacked <-
    feols(
        y ~ `rel_year_-5` + `rel_year_-4` + `rel_year_-3` +
            `rel_year_-2` + rel_year_0 + rel_year_1 + rel_year_2 + rel_year_3 +
            rel_year_4 + rel_year_5 |
            id ^ df + year ^ df,
        data = stacked_data
    )$coefficients

stacked_se = feols(
    y ~ `rel_year_-5` + `rel_year_-4` + `rel_year_-3` +
        `rel_year_-2` + rel_year_0 + rel_year_1 + rel_year_2 + rel_year_3 +
        rel_year_4 + rel_year_5 |
        id ^ df + year ^ df,
    data = stacked_data
)$se

# add in 0 for omitted -1
stacked <- c(stacked[1:4], 0, stacked[5:10])
stacked_se <- c(stacked_se[1:4], 0, stacked_se[5:10])


cs_out <- att_gt(
    yname = "y",
    data = base_stagg,
    gname = "year_treated",
    idname = "id",
    # xformla = "~x1",
    tname = "year"
)
cs <-
    aggte(
        cs_out,
        type = "dynamic",
        min_e = -5,
        max_e = 5,
        bstrap = FALSE,
        cband = FALSE
    )



res_sa20 = feols(y ~ sunab(year_treated, year) |
                     id + year, base_stagg)
sa = tidy(res_sa20)[5:14, ] %>% pull(estimate)
sa = c(sa[1:4], 0, sa[5:10])

sa_se = tidy(res_sa20)[6:15, ] %>% pull(std.error)
sa_se = c(sa_se[1:4], 0, sa_se[5:10])

compare_df_est = data.frame(
    period = -5:5,
    cs = cs$att.egt,
    sa = sa,
    stacked = stacked
)

compare_df_se = data.frame(
    period = -5:5,
    cs = cs$se.egt,
    sa = sa_se,
    stacked = stacked_se
)

compare_df_longer <- compare_df_est %>%
    pivot_longer(!period, names_to = "estimator", values_to = "est") %>%
    
    full_join(compare_df_se %>% 
                  pivot_longer(!period, names_to = "estimator", values_to = "se")) %>%
    
    mutate(upper = est +  1.96 * se,
           lower = est - 1.96 * se)


ggplot(compare_df_longer) +
    geom_ribbon(aes(
        x = period,
        ymin = lower,
        ymax = upper,
        group = estimator
    )) +
    geom_line(aes(
        x = period,
        y = est,
        group = estimator,
        col = estimator
    ),
    linewidth = 1)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-9-1.png" width="90%" style="display: block; margin: auto;" />

##### Stack Data

##### Estimating Equation

$$
Y_{itd} = \beta_0 + \beta_1 + T_{id} + \beta_2 + P_{td} + \beta_3 (T_{id} \times P_{td}) + \epsilon_{itd}
$$

where

-   $T_{id}$ = 1 if unit $i$ is treated in sub-experiment d, 0 if control

-   $P_{td}$ = 1 if it's the period after the treatment in sub-experiment d

Equivalently,

$$
Y_{itd} = \beta_3 (T_{id} \times P_{td}) + \theta_{id} + \gamma_{td} + \epsilon_{itd}
$$

$\beta_3$ averages all the time-varying effects into a single number (can't see the time-varying effects)

##### Stacked Event Study

Let $YSE_{td} = t - \omega_d$ be the "time since event" variable in sub-experiment d

Then, $YSE_{td} = -\kappa_a, \dots, 0, \dots, \kappa_b$ in every sub-experiment

In each sub-experiment, we can fit

$$
Y_{it}^d = \sum_{j = -\kappa_a}^{\kappa_b} \beta_j^d \times 1(TSE_{td} = j) + \sum_{m = -\kappa_a}^{\kappa_b} \delta_j^d (T_{id} \times 1 (TSE_{td} = j)) + \theta_i^d + \epsilon_{it}^d
$$

-   Different set of event study coefficients in each sub-experiment

-   

$$
Y_{itd} = \sum_{j = -\kappa_a}^{\kappa_b} \beta_j \times 1(TSE_{td} = j) + \sum_{m = -\kappa_a}^{\kappa_b} \delta_j (T_{id} \times 1 (TSE_{td} = j)) + \theta_{id} + \epsilon_{itd}
$$

##### Clustering

-   Clustered at the unit x sub-experiment level [@cengiz2019effect]

-   Clustered at the unit level [@deshpande2019screened]

#### Example by @doleac2020unintended

-   The purpose of banning a checking box for ex-criminal was banned because we thought that it gives more access to felons

-   Even if we ban the box, employers wouldn't just change their behaviors. But then the unintended consequence is that employers statistically discriminate based on race

3 types of ban the box

1.  Public employer only
2.  Private employer with government contract
3.  All employers

Main identification strategy

-   If any county in the Metropolitan Statistical Area (MSA) adopts ban the box, it means the whole MSA is treated. Or if the state adopts "ban the ban," every county is treated

Under [Simple Dif-n-dif]

$$
Y_{it} = \beta_0 + \beta_1 Post_t + \beta_2 treat_i + \beta_2 (Post_t \times Treat_i) + \epsilon_{it}
$$

But if there is no common post time, then we should use [Staggered Dif-n-dif]

$$
\begin{aligned}
E_{imrt} &= \alpha + \beta_1 BTB_{imt} W_{imt} + \beta_2 BTB_{mt} + \beta_3 BTB_{mt} H_{imt}\\ 
&+ \delta_m + D_{imt} \beta_5 + \lambda_{rt} + \delta_m\times f(t) \beta_7 + e_{imrt}
\end{aligned}
$$

where

-   $i$ = person; $m$ = MSA; $r$ = region (US regions e.g., Midwest) ; $r$ = region; $t$ = year

-   $W$ = White; $B$ = Black; $H$ = Hispanic

-   $\beta_1 BTB_{imt} W_{imt} + \beta_2 BTB_{mt} + \beta_3 BTB_{mt} H_{imt}$ are the 3 dif-n-dif variables ($BTB$ = "ban the box")

-   $\delta_m$ = dummy for MSI

-   $D_{imt}$ = control for people

-   $\lambda_{rt}$ = region by time fixed effect

-   $\delta_m \times f(t)$ = linear time trend within MSA (but we should not need this if we have good pre-trend)

If we put $\lambda_r - \lambda_t$ (separately) we will more broad fixed effect, while $\lambda_{rt}$ will give us deeper and narrower fixed effect.

Before running this model, we have to drop all other races. And $\beta_1, \beta_2, \beta_3$ are not collinear because there are all interaction terms with $BTB_{mt}$

If we just want to estimate the model for black men, we will modify it to be

$$
E_{imrt} = \alpha + BTB_{mt} \beta_1 + \delta_m + D_{imt} \beta_5 + \lambda_{rt} + (\delta_m \times f(t)) \beta_7 + e_{imrt}
$$

$$
\begin{aligned}
E_{imrt} &= \alpha + BTB_{m (t - 3t)} \theta_1 + BTB_{m(t-2)} \theta_2 + BTB_{mt} \theta_4 \\
&+ BTB_{m(t+1)}\theta_5 + BTB_{m(t+2)}\theta_6 + BTB_{m(t+3t)}\theta_7 \\
&+ [\delta_m + D_{imt}\beta_5 + \lambda_r + (\delta_m \times (f(t))\beta_7 + e_{imrt}]
\end{aligned}
$$

We have to leave $BTB_{m(t-1)}\theta_3$ out for the category would not be perfect collinearity

So the year before BTB ($\theta_1, \theta_2, \theta_3$) should be similar to each other (i.e., same pre-trend). Remember, we only run for places with BTB.

If $\theta_2$ is statistically different from $\theta_3$ (baseline), then there could be a problem, but it could also make sense if we have pre-trend announcement.

Example by [Philipp Leppert](https://rpubs.com/phle/r_tutorial_difference_in_differences) replicating [Card and Krueger (1994)](https://davidcard.berkeley.edu/data_sets.html)

Example by [Anthony Schmidt](https://bookdown.org/aschmi11/causal_inf/difference-in-differences.html)

#### Goodman-Bacon Decomposition

Paper: [@goodman2021difference]

For an excellent explanation slides by the author, [see](https://www.stata.com/meeting/chicago19/slides/chicago19_Goodman-Bacon.pdf)

Takeaways:

-   A pairwise DID ($\tau$) gets more weight if the change is close to the middle of the study window

-   A pairwise DID ($\tau$) gets more weight if it includes more observations.

Code from `bacondecomp` vignette


```r
library(bacondecomp)
library(tidyverse)
data("castle")
castle <- castle %>% 
    select(l_homicide, post, state, year)
head(castle)
#>   l_homicide post   state year
#> 1   2.027356    0 Alabama 2000
#> 2   2.164867    0 Alabama 2001
#> 3   1.936334    0 Alabama 2002
#> 4   1.919567    0 Alabama 2003
#> 5   1.749841    0 Alabama 2004
#> 6   2.130440    0 Alabama 2005


df_bacon <- bacon(
    l_homicide ~ post,
    data = castle,
    id_var = "state",
    time_var = "year"
)
#>                       type  weight  avg_est
#> 1 Earlier vs Later Treated 0.05976 -0.00554
#> 2 Later vs Earlier Treated 0.03190  0.07032
#> 3     Treated vs Untreated 0.90834  0.08796

# weighted average of the decomposition
sum(df_bacon$estimate * df_bacon$weight)
#> [1] 0.08181162
```

Two-way Fixed effect estimate


```r
library(broom)
fit_tw <- lm(l_homicide ~ post + factor(state) + factor(year), 
             data = bacondecomp::castle)
head(tidy(fit_tw))
#> # A tibble: 6 × 5
#>   term                    estimate std.error statistic   p.value
#>   <chr>                      <dbl>     <dbl>     <dbl>     <dbl>
#> 1 (Intercept)               1.95      0.0624    31.2   2.84e-118
#> 2 post                      0.0818    0.0317     2.58  1.02e-  2
#> 3 factor(state)Alaska      -0.373     0.0797    -4.68  3.77e-  6
#> 4 factor(state)Arizona      0.0158    0.0797     0.198 8.43e-  1
#> 5 factor(state)Arkansas    -0.118     0.0810    -1.46  1.44e-  1
#> 6 factor(state)California  -0.108     0.0810    -1.34  1.82e-  1
```

Hence, naive TWFE fixed effect equals the weighted average of the Bacon decomposition (= 0.08).


```r
library(ggplot2)

ggplot(df_bacon) +
    aes(
        x = weight,
        y = estimate,
        # shape = factor(type),
        color = type
    ) +
    labs(x = "Weight", y = "Estimate", shape = "Type") +
    geom_point()
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-12-1.png" width="90%" style="display: block; margin: auto;" />

With time-varying controls that can identify variation within-treatment timing group, the"early vs. late" and "late vs. early" estimates collapse to just one estimate (i.e., both treated).

#### Chaisemartin-d'Haultfoeuille

use `twowayfeweights` from [GitHub](https://github.com/shuo-zhang-ucsb/twowayfeweights) [@de2020two]

#### didimputation

use `didimputation` from [GitHub](https://github.com/kylebutts/didimputation)

#### staggered

`staggered` [package](https://github.com/jonathandroth/staggered)

#### Wooldridge's Solution

use [etwfe](https://grantmcdermott.com/etwfe/)(Extended two-way Fixed Effects) [@wooldridge2022simple]

### Two-stage DiD

[Example](https://cran.r-project.org/web/packages/did2s/vignettes/Two-Stage-Difference-in-Differences.html) from CRAN

### Multiple Treatment groups

When you have 2 treatments in a setting, you should always try to model both of them under one regression to see whether they are significantly different.

-   Never use one treated groups as control for the other, and run separate regression.
-   Could check this [answer](https://stats.stackexchange.com/questions/474533/difference-in-difference-with-two-treatment-groups-and-one-control-group-classi)

$$
\begin{aligned}
Y_{it} &= \alpha + \gamma_1 Treat1_{i} + \gamma_2 Treat2_{i} + \lambda Post_t  \\
&+ \delta_1(Treat1_i \times Post_t) + \delta_2(Treat2_i \times Post_t) + \epsilon_{it}
\end{aligned}
$$

[@fricke2017identification]

### Multiple Treatments

[@de2022two] [video](https://www.youtube.com/watch?v=UHeJoc27qEM&ab_channel=TaylorWright) [code](https://drive.google.com/file/d/156Fu73avBvvV_H64wePm7eW04V0jEG3K/view)

## Assumption Violation

### Endogenous Timing

If the timing of units can be influenced by strategic decisions in a DID analysis, an instrumental variable approach with a control function can be used to control for endogeneity in timing.



### Questionable Counterfactuals

In situations where the control units may not serve as a reliable counterfactual for the treated units, matching methods such as propensity score matching or generalized random forest can be utilized. Additional methods can be found in [Matching Methods].

## Mediation Under DiD

Check this [post](https://stats.stackexchange.com/questions/261218/difference-in-difference-model-with-mediators-estimating-the-effect-of-differen)

## Assumptions

-   **Parallel Trends**: Difference between the treatment and control groups remain constant if there were no treatment.

    -   should be used in cases where

        -   you observe before and after an event

        -   you have treatment and control groups

    -   not in cases where

        -   treatment is not random

        -   confounders.

    -   To support we use

        -   [Placebo test]

        -   [Prior Parallel Trends Test]

-   **Linear additive effects** (of group/unit specific and time-specific):

    -   If they are not additively interact, we have to use the weighted 2FE estimator [@imai2021use]

    -   Typically seen in the [Staggered Dif-n-dif]

-   No anticipation: There is no causal effect of the treatment before its implementation.

**Possible issues**

-   Estimate dependent on functional form:

    -   When the size of the response depends (nonlinearly) on the size of the intervention, we might want to look at the the difference in the group with high intensity vs. low.

-   Selection on (time--varying) unobservables

    -   Can use the overall sensitivity of coefficient estimates to hidden bias using [Rosenbaum Bounds]

-   Long-term effects

    -   Parallel trends are more likely to be observed over shorter period (window of observation)

-   Heterogeneous effects

    -   Different intensity (e.g., doses) for different groups.

-   Ashenfelter dip [@ashenfelter1985] (job training program participant are more likely to experience an earning drop prior enrolling in these programs)

    -   Participants are systemically different from nonparticipants before the treatment, leading to the question of permanent or transitory changes.
    -   A fix to this transient endogeneity is to calculate long-run differences (exclude a number of periods symmetrically around the adoption/ implementation date). If we see a sustained impact, then we have strong evidence for the causal impact of a policy. [@proserpio2017] [@heckman1999c] [@jepsen2014] [@li2011]

-   Response to event might not be immediate (can't be observed right away in the dependent variable)

    -   Using lagged dependent variable $Y_{it-1}$ might be more appropriate [@blundell1998initial]

-   Other factors that affect the difference in trends between the two groups (i.e., treatment and control) will bias your estimation.

-   Correlated observations within a group or time

-   Incidental parameters problems [@lancaster2000incidental]: it's always better to use individual and time fixed effect.

-   When examining the effects of variation in treatment timing, we have to be careful because negative weights (per group) can be negative if there is a heterogeneity in the treatment effects over time. Example: [@athey2022design][@borusyak2021revisiting][@goodman2021difference]. In this case you should use new estimands proposed by [@callaway2021difference][@de2020two], in the `did` package. If you expect lags and leads, see [@sun2021estimating]

-   [@gibbons2018broken] caution when we suspect the treatment effect and treatment variance vary across groups

### Prior Parallel Trends Test

1.  Plot the average outcomes over time for both treatment and control group before and after the treatment in time.
2.  Statistical test for difference in trends (**using data from before the treatment period**)

$$
Y = \alpha_g + \beta_1 T + \beta_2 T\times G + \epsilon
$$

where

-   $Y$ = the outcome variable

-   $\alpha_g$ = group fixed effects

-   $T$ = time (e.g., specific year, or month)

-   $\beta_2$ = different time trends for each group

Hence, if $\beta_2 =0$ provides evidence that there are no differences in the trend for the two groups prior the time treatment.

You can also use different functional forms (e..g, polynomial or nonlinear).

If $\beta_2 \neq 0$ statistically, possible reasons can be:

-   Statistical significance can be driven by large sample

-   Or the trends are so consistent, and just one period deviation can throw off the trends. Hence, statistical statistical significance.

Technically, we can still salvage the research by including time fixed effects, instead of just the before-and-after time fixed effect (actually, most researchers do this mechanically anyway nowadays). However, a side effect can be that the time fixed effects can also absorb some part your treatment effect as well, especially in cases where the treatment effects vary with time (i.e., stronger or weaker over time) [@wolfers2003business].

Debate:

-   [@kahn2020promise] argue that DiD will be more plausible when the treatment and control groups are similar not only in **trends**, but also in **levels**. Because when we observe dissimilar in levels prior to the treatment, why is it okay to think that this will not affect future trends?

    -   Show a plot of the dependent variable's time series for treated and control groups and also a similar plot with matched sample. [@ryan2019now] show evidence of matched DiD did well in the setting of non-parallel trends (at least in health care setting).

    -   In the case that we don't have similar levels ex ante between treatment and control groups, functional form assumptions matter and we need justification for our choice.

-   Pre-trend statistical tests: [@roth2022pretest] provides evidence that these test are usually under powered.

    -   See [PretrendsPower](https://github.com/jonathandroth/PretrendsPower) and [pretrends](https://github.com/jonathandroth/pretrends) packages for correcting this.


```r
library(tidyverse)
library(fixest)
od <- causaldata::organ_donations %>%
    # Use only pre-treatment data
    filter(Quarter_Num <= 3) %>% 
    # Treatment variable
    mutate(California = State == 'California')

# always good but plot the dependent out
od |>
    # group by treatment status and time
    group_by(California, Quarter) |>
    summarize_all(mean) |>
    ungroup() |>
    # view()
    
    ggplot(aes(x = Quarter_Num, y = Rate, color = California)) +
    geom_line()
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-14-1.png" width="90%" style="display: block; margin: auto;" />

```r


# but it's also important to use statistical test
prior_trend <- feols(Rate ~ i(Quarter_Num, California) | State + Quarter,
               data = od)

coefplot(prior_trend)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-14-2.png" width="90%" style="display: block; margin: auto;" />

```r
iplot(prior_trend)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-14-3.png" width="90%" style="display: block; margin: auto;" />

This is alarming since one of the periods is significantly different from 0, which means that our parallel trends assumption is not plausible.

In cases where you might have violations of parallel trends assumption, check [@rambachan2023more]

-   Impose restrictions on how different the post-treatment violations of parallel trends can be from the pre-trends.

-   Partial identification of causal parameter

-   A type of sensitivity analysis


```r
# https://github.com/asheshrambachan/HonestDiD
# install.packages("HonestDiD")
```

Alternatively, @ban2022generalized propose a method that with an information set (i.e., pre-treatment covariates), and an assumption on the selection bias in the post-treatment period (i.e., lies within the convex hull of all selection biases), they can still identify a set of ATT, and with stricter assumption on selection bias from the policymakers perspective, they can also have a point estimate.

Alternatively, we can use the `pretrends` package to examine our assumptions [@roth2022pretest]

### Placebo Test

Procedure:

1.  Sample data only in the period before the treatment in time.
2.  Consider different fake cutoff in time, either
    1.  Try the whole sequence in time

    2.  Generate random treatment period, and use **randomization inference** to account for sampling distribution of the fake effect.
3.  Estimate the DiD model but with the post-time = 1 with the fake cutoff
4.  A significant DiD coefficient means that you violate the parallel trends! You have a big problem.

Alternatively,

-   When data have multiple control groups, drop the treated group, and assign another control group as a "fake" treated group. But even if it fails (i.e., you find a significant DiD effect) among the control groups, it can still be fine. However, this method is used under [Synthetic Control]

[Code by theeffectbook.net](https://theeffectbook.net/ch-DifferenceinDifference.html)


```r
library(tidyverse)
library(fixest)
od <- causaldata::organ_donations %>%
    # Use only pre-treatment data
    filter(Quarter_Num <= 3) %>% 

# Create fake treatment variables
    mutate(
        FakeTreat1 = State == 'California' &
            Quarter %in% c('Q12011', 'Q22011'),
        FakeTreat2 = State == 'California' &
            Quarter == 'Q22011'
    )


clfe1 <- feols(Rate ~ FakeTreat1 | State + Quarter,
               data = od)
clfe2 <- feols(Rate ~ FakeTreat2 | State + Quarter,
               data = od)

etable(clfe1,clfe2)
#>                           clfe1            clfe2
#> Dependent Var.:            Rate             Rate
#>                                                 
#> FakeTreat1TRUE  0.0061 (0.0051)                 
#> FakeTreat2TRUE                  -0.0017 (0.0028)
#> Fixed-Effects:  --------------- ----------------
#> State                       Yes              Yes
#> Quarter                     Yes              Yes
#> _______________ _______________ ________________
#> S.E.: Clustered       by: State        by: State
#> Observations                 81               81
#> R2                      0.99377          0.99376
#> Within R2               0.00192          0.00015
#> ---
#> Signif. codes: 0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

We would like the "supposed" DiD to be insignificant.

**Robustness Check**

-   Placebo DiD (if the DiD estimate $\neq 0$, parallel trend is violated, and original DiD is biased):

    -   Group: Use fake treatment groups: A population that was **not** affect by the treatment

    -   Time: Redo the DiD analysis for period before the treatment (expected treatment effect is 0) (e..g, for previous year or period).

-   Possible alternative control group: Expected results should be similar

-   Try different windows (further away from the treatment point, other factors can creep in and nullify your effect).

-   Treatment Reversal (what if we don't see the treatment event)

-   Higher-order polynomial time trend (to relax linearity assumption)

-   Test whether other dependent variables that should be affected by the event are indeed unaffected.

    -   Use the same control and treatment period (DiD $\neq0$, there is a problem)

### Rosenbaum Bounds

[Rosenbaum Bounds] assess the overall sensitivity of coefficient estimates to hidden bias [@rosenbaum2002overt] without having knowledge (e.g., direction) of the bias. This method is also known as worst case analyses [@diprete2004assessing].

Consider the treatment assignment is based in a way that the odds of treatment of a unit and its control is different by a multiplier $\Gamma$ (where $\Gamma = 1$ mean that the odds of assignment is identical, which mean random treatment assignment).

-   This bias is the product of an unobservable that influences both treatment selection and outcome by a factor $\Gamma$ (omitted variable bias)

Using this technique, we may estimate the upper limit of the p-value for the treatment effect while assuming selection on unobservables of magnitude $\Gamma$.

Usually, we would create a table of different levels of $\Gamma$ to assess how the magnitude of biases can affect our evidence of the treatment effect (estimate).

If we have treatment assignment is clustered (e.g., within school, within state) we need to adjust the bounds for clustered treatment assignment [@hansen2014clustered] (similar to clustered standard errors)

Then, we can report the minimum value of $\Gamma$ at which the treatment treat is nullified (i.e., become insignificant). And the literature's rules of thumb is that if $\Gamma > 2$, then we have strong evidence for our treatment effect is robust to large biases [@proserpio2017online]

Packages

-   `rbounds` [@keele2010overview]

-   `sensitivitymv` [@rosenbaum2015two]

-   `sensitivitymw` [@rosenbaum2015two]
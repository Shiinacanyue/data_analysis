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

$$ Y_{it} = \beta_0 + \beta_1 Post_t + \beta_2 treat_i + \beta_2 (Post_t \times Treat_i) + \epsilon_{it} $$

But if there is no common post time, then we should use [Staggered Dif-n-dif]

$$ \begin{aligned} E_{imrt} &= \alpha + \beta_1 BTB_{imt} W_{imt} + \beta_2 BTB_{mt} + \beta_3 BTB_{mt} H_{imt}\\  &+ \delta_m + D_{imt} \beta_5 + \lambda_{rt} + \delta_m\times f(t) \beta_7 + e_{imrt} \end{aligned} $$

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

$$ E_{imrt} = \alpha + BTB_{mt} \beta_1 + \delta_m + D_{imt} \beta_5 + \lambda_{rt} + (\delta_m \times f(t)) \beta_7 + e_{imrt} $$

$$ \begin{aligned} E_{imrt} &= \alpha + BTB_{m (t - 3t)} \theta_1 + BTB_{m(t-2)} \theta_2 + BTB_{mt} \theta_4 \\ &+ BTB_{m(t+1)}\theta_5 + BTB_{m(t+2)}\theta_6 + BTB_{m(t+3t)}\theta_7 \\ &+ [\delta_m + D_{imt}\beta_5 + \lambda_r + (\delta_m \times (f(t))\beta_7 + e_{imrt}] \end{aligned} $$

We have to leave $BTB_{m(t-1)}\theta_3$ out for the category would not be perfect collinearity

So the year before BTB ($\theta_1, \theta_2, \theta_3$) should be similar to each other (i.e., same pre-trend). Remember, we only run for places with BTB.

If $\theta_2$ is statistically different from $\theta_3$ (baseline), then there could be a problem, but it could also make sense if we have pre-trend announcement.

Example by [Philipp Leppert](https://rpubs.com/phle/r_tutorial_difference_in_differences) replicating [Card and Krueger (1994)](https://davidcard.berkeley.edu/data_sets.html)

Example by [Anthony Schmidt](https://bookdown.org/aschmi11/causal_inf/difference-in-differences.html)

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

-   Effects are homogeneous across units and across time periods (i.e., no dynamic changes in the effects of treatment). See [@goodman2021difference; @de2020two; @sun2021estimating; @borusyak2021revisiting] for details. Similarly, it relies on the assumption of **linear additive effects** [@imai2021use]

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

**Remedies for TWFE's shortcomings**

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

1.  Choose Event Window
2.  Enumerate Sub-experiments
3.  Define Inclusion Criteria
4.  Stack Data
5.  Specify Estimating Equation

**Event Window**

Let

-   $\kappa_a$ be the length of the pre-event window

-   $\kappa_b$ be the length of the post-event window

By setting a common event window for the analysis, we essentially exclude all those events that do not meet this criteria.

**Sub-experiments**

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

**Inclusion Criteria**

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

**Stack Data**

Estimating Equation

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

**Stacked Event Study**

Let $YSE_{td} = t - \omega_d$ be the "time since event" variable in sub-experiment $d$

Then, $YSE_{td} = -\kappa_a, \dots, 0, \dots, \kappa_b$ in every sub-experiment

In each sub-experiment, we can fit

$$
Y_{it}^d = \sum_{j = -\kappa_a}^{\kappa_b} \beta_j^d \times 1(TSE_{td} = j) + \sum_{m = -\kappa_a}^{\kappa_b} \delta_j^d (T_{id} \times 1 (TSE_{td} = j)) + \theta_i^d + \epsilon_{it}^d
$$

-   Different set of event study coefficients in each sub-experiment

$$
Y_{itd} = \sum_{j = -\kappa_a}^{\kappa_b} \beta_j \times 1(TSE_{td} = j) + \sum_{m = -\kappa_a}^{\kappa_b} \delta_j (T_{id} \times 1 (TSE_{td} = j)) + \theta_{id} + \epsilon_{itd}
$$

**Clustering**

-   Clustered at the unit x sub-experiment level [@cengiz2019effect]

-   Clustered at the unit level [@deshpande2019screened]

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

#### DID with in and out treatment condition

@imai2021use

This case generalizes the staggered adoption setting, allowing units to vary in treatment over time. For $N$ units across $T$ time periods (with potentially unbalanced panels), let $X_{it}$ represent treatment and $Y_{it}$ the outcome for unit $i$ at time $t$. We use the two-way linear fixed effects model:

$$
Y_{it} = \alpha_i + \gamma_t + \beta X_{it} + \epsilon_{it}
$$

for $i = 1, \dots, N$ and $t = 1, \dots, T$. Here, $\alpha_i$ and $\gamma_t$ are unit and time fixed effects. They capture time-invariant unit-specific and unit-invariant time-specific unobserved confounders, respectively. We can express these as $\alpha_i = h(\mathbf{U}_i)$ and $\gamma_t = f(\mathbf{V}_t)$, with $\mathbf{U}_i$ and $\mathbf{V}_t$ being the confounders. The model doesn't assume a specific form for $h(.)$ and $f(.)$, but that they're additive and separable given binary treatment.

The least squares estimate of $\beta$ leverages the covariance in outcome and treatment [@imai2021use, p. 406]. Specifically, it uses the within-unit and within-time variations. Many researchers prefer the two fixed effects (2FE) estimator because it adjusts for both types of unobserved confounders without specific functional-form assumptions, but this is wrong [@imai2019should]. We do need functional-form assumption (i.e., linearity assumption) for the 2FE to work [@imai2021use, p. 406]

-   **Two-Way Matching Estimator**:

    -   It can lead to mismatches; units with the same treatment status get matched when estimating counterfactual outcomes.

    -   Observations need to be matched with opposite treatment status for correct causal effects estimation.

    -   Mismatches can cause attenuation bias.

    -   The 2FE estimator adjusts for this bias using the factor $K$, which represents the net proportion of proper matches between observations with opposite treatment status.

-   **Weighting in 2FE**:

    -   Observation $(i,t)$ is weighted based on how often it acts as a control unit.

    -   The weighted 2FE estimator still has mismatches, but fewer than the standard 2FE estimator.

    -   Adjustments are made based on observations that neither belong to the same unit nor the same time period as the matched observation.

    -   This means there are challenges in adjusting for unit-specific and time-specific unobserved confounders under the two-way fixed effect framework.

-   **Equivalence & Assumptions**:

    -   Equivalence between the 2FE estimator and the DID estimator is dependent on the linearity assumption.

    -   The multi-period DiD estimator is described as an average of two-time-period, two-group DiD estimators applied during changes from control to treatment.

-   **Comparison with DiD**:

    -   In simple settings (two time periods, treatment given to one group in the second period), the standard nonparametric DiD estimator equals the 2FE estimator.

    -   This doesn't hold in multi-period DiD designs where units change treatment status multiple times at different intervals.

    -   Contrary to popular belief, the unweighted 2FE estimator isn't generally equivalent to the multi-period DiD estimator.

    -   While the multi-period DiD can be equivalent to the weighted 2FE, some control observations may have negative regression weights.

-   **Conclusion**:

    -   Justifying the 2FE estimator as the DID estimator isn't warranted without imposing the linearity assumption.

**Application [@imai2021matching]**

-   **Matching Methods**:

    -   Enhance the validity of causal inference.

    -   Reduce model dependence and provide intuitive diagnostics [@ho2007matching]

    -   Rarely utilized in analyzing time series cross-sectional data.

    -   The proposed matching estimators are more robust than the standard two-way fixed effects estimator, which can be biased if mis-specified

    -   Better than synthetic controls (e.g., [@xu2017generalized]) because it needs less data to achieve good performance and and adapt the the context of unit switching treatment status multiple times.

-   Notes:

    -   Potential carryover effects (treatment may have a long-term effect), leading to post-treatment bias.

-   **Proposed Approach**:

    1.  Treated observations are matched with control observations from other units in the same time period with the same treatment history up to a specified number of lags.

    2.  Standard matching and weighting techniques are employed to further refine the matched set.

    3.  Apply a DiD estimator to adjust for time trend.

    4.  The goal is to have treated and matched control observations with similar covariate values.

-   **Assessment**:

    -   The quality of matches is evaluated through covariate balancing.

-   **Estimation**:

    -   Both short-term and long-term average treatment effects on the treated (ATT) are estimated.


```r
library(PanelMatch)
```

**Treatment Variation plot**

-   Visualize the variation of the treatment across space and time

-   Aids in discerning whether the treatment fluctuates adequately over time and units or if the variation is primarily clustered in a subset of data.


```r
DisplayTreatment(
    unit.id = "wbcode2",
    time.id = "year",
    legend.position = "none",
    xlab = "year",
    ylab = "Country Code",
    treatment = "dem",
    
    hide.x.tick.label = TRUE, hide.y.tick.label = TRUE, 
    # dense.plot = TRUE,
    data = dem
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-14-1.png" width="90%" style="display: block; margin: auto;" />

1.  Select $F$ (i.e., the number of leads - time periods after treatment). Driven by what authors are interested in estimating:

-   $F = 0$ is the contemporaneous effect (short-term effect)

-   $F = n$ is the the treatment effect on the outcome two time periods after the treatment. (cumulative or long-term effect)

2.  Select $L$ (number of lags to adjust).

-   Driven by the identification assumption.

-   Balances bias-variance tradeoff.

-   Higher $L$ values increase credibility but reduce efficiency by limiting potential matches.

**Model assumption**:

-   No spillover effect assumed.

-   Carryover effect allowed up to $L$ periods.

-   Potential outcome for a unit depends neither on others' treatment status nor on its past treatment after $L$ periods.

After defining causal quantity with parameters $L$ and $F$.

-   Focus on the average treatment effect of treatment status change.
-   $\delta(F,L)$ is the average causal effect of treatment change (ATT), $F$ periods post-treatment, considering treatment history up to $L$ periods.
-   Causal quantity considers potential future treatment reversals, meaning treatment could revert to control before outcome measurement.

Also possible to estimate the average treatment effect of treatment reversal on the reversed (ART).

Choose $L,F$ based on specific needs.

-   A large $L$ value:

    -   Increases the credibility of the limited carryover effect assumption.

    -   Allows more past treatments (up to $t−L$) to influence the outcome $Y_{i,t+F}$.

    -   Might reduce the number of matches and lead to less precise estimates.

-   Selecting an appropriate number of lags

    -   Researchers should base this choice on substantive knowledge.

    -   Sensitivity of empirical results to this choice should be examined.

-   The choice of $F$ should be:

    -   Substantively motivated.

    -   Decides whether the interest lies in short-term or long-term causal effects.

    -   A large $F$ value can complicate causal effect interpretation, especially if many units switch treatment status during the $F$ lead time period.

**Identification Assumption**

-   Parallel trend assumption conditioned on treatment, outcome (excluding immediate lag), and covariate histories.

-   Doesn't require strong unconfoundedness assumption.

-   Cannot account for unobserved time-varying confounders.

-   Essential to examine outcome time trends.

    -   Check if they're parallel between treated and matched control units using pre-treatment data

-   **Constructing the Matched Sets**:

    -   For each treated observation, create matched control units with identical treatment history from $t−L$ to $t−1$.

    -   Matching based on treatment history helps control for carryover effects.

    -   Past treatments often act as major confounders, but this method can correct for it.

    -   Exact matching on time period adjusts for time-specific unobserved confounders.

    -   Unlike staggered adoption methods, units can change treatment status multiple times.

    -   Matched set allows treatment switching in and out of treatment

-   **Refining the Matched Sets**:

    -   Initially, matched sets adjust only for treatment history.

    -   Parallel trend assumption requires adjustments for other confounders like past outcomes and covariates.

    -   Matching methods:

        -   Match each treated observation with up to $J$ control units.

        -   Distance measures like Mahalanobis distance or propensity score can be used.

        -   Match based on estimated propensity score, considering pretreatment covariates.

        -   Refined matched set selects most similar control units based on observed confounders.

    -   Weighting methods:

        -   Assign weight to each control unit in a matched set.

        -   Weights prioritize more similar units.

        -   Inverse propensity score weighting method can be applied.

        -   Weighting is a more generalized method than matching.

**The Difference-in-Differences Estimator**:

-   Using refined matched sets, the ATT (Average Treatment Effect on the Treated) of policy change is estimated.

-   For each treated observation, estimate the counterfactual outcome using the weighted average of control units in the refined set.

-   The DiD estimate of the ATT is computed for each treated observation, then averaged across all such observations.

-   For noncontemporaneous treatment effects where $F > 0$:

    -   The ATT doesn't specify future treatment sequence.

    -   Matched control units might have units receiving treatment between time $t$ and $t + F$.

    -   Some treated units could return to control conditions between these times.

**Checking Covariate Balance**:

-   The proposed methodology offers the advantage of checking covariate balance between treated and matched control observations.

-   This check helps to see if treated and matched control observations are comparable with respect to observed confounders.

-   Once matched sets are refined, covariate balance examination becomes straightforward.

-   Examine the mean difference of each covariate between a treated observation and its matched controls for each pretreatment time period.

-   Standardize this difference using the standard deviation of each covariate across all treated observations in the dataset.

-   Aggregate this covariate balance measure across all treated observations for each covariate and pretreatment time period.

-   Examine balance for lagged outcome variables over multiple pretreatment periods and time-varying covariates.

    -   This helps evaluate the validity of the parallel trend assumption underlying the proposed DiD estimator.

**Relations with Linear Fixed Effects Regression Estimators**:

-   The standard DiD estimator is equivalent to the linear two-way fixed effects regression estimator when:

    -   Only two time periods exist.

    -   Treatment is given to some units exclusively in the second period.

-   This equivalence doesn't extend to multiperiod DiD designs, where:

    -   More than two time periods are considered.

    -   Units might receive treatment multiple times.

-   Despite this, many researchers relate the use of the two-way fixed effects estimator to the DiD design.

**Standard Error Calculation**:

-   Approach:

    -   Condition on the weights implied by the matching process.

    -   These weights denote how often an observation is utilized in matching [@imbens2015causal]

-   Context:

    -   Analogous to the conditional variance seen in regression models.

    -   Resulting standard errors don't factor in uncertainties around the matching procedure.

    -   They can be viewed as a measure of uncertainty conditional upon the matching process [@ho2007matching].

**Key Findings**:

-   Even in conditions favoring OLS, the proposed matching estimator displayed higher robustness to omitted relevant lags than the linear regression model with fixed effects.

-   The robustness offered by matching came at a cost - reduced statistical power.

-   This emphasizes the classic statistical tradeoff between bias (where matching has an advantage) and variance (where regression models might be more efficient).

**Data Requirements**

-   The treatment variable is binary:

    -   0 signifies "assignment" to control.

    -   1 signifies assignment to treatment.

-   Variables identifying units in the data must be: Numeric or integer.

-   Variables identifying time periods should be: Consecutive numeric/integer data.

-   Data format requirement: Must be provided as a standard `data.frame` object.

Basic functions:

1.  Utilize treatment histories to create matching sets of treated and control units.

2.  Refine these matched sets by determining weights for each control unit in the set.

    -   Units with higher weights have a larger influence during estimations.

**Matching on Treatment History**:

-   Goal is to match units transitioning from untreated to treated status with control units that have similar past treatment histories.

-   Setting the Quantity of Interest (`qoi =`)

    -   `att` average treatment effect on treated units

    -   `atc` average treatment effect of treatment on the control units

    -   `art` average effect of treatment reversal for units that experience treatment reversal

    -   `ate` average treatment effect


```r
# All examples follow the package's vignette
# Create the matched sets
PM.results.none <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "none",
        data = dem,
        match.missing = TRUE,
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )

# visualize the treated unit and matched controls
DisplayTreatment(
    unit.id = "wbcode2",
    time.id = "year",
    legend.position = "none",
    xlab = "year",
    ylab = "Country Code",
    treatment = "dem",
    data = dem,
    matched.set = PM.results.none$att[1],
    # highlight the particular set
    show.set.only = TRUE
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-15-1.png" width="90%" style="display: block; margin: auto;" />

Control units and the treated unit have identical treatment histories over the lag window (1988-1991)


```r
DisplayTreatment(
    unit.id = "wbcode2",
    time.id = "year",
    legend.position = "none",
    xlab = "year",
    ylab = "Country Code",
    treatment = "dem",
    data = dem,
    matched.set = PM.results.none$att[2],
    # highlight the particular set
    show.set.only = TRUE
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-16-1.png" width="90%" style="display: block; margin: auto;" />

This set is more limited than the first one, but we can still see that we have exact past histories.

-   **Refining Matched Sets**

    -   Refinement involves assigning weights to control units.

    -   Users must:

        1.  Specify a method for calculating unit similarity/distance.

        2.  Choose variables for similarity/distance calculations.

-   **Select a Refinement Method**

    -   Users determine the refinement method via the **`refinement.method`** argument.

    -   Options include:

        -   `mahalanobis`

        -   `ps.match`

        -   `CBPS.match`

        -   `ps.weight`

        -   `CBPS.weight`

        -   `ps.msm.weight`

        -   `CBPS.msm.weight`

        -   `none`

    -   Methods with "match" in the name and Mahalanobis will assign equal weights to similar control units.

    -   "Weighting" methods give higher weights to control units more similar to treated units.

-   **Variable Selection**

    -   Users need to define which covariates will be used through the **`covs.formula`** argument, a one-sided formula object.

    -   Variables on the right side of the formula are used for calculations.

    -   "Lagged" versions of variables can be included using the format: **`I(lag(name.of.var, 0:n))`**.

-   **Understanding `PanelMatch` and `matched.set` objects**

    -   The **`PanelMatch` function** returns a **`PanelMatch` object**.

    -   The most crucial element within the `PanelMatch` object is the **matched.set object**.

    -   Within the `PanelMatch` object, the matched.set object will have names like att, art, or atc.

    -   If **`qoi = ate`**, there will be two matched.set objects: att and atc.

-   **Matched.set Object Details**

    -   matched.set is a named list with added attributes.

    -   Attributes include:

        -   Lag

        -   Names of treatment

        -   Unit and time variables

    -   Each list entry represents a matched set of treated and control units.

    -   Naming follows a structure: **`[id variable].[time variable]`**.

    -   Each list element is a vector of control unit ids that match the treated unit mentioned in the element name.

    -   Since it's a matching method, weights are only given to the **`size.match`** most similar control units based on distance calculations.


```r
# PanelMatch without any refinement
PM.results.none <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "none",
        data = dem,
        match.missing = TRUE,
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )

# Extract the matched.set object
msets.none <- PM.results.none$att

# PanelMatch with refinement
PM.results.maha <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "mahalanobis", # use Mahalanobis distance
        data = dem,
        match.missing = TRUE,
        covs.formula = ~ tradewb,
        size.match = 5,
        qoi = "att" ,
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )
msets.maha <- PM.results.maha$att
```


```r
# these 2 should be identical because weights are not shown
msets.none |> head()
#>   wbcode2 year matched.set.size
#> 1       4 1992               74
#> 2       4 1997                2
#> 3       6 1973               63
#> 4       6 1983               73
#> 5       7 1991               81
#> 6       7 1998                1
msets.maha |> head()
#>   wbcode2 year matched.set.size
#> 1       4 1992               74
#> 2       4 1997                2
#> 3       6 1973               63
#> 4       6 1983               73
#> 5       7 1991               81
#> 6       7 1998                1
# summary(msets.none)
# summary(msets.maha)
```

**Visualizing Matched Sets with the plot method**

-   Users can visualize the distribution of the matched set sizes.

-   A red line, by default, indicates the count of matched sets where treated units had no matching control units (i.e., empty matched sets).

-   Plot adjustments can be made using **`graphics::plot`**.


```r
plot(msets.none)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-19-1.png" width="90%" style="display: block; margin: auto;" />

**Comparing Methods of Refinement**

-   Users are encouraged to:

    -   Use substantive knowledge for experimentation and evaluation.

    -   Consider the following when configuring `PanelMatch`:

        1.  The number of matched sets.

        2.  The number of controls matched to each treated unit.

        3.  Achieving covariate balance.

    -   **Note**: Large numbers of small matched sets can lead to larger standard errors during the estimation stage.

    -   Covariates that aren't well balanced can lead to undesirable comparisons between treated and control units.

    -   Aspects to consider include:

        -   Refinement method.

        -   Variables for weight calculation.

        -   Size of the lag window.

        -   Procedures for addressing missing data (refer to **`match.missing`** and **`listwise.delete`** arguments).

        -   Maximum size of matched sets (for matching methods).

-   **Supportive Features:**

    -   **`print`**, **`plot`**, and **`summary`** methods assist in understanding matched sets and their sizes.

    -   **`get_covariate_balance`** helps evaluate covariate balance:

        -   Lower values in the covariate balance calculations are preferred.


```r
PM.results.none <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "none",
        data = dem,
        match.missing = TRUE,
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )
PM.results.maha <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "mahalanobis",
        data = dem,
        match.missing = TRUE,
        covs.formula = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )

# listwise deletion used for missing data
PM.results.listwise <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "mahalanobis",
        data = dem,
        match.missing = FALSE,
        listwise.delete = TRUE,
        covs.formula = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )

# propensity score based weighting method
PM.results.ps.weight <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "ps.weight",
        data = dem,
        match.missing = FALSE,
        listwise.delete = TRUE,
        covs.formula = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE
    )

get_covariate_balance(
    PM.results.none$att,
    data = dem,
    covariates = c("tradewb", "y"),
    plot = FALSE
)
#>         tradewb            y
#> t_4 -0.07245466  0.291871990
#> t_3 -0.20930129  0.208654876
#> t_2 -0.24425207  0.107736647
#> t_1 -0.10806125 -0.004950238

get_covariate_balance(
    PM.results.maha$att,
    data = dem,
    covariates = c("tradewb", "y"),
    plot = FALSE
)
#>         tradewb          y
#> t_4  0.04558637 0.09701606
#> t_3 -0.03312750 0.10844046
#> t_2 -0.01396793 0.08890753
#> t_1  0.10474894 0.06618865


get_covariate_balance(
    PM.results.listwise$att,
    data = dem,
    covariates = c("tradewb", "y"),
    plot = FALSE
)
#>         tradewb          y
#> t_4  0.05634922 0.05223623
#> t_3 -0.01104797 0.05217896
#> t_2  0.01411473 0.03094133
#> t_1  0.06850180 0.02092209

get_covariate_balance(
    PM.results.ps.weight$att,
    data = dem,
    covariates = c("tradewb", "y"),
    plot = FALSE
)
#>         tradewb          y
#> t_4 0.014362590 0.04035905
#> t_3 0.005529734 0.04188731
#> t_2 0.009410044 0.04195008
#> t_1 0.027907540 0.03975173
```

**get_covariate_balance Function Options:**

-   Allows for the generation of plots displaying covariate balance using **`plot = TRUE`**.

-   Plots can be customized using arguments typically used with the base R **`plot`** method.

-   Option to set **`use.equal.weights = TRUE`** for:

    -   Obtaining the balance of unrefined sets.

    -   Facilitating understanding of the refinement's impact.


```r
# Use equal weights
get_covariate_balance(
    PM.results.ps.weight$att,
    data = dem,
    use.equal.weights = TRUE,
    covariates = c("tradewb", "y"),
    plot = TRUE,
    # visualize by setting plot to TRUE
    ylim = c(-1, 1)
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-21-1.png" width="90%" style="display: block; margin: auto;" />

```r

# Compare covariate balance to refined sets
# See large improvement in balance
get_covariate_balance(
    PM.results.ps.weight$att,
    data = dem,
    covariates = c("tradewb", "y"),
    plot = TRUE,
    # visualize by setting plot to TRUE
    ylim = c(-1, 1)
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-21-2.png" width="90%" style="display: block; margin: auto;" />

```r

balance_scatter(
    matched_set_list = list(PM.results.maha$att,
                            PM.results.ps.weight$att),
    data = dem,
    covariates = c("y", "tradewb")
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-21-3.png" width="90%" style="display: block; margin: auto;" />

**`PanelEstimate`**

-   **Standard Error Calculation Methods**

    -   There are different methods available:

        -   **Bootstrap** (default method with 1000 iterations).

        -   **Conditional**: Assumes independence across units, but not time.

        -   **Unconditional**: Doesn't make assumptions of independence across units or time.

    -   For **`qoi`** values set to `att`, `art`, or `atc` [@imai2021matching]:

        -   You can use analytical methods for calculating standard errors, which include both "conditional" and "unconditional" methods.


```r
PE.results <- PanelEstimate(
    sets = PM.results.ps.weight,
    data = dem,
    se.method = "bootstrap",
    number.iterations = 1000,
    confidence.level = .95
)

# point estimates
PE.results[["estimates"]]
#>       t+0       t+1       t+2       t+3       t+4 
#> 0.2609565 0.9630847 1.2851017 1.7370930 1.4871846

# standard errors
PE.results[["standard.error"]]
#>       t+0       t+1       t+2       t+3       t+4 
#> 0.6166251 1.0504656 1.4373638 1.8325026 2.2398552


# use conditional method
PE.results <- PanelEstimate(
    sets = PM.results.ps.weight,
    data = dem,
    se.method = "conditional",
    confidence.level = .95
)

# point estimates
PE.results[["estimates"]]
#>       t+0       t+1       t+2       t+3       t+4 
#> 0.2609565 0.9630847 1.2851017 1.7370930 1.4871846

# standard errors
PE.results[["standard.error"]]
#>       t+0       t+1       t+2       t+3       t+4 
#> 0.4844805 0.8170604 1.1171942 1.4116879 1.7172143

summary(PE.results)
#> Weighted Difference-in-Differences with Propensity Score
#> Matches created with 4 lags
#> 
#> Standard errors computed with conditional  method
#> 
#> Estimate of Average Treatment Effect on the Treated (ATT) by Period:
#> $summary
#>      estimate std.error       2.5%    97.5%
#> t+0 0.2609565 0.4844805 -0.6886078 1.210521
#> t+1 0.9630847 0.8170604 -0.6383243 2.564494
#> t+2 1.2851017 1.1171942 -0.9045586 3.474762
#> t+3 1.7370930 1.4116879 -1.0297644 4.503950
#> t+4 1.4871846 1.7172143 -1.8784937 4.852863
#> 
#> $lag
#> [1] 4
#> 
#> $qoi
#> [1] "att"

plot(PE.results)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-22-1.png" width="90%" style="display: block; margin: auto;" />

**Moderating Variables**


```r
# moderating variable
dem$moderator <- 0
dem$moderator <- ifelse(dem$wbcode2 > 100, 1, 2)

PM.results <-
    PanelMatch(
        lag = 4,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        refinement.method = "mahalanobis",
        data = dem,
        match.missing = TRUE,
        covs.formula = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        # lags
        size.match = 5,
        qoi = "att",
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        use.diagonal.variance.matrix = TRUE
    )
PE.results <-
    PanelEstimate(sets = PM.results,
                  data = dem,
                  moderator = "moderator")

# Each element in the list corresponds to a level in the moderator
plot(PE.results[[1]])
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-23-1.png" width="90%" style="display: block; margin: auto;" />

```r

plot(PE.results[[2]])
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-23-2.png" width="90%" style="display: block; margin: auto;" />

To write up for journal submission, you can follow the following report:

In this study, closely aligned with the research by [@acemoglu2019democracy], two key effects of democracy on economic growth are estimated: the impact of democratization and that of authoritarian reversal. The treatment variable, $X_{it}$, is defined to be one if country $i$ is democratic in year $t$, and zero otherwise.

The Average Treatment Effect for the Treated (ATT) under democratization is formulated as follows:

$$
\begin{aligned}
\delta(F, L) &= \mathbb{E} \left\{ Y_{i, t + F} (X_{it} = 1, X_{i, t - 1} = 0, \{X_{i,t-l}\}_{l=2}^L) \right. \\
&\left. - Y_{i, t + F} (X_{it} = 0, X_{i, t - 1} = 0, \{X_{i,t-l}\}_{l=2}^L) | X_{it} = 1, X_{i, t - 1} = 0 \right\}
\end{aligned}
$$

In this framework, the treated observations are countries that transition from an authoritarian regime $X_{it-1} = 0$ to a democratic one $X_{it} = 1$. The variable $F$ represents the number of leads, denoting the time periods following the treatment, and $L$ signifies the number of lags, indicating the time periods preceding the treatment.

The ATT under authoritarian reversal is given by:

$$
\begin{aligned}
&\mathbb{E} \left[ Y_{i, t + F} (X_{it} = 0, X_{i, t - 1} = 1, \{ X_{i, t - l}\}_{l=2}^L ) \right. \\
&\left. - Y_{i, t + F} (X_{it} = 1, X_{it-1} = 1, \{X_{i, t - l} \}_{l=2}^L ) | X_{it} = 0, X_{i, t - 1} = 1 \right]
\end{aligned}
$$

The ATT is calculated conditioning on 4 years of lags ($L = 4$) and up to 4 years following the policy change $F = 1, 2, 3, 4$. Matched sets for each treated observation are constructed based on its treatment history, with the number of matched control units generally decreasing when considering a 4-year treatment history as compared to a 1-year history.

To enhance the quality of matched sets, methods such as Mahalanobis distance matching, propensity score matching, and propensity score weighting are utilized. These approaches enable us to evaluate the effectiveness of each refinement method. In the process of matching, we employ both up-to-five and up-to-ten matching to investigate how sensitive our empirical results are to the maximum number of allowed matches. For more information on the refinement process, please see the Web Appendix

> The Mahalanobis distance is expressed through a specific formula. We aim to pair each treated unit with a maximum of $J$ control units, permitting replacement, denoted as $| \mathcal{M}_{it} \le J|$. The average Mahalanobis distance between a treated and each control unit over time is computed as:
>
> $$ S_{it} (i') = \frac{1}{L} \sum_{l = 1}^L \sqrt{(\mathbf{V}_{i, t - l} - \mathbf{V}_{i', t -l})^T \mathbf{\Sigma}_{i, t - l}^{-1} (\mathbf{V}_{i, t - l} - \mathbf{V}_{i', t -l})} $$
>
> For a matched control unit $i' \in \mathcal{M}_{it}$, $\mathbf{V}_{it'}$ represents the time-varying covariates to adjust for, and $\mathbf{\Sigma}_{it'}$ is the sample covariance matrix for $\mathbf{V}_{it'}$. Essentially, we calculate a standardized distance using time-varying covariates and average this across different time intervals.
>
> In the context of propensity score matching, we employ a logistic regression model with balanced covariates to derive the propensity score. Defined as the conditional likelihood of treatment given pre-treatment covariates [@rosenbaum1983central], the propensity score is estimated by first creating a data subset comprised of all treated and their matched control units from the same year. This logistic regression model is then fitted as follows:
>
> $$ \begin{aligned} & e_{it} (\{\mathbf{U}_{i, t - l} \}^L_{l = 1}) \\ &= Pr(X_{it} = 1| \mathbf{U}_{i, t -1}, \ldots, \mathbf{U}_{i, t - L}) \\ &= \frac{1}{1 = \exp(- \sum_{l = 1}^L \beta_l^T \mathbf{U}_{i, t - l})} \end{aligned} $$
>
> Given this model, the estimated propensity score for all treated and matched control units is then computed. This enables the adjustment for lagged covariates via matching on the calculated propensity score, resulting in the following distance measure:
>
> $$ S_{it} (i') = | \text{logit} \{ \hat{e}_{it} (\{ \mathbf{U}_{i, t - l}\}^L_{l = 1})\} - \text{logit} \{ \hat{e}_{i't}( \{ \mathbf{U}_{i', t - l} \}^L_{l = 1})\} | $$
>
> Here, $\hat{e}_{i't} (\{ \mathbf{U}_{i, t - l}\}^L_{l = 1})$ represents the estimated propensity score for each matched control unit $i' \in \mathcal{M}_{it}$.
>
> Once the distance measure $S_{it} (i')$ has been determined for all control units in the original matched set, we fine-tune this set by selecting up to $J$ closest control units, which meet a researcher-defined caliper constraint $C$. All other control units receive zero weight. This results in a refined matched set for each treated unit $(i, t)$:
>
> $$ \mathcal{M}_{it}^* = \{i' : i' \in \mathcal{M}_{it}, S_{it} (i') < C, S_{it} \le S_{it}^{(J)}\} $$
>
> $S_{it}^{(J)}$ is the $J$th smallest distance among the control units in the original set $\mathcal{M}_{it}$.
>
> For further refinement using weighting, a weight is assigned to each control unit $i'$ in a matched set corresponding to a treated unit $(i, t)$, with greater weight accorded to more similar units. We utilize inverse propensity score weighting, based on the propensity score model mentioned earlier:
>
> $$ w_{it}^{i'} \propto \frac{\hat{e}_{i't} (\{ \mathbf{U}_{i, t-l} \}^L_{l = 1} )}{1 - \hat{e}_{i't} (\{ \mathbf{U}_{i, t-l} \}^L_{l = 1} )} $$
>
> In this model, $\sum_{i' \in \mathcal{M}_{it}} w_{it}^{i'} = 1$ and $w_{it}^{i'} = 0$ for $i' \notin \mathcal{M}_{it}$. The model is fitted to the complete sample of treated and matched control units.

> Checking Covariate Balance 
> A distinct advantage of the proposed methodology over regression methods is the ability it offers researchers to inspect the covariate balance between treated and matched control observations. This facilitates the evaluation of whether treated and matched control observations are comparable regarding observed confounders. To investigate the mean difference of each covariate (e.g., $V_{it'j}$, representing the $j$-th variable in $\mathbf{V}_{it'}$) between the treated observation and its matched control observation at each pre-treatment time period (i.e., $t' < t$), we further standardize this difference. For any given pretreatment time period, we adjust by the standard deviation of each covariate across all treated observations in the dataset. Thus, the mean difference is quantified in terms of standard deviation units. Formally, for each treated observation $(i,t)$ where $D_{it} = 1$, we define the covariate balance for variable $j$ at the pretreatment time period $t - l$ as: 
> \begin{equation}
> B_{it}(j, l) = \frac{V_{i, t- l,j}- \sum_{i' \in \mathcal{M}_{it}}w_{it}^{i'}V_{i', t-l,j}}{\sqrt{\frac{1}{N_1 - 1} \sum_{i'=1}^N \sum_{t' = L+1}^{T-F}D_{i't'}(V_{i', t'-l, j} - \bar{V}_{t' - l, j})^2}}
> \label{eq:covbalance}
> \end{equation} 
> where $N_1 = \sum_{i'= 1}^N \sum_{t' = L+1}^{T-F} D_{i't'}$ denotes the total number of treated observations and $\bar{V}_{t-l,j} = \sum_{i=1}^N D_{i,t-l,j}/N$. We then aggregate this covariate balance measure across all treated observations for each covariate and pre-treatment time period: 
> \begin{equation}
> \bar{B}(j, l) = \frac{1}{N_1} \sum_{i=1}^N \sum_{t = L+ 1}^{T-F}D_{it} B_{it}(j,l)
> \label{eq:aggbalance}
> \end{equation} 
> Lastly, we evaluate the balance of lagged outcome variables over several pre-treatment periods and that of time-varying covariates. This examination aids in assessing the validity of the parallel trend assumption integral to the DiD estimator justification.

In Figure \@ref(fig:balancescatter), we demonstrate the enhancement of covariate balance thank to the refinement of matched sets. Each scatter plot contrasts the absolute standardized mean difference, as detailed in Equation \@ref(eq:aggbalance), before (horizontal axis) and after (vertical axis) this refinement. Points below the 45-degree line indicate an improved standardized mean balance for certain time-varying covariates post-refinement. The majority of variables benefit from this refinement process. Notably, the propensity score weighting (bottom panel) shows the most significant improvement, whereas Mahalanobis matching (top panel) yields a more modest improvement.


```r
library(PanelMatch)
library(causalverse)

runPanelMatch <- function(method, lag, size.match=NULL, qoi="att") {
    
    # Default parameters for PanelMatch
    common.args <- list(
        lag = lag,
        time.id = "year",
        unit.id = "wbcode2",
        treatment = "dem",
        data = dem,
        covs.formula = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        qoi = qoi,
        outcome.var = "y",
        lead = 0:4,
        forbid.treatment.reversal = FALSE,
        size.match = size.match  # setting size.match here for all methods
    )
    
    if(method == "mahalanobis") {
        common.args$refinement.method <- "mahalanobis"
        common.args$match.missing <- TRUE
        common.args$use.diagonal.variance.matrix <- TRUE
    } else if(method == "ps.match") {
        common.args$refinement.method <- "ps.match"
        common.args$match.missing <- FALSE
        common.args$listwise.delete <- TRUE
    } else if(method == "ps.weight") {
        common.args$refinement.method <- "ps.weight"
        common.args$match.missing <- FALSE
        common.args$listwise.delete <- TRUE
    }
    
    return(do.call(PanelMatch, common.args))
}

methods <- c("mahalanobis", "ps.match", "ps.weight")
lags <- c(1, 4)
sizes <- c(5, 10)

res_pm <- list()

for(method in methods) {
    for(lag in lags) {
        for(size in sizes) {
            name <- paste0(method, ".", lag, "lag.", size, "m")
            res_pm[[name]] <- runPanelMatch(method, lag, size)
        }
    }
}

# Now, you can access res_pm using res_pm[["mahalanobis.1lag.5m"]] etc.

# for treatment reversal
res_pm_rev <- list()

for(method in methods) {
    for(lag in lags) {
        for(size in sizes) {
            name <- paste0(method, ".", lag, "lag.", size, "m")
            res_pm_rev[[name]] <- runPanelMatch(method, lag, size, qoi = "art")
        }
    }
}
```


```r
library(gridExtra)

# Updated plotting function
create_balance_plot <- function(method, lag, sizes, res_pm, dem) {
    matched_set_lists <- lapply(sizes, function(size) {
        res_pm[[paste0(method, ".", lag, "lag.", size, "m")]]$att
    })
    
    return(balance_scatter_custom(
        matched_set_list = matched_set_lists,
        legend.title = "Possible Matches",
        set.names = as.character(sizes),
        legend.position = c(0.2,0.8),
        
        # for compiled plot, you don't need x,y, or main labs
        x.axis.label = "",
        y.axis.label = "",
        main = "",
        data = dem,
        dot.size = 5,
        # show.legend = F,
        them_use = causalverse::ama_theme(base_size = 32),
        covariates = c("y", "tradewb")
    ))
}

plots <- list()

for (method in methods) {
  for (lag in lags) {
    plots[[paste0(method, ".", lag, "lag")]] <-
      create_balance_plot(method, lag, sizes, res_pm, dem)
  }
}

# # Arranging plots in a 3x2 grid
# grid.arrange(plots[["mahalanobis.1lag"]],
#              plots[["mahalanobis.4lag"]],
#              plots[["ps.match.1lag"]],
#              plots[["ps.match.4lag"]],
#              plots[["ps.weight.1lag"]],
#              plots[["ps.weight.4lag"]],
#              ncol=2, nrow=3)


# Standardized Mean Difference of Covariates
library(gridExtra)
library(grid)

# Create column and row labels using textGrob
col_labels <- c("1-year Lag", "4-year Lag")
row_labels <- c("Maha Matching", "PS Matching", "PS Weigthing")

major.axes.fontsize = 40
minor.axes.fontsize = 30

png(file.path(getwd(), "images", "did_balance_scatter.png"), width=1200, height=1000)

# Create a list-of-lists, where each inner list represents a row
grid_list <- list(
    list(
        nullGrob(),
        textGrob(col_labels[1], gp = gpar(fontsize = minor.axes.fontsize)),
        textGrob(col_labels[2], gp = gpar(fontsize = minor.axes.fontsize))
    ),
    
    list(textGrob(
        row_labels[1], gp = gpar(fontsize = minor.axes.fontsize), rot = 90
    ), plots[["mahalanobis.1lag"]], plots[["mahalanobis.4lag"]]),
    
    list(textGrob(
        row_labels[2], gp = gpar(fontsize = minor.axes.fontsize), rot = 90
    ), plots[["ps.match.1lag"]], plots[["ps.match.4lag"]]),
    
    list(textGrob(
        row_labels[3], gp = gpar(fontsize = minor.axes.fontsize), rot = 90
    ), plots[["ps.weight.1lag"]], plots[["ps.weight.4lag"]])
)

# "Flatten" the list-of-lists into a single list of grobs
grobs <- do.call(c, grid_list)

grid.arrange(
  grobs = grobs,
  ncol = 3,
  nrow = 4,
  widths = c(0.15, 0.42, 0.42),
  heights = c(0.15, 0.28, 0.28, 0.28)
)

grid.text(
  "Before Refinement",
  x = 0.5,
  y = 0.03,
  gp = gpar(fontsize = major.axes.fontsize)
)
grid.text(
  "After Refinement",
  x = 0.03,
  y = 0.5,
  rot = 90,
  gp = gpar(fontsize = major.axes.fontsize)
)
dev.off()
#> png 
#>   2
```


```r
library(knitr)
include_graphics(file.path(getwd(), "images", "did_balance_scatter.png"))
```

<div class="figure" style="text-align: center">
<img src="images/did_balance_scatter.png" alt="Variable Balance After Matched Set Refinement" width="100%" />
<p class="caption">(\#fig:balancescatter)Variable Balance After Matched Set Refinement</p>
</div>

Note: Scatter plots display the standardized mean difference of each covariate $j$ and lag year $l$ as defined in Equation \@ref(eq:aggbalance) before (x-axis) and after (y-axis) matched set refinement. Each plot includes varying numbers of possible matches for each matching method. Rows represent different matching/weighting methods, while columns indicate adjustments for various lag lengths.



```r
# Step 1: Define configurations
configurations <- list(
    list(refinement.method = "none", qoi = "att"),
    list(refinement.method = "none", qoi = "art"),
    list(refinement.method = "mahalanobis", qoi = "att"),
    list(refinement.method = "mahalanobis", qoi = "art"),
    list(refinement.method = "ps.match", qoi = "att"),
    list(refinement.method = "ps.match", qoi = "art"),
    list(refinement.method = "ps.weight", qoi = "att"),
    list(refinement.method = "ps.weight", qoi = "art")
)

# Step 2: Use lapply or loop to generate results
results <- lapply(configurations, function(config) {
    PanelMatch(
        lag                       = 4,
        time.id                   = "year",
        unit.id                   = "wbcode2",
        treatment                 = "dem",
        data                      = dem,
        match.missing             = FALSE,
        listwise.delete           = TRUE,
        size.match                = 5,
        outcome.var               = "y",
        lead                      = 0:4,
        forbid.treatment.reversal = FALSE,
        refinement.method         = config$refinement.method,
        covs.formula              = ~ I(lag(tradewb, 1:4)) + I(lag(y, 1:4)),
        qoi                       = config$qoi
    )
})


# Step 3: Get covariate balance and plot
plots <- mapply(function(result, config) {
    df <- get_covariate_balance(
        if(config$qoi == "att") result$att else result$art, 
        data = dem, 
        covariates = c("tradewb", "y"), 
        plot = F
    )
    causalverse::plot_covariate_balance_pretrend(df, main = "")
}, results, configurations, SIMPLIFY = FALSE)

# Set names for plots
names(plots) <- sapply(configurations, function(config) {
    paste(config$qoi, config$refinement.method, sep = ".")
})


library(gridExtra)
library(grid)

# Column and row labels
col_labels <-
  c("None",
    "Mahalanobis",
    "Propensity Score Matching",
    "Propensity Score Weighting")
row_labels <- c("ATT", "ART")

# Specify your desired fontsize for labels
minor.axes.fontsize <- 16
major.axes.fontsize <- 20

# Create a list-of-lists, where each inner list represents a row
grid_list <- list(
  list(
    nullGrob(),
    textGrob(col_labels[1], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[2], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[3], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[4], gp = gpar(fontsize = minor.axes.fontsize))
  ),
  
  list(
    textGrob(row_labels[1], gp = gpar(fontsize = minor.axes.fontsize), rot = 90),
    plots$att.none,
    plots$att.mahalanobis,
    plots$att.ps.match,
    plots$att.ps.weight
  ),
  
  list(
    textGrob(row_labels[2], gp = gpar(fontsize = minor.axes.fontsize), rot = 90),
    plots$art.none,
    plots$art.mahalanobis,
    plots$art.ps.match,
    plots$art.ps.weight
  )
)

# "Flatten" the list-of-lists into a single list of grobs
grobs <- do.call(c, grid_list)

# Arrange your plots with text labels
grid.arrange(
  grobs   = grobs,
  ncol    = 5,
  nrow    = 3,
  widths  = c(0.1, 0.225, 0.225, 0.225, 0.225),
  heights = c(0.1, 0.45, 0.45)
)

# Add main x and y axis titles
grid.text(
  "Refinement Methods",
  x  = 0.5,
  y  = 0.01,
  gp = gpar(fontsize = major.axes.fontsize)
)
grid.text(
  "Quantities of Interest",
  x   = 0.02,
  y   = 0.5,
  rot = 90,
  gp  = gpar(fontsize = major.axes.fontsize)
)
```

<div class="figure" style="text-align: center">
<img src="25-dif-in-dif_files/figure-html/balancepretreat-1.png" alt="Variable Balance in Pre-Treatment Period" width="90%" />
<p class="caption">(\#fig:balancepretreat)Variable Balance in Pre-Treatment Period</p>
</div>

Note: Each graph displays the standardized mean difference, as outlined in Equation \@ref(eq:aggbalance), plotted on the vertical axis across a pre-treatment duration of four years represented on the horizontal axis. The leftmost column illustrates the balance prior to refinement, while the subsequent three columns depict the covariate balance post the application of distinct refinement techniques. Each individual line signifies the balance of a specific variable during the pre-treatment phase.

In Figure \@ref(fig:balancepretreat), we observe a marked improvement in covariate balance due to the implemented matching procedures during the pre-treatment period. Our analysis prioritizes methods that adjust for time-varying covariates over a span of four years preceding the treatment initiation. The two rows delineate the standardized mean balance for both treatment modalities, with individual lines representing the balance for each covariate.

Across all scenarios, the refinement attributed to matched sets significantly enhances balance. Notably, using propensity score weighting considerably mitigates imbalances in confounders. While some degree of imbalance remains evident in the Mahalanobis distance and propensity score matching techniques, the standardized mean difference for the lagged outcome remains stable throughout the pre-treatment phase. This consistency lends credence to the validity of the proposed DiD estimator.

**Estimation Results**

We now detail the estimated ATTs derived from the matching techniques. Figure XXX offers visual representations of the impacts of treatment initiation (upper panel) and treatment reversal (lower panel) on the outcome variable for a duration of 5 years post-transition, specifically, (F = 0, 1, ..., 4). Across the five methods (columns), it becomes evident that the point estimates of effects associated with treatment initiation consistently approximate zero over the 5-year window. In contrast, the estimated outcomes of treatment reversal are notably negative and maintain statistical significance through all refinement techniques during the initial year of transition and the 1 to 4 years that follow, provided treatment reversal is permissible. These effects are notably pronounced, pointing to an estimated reduction of roughly XXX in the outcome variable.

Collectively, our findings indicate that the transition into the treated state from its absence doesn't invariably lead to a heightened outcome. Instead, the transition from the treated state back to its absence exerts a considerable negative effect on the outcome variable in both the short and intermediate terms. Hence, the positive effect of the treatment (if we were to use traditional DiD) is actually driven by the negative effect of treatment reversal. 


```r
# Step 1: Apply PanelEstimate function

# Initialize an empty list to store results
res_est <- vector("list", length(res_pm))

# Iterate over each element in res_pm
for (i in 1:length(res_pm)) {
  res_est[[i]] <- PanelEstimate(
    res_pm[[i]],
    data = dem,
    se.method = "bootstrap",
    number.iterations = 1000,
    confidence.level = .95
  )
  # Transfer the name of the current element to the res_est list
  names(res_est)[i] <- names(res_pm)[i]
}

# Step 2: Apply plot_PanelEstimate function

# Initialize an empty list to store plot results
res_est_plot <- vector("list", length(res_est))

# Iterate over each element in res_est
for (i in 1:length(res_est)) {
    res_est_plot[[i]] <-
        plot_PanelEstimate(res_est[[i]],
                           main = "",
                           theme_use = causalverse::ama_theme(base_size = 14))
    # Transfer the name of the current element to the res_est_plot list
    names(res_est_plot)[i] <- names(res_est)[i]
}
```



```r
# Step 1: Apply PanelEstimate function for res_pm_rev

# Initialize an empty list to store results
res_est_rev <- vector("list", length(res_pm_rev))

# Iterate over each element in res_pm_rev
for (i in 1:length(res_pm_rev)) {
  res_est_rev[[i]] <- PanelEstimate(
    res_pm_rev[[i]],
    data = dem,
    se.method = "bootstrap",
    number.iterations = 1000,
    confidence.level = .95
  )
  # Transfer the name of the current element to the res_est_rev list
  names(res_est_rev)[i] <- names(res_pm_rev)[i]
}

# Step 2: Apply plot_PanelEstimate function for res_est_rev

# Initialize an empty list to store plot results
res_est_plot_rev <- vector("list", length(res_est_rev))

# Iterate over each element in res_est_rev
for (i in 1:length(res_est_rev)) {
    res_est_plot_rev[[i]] <-
        plot_PanelEstimate(res_est_rev[[i]],
                           main = "",
                           theme_use = causalverse::ama_theme(base_size = 14))
  # Transfer the name of the current element to the res_est_plot_rev list
  names(res_est_plot_rev)[i] <- names(res_est_rev)[i]
}
```



```r
library(gridExtra)
library(grid)

# Column and row labels
col_labels <- c("Mahalanobis 5m", 
                "Mahalanobis 10m", 
                "PS Matching 5m", 
                "PS Matching 10m", 
                "PS Weighting 5m")

row_labels <- c("ATT", "ART")

# Specify your desired fontsize for labels
minor.axes.fontsize <- 16
major.axes.fontsize <- 20

# Create a list-of-lists, where each inner list represents a row
grid_list <- list(
  list(
    nullGrob(),
    textGrob(col_labels[1], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[2], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[3], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[4], gp = gpar(fontsize = minor.axes.fontsize)),
    textGrob(col_labels[5], gp = gpar(fontsize = minor.axes.fontsize))
  ),
  
  list(
    textGrob(row_labels[1], gp = gpar(fontsize = minor.axes.fontsize), rot = 90),
    res_est_plot$mahalanobis.1lag.5m,
    res_est_plot$mahalanobis.1lag.10m,
    res_est_plot$ps.match.1lag.5m,
    res_est_plot$ps.match.1lag.10m,
    res_est_plot$ps.weight.1lag.5m
  ),
  
  list(
    textGrob(row_labels[2], gp = gpar(fontsize = minor.axes.fontsize), rot = 90),
    res_est_plot_rev$mahalanobis.1lag.5m,
    res_est_plot_rev$mahalanobis.1lag.10m,
    res_est_plot_rev$ps.match.1lag.5m,
    res_est_plot_rev$ps.match.1lag.10m,
    res_est_plot_rev$ps.weight.1lag.5m
  )
)

# "Flatten" the list-of-lists into a single list of grobs
grobs <- do.call(c, grid_list)

# Arrange your plots with text labels
grid.arrange(
  grobs   = grobs,
  ncol    = 6,
  nrow    = 3,
  widths  = c(0.1, 0.18, 0.18, 0.18, 0.18, 0.18),
  heights = c(0.1, 0.45, 0.45)
)

# Add main x and y axis titles
grid.text(
  "Methods",
  x  = 0.5,
  y  = 0.02,
  gp = gpar(fontsize = major.axes.fontsize)
)
grid.text(
  "",
  x   = 0.02,
  y   = 0.5,
  rot = 90,
  gp  = gpar(fontsize = major.axes.fontsize)
)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-28-1.png" width="90%" style="display: block; margin: auto;" />


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

# use my package
causalverse::plot_par_trends(
    data = od,
    metrics_and_names = list("Rate" = "Rate"),
    treatment_status_var = "California",
    time_var = list(Quarter_Num = "Time"),
    display_CI = F
)
#> [[1]]
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-30-1.png" width="90%" style="display: block; margin: auto;" />

```r

# do it manually
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

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-30-2.png" width="90%" style="display: block; margin: auto;" />

```r


# but it's also important to use statistical test
prior_trend <- feols(Rate ~ i(Quarter_Num, California) | State + Quarter,
               data = od)

coefplot(prior_trend)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-30-3.png" width="90%" style="display: block; margin: auto;" />

```r
iplot(prior_trend)
```

<img src="25-dif-in-dif_files/figure-html/unnamed-chunk-30-4.png" width="90%" style="display: block; margin: auto;" />

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

-   Test whether other dependent variables that should not be affected by the event are indeed unaffected.

    -   Use the same control and treatment period (DiD $\neq0$, there is a problem)

### Rosenbaum Bounds

[Rosenbaum Bounds] assess the overall sensitivity of coefficient estimates to hidden bias [@rosenbaum2002overt] without having knowledge (e.g., direction) of the bias. This method is also known as **worst case analyses** [@diprete2004assessing].

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

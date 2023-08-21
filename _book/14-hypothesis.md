# Hypothesis Testing

Error types:

-   Type I Error (False Positive):

    -   Reality: nope
    -   Diagnosis/Analysis: yes

-   Type II Error (False Negative):

    -   Reality: yes
    -   Diagnosis/Analysis: nope

Power: The probability of rejecting the null hypothesis when it is actually false

**Note:**

-   Always written in terms of the population parameter ($\beta$) not the estimator/estimate ($\hat{\beta}$)

-   Sometimes, different disciplines prefer to use $\beta$ (i.e., standardized coefficient), or $\mathbf{b}$ (i.e., unstandardized coefficient)

    -   $\beta$ and $\mathbf{b}$ are similar in interpretation; however, $\beta$ is scale free. Hence, you can see the relative contribution of $\beta$ to the dependent variable. On the other hand, $\mathbf{b}$ can be more easily used in policy decisions.

    -   $$
        \beta_j = \mathbf{b} \frac{s_{x_j}}{s_y}
        $$

-   Assuming the null hypothesis is true, what is the (asymptotic) distribution of the estimator

-   Two-sided

$$
\begin{aligned}
&H_0: \beta_j = 0 \\
&H_1: \beta_j \neq 0 
\end{aligned}
$$

then under the null, the OLS estimator has the following distribution

$$
A1-A3a, A5: \sqrt{n} \hat{\beta_j}  \sim  N(0,Avar(\sqrt{n}\hat{\beta}_j))
$$

-   For the one-sided test, the null is a set of values, so now you choose the worst case single value that is hardest to prove and derive the distribution under the null
-   One-sided

$$
\begin{aligned}
&H_0: \beta_j\ge 0 \\
&H_1: \beta_j < 0 
\end{aligned}
$$

then the hardest null value to prove is $H_0: \beta_j=0$. Then under this specific null, the OLS estimator has the following asymptotic distribution

$$
A1-A3a, A5: \sqrt{n}\hat{\beta_j} \sim N(0,Avar(\sqrt{n}\hat{\beta}_j))
$$

## Types of hypothesis testing

$H_0 : \theta = \theta_0$

$H_1 : \theta \neq \theta_0$

How far away / extreme $\theta$ can be if our null hypothesis is true

Assume that our likelihood function for q is $L(q) = q^{30}(1-q)^{70}$ **Likelihood function**


```r
q = seq(0, 1, length = 100)
L = function(q) {
    q ^ 30 * (1 - q) ^ 70
}

plot(q,
     L(q),
     ylab = "L(q)",
     xlab = "q",
     type = "l")
```

<img src="14-hypothesis_files/figure-html/unnamed-chunk-1-1.png" width="90%" style="display: block; margin: auto;" />

**Log-Likelihood function**


```r
q = seq(0, 1, length = 100)
l = function(q) {
    30 * log(q) + 70 * log(1 - q)
}
plot(q,
     l(q) - l(0.3),
     ylab = "l(q) - l(qhat)",
     xlab = "q",
     type = "l")
abline(v = 0.2)
```

<img src="14-hypothesis_files/figure-html/unnamed-chunk-2-1.png" width="90%" style="display: block; margin: auto;" />

![](images/nested_tests.jpg){style="display: block; margin: 1em auto" width="600" height="400"}

Figure from[@fox1997applied]

typically, [The likelihood ratio test] (and [Lagrange Multiplier (Score)](#lagrange-multiplier-score)) performs better with small to moderate sample sizes, but the [Wald test] only requires one maximization (under the full model).

## Wald test

$$
\begin{aligned}
W &= (\hat{\theta}-\theta_0)'[cov(\hat{\theta})]^{-1}(\hat{\theta}-\theta_0) \\
W &\sim \chi_q^2
\end{aligned}
$$

where $cov(\hat{\theta})$ is given by the inverse Fisher Information matrix evaluated at $\hat{\theta}$ and q is the rank of $cov(\hat{\theta})$, which is the number of non-redundant parameters in $\theta$

Alternatively,

$$
t_W=\frac{(\hat{\theta}-\theta_0)^2}{I(\theta_0)^{-1}} \sim \chi^2_{(v)}
$$

where v is the degree of freedom.

Equivalently,

$$
s_W= \frac{\hat{\theta}-\theta_0}{\sqrt{I(\hat{\theta})^{-1}}} \sim Z
$$

How far away in the distribution your sample estimate is from the hypothesized population parameter.

For a null value, what is the probability you would have obtained a realization "more extreme" or "worse" than the estimate you actually obtained?

Significance Level ($\alpha$) and Confidence Level ($1-\alpha$)

-   The significance level is the benchmark in which the probability is so low that we would have to reject the null
-   The confidence level is the probability that sets the bounds on how far away the realization of the estimator would have to be to reject the null.

**Test Statistics**

-   Standardized (transform) the estimator and null value to a test statistic that always has the same distribution
-   Test Statistic for the OLS estimator for a single hypothesis

$$
T = \frac{\sqrt{n}(\hat{\beta}_j-\beta_{j0})}{\sqrt{n}SE(\hat{\beta_j})} \sim^a N(0,1)
$$

Equivalently,

$$
T = \frac{(\hat{\beta}_j-\beta_{j0})}{SE(\hat{\beta_j})} \sim^a N(0,1)
$$

the test statistic is another random variable that is a function of the data and null hypothesis.

-   T denotes the random variable test statistic
-   t denotes the single realization of the test statistic

Evaluating Test Statistic: determine whether or not we reject or fail to reject the null hypothesis at a given significance / confidence level

Three equivalent ways

1.  Critical Value

2.  P-value

3.  Confidence Interval

4.  Critical Value

For a given significance level, will determine the critical value $(c)$

-   One-sided: $H_0: \beta_j \ge \beta_{j0}$

$$
P(T<c|H_0)=\alpha
$$

Reject the null if $t<c$

-   One-sided: $H_0: \beta_j \le \beta_{j0}$

$$
P(T>c|H_0)=\alpha
$$

Reject the null if $t>c$

-   Two-sided: $H_0: \beta_j \neq \beta_{j0}$

$$
P(|T|>c|H_0)=\alpha
$$

Reject the null if $|t|>c$

2.  p-value

Calculate the probability that the test statistic was worse than the realization you have

-   One-sided: $H_0: \beta_j \ge \beta_{j0}$

$$
\text{p-value} = P(T<t|H_0)
$$

-   One-sided: $H_0: \beta_j \le \beta_{j0}$

$$
\text{p-value} = P(T>t|H_0)
$$

-   Two-sided: $H_0: \beta_j \neq \beta_{j0}$

$$
\text{p-value} = P(|T|<t|H_0)
$$

reject the null if p-value $< \alpha$

3.  Confidence Interval

Using the critical value associated with a null hypothesis and significance level, create an interval

$$
CI(\hat{\beta}_j)_{\alpha} = [\hat{\beta}_j-(c \times SE(\hat{\beta}_j)),\hat{\beta}_j+(c \times SE(\hat{\beta}_j))]
$$

If the null set lies outside the interval then we reject the null.

-   We are not testing whether the true population value is close to the estimate, we are testing that given a field true population value of the parameter, how like it is that we observed this estimate.
-   Can be interpreted as we believe with $(1-\alpha)\times 100 \%$ probability that the confidence interval captures the true parameter value.

With stronger assumption (A1-A6), we could consider [Finite Sample Properties]

$$
T = \frac{\hat{\beta}_j-\beta_{j0}}{SE(\hat{\beta}_j)} \sim T(n-k)
$$

-   This above distributional derivation is strongly dependent on [A4][A4 Homoskedasticity] and [A5][A5 Data Generation (random Sampling)]
-   T has a student t-distribution because the numerator is normal and the denominator is $\chi^2$.
-   Critical value and p-values will be calculated from the student t-distribution rather than the standard normal distribution.
-   $n \to \infty$, $T(n-k)$ is asymptotically standard normal.

**Rule of thumb**

-   if $n-k>120$: the critical values and p-values from the t-distribution are (almost) the same as the critical values and p-values from the standard normal distribution.

-   if $n-k<120$

    -   if (A1-A6) hold then the t-test is an exact finite distribution test
    -   if (A1-A3a, A5) hold, because the t-distribution is asymptotically normal, computing the critical values from a t-distribution is still a valid asymptotic test (i.e., not quite the right critical values and p0values, the difference goes away as $n \to \infty$)

### Multiple Hypothesis

-   test multiple parameters as the same time

    -   $H_0: \beta_1 = 0\ \& \ \beta_2 = 0$
    -   $H_0: \beta_1 = 1\ \& \ \beta_2 = 0$

-   perform a series of simply hypothesis does not answer the question (joint distribution vs. two marginal distributions).

-   The test statistic is based on a restriction written in matrix form.

$$
y=\beta_0+x_1\beta_1 + x_2\beta_2 + x_3\beta_3 + \epsilon
$$

Null hypothesis is $H_0: \beta_1 = 0$ & $\beta_2=0$ can be rewritten as $H_0: \mathbf{R}\beta -\mathbf{q}=0$ where

-   $\mathbf{R}$ is a $m \times k$ matrix where m is the number of restrictions and $k$ is the number of parameters. $\mathbf{q}$ is a $k \times 1$ vector
-   $\mathbf{R}$ "picks up" the relevant parameters while $\mathbf{q}$ is a the null value of the parameter

$$
\mathbf{R}= 
\left(
\begin{array}{cccc}
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
\end{array}
\right),
\mathbf{q} = 
\left(
\begin{array}{c}
0 \\
0 \\
\end{array}
\right)
$$

Test Statistic for OLS estimator for a multiple hypothesis

$$
F = \frac{(\mathbf{R\hat{\beta}-q})\hat{\Sigma}^{-1}(\mathbf{R\hat{\beta}-q})}{m} \sim^a F(m,n-k)
$$

-   $\hat{\Sigma}^{-1}$ is the estimator for the asymptotic variance-covariance matrix

    -   if [A4][A4 Homoskedasticity] holds, both the homoskedastic and heteroskedastic versions produce valid estimator
    -   If [A4][A4 Homoskedasticity] does not hold, only the heteroskedastic version produces valid estimators.

-   When $m = 1$, there is only a single restriction, then the $F$-statistic is the $t$-statistic squared.

-   $F$ distribution is strictly positive, check [F-Distribution] for more details.

### Linear Combination

Testing multiple parameters as the same time

$$
\begin{aligned}
H_0&: \beta_1 -\beta_2 = 0 \\
H_0&: \beta_1 - \beta_2 > 0 \\
H_0&: \beta_1 - 2\times\beta_2 =0
\end{aligned}
$$

Each is a single restriction on a function of the parameters.

Null hypothesis:

$$
H_0: \beta_1 -\beta_2 = 0
$$

can be rewritten as

$$
H_0: \mathbf{R}\beta -\mathbf{q}=0
$$

where $\mathbf{R}$=(0 1 -1 0 0) and $\mathbf{q}=0$

### Estimate Difference in Coefficients

There is no package to estimate for the difference between two coefficients and its CI, but a simple function created by [Katherine Zee](https://kzee.github.io/CoeffDiff_Demo.html) can be used to calculate this difference. Some modifications might be needed if you don't use standard `lm` model in R.


```r
difftest_lm <- function(x1, x2, model) {
    diffest <-
        summary(model)$coef[x1, "Estimate"] - summary(model)$coef[x2, "Estimate"]
    
    vardiff <- (summary(model)$coef[x1, "Std. Error"] ^ 2 +
                    summary(model)$coef[x2, "Std. Error"] ^ 2) - (2 * (vcov(model)[x1, x2]))
    # variance of x1 + variance of x2 - 2*covariance of x1 and x2
    diffse <- sqrt(vardiff)
    tdiff <- (diffest) / (diffse)
    ptdiff <- 2 * (1 - pt(abs(tdiff), model$df, lower.tail = T))
    upr <-
        # will usually be very close to 1.96
        diffest + qt(.975, df = model$df) * diffse 
    lwr <- diffest + qt(.025, df = model$df) * diffse
    df <- model$df
    return(list(
        est = round(diffest, digits = 2),
        t = round(tdiff, digits = 2),
        p = round(ptdiff, digits = 4),
        lwr = round(lwr, digits = 2),
        upr = round(upr, digits = 2),
        df = df
    ))
}
```

### Application


```r
library("car")

# Multiple hypothesis
mod.davis <- lm(weight ~ repwt, data=Davis)
linearHypothesis(mod.davis, c("(Intercept) = 0", "repwt = 1"),white.adjust = TRUE)
#> Linear hypothesis test
#> 
#> Hypothesis:
#> (Intercept) = 0
#> repwt = 1
#> 
#> Model 1: restricted model
#> Model 2: weight ~ repwt
#> 
#> Note: Coefficient covariance matrix supplied.
#> 
#>   Res.Df Df      F  Pr(>F)  
#> 1    183                    
#> 2    181  2 3.3896 0.03588 *
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

# Linear Combination
mod.duncan <- lm(prestige ~ income + education, data=Duncan)
linearHypothesis(mod.duncan, "1*income - 1*education = 0")
#> Linear hypothesis test
#> 
#> Hypothesis:
#> income - education = 0
#> 
#> Model 1: restricted model
#> Model 2: prestige ~ income + education
#> 
#>   Res.Df    RSS Df Sum of Sq      F Pr(>F)
#> 1     43 7518.9                           
#> 2     42 7506.7  1    12.195 0.0682 0.7952
```

### Nonlinear

Suppose that we have q nonlinear functions of the parameters\
$$
\mathbf{h}(\theta) = \{ h_1 (\theta), ..., h_q (\theta)\}'
$$

The,n, the Jacobian matrix ($\mathbf{H}(\theta)$), of rank q is

$$
\mathbf{H}_{q \times p}(\theta) = 
\left(
\begin{array}
{ccc}
\frac{\partial h_1(\theta)}{\partial \theta_1} & ... & \frac{\partial h_1(\theta)}{\partial \theta_p} \\
. & . & . \\
\frac{\partial h_q(\theta)}{\partial \theta_1} & ... & \frac{\partial h_q(\theta)}{\partial \theta_p}
\end{array}
\right)
$$

where the null hypothesis $H_0: \mathbf{h} (\theta) = 0$ can be tested against the 2-sided alternative with the Wald statistic

$$
W = \frac{\mathbf{h(\hat{\theta})'\{H(\hat{\theta})[F(\hat{\theta})'F(\hat{\theta})]^{-1}H(\hat{\theta})'\}^{-1}h(\hat{\theta})}}{s^2q} \sim F_{q,n-p}
$$

## The likelihood ratio test

$$
t_{LR} = 2[l(\hat{\theta})-l(\theta_0)] \sim \chi^2_v
$$

where v is the degree of freedom.

Compare the height of the log-likelihood of the sample estimate in relation to the height of log-likelihood of the hypothesized population parameter

Alternatively,

This test considers a ratio of two maximizations,

$$
\begin{aligned}
L_r &= \text{maximized value of the likelihood under $H_0$ (the reduced model)} \\
L_f &= \text{maximized value of the likelihood under $H_0 \cup H_a$ (the full model)}
\end{aligned}
$$

Then, the likelihood ratio is:

$$
\Lambda = \frac{L_r}{L_f}
$$

which can't exceed 1 (since $L_f$ is always at least as large as $L-r$ because $L_r$ is the result of a maximization under a restricted set of the parameter values).

The likelihood ratio statistic is:

$$
\begin{aligned}
-2ln(\Lambda) &= -2ln(L_r/L_f) = -2(l_r - l_f) \\
\lim_{n \to \infty}(-2ln(\Lambda)) &\sim \chi^2_v
\end{aligned}
$$

where $v$ is the number of parameters in the full model minus the number of parameters in the reduced model.

If $L_r$ is much smaller than $L_f$ (the likelihood ratio exceeds $\chi_{\alpha,v}^2$), then we reject he reduced model and accept the full model at $\alpha \times 100 \%$ significance level

## Lagrange Multiplier (Score) {#lagrange-multiplier-score}

$$
t_S= \frac{S(\theta_0)^2}{I(\theta_0)} \sim \chi^2_v
$$

where $v$ is the degree of freedom.

Compare the slope of the log-likelihood of the sample estimate in relation to the slope of the log-likelihood of the hypothesized population parameter

## Two One-Sided Tests (TOST) Equivalence Testing

This is a good way to test whether your population effect size is within a range of practical interest (e.g., if the effect size is 0).


```r
library(TOSTER)
```

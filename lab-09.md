Lab 09 - Grading the professor, Pt. 1
================
Lindley Slipetz
03/30/2021

### Load packages and data

``` r
#install.packages("vctrs")
#install.packages("tibble")
#install.packages("tidyverse")
library(tidyverse) 
#install.packages("broom")
#install.packages("tidymodels")
#install.packages("ggplot2")
library(tidymodels)
library(openintro)
```

Okay, loading tidymodels ended up being kind of a nightmare, but now
we’re ready to go.

### Exercise 1

``` r
ggplot(evals, aes(x=factor(score)))+
  geom_bar(stat="count", width=0.7, fill="black")+
  labs(x = "Score", y = "Count", title = "Distribution of teaching evaluation scores") +
  theme(axis.text.x = element_text(angle = 90))
```

![](lab-09_files/figure-gfm/score_bar-1.png)<!-- -->

``` r
evals %>%
  summarise(averageRating = mean(score),
             sdRating = sd(score))
```

    ## # A tibble: 1 x 2
    ##   averageRating sdRating
    ##           <dbl>    <dbl>
    ## 1          4.17    0.544

The distribution is positively skewed. Students tend to rate their
courses highly. The average score is 4.17 with a standard deviation of
0.54. I’m actually a bit surprised by this, but I would explain it by my
perception of the gender disparity in evaluations (of course, I’m going
to evaluate that later, so maybe this will change). Men tend to get high
scores and there’s more male professors than female professors, so that
would skew the distribution.

### Exercise 2

``` r
ggplot(evals, aes(x= bty_avg, y=score)) +
  geom_point() +
  labs(x = "Evaluation Score", y = "Beauty Score", title = "Evaluation score vs beauty score") 
```

![](lab-09_files/figure-gfm/scatter_1-1.png)<!-- -->

While it does seem true that, generally, higher beauty score is
associated with higher evaluations (because the higher beauty scores are
associated with higher evaluations), it’s not a strong association.

### Exercise 3

``` r
ggplot(evals, aes(x=bty_avg, y=score)) +
  geom_jitter() +
  labs(x = "Evaluation Score", y = "Beauty Score", title = "Evaluation score vs beauty score") 
```

![](lab-09_files/figure-gfm/jitter_1-1.png)<!-- -->

Jitter makes it so the points aren’t in straight lines. It makes it more
clear that there might be a linear relationship between evaluation score
and beauty score.

### Exercise 4

``` r
lm_model <- linear_reg() %>% 
            set_engine('lm') %>% 
            set_mode('regression')
lm_fit <- lm_model %>% 
          fit(score ~ bty_avg, data = evals)
lm_fit
```

    ## parsnip model object
    ## 
    ## Fit time:  20ms 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ bty_avg, data = data)
    ## 
    ## Coefficients:
    ## (Intercept)      bty_avg  
    ##     3.88034      0.06664

**Regression model**: score = 0.6664bty\_avg + 3.88034

\#\#\#Exercise 5

``` r
ggplot(evals, aes(x=bty_avg, y=score)) + 
  geom_jitter() +
  geom_smooth(method=lm, se=FALSE, color="orange") +
  labs(x = "Evaluation Score", y = "Beauty Score", title = "Evaluation score vs beauty score") 
```

    ## `geom_smooth()` using formula 'y ~ x'

![](lab-09_files/figure-gfm/lin_reg_graph-1.png)<!-- -->

### Exercise 6

A one unit change in score corresponds to a 0.6664 unit change in beauty
score average.

### Exercise 7

The mean of beauty score is 3.88034 when evaluation score is zero. I
don’t think this value is realistic in practice. I don’t think someone
would ever teach so bad as to get 0 from every student.

### Exercise 8

``` r
glance(lm_fit$fit)
```

    ## # A tibble: 1 x 12
    ##   r.squared adj.r.squared sigma statistic   p.value    df logLik   AIC   BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>     <dbl> <dbl>  <dbl> <dbl> <dbl>
    ## 1    0.0350        0.0329 0.535      16.7 0.0000508     1  -366.  738.  751.
    ## # ... with 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

\(R^2\) = 0.035, so the model explains 3.5% of the variance. It also
tells us that the residuals are quite large.

### Exercise 9

``` r
m_gen <- linear_reg() %>% 
            set_engine('lm') %>% 
            set_mode('regression')
m_gen_fit <- m_gen %>% 
          fit(score ~ gender, data = evals)
m_gen_fit
```

    ## parsnip model object
    ## 
    ## Fit time:  0ms 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ gender, data = data)
    ## 
    ## Coefficients:
    ## (Intercept)   gendermale  
    ##      4.0928       0.1415

The average difference between male and female scores is 0.1415. Female
is associated with a score of 4.0928.

### Exercise 10

The equation for females is score = 4.0928 + 0.1415gender. The equation
for male is score = 4.2343 + 0.1415gender.

### Exercise 11

``` r
m_rank <- linear_reg() %>% 
            set_engine('lm') %>% 
            set_mode('regression')
m_rank_fit <- m_rank %>% 
          fit(score ~ rank, data = evals)
m_rank_fit
```

    ## parsnip model object
    ## 
    ## Fit time:  0ms 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ rank, data = data)
    ## 
    ## Coefficients:
    ##      (Intercept)  ranktenure track       ranktenured  
    ##           4.2843           -0.1297           -0.1452

The equation is score = 4.2843 - 0.1297rank\_TT - 0.1452rank\_T. The
average score for teaching is 4.2843. As TT rank increases by one rank,
holding T constant, scores decrease by 0.1297. As T rank increases by
one rank, holding TT constant, scores decrease by 0.1452.

### Exercise 12

``` r
evals <- evals %>%
  mutate(rank_relevel = case_when(
    rank == "teaching" ~ -1,
    rank == "tenure track" ~ 0,
    rank == "tenured" ~ 1
  ))
```

### Exercise 13

``` r
m_rank_relevel <- linear_reg() %>% 
            set_engine('lm') %>% 
            set_mode('regression')
m_rank_relevel_fit <- m_rank_relevel %>% 
          fit(score ~ rank_relevel, data = evals)
m_rank_relevel_fit
```

    ## parsnip model object
    ## 
    ## Fit time:  0ms 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ rank_relevel, data = data)
    ## 
    ## Coefficients:
    ##  (Intercept)  rank_relevel  
    ##      4.19626      -0.06601

``` r
glance(m_rank_relevel_fit$fit)
```

    ## # A tibble: 1 x 12
    ##   r.squared adj.r.squared sigma statistic p.value    df logLik   AIC   BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>   <dbl> <dbl>  <dbl> <dbl> <dbl>
    ## 1   0.00975       0.00760 0.542      4.54  0.0337     1  -372.  750.  763.
    ## # ... with 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

The equation is score = 4.19626 - 0.06601rank\_relevel. When rank is
tenure track, the average score is 4.19626. Being tenured is associated
with lower scores. R^2 = 0.00975, so the model explains .975% of the
variance. Again we see the residuals are large.

### Exercise 14

``` r
evals <- evals %>%
  mutate(tenure_eligible = case_when(
    rank == "teaching" ~ "no",
    rank == "tenure track" ~ "yes",
    rank == "tenured" ~ "yes"
  ))
```

### Exercise 15

Fit a new linear model called m\_tenure\_eligible to predict average
professor evaluation score based on tenure\_eligibleness of the
professor. This is the new (regrouped) variable you created in Exercise
15. Based on the regression output, write the linear model and interpret
the slopes and intercept in context of the data. Also determine and
interpret the  
R2 of the model.

``` r
m_tenure_eligible <- linear_reg() %>% 
            set_engine('lm') %>% 
            set_mode('regression')
m_tenure_eligible_fit <- m_tenure_eligible %>% 
          fit(score ~ tenure_eligible, data = evals)
m_tenure_eligible_fit
```

    ## parsnip model object
    ## 
    ## Fit time:  0ms 
    ## 
    ## Call:
    ## stats::lm(formula = score ~ tenure_eligible, data = data)
    ## 
    ## Coefficients:
    ##        (Intercept)  tenure_eligibleyes  
    ##             4.2843             -0.1405

``` r
glance(m_tenure_eligible_fit$fit)
```

    ## # A tibble: 1 x 12
    ##   r.squared adj.r.squared sigma statistic p.value    df logLik   AIC   BIC
    ##       <dbl>         <dbl> <dbl>     <dbl>   <dbl> <dbl>  <dbl> <dbl> <dbl>
    ## 1    0.0115       0.00935 0.541      5.36  0.0210     1  -372.  750.  762.
    ## # ... with 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>

The equation is score = 4.2843 - 0.1405tenure\_eligible. For teaching
faculty, the average score is 4.2843. Being tenure eligible is
associated with a 0.1405 decrease in score. R^2 = 0.0115, meaning the
model explains 1.15% of the variance and again there are large
residuals.

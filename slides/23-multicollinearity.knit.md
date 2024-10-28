---
title: "Multicollinearity"
author: "Prof. Eric Friedlander"
date: "2024-10-30"
date-format: "MMM DD, YYYY"
footer: "[🔗 MAT 212 - Fall 2024 -  Schedule](https://mat212fa24.netlify.app/schedule)"
logo: "../images/logo.png"
format: 
  revealjs:
    theme: slides.scss
    multiplex: false
    transition: fade
    slide-number: false
    incremental: false 
    chalkboard: true
html-math-method:
  method: mathjax
  url: "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"
execute:
  freeze: auto
  echo: true
  cache: false
knitr:
  opts_chunk: 
    R.options:      
    width: 200
bibliography: references.bib
---




## First things first

-   Finish-up AE-16

## Announcements

-   Project: EDA Due Today
-   Project: [Paper Due November 18th](https://mat212fa24.netlify.app/project-instructions#draft-report)
-   Oral R Quiz

::: appex
📋 [AE 17 - Multicollinearity](https://mat212fa24.netlify.app/ae/ae-17-multicollinearity)

- Open up AE 17 and Complete Exercise 0
:::

## Topics

::: nonincremental
-   Defining Multicollinearity
-   Detecting Multicollinearity
-   Variance Inflation Factors
:::

## Computational setup


::: {.cell layout-align="center"}

```{.r .cell-code}
# load packages
library(tidyverse)
library(broom)
library(mosaic)
library(mosaicData)
library(patchwork)
library(knitr)
library(kableExtra)
library(scales)
library(countdown)
library(rms)

# set default theme and larger font size for ggplot2
ggplot2::theme_set(ggplot2::theme_minimal(base_size = 16))
```
:::



## Data: `rail_trail` {.smaller}

::: nonincremental
-   The Pioneer Valley Planning Commission (PVPC) collected data for ninety days from April 5, 2005 to November 15, 2005.
-   Data collectors set up a laser sensor, with breaks in the laser beam recording when a rail-trail user passed the data collection station.
:::


::: {.cell layout-align="center"}
::: {.cell-output .cell-output-stdout}

```
# A tibble: 90 × 7
   volume hightemp avgtemp season cloudcover precip day_type
    <dbl>    <dbl>   <dbl> <chr>       <dbl>  <dbl> <chr>   
 1    501       83    66.5 Summer       7.60 0      Weekday 
 2    419       73    61   Summer       6.30 0.290  Weekday 
 3    397       74    63   Spring       7.5  0.320  Weekday 
 4    385       95    78   Summer       2.60 0      Weekend 
 5    200       44    48   Spring      10    0.140  Weekday 
 6    375       69    61.5 Spring       6.60 0.0200 Weekday 
 7    417       66    52.5 Spring       2.40 0      Weekday 
 8    629       66    52   Spring       0    0      Weekend 
 9    533       80    67.5 Summer       3.80 0      Weekend 
10    547       79    62   Summer       4.10 0      Weekday 
# ℹ 80 more rows
```


:::
:::



Source: [Pioneer Valley Planning Commission](http://www.fvgreenway.org/pdfs/Northampton-Bikepath-Volume-Counts%20_05_LTA.pdf) via the **mosaicData** package.

## Full model


::: {.cell layout-align="center"}
::: {.cell-output-display}


|term            |   estimate| std.error|  statistic|   p.value|
|:---------------|----------:|---------:|----------:|---------:|
|(Intercept)     |  17.622161| 76.582860|  0.2301058| 0.8185826|
|hightemp        |   7.070528|  2.420523|  2.9210743| 0.0045045|
|avgtemp         |  -2.036685|  3.142113| -0.6481896| 0.5186733|
|seasonSpring    |  35.914983| 32.992762|  1.0885716| 0.2795319|
|seasonSummer    |  24.153571| 52.810486|  0.4573632| 0.6486195|
|cloudcover      |  -7.251776|  3.843071| -1.8869743| 0.0627025|
|precip          | -95.696525| 42.573359| -2.2478030| 0.0272735|
|day_typeWeekend |  35.903750| 22.429056|  1.6007696| 0.1132738|


:::
:::


# Multicollinearity

## What is multicollinearity

-   **Multicollinearity** is the case when one or more predictor variables are strongly correlated with some combination of other predictors
-   **Intuition:** if you could fit a good linear model with one of your predictors as the response and the rest of the predictors as your explanatory variables, then your predictors are exhibiting multicollinearity



## Example

Let's assume the true population regression equation is $y = 3 + 4x$

. . .

Suppose we try estimating that equation using a model with variables $x$ and $z = x/10$

$$
\begin{aligned}\hat{y}&= \hat{\beta}_0 + \hat{\beta}_1x  + \hat{\beta}_2z\\
&= \hat{\beta}_0 + \hat{\beta}_1x  + \hat{\beta}_2\frac{x}{10}\\
&= \hat{\beta}_0 + \bigg(\hat{\beta}_1 + \frac{\hat{\beta}_2}{10}\bigg)x
\end{aligned}
$$

## Example {.smaller}

$$\hat{y} = \hat{\beta}_0 + \bigg(\hat{\beta}_1 + \frac{\hat{\beta}_2}{10}\bigg)x$$

-   We can set $\hat{\beta}_1$ and $\hat{\beta}_2$ to any two numbers such that $\hat{\beta}_1 + \frac{\hat{\beta}_2}{10} = 4$

-   Therefore, we are unable to choose the "best" combination of $\hat{\beta}_1$ and $\hat{\beta}_2$

-   In statistics, we say this model is "unidentifiable" because different parameters combinations can result in the same model

-   This is also why we need to set a reference level for categorical variables

-   Complete Exercises 1-2.

## Why multicollinearity is a problem

-   When we have perfect collinearities, we are unable to get estimates for the coefficients

    -   When we have almost perfect collinearities (i.e. highly correlated predictor variables), the standard errors for our regression coefficients inflate

    -   In other words, we lose precision in our estimates of the regression coefficients

    -   This impedes our ability to use the model for inference

    -   It is also difficult to interpret the model coefficients

## Detecting Multicollinearity {.midi}

Multicollinearity may occur when...

-   There are very high correlations $(r > 0.9)$ among two or more predictor variables, especially when the sample size is small

-   One (or more) predictor variables is an almost perfect linear combination of the others

-   There are interactions between two or more continuous variables


## Detecting multicollinearity in the EDA

-   Look at a correlation matrix of the predictor variables, including all indicator variables
    -   Look out for values close to 1 or -1
-   Look at a scatter plot matrix of the predictor variables
    -   Look out for plots that show a relatively linear relationship
-   Complete Exercises 3-4.

## Detecting Multicollinearity (VIF)

**Variance Inflation Factor (VIF)**: Measure of multicollinearity in the regression model

$$VIF(\hat{\beta}_j) = \frac{1}{1-R^2_{X_j|X_{-j}}}$$

where $R^2_{X_j|X_{-j}}$ is the proportion of variation in $X_j$ that is explained by the linear combination of the other explanatory variables in the model.

## Detecting Multicollinearity (VIF)

-   Typically $VIF > 10$ indicates concerning multicollinearity

-   Variables with similar values of VIF are typically the ones correlated with each other

-   Use the `vif()` function in the **rms** R package to calculate VIF

## VIF for rail trail model

Complete Exercise 5.

. . .


::: {.cell layout-align="center"}

```{.r .cell-code}
vif(rt_full_fit)
```

::: {.cell-output .cell-output-stdout}

```
       hightemp         avgtemp    seasonSpring    seasonSummer      cloudcover 
      10.259978       13.086175        2.751577        5.841985        1.587485 
         precip day_typeWeekend 
       1.295352        1.125741 
```


:::
:::


<br>

. . .

`hightemp` and `avgtemp` are correlated. 

## What to do about Multicollinearity

1. Drop some predictors.
    - Example: Remove <u>**one**</u> of these variables and refit the model.
2. Combine some predictors.
    - Example: Create a new variable `temp_comsite` that is the average of `avgtemp` and `hightemp`.
3. Discount the individual coefficients and t-tests.
    - Example: Think about `avgtemp` and `hightemp` together with their individual $\beta$'s and p-values not having much meaning.
    
. . .

Complete Exercises 6 & 7.

## Model without `hightemp` {.smaller}


::: {.cell layout-align="center"}
::: {.cell-output-display}


|term            | estimate| std.error| statistic| p.value|
|:---------------|--------:|---------:|---------:|-------:|
|(Intercept)     |   76.071|    77.204|     0.985|   0.327|
|avgtemp         |    6.003|     1.583|     3.792|   0.000|
|seasonSpring    |   34.555|    34.454|     1.003|   0.319|
|seasonSummer    |   13.531|    55.024|     0.246|   0.806|
|cloudcover      |  -12.807|     3.488|    -3.672|   0.000|
|precip          | -110.736|    44.137|    -2.509|   0.014|
|day_typeWeekend |   48.420|    22.993|     2.106|   0.038|


:::
:::


## Model without `avgtemp` {.smaller}


::: {.cell layout-align="center"}
::: {.cell-output-display}


|term            | estimate| std.error| statistic| p.value|
|:---------------|--------:|---------:|---------:|-------:|
|(Intercept)     |    8.421|    74.992|     0.112|   0.911|
|hightemp        |    5.696|     1.164|     4.895|   0.000|
|seasonSpring    |   31.239|    32.082|     0.974|   0.333|
|seasonSummer    |    9.424|    47.504|     0.198|   0.843|
|cloudcover      |   -8.353|     3.435|    -2.431|   0.017|
|precip          |  -98.904|    42.137|    -2.347|   0.021|
|day_typeWeekend |   37.062|    22.280|     1.663|   0.100|


:::
:::


## Model without `temp_composite` {.smaller}


::: {.cell layout-align="center"}
::: {.cell-output-display}


|term            | estimate| std.error| statistic| p.value|
|:---------------|--------:|---------:|---------:|-------:|
|(Intercept)     |   18.823|    77.430|     0.243|   0.809|
|seasonSpring    |   28.458|    33.059|     0.861|   0.392|
|seasonSummer    |   -0.986|    51.234|    -0.019|   0.985|
|cloudcover      |  -10.367|     3.409|    -3.041|   0.003|
|precip          | -104.475|    42.725|    -2.445|   0.017|
|day_typeWeekend |   40.914|    22.479|     1.820|   0.072|
|temp_composite  |    6.292|     1.376|     4.571|   0.000|


:::
:::


## Choosing a model {.midi}

Model without `hightemp`:


::: {.cell layout-align="center"}
::: {.cell-output-display}


| adj.r.squared|    AIC|    BIC|
|-------------:|------:|------:|
|          0.42| 1087.5| 1107.5|


:::
:::


Model without `avgtemp`:


::: {.cell layout-align="center"}
::: {.cell-output-display}


| adj.r.squared|     AIC|     BIC|
|-------------:|-------:|-------:|
|          0.47| 1079.05| 1099.05|


:::
:::


Model with `temp_composite`:


::: {.cell layout-align="center"}
::: {.cell-output-display}


| adj.r.squared|     AIC|     BIC|
|-------------:|-------:|-------:|
|          0.46| 1081.67| 1101.67|


:::
:::



. . .

Based on Adjusted $R^2$, AIC, and BIC, the model without **avgtemp** is a better fit. Therefore, we choose to remove **avgtemp** from the model and leave **hightemp** in the model to deal with the multicollinearity.

## Selected model (for now)


::: {.cell layout-align="center"}
::: {.cell-output-display}


|term            | estimate| std.error| statistic| p.value|
|:---------------|--------:|---------:|---------:|-------:|
|(Intercept)     |    8.421|    74.992|     0.112|   0.911|
|hightemp        |    5.696|     1.164|     4.895|   0.000|
|seasonSpring    |   31.239|    32.082|     0.974|   0.333|
|seasonSummer    |    9.424|    47.504|     0.198|   0.843|
|cloudcover      |   -8.353|     3.435|    -2.431|   0.017|
|precip          |  -98.904|    42.137|    -2.347|   0.021|
|day_typeWeekend |   37.062|    22.280|     1.663|   0.100|


:::
:::


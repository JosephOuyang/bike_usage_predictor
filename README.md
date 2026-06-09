# Pedaling Patterns and Factors Influencing Casual Bike Ridership
**Author:** Joseph Ouyang

---

Bike sharing programs have become a practical tool for addressing urban congestion and pollution, now operating in hundreds of cities globally. This project analyzes Capital Bikeshare ridership data from the Washington D.C./Arlington, VA/MD area to identify environmental factors that influence hourly casual bike usage, and constructs a multiple linear regression model to predict ridership.

---

## Table of Contents
- [Dataset](#dataset)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Modeling](#modeling)
- [Prediction](#prediction)
- [Discussion](#discussion)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)

---

## Dataset

The dataset contains a random sample of 656 hourly observations from Capital Bikeshare records. Each row represents one hour block of ridership activity along with weather and environmental conditions recorded at that time.

### Variables

| Variable | Type | Description |
|---|---|---|
| `Casual` | Response | Number of casual bike users in the hour block |
| `Weather` | Categorical | Weather type: clear, misty, or rain/snow |
| `Temp` | Quantitative | Temperature scaled as a proportion of the dataset's overall maximum, from 0 to 1 |
| `Windspeed` | Quantitative | Windspeed scaled as a proportion of the dataset's overall maximum, from 0 to 1 |

### Numerical Summaries

| Variable | Min | Q1 | Median | Mean | Q3 | Max |
|---|---|---|---|---|---|---|
| Casual | 0 | 2 | 8 | 11.51 | 20 | 39 |
| Temp | 0.02 | 0.30 | 0.44 | 0.4429 | 0.585 | 0.94 |
| Windspeed | 0.00 | 0.1045 | 0.1642 | 0.1840 | 0.2537 | 0.7164 |

Weather counts: 387 clear hours, 197 misty hours, 72 rain/snow hours.

---

## Exploratory Data Analysis

### Univariate EDA

- **Casual**: right skewed and unimodal, with a peak of roughly 275 hour blocks at 0 to 5 users and a long right tail up to 39
- **Weather**: majority of observations occur under clear conditions, with misty and rain/snow hours less frequent
- **Temp**: approximately symmetric and unimodal, centered around 0.44 with an IQR of 0.285
- **Windspeed**: right skewed and unimodal, centered around 0.1642 with a secondary smaller peak around 0.30

### Variable Transformations

Both `Casual` and `Windspeed` are right skewed and receive log transformations before bivariate analysis and modeling.

```r
bikes$log.casual    <- log(bikes$Casual + 1.1)
bikes$log.windspeed <- log(bikes$Windspeed + 1.1)
```

After transformation, `log(Casual)` becomes approximately normal. `log(Windspeed)` retains a slight right skew, but the transformation meaningfully reduces it and better satisfies linear regression assumptions.

### Bivariate EDA

- **Weather vs. log(Casual)**: rain/snow hours have a noticeably lower median log ridership compared to clear and misty hours, which are similar to each other; worsening weather is associated with fewer casual riders
- **Temp vs. log(Casual)**: a positive association is visible, with higher temperatures corresponding to more casual riders on average
- **log(Windspeed) vs. log(Casual)**: no clear linear pattern; points appear randomly distributed, suggesting no relationship or a very weak one at best

---

## Modeling

### Model Selection

The final model predicts `log(Casual)` from `Weather`, `Temp`, and `log(Windspeed)` using multiple linear regression. Although `log(Windspeed)` showed no clear visual relationship with the response in EDA, it was statistically significant at the 5% level and improved adjusted R-squared when combined with the other predictors, so it was retained.

Two alternative models were considered and rejected:

- **Interaction terms** between Weather and Temp, and between Weather and log(Windspeed): adjusted R-squared increased by about 1%, but no interaction terms reached statistical significance at the 5% level
- **Quadratic term for Temp**: improved adjusted R-squared and better captured curvature in the temperature-ridership relationship, but was excluded to satisfy the project requirement for a linear regression model

### Multicollinearity Check

Variance inflation factors for all predictors are well below the threshold of 2.5, confirming no multicollinearity concerns.

| Variable | GVIF | Df | GVIF^(1/(2*Df)) |
|---|---|---|---|
| Weather | 1.007 | 2 | 1.002 |
| Temp | 1.020 | 1 | 1.010 |
| log.windspeed | 1.019 | 1 | 1.009 |

### Regression Assumptions

The residuals vs. fitted plot shows no strong pattern, with roughly constant spread above and below zero across fitted values. The mean zero and constant spread assumptions are reasonably satisfied. The Q-Q plot shows residuals closely following the diagonal line with minor tail deviations, which do not invalidate the normality assumption given the large sample size.

### Regression Results

```
lm(log.casual ~ Weather + Temp + log.windspeed, data = bikes)
```

| Coefficient | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| Intercept | 0.62812 | 0.17313 | 3.628 | 0.000308 |
| Weather: misty | 0.01105 | 0.09293 | 0.119 | 0.905365 |
| Weather: rain/snow | -0.41753 | 0.13630 | -3.063 | 0.002279 |
| Temp | 2.27067 | 0.23822 | 9.532 | < 2e-16 |
| log(Windspeed) | 1.69646 | 0.45589 | 3.721 | 0.000215 |

- Adjusted R-squared: 0.1377
- F-statistic: 27.14 on 4 and 651 degrees of freedom, p-value < 2.2e-16

### Interpretation

All predictors except misty weather are statistically significant at the 5% level. Key takeaways:

- **Rain/snow weather** is associated with lower casual ridership on average relative to clear weather, consistent with the EDA boxplots
- **Temperature** has a strong positive association with ridership; a one-unit increase in scaled temperature corresponds to an increase of 2.271 in log casual ridership
- **log(Windspeed)** has a positive coefficient of 1.696, suggesting higher windspeed hours are weakly associated with slightly higher ridership in this dataset
- **Misty weather** shows no statistically significant difference from clear weather in ridership

The low adjusted R-squared of 0.1377 is expected for behavioral data of this kind. Ridership is influenced by many unobserved factors such as time of day, day of week, proximity to stations, and personal motivations that fall outside the scope of this model.

---

## Prediction

Using the fitted model, we predict casual ridership for an hour with misty weather, a scaled temperature of 0.75, and a scaled windspeed of 0.25.

```r
log_pred <- 0.62812 + 0.01105 + 2.27067 * 0.75 + 1.69646 * log(0.25 + 1.1)
# log_pred = 2.851288

exp(2.851288) - 1.1
# predicted casual riders = 16.21
```

The predicted count of 16.21 casual riders falls between the mean of 11.51 and the Q3 of 20.00, which is a reasonable outcome given the moderately warm temperature and light wind conditions.

---

## Discussion

Weather, temperature, and windspeed are all meaningfully associated with casual hourly ridership. Hours with clear or misty weather, higher temperatures, and faster windspeeds tend to see more casual bike users on average.

Two areas warrant further investigation. First, the similarity in ridership between clear and misty weather is counterintuitive and could reflect that casual riders are less deterred by light cloud cover than expected. Second, the residual plot hints at a possible non-linear relationship between temperature and ridership, which could level off or reverse at very high temperatures. Identifying the temperature threshold where ridership gains diminish could provide actionable insights for bike-share operators managing fleet availability.

---

## Tech Stack

| Tool | Use |
|---|---|
| R | Primary analysis language |
| stats | Linear regression via lm() |
| car | Variance inflation factors via vif() |
| dplyr, readr | Data wrangling and CSV ingestion |
| knitr, kableExtra, pander | Report generation and table formatting |

---

## Repository Structure

```
bikesharing-ridership/
├── BikeSharing.Rmd                  # R Markdown source file
├── FinalBikeSharingAnalysis.pdf     # compiled writeup
├── bikes.csv                        # dataset (656 hourly observations)
└── README.md
```

---

*Completed for 36-202 Methods for Statistics and Data Science, Carnegie Mellon University, October 2025.*

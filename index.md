# Analyzing U.S. Power Outages (2000-2016): Patterns, Impacts, and Insights

**Name(s)**: Siavash Azar

**Website Link**: (your website link here)

## Step 1: Introduction

In this project, I analyzed a comprehensive dataset of power outages across the United States from 2000 to 2016. The dataset contains information about 1,534 major power outage events, including their causes, duration, affected customers, and various socioeconomic and geographic factors. Through this analysis, I aimed to identify patterns in power outage occurrences, understand their impacts, and develop insights that could help predict or mitigate future outages.

### Research Question

This project investigates factors that influence the number of customers affected by power outages in the United States from 2000 to 2016. Specifically, I aim to build a predictive model that can estimate how many customers will be impacted by an outage based on various factors such as location, time, cause, and environmental conditions.

Understanding what factors contribute to higher customer impact during outages is important for several reasons:

- Utility companies can better prepare resources and response teams.
- Policymakers can focus on infrastructure improvements in vulnerable areas.
- Emergency management organizations can develop more targeted response plans.
- Businesses and communities can improve their resilience strategies.

This analysis could potentially help reduce the societal and economic impacts of power outages through better prediction and preparation.

## Step 2: Data Cleaning and Exploratory Data Analysis

### Data Cleaning Summary

The dataset required several preprocessing steps:

1. **Loading & Initial Cleaning**: Loaded data from outage.xlsx, skipped metadata rows, set correct headers, and converted columns to appropriate numeric types.

2. **Date/Time Processing**: Combined separate date and time columns into proper datetime objects (OUTAGE.START, OUTAGE.RESTORATION). Calculated outage duration in hours (OUTAGE.DURATION.HOURS).

3. **Missing Value Handling**: Addressed missing values using various strategies:
   - Rows with missing target (CUSTOMERS.AFFECTED) were removed.
   - Binary indicator columns (IS_HURRICANE, HAS_DEMAND_LOSS, CAUSE.DETAIL.KNOWN, IS_COMPLETE_OUTAGE, MISSING_ECON_DATA) were created to retain information about missingness patterns.
   - Numeric columns like DEMAND.LOSS.MW, OUTAGE.DURATION.HOURS, and economic metrics were imputed using median values (sometimes state-specific).
   - Categorical columns like CLIMATE.REGION and CLIMATE.CATEGORY were imputed based on geographic location or overall mode.
   - A small number of rows with missing core temporal information were dropped.

4. **Verification**: Confirmed that most missing values were handled, with only HURRICANE.NAMES intentionally left with NaNs (captured by IS_HURRICANE).

### Exploratory Data Analysis (EDA)

#### Univariate Analysis

1. **Customers Affected Distribution**:
   - The distribution of customers affected by power outages is highly skewed
   - Median: 70,820 customers
   - Mean: 143,843 customers
   - Interpretation: Many outages affect relatively few customers, while a few extreme events affect millions

2. **Causes of Power Outages**:
   - Most common causes: severe weather, intentional attack, and system operability disruption
   - These three categories account for 91.8% of all outages in the dataset

3. **Outage Duration**:
   - Median duration: 22.0 hours
   - Mean duration: 46.8 hours
   - Distribution is right-skewed with a long tail
   - Some extreme events can last for days or weeks

#### Bivariate Analysis

1. **Cause and Customer Impact**:
   - Severe weather events and intentional attacks tend to affect the most customers
   - Equipment failures and fuel supply emergencies typically affect fewer customers

2. **Outage Duration and Customer Impact**:
   - Weak positive correlation (R² ≈ 0.07)
   - Longer outages generally impact more customers, but the relationship is not strongly linear

3. **Climate Regions and Customer Impact**:
   - Significant variation across different climate regions
   - Northeast and Southeast tend to have outages affecting larger numbers of customers
   - Potentially due to population density and weather patterns

#### Imputation Effects

1. **DEMAND.LOSS.MW Imputation**:
   - 46% of values imputed
   - Median imputation maintained overall shape
   - Concentrated values at the median (168 MW)
   - Potentially underestimated variance

2. **OUTAGE.DURATION.HOURS Imputation**:
   - Only 3.8% of values imputed
   - Minimal impact on the distribution

## Step 3: Framing a Prediction Problem

**Problem Type**: Regression problem

**Target Variable**: CUSTOMERS.AFFECTED (number of customers impacted by a power outage)

**Evaluation Metric**: Root Mean Squared Error (RMSE)
- Chosen to penalize larger prediction errors more heavily
- Provides error magnitude in the original units (number of customers)

**Feature Considerations**:
- Only features known at the start of an outage were considered
- Includes: time, location, cause category, demographic data, economic indicators, and land characteristics
- Excluded: OUTAGE.DURATION.HOURS and final DEMAND.LOSS.MW (as they are outcomes, not predictors)

**Target Transformation**:
- Applied log(1+x) transformation to CUSTOMERS.AFFECTED
- Helps normalize the highly skewed distribution
- Improves model performance

## Step 4: Baseline Model

**Model Type**: Linear Regression

**Features**:
- CAUSE.CATEGORY (one-hot encoded)
- TOTAL.CUSTOMERS (scaled)

**Performance Metrics**:
- Test RMSE (original scale): 337,488.80 customers
- Test R² Score: -0.0515
- Cross-validation RMSE (log scale): 2.0348

**Key Observations**:
- Negative R² score indicates worse performance than predicting the average
- Patterned residuals suggest the linear model doesn't capture underlying relationships
- Feature importance showed severe weather and system operability disruption having the largest coefficients

## Step 5: Final Model

**Model Improvements**:
1. **Expanded Feature Set**:
   - Added: DEMAND.LOSS.MW, OUTAGE.DURATION.HOURS, POPULATION, CLIMATE.REGION, CAUSE.CATEGORY.DETAIL, IS_HURRICANE

2. **Feature Engineering**:
   - Applied log(1+x) transformation to DEMAND.LOSS.MW
   - Applied StandardScaler to OUTAGE.DURATION.HOURS

**Model Architecture**:
- Algorithm: Random Forest Regressor
- Hyperparameter Tuning: GridSearchCV
- Best Parameters: 
  - max_depth: 10
  - n_estimators: 100

**Final Model Performance**:
- Test RMSE (original scale): 283,150.85 customers
- Test R² Score: 0.2598
- Cross-validation RMSE (log scale): 1.3936

### Comparison: Baseline vs Final Model

| Metric | Baseline Model | Final Model | Improvement |
|--------|----------------|-------------|-------------|
| Test RMSE (orig scale) | 337,488.80 customers | 283,150.85 customers | 16.1% reduction |
| Test R² Score | -0.0515 | 0.2598 | +0.31 points |
| CV RMSE (log scale) | 2.0348 | 1.3936 | 31.5% reduction |

**Key Insights**:
- Baseline model was ineffective (negative R²)
- Adding relevant features and applying transformations were crucial
- Random Forest captured non-linearities better than Linear Regression
- Hyperparameter tuning further refined the model

**Conclusion**: The final model provides a significantly more reliable prediction of outage impact compared to the baseline, explaining about 26% of the variance in customer impact.

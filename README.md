# California Property Close Price Prediction (IDX-Exchange LLC)


## Project Overview

This project develops an Automated Valuation Model (AVM) to estimate the final sale price (ClosePrice) of single-family residential properties in California.

The goal is to build a model that can estimate property value for both on-market and off-market properties, similar in concept to Zillow's Zestimate. Therefore, the predictive features are limited to information that would reasonably be available for a property regardless of whether it is currently listed for sale.

The project progresses from data preprocessing and exploratory data analysis to baseline linear regression, tree-based ensemble models, gradient boosting, and quantile regression for prediction uncertainty.

---

## Data Access and Preparation

### Data Source

- **Provider** : California Regional Multiple Listing Service (CRMLS)
- **Data Access** : FTP server provided through IDX Exchange
- **Property Type** : Residential
- **Property Subtype** : Single Family Residence


## Data Requirements

- **Filter only for:**
    - `PropertyType = "Residential"`
    - `PropertySubType = "SingleFamilyResidence"`
- **Exclude:**
    - Any features or columns that would not be available for a property currently *not* for sale (e.g., `ListPrice`, MLS listing-only fields).
      
    - Fields with all missing values.

---


## Task Specification


### 1. Data Preprocessing

file : Data_Preprocessing_idx

The first stage focused on cleaning and preparing property, city, and geographic features for modeling.

Preprocessing included:

- Identifying invalid and missing observations
- Preparing city, county, and geographic variables
- Cleaning numerical and categorical property features
- Combining separately processed feature groups into a unified dataset
- Creating consistent variables for downstream exploratory analysis and modeling

Only features considered appropriate for property valuation were retained.


### 2. Consolidating Preprocessed Features

file : Cleaning_dummies_data

After preprocessing work was completed across different feature groups, the processed data were combined into a unified modeling dataset.

Several features originally represented through large groups of dummy variables were reconstructed into more interpretable categorical variables for exploratory analysis.

Examples included:

- Flooring type
- School district

This step made the dataset easier to analyze and visualize while preserving the processed information generated during feature engineering


### 3. Data Exploration

file : Explanatory_Data_Analysis_idx

The EDA used the final cleaned modeling sample rather than the original raw CRMLS dataset.

- Inspect data structure and feature availability.
- Identify and select only those features that an end user could provide or research, such as:
    - Square footage (living area)
    - Lot size
    - Address (city, zip code, etc.)
    - Number of bedrooms/bathrooms
    - Year built
    - Other public property attributes


The EDA showed that California property prices were strongly right-skewed and that medians were generally more representative than means for describing prices.

Living area and bathroom count showed some of the strongest numerical relationships with closing price, while geographic location produced substantial price variation.


### 4. Model

#### 1) Linear Regression

file : **Regression_Model**
- Linear regression was used as the initial modeling baseline. Both the original close-price scale and a log-transformed close-price target were evaluated.

- Linear model diagnostics
  
  - Multicollinearity

    Strong multicollinearity was identified among several predictors, particularly:
    
    - CPI
    - Unemployment Rate
    - Federal Interest Rate
    - 30-Year Mortgage Rate
    - Latitude
    - Longitude
    
    High VIF values indicate that some predictors contain substantial overlapping information.
    
    Because of this, individual OLS coefficients, standard errors, p-values, and confidence intervals should be interpreted cautiously.

    
  - Residual Normality
    
    The Jarque-Bera test rejected the null hypothesis of normally distributed residuals, and the Q-Q plot showed noticeable deviations in the tails.

    
  - Homoscedasticity
    
    The Scale-Location plot showed changing residual spread across fitted values, indicating that the constant-variance assumption was not fully satisfied.
    

  - Independence
    
    The Durbin-Watson statistic was approximately 1.995, suggesting little evidence of serious residual autocorrelation.
    
    Overall, the linear model provided a useful and interpretable baseline, but the diagnostics indicated limitations in using a purely linear framework for property valuation.


#### 2) Bagging and Boosting 

file : **Bagging_Boosting_Model2**

- Random Forest Regression

    A Random Forest Regressor was developed as the primary bagging model.
    
    Random Forest was selected because it can model:
    
    - Nonlinear relationships
    - Feature interactions
    - Complex threshold effects
    - Heterogeneous relationships across properties
    
    Hyperparameter tuning was performed in stages using cross-validation.
    
    Tree complexity, sampling settings, number of trees, and other regularization-related parameters were evaluated progressively.
    
    The final Random Forest model achieved approximately:
    
    - MAPE: 12%
    - MAE: $137,411
    - Test R²: 86.6%
    
    The model generalized substantially better than the linear regression baseline, although some train-test performance gap remained.
    
    Prediction errors also increased for more expensive properties, with the model tending to underpredict some high-value homes.
    

- XGBoost Regression
  
    An XGBoost Regressor was then developed as the primary boosting model.
    
    Hyperparameters were tuned progressively rather than through one very large grid search.
    
    The tuning stages investigated:
    
    - Tree depth
    - Minimum child weight
    - Sampling parameters
    - Regularization
    - Learning rate
    - Number of estimators
    
    The final XGBoost model achieved approximately:
    
    - MAPE: 11.5%
    - MAE: $129,701
    - RMSE: $245,917
    - Test R²: 88.6%
    
    Among the point-prediction models, XGBoost provided the strongest overall predictive performance.
    
    The model performed particularly well for lower- and middle-priced properties, while prediction errors increased for high-priced properties.

  
### 4. Model Training
- Train selected models using training data.
- Tune hyperparameters for optimal performance.

### 5. Model Evaluation
- Evaluate using appropriate metrics (e.g., R-squared) on the test set.
- Document model performance and areas for improvement.

### 6. Prediction
- Use the trained model to predict the close price for any property given user-supplied features.
- Ensure the model can be integrated into a web application, with required inputs matching those a user can provide.

### 7. Documentation
- Document all steps: exploration, preprocessing, modeling, evaluation, and prediction.
- Provide clear reasoning for all decisions.
- Prepare a live presentation summarizing findings, model performance, and the prediction process.

---

## Deliverables

- **Python Script:** End-to-end code for preprocessing, training, evaluation, and prediction.
- **Documentation:** Detailed write-up of the workflow, findings, and rationale.
- **Presentation:** Slides and/or talking points for a live Zoom presentation to stakeholders.

---

## Additional Guidelines

- Code must be clean, well-organized, and reproducible.
- Seek feedback from team members or domain experts for validation.
- Experiment with features, models, and hyperparameters to maximize prediction accuracy.
- Ensure that all model-required features are available to the end user at prediction time.

---

## Project Goal

To build a robust, user-focused machine learning model that accurately predicts the close price of any single-family residential property in California using only features available to consumers—enabling integration into a web application for real-time property value estimation.


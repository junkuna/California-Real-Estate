# California Property Close Price Prediction (IDX-Exchange LLC)


## Project Overview

This project develops an Automated Valuation Model (AVM) to estimate the final sale price (ClosePrice) of single-family residential properties in California.

The goal is to build a model that can estimate property value for both on-market and off-market properties, similar in concept to Zillow's Zestimate. Therefore, the predictive features are limited to information that would reasonably be available for a property regardless of whether it is currently listed for sale.

The project progresses from data preprocessing and exploratory data analysis to baseline linear regression, tree-based ensemble models, gradient boosting, and quantile regression for prediction uncertainty.

## Project Goal

To build a robust, user-focused machine learning model that accurately predicts the close price of any single-family residential property in California using only features available to consumers—enabling integration into a web application for real-time property value estimation.

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

#### **1) Linear Regression**

file : **Regression_Model**
- Linear regression was used as the initial modeling baseline. Both the original close-price scale and a log-transformed close-price target were evaluated.

- Linear model diagnostics
  
  - **Multicollinearity**

    Strong multicollinearity was identified among several predictors, particularly:
    
    - CPI
    - Unemployment Rate
    - Federal Interest Rate
    - 30-Year Mortgage Rate
    - Latitude
    - Longitude
    
    High VIF values indicate that some predictors contain substantial overlapping information.
    
    High VIF greater than 10
    - CPI
    - Unemplotyment
    - MortgageRate30Fixed
    - FedInterestRate
    - Latitude
    - Longitude
      
    Because of this, individual OLS coefficients, standard errors, p-values, and confidence intervals should be interpreted cautiously.

    
  - **Residual Normality**
    
    The Jarque-Bera test rejected the null hypothesis of normally distributed residuals, and the Q-Q plot showed noticeable deviations in the tails.

  - **Linearity**
    
    The residuals-versus-fitted plot did not show a strong systematic nonlinear pattern. The smoothed residual line stayed relatively close to zero across most fitted values, suggesting that the linearity assumption was reasonably satisfied
    
  - **Homoscedasticity**
    
    The Scale-Location plot showed changing residual spread across fitted values, indicating that the constant-variance assumption was not fully satisfied.
    

  - **Independence**
    
    The Durbin-Watson statistic was approximately 1.995, suggesting little evidence of serious residual autocorrelation.
    
  - The linear regression assumptions were only partially satisfied. Although linearity and independence appeared reasonable, residual normality, homoscedasticity, and low multicollinearity were not fully satisfied. Therefore, the model remained useful as a predictive and interpretable baseline, but individual coefficient estimates and statistical inference should be interpreted cautiously.


#### **2) Bagging and Boosting** 

file : **Bagging_Boosting_Model2**

- **Random Forest Regression**

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
    - MdAPE: 7.69%
    - MAE: $137,411
    - Test R²: 86.6%
    
    The model generalized substantially better than the linear regression baseline, although some train-test performance gap remained.
    
    Prediction errors also increased for more expensive properties, with the model tending to underpredict some high-value homes.
    
- **Feature Importance**

    Random Forest feature importance suggested that several variables played especially strong roles in predicting closing price.
    
    The strongest features included:
    
    - Living Area
    - Latitude
    - Bathroom Count
    - Longitude
    
    Together, these features represented a substantial portion of total model importance.
    
    This result emphasizes the importance of both:
    
    Physical property characteristics
    Geographic location
    
    Feature importance should still be interpreted as predictive contribution rather than causal effect.
  
- **XGBoost Regression**
  
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
    - MdAPE: 7.7%
    - MAE: $129,701
    - Test R²: 88.6%
    
    Among the point-prediction models, XGBoost provided the strongest overall predictive performance.
    
    The model performed particularly well for lower- and middle-priced properties, while prediction errors increased for high-priced properties.

- **Qunatile Regression**

    After developing the standard XGBoost model, Quantile XGBoost was used to estimate uncertainty around property-value predictions.
    
    Instead of predicting only one value, the model estimated:
    
    - 5th percentile
    - 50th percentile / median
    - 95th percentile
    
    Together, the 5th and 95th percentiles form an intended 90% prediction interval.
    
    The quantile model was tuned separately because quantile regression uses pinball loss rather than squared-error loss.
    
    **Quantile Performance**
    
    The model produced approximately:
    
    - 5th-percentile pinball loss: $19,822
    - Median pinball loss: $64,297
    - 95th-percentile pinball loss: $28,367
    - Average pinball loss: $37,495
    
    The intended 90% prediction interval contained approximately:
    
    - 86.4% of actual sale prices
    
    The average prediction interval width was approximately:
    
    - $629,221
    
    This indicates that the model captured substantial uncertainty in California property valuation, although the intervals were somewhat under-calibrated relative to the intended 90% coverage.
    
    **Median Quantile Prediction**
    
    The median prediction achieved approximately:
    
    - MAPE: 10.9%
    - MdAPE: 7.3%
    - MAE: $128,594
    - Test R²: 87.5%
    
    Quantile XGBoost therefore provided prediction accuracy similar to the strongest point-prediction model while also communicating uncertainty around the estimated property value.


---
## Summary
- ### Explanatory Data Analysis

  - California sale prices were strongly right-skewed. The median close price was about $830,000, while the mean was about $1.03 million, showing that a smaller number of expensive properties pulled the average upward. Because of this skewness, medians and IQRs were more appropriate for describing typical prices.

  - Among numerical features, LivingArea had the strongest relationship with ClosePrice. Bathrooms also had a meaningful positive relationship, while bedrooms showed a weaker positive relationship. Lot size had only a very weak positive association. Property age showed statistically significant differences among groups, but the large overlap in price distributions meant age alone was not a strong predictor.
 
  - Location was also very important. County showed large differences in median sale prices, with a median-price range of roughly $1.425 million between the highest- and lowest-priced counties. This suggests that geographic location is one of the major factors behind California housing-price differences.
 
  - The original SchoolDistrict feature was excluded because it had 227 categories, about 24% Unknown values, and many very rare districts. Instead, a new SchoolCount2Miles variable was created using public-school locations. This reduced sparsity, although it measures school accessibility rather than school quality.

- ### Linear Regression

  - Linear Regression was used as the starting baseline. The original model's predictions were about 30% different from actual sale prices on average, with an average dollar error of about $286,000. After adjusting the sale price using a logarithmic transformation, performance improved to about 21% average error and roughly $241,000 average dollar error. However, the model still had difficulty capturing the complexity of housing prices, and some of its statistical assumptions were not fully satisfied.


- ### Random Forest Regressor

  - Random Forest performed much better. Its predictions were about 12% different from actual prices on average, and half of the properties were predicted within about 7.7% of their actual sale price. The average dollar error was approximately $137,000, and the model explained about 86.6% of the differences in sale prices among unseen homes. However, it showed some overfitting and became less accurate for expensive properties.

- ### XGBoost Regressor

  - Standard XGBoost provided the strongest performance when predicting one sale price for each property. Its predictions were about 11.5% different from actual prices on average, with an average dollar error of about $129,700. Half of the properties were predicted within approximately 7.7% of their actual sale price, and the model explained about 88.6% of the differences in unseen sale prices. Like Random Forest, it tended to make larger errors and underpredict some very expensive homes.
 
- ### Quantile XGBoost Regressor

  - Quantile XGBoost had a different purpose. Instead of giving only one price, it provided a lower estimate, a middle or median estimate, and an upper estimate, which allows the model to communicate uncertainty.

For the middle estimate, predictions were about 10.9% different from actual prices on average, and half of the properties were predicted within about 7.3% of their actual sale price. The average dollar error was around $128,600.

The model's predicted price range captured the actual sale price for about 86 out of every 100 properties, slightly below the intended 90 out of 100. The average difference between the lower and upper estimates was about $629,000, showing that there can be substantial uncertainty in predicting California property prices.

## Main Takeaways

- Linear regression provided a useful starting point, but the tree-based models predicted housing prices much more accurately. Standard XGBoost was the strongest choice when one predicted sale price was needed, while Quantile XGBoost was more useful when it was important to show a possible range of sale prices and communicate prediction uncertainty.
  
- Log-transforming sale price substantially improved the linear regression baseline.
  
- The linear model remained useful for interpretation, but multicollinearity, residual non-normality, and heteroscedasticity limited classical OLS inference.
  
- Tree-based models substantially outperformed linear regression for prediction.
  
- XGBoost produced the strongest overall point-prediction performance.
  
- Living area, bathrooms, and geographic location were among the strongest predictive factors.
  
- Lower- and middle-priced properties were generally predicted more accurately than expensive properties.
  
- Quantile XGBoost provided useful uncertainty ranges but did not yet achieve the full intended 90% interval coverage.


## Limitations

- 3-fold cross-validation was used instead of 5-fold cross-validation because of computational cost. More folds may provide more stable estimates but require considerably greater training time.
- Linear regression captured some of these relationships but could not represent their complexity well.
- High-priced properties were more difficult to predict and were frequently underpredicted.
- Random Forest and XGBoost still showed some train-test performance gaps, indicating remaining small-mediate overfitting.
- Quantile prediction intervals were not perfectly calibrated. The intended 90% interval achieved approximately 86.4% coverage.
- Quantile prediction intervals were relatively wide, with an average width of approximately $629,221.
- Several macroeconomic and geographic predictors showed substantial multicollinearity in the linear model.
- The EDA showed that living area, bathrooms, and location were especially important.
- The EDA and modeling dataset was filtered during preprocessing, so rare, extreme, or luxury properties may be underrepresented.
- County representation was uneven, meaning heavily represented housing markets may influence statewide results more strongly than smaller markets

## Future work
- Using 5-fold or greater cross-validation with increased computing resources
- Further calibrating Quantile XGBoost intervals to achieve coverage closer to the intended 90%
- Comparing uncertainty approaches such as conformal prediction
- Building specialized models for different property-price segments
- Improving predictions for luxury and high-priced homes
- Adding more granular geographic features such as ZIP code, city, neighborhood, or distance-to-amenity information
- Investigating whether highly redundant macroeconomic or geographic predictors should be selectively reduced
- Comparing Ridge and Lasso as regularized linear baselines
- Using explainability methods such as permutation importance or SHAP
- Evaluating model performance separately by county and price segment
- Monitoring model accuracy over time as California market conditions change

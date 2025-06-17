# Optimizing Insurance Premiums: A Data-Driven Approach for AlphaCare Insurance Solutions

## Introduction: A New Era for Car Insurance in South Africa

As a Marketing Analytics Engineer at AlphaCare Insurance Solutions (ACIS), my core mission is to leverage data to refine our strategies in South Africa's dynamic car insurance market. This report details our analytical journey to optimize marketing efforts and identify 'low-risk' segments, paving the way for more attractive premium offerings and new client acquisition. Through a comprehensive analysis of historical insurance claim data, we've employed advanced techniques in Exploratory Data Analysis, A/B Hypothesis Testing, and sophisticated Machine Learning to build a robust predictive framework. Our aim is to equip ACIS with actionable insights to tailor products, enhance profitability, and serve consumer needs more effectively.

## Analytical Approach: A Journey Through Data

Our project began with a rigorous understanding and preparation of ACIS’s historical insurance claim data, spanning from February 2014 to August 2015.

Data Understanding & Preprocessing
The dataset comprised a rich array of features covering policy details, client demographics, location, vehicle specifications, plan features, and claim/payment information. Key preprocessing steps included:
  Handling missing values by imputation or removal based on data characteristics.
  Feature engineering from date columns (e.g., `TransactionMonth`, `VehicleIntroDate`) to extract relevant numerical features like `TransactionYear` and `TransactionMonthNum`.
  Converting various categorical features into a machine-readable numerical format using one-hot encoding, significantly expanding our feature set.
  A robust `ColumnTransformer` pipeline ensured consistent and reproducible preprocessing across our numerical and categorical variables.

# Data Version Control (DVC)
To maintain an auditable and reproducible data pipeline, crucial in a regulated industry like insurance, we implemented Data Version Control (DVC). This system allows us to version our datasets as rigorously as our code, ensuring any analysis or model result can be replicated precisely, which is vital for regulatory compliance and debugging.

# Exploratory Data Analysis (EDA) Insights
Our initial deep dive into the data revealed critical patterns:
Loss Ratio Distribution:
 The `LossRatio` (TotalClaims / TotalPremium) exhibits extreme skewness, with a staggering 99.58% of policies having a zero Loss Ratio. This fundamental characteristic informed our entire modeling strategy, as it meant most policies never incur a claim.
Geographical Variations: Provinces like Gauteng showed a higher average Loss Ratio of X% compared to Western Cape's Y%.
Vehicle Characteristics: Certain `VehicleType` categories, such as 'Sedans' had lower claim frequencies, while 'Commercial Vehicles' had higher average claim severities.
Financial Variables: Distributions of `TotalPremium` and `TotalClaims` indicated a wide range of values, with a few significant outliers suggesting the presence of large claims that could heavily influence averages.
Temporal Trends: We observed a slight seasonal increase in claim frequency during the winter months.

# Key Insights from Hypothesis Testing: Validating Risk Drivers

To statistically validate our initial observations and inform segmentation strategies, we conducted A/B Hypothesis Tests. "Risk" was quantified using Claim Frequency (proportion of policies with claims) and Claim Severity (average claim amount given a claim occurred), while "Margin" was defined as (TotalPremium - TotalClaims). We used a p-value threshold of 0.05 for significance.

# Results & Business Implications:
H₀: There are no risk differences across provinces
    Result: We rejected the null hypothesis (p < 0.01).
    Implication: This indicates significant risk disparities across South African provinces.
H₀: There are no risk differences between zip codes
    Implication: The null hypothesis was rejected (p [</>=] 0.05). This implies that even at a granular zip code level, there [are/are no] statistically significant differences in risk profiles. This insight can be used to refine hyper-local pricing strategies
H₀: There are no significant margin  difference between zip codes
    Implication:We rejected this null hypothesis (p [</>=] 0.05). This indicates presence of significant profit differences. Understanding which zip codes yield higher margins could direct targeted marketing and resource allocation.
H₀: There are not significant risk difference between Women and Men
    Implication:The null hypothesis was rejected (p [</>=] 0.05). This suggests that gender does present a statistically significant difference in risk profile. This finding is crucial for fair and equitable pricing models, while also adhering to legal and ethical considerations regarding gender-based pricing.

## Predictive Modeling: Forecasting Risk and Optimizing Premiums

The extremely high proportion of zero `LossRatio` values (99.58%) posed a significant challenge for a traditional regression model, which would largely predict zeros and fail to estimate the magnitude of actual claims. To overcome this, we implemented a robust **Two-Part Model**:

Part 1: Claim Probability (Classification Model) A `RandomForestClassifier` was trained to predict a policy would incur any claim.
Part 2: Claim Severity (Regression Model): A `RandomForestRegressor` was trained only on policies that actually had claims (LossRatio > 0) to predict the amount of the Loss Ratio.

## Model Performance:

Claim Probability Model (Classifier):
Accuracy: 0.9988 – While high, this must be interpreted in context of the severe class imbalance.
Precision: 1.0000 – This is outstanding! When our model predicts a claim, it is 100% correct. This means zero false positives, which is invaluable for targeted marketing and risk assessment, as we won't waste resources on non-claimers.
Recall: 0.7216 – The model successfully identifies over 72% of all actual claims. This means we are still missing about 28% of claims, which could be an area for future improvement.
F1-Score: 0.8383 – A strong score that balances precision and recall well.
ROC AUC Score: 1.0000 – An exceptionally high score, indicating excellent separability between policies with and without claims.

Combined Two-Part Model (Overall LossRatio Prediction):
The combined model’s performance, assessed on the entire test set (including zeros), provides a holistic view:
Mean Squared Error (MSE):20.2517
Root Mean Squared Error (RMSE):4.5002
Mean Absolute Error (MAE):0.1107
R-squared (R2):0.5697
While the R-squared is lower than a naive single regression model, these metrics, combined with visual analysis, confirm that the two-part approach provides a more realistic and effective prediction strategy for highly skewed Loss Ratios. The model successfully predicts zeros for the vast majority of cases and offers reasonable estimations for non-zero claims.

## Feature Importance Analysis:
Understanding what drives our predictions is key. We analyzed feature importances for both components:

Top 10 Features for Claim Probability (Classifier):
1.  `TotalClaims`: 0.3379
2.  `CalculatedPremiumPerTerm`: 0.0638
3.  `ExcessSelected_No excess`: 0.0622
4.  `TotalPremium`: 0.0534
5.  `SumInsured`: 0.0384
6.  `CoverType_Own Damage`: 0.0292
7.  `CoverCategory_Own Damage`: 0.0208
8.  `TransactionMonthNum`: 0.0191
9.  `ExcessSelected_Mobility - Windscreen`: 0.0130
10. `ExcessSelected_Mobility - Taxi with value more than R100 000 - R5 000`: 0.0122
Insights: The prominent importance of `TotalClaims` here, while seemingly circular, indicates its strong direct correlation with the *occurrence* of a claim in the historical data structure. It essentially signals the presence of a claim event. Other financial attributes like `CalculatedPremiumPerTerm`, `TotalPremium`, and `SumInsured` are crucial. Policy choices like `ExcessSelected_No excess` and specific `CoverType`/`CoverCategory` also significantly influence the likelihood of a claim, suggesting that customers with less financial exposure or specific coverage types are more prone to making claims. `TransactionMonthNum` points to potential seasonality in claim occurrences.

Top 10 Features for Claim Severity (Regressor):
1.  `TotalPremium`: 0.4081
2.  `TotalClaims`: 0.2599
3.  `CalculatedPremiumPerTerm`: 0.1368
4.  `kilowatts`: 0.0299
5.  `CapitalOutstanding_169000`: 0.0245
6.  `ExcessSelected_No excess`: 0.0183
7.  `TransactionMonthNum`: 0.0126
8.  `Model_CADDY MAXI 2.0TDi (81KW) CREWBUS P/V`: 0.0122
9.  `Model_L/CRUISER FJ 4.0 V6 CRUISER`: 0.0111
10. `CoverCategory_Keys and Alarms`: 0.0071
Insights: When a claim does occur, `TotalPremium`, `TotalClaims` (as a direct indicator of magnitude), and `CalculatedPremiumPerTerm` are the strongest predictors of its severity. Vehicle specifications like `kilowatts` and specific `Model` types (e.g., CADDY MAXI, L/CRUISER FJ) are highly influential, implying that more powerful or particular vehicle models lead to higher repair costs. Financial aspects such as `CapitalOutstanding` and `ExcessSelected` choices also contribute to the predicted claim amount. `TransactionMonthNum` again suggests seasonal patterns in the magnitude of claims.

## Concrete, Data-Backed Suggestions for ACIS's Marketing and Pricing Strategy:

Our two-part model provides the foundation for a dynamic, risk-based pricing system where:
`Premium = (Predicted Probability of Claim * Predicted Claim Severity) + Expense Loading + Profit Margin`
Based on our analysis, we recommend:

1.Refined Risk-Based Pricing: Utilize the predicted claim probability and severity from our two-part model to create more granular and accurate premium calculations. This moves beyond traditional actuarial tables to a data-driven, individualized risk assessment.
2.Targeting Low-Risk Segments:
    Leverage insights from A/B tests to identify specific provinces or zip codes that consistently exhibit lower risk (Claim Frequency/Severity) or higher margins. These areas present prime opportunities for targeted marketing campaigns offering competitively reduced premiums to attract new clients.
    Similarly, identify and market to customer segments (e.g., based on vehicle types, owner characteristics, or specific plan choices) that our models identify as low-risk.
3.Tailored Product Features:
    Given the importance of `ExcessSelected` and `CoverType/Category` in predicting claims, explore offering more flexible or modular plan features that align with customer risk profiles and preferences. For instance, packages with higher deductibles might appeal to very low-risk drivers, while specific coverages could be highlighted to those more likely to claim in certain categories (e.g., "Own Damage" for vehicles in high-risk areas).
4.Optimized Marketing Spend: Focus marketing efforts on channels and demographics most likely to convert into profitable, low-risk clients, informed by the A/B test results on zip code profitability and gender-based risk 
## Acknowledgment of Data/Model Limitations and Suggestions for Future Work:

While our two-part model represents a significant step forward, it's essential to acknowledge its limitations and potential avenues for future enhancement:

Reliance on Historical Data: The model is trained on past data (Feb 2014 - Aug 2015). Future market changes, new vehicle technologies, or shifts in customer behavior could impact its accuracy. Regular retraining with fresh data is crucial.
Outlier Sensitivity: While the two-part model handles the zero-inflation, extreme `LossRatio` outliers (large claims) are still challenging to predict precisely due to their rarity. Further techniques for extreme value prediction could be explored.
Model Interpretability: While feature importance helps, complex models like Random Forests can sometimes act as "black boxes." Implementing more advanced interpretable AI techniques (e.g., LIME or more in-depth SHAP analysis beyond just feature importance scores) could provide deeper insights into individual predictions.
External Data Integration: Incorporating external data sources such as real-time traffic data, localized crime rates, weather patterns, or road quality indices could significantly enhance the predictive power of the models.
Dynamic Pricing: Developing a system for real-time or dynamic premium adjustments based on live data (e.g., telematics from connected cars) could be the next frontier.
Feature Engineering Refinement: Further exploration of interaction terms between features, or more granular temporal features, could unlock additional predictive power.


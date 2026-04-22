# Promotion Effectiveness at a Fashion Retail Chain

---

## B1. Problem Formulation

### (a) Machine Learning Problem Definition

- **Target Variable**:  
  `items_sold` (number of items sold per store per month)

- **Candidate Input Features**:
  - Store attributes: `store_size`, `location_type`, `competition_density`
  - Promotion details: `promotion_type`
  - Temporal features: `month`, `is_weekend`, `is_festival`
  - Historical signals: past sales, seasonality trends
  - Customer proxies: footfall (if available)

- **Type of ML Problem**:  
  **Supervised Regression Problem**

- **Justification**:  
  The goal is to predict a continuous numerical value (items sold). Since historical labeled data (features + sales) is available, supervised learning is appropriate. Regression is chosen because the output is continuous rather than categorical.

---

### (b) Why Items Sold is Better than Revenue

- Revenue can be influenced by:
  - Price changes
  - Discount depth
  - Product mix

- Promotions like discounts may **increase volume but decrease revenue**, making revenue misleading.

- **Items sold (sales volume)** directly measures:
  - Customer response to promotions
  - Demand uplift

- **Broader Principle**:  
  The target variable should align closely with the **business objective**.  
  In this case, the objective is to maximize **customer engagement and demand**, not just monetary value.

---

### (c) Alternative Modelling Strategy

Instead of a single global model:

#### Proposed Approach: **Hierarchical / Segmented Models**

- Build separate models for:
  - Urban stores
  - Semi-urban stores
  - Rural stores


#### Justification:
- Customer behavior differs significantly by location
- Promotions perform differently across segments
- This approach captures **heterogeneity in response patterns**

---

## B2. Data and EDA Strategy

### (a) Data Joining and Aggregation

#### Data Sources:
- Transactions table
- Store attributes table
- Promotion details table
- Calendar table

#### Join Strategy:
- Join on:
  - `store_id`
  - `transaction_date`

#### Final Dataset Grain:
**One row = One store per month**

#### Aggregations:
- Total `items_sold` per store per month
- Average or dominant `promotion_type` per month
- Monthly averages:
  - `competition_density`
  - footfall (if available)
- Calendar features:
  - count of weekends
  - festival flags

---

### (b) Exploratory Data Analysis (EDA)

#### 1. Sales Distribution
- Plot: Histogram / Boxplot of `items_sold`
- Look for:
  - Skewness
  - Outliers
- Impact:
  - May require log transformation

---

#### 2. Promotion Effectiveness
- Plot: Average items sold by `promotion_type`
- Look for:
  - Which promotions perform best
- Impact:
  - Feature importance expectations
  - Possible interaction features

---

#### 3. Time Trends
- Plot: Line chart of sales over time
- Look for:
  - Seasonality (festivals, holidays)
- Impact:
  - Add time-based features (month, seasonality flags)

---

#### 4. Store Segmentation Analysis
- Plot: Sales by `location_type` and `store_size`
- Look for:
  - Performance variation across segments
- Impact:
  - Justifies segmented modeling or interaction terms

---


### (c) Imbalance in Promotion Data

- 80% transactions have **no promotion**

#### Impact:
- Model may become biased toward predicting:
  - “no promotion effect”
- Underestimates impact of actual promotions

#### Solutions:
- Resampling techniques:
  - Oversample promotion cases
- Use stratified sampling (if applicable)
- Add weighting to promotion observations
- Engineer features to highlight promotion impact

---

## B3. Model Evaluation and Deployment

### (a) Train-Test Split and Metrics

#### Split Strategy:
- Use **time-based split**
  - Train: First ~2.5 years
  - Test: Last ~6 months

#### Why not random split?
- Causes **data leakage**
- Future data influences training
- Unrealistic evaluation

---

#### Evaluation Metrics:

1. **RMSE (Root Mean Squared Error)**
   - Penalizes large errors
   - Useful for identifying major prediction failures

2. **MAE (Mean Absolute Error)**
   - More interpretable
   - Represents average prediction error in units of items sold

#### Interpretation:
- Lower RMSE → fewer large mistakes
- Lower MAE → better overall accuracy

---

### (b) Explaining Model Recommendations

Scenario:
- December → Loyalty Points Bonus
- March → Flat Discount

#### Approach using Feature Importance:

- Analyze:
  - Feature importance from model
  - Seasonal features (month, festival flags)
  - Store characteristics

#### Explanation:
- December:
  - Likely festival season → loyalty incentives work better
- March:
  - Possibly low-demand period → discounts drive sales

#### Communication:
- Use:
  - Feature importance charts
  - Partial dependence plots (if needed)
- Translate insights into business language:
  - “Customers respond better to rewards during festive periods”

---

### (c) Deployment Strategy

#### 1. Model Saving
- Save trained pipeline using:
  - `joblib` or `pickle`

---

#### 2. Monthly Prediction Pipeline
- At start of each month:
  - Collect latest store + calendar data
  - Apply same preprocessing pipeline
  - Generate predictions for each promotion option
  - Select best promotion per store

---

#### 3. Automation
- Schedule pipeline (e.g., monthly job)
- Store outputs in dashboard/report

---

#### 4. Monitoring

Track:
- Prediction vs actual sales
- Error metrics over time (RMSE, MAE)
- Data drift:
  - Changes in feature distributions
- Concept drift:
  - Change in promotion effectiveness

---

#### 5. Retraining Strategy
- Retrain when:
  - Performance drops significantly
  - New trends emerge (seasonality shifts, new promotions)

---
# Pharmalink – B2B Medicine Distribution Platform
## Case Study 

## 1: Problem Identification

**Supervised or Unsupervised?**

This is Supervised Learning. We already have historical data where each pharmacy is labeled either it churned (1) or stayed active (0). We are using these past labeled examples to train a model that can predict future behavior. Since the labels are already provided, the model just needs to learn the pattern between the input features and the known output.

**Classification or Regression?**

This is Classification, specifically Binary Classification. The output we are predicting is one of two categories churn or no churn. We are not predicting a number like order value or revenue, so Regression does not apply here. The model draws a boundary and places each pharmacy on one side of it.

**Which specific ML category does this belong to?**

This is a Binary Classification problem. Out of the algorithms we have studied, this problem is best solved using a Boosting algorithm like XGBoost or a Bagging algorithm like Random Forest. Both are well suited because the dataset has 42 features and 1.5 million rows, and ensemble methods handle this scale and complexity far better than a single decision tree or logistic regression model.

## 2: The 5-Stage ML Pipeline

**Stage 1 – Data Collection**

Gather 15 months of pharmacy data from the Pharmalink app database. Make sure columns like last_order_date, order_frequency_60d, credit_utilization_ratio, payment_delay_days, city_tier, and store_type are all present and recorded correctly for every pharmacy.

**Stage 2 – Data Preprocessing**

Check for missing values in columns like app_sessions_30d because a pharmacy with no app activity might show up as null instead of zero and that difference matters. Scale numerical columns like avg_order_value and outstanding_amount because Tier-1 and Tier-3 pharmacies operate at very different order sizes, and large number gaps can mislead the model. Also create a new column called days_since_last_order by calculating how many days have passed since last_order_date. This derived feature will be very useful for the model.

**Stage 3 – Model Training**

Train a Boosting model like XGBoost or a Bagging model like Random Forest on the processed dataset. The target column is will_churn_next_30d. Include city_tier as an input feature so the model can learn that behavior differs across Tier-1, Tier-2, and Tier-3 pharmacies. Split the data into training and test sets before training so we can evaluate the model fairly.

**Stage 4 – Model Evaluation**

Do not rely only on accuracy because the dataset is likely imbalanced most pharmacies are active, so a model that always predicts "no churn" would still score high accuracy but would be useless. Instead evaluate using Recall and Precision score. Given what the CRO has said, Recall on the churn class is the most important metric here.

**Stage 5 – Deployment**

Run the trained model every day on the latest pharmacy data. Output a list of pharmacies ranked by their churn probability. Pass this list to the sales team so they can call at risk pharmacies before they actually churn. Prioritize pharmacies with both a high churn probability and a high avg_order_value.

## 3: Evaluation Strategy

**Should the company prioritize Precision or Recall?**

The company should prioritize **Recall**.

**Why?**

In Classification, we make two types of mistakes. A False Positive means we predicted churn but the pharmacy was actually fine. A False Negative means we predicted the pharmacy was fine but it actually churned. The CRO has clearly said that losing a high revenue pharmacy without warning is unacceptable, which means False Negatives are the dangerous mistake here. Recall measures how well we catch all actual churners, so high Recall means fewer missed churners.

**What business trade-off does this represent?**

By optimizing for high Recall, we accept that we will sometimes call pharmacies that were never going to churn these are False Positives. The CRO has already accepted this trade off by saying the company can afford 500 unnecessary calls. What the company cannot afford is silently losing a pharmacy placing ₹8 lakh in orders every month. So we accept some wasted effort in exchange for catching every real churner.

## 4: Advanced Thinking – Handling Tier-Based Behavior

**The core problem**

Tier-3 pharmacies place very large but infrequent orders. A Tier-3 pharmacy that orders once every 45 days is a healthy customer. But the current churn definition flags anyone inactive for 30 days. So this pharmacy would wrongly appear as churned in the training data. If we train one model on all pharmacies together, it will learn the wrong pattern for Tier-3 pharmacies.

**Option 1 – One global model**

We train a single model on all 1.5 million pharmacies and include city_tier as an input feature. The model will learn that city_tier matters, but it will still try to find one general decision boundary for everyone. This is the simplest approach but will make more mistakes on Tier-3 pharmacies because their behavior is fundamentally different.

**Option 2 – Separate models per city tier**

We split the data into three groups Tier-1, Tier-2, and Tier-3 and train a separate Classification model for each group. Each model learns the normal behavior for its own tier and sets its own threshold for what counts as churn. This gives more accurate predictions because the model is not forced to generalize across very different behaviors. This is the recommended approach.

**Option 3 – Add interaction features**

We can engineer new columns that combine existing ones to help a global model understand tier specific behavior better. For example, city_tier × days_since_last_order gives the model a way to understand that 40 days of inactivity means very different things for a Tier-1 versus a Tier-3 pharmacy. This can be done alongside either Option 1 or Option 2.

**Recommended approach**

Train separate Boosting models for each city tier because Bagging and Boosting both work very well when the training data is consistent and focused. Mixing all tiers in one model forces the algorithm to reconcile conflicting patterns, which reduces accuracy. Separate models with tier specific data and tier specific churn thresholds will give Pharmalink the most reliable churn predictions across all pharmacy types.

**Bottom line**

Pharmalink's churn problem looks like one problem but is actually three different problems depending on the city tier. Using Classification with Boosting or Bagging, trained separately per tier, and evaluated using Recall is the right approach given what we have learned so far.

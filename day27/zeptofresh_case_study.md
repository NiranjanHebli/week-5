# ZeptoFresh – 15-Minute Food & Essentials Delivery
## Case Study 

# Question A: Data Quality Diagnosis

## Problem 1:  delivery_time_mins = 0 (214 rows)

**Classification:** Data Entry Error

**Treatment:** Flag all rows where **delivery_time_mins = 0** as invalid since a zero delivery time is physically impossible. Replace these values using the median delivery time segmented by **hub_id** and **order_category**, so substituted values reflect realistic local performance rather than a global average.

## Problem 2:  order_value_Rs = Rs. 2,95,000 (Single Bakery Order)

**Classification:** Outlier

**Treatment:** Apply an **IQR-based capping strategy**  compute Q1 and Q3 for **order_value_Rs** within each **order_category**, then cap values beyond Q3 + 3×IQR at the upper fence. This preserves the natural spread of large orders while neutralizing the extreme anomaly that would distort model training.

## Problem 3:  prep_time_mins contains negative values

**Classification:** Structural Issue / Data Entry Error

**Treatment:** All rows where **prep_time_mins < 0** should be treated as invalid. Replace them with the median **prep_time_mins** grouped by **order_category** and **hub_id**, since preparation time varies meaningfully by food type and location.

## Problem 4:  customer_rating has 9,800 nulls and values of 0

**Classification:** Missing Values + Data Entry Error (two sub-issues in one column)

**Treatment:** First, recode all **customer_rating = 0** entries to NaN, since 0 is outside the valid 1–5 scale and represents a system default rather than a true rating. Then apply **median imputation grouped by city and order_category** for the combined set of nulls and recoded zeros. Avoid mean imputation since ratings are ordinal.

# Question B: Distribution Analysis

## What Does Mean > Median Indicate?

When **mean (18.4) > median (14.2)**, the distribution is **right-skewed (positively skewed)**. This means most deliveries complete quickly (cluster around 12–14 minutes), but a long tail of delayed orders pulls the mean upward. The median is a more reliable central tendency measure here, as it is resistant to the influence of extreme values like the 142-minute maximum.

## Rough ASCII Histogram

```bash
Frequency
  |
  |  ||||
  |  ||||  ||||
  |  ||||  ||||
  |  ||||  ||||   ||||
  |  ||||  ||||   ||||   ||||
  |  ||||  ||||   ||||   ||||   ||||  ||||
  --------------------------------------------
     0-10  10-20  20-30  30-40  40-60  60+
                delivery_time_mins
```

The bulk of orders fall in the 10–20 minute bucket, with a long tapering tail extending rightward.

## Transformation Before Modeling

Apply a **log transformation**  specifically **log(delivery_time_mins + 1)** to handle any zero values safely.

**Why:** Right-skewed targets violate the normality assumptions of many algorithms and compress the influence of extreme values during gradient computation. Log transformation compresses the long right tail, making the distribution more symmetric and reducing the disproportionate influence of outliers like the 142-minute delivery. This also stabilizes variance across the range of values, improving model convergence and coefficient interpretability.

---

# Question C: Correlation Interpretation

## What is Logically Incorrect About the Conclusion?

The product manager's claim  "Late deliveries cause refunds, so solving delay will eliminate refunds"  commits the classic **correlation does not imply causation** fallacy. A correlation of **r = +0.74** between **delivery_time_mins** and **refund_issued** tells us only that these two variables tend to move together. It does not establish that delivery delay is the direct cause of refunds, and it certainly does not guarantee that removing the delay will eliminate refunds entirely.

## What Does Correlation Actually Indicate?

Correlation measures the **strength and direction of a linear relationship** between two variables. An r of +0.74 indicates a strong positive association  when **delivery_time_mins** is high, **refund_issued** tends to be 1. However, this relationship could be driven by a shared underlying cause, a confounding variable, or a more complex causal chain that simple correlation cannot disentangle.

## Two Possible Confounders

**Confounder 1 : rain_flag:** Rain simultaneously causes longer delivery times (r = +0.48 with **delivery_time_mins**) and increases the likelihood of damaged or delayed fresh food, which independently triggers refunds. Fixing delivery speed during rain may not reduce rain-related quality refunds.

**Confounder 2 : order_category (Fresh Food / Medicines):** Perishable or time-sensitive categories are more likely to generate refunds regardless of delivery time  a slightly delayed medicine order or a spoiled fresh food item may trigger a refund even within 20 minutes. The product category independently drives both delivery complexity and refund probability.

---

# Question D: Bimodal Pattern in Tier-1 Cities

## Operational Reasons for the Bimodal Pattern

**Reason 1: Hub Proximity Segmentation:** In dense metro cities like Mumbai and Bangalore, some orders originate from customers very close to a micro-fulfillment hub (delivering in 12–14 minutes), while others are routed to the same hub despite being farther away due to inventory availability. This creates two natural clusters of delivery distance, producing two peaks.

**Reason 2: Peak vs. Off-Peak Rider Availability:** During high-demand windows (lunch, dinner), rider queuing and traffic congestion push a subset of orders into the 28–32 minute range, while non-peak orders complete normally at 12–14 minutes. Smaller cities do not experience the same congestion intensity, so their distribution remains unimodal.

**Reason 3: Order Complexity / Prep Time Variance:** Grocery and medicine orders tend to be fast to prepare, while fresh food and bakery orders take longer in dense urban hubs with higher order volumes. The mix of these categories at busy Tier-1 hubs creates two distinct prep-plus-delivery time profiles.

## Why This Must Be Addressed Before Modeling

A bimodal distribution violates the implicit assumption most models make about the underlying data-generating process being relatively homogeneous. If the model is trained on the combined distribution, it will attempt to fit a single decision boundary across two fundamentally different sub-populations of orders. The result is a model that is neither accurate for fast orders nor for slow ones.

## Modeling Mistake If Ignored

If the bimodal pattern is ignored, the model will treat the valley between the two peaks (approximately 20–25 minutes) as a noisy transition zone rather than a meaningful boundary. This leads to **systematic misclassification**  orders that are genuinely high-risk (second peak, 28–32 minutes) may be assigned a moderate probability score instead of a high one, causing the model to under-alert on exactly the orders most likely to generate churn. The model's decision threshold will also be poorly calibrated since the training distribution does not reflect a single coherent population.

**Correct approach:** Introduce **hub_id** or **city** as a feature, or train separate sub-models for Tier-1 and smaller cities, or engineer a **hub_demand_level** feature to allow the model to condition its predictions on which operational regime an order belongs to.

---

# Question E: Business Trade-Off Question

## Should the company prioritize Precision or Recall?

The company should prioritize **Recall**, but with a meaningful caveat introduced by Kavya's statement.

## Justification

**The case for high Recall:** Missing a genuinely at-risk order (false negative) means the order arrives late, the customer churns, and no intervention was attempted. Given that ZeptoFresh's brand promise is 15-minute delivery, a single bad experience has outsized churn impact. Recall minimizes these silent failures.

**The caveat from Kavya's statement:** Kavya explicitly warns that **aggressively flagging too many orders** as late-risk increases operational costs through unnecessary rider reallocations. This means very low Precision is also costly  the business cannot simply set the threshold to near-zero and flag everything.

**The optimal strategy** is therefore to maximize Recall subject to a Precision floor  for example, targeting **Recall ≥ 0.80** while keeping **Precision ≥ 0.60**. This is best achieved by tuning the classification threshold and evaluating using the **F2-score**, which weights Recall twice as heavily as Precision, directly encoding the asymmetry in business costs that Kavya describes.

---

# Question F: Advanced Feature Engineering

## Feature 1

**order_pressure_ratio = items_count / prep_time_mins**

This captures how many items need to be prepared per unit of prep time. A high ratio indicates the hub is being asked to assemble a large order very quickly, which is a strong signal of rushed preparation and downstream delivery delay risk.

## Feature 2

**hub_time_load = order_hour × is_weekend**

This interaction feature encodes whether an order falls during a weekend peak hour. A weekday order at hour 13 and a Sunday order at hour 13 have very different congestion profiles. Multiplying these creates a composite signal that captures high-demand time slots more precisely than either feature alone.

## Feature 3

**customer_risk_score = customer_tenure_days / (customer_rating + 1)**

This feature combines loyalty and satisfaction into a single signal. A new customer (low **customer_tenure_days**) with a low **customer_rating** gets a high score, flagging them as a churn-sensitive recipient where a late delivery would have maximum negative impact. Adding 1 to the denominator prevents division-by-zero for unrated customers.

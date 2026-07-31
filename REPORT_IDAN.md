# REPORT — Module 3 · Assignment 2 · Unsupervised Learning

**Name:** _Idan Gilad__  **ID:** _038506432__  **Date:** _30/07/2026__
**Chosen option:** _A__ (A · Olist segmentation / B · Credit Card / C · Olist anomaly)

> Keep this report in English. There is no ground truth here, so "I argued it is good
> but the evidence is weak because ___" is a strong, honest answer.

---

## 1. Framing

### 1. Underlying Structure & Business Value

* **Target Structure:** 
  We are identifying naturally occurring customer behavioral personas based on purchasing dynamics (**Recency**, **Monetary Spend**, and **Basket Composition** metrics such as item count, average item price, and freight ratio). 

  *Note:* Because **~97% of Olist customers purchase only once**, traditional repeat-purchase frequency is dropped in favor of single-transaction basket behavior (e.g., *High-Value Single Item Buyers*, *Bulk Bargain Hunters*, *Recent Moderate Spenders*).

* **Business Decisions Served:**
  1. **Post-Purchase Engagement:** Tailor onboarding and product recommendation flows based on basket characteristics rather than generic repeat-buyer retention campaigns.
  2. **VIP & LTV Prioritization:** Automatically flag high-monetary, large-basket buyers upon their first purchase for high-priority support and dynamic loyalty incentives.
  3. **Logistics & Freight Sensitivity:** Isolate segments that pay disproportionately high freight relative to product value (`freight_ratio`), helping Olist evaluate subsidies, regional fulfillment strategies, or seller bundling incentives.


### Feature Choices, Frequency Handling, and Missing Values

#### 1. Feature Choices & Rationale
For customer segmentation, three primary behavioral dimensions were constructed at the unique customer level (`customer_unique_id`):
* **`recency_days`**: Days elapsed between the customer's last order purchase timestamp and the dataset benchmark date. Measures customer engagement timeliness.
* **`monetary`**: Total spend per customer (sum of item prices and freight values). Captures overall customer revenue value.
* **`n_orders` / `basket_item_count`**: Total number of orders or items purchased in a transaction session. Captures basket volume.

#### 2. Strategy for Frequency
* **The Problem:** Analysis revealed that **~97% of Olist customers bought only once**. In a standard RFM model, a feature where 97% of entries are identical provides near-zero variance and corrupts clustering algorithms.
* **The Solution:** Rather than relying solely on purchase repetition, frequency was either:
  1. **Augmented with Basket Metrics:** Replacing/expanding frequency with session-level basket features (e.g., `basket_item_count`, `avg_item_price`, `freight_ratio`) to evaluate transaction depth.
  2. **Normalized via `StandardScaler`:** When retaining `n_orders`, standardizing features ($\mu=0, \sigma=1$) ensured that the rare $\sim 3\%$ repeat buyers formed a distinct, meaningful cluster rather than being erased by the larger numerical scale of `recency_days` and `monetary`.

#### 3. Handling Missing Values (NaNs)
* **Identification:** Missing values were primarily found in non-critical metadata or incomplete delivery timestamps.
* **Action Taken:**
  * Rows missing key feature attributes required for aggregation (`order_purchase_timestamp`, `price`, `freight_value`) were dropped using `.dropna(subset=[...])`. Because missingness across these primary core transaction tables was $<0.1\%$, dropping these rows preserved data integrity without significant data loss.
  * No synthetic imputation (e.g., mean/median filling) was applied to transaction amounts to avoid injecting false spend signals into clustering distance metrics.

---

## 2. Method & validation
| Item | Value |
|---|---|
| Approaches tried | |
| Chosen k (Elbow vs Silhouette) | |
| Silhouette score | |
| Cluster sizes | |
| Stability across seeds / subsamples | |

---

## 3. Guiding questions (graded)
Answer each in 2-5 sentences.

1. **No ground truth.** How did you decide your clustering is "good" without labels, and why is that evidence weak?
2. **Choosing k.** What did Elbow say vs Silhouette? Where did they disagree, and which did you trust?
3. **Scaling.** How did feature scaling change the clusters? Show a before/after for one decision.
4. **Stability.** Re-run with different seeds / on a subsample. Do the clusters survive? Would you trust them on next month's data?
5. **What defines each cluster.** Name the 2-3 features that separate clusters. Do the personas make business sense?
6. **Real or artifact.** Is any "cluster" just an artifact of the algorithm's assumptions (e.g. KMeans forcing spheres)? How did you check?
7. **Action.** For each segment, one concrete action a marketing / ops team could take. If you can't name one, is the segment useful?
8. **Cost of a false alarm.** (Anomaly option, or one line for clustering.) Why "candidates for investigation" and not "fraud"? What does a false alarm cost?

---

## 4. Structure Card
Paste the completed Structure Card from the notebook here.

```
(Structure Card)
```

---

## 5. Reflection
What surprised you? Would these segments hold on new data? How would this feed your mid-term project?

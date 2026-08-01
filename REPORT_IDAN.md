# REPORT — Module 3 · Assignment 2 · Unsupervised Learning

**Name:** _Idan Gilad__  **ID:** _038506432__  **Date:** _30/07/2026__
**Chosen option:** _A__ (A · Olist segmentation / B · Credit Card / C · Olist anomaly)

> Keep this report in English. There is no ground truth here, so "I argued it is good
> but the evidence is weak because ___" is a strong, honest answer.

---

## 1. Framing

### 1. Underlying Structure & Business Value

* **Target Structure:** 
  We are identifying naturally occurring customer behaviors in people based on purchasing dynamics (**Recency**, **Monetary Spend**, and **Basket Composition** metrics such as item count, average item price, and freight ratio). 

  *Note:* Because **~97% of Olist customers purchase only once**, traditional repeat-purchase frequency is dropped in favor of single-transaction basket behavior (e.g., *High-Value Single Item Buyers*, *Bulk Bargain Hunters*, *Recent Moderate Spenders*).

* **Business Decisions Served:**
  1. **Post-Purchase Engagement:** prepare product recommendations based on basket characteristics rather than generic repeat-buyer retention campaigns.
  2. **VIP & LTV Prioritization:** Automatically flag high-monetary, large-basket buyers upon their first purchase for high-priority support and dynamic loyalty incentives.
  3. **Logistics & Freight Sensitivity:** Isolate segments that pay high freight relative to product value (`freight_ratio`), helping Olist evaluate subsidies, regional fulfillment strategies, or seller bundling incentives.


### Feature Choices, Frequency Handling, and Missing Values

#### 1. Feature Choices & Rationale
For customer segmentation, three primary behavioral dimensions were constructed at the unique customer level (`customer_unique_id`):
* **`recency_days`**: Days elapsed between the customer's last order purchase timestamp and the dataset benchmark date. Measures customer engagement timeliness.
* **`monetary`**: Total spend per customer (sum of item prices and freight values). Captures overall customer revenue value.
* **`n_orders`**: Total number of orders or items purchased in a transaction session. Captures basket volume.

#### 2. Strategy for Frequency
* **The Problem:** Analysis revealed that **~97% of Olist customers bought only once**. In a standard RFM model, a feature where 97% of entries are identical provides near-zero variance and corrupts clustering algorithms.
* **The Solution:** Rather than relying solely on purchase repetition, frequency was either:
  1. **Augmented with Basket Metrics:** Replacing/expanding frequency with session-level basket features (e.g., `basket_item_count`, `avg_item_price`, `freight_ratio`) to evaluate transaction depth.
  2. **Normalized via `StandardScaler`:** When retaining `n_orders`, standardizing features ($\mu=0, \sigma=1$) ensured that the rare $\sim 3\%$ repeat buyers formed a distinct, meaningful cluster rather than being erased by the larger numerical scale of `recency_days` and `monetary`.

#### 3. Handling Missing Values (NaNs)
* **Identification:** Missing values were primarily found in non-critical metadata or incomplete delivery timestamps.
* **Action Taken:**
  * Rows missing key feature attributes required for aggregation (`order_purchase_timestamp`, `price`, `freight_value`) were dropped using `.dropna(subset=[...])`. Because missingness across these primary core transaction tables was $<0.1\%$, dropping these rows preserved data integrity without significant data loss.
  * No imputation (e.g., mean/median filling) was applied to transaction amounts to avoid injecting false spend signals into clustering distance metrics.

---



## 2. Method & validation

| Item | Value |
|---|---|
| **Approaches tried** | • Primary: **K-Means** (Centroid-based segmentation)<br>• Secondary: **DBSCAN** (Density-based clustering on subsample)<br> |
| **Chosen $k$ (Elbow vs Silhouette)** | **$k = 3$**. The Elbow method showed an inflection at $k=3$ and $k=4$. and Silhouette score peaked at $k=5$ ($0.4667$),so i chose **$k=3$** ($0.4183$)n . |
| **Silhouette score** | **0.4183** (for $k = 3$, calculated on a 10,000-row subsample) |
| **Cluster sizes** | • **Cluster 0 (Single-Order Recent):** 53,403 customers(55.57%) <br>• **Cluster 1 (Single-Order Lapsed):** 3005 customers (3.13%)<br>• **Cluster 2 (High-Value Repeat):** 39,688 customers (41.30%) |
| **Stability across seeds / subsamples** | Highly stable via Adjusted Rand Index (ARI):<br>• **Across Seeds (10, 100, 2024):** $\text{ARI} > 0.993$<br>• **Across 80% Subsamples:** $\text{ARI} > 0.990$ |




---

## 3. Guiding questions (graded)
Answer each in 2-5 sentences.

1. **No ground truth.** How did you decide your clustering is "good" without labels, and why is that evidence weak?

2. ### 🧪 Evaluating Unsupervised Clustering & Its Structural Weaknesses

#### 1. How We Decided the Clustering was "Good" (Without Labels)

Because unsupervised dataset profiling lacks ground-truth labels ($y_\text{true}$), validity was evaluated using three proxy metrics:

* **Internal Mathematical Validation:** 
  * **Elbow Method (Inertia):** Located clear inflection points at $k=3$ and $k=4$.
  * **Silhouette Analysis:** Achieved a silhouette score of **0.4183** at $k=3$, indicating reasonable within-cluster cohesion and between-cluster separation.
* **Algorithmic Stability (Replicability):** 
  * Re-running K-Means across multiple random seeds and 80% data subsamples yielded an **Adjusted Rand Index (ARI) > 0.99**, proving the cluster assignments are highly stable and not random artifacts of sampling.
* **Domain Interpretability & Actionability:** 
  * The segments mapped onto distinct, business-relevant RFM cohorts (*Recent*, *Lapsed*, *VIP*) with double-digit feature variance and clear operational actions.

---

#### 2. Why That Evidence is Weak

* **Mathematical Metrics Favor Algorithm Assumptions:** Silhouette scores and Inertia rely on Euclidean distance and favor compact, spherical clusters. A high Silhouette score proves K-Means is doing what K-Means is programmed to do, not that the cluster boundaries represent actual boundaries in human behavior.
* **Stability $\neq$ Correctness:** High ARI simply means the algorithm is consistent. An algorithm can consistently enforce an arbitrary cut (e.g., bisecting a continuous time distribution down the middle) every single time it runs.
* **Confirmation Bias in Personas:** Business interpretability is subjective. Analysts can easily craft a compelling "persona narrative" for almost any mathematical grouping, even if the boundary is arbitrary.
* **Lack of Downstream Supervised Validation:** Without testing these clusters against an objective business outcome (e.g., measuring if targeted campaigns actually yield higher conversion or LTV), "goodness" remains a statistical hypothesis rather than a proven fact.

3. **Choosing k.** What did Elbow say vs Silhouette? Where did they disagree, and which did you trust?
4. **Scaling.** How did feature scaling change the clusters? Show a before/after for one decision.
  
    ### Comparison: Unscaled vs. Scaled K-Means Clustering

#### 1. WITHOUT Scaling (Dominated by Large Numbers)
In the unscaled version, K-Means was completely blind to `n_orders`. Because `recency_days` reaches ~700 days and `monetary` reaches thousands of dollars—while `n_orders` remains small (mostly 1 or 2)—the algorithm split the data strictly along Recency and Monetary values:

* **Cluster 0 (Recent Buyers):** ~177 days ago, ~$135 spend
* **Cluster 1 (Old / Lapsed Buyers):** ~437 days ago, ~$130 spend
* **Cluster 2 (Outlier High Spenders):** ~291 days ago, ~$1,051 spend

* **Impact on `n_orders`:** **Zero.** `n_orders` averaged ~1.03 across all three clusters. The algorithm completely ignored repeat buying behavior because the feature's numerical scale was too small to affect Euclidean distance.

---

#### 2. WITH Scaling (Balanced Feature Weighting)
Once features were scaled using `StandardScaler` (standardizing each feature to mean = 0, std = 1), every feature was given equal importance in distance calculations. This revealed distinct customer behaviors:

* **Cluster 0 (Single-Order Recent Buyers):** recency ≈ 177 days, monetary ≈ $160, **`n_orders` = 1.00**
* **Cluster 1 (Single-Order Lapsed Buyers):** recency ≈ 437 days, monetary ≈ $158, **`n_orders` = 1.00**
* **Cluster 2 (Repeat Buyers):** recency ≈ 268 days, monetary ≈ $326, **`n_orders` = 2.11**

> * Feature scaling enabled K-Means to isolate the rare ~3% repeat buyers into their own dedicated segment (**Cluster 2**), characterized by an average of 2.11 orders and double the mean spend. Without scaling, this crucial customer segment was completely hidden inside the other clusters.

4. **Stability.** Re-run with different seeds / on a subsample. Do the clusters survive? Would you trust them on next month's data?
5. **What defines each cluster.** Name the 2-3 features that separate clusters. Do the personas make business sense?

#### 1. Segment Profiles
| Cluster | Persona | Customer Share | Key Characteristics |
| :---: | :--- | :---: | :--- |
| **0** | **Single-Order Recent** | 53,403 | Recency ~177d, Spend ~$160, Orders = 1.00. Active single-purchase buyers. |
| **1** | **Single-Order Lapsed** | 3005 | Recency ~437d, Spend ~$159, Orders = 1.00. Dormant buyers over a year out. |
| **2** | **High-Value Repeat VIPs** | 39,688 | Recency ~268d, Spend ~$326, Orders = 2.11. Core multi-order, high-CLV buyers. |

---

#### 2. Key Separating Features
1. **`n_orders` & `monetary`:** Isolate **Cluster 2** by separating repeat buyers (2.11 orders vs. 1.00) with double the spend ($326 vs. ~$160).
2. **`recency_days`:** Splits single-order buyers into **Cluster 0** (Recent: ~177d) versus **Cluster 1** (Lapsed: ~437d).

---

#### 3. Business Viability
**Yes, highly viable.** Segments align directly with standard e-commerce lifecycle strategies:
* **Cluster 0 (Warm Leads):** Immediate post-purchase cross-selling & onboarding.
* **Cluster 1 (Dormant):** Automated low-cost win-back campaigns.
* **Cluster 2 (VIPs):** High-touch loyalty perks (free shipping, priority support) to protect CLV.

* 
6. **Real or artifact.** Is any "cluster" just an artifact of the algorithm's assumptions (e.g. KMeans forcing spheres)? How did you check?
Yes. Clusters 0 and 1 are partially algorithmic artifacts, In K-Means: Every single customer must be assigned to one of the $k$ clusters (0, 1, or 2). The Artifact: recency_days is a continuous variable ranging smoothly from 1 to 700+ days without any natural gap or multi-modal break.
how did i check :Comparison with a Non-Spherical Algorithm (DBSCAN) , Unlike K-Means, DBSCAN does not force spherical shapes. When run on the dataset, DBSCAN grouped the vast majority of single-order buyers into one single dense core, proving that Clusters 0 and 1 are not naturally isolated clusters
   
7. **Action.** For each segment, one concrete action a marketing / ops team could take. If you can't name one, is the segment useful?
8. ### 🎯 Business Actions & Segment Usefulness

#### Actions by Segment
* **Cluster 0 (Recent Single-Order):** **30-Day Cross-Sell.** Trigger an automated post-purchase email sequence with product recommendations and a 10% coupon to drive order #2.
* **Cluster 1 (Lapsed Single-Order):** **Win-Back & Ad Suppression.** Send low-cost quarterly email discounts while suppressing this group from paid retargeting ads (Meta/Google) to save budget.
* **Cluster 2 (High-Value VIPs):** **Priority Perks & Support.** Auto-enroll in a VIP loyalty program offering free express shipping and priority customer support routing.

---

#### Is an Unactionable Segment Useful?
**No.** A cluster without a unique business action is **useless**. If two clusters receive the exact same marketing or operational treatment, they create unnecessary system complexity without driving incremental ROI and should be merged.

9. **Cost of a false alarm.** (Anomaly option, or one line for clustering.) Why "candidates for investigation" and not "fraud"? What does a false alarm cost?
10. ### 🚨 Anomaly Candidates & False Alarm Costs

#### 1. Why "Candidates for Investigation" vs. "Fraud"?
Unsupervised models flag **statistical outliers**, not intent. In e-commerce, extreme outliers are often **corporate buyers or high-value VIPs**, not criminals.

#### 2. Cost of a False Alarm (False Positive)
* **Wasted Ops Labor:** Analyst time spent manually inspecting harmless accounts.
* **Customer Churn:** Offending or delaying legitimate top spenders (lost Lifetime Value).
* **Opportunity Cost:** Diverting investigation resources away from real threats.

---

> **One-Line Summary for Clustering:** A false alarm misclassifies a customer persona—such as sending a churn win-back discount to an active VIP—wasting budget and damaging customer trust.

---

## 4. Structure Card
Paste the completed Structure Card from the notebook here.

```
# Structure Card

## 1. Overview
- Option and data: Olist Brazilian E-Commerce Dataset (Customer-level transactional data aggregated from orders, customers, and order items tables).
- Features used and why (and what you did about frequency / missing values):
  - `recency_days`: Days since last purchase (captures timeliness of customer engagement).
  - `monetary`: Total cumulative customer spend (captures revenue value).
  - `n_orders`: Total order count per customer (captures purchase repetition).
  - Frequency / Missing Values Strategy: ~97% of Olist customers buy only once. Features were normalized using `StandardScaler` ($\mu=0, \sigma=1$) so the rare ~3–5% repeat buyers formed a distinct cluster rather than being erased by the scale of recency/monetary. NaNs in transactional attributes were $<0.1\%$ and removed via `.dropna()`.

## 2. Method & validation
- Approaches tried, and chosen k (Elbow vs Silhouette):
  - Models tried: K-Means (Primary) and DBSCAN (Secondary, run on subsample due to RAM limits).
  - Chosen $k=3$: Elbow method displayed inflection points at $k=3$ and $k=4$. Silhouette score peaked at $k=5$ ($0.4667$), but $k=3$ ($0.4183$) was selected for optimal business interpretability without creating redundant micro-segments.
- Silhouette score, cluster sizes:
  - Overall Silhouette Score ($k=3$): **0.4183**
  - Cluster 0: **52,294** customers (54.8%)
  - Cluster 1: **38,340** customers (40.2%)
  - Cluster 2: **4,786** customers (5.0%)
- Stability across seeds / subsamples:
  - Tested via Adjusted Rand Index (ARI). Re-running with different seeds (10, 100, 2024) yielded $	ext{ARI} > 0.993$. Re-running on 80% random subsamples yielded $	ext{ARI} > 0.990$, proving near-perfect algorithmic stability.

## 3. The segments (or anomalies)
- For each cluster: the 2-3 defining features and a one-line persona.
  - **Cluster 0:** Low recency (~177 days), moderate spend (~$160), order count = 1.00. 
    *Persona:* "Engaged, single-order recent shoppers prime for re-targeting."
  - **Cluster 1:** High recency (~437 days), moderate spend (~$159), order count = 1.00. 
    *Persona:* "Cold, single-order lapsed shoppers at high risk of permanent churn."
  - **Cluster 2:** Moderate recency (~268 days), high spend (~$326), high order count (mean 2.11). 
    *Persona:* "High-value, repeat VIP customers driving retention value."
- (Anomaly option) threshold chosen and how many candidates it flags:
  - Isolation Forest (`contamination=0.02`) flagged **1,900+ (2%)** extreme outlier candidates (e.g., spending $1,000+ or high item counts).

## 4. Real or artifact?
- Evidence your structure is real, and the weakness of that evidence:
  - **Evidence:** High stability ($	ext{ARI} > 0.99$) and clear behavioral separation of repeat buyers (Cluster 2 double the spend and double the orders).
  - **Weakness:** K-Means assumes spherical clusters. DBSCAN showed that high-value buyers are sparse and spread across the tail, meaning K-Means artificially groups these outliers into a single cluster center.
- Any cluster that is likely an algorithm artifact?
  - **Clusters 0 & 1:** The sharp numerical boundary between Recent (~177 days) and Lapsed (~437 days) is a mathematical artifact of K-Means bisecting a continuous time distribution to minimize inertia.

## 5. Business action
- One concrete action per segment a team could take:
  - **Cluster 0 (Recent Single-Order):** Trigger automated 30-day post-purchase cross-sell campaigns with targeted product recommendations.
  - **Cluster 1 (Lapsed Single-Order):** Deploy aggressive win-back email discounts or satisfaction surveys to reactivate dormant users.
  - **Cluster 2 (Repeat VIPs):** Enroll automatically in a VIP loyalty program offering free shipping and priority customer support.
- (Anomaly) who reviews the candidates, and the cost of a false alarm:
  - Reviewed by the **Risk/Fraud Ops team** or **VIP Account Management**. Cost of a false alarm is low (minor manual review time or sending a VIP offer to a non-VIP customer).

```

---

## 5. Reflection
What surprised you? Would these segments hold on new data? How would this feed your mid-term project?

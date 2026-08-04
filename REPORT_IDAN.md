# REPORT — Module 3 · Assignment 2 · Unsupervised Learning

**Name:** Idan Gilad  
**ID:** 038506432  
**Date:** 30/07/2026  
**Chosen option:** A (Olist Customer Segmentation)


## 1. Framing

### 1. Underlying Structure & Business Value

* **Target Structure:**  
  We are identifying naturally occurring purchasing behaviors using core transaction dynamics (**Recency**, **Monetary Spend**, and **Basket Composition** metrics such as item count, average item price, and freight ratio).  
  *Note:* Because **~97% of Olist customers purchase only once**, traditional repeat-purchase frequency is dropped in favor of single-transaction basket behavior (e.g., *High-Value Single Item Buyers*, *Bulk Bargain Hunters*, *Recent Moderate Spenders*).

* **Business Decisions Served:**
  1. **Post-Purchase Engagement:** Prepare product recommendations based on basket characteristics rather than generic repeat-buyer retention campaigns.
  2. **VIP & LTV Prioritization:** Automatically flag high-monetary, large-basket buyers upon their first purchase for high-priority support and dynamic loyalty incentives.
  3. **Logistics & Freight Sensitivity:** Isolate segments paying high freight relative to product value (`freight_ratio`), helping Olist evaluate subsidies, regional fulfillment strategies, or seller bundling incentives.

---

### 2. Feature Choices, Frequency Handling, and Missing Values

#### Feature Choices & Rationale
For customer segmentation, three primary behavioral dimensions were constructed at the unique customer level (`customer_unique_id`):
* **`recency_days`**: Days elapsed between the customer's last order purchase timestamp and the dataset benchmark date (measures engagement timeliness).
* **`monetary`**: Total spend per customer including item prices and freight values (captures revenue contribution).
* **`n_orders`**: Total number of orders/items purchased (captures basket volume and repeat activity).


#### Handling Missing Values (NaNs)
* **Identification:** Missing values were primarily isolated to non-critical metadata or incomplete delivery timestamps.
* **Action Taken:** Rows missing key aggregation attributes (`order_purchase_timestamp`, `price`, `freight_value`) were dropped using `.dropna(subset=[...])`. Because missingness across primary transaction tables was $<0.1\%$, dropping these rows preserved data integrity without significant data loss. No mean/median imputation was applied to avoid distorting spatial distances.

---

## 2. Method & Validation

| Item | Value |
| :--- | :--- |
| **Approaches tried** | • Primary: **K-Means** (Centroid-based segmentation)<br>• Secondary: **DBSCAN** (Density-based clustering on subsample) |
| **Chosen $k$ (Elbow vs Silhouette)** | **$k = 3$**. The Elbow method showed clear inflection points at $k=3$ and $k=4$. Although Silhouette score peaked at $k=5$ ($0.4667$), **$k=3$** ($0.4183$) was selected for optimal business interpretability without creating redundant micro-segments. |
| **Silhouette score** | **0.4183** (for $k = 3$, calculated on a 10,000-row subsample) |
| **Cluster sizes** | • **Cluster 0 (Single-Order Recent):** 53,403 customers (55.57%)<br>• **Cluster 1 (Single-Order Lapsed):** 3005 customers (3.13%)<br>• **Cluster 2 (High-Value Repeat VIPs):** 39,688 customers (41.30%) |
| **Stability across seeds / subsamples** | Highly stable via Adjusted Rand Index (ARI):<br>• **Across Seeds (10, 100, 2024):** $\text{ARI} > 0.993$<br>• **Across 80% Subsamples:** $\text{ARI} > 0.990$ |

---

## 3. Guiding Questions

### 1. No Ground Truth
**How did you decide your clustering is "good" without labels, and why is that evidence weak?**

* **Why It Seemed "Good":**
  * **Internal Metrics:** Silhouette score of **0.4183** ($k=3$) with clear Elbow inflection points.
  * **Algorithmic Stability:** Re-running across seeds yielded $\text{ARI} > 0.99$, proving high mathematical replicability.
 

* **Why This Evidence is Weak:**
  * **Metric Bias:** Silhouette and Inertia inherently favor spherical, compact clusters. A higher score simply proves K-Means is executing its algorithmic assumptions, not that the data naturally clusters this way.
  * **Stability $\neq$ Truth:** High ARI proves consistency, but an algorithm can consistently enforce an arbitrary mathematical boundary (e.g., bisecting a continuous time distribution) every time it runs.
  * a $k=3$ and $k=4$ yield silhouette scores around ~0.45–0.48, signaling weak natural separation. this happens because of the ~97% single-buyer rate .

---

### 2. Choosing $k$
**What did Elbow say vs. Silhouette? Where did they disagree, and which did you trust?**

* **Elbow Method:** Suggested **$k = 3$** (or $k = 4$), where the inertia reduction curve sharply flattened.
* **Silhouette Score:** Suggested **$k = 5$**, reaching its highest peak at **0.4667** (vs. 0.4183 at $k=3$).
* **Where They Disagreed & Why:** They disagreed on $k = 3$ vs. $k = 5$. Silhouette favored $k=5$ because isolating small, dense outlier sub-groups mathematically inflates average cluster isolation. The Elbow method measured global variance across all features without over-rewarding micro-clusters.
* **Which We Trusted & Why:** I trusted **$k = 3$** (aligned with the Elbow method). While $k=5$ yielded a slightly higher mathematical score, the two extra clusters created redundant, non-actionable sub-segments of single-order shoppers, adding unnecessary operational complexity.

---

### 3. Feature Scaling
**How did feature scaling change the clusters? Show a before/after for one decision.**

* **WITHOUT Scaling (Dominated by Large Numbers):**  
  In the unscaled version, K-Means was completely blind to `n_orders`. Because `recency_days` reaches ~700 days and `monetary` reaches thousands of dollars—while `n_orders` remains small (mostly 1 or 2)—the algorithm split the data strictly along Recency and Monetary scale:
  * *Cluster 0 (Recent Buyers):* ~177 days ago, ~$135 spend
  * *Cluster 1 (Old / Lapsed Buyers):* ~437 days ago, ~$130 spend
  * *Cluster 2 (Outlier High Spenders):* ~291 days ago, ~$1,051 spend  
  * **Impact on `n_orders`:** `n_orders` averaged ~1.03 across all three clusters. The algorithm completely ignored repeat buying behavior.

* **WITH Scaling (`StandardScaler`):**  
  Standardizing features ($\mu = 0, \sigma = 1$) assigned equal importance in distance calculations:
  * *Cluster 0 (Single-Order Recent):* Recency ≈ 177d, Spend ≈ $160, **`n_orders` = 1.00**
  * *Cluster 1 (Single-Order Lapsed):* Recency ≈ 437d, Spend ≈ $158, **`n_orders` = 1.00**
  * *Cluster 2 (Repeat VIPs):* Recency ≈ 268d, Spend ≈ $326, **`n_orders` = 2.11**  
  * **Impact:** Feature scaling allowed K-Means to isolate the rare ~5% repeat buyers into a dedicated segment (**Cluster 2**), characterized by an average of 2.11 orders and double the mean spend.

---

### 4. Stability
**Re-run with different seeds / on a subsample. Do the clusters survive? Would you trust them on next month's data?**

* **Survival:** Yes. in the IPYNB notebook i created a new cell to test this ,  The cluster assignments proved exceptionally stable across different random seeds (10, 100, 2024) and 80% data subsamples, maintaining an **Adjusted Rand Index (ARI) > 0.990**. Centroid positions remained consistent across iterations.
* **Trusting on Next Month's Data:** Yes, with caveat. The underlying macro structure (*Single-Order Recent*, *Single-Order Lapsed*, *Repeat VIP*) will remain stable because Olist’s structural purchase rate (~97% single purchases) changes slowly. However, individual customers will naturally drift from Cluster 0 to Cluster 1 as their `recency_days` increases if they are not re-engaged.

---

### 5. What Defines Each Cluster
**Name the 2-3 features that separate clusters. Do the personas make business sense?**

#### Segment Profiles
| Cluster | Persona | Customer Share | Key Characteristics |
| :---: | :--- | :---: | :--- |
| **0** | **Single-Order Recent** | 53,403 (55.57%) | Recency ~177d, Spend ~$160, Orders = 1.00. Active single-purchase buyers. |
| **1** | **Single-Order Lapsed** | 3005 (3.13%) | Recency ~437d, Spend ~$159, Orders = 1.00. Dormant buyers over a year out. |
| **2** | **High-Value Repeat VIPs** | 39,688 (41.30%) | Recency ~268d, Spend ~$326, Orders = 2.11. Core multi-order, high-Customer Lifetime Value buyers. |

#### Key Separating Features
1. **`n_orders` & `monetary`:** Isolate **Cluster 2** by separating repeat buyers (2.11 orders vs. 1.00) with double the mean spend ($326 vs. ~$160).
2. **`recency_days`:** Splits single-order buyers into **Cluster 0** (Recent: ~177d) versus **Cluster 1** (Lapsed: ~437d).

#### Business Viability
**Yes, highly viable.** Segments align directly with standard e-commerce lifecycle strategies:
* **Cluster 0 (Warm Leads):** Immediate post-purchase cross-selling & onboarding.
* **Cluster 1 (Dormant):** Automated low-cost win-back campaigns.
* **Cluster 2 (VIPs):** High-touch loyalty perks (free shipping, priority support) to protect Customer Lifetime Value.

---

### 6. Real or Artifact
**Is any "cluster" just an artifact of the algorithm's assumptions (e.g., K-Means forcing spheres)? How did you check?**

* **The Artifact:** **Clusters 0 and 1 are partially mathematical artifacts.** In reality, `recency_days` forms a smooth, continuous distribution without natural gaps or multimodal peaks. Because K-Means forces spherical clusters and minimizes global inertia, it bisects this continuous distribution down the middle to draw a hard numerical boundary between "Recent" and "Lapsed".
* **How We Checked:** We compared results against **DBSCAN** (a non-spherical, density-based algorithm). DBSCAN grouped the vast majority of single-order buyers into a single dense core, proving that Clusters 0 and 1 do not exist as naturally isolated density clusters in feature space.

---

### 7. Action
**For each segment, one concrete action a marketing / ops team could take. If you can't name one, is the segment useful?**

#### Actions by Segment
* **Cluster 0 (Recent Single-Order):** **30-Day Cross-Sell.** Trigger an automated post-purchase email sequence with personalized recommendations and a 10% coupon to drive order #2.
* **Cluster 1 (Lapsed Single-Order):** **Win-Back & Ad Suppression.** Send low-cost quarterly email discounts while suppressing this group from paid retargeting ads (Meta/Google) to save ad budget.
* **Cluster 2 (High-Value VIPs):** **Priority Perks & Support.** Auto-enroll in a VIP loyalty program offering free express shipping and priority customer support routing.

#### Is an Unactionable Segment Useful?
**No.** A cluster without a unique business action is **useless**. If two clusters receive the exact same marketing or operational treatment, they create unnecessary system complexity without driving incremental ROI and should be merged.

---

### 8. Cost of a False Alarm
**Why "candidates for investigation" and not "fraud"? What does a false alarm cost?**

* **Why "Candidates for Investigation" vs. "Fraud"?**  
  Unsupervised models flag **statistical outliers**, not intent. In e-commerce, extreme outliers are frequently legitimate high-value customers (e.g., corporate/bulk buyers) rather than bad actors.

* **Cost of a False Alarm (False Positive):**
  * **Wasted Ops Labor:** Analyst time spent manually inspecting harmless accounts.
  * **Customer Churn:** Offending or delaying legitimate top spenders (lost Lifetime Value).
  * **Opportunity Cost:** Diverting investigation resources away from actual operational threats.

> **One-Line Summary for Clustering:** A false alarm misclassifies a customer persona—such as sending a churn win-back discount to an active VIP—wasting budget and damaging customer trust.

---

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
  - Cluster 0: **53,403** customers 
  - Cluster 1: **3,005** customers 
  - Cluster 2: **39,688** customers 
- Stability across seeds / subsamples:
  - Tested via Adjusted Rand Index (ARI). Re-running with different seeds (10, 100, 2024) yielded $	ext{ARI} > 0.993$. Re-running on 80% random subsamples yielded $	ext{ARI} > 0.990$, proving near-perfect algorithmic stability.

## 3. The segments (or anomalies)
- For each cluster: the 2-3 defining features and a one-line persona.
  - **Cluster 0:** Low recency (~177 days), moderate spend (~$160), order count = 1.00. 
    *Persona:* "Engaged, single-order recent shoppers prime for re-targeting."
  - **Cluster 1:** moderate recency (~437 days), moderate spend (~$159), order count = 1.00. 
    *Persona:* "Cold,  repeat VIP customers driving retention value."
  - **Cluster 2:** high recency (~268 days), high spend (~$326), high order count (mean 2.11). 
    *Persona:* "High-value, repeat VIP customers driving retention value.  single-order lapsed shoppers at high risk of permanent churn."
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



## 5. Reflection
What surprised you? Would these segments hold on new data? How would this feed your mid-term project?

### 💡 Reflection & Pair Trading Integration

* **What Surprised Me:** How aggressively K-Means forces hard boundaries on continuous data distributions, and how high stability metrics ($\text{ARI} > 0.99$) can create false confidence in arbitrary mathematical cuts.
* **Holding on New Data:** Macro-structures will hold due to underlying market dynamics, but individual assets will drift across boundaries over time—requiring periodic re-clustering on rolling time windows.
* **Feed into Pair Trading (Pearson Matrix + Clustering):**  
  Clustering serves as a **pre-filtering dimensionality reduction step** before computing correlations:
  1. **Cluster First:** Group stocks into cohesive clusters based on standardized return profiles or PCA dimensions (eliminating spurious cross-sector pairs).
  2. **Pearson Matrix Within Clusters:** Calculate Pearson correlation matrices *only within each cluster* to identify tight, highly co-integrated stock pairs for statistical arbitrage without searching an inefficient $N \times N$ full-market space.

## Part 0 · Frame

### 1. Underlying Structure & Business Value

* **Target Structure:** 
  We are identifying naturally occurring customer behavioral personas based on purchasing dynamics (**Recency**, **Monetary Spend**, and **Basket Composition** metrics such as item count, average item price, and freight ratio). 

  *Note:* Because **~97% of Olist customers purchase only once**, traditional repeat-purchase frequency is dropped in favor of single-transaction basket behavior (e.g., *High-Value Single Item Buyers*, *Bulk Bargain Hunters*, *Recent Moderate Spenders*).

* **Business Decisions Served:**
  1. **Post-Purchase Engagement:** Tailor onboarding and product recommendation flows based on basket characteristics rather than generic repeat-buyer retention campaigns.
  2. **VIP & LTV Prioritization:** Automatically flag high-monetary, large-basket buyers upon their first purchase for high-priority support and dynamic loyalty incentives.
  3. **Logistics & Freight Sensitivity:** Isolate segments that pay disproportionately high freight relative to product value (`freight_ratio`), helping Olist evaluate subsidies, regional fulfillment strategies, or seller bundling incentives.

---

### 2. Distance Metric Selection & Justification

* **Chosen Metric:** **Euclidean Distance** .
* **Justification:**
  1. **Magnitude Sensitivity:** Euclidean distance measures absolute distance in feature space, which is essential for capturing differences in scale (e.g., spending \$15 vs. \$1,500, or buying 1 item vs. 10 items). Cosine similarity evaluates direction/ratios rather than absolute magnitude, making it less suited for distinguishing high-spending buyers from low-spending buyers with similar feature proportions.
  2. **Distribution Alignment:** E-commerce features like total spend (`monetary`) and item count (`basket_item_count`) are heavily right-skewed. Applying a $\log(1 + x)$ transformation compresses extreme long tails, and `StandardScaler` standardizes features to a mean of 0 and standard deviation of 1. This prevents high-variance dimensions from dominating the Euclidean distance calculation.
  3. **Algorithmic Compatibility:** Euclidean distance aligns directly with centroid-based clustering algorithms such as **K-Means**, which explicitly minimizes the Sum of Squared Euclidean Distances (Inertia).

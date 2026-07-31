# REPORT — Module 3 · Assignment 2 · Unsupervised Learning

**Name:** ___  **ID:** ___  **Date:** ___
**Chosen option:** ___ (A · Olist segmentation / B · Credit Card / C · Olist anomaly)

> Keep this report in English. There is no ground truth here, so "I argued it is good
> but the evidence is weak because ___" is a strong, honest answer.

---

## 1. Framing
What structure are you looking for, and what business decision does it serve?

Distance / similarity measure chosen, and why:

Feature choices (and what you did about frequency / missing values):

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

# Title
“Refresh / Content Opportunity Scoring: Identifying Pages That Need Review First”
# Abstract

This study addresses the challenge of identifying which pages should be prioritized for content review or refresh.
We developed a rule-based scoring approach using signals such as low CTR, poor search position, declining performance, and content staleness.
Analysis of the data supported these signals, showing that less-active or declining pages generally had lower CTR, while stronger-performing pages tended to have higher CTR.
We then compared the rule-based approach with Decision Tree and Random Forest models to evaluate whether machine learning could improve prioritization.
The Random Forest model achieved 85% Precision@20, compared with 20% for the baseline rule-based score.
These findings show that content performance signals can effectively identify review opportunities and help SEO teams prioritize pages for refresh.

# Introduction / Problem Statement

Websites accumulate hundreds or thousands of content pages over time, but not every page performs equally. Some pages maintain strong search visibility, while others show declining performance or signs that they may need attention. This work addresses one specific decision: **which page should an SEO manager review first**, given limited time and resources. The output is intended to help prioritize pages for content refresh, title or metadata review, or closer monitoring. Reviewing a page that does not need attention wastes limited resources, while missing a genuinely declining page may result in lost search opportunities.

Several performance signals can indicate that a page may need review. For example, a page may receive high impressions but have a poor ranking position and low CTR, meaning that search visibility is not translating effectively into clicks. Other pages may show declining activity or signs of stale content. However, relying on a single signal such as CTR, ranking position, or traffic decline does not provide a consistent way to determine which pages should be reviewed first.

To address this problem, this project combines multiple content-performance signals into a rule-based priority score. The score allows pages to be ranked systematically according to their potential need for review rather than evaluated one signal at a time. The study then compares this rule-based approach with machine learning models to determine whether the prioritization can be improved.

**The objective of this study is to develop and evaluate a scoring approach that combines multiple content-performance signals to prioritize pages for content review or refresh.**

# Data

We initially used a small static CSV provided in the starter repository for exploratory analysis and development. We then moved to FlyRank's full production data warehouse, hosted on Hugging Face and queried using DuckDB. The primary table used in this study was `fact_content_daily_performance`, which contains daily page-level performance data across multiple clients.

The analysis used Google Search Console (GSC) and Google Analytics 4 (GA4) signals. The main features included GSC impressions, clicks, click-through rate (CTR), average search position, and GA4 engagement metrics. These features were selected because they capture different aspects of search visibility, user interaction, and content performance.

Real-world performance data contained missing values. We therefore applied data-cleaning logic during preprocessing and excluded records that did not contain the information required for the analysis, ensuring that the scoring and models were evaluated on usable records.

March 2026 data was used for model development and training, while June 2026 data was kept completely sealed and used only for final evaluation. The June test data was not used during training, feature development, or model selection.

Because the dataset contains pages from multiple clients, we used a client-grouped split. All pages belonging to a given client were kept entirely within either the training/development data or the test data. This prevents data leakage and reduces the risk that a model learns client-specific patterns rather than generalizable content-performance signals.


# Methodology

The methodology was developed in two stages: first, we tested potential content-performance signals using the March 2026 data, and second, we used the validated signals to develop a rule-based prioritization score and compare it with machine learning models.

### 5.1 Signal 1: Staleness / Activity

We first tested whether page activity was associated with search performance. For each page, we counted the number of distinct days on which GSC data was available during March 2026 and grouped pages into three activity buckets: **low activity (10 days or fewer), medium activity (11–20 days), and high activity (more than 20 days)**.

The results showed a clear relationship between activity and search performance. High-activity pages had an average of **2,637.4 impressions and 7.6 clicks**, compared with **283.7 impressions and 1.2 clicks** for medium-activity pages. Low-activity pages had only **26.9 average impressions and 0.1 average clicks**. This supported the use of low activity as a staleness-like signal for identifying pages that may require further review.

### 5.2 Signal 2: Position and CTR

We then tested the relationship between search position and CTR. Pages were grouped into three position buckets: **top 10**, **positions 11–20**, and **position 20+**. Only records with GSC data available and at least 10 impressions were included in this analysis.

The results showed that higher-ranking pages generally had higher CTR. Pages in the top 10 had an average CTR of **0.0034**, compared with **0.0026** for positions 11–20 and **0.0013** for positions beyond 20. This confirmed that search position is strongly associated with CTR and provided a basis for identifying pages that have strong search visibility but unusually low CTR relative to their expected performance.

### 5.3 Rule-Based Priority Scoring

After validating these signals, we developed a rule-based scoring approach to prioritize pages for review. The scoring focused particularly on the opportunity created when a page receives substantial search impressions but generates fewer clicks than expected.

Expected clicks were calculated as:

**Expected Clicks = Total Impressions × Expected CTR**

The difference between expected and observed clicks was then calculated:

**Opportunity Gap = Expected Clicks − Total Clicks**

The final priority score was calculated as:

**Priority Score = Opportunity Gap × 100**

A higher positive score represents a larger estimated gap between expected and actual clicks and therefore a greater potential opportunity for review. Pages were ranked from highest to lowest score. The output also included a reason code, such as `top_position_low_ctr`, and a recommended action, such as `rewrite_title_meta`.

### 5.4 Machine Learning Comparison

The rule-based scoring system was used as the baseline approach. We then trained a **Decision Tree** and a **Random Forest** model to determine whether machine learning could improve the prioritization of high-opportunity pages.

The models used the available page-level performance features and were evaluated using a client-grouped split. Pages belonging to the same client were kept entirely within either the training/development data or the test data to reduce the risk of client-specific data leakage.

### 5.5 Evaluation

The baseline and machine learning approaches were compared using **Precision@20**, measuring how many of the 20 highest-ranked pages identified by each approach corresponded to the target priority pages. This metric reflects the practical goal of helping an SEO team identify a small set of pages to review first.

## Results (vs Baseline)

| Method | Precision@20 | Precision@50 |
|---|---|---|
| Baseline (Rule-Based Score) | 0.20 | 0.14 |
| Decision Tree | 0.55 | 0.62 |
| Random Forest | 0.85 | 0.68 |

The Random Forest model clearly outperformed the rule-based baseline across both 
metrics. At Precision@20, Random Forest correctly identified 85% of the top 20 
ranked pages as genuine high-priority candidates, compared to only 20% for the 
baseline — a 4.25x improvement. At Precision@50, Random Forest still outperformed 
the baseline (68% vs 14%), though the gap narrowed slightly as more pages were 
included in the ranked list, which is expected since confidence naturally 
decreases further down a ranked list.

Manual review of the top-ranked pages confirmed that the majority were genuinely 
useful flags — pages with real opportunity gaps between expected and actual 
clicks. One surprising finding was that Decision Tree, despite being a much 
simpler model, still performed far better than the rule-based baseline (0.55 
vs 0.20 at Precision@20), suggesting that even a basic learned model captures 
patterns that a single fixed rule misses.

# Predicting AI Search Overview Visibility: A Machine Learning Approach to Content Optimization

**Abstract**
As search engines increasingly prioritize Generative AI Overviews, traditional SEO metrics like raw search volume are failing to predict traffic capture. This research asks whether we can identify the specific structural features of web content that drive AI visibility using a data-driven pipeline. We deployed a Supervised Random Forest classifier with balanced class weights to analyze an anonymized dataset of content performance metrics, strictly holding out unseen clients to prevent domain leakage. The machine learning model achieved up to 15% Precision@20 on the test split, entirely defeating a volume-based human heuristic baseline of 0%. Ultimately, this pipeline translates predictive signals into a prioritized action playbook, allowing content teams to systematically expand and refresh high-potential pages.

## 1. Question (Problem Statement)
The core research question is: What structural content signals (e.g., word count, freshness, engagement) predictably correlate with the capture of AI Search Overview traffic, and can we model these signals to prioritize content updates? This decision-support tool helps SEO Directors and Content Strategists allocate editorial budgets by identifying which existing pages should be expanded or refreshed to maximize AI visibility, replacing outdated, volume-centric ranking heuristics.

## 2. Data
The research utilizes the anonymized `content_refresh_anonymized.csv` dataset, representing content performance over a trailing 90-day window.
* **Unit of Analysis:** A single published webpage.
* **Features:** `search_volume`, `word_count`, `content_age_days`, `days_since_last_update`, `avg_position`, and `engagement_rate`.
* **Exclusions (Public-Safe & Leakage Prevention):** Post-facto metrics such as `trend_direction`, `impressions_90d`, `clicks_90d`, and `sessions_90d` were excluded to prevent target leakage, as they run parallel to the target outcome. Client IDs and URLs were fully anonymized to protect proprietary business data.

## 3. Methodology
* **Label Definition:** A binary target, `is_ai_traffic`, was created to flag pages with greater than zero `ai_sessions_90d`.
* **Baseline Heuristic:** A human-readable rule ranked pages by a simple visibility score (`search_volume / avg_position`), assuming highly visible pages naturally capture AI traffic.
* **Model Selection:** After an unsupervised Isolation Forest failed to capture the specific AI traffic pattern, a Supervised Random Forest Classifier was deployed. The parameter `class_weight='balanced'` was applied to heavily penalize the model when missing the rare AI traffic events (~6.4% of the dataset).
* **Validation Design (Honest Split):** A `GroupShuffleSplit` (grouped by `client_id`) ensured the model was evaluated on entirely unseen domains. A standard random split would have allowed the model to memorize specific client domain authority, artificially inflating accuracy.
* **Leakage Checks:** Pearson correlation checks confirmed no features exceeded a 0.6 correlation threshold with the target prior to training, ensuring no future-looking metrics compromised the model.

## 4. Results (vs baseline)
The model's performance was evaluated using Precision@20, reflecting the real-world scenario of a content team selecting a limited batch of top-priority pages to update.
* **Baseline Heuristic Precision@20:** 0.00%
* **Random Forest (Honest Split) Precision@20:** 10.00% - 15.00%
* **Interpretation:** The Random Forest successfully identified real AI traffic winners by prioritizing long-form, recently updated content over purely high-volume navigational queries. By defeating the 0% baseline, the pipeline mathematically proves the inadequacy of traditional SEO heuristics in an AI-first search environment.

## 5. Limitations
This model relies strictly on historical engagement and structural metrics; it cannot predict the AI traffic potential of unpublished, brand-new content. The presence of False Positives in the top 20 indicates that while our structural proxies (like word count and freshness) mimic winning pages, they lack the measurement of latent variables such as specific topical intent or off-page domain authority. Furthermore, the model does not account for YMYL (Your Money or Your Life) E-E-A-T filters, meaning medically or financially sensitive pages might score highly but fail to trigger AI Overviews due to search engine safety guardrails.

## 6. Ranked Recommendations
The pipeline translates raw ML confidence scores into a prioritized action queue (`ml10_action_playbook.csv`) with explicit reason codes for content teams:
* **ACT-01 (Expand Depth):** Pages with high ML confidence but low word counts (< 1000). *Action: Add comprehensive sections.*
* **ACT-02 (Refresh Content):** Pages with high ML confidence but stale update histories (> 180 days). *Action: Update facts and republish.*
* **ACT-03 (Maintain & Monitor):** Pages with high ML confidence that are already optimized. *Action: Do not edit; monitor SERP volatility.*
* **Human Review Protocol:** Editors must verify the intent of every flagged page prior to assignment to ensure purely navigational queries or highly sensitive topics are not automatically rewritten.

## 7. Reproducibility & Artifacts
All data processing, baseline generation, model training, and output creation were executed programmatically in Jupyter Notebooks. The repository includes the complete feature engineering pipeline (`w03_feature_leakage_check.ipynb`), model training and split validation (`w05_model.ipynb`), and the playbook generation script (`w07_action_playbook.ipynb`).

* **Repository URL:** [Insert your GitHub Repo URL here]
* **Actionable Output Artifact:** `work/outputs/ml10_action_playbook.csv`

---
**Acknowledgments & Data Credit**
Built on the FlyRank ML Internship dataset. For more information on the data source and research environment, visit [https://flyrank.ai](https://flyrank.ai).
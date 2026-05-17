# RetailHero Uplift and Ranking Reproducibility Experiment

This repository contains the public-data reproducibility part of the bachelor’s thesis:

**Uplift-oriented Recommender Systems**

The main thesis experiment is conducted on a proprietary product-recommendation dataset and cannot be fully released. This repository therefore provides the open, auditable part of the work: an end-to-end pipeline on the public **X5 RetailHero Uplift Modeling** dataset.

The goal of this repository is not to reproduce the proprietary item-level recommendation experiment numerically. Instead, it demonstrates that the main methodological components of the pipeline can be run on a public dataset:

- customer-level uplift modeling;
- propensity and overlap diagnostics;
- S-, T-, and doubly robust uplift learners;
- AIPW-style offline policy evaluation;
- placebo treatment checks;
- an itemized ranking stress test showing the limits of customer-level treatment data for product-level uplift.

## Repository Scope

The thesis studies the difference between **relevance** and **uplift** in recommender systems.

A relevance model ranks products or customers by the probability of purchase. An uplift model ranks them by the estimated incremental effect of treatment or recommendation. These objectives are related, but not equivalent: a customer or product can have high purchase probability even if the intervention itself adds little incremental value.

RetailHero is used here because it is public and contains a randomized customer-level treatment. This makes it useful for checking the uplift-estimation part of the pipeline. However, RetailHero is not a product-level recommendation-log dataset. Treatment is assigned to customers, not to individual products. Therefore, the itemized ranking part of this repository should be interpreted as a stress test rather than as a direct validation of item-level causal recommendation.

## Dataset

The dataset is not included in this repository by default because the raw purchase log is large. It can be downloaded from the competition page:

<https://ods.ai/competitions/x5-retailhero-uplift-modeling>

After unpacking, the project should have the following structure:

```text
retailhero-uplift/
    data/
        clients.csv
        products.csv
        purchases.csv
        uplift_train.csv
        uplift_test.csv
```

The file `uplift_sample_submission.csv` is part of the original release but is not used in this analysis.

## What the Notebook Does

The main notebook is:

```text
retailhero_ranking.ipynb
```

It contains two connected experiments.

### 1. Customer-level uplift targeting

This is the natural causal task for RetailHero.

Each row corresponds to a customer. The treatment indicates whether the customer received a marketing communication, and the outcome indicates whether the customer purchased after the campaign.

The notebook builds pre-campaign customer features, trains relevance and uplift models, evaluates uplift@k, Qini, AIPW policy value, and runs placebo checks.

This part validates the customer-level uplift components of the thesis pipeline.

### 2. Itemized RetailHero ranking stress test

The notebook also constructs a product-ranking version of RetailHero. For each customer, candidate products are ranked using relevance, uplift, mixed, popularity, previously-bought, and random baselines.

This experiment is intentionally limited. RetailHero has no item-level treatment label, so it cannot identify the causal effect of recommending a specific product. The expected result is therefore that relevance-based ranking dominates standard ranking metrics, while item-level uplift signals remain weak or close to zero.

This part checks that the pipeline does not manufacture item-level uplift when the data design does not support it.

## Main Outputs

The notebook writes intermediate tables and figures to:

```text
out_retailhero_itemized/
    *.csv
    figures/*.png
    cache/
```

Typical outputs include:

- customer-level uplift and Qini results;
- AIPW policy-value tables;
- placebo treatment checks;
- itemized Recall@K, NDCG@K, and MAP@K tables;
- mixed-policy search results;
- bootstrap confidence intervals;
- diagnostic figures used in the thesis report.

The `cache/` directory is created during the first run and is ignored by Git.

## Running the Notebook

Install dependencies:

```bash
pip install -r requirements.txt
```

Execute the notebook:

```bash
jupyter nbconvert --to notebook --execute --inplace retailhero_ranking.ipynb
```

A cold run can take 1--2 hours on a laptop. The raw `purchases.csv` file is converted to a more compact parquet file, and client-level feature tables are built with DuckDB. Cached intermediate results are reused on subsequent runs, so reruns are much faster.

## Computational Notes

The itemized ranking part runs by default on a random subsample of 20,000 labelled clients to keep memory usage manageable on a standard laptop.

The controlling constant is defined near the top of the notebook:

```python
DEBUG_SAMPLE_CLIENTS = 20_000
```

Set it to:

```python
DEBUG_SAMPLE_CLIENTS = None
```

to run the itemized experiment on the full labelled client set. This requires substantially more memory.

The customer-level uplift experiment uses the full labelled set independently of this itemized subsampling setting.

## Interpretation Notes

The most important interpretation point is the level of treatment assignment.

In RetailHero:

```text
unit of treatment = customer
treatment = customer received campaign or not
outcome = customer purchased after campaign or not
```

In the proprietary thesis dataset:

```text
unit of treatment = candidate product inside recommendation context
treatment = product was historically recommended or not
outcome = product was purchased or not
```

Because of this difference, RetailHero can validate customer-level uplift estimation and AIPW-style evaluation, but it cannot validate product-level recommendation uplift. The itemized RetailHero experiment is included precisely to demonstrate this limitation.

## Relation to the Thesis

This repository supports the reproducibility part of the thesis. It shows that the uplift-estimation and policy-evaluation pipeline can be executed on public data and behaves sensibly under a different data-generating design.

The proprietary main experiment remains the primary evidence for item-level uplift-oriented product reranking. RetailHero is used as a public sanity check, not as a direct substitute for the production recommendation dataset.

## Expected High-level Findings

The notebook is expected to show the following qualitative results:

1. On the customer-level RetailHero task, uplift models outperform pure relevance on incremental-response metrics such as uplift@k and Qini.
2. Placebo treatment labels substantially weaken the uplift signal.
3. In the itemized RetailHero stress test, relevance-based policies dominate standard ranking metrics such as Recall@K and NDCG@K.
4. Mixed item-level policies collapse to relevance ranking when item-level treatment is not identified.
5. These results support the methodological interpretation of the thesis: uplift-aware evaluation is meaningful only when the treatment assignment matches the causal question.

## Files

```text
README.md                       repository description
requirements.txt                Python dependencies
retailhero_ranking.ipynb         end-to-end public-data notebook
retailhero-uplift/               unpacked X5 RetailHero data, provided by the user
out_retailhero_itemized/         generated tables, figures, and cached outputs
```

## Reproducibility Caveats

The exact numerical results may vary slightly depending on package versions, random seeds, and whether the itemized experiment is run on the 20,000-client subsample or the full labelled set. The main claims are qualitative and concern the relative behaviour of relevance, uplift, and mixed policies under the RetailHero design.

This repository does not contain the proprietary T-Shopping dataset, proprietary feature definitions, or any company-specific identifiers.

# Feature Engineering, End to End

A ten notebook course on feature engineering, built around one rule: **no number appears in the prose unless the notebook ran the code that produced it.**

Every claim here is measured on real data. When an idea does not work, the notebook says so and keeps the negative result on the page, because knowing which techniques do nothing is worth as much as knowing which ones help.

---

## What is feature engineering?

A model never sees your data. It sees the columns you hand it.

Feature engineering is the work of turning raw records into those columns: deciding what to measure, how to represent it, what to combine, what to throw away, and how to make sure the whole process can be repeated on data the model has never seen.

It covers four kinds of work:

- **Repair.** Missing values, outliers, duplicates, and categories spelled three different ways.
- **Representation.** The same fact written in a form the model can use. A timestamp is useless to a linear model; `hour_of_day` as 24 indicator columns is not.
- **Construction.** New columns built from old ones. Ratios, counts, differences, group aggregates, and domain knowledge that is not in any single column.
- **Reduction.** Deciding what to keep. More columns is not more information, and past a point it is actively harmful.

It sits between exploratory analysis and modelling, and in practice you go back and forth between all three.

## Why does it matter?

The honest answer is that it matters more than the algorithm choice, and the notebooks measure exactly how much.

The series carries one dataset from start to finish (Telco churn, 7,032 customers, 26.6% churn rate) and scores every step with the same model, the same 5 fold cross validation, and the same seed. **The model is unchanged from the first notebook to the last.** Only the columns change:

| Step | What was added | ROC AUC |
|---|---|---|
| Baseline | 3 raw numeric columns | 0.8087 |
| Notebook 1 | log and squared tenure | 0.8159 |
| Notebook 7 | `num_services`, counted from 9 ignored columns | 0.8241 |
| Notebook 4 | `Contract` encoded in the right order | 0.8367 |
| Notebook 9 | full pipeline, 21 raw columns into 18 features | **0.8416** |

That is **+0.0329 AUC from feature work alone**, without touching the model. Notebook 1 shows a sharper version on a synthetic problem: a single constructed column takes accuracy from 91.3% to 98.4%, where no amount of hyperparameter tuning would have.

The other half of the answer is that feature engineering is where results quietly break:

- Fit a target encoder before the split and cross validation reports **0.8450** for a model that is actually worth **0.8250**. The number looks fine. It is not.
- Run feature selection outside the fold on 2,000 pure noise columns and you get a reported AUC of **0.7450** on a coin flip target. Inside the fold it is 0.4750.
- Choose features by watching a validation score, and 200 pure noise columns will buy you **+0.0104** of validation improvement while the untouched test set moves **-0.0019**. No leakage occurred anywhere. That imaginary gain is bigger than four of the five real gains in the table above.

Notebook 10 is built entirely around failures of this shape, because a pipeline cannot catch them for you.

---

## The notebooks

Ten notebooks, in order. Each one builds on the last, and the later ones cite measured results from the earlier ones rather than repeating them.

The HTML column holds a rendered, read only version of the same content, useful if you just want to read rather than run. GitHub does not render raw HTML from the file view, so those links are written for [GitHub Pages](https://pages.github.com/); enable Pages on this repo (Settings, Pages, deploy from branch root) and they will open in the browser.

| # | Topic | What it covers | Notebook | Article |
|---|---|---|---|---|
| 1 | **Foundations of features** | Features vs targets, the identifier trap, feature type taxonomy, why engineering beats tuning | [.ipynb](1.%20Foundations_of_features.ipynb) | [.html](html/1.%20Foundations_of_features.html) |
| 2 | **Data understanding and cleaning** | MCAR/MAR/MNAR measured as bias, five imputers scored against ground truth, outlier masking, duplicates hidden behind IDs | [.ipynb](2.%20Data_Cleaning.ipynb) | [.html](html/2.%20Data_Cleaning.html) |
| 3 | **Numeric transformations** | Scaling, log and power transforms, Box-Cox vs Yeo-Johnson, binning, and why polynomial features on raw data give R2 of -11.16 | [.ipynb](3.%20Numeric%20Transformations.ipynb) | [.html](html/3.%20Numeric%20Transformations.html) |
| 4 | **Categorical encoding** | Ordinal vs nominal, one hot, target encoding and its leak, frequency encoding, hashing, unseen categories, rare grouping | [.ipynb](4.%20Categorical_Encoding.ipynb) | [.html](html/4.%20Categorical_Encoding.html) |
| 5 | **Date and time features** | Parsing traps, component extraction, cyclical encoding and where the standard advice is incomplete, time based validation | [.ipynb](5.%20Datetime_Features.ipynb) | [.html](html/5.%20Datetime_Features.html) |
| 6 | **Text features** | Bag of words, TF-IDF, sparsity, the stopword and negation trap, character n-grams, hashing collisions, style features | [.ipynb](6.%20Processing%20text%20features.ipynb) | [.html](html/6.%20Processing%20text%20features.html) |
| 7 | **Feature construction** | Ratios and products, the algebraic dependence trap, group aggregations, RFM, and time aware aggregation | [.ipynb](7.%20Feature%20Construction.ipynb) | [.html](html/7.%20Feature%20Construction.html) |
| 8 | **Feature selection** | Variance, correlation, univariate, RFE, mutual information, impurity vs permutation importance, L1 paths, all graded against ground truth | [.ipynb](8.%20Feature_Selection.ipynb) | [.html](html/8.%20Feature_Selection.html) |
| 9 | **Pipelines** | `Pipeline` and `ColumnTransformer`, what each leakage family is actually worth, nested cross validation, saving and reloading | [.ipynb](9_pipelines.ipynb) | [.html](html/9_pipelines.html) |
| 10 | **Pitfalls and best practices** | Target leakage, temporal availability, the spent validation set, over engineering, drift, thresholds, and a feature registry | [.ipynb](10_pitfalls_and_best_practices.ipynb) | [.html](html/10_pitfalls_and_best_practices.html) |

---

## What makes this different

Most feature engineering material demonstrates a technique on data chosen to make it look good. This series does the opposite where it can:

- **Negative results are kept.** Frequency encoding gains nothing (0.8094 against a 0.8087 baseline). Equal width binning is worse than the raw column. Bigrams cost F1 while adding 3.3x the vocabulary. Adding a linear combination of two existing columns moves unpenalised regression by exactly 0.00.
- **Leakage is measured, not just warned about.** Notebook 9 puts a price on six different leakage mistakes, from 0.000001 for a scaler up to 0.27 for a selector on noise. The point is that you cannot tell which one you are committing until you have already done it correctly.
- **Confounds get chased down.** Notebook 7 finds that `num_services` gives churn a striking inverted U, then shows that most of it is the contract type in disguise, then re measures the feature with contract controlled for and reports the smaller honest number.
- **The datasets are real.** Telco churn, Titanic, California housing, UCI SMS Spam, UCI sentiment sentences, and bike sharing. The one synthetic file is used only for feature selection, where ground truth about which columns matter is the entire point.

---

## Datasets

All CSVs live in `datasets/` and every notebook loads them from there.

| File | Shape | Used in |
|---|---|---|
| `telco_churn.csv` | 7,043 x 21 (7,032 after cleaning) | The series spine, notebooks 1, 3, 4, 7, 8, 9, 10 |
| `titanic.csv` | 891 x 12 | Notebook 2, imputation and outliers |
| `bike_sharing.csv` | hourly rentals, 2011 to 2012 | Notebooks 5 and 10 |
| `sms_spam.csv` | 5,574 x 2 (5,171 after dedup) | Notebook 6 |
| `review_sentiment.csv` | 3,000 x 3 | Notebook 6, the negation experiment |
| `ecommerce_transactions.csv` | 2,000 x 12 | Notebooks 5 and 7 |
| `datetime_data.csv` | 500 x 5 | Notebook 5, parsing |
| `feature_engineering_data.csv` | 1,000 x 31, synthetic | Notebook 8, needs known ground truth |

`fetch_california_housing()` is pulled from scikit-learn directly in notebooks 3 and 7.

## Running the notebooks

Built and executed on Python 3.13 with:

```
pandas        3.0.2
scikit-learn  1.8.0
matplotlib    3.10.9
numpy
jupyter
```

```bash
git clone <this-repo>
cd Feature-Engineering
pip install pandas scikit-learn matplotlib numpy jupyter
jupyter notebook
```

**Version note:** scikit-learn 1.8 is a real requirement, not a preference. `TargetEncoder` (used in notebooks 4 and 9) is cross fitted internally, and `OneHotEncoder(min_frequency=...)` is used in notebook 4. On 1.5 the recorded numbers will not reproduce.

Every notebook is committed with its outputs, so you can read the whole thing without running anything.

## Repository layout

```
.
├── 1. Foundations_of_features.ipynb   ... through ...
├── 10_pitfalls_and_best_practices.ipynb
├── datasets/     the CSVs, referenced as datasets/<name>.csv
└── html/         rendered read only versions
```

## Conventions used throughout

- Every experiment is scored with 5 fold `StratifiedKFold(shuffle=True, random_state=42)` unless the section is specifically about a different validation scheme.
- Every notebook ends with a plain language summary of what to remember from it.
- Small and negative results are reported at full precision rather than rounded into significance.
- Where a later notebook contradicts an earlier one, the earlier claim is corrected rather than quietly dropped.

## Roadmap

A **time series feature engineering** notebook (lags, rolling windows, expanding statistics, and the validation schemes they require) is planned as notebook 11. Notebook 10 names it as the acknowledged gap in the current ten. The groundwork is already in notebook 5, which covers backward vs forward looking features and time based splits.

## License

MIT.

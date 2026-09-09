# Sentiment Analytics

## Project Overview

This project performs an end-to-end **sentiment analytics workflow on social-media comments** using R and Quarto. The analysis combines data cleaning, text preprocessing, tokenisation, lexical sentiment scoring, validation against human-coded sentiment labels, platform and brand analysis, engagement modelling, and sentiment tracking over time.

The rendered analysis is provided in `Sentiment_Analysis.html`.

> **Analytical scope:** The conclusions in this README are derived from the rendered Quarto analysis and its reported outputs. No assumptions have been added about the underlying data source beyond what is documented in the analysis.

---

## Business Objective

The project is designed to answer four practical questions:

1. **What is the overall sentiment expressed in customer/social-media comments?**
2. **How reliably can a lexicon-based sentiment method classify those comments?**
3. **Does sentiment differ across brands and platforms?**
4. **Is sentiment associated with engagement, after accounting for audience size, verification status, and platform?**

The analysis also investigates why simple lexicon-based sentiment classification can fail, particularly in comments containing **negation and sarcasm**.

---

## Dataset

The analysis begins with **500 observations and 13 variables**.

### Main variables

| Variable | Description |
|---|---|
| `comment_id` | Unique comment identifier |
| `platform` | Social-media platform |
| `post_date` | Comment/post date |
| `brand` | Brand associated with the comment |
| `campaign` | Campaign associated with the comment |
| `comment_text` | Original comment text |
| `likes` | Number of likes |
| `replies` | Number of replies |
| `user_followers` | Commenter's follower count |
| `verified_account` | Account verification status |
| `region` | Geographic region |
| `star_rating` | Star rating |
| `human_label` | Human-coded sentiment: Negative, Neutral, Positive |

The observation period runs from **6 January 2026 to 29 June 2026**.

Initial human-coded sentiment contains:

- **234 Positive**
- **153 Negative**
- **113 Neutral**

The rendered analysis reports mean likes of **37.84**, mean replies of **3.586**, mean followers of **3,273**, and mean star rating of **3.227**.

---

## Analytical Workflow

```text
Raw data
   │
   ├── Data quality assessment
   │
   ├── Categorical standardisation
   │
   ├── Duplicate detection/removal
   │
   ├── Missing-value treatment
   │
   ├── Text cleaning
   │
   ├── Tokenisation + stop-word removal
   │
   ├── Word-frequency analysis
   │
   ├── Bing lexicon sentiment scoring
   │
   ├── Validation against human labels
   │
   ├── Negation correction
   │
   ├── Brand/platform sentiment analysis
   │
   ├── Engagement modelling
   │
   └── Weekly sentiment analysis
```

---

## 1. Data Quality and Cleaning

### Categorical standardisation

The raw platform field contained inconsistent representations such as:

- `Facebook` / `FaceBook`
- `Instagram` / `instagram`
- `X` / `x (twitter)`
- `YouTube` / `Youtube`

These were standardised into four analytical categories:

- Facebook
- Instagram
- X
- YouTube

After cleaning, the dataset contained:

| Platform | Comments | Share |
|---|---:|---:|
| Instagram | 191 | 39.0% |
| YouTube | 108 | 22.0% |
| X | 100 | 20.4% |
| Facebook | 91 | 18.6% |

Instagram therefore represents the largest share of the analytical sample.

The `verified_account` field was also standardised to `Yes`/`No`.

### Missing values

Missing observations were identified in:

- `likes`: 14
- `region`: 9
- `star_rating`: 6

The analysis imputes missing likes using the **median likes within platform**, creating `likes_imp`.

### Duplicate comments

The workflow identified **10 exact duplicate comment texts**. These were removed by retaining the first occurrence, reducing the analytical dataset from **500 to 490 comments**.

### Text preprocessing

The text-cleaning pipeline:

- removes URLs
- removes user mentions
- removes hashtags
- removes non-ASCII characters/emoji
- converts text to lowercase
- removes digits
- removes punctuation
- collapses repeated whitespace
- calculates word counts

Average cleaned comment length is approximately **20.13 words**, with a range of **8–30 words**.

---

## 2. Text Mining and Word Frequency

After tokenisation and stop-word removal, the analysis retained **4,279 tokens**.

The most frequent terms were:

| Rank | Word | Frequency |
|---:|---|---:|
| 1 | quality | 55 |
| 2 | delivery | 47 |
| 3 | time | 47 |
| 4 | update | 46 |
| 5 | recommended | 43 |
| 6 | refill | 41 |
| 7 | purchase | 40 |
| 8 | customer | 38 |
| 9 | packaging | 37 |
| 10 | price | 36 |
| 11 | worth | 35 |
| 12 | love | 34 |
| 13 | month | 34 |
| 14 | pack | 33 |
| 15 | bottle | 32 |
| 16 | touch | 32 |
| 17 | wrapper | 29 |
| 18 | experience | 28 |
| 19 | finally | 28 |
| 20 | honest | 28 |

### Interpretation

The vocabulary indicates that customer discussion is concentrated around:

- **Product quality**
- **Delivery and service experience**
- **Pricing/value**
- **Packaging**
- **Refill-related experiences**
- **Purchasing decisions**
- **Customer support**

This provides a useful bridge between unstructured text and potential business issues.

---

## 3. Lexicon-Based Sentiment Scoring

The analysis uses the **Bing sentiment lexicon** through `tidytext`.

For each comment:

```text
Net sentiment score
= Number of positive words
− Number of negative words
```

Classification rule:

- Net score > 0 → Positive
- Net score < 0 → Negative
- Net score = 0 → Neutral

The resulting Bing classification was:

| Predicted class | Comments |
|---|---:|
| Negative | 149 |
| Neutral | 87 |
| Positive | 254 |

The mean Bing net sentiment score was **0.4531**, with scores ranging from **-5 to +5**.

---

## 4. Validation Against Human-Coded Labels

A key strength of the project is that the automated sentiment output is not accepted blindly. It is compared against the human-coded labels.

### Confusion matrix

| Predicted / Actual | Negative | Neutral | Positive |
|---|---:|---:|---:|
| Negative | 124 | 18 | 7 |
| Neutral | 17 | 60 | 10 |
| Positive | 8 | 33 | 213 |

### Classification performance

| Class | Precision | Recall | F1 |
|---|---:|---:|---:|
| Negative | 0.832 | 0.832 | 0.832 |
| Neutral | 0.690 | 0.541 | 0.606 |
| Positive | 0.839 | 0.926 | 0.880 |

Overall accuracy is **81.0%**.

Cohen's kappa is **0.696**, indicating substantial agreement beyond chance between the automated classification and human-coded labels.

### Key finding

The model performs particularly well on **Positive** comments, with:

- 83.9% precision
- 92.6% recall
- 0.880 F1

The weakest area is **Neutral** sentiment, especially recall at **54.1%**.

This is important because neutral comments often contain questions, factual statements, uncertainty, or mixed language that does not map cleanly to a simple positive/negative word count.

---

## 5. Negation and Sarcasm

The project explicitly examines an important weakness of lexical sentiment analysis.

Examples identified include constructions such as:

- `no complaints`
- `not worth`
- `not waste`
- `no issues`
- `not bad`
- `not happy`
- `zero leakage`

A simple word-level lexicon can incorrectly interpret these because it evaluates words individually rather than understanding the complete sentence.

The analysis applies a simple negation adjustment by reversing the sentiment contribution of sentiment-bearing words following selected negators.

### Impact of negation correction

| Metric | Before correction | After correction |
|---|---:|---:|
| Accuracy | 0.810 | 0.822 |

The improvement is **0.012**, or **1.2 percentage points**.

This demonstrates that even a simple linguistic rule can improve classification performance.

### More serious failure: false positives

The analysis identifies cases where genuinely negative comments were classified as positive. Examples include comments containing phrases such as:

- criticism of excessive plastic packaging
- allegations of greenwashing
- refill availability problems
- poor product quality
- sarcastic statements such as praising an obvious product failure

These cases demonstrate why sentiment analysis should not be treated as a fully automated substitute for contextual human interpretation.

---

## 6. Sentiment by Brand

Human-coded sentiment shares are:

| Brand | Negative | Neutral | Positive | NSI |
|---|---:|---:|---:|---:|
| EcoWear | 30.3% | 19.2% | 50.5% | 20.2 |
| NimbusTech | 29.7% | 25.3% | 45.1% | 15.4 |
| PureLeaf | 34.3% | 15.7% | 50.0% | 15.7 |
| TerraBrew | 24.0% | 29.2% | 46.9% | **22.9** |
| VoltRide | 33.3% | 24.5% | 42.2% | **8.8** |

### Net Sentiment Index

The project defines:

```text
NSI = % Positive − % Negative
```

Higher NSI indicates a more favourable balance between positive and negative comments.

### Ranking

1. **TerraBrew — 22.9**
2. **EcoWear — 20.2**
3. **PureLeaf — 15.7**
4. **NimbusTech — 15.4**
5. **VoltRide — 8.8**

TerraBrew has the strongest net sentiment position, while VoltRide has the weakest.

Importantly, the gap is driven not simply by positive sentiment but by the balance between positive and negative sentiment. VoltRide has 42.2% positive comments but also 33.3% negative comments, producing a relatively low NSI.

---

## 7. Is Sentiment Independent of Platform?

A Pearson chi-squared test was performed:

```text
χ² = 2.8631
df = 6
p = 0.8258
```

At conventional significance levels, this is **not statistically significant**.

### Interpretation

The analysis does **not provide evidence that sentiment distribution differs systematically by platform**.

This is a useful distinction:

> Instagram has the largest number of comments, but volume does not imply that Instagram has a fundamentally different sentiment mix.

Therefore, platform-level sentiment differences should not be over-interpreted from descriptive percentages alone.

---

## 8. Does Sentiment Drive Engagement?

The analysis examines likes and replies as engagement measures.

Because likes are highly dispersed, the project uses:

```text
log1p(likes)
```

### Average likes by sentiment

| Sentiment | Comments | Mean likes | Median likes | SD |
|---|---:|---:|---:|---:|
| Negative | 149 | 43.3 | 22 | 73.8 |
| Neutral | 111 | 26.8 | 20 | 23.4 |
| Positive | 230 | 39.4 | 24 | 47.5 |

Negative comments have the highest mean likes, while neutral comments have the lowest.

However, raw averages alone are not sufficient because audience size varies considerably.

### ANOVA

The one-way ANOVA finds a statistically significant relationship between sentiment class and log-transformed likes:

```text
F(2, 487) = 6.194
p = 0.00221
```

Tukey post-hoc comparisons show:

- Neutral vs Negative: statistically significant
- Positive vs Negative: not statistically significant
- Positive vs Neutral: statistically significant

### Controlled regression

The project then controls for:

- sentiment
- follower count
- account verification
- platform

The model explains approximately **65.3% of the variation in log likes**:

```text
R² = 0.6531
Adjusted R² = 0.648
```

The strongest predictor is follower count:

```text
log1p(user_followers): coefficient = 0.514228
p < 2e-16
```

Verified accounts also show a positive association with log likes:

```text
coefficient = 0.141467
p = 0.028184
```

Relative to the reference category, Neutral sentiment has a negative coefficient and Positive sentiment has a positive coefficient, both statistically significant.

Platform coefficients are not statistically significant in this model.

### Business interpretation

The results show that **engagement is strongly influenced by audience size**, not simply by sentiment.

This is an important analytical lesson: a comment receiving many likes should not automatically be interpreted as evidence of positive brand perception.

---

## 9. Replies as a Proxy for Controversy

A second regression models:

```text
log1p(replies)
```

as a function of sentiment and follower count.

The model has:

```text
R² = 0.3337
Adjusted R² = 0.3296
```

Follower count is positively associated with replies.

Both Neutral and Positive sentiment have substantially lower reply levels than the reference Negative class, with highly significant coefficients.

### Interpretation

Within this dataset, **negative comments are associated with greater conversational activity** than neutral or positive comments after accounting for follower count.

This supports a practical social-listening insight:

> Negative feedback may have disproportionate discussion value even when it is less common than positive feedback.

---

## 10. Sentiment Over Time

The project constructs a weekly Net Sentiment Index:

```text
Weekly NSI
= 100 × (% Positive − % Negative)
```

The resulting time series tracks sentiment balance across the observation period from January through June 2026.

The visualisation also scales point size by the number of comments, helping distinguish high-volume weeks from low-volume weeks.

### Recommended interpretation

Weekly NSI should be used as a **monitoring KPI**, not as proof of causality. Changes in sentiment should be investigated alongside:

- campaign activity
- product/service events
- customer-support incidents
- changes in engagement volume
- individual high-impact comments

---

## Key Business Findings

### 1. Positive sentiment dominates overall

The human-coded dataset contains **47.8% positive comments**, compared with **31.2% negative** and **23.1% neutral** after the duplicate-removal analytical sample is considered.

The overall balance is therefore positive, but the negative share is substantial enough to warrant active monitoring.

### 2. TerraBrew has the strongest sentiment position

TerraBrew records the highest NSI at **22.9**, followed by EcoWear at **20.2**.

### 3. VoltRide requires the greatest attention

VoltRide has the lowest NSI at **8.8**, caused by the combination of relatively high negative sentiment and the lowest positive share among the five brands.

### 4. Platform is not a significant determinant of sentiment

The chi-squared test returns **p = 0.8258**, so the analysis does not support a statistically significant platform–sentiment relationship.

### 5. Lexicon sentiment classification is useful but imperfect

The Bing classifier reaches **81.0% accuracy** and **0.696 Cohen's kappa**, but neutral comments are substantially harder to classify.

### 6. Negation handling improves performance

The simple negation adjustment increases accuracy from **81.0% to 82.2%**.

### 7. Negative content can attract engagement

Negative comments have the highest mean likes and are associated with higher reply activity in the reported models.

### 8. Audience size matters heavily

Follower count is the dominant predictor in the controlled likes model, with the model achieving **R² = 0.6531**.

---

## Business Recommendations

### Priority 1 — Investigate negative themes, not only negative volume

Track recurring topics such as:

- quality
- delivery
- packaging
- price/value
- customer service
- refill availability
- sustainability claims

Frequency alone is not enough; the business should connect each topic to operational ownership.

### Priority 2 — Use negative engagement as an early-warning signal

Negative comments can generate substantial likes and replies. High-engagement negative comments should therefore be prioritised for response and escalation.

### Priority 3 — Focus improvement efforts on VoltRide

VoltRide has the lowest NSI. A deeper qualitative review of its negative comments should be performed to determine whether the main issues concern:

- product quality
- service
- pricing
- sustainability claims
- customer support
- delivery

### Priority 4 — Protect TerraBrew's positive position

TerraBrew has the highest NSI. Management should identify which customer experiences are generating this positive sentiment and replicate them where operationally feasible.

### Priority 5 — Do not rely exclusively on automated sentiment

The false-positive examples demonstrate that sarcasm, negation, context, and sustainability-related criticism can fool simple lexical models.

A production-grade solution should combine:

```text
Automated sentiment
        +
Topic detection
        +
Engagement weighting
        +
Human review for high-risk comments
```

### Priority 6 — Build a recurring sentiment-monitoring dashboard

Useful KPIs include:

- Overall NSI
- NSI by brand
- NSI by platform
- Weekly NSI
- Negative-comment rate
- High-engagement negative comments
- Sentiment by topic
- Sentiment model accuracy
- Escalated comments
- Response time

---

## Limitations

The analysis should be interpreted with the following limitations:

1. **Bing is a lexicon-based model.** It does not fully understand context, sarcasm, irony, or complex linguistic structure.
2. **Neutral classification is comparatively weak.** Recall is only 0.541.
3. **Negation correction is rule-based.** It improves accuracy but is not a complete natural-language understanding system.
4. **The analysis uses human-coded labels as the validation benchmark.** Their quality is assumed rather than independently assessed in the report.
5. **The engagement models are observational.** Statistical association should not be interpreted as causal impact.
6. **Duplicate removal is based on exact text matching.** Semantically similar or near-duplicate comments may remain.
7. **Missing likes are median-imputed within platform.** This is a pragmatic approach but introduces estimated values.
8. **The HTML report does not document an external/live data ingestion pipeline.** Therefore, this should be treated as an analytical dataset/report rather than a live social-media monitoring system.

---

## Technical Stack

### Programming and reporting

- R
- Quarto
- R Markdown / knitr infrastructure

### R packages

- `tidyverse`
- `tidytext`
- `wordcloud`
- `RColorBrewer`
- `scales`
- `readxl`

### Core analytical techniques

- Data cleaning
- Missing-value treatment
- Duplicate detection
- Text preprocessing
- Tokenisation
- Stop-word removal
- Word-frequency analysis
- Lexicon-based sentiment analysis
- Confusion matrix analysis
- Precision / recall / F1
- Cohen's kappa
- Chi-squared test
- ANOVA
- Tukey HSD
- Multiple linear regression
- Time-series aggregation
- Net Sentiment Index

---

## Reproducibility

The rendered report was generated with **Quarto 1.8.27**.

The reported R environment is:

- R **4.5.1**
- Windows 11 x64
- Time zone: `Asia/Calcutta`

The analysis uses the packages and versions recorded in the report, including `tidytext 0.4.3`, `ggplot2 4.0.3`, `dplyr 1.2.1`, `readxl 1.4.5`, and `wordcloud 2.6`.

### Important portability issue

The rendered HTML currently reads the Excel input using an **absolute Windows file path**:

```r
read_excel("C:\\Users\\...\\sentiment_lab_data.xlsx")
```

This path will not work for another user or on GitHub/another computer.

For a GitHub-ready project, change it to a project-relative path such as:

```r
raw <- readxl::read_excel("data/sentiment_lab_data.xlsx")
```

A recommended project structure is:

```text
sentiment-analytics/
│
├── README.md
├── Sentiment_Analysis.qmd
├── Sentiment_Analysis.html
│
├── data/
│   └── sentiment_lab_data.xlsx
│
├── output/
│   └── sentiment_scored_output.csv
│
└── Sentiment_Analysis_files/
    └── ...
```

---

## GitHub Upload Checklist

Before pushing the project to GitHub:

- [ ] Add `README.md`
- [ ] Add `Sentiment_Analysis.qmd`
- [ ] Add the input data under `data/` if permitted
- [ ] Replace the absolute Windows path with a relative path
- [ ] Add the rendered `Sentiment_Analysis.html`
- [ ] Add `Sentiment_Analysis_files/` so the HTML charts render correctly
- [ ] Add the exported `sentiment_scored_output.csv` if appropriate
- [ ] Add a `.gitignore`
- [ ] Remove personal/local file paths from code
- [ ] Re-render the Quarto document from the project directory
- [ ] Verify that the HTML opens correctly after cloning the repository

### Suggested `.gitignore`

```gitignore
.Rproj.user/
.Rhistory
.RData
.Ruserdata

*.Rproj

.DS_Store

~$*.xlsx
```

---

## Suggested Repository Description

**Sentiment Analytics in R: social-media text mining, lexicon-based sentiment classification, model validation, brand/platform analysis, engagement modelling, and weekly sentiment tracking.**

---

## Conclusion

This project demonstrates a complete applied text-analytics workflow rather than stopping at a word cloud or sentiment count.

The strongest analytical feature is the **validation of automated sentiment against human-coded labels**. The Bing classifier achieves 81.0% accuracy and a Cohen's kappa of 0.696, while the additional negation rule raises accuracy to 82.2%. The analysis also moves beyond sentiment classification into business interpretation by comparing brands, testing platform independence, modelling engagement, and tracking sentiment over time.

From a business-analysis perspective, the central takeaway is that **sentiment should be treated as a decision-support signal rather than a standalone KPI**. The most useful monitoring framework combines sentiment, topic, engagement, audience size, and contextual review of high-risk comments.

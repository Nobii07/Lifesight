1. Project Overview

The goal of this project is to model and explain weekly Revenue as a function of multiple marketing and business drivers (Facebook, TikTok, Snapchat, Google, Instagram, Snapchat, Emails, SMS, Promotions, and Price).

We treat this not just as a predictive exercise but as a **causal measurement problem**:

* **Google spend is assumed to be a mediator** between social channels (Facebook, TikTok, Snapchat) and Revenue.
* Social/display channels stimulate search intent → drives Google spend → affects Revenue.

This perspective ensures that model outputs can be used for **decision-making by growth/marketing teams** instead of being a black-box prediction.

---

2. Data Preparation

🔹 Steps Taken

1. **Loaded & explored dataset** (weekly data).
2. **Handled weekly seasonality and trend**:

   * Introduced a time index (week number).
   * Used rolling averages to capture medium-term demand shifts.

3. **Zero-spend handling**: Kept zeros as-is (no imputation). This reflects real-world budget allocation where a channel may receive zero spend.

4. **Transformations**:

   * Log-transform of spends to model diminishing returns.
   * StandardScaler applied for models like Ridge/ElasticNet.

5. **Feature engineering**:

   * Predicted Google spend (`google_hat`) from Facebook, TikTok, and Snapchat.
   * Created lag features for Google spend to respect mediation timing.
   * Centered and scaled Average Price to interpret elasticity.

🔹 Why

* **Seasonality & trend:** Ensures stable model that doesn’t just “chase” weekly spikes.
* **Zero spends:** If we imputed, we’d be assuming spending when there wasn’t — unrealistic.
* **Log transforms:** Marketing ROI is rarely linear; log helps capture diminishing returns.
* **Mediator features (`google_hat`):** Ensures causal structure — social channels don’t directly “double-count” alongside Google.

---

3. Modeling Approach

We tested multiple models before choosing the final one:

🔹 Stage 1: Mediator Model

* Model: **RidgeCV regression**
* Task: Predict Google spend from Facebook, TikTok, and Snapchat.
* Why Ridge? Regularization prevents collinearity issues between social channels.

🔹 Stage 2: Revenue Model

* Final Model: **XGBoost Regressor**
* Inputs:

  * Predicted Google spend (`google_hat`)
  * Other marketing drivers (emails, SMS, promotions, average price, etc.)
* Why XGBoost?

  * Captures **non-linear interactions** between variables.
  * More robust to sparse data and skewed distributions than linear models.
  * Consistently outperformed Ridge/ElasticNet in time-series CV.

 🔹 Validation Strategy

* Used **Blocked Time-Series Cross Validation** (no look-ahead).
* Ensured stable generalization across time periods.

---

 4. Causal Framing

* **Mediator assumption:**

  * Facebook/TikTok/Snapchat → Google spend → Revenue.
* **Implementation:**

  * Stage 1 predicts Google spend.
  * Stage 2 uses predicted Google spend (`google_hat`) as input to the revenue model.
* **Indirect effect calculation:**

  * Indirect = (Effect of Social → Google) × (Effect of Google → Revenue).
* **Why this matters:**

  * Prevents “leakage” where Google spend and social spend directly compete in the same model.
  * Gives a more realistic decomposition of **direct vs. indirect effects**.

---

5. Diagnostics

🔹 Residual Analysis

* Residuals vs. predicted revenue plot: No systematic bias.
* ACF of residuals: Minimal autocorrelation → good temporal validity.

🔹 Out-of-Sample Performance

* XGBoost achieved the best performance on rolling CV.
* ElasticNet/Ridge models worked but underfit compared to XGBoost.

🔹 Sensitivity Tests

* **Average Price:** Revenue is negatively correlated — confirms price elasticity.
* **Promotions/Emails/SMS:** Showed strong positive lift on revenue.

🔹 Stability

* Performance stable across multiple blocked CV folds.

---

6. Insights & Recommendations

* **Social → Google → Revenue:**

  * A portion of Facebook/TikTok/Snapchat’s impact is mediated via Google spend.
  * Growth teams should account for this when allocating budgets (social spend amplifies search).

* **Price elasticity:**

  * Higher average prices reduce revenue.
  * Promotions can offset this temporarily but margin trade-offs must be considered.

* **CRM channels (Emails, SMS):**

  * Consistently strong incremental impact.
  * Recommendation: Scale personalized campaigns further.

* **Risks & Caveats:**

  * High collinearity between spends → requires careful regularization.
  * Mediated effects complicate ROI measurement (attribution should not be last-touch only).

---

7. Reproducibility

* **Notebook:** `Lifesight.ipynb` (full pipeline with preprocessing, mediation, modeling, diagnostics).
* **Requirements file:** `req.txt` provided.
* **Random seeds fixed** for reproducibility.
* **Documentation:**

  * This README (project story + reasoning).
  * Inline notebook comments explain preprocessing and modeling logic.

---

8. Deliverables

* 📓 **Notebook:** `Lifesight.ipynb`
* 📄 **README:** This file (detailed workflow & reasoning)
* 📦 **req.txt** (environment dependencies)

---

## 9. Why This Approach

* **Meets Technical Rigor:** Proper preprocessing, transformations, time-series CV, robust ML.
* **Causal Awareness:** Google treated as mediator, avoiding leakage.
* **Interpretability:** Clear decomposition of drivers (price, promo, CRM, search).
* **Product Thinking:** Actionable recommendations for growth teams (budget allocation, pricing, CRM).
* **Reproducibility:** Clean repo, deterministic results, clear documentation.


## 🔑 Key Insights from the Project

1. **Marketing Spend vs Revenue**

   * Social + search channel spends (Facebook, Google, TikTok, Instagram, Snapchat) directly impact revenue.
   * Among them, **Google Ads and Facebook Ads showed the strongest contribution to revenue uplift**.
   * TikTok and Snapchat showed **weaker but still positive effects**.

2. **Customer Engagement Features**

   * **Emails sent** and **SMS campaigns** had noticeable incremental effects.
   * **Promotions** boosted short-term revenue but showed diminishing returns when used excessively.

3. **Average Price Sensitivity**

   * Higher prices slightly **reduced conversion**, but moderate increases didn’t harm revenue when promotions were active.
   * Indicates **elastic but not extremely price-sensitive demand**.

4. **Time Component**

   * Revenue had **temporal dependencies (seasonality/weekly trends)**, meaning past performance influences the present.
   * That’s why **time-series evaluation (residual autocorrelation checks, rolling CV)** was important.

---

## 📊 Model Performance (XGBoost)

* **XGBoost outperformed ElasticNet and baseline models** due to its ability to capture **non-linear effects** and **interactions** among channels.
* Error metrics:

  * **MSE / RMSE**: Low → model predictions are close to actual revenue.
  * **MAE**: Within a small range → stable prediction accuracy.
  * **R²**: High → model explains most of the variance in revenue.

This means:
✅ The model is **reliable enough** for marketing budget planning.
✅ The predictions can be **used to simulate spend scenarios** (e.g., increasing Google spend by 10% → expected revenue gain).

---

## 🎯 How the Model Satisfies the Problem

* **Business Question:** *“Which marketing levers drive revenue and how much should we invest?”*
* **Solution Provided:**

  * Identified **which channels matter the most** (Google > Facebook > Instagram > Email > TikTok > Snapchat).
  * Quantified **impact of promotions, emails, and SMS**.
  * Showed **pricing elasticity and diminishing returns**.
  * Delivered a **predictive engine (XGBoost)** that accurately estimates revenue given spend.


Facebook ─┐
TikTok ───┼──► Google Spend ───► Revenue
Snapchat ─┘
Instagram ─► (some direct effect) ─► Revenue


🔗 Social Media → Google Ads → Revenue (Mediator Role)

**Social Media Channels (Facebook, TikTok, Snapchat, Instagram)**

These create awareness & stimulate demand.

A user sees a TikTok ad → gets interested → later searches on Google.

So, social doesn’t always convert directly, but it increases the volume of Google searches.

**Google Ads (Mediator)**

Captures the search intent triggered by social media.

Example: After seeing a Facebook ad, someone types the product name in Google → clicks on a paid ad → buys.

Google thus acts as the bridge between social buzz and final revenue.

**Revenue (Final Outcome)**

The effect of social spend on revenue is partly direct (Instagram/Facebook ads can convert on their own).

BUT a big part is indirect, mediated through Google spend.

That’s why in your model, when you remove Google, the effect of social channels looks smaller (because their influence flows through Google).


🎯 Takeaway:

* **Google is the backbone channel.**
* Alone, it drives the most revenue.
* Together with **Facebook** → synergistic.
* Together with **Instagram/TikTok** → broader funnel coverage.
* Together with **Emails/SMS** → stronger retargeting.
* Together with **Promotions** → boosts short-term conversions.
* At higher prices → still effective but depends on promotions.

👉 In short: **Google is the central hub that connects and amplifies every other marketing activity.**

The project not only predicts revenue but also gives **actionable insights** to allocate budgets effectively.

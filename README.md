# GA4 E-Commerce Analytics for a Luxury Fashion Retailer: Decoding the User Journey and Basket Composition

## Overview
A luxury fashion e-commerce retailer tracks on-site behavior through Google Analytics 4 (GA4), but converting raw event
data into an understanding of *who is likely to buy* and *why they abandon* requires dedicated modeling. 

This project analyzes ~2 months of GA4 session and e-commerce event data to answer three business questions: what drives the time between a user's first product view and eventual purchase, which session-level behaviors predict cart abandonment vs. checkout, and
which products/categories are most often bought together. 

The goal is to turn these findings into concrete actions for retargeting, on-site UX, and cross-sell design.

*[This project was completed as part of a team assignment. My primary contribution was Approach 1, which I led end-to-end. I also contributed to reviewing and refining Approach 2 and the bonus Market Basket Analysis.]*

## Approach/Methods
The analysis is split into three notebooks, each tackling a different angle on the same
underlying GA4 event data:

**1. Time to Purchase & Cross-Session Product Exploration**
* Feature engineering to reshape raw event logs into a longitudinal, user-level structure (session sequencing, UTM/channel classification, item-similarity scores via OpenAI `text-embedding-3-small` embeddings for browsing focus).
* Survival analysis - Cox Proportional Hazards (baseline + enriched with the item-similarity feature), XGBoost AFT, and Random Survival Forest (via `scikit-survival`) - to model time-to-purchase and identify drivers of conversion speed, evaluated via concordance index (C-index).
* Binary session-level purchase classification (Logistic Regression, Random Forest, XGBoost, HistGBM, soft-vote ensemble), with group-aware train/test splitting and permutation importance / SHAP for interpretation.
* Delayed co-purchase analysis: tracking whether items viewed together in one session are purchased together in a later session, at both the item and category level, including an LLM-based (GPT-4o-mini) classification of item-pair relationships (complementary / substitute / style-match).

**2. Cart Abandonment vs. Purchase Prediction**
* Two-stage funnel model: Stage 1 predicts P(add-to-cart | session) on the full dataset; Stage 2 predicts P(purchase | cart) restricted to cart sessions, with the two combined into a single full-funnel purchase score.
* Logistic Regression (baseline, expanded, engineered, and Lasso specifications) for interpretability, plus Random Forest and LightGBM for predictive performance, compared on ROC-AUC and PR-AUC under strong class imbalance.
* Threshold tuning (F1-optimal) to define an actionable intervention point for real-time abandonment risk scoring.

**3. Market Basket Analysis (bonus)**
* Association rule mining (Apriori-style, with a custom pairwise fallback) at both the product and product-category level.
* Summary tables reporting, for every focal product/category: the associated item, conditional joint-purchase probability, support, and lift.

## Key Results
* **Conversion window is narrow:** most purchases happen within the first few days of first item view; users who don't convert quickly rarely convert within the same consideration cycle. Browsing focus (how concentrated a user's viewed items are) is the strongest behavioral driver of both conversion speed and in-session purchase, outweighing marketing/channel variables and user history.
* **Cart abandonment is dominated by two signals:** time from cart to checkout and login status. Users who move quickly from cart to checkout, and users who are logged in, are substantially more likely to complete the purchase: logged-in users add to cart and convert at markedly higher rates than guests. A tuned gradient-boosted model on these features achieves strong discrimination between purchasers and abandoners, with a tuned probability threshold used to flag at-risk sessions for intervention.
* **Cross-session intent carries over more at the category level than the item level:** users rarely return to buy the exact same item viewed earlier, but they do return to buy within the same product category, with complementary pairs (e.g. blazers + skirts, bathrobes + slippers) far more likely to convert jointly than substitutes.
* **Basket data is highly sparse**, with the large majority of baskets containing a single item: making category-level association rules more actionable than product-level ones for cross-sell and merchandising design.
* Full business recommendations (retargeting timing, checkout friction reduction, login prompts, "complete the look" bundling) are detailed in each notebook's closing section.

## Tools Used
Python (pandas, numpy, scikit-learn, statsmodels, lifelines, scikit-survival, XGBoost,
LightGBM, SHAP, mlxtend), OpenAI API (`text-embedding-3-small` for item-similarity
embeddings, GPT-4o-mini for item-pair relationship classification)

## Repository Contents
* `01_time_to_purchase.ipynb`: EDA, feature engineering, survival analysis, binary session classification, delayed co-purchase analysis, findings & recommendations
* `02_cart_abandonment.ipynb`: EDA two-stage funnel modeling (add-to-cart → purchase), model interpretation, threshold selection, business recommendations
* `03_market_basket_analysis.ipynb`: EDA, association rule mining and product/category recommendation tables
* `data_dictionary.md`: field reference for the three source datasets
  
  This analysis uses real GA4 data provided for academic use only, so the raw dataset is not included in this repository. Notebooks are shared to demonstrate methodology.

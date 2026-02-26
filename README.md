# Women's Hockey Expected Goals (xG) Model

**Brian Johns · February 2026**

---

## Overview

**Can we build a reliable Expected Goals model for Women's professional hockey — and what does it tell us about shot quality, player performance, and the state of the game?**

This project scrapes and combines shot data from the PWHL and PWHPA to build an xG model from scratch. Rather than relying on the relatively rare outcome of a goal, an xG model assigns each shot a probability of scoring based on factors like location, shot type, and game situation — providing a more stable measure of offensive and defensive performance.

The Women's game presents a unique analytical challenge: the PWHL is only in its third season, public analytics tooling is virtually non-existent, and the dataset (~16,700 shots) is a fraction of what NHL xG models typically train on.

---

## Data & Tools

- **Sources:** PWHL HockeyTech API (scraped across all available seasons) · PWHPA SportLogiq play-by-play data
- **Tools:** Python, Pandas, NumPy, Scikit-Learn, XGBoost, Matplotlib, Seaborn, Shapely
- **Models:** Logistic Regression, XGBoost Classifier

---

## Notebooks

| # | Notebook | Description |
|---|----------|-------------|
| 1 | Data Acquisition & Cleaning | API scraping, coordinate rescaling, PWHPA merge, filtering |
| 2 | Feature Engineering & EDA | Shot quality features, Bayesian player quality priors, exploratory analysis |
| 3 | Modelling Part 1 | Logistic Regression and XGBoost across all strength states |
| 4 | Even Strength Modelling | Refined modelling on cleaned ES-only data, final xG model |
| 5 | Project Summary | Process overview, key findings, and future directions |

---

## Key Findings

### What predicts a goal?

Across all models, shot location was consistently the strongest predictor:

- **Distance and angle** from the net were the most important features — closer, more central shots score at a significantly higher rate
- **Rebounds** showed a meaningfully elevated goal rate, reflecting the chaos of second-chance opportunities
- **Shooter quality** (Bayesian career goal rate) added predictive value beyond pure location
- **Shot type** was a known strong predictor in NHL models but was largely unusable here — the majority of PWHL shots were labeled `Default` rather than a meaningful type (wrist, slap, backhand, etc.)

### Best performing model: XGBoost (Even Strength)

| Metric | Score |
|--------|-------|
| AUC | 0.742 |
| Log Loss | 0.270 |
| Data | ~10,000 Even Strength shots |

A higher AUC of 0.775 was achieved on the full dataset, but coefficient analysis revealed multicollinearity between location features and distortion from mixing strength states. The Even Strength model was chosen for deployment because it is built on cleaner data and produces more trustworthy probabilities for use as xG values.

### Data quality challenges

Two significant data issues shaped the entire project:

**Coordinate tracking inconsistencies** — especially in early seasons, shots were recorded with implausible locations. Goals from beyond the blue line and behind the goal line were filtered, and coordinates required rescaling from the API's 600x300 grid to real-world rink dimensions.

**Shot type recording** — the majority of PWHL shots lack a meaningful shot type label. This is one of the strongest known predictors in NHL xG models and is largely unavailable here, representing a ceiling on model performance that only better league data collection can address.

### Class imbalance

With only ~8.4% of shots resulting in goals, every model needed to navigate a severe class imbalance. Techniques including `class_weight='balanced'`, SMOTE oversampling, and lowered decision thresholds were all explored. Balancing techniques consistently hurt Log Loss without improving AUC — the models were right to be conservative about predicting goals, reflecting the genuine difficulty of scoring in hockey.

---

## Future Directions

- **More seasons** — the single biggest improvement will come from additional PWHL data as the league matures
- **Separate strength state models** — Even Strength, Power Play, and Shorthanded shots have distinct spatial distributions and goal rates that warrant individual models
- **Fix shooter/goalie quality leakage** — career quality features currently use a player's final value applied retroactively; a proper implementation would use only historically available data at the time of each shot
- **Apply xG to team and player evaluation** — xG generated vs. actual goals scored, Goals Saved Above Expected for goalies, and team-level shot quality comparisons are all now possible with the model output
- **Add Player Tracking data** - Player tracking data would greatly enhance these models, especially given the limited dataset
- **Add Shots On Net data** - This data was limited to shots on goal.  Using shots towards the net would also help with the limited dataset

---

# Dive into League of Legends

**By Ethan Lam and Sreekar Vemula**

---

## Introduction

In competitive League of Legends, games are often decided long before they end. Professional players and analysts widely believe that the first 10 minutes — the "early game" — set the trajectory for the entire match. But how much does an early lead actually matter? **Can we predict whether a player's team wins a match using only stats available at 10 minutes into the game?**

This question matters because understanding the predictive power of early-game stats has real implications for how teams draft, coach, and strategize. If a gold lead at 10 minutes is a near-certain predictor of victory, teams should prioritize early-game compositions. If not, late-game scaling may be undervalued.

We use the **2022 League of Legends Esports Match Data** from [Oracle's Elixir](https://oracleselixir.com/), which tracks professional match statistics across major global leagues. The full dataset has **148,980 rows and 165 columns**, where each game corresponds to 12 rows (10 player rows + 2 team-summary rows). After filtering to player-level rows only, we work with **124,150 rows** representing individual player performances across **12,415 games**.

### Relevant Columns

| Column | Description |
|--------|-------------|
| `result` | Whether the player's team won the match (`True` = win, `False` = loss) |
| `position` | The player's role: `top`, `jng` (jungle), `mid`, `bot` (ADC), or `sup` (support) |
| `side` | Which side of the map the team started on: `Blue` or `Red` |
| `playoffs` | Whether the match was played in a playoff setting (`True`/`False`) |
| `gamelength` | Total game duration in seconds |
| `kills` | Total kills by the player during the match |
| `deaths` | Total deaths by the player during the match |
| `assists` | Total assists by the player during the match |
| `dpm` | Damage per minute dealt to enemy champions |
| `goldat10` | Gold earned by the player at exactly 10 minutes |
| `xpat10` | Experience points accumulated by the player at 10 minutes |
| `csat10` | Creep score (minion kills) at 10 minutes |
| `killsat10` | Kills secured at 10 minutes |
| `assistsat10` | Assists at 10 minutes |
| `deathsat10` | Deaths at 10 minutes |
| `golddiffat10` | Gold difference vs. the opposing player in the same role at 10 minutes |
| `xpdiffat10` | XP difference vs. the opposing player in the same role at 10 minutes |
| `csdiffat10` | CS difference vs. the opposing player in the same role at 10 minutes |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The original dataset has **148,980 rows and 165 columns**. Each game generates 12 rows: 10 for individual players and 2 for team summaries (`participantid` 100 or 200). Since our analysis focuses on player-level performance, we filter out the team rows, resulting in **124,150 rows**.

Several columns stored as `0`/`1` integers were converted to booleans for clarity: `playoffs`, `firstPick`, `result`, `firstblood`, `firstbloodkill`, `firstbloodassist`, and `firstbloodvictim`. This makes filtering and grouping more intuitive and prevents accidental numerical operations on binary flags.

### Univariate Analysis

The distribution of game lengths shows that most professional matches last between 1,500–2,400 seconds (~25–40 minutes), with a right-skewed tail of unusually long games.

<iframe
  src="assets/fig_gamelength.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Kill counts per player are heavily right-skewed — most players record 0–5 kills per game, consistent with the controlled, objective-focused nature of professional play.

<iframe
  src="assets/fig_kills.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Teams that secure First Blood win at a noticeably higher rate than those that don't, suggesting early combat advantages translate into game outcomes.

<iframe
  src="assets/fig_winrate_firstblood.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

Average combat stats grouped by position reveal a clear role hierarchy. Bot and mid laners lead in damage per minute (DPM), consistent with their damage-focused role. Supports have the highest assists and lowest kills, while junglers rack up the most assists of any carry role.

| position | kills | deaths | assists | dpm | win_rate |
|----------|-------|--------|---------|-----|----------|
| bot | 4.25 | 2.54 | 5.36 | 560.73 | 0.5 |
| mid | 3.49 | 2.65 | 5.89 | 545.82 | 0.5 |
| top | 2.79 | 2.95 | 5.02 | 488.51 | 0.5 |
| jng | 3.04 | 3.11 | 6.85 | 324.15 | 0.5 |
| sup | 0.87 | 3.24 | 9.17 | 171.86 | 0.5 |

All positions have a 50% win rate by construction (every game has one winner and one loser), confirming the dataset is balanced.

---

## Assessment of Missingness

### MNAR Analysis

We believe the **`ban1` through `ban5`** columns (champion bans) are **MNAR** (Missing Not At Random). In the dataset, ban data is absent for certain matches — and crucially, this missingness cannot be fully explained by any other observed column. The bans are missing not because of the league, patch, or game outcome, but because those particular matches were played in non-standard formats or recorded under incomplete data collection pipelines. The absence of ban data is tied to unobserved characteristics of the match's recording context (e.g., whether a tournament organizer used the full OraclesElixir data feed), which is not captured elsewhere in the dataset. To confirm this, we would need metadata about each match's data-collection method — information not present in any observed column.

### Missingness Dependency: `goldat10`

The `goldat10` column (gold earned at the 10-minute mark) is missing in **15.2% of player rows**, because certain leagues don't record 10-minute snapshots. We tested whether this missingness depends on other columns using permutation tests with test statistic |difference in missingness rates| at **α = 0.05**.

**Depends on `playoffs`** (p ≈ 0.0000): Playoff games have significantly different missingness rates for `goldat10` than regular season games. This makes sense — leagues that track detailed time-series stats are more likely to do so consistently in high-stakes playoff matches.

<iframe
  src="assets/fig_missingness_playoffs.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Does not depend on `side`** (p = 1.0000): Which side of the map a team plays on has no relationship to whether `goldat10` is missing, as expected — map side is unrelated to data collection practices.

<iframe
  src="assets/fig_missingness_side.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Hypothesis Testing

Bot laners (ADCs) and mid laners are both considered primary damage dealers, but the community often debates which role truly does more damage. We test this directly.

**Null Hypothesis (H₀):** The distribution of damage per minute (DPM) is the same for ADC (bot) and mid laners. Any observed difference is due to random chance.

**Alternative Hypothesis (H₁):** ADC players deal more damage per minute than mid laners on average.

**Test Statistic:** Difference in group means: mean DPM(bot) − mean DPM(mid). Positive values favor H₁.

**Significance Level:** α = 0.05

We ran a permutation test with 10,000 shuffles. The observed difference was **+14.90 DPM**, and the resulting **p-value was ≈ 0.0000** — far below our threshold.

**Conclusion:** We reject H₀. The data provides strong evidence that ADC players deal more damage per minute than mid laners on average in professional 2022 play. This is consistent with ADCs building sustained damage items and fighting from a safe distance throughout fights, while mid laners often play more utility-oriented or roam-heavy styles. Note that this result does not prove ADCs are "better" — it reflects a difference in role function, not overall impact.

<iframe
  src="assets/fig_hypothesis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

**Prediction Problem:** Can we predict whether a player's team wins a match using only stats available at 10 minutes into the game?

**Type:** Binary Classification
**Response Variable:** `result` — `True` (win) or `False` (loss)

We chose `result` as the response variable because winning is the ultimate objective of every match, and predicting it from early-game data directly answers our central question.

**Evaluation Metric:** Accuracy

We use accuracy because the dataset is perfectly balanced (every game has exactly one winning and one losing team per side), so there is no class imbalance issue. Accuracy directly measures how often the model correctly calls the outcome, which aligns with our question. Precision or recall would matter more in imbalanced settings, but here they reduce to accuracy anyway.

**Time of Prediction Constraint:** We only use features available at exactly the 10-minute mark — `goldat10`, `xpat10`, `killsat10`, `assistsat10`, `deathsat10`, `csat10`, and `position`. Post-game stats like total kills, DPM, or final gold are excluded to prevent data leakage.

---

## Baseline Model

### Model & Features

The baseline model is a **Logistic Regression** trained on six quantitative features and one nominal feature, all implemented in a single `sklearn` Pipeline:

| Feature | Type | Encoding |
|---------|------|----------|
| `goldat10` | Quantitative | None (used as-is) |
| `xpat10` | Quantitative | None |
| `killsat10` | Quantitative | None |
| `assistsat10` | Quantitative | None |
| `deathsat10` | Quantitative | None |
| `csat10` | Quantitative | None |
| `position` | Nominal | One-hot encoding (drop first) |

`position` is one-hot encoded because it is a categorical variable with no natural ordering. The numeric features are left as-is since Logistic Regression can handle them directly.

### Performance

| Split | Accuracy |
|-------|----------|
| Train | 59.78% |
| Test | **58.82%** |
| Naive baseline (always predict win) | 50.00% |

The baseline achieves **58.82% test accuracy**, a ~9 percentage point improvement over always guessing "win." This suggests the 10-minute stats do carry meaningful signal — but there is substantial room for improvement, since the model is only moderately better than chance.

---

## Final Model

### Engineered Features

Three new features were added to capture information that the raw stats miss:

**1. KDA at 10** = `(killsat10 + assistsat10) / (deathsat10 + 1)`
The baseline uses kills, assists, and deaths as separate linear inputs. KDA collapses them into a single combat-efficiency ratio, which better captures what matters: how much a player contributed relative to how much they fed the enemy. Dividing by `deathsat10 + 1` avoids division by zero for players who haven't died yet.

**2. Resource Lead Score** = `z(golddiffat10) + z(csdiffat10)`
Both `golddiffat10` and `csdiffat10` measure lane dominance, but their raw scales differ by ~34×. Summing them directly effectively discards the CS component. We z-score each independently before summing so both dimensions of resource advantage contribute equally. A player winning both gold *and* CS should be penalized more strongly than winning just one.

**3. XP Lead Ratio** = `xpdiffat10 / (xpat10 + 1)`
Raw `xpdiffat10` measures the absolute XP gap, but a +200 XP lead is far more meaningful at low levels than at high levels (when both players have accumulated thousands of XP). Dividing by the player's total XP normalizes the advantage to a fraction of their own experience pool, capturing *relative* level dominance.

We also retained the raw diff columns (`golddiffat10`, `xpdiffat10`, `csdiffat10`) alongside the engineered features.

### Algorithm & Hyperparameter Tuning

We used a **Random Forest Classifier**, which captures non-linear interactions between features that Logistic Regression cannot. We tuned three hyperparameters via `GridSearchCV` with 5-fold cross-validation on the training set:

| Hyperparameter | Values Searched | Best Value |
|----------------|----------------|------------|
| `n_estimators` | 100, 200 | 100 |
| `max_depth` | 5, 10, None | 10 |
| `min_samples_leaf` | 1, 5, 10 | 1 |

`max_depth=10` prevents the trees from overfitting to noise while still capturing complex feature interactions. `min_samples_leaf=1` allows fine-grained splits, which works well given the large training set (~84k rows).

### Performance

| Model | Test Accuracy |
|-------|--------------|
| Baseline (Logistic Regression) | 58.82% |
| **Final (Random Forest)** | **60.99%** |
| Improvement | +2.17 pp |

The final model improves over the baseline by **+2.17 percentage points** on held-out test data (same train/test split used throughout). The CV score of 61.14% is close to the test score, indicating the model generalizes well and is not overfitting.

---

## Fairness Analysis

**Groups:** Blue side players vs. Red side players
**Evaluation Metric:** Accuracy

In competitive League of Legends, blue side picks first in champion select, granting structural agency over team composition. If the model implicitly learned blue-side patterns better (since blue-side early leads may translate more consistently to wins), it could produce systematically higher accuracy for blue-side players.

**Null Hypothesis (H₀):** The model is fair with respect to side. Its accuracy for blue side and red side players are roughly the same, and any observed difference is due to random chance.

**Alternative Hypothesis (H₁):** The model is unfair — its accuracy for blue side players is *higher* than for red side players.

**Test Statistic:** Accuracy(Blue) − Accuracy(Red). Positive values favor H₁.
**Significance Level:** α = 0.05

| Group | Accuracy |
|-------|----------|
| Blue side | 61.18% |
| Red side | 60.80% |
| Observed difference | +0.0038 |
| p-value (one-sided) | 0.2863 |

**Conclusion:** We fail to reject H₀ (p = 0.2863 ≥ 0.05). The observed +0.38 percentage point accuracy gap between blue and red side is consistent with random variation. We have no statistically significant evidence that the model is unfair with respect to map side.

<iframe
  src="assets/fig_fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

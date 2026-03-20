# Dive into League of Legends

**By Ethan Lam and Sreekar Vemula**

---

## Introduction

The dataset used is the 2022 League of Legends Esports Match Data from Oracle's Elixir, which tracks professional match statistics across major leagues globally. This year was specifically chosen because both team members were heavily active in the game at the time. 

In League of Legends, games are often decided long before they end. When a player performs well early on in the game, a 'snowball' effect can often be seen as the gap between players grow over time. Many believe that the "early game" of a player's performance gives enough information to set the trajectory for the entire match. How much does an early lead actually matter? **Can we predict whether a player's team wins a match using only stats available at 10 minutes into the game?**

This question is crucial because understanding the importance of early-game stats has a large impact on how teams draft, strategize, and play. For example, if a gold lead at 10 minutes is a strong predictor of winning the game, teams should prioritize early-game resources. Otherwise, late game scaling should be a priority.



### Relevant Columns

| Column | Description |
|--------|-------------|
| `result` | Whether the player's team won the match (`True` = win, `False` = loss) |
| `position` | Player's role: `top`, `jng` (jungle), `mid`, `bot` (ADC), or `sup` (support) |
| `side` | Which side of the map the team started on: `Blue` or `Red` |
| `playoffs` | Whether the match was played in a playoff setting (`True`/`False`) |
| `gamelength` | Total game duration in seconds |
| `kills` | Player's total kills |
| `deaths` | Player's total deaths |
| `assists` | Player's total assists |
| `dpm` | Damage per minute dealt to enemy champions |
| `goldat10` | Gold earned at 10 minutes |
| `xpat10` | Total experience points at 10 minutes |
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

The original dataset has 148,980 rows and 165 columns. Each game corresponds to 12 rows: 10 player rows and 2 for team summaries (`participantid` = 100 or 200). Since our analysis focuses on player-level performance, we will only look at and keep player rows, resulting in 124,150 rows.

Many columns that consist of only `0` and `1`s are converted to booleans: `playoffs`, `firstPick`, `result`, `firstblood`, `firstbloodkill`, `firstbloodassist`, and `firstbloodvictim`. Having boolean variables helps prevent confusion with numerical calculations and makes the variables easier to use.

### DataFrame Head

There are over 100+ columns in the dataframe. This output of the first 5 rows only shows the columns we look at in our analysis.

| result | position | side | playoffs | gamelength | kills | deaths | assists | dpm | goldat10 | xpat10 | csat10 | killsat10 | assistsat10 | deathsat10 | golddiffat10 | xpdiffat10 | csdiffat10 |
|--------|----------|------|----------|------------|-------|--------|---------|--------|----------|--------|--------|-----------|-------------|------------|--------------|------------|------------|
| False | top | Blue | False | 1713 | 2 | 3 | 2 | 552.29 | 3228.0 | 4909.0 | 89.0 | 0.0 | 0.0 | 0.0 | 52.0 | -44.0 | 8.0 |
| False | jng | Blue | False | 1713 | 2 | 5 | 6 | 412.08 | 3429.0 | 3484.0 | 58.0 | 1.0 | 2.0 | 0.0 | 485.0 | 432.0 | -5.0 |
| False | mid | Blue | False | 1713 | 2 | 2 | 3 | 499.40 | 3283.0 | 4556.0 | 81.0 | 0.0 | 1.0 | 0.0 | 162.0 | 71.0 | 0.0 |
| False | bot | Blue | False | 1713 | 2 | 4 | 2 | 389.00 | 3600.0 | 3103.0 | 78.0 | 1.0 | 1.0 | 0.0 | 296.0 | 265.0 | -12.0 |
| False | sup | Blue | False | 1713 | 1 | 5 | 6 | 128.30 | 2678.0 | 2161.0 | 16.0 | 1.0 | 1.0 | 0.0 | 528.0 | -587.0 | 1.0 |


### Univariate Analysis

The distribution of game lengths shows most matches last between 1,400–2,300 seconds (~25–40 minutes), with a right-skewed tail of some unusually long games.

<iframe
  src="assets/fig_gamelength.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of kills is skewed to the right with most players recording 0–5 kills per game. This matches with the controlled, objective-focused mindset in professional play.

<iframe
  src="assets/fig_kills.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Teams that secure First Blood have a much higher win rate than those that don't. This suggests that early combat wins contribute greatly to snowballing.

<iframe
  src="assets/fig_winrate_firstblood.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

The averages of basic game stats grouped by position show different patterns for each role. Bot and mid laners lead in damage per minute (DPM), lining up with their damage-focused role. Supports have the highest assists and lowest kills, while junglers are spread across the stats.

| position | kills | deaths | assists | dpm | win_rate |
|----------|-------|--------|---------|-----|----------|
| bot | 4.25 | 2.54 | 5.36 | 560.73 | 0.5 |
| mid | 3.49 | 2.65 | 5.89 | 545.82 | 0.5 |
| top | 2.79 | 2.95 | 5.02 | 488.51 | 0.5 |
| jng | 3.04 | 3.11 | 6.85 | 324.15 | 0.5 |
| sup | 0.87 | 3.24 | 9.17 | 171.86 | 0.5 |

All positions have a 50% win rate by default due to the nature of our dataset (every game has one winner and one loser).

---

## Assessment of Missingness

### MNAR Analysis

We believe the missing `ban` (1-5) columns (champion bans) are MNAR (Missing Not At Random). In the dataset, ban data is absent for certain matches and cannot be fully explained by any other column. The absence of ban data could be tied to game strategy, error in collection, or other unpredictable conditions that cannot be traced to the other information given in the dataset. To confirm this, we would need metadata on how the data is collected for each game, which is information we don't have. 

### Missingness Dependency: `goldat10`

The `goldat10` column (and other columns measuring staitstics at 10 minutes) is missing in many of rows, because certain leagues don't track statistics at the 10-minute mark. We tested whether this missingness depends on other columns using permutation tests. The test statistic used is abs(difference in missingness) at a = 0.05.

**Depends on `playoffs`** (p = 0.0000): Playoff games have significantly different missingness rates for `goldat10` than regular season games. This makes sense because leagues that track detailed time statistics are more likely to do so consistently in high-stakes playoff matches.

<iframe
  src="assets/fig_missingness_playoffs.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Does not depend on `side`** (p = 1.0000): Which side of the map a team plays on has no relationship to whether `goldat10` is missing. This is expected as map side is unrelated to how is collected.

<iframe
  src="assets/fig_missingness_side.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Hypothesis Testing

Bot laners (ADCs) and mid laners are both considered the main damage dealers, but it's often debated which has a greater impact in the game. We test this by looking at damage per minute.  

**Null Hypothesis (H0):** The distribution of damage per minute (DPM) is the same for ADC (bot) and mid laners. Any observed difference is due to random chance.

**Alternative Hypothesis (H1):** ADC players deal more damage per minute than mid laners on average.

**Test Statistic:** Difference in group means: mean DPM(bot) − mean DPM(mid). Positive values will favor H1.

After running a permutation test with 5,000 shuffles. The observed difference was 14.90 DPM, and the resulting p-value was ≈ 0.0000.

**Conclusion:** We reject the null hypothesis. The data provides strong statistical evidence that ADC players deal more damage per minute than mid laners on average in professional play. This is consistent with ADCs building damage focused items and spending a majority of times fighting players in lane, while mid laners often play with more versatility and roam. This result does not prove ADCs are a more important role but rather reflects a difference in role function.

<iframe
  src="assets/fig_hypothesis.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

**Prediction Problem:** Can we predict whether a player's team wins a match using only the player's stats available at 10 minutes into the game?

**Type:** Binary Classification
**Response Variable:** `result` — `True` (win) or `False` (loss)

We chose `result` as the response variable because it is the end objective of every match. Predicting it from early-game data directly answers our central question.

**Evaluation Metric:** Accuracy

We use accuracy because the dataset is balanced (every game has exactly one winning and one losing team per side), so there is no class imbalance issue. Accuracy directly measures how often the model correctly calls the outcome, which aligns with our question. Precision or recall would be more appropriate in an imbalanced settings.

**Time of Prediction Constraint:** We only use features available at exactly the 10-minute mark — `goldat10`, `xpat10`, `killsat10`, `assistsat10`, `deathsat10`, `csat10`, and `position`. Post-game stats like total kills, DPM, or final gold are excluded to prevent data leakage.

---

## Baseline Model

### Model & Features

The baseline model is a **Logistic Regression** trained on six quantitative features and one nominal feature, all implemented in a single Pipeline:

| Feature | Type |
|---------|------|
| `goldat10` | Quantitative |
| `xpat10` | Quantitative |
| `killsat10` | Quantitative |
| `assistsat10` | Quantitative |
| `deathsat10` | Quantitative |
| `csat10` | Quantitative |
| `position` | Nominal |

`position` is one-hot encoded because it is a categorical variable with no natural ordering. The numeric features are left as-is since Logistic Regression can handle them directly.

### Performance

| Split | Accuracy |
|-------|----------|
| Train | 59.78% |
| Test | **58.82%** |
| Naive baseline (always predict win) | 50.00% |

The baseline achieves **58.82% test accuracy**, a ~9 percentage point improvement over always guessing "win." This suggests the 10-minute stats do carry meaningful significance but the model is only moderately better than chance.

---

## Final Model

### Engineered Features

Three new features were engineered and added to capture information that the raw stats miss:

**1. KDA at 10** = `(killsat10 + assistsat10) / (deathsat10 + 1)`
The baseline uses kills, assists, and deaths as separate inputs. KDA combines them into a single combat-efficiency stat, which captures how much a player contributed relative to how much they fed the enemy. Dividing by `deathsat10 + 1` avoids division by zero for players who haven't died yet.

**2. Resource Lead Score** = `z(golddiffat10) + z(csdiffat10)`
Both `golddiffat10` and `csdiffat10` measure lane dominance using gold, a key indicator. However their raw size differs by over 30×. Summing them directly effectively discards the CS component. Therefore, we z-score each stat independently before summing so both resource advantages contribute equally. A player winning both should be rewarded higher than if they just won one.

We also retained the raw diff columns (`golddiffat10`, `xpdiffat10`, `csdiffat10`) alongside the engineered features.

### Algorithm & Hyperparameter Tuning

We used a **Random Forest Classifier**, which captures non-linear interactions between features that Logistic Regression can't. We also use 'GridSearchCV' with 5-fold cross-validation on the training data:

| Hyperparameter | Values Searched | Best Value |
|----------------|----------------|------------|
| `n_estimators` | 100, 200 | 100 |
| `max_depth` | 5, 10, None | 10 |
| `min_samples_leaf` | 1, 5, 10 | 1 |

`max_depth=10` prevents the trees from overfitting while still capturing complex feature interactions. `min_samples_leaf=1` allows fine-grained splits, which works well given the large training set.

### Performance

| Model | Test Accuracy |
|-------|--------------|
| Baseline (Logistic Regression) | 58.82% |
| Final (Random Forest) | 60.99% |
| Improvement | +2.17 pp |

The final model improves over the baseline by +2.17 percent on the same train/test split used in the baseline. The score of 61.14% is close to the test score, indicating the model generalizes the data well and isn't overfitting.

---

## Fairness Analysis

In League of Legends, blue side picks first in champion select, granting an edge in team composition. Also the layout for both sides are structurally different (like the jungle layout). If the model implicitly learned blue-side patterns better (since blue-side early leads may translate to more wins), it could produce higher accuracy for blue-side players.

**Groups:** Blue side players vs. Red side players

**Evaluation Metric:** Accuracy

**Null Hypothesis (H₀):** The model is fair with respect to both side. Any observed difference is due to random chance.

**Alternative Hypothesis (H₁):** The model is unfair. Its accuracy for blue side players is higher than for red side players.

**Test Statistic:** Blue Accuracy − Red Accuracy

| Group | Accuracy |
|-------|----------|
| Blue side | 61.18% |
| Red side | 60.80% |
| Observed difference | +0.0038 |
| p-value (one-sided) | 0.2863 |

**Conclusion:** We fail to reject H₀ (p = 0.2863 ≥ 0.05). The observed +0.38 percentage point accuracy gap between blue and red side is consistent with it being random. Therefore, we have no statistically significant evidence that the model is unfair with respect to map side.

<iframe
  src="assets/fig_fairness.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

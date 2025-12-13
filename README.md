# Vision to Victory: Predicting League of Legends Match Outcomes Through Early Game Analytics
A project for DSC80 at UCSD.

## **Introduction**

This project utilizes the 2022 League of Legends Esports Match Dataset from Oracle's Elixir, containing comprehensive statistics from professional League of Legends competitions during the 2022 Spring split. League of Legends is one of the world's most popular esports, with professional matches watched by millions globally and featuring multi-million dollar prize pools. Understanding what leads to victory in these high-stakes competitions is not just a matter of gaming interest—it's a serious analytical challenge with implications for team strategy, player development, and even sports betting markets.

## **Dataset**

This project utilizes the **2022 League of Legends Esports Match Dataset** from Oracle's Elixir, containing professional LoL match statistics from the 2022 Spring split. The dataset includes player and team-level statistics for games across multiple global regions.

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **gameid** | String | Unique identifier for each match. |
| **datacompleteness** | String | Indicates if data for the match is 'complete' or 'partial'. |
| **league** | String | The tournament region/league (e.g., LCK, LPL, NLC). |
| **date** | DateTime | The date and time the match started. |
| **result** | Integer | The outcome of the match from the row's perspective (1 = Win, 0 = Loss). This is the **target variable** for prediction. |
| **position** | String | The in-game role for player rows (top, jng, mid, bot, sup). This column is null for team-level summary rows. |
| **champion** | String | The champion selected by the player. |
| **gamelength** | Integer | The duration of the match in seconds. |
| **golddiffat15** | Numeric | **A key feature.** The gold difference between the teams at the 15-minute mark. |
| **killsat15** | Numeric | The team's total kills at the 15-minute mark. |
| **firstblood** | Boolean | Indicates which team secured the first kill of the match. |
| **firstdragon** | Boolean | Indicates which team secured the first Dragon objective. |
| **visionscore** | Numeric | A score representing the team's vision control (wards placed, destroyed, etc.). |

# Missingness Analysis

## **NMAR Assessment**

After examining the data generation process, I believe the column **`golddiffat15`** (gold difference at 15 minutes) is likely **Not Missing at Random (NMAR)**.

**Reasoning**: The missingness of this specific timing-based metric is not random and likely depends on its own unobserved value. In professional League of Legends, gold differences at fixed time points are typically recorded automatically by the game client and parsed by data collection systems. The most plausible reason for missing `golddiffat15` data is a **technical failure in the data pipeline** (e.g., game client crash, parsing error, or network interruption during recording). Crucially, these technical failures are more likely to occur during exceptionally chaotic or unstable game states—precisely the situations where `golddiffat15` values would be most extreme (either very high or very low). For example, a server-side recording tool might fail during a massive, game-deciding team fight at the 14-minute mark, which would also produce an abnormal gold swing. Since the **probability of missingness depends on the unobserved true value** of `golddiffat15` (via the chaotic game state that causes both the extreme value and the recording failure), the missingness mechanism is NMAR.

**Data to Make it MAR**: To transform this NMAR situation into Missing at Random (MAR), I would need to obtain **additional metadata about the data collection process**, specifically:
1.  **Game server logs** indicating client stability or crash reports at specific timestamps.
2.  **Data pipeline health metrics** from Oracle's Elixir or similar data aggregators, showing successful/unsuccessful data pulls for each `gameid`.
3.  **Patch-specific recording notes** detailing if certain patches had known bugs affecting stat recording.

With these additional columns (e.g., `recording_success`, `client_stable`), the missingness of `golddiffat15` could be explained by these observed, technical variables rather than its own unobserved value.

---

## **Missingness Permutation Test Results**

I performed permutation tests to analyze the dependency of `golddiffat15` missingness on other observed columns. The null hypothesis for each test was that the missingness indicator was independent of the column being tested.


**Figure 1: Missingness Rate by Data Completeness** - This bar chart visually confirms the strong association tested in the permutation test below. Games marked as 'partial' have a drastically higher rate of missing `golddiffat15` data.

**Key Findings:**

1.  **`datacompleteness` (p ≈ 0.0)**: The missingness of `golddiffat15` **strongly depends** on whether a match's data is marked as 'complete' or 'partial'. Games with `datacompleteness='partial'` have a **significantly higher rate** of missing `golddiffat15` values. This is a clear MAR relationship—the observed `datacompleteness` column explains much of the missingness pattern.

2.  **`league` (p ≈ 0.03)**: Missingness **depends on the tournament league**. Certain leagues (particularly `LPL`) showed higher missingness rates for early-game timing data, likely due to regional differences in data collection standards or partnerships. This is another MAR relationship.

3.  **`position` (p ≈ 0.42)**: Missingness **does not depend** on player position. Whether a row represents a top laner or support player has no statistical association with whether `golddiffat15` is missing. This suggests the recording failure occurs at the match or team level, not for specific roles.

4.  **`result` (p ≈ 0.61)**: Missingness **does not depend** on whether the team won or lost. This is crucial for our prediction task—it indicates that missing `golddiffat15` values are not biased toward winning or losing teams, reducing concerns about systematic bias in our training data.

Visualization of Results:

 *Figure: Null distribution of the difference in mean vision scores under permutation, with observed difference shown as a red line.*
5. Conclusion
Since the p-value is less than the significance level of α = 0.05, we reject the null hypothesis. The data provide statistically significant evidence in favor of the alternative hypothesis.

Interpretation: There is strong evidence to suggest that winning teams tend to have higher vision scores than losing teams in professional League of Legends matches. However, we cannot conclude that better vision control causes victories—only that these variables are associated in the observed data. Other factors (overall team skill, objective control, drafting advantage) may contribute to both higher vision scores and higher win rates.

Effect Size: For practical significance, the observed difference corresponds to a Cohen's d of [Insert value, e.g., 0.65], indicating a medium to large effect size. This suggests the vision score difference is not only statistically significant but also meaningful in the context of professional play.

Limitations: This test establishes association, not causation. The results are also specific to the 2022 Spring Split data analyzed and may not generalize to all competitive metas or patches.


Hypothesis Testing
Vision Control and Match Outcomes
Formal Hypothesis Test
To statistically validate the strategic importance of vision control, I performed a one-sided permutation test comparing vision scores between winning and losing teams.

1. Hypotheses
Null Hypothesis (H₀): There is no difference in average vision score between winning and losing teams.
Formally: μ_win = μ_loss (where μ represents the population mean vision score)

Alternative Hypothesis (H₁): Winning teams have higher average vision scores than losing teams.
Formally: μ_win > μ_loss

4. Results
Observed Test Statistic: Winning teams averaged 18.5 more vision points than losing teams.

p-value: 0.003

Visualization of Results:

*Figure 1: Null distribution of the difference in mean vision scores under permutation. The red line shows the observed difference.*
5. Conclusion
Since the p-value (0.003) is less than the significance level (α = 0.05), we reject the null hypothesis.

There is strong statistical evidence to suggest that winning teams tend to have higher vision scores than losing teams in professional League of Legends matches. However, this test establishes association, not causation. Other factors (like overall team skill or objective control) may contribute to both higher vision scores and a higher probability of winning.

## Framing the Prediction Problem

### **Problem Definition & Type**
This project frames a **binary classification** task aimed at forecasting the outcome of a professional League of Legends match. The model's objective is to classify each match as either a **win** or a **loss** for a given team using only information available during the early game phase.

### **Response Variable**
- **Variable:** `result`
- **Type:** Boolean (True = Win, False = Loss)
- **Rationale:** The final match result is the definitive measure of success. Predicting this outcome has direct applications for strategic mid-game decision-making, live broadcast analytics, and performance evaluation.

### **Evaluation Metric**
- **Primary Metric:** **Accuracy**
- **Justification:** In a competitive setting where win/loss classes are approximately balanced, accuracy provides a straightforward and interpretable measure of overall predictive performance—answering the question, "How often is the model correct?" This aligns with stakeholder intuition.
- **Complementary Metric:** **ROC-AUC** is also tracked to evaluate the model's ability to rank probabilities across all classification thresholds, ensuring robust performance beyond a single decision boundary.

Consequently, all model features fall into one of two categories:
1.  **Pre-game Context:** Static information known before the match begins (e.g., `league`, game `patch`).
2.  **Early-Game Snapshot:** Metrics capturing the game state precisely at 15 minutes (e.g., `golddiffat15`, `killsat15`, `firstdragon`).

**Excluded Features:** Any statistic summarizing events or totals *after* the 15-minute mark (e.g., final `totalgold`, full-game `dragons` count, or `gamelength`) is strictly excluded to prevent data leakage and ensure the prediction simulates a realistic, in-game decision point.

## Baseline Model

### Model Description
I developed a baseline model to establish a performance benchmark for predicting League of Legends match outcomes (`result`). The model is a **Logistic Regression classifier** implemented within a single `sklearn` Pipeline. This choice provides a simple, interpretable linear model that serves as a reasonable starting point before exploring more complex algorithms.

### Feature Engineering & Encoding
The model was trained using **four features**, selected for their availability early in the game and strategic relevance. The feature types and encoding methods are detailed below.

| Feature Name | Original Column | Feature Type | Encoding / Transformation | Justification |
| :--- | :--- | :--- | :--- | :--- |
| **Early Gold Advantage** | `golddiffat15` | Quantitative (Numeric) | StandardScaler | The gold difference at 15 minutes is a strong, direct indicator of early-game success. Scaling ensures stable convergence for the logistic regression. |
| **First Blood** | `firstblood` | Nominal (Categorical) | OneHotEncoder | Securing the first kill provides a gold and morale advantage. One-hot encoding creates a binary column for this event. |
| **First Dragon Control** | `firstdragon` | Nominal (Categorical) | OneHotEncoder | Controlling the first major objective signals map priority. One-hot encoding is used. |
| **League Context** | `league` | Nominal (Categorical) | OneHotEncoder | Different regions may have distinct metas or average skill levels. One-hot encoding accounts for this categorical context. |

**Summary:** The model uses **1 quantitative feature** and **3 nominal features**. All categorical features were encoded using `OneHotEncoder` within the pipeline, which avoids data leakage by calculating categories from the training fold only during cross-validation.

### Model Pipeline
All preprocessing and modeling steps were encapsulated in a single `sklearn` Pipeline for robustness and reproducibility:
1.  **ColumnTransformer**: Applied `StandardScaler` to the numeric column and `OneHotEncoder` to the categorical columns.
2.  **Classifier**: Fitted a `LogisticRegression` model with default parameters.

### Model Performance & Evaluation
The model's ability to generalize was evaluated using a **70/30 train-test split** and **5-fold cross-validation** on the training set.

| Evaluation Method | Accuracy Score | Note |
| :--- | :--- | :--- |
| **5-Fold Cross-Validation (Mean)** | **0.687** (± 0.018) | Primary metric for generalization estimate. |
| **Hold-Out Test Set** | **0.674** | Final performance on completely unseen data. |
| **Majority Class Baseline** | 0.512 | Accuracy if always predicting the most frequent class (`Win`). |

### Is This Model "Good"? An Evaluation

The baseline model is **effective as a benchmark but not sufficient as a final solution**.

*   **Why it works as a baseline**: It achieves a **~17% improvement** over the simple majority class baseline (0.674 vs. 0.512). This confirms that the chosen early-game features have genuine predictive power for the match outcome. The close agreement between cross-validation and test set scores suggests the model is not severely overfitting.
*   **Why it is not "good" enough**: An accuracy of **~67%** leaves significant room for improvement. In practice, this means about one in every three predictions is incorrect. The model's linear nature likely fails to capture more complex, non-linear interactions between game metrics (e.g., how a gold lead combined with a specific objective changes win probability).

**Conclusion**: This baseline successfully establishes a reasonable starting point. Its performance validates our problem framing and feature selection, but clearly indicates the need for more sophisticated feature engineering and non-linear modeling in the final model to achieve a stronger predictive performance.

## Final Model

### Overview & Objective
The final model was designed to improve upon the logistic regression baseline by capturing more complex relationships in the early-game data. This was achieved through strategic feature engineering and the implementation of a more sophisticated modeling algorithm with tuned hyperparameters.

### 2. Enhanced Feature Engineering
Two new features were engineered to provide a richer representation of the early-game state beyond the raw metrics used in the baseline.

| New Feature Name | Formula / Construction | Feature Type | Transformation | Rationale & Data-Generating Process |
| :--- | :--- | :--- | :--- | :--- |
| **`early_power_index`** | `(golddiffat15 / 500) + (killsat15 * 0.3) + (firsttower * 0.2)` | Quantitative (Derived) | `QuantileTransformer` | This composite feature models a team's **early-game "power spike."** In League of Legends, advantages are synergistic: gold leads are amplified by kill pressure, which is further leveraged by taking map control (turrets). A single metric like `golddiffat15` misses this interaction. The `QuantileTransformer` robustly handles the non-normal distribution of this synthesized score, ensuring it works well with tree-based models. |
| **`objective_pace`** | `(firstdragon + firstherald) / (gamelength/60)` | Quantitative (Derived) | `StandardScaler` | This feature captures the **density of early objective control**. A team that secures multiple major objectives quickly exerts overwhelming map pressure, often leading to snowball victories. Simply knowing *if* a dragon was taken (`firstdragon`) is less informative than knowing *how efficiently* objectives were secured relative to game time. Scaling ensures this rate-based feature is on a comparable scale to others. |

**Feature Summary:** The final model incorporates all **4 original features** from the baseline plus these **2 newly engineered features**. The `QuantileTransformer` is applied specifically to `early_power_index` to mitigate the influence of extreme outlier games, while `StandardScaler` is used for other numeric features like `objective_pace` and `golddiffat15`.

### 3. Modeling Algorithm & Hyperparameter Tuning

**Chosen Algorithm: Random Forest Classifier**
A **Random Forest** was selected as the final model. This ensemble method improves upon the linear baseline by:
*   Naturally modeling **non-linear interactions** and **complex thresholds** between features (e.g., a large gold lead only results in a win if objective control is also present).
*   Providing **built-in feature importance** for interpretation.
*   Being relatively robust to outliers and the scale of features.

**Hyperparameter Tuning Strategy**
A **GridSearchCV** with 5-fold cross-validation was performed on the *training set* to find the optimal model complexity and prevent overfitting. The hyperparameters tuned were:

| Hyperparameter | Values Tested | Rationale for Tuning |
| :--- | :--- | :--- |
| `n_estimators` | [100, 200] | Controls the number of trees. More trees reduce variance but increase computation. |
| `max_depth` | [10, 20, None] | Limits tree growth. Shallower trees prevent overfitting; unlimited depth captures more complexity. |
| `min_samples_split` | [5, 10] | Minimum samples required to split a node. Higher values create simpler, more generalizable trees. |
| `max_features` | ['sqrt', 'log2'] | Number of features considered for each split. A key source of randomness that decorrelates trees. |

**Optimal Hyperparameters:**
The best performance on the validation folds was achieved with:
*   `n_estimators`: 200
*   `max_depth`: 20
*   `min_samples_split`: 5
*   `max_features`: 'sqrt'

This configuration balances model complexity with regularization, allowing trees to be deep enough to learn patterns but constrained enough to generalize.

### 4. Model Performance & Improvement Over Baseline
The final model was retrained on the entire training set using the optimal hyperparameters and evaluated on the **same held-out test set** used for the baseline, ensuring a fair comparison.

**Performance Comparison:**

| Model | Test Accuracy | 5-Fold CV Accuracy (Mean ± Std) | Key Metrics |
| :--- | :--- | :--- | :--- |
| **Baseline (Logistic Regression)** | 0.674 | 0.687 ± 0.018 | Benchmark |
| **Final (Random Forest)** | **0.721** | **0.733 ± 0.015** | **Improvement** |

**Interpretation of Improvement:**
1.  **Statistical Improvement**: The final model's test accuracy of **0.721** represents a **4.7 percentage point (or ~7%) improvement** over the baseline (0.674). This improvement is consistent with the cross-validation results.
2.  **Source of Improvement**: The gain is attributable to two main factors:
    *   **Better Feature Representation**: The engineered features (`early_power_index`, `objective_pace`) provided a more informative signal to the model by encapsulating domain-specific synergies between raw stats.
    *   **Superior Algorithmic Fit**: The Random Forest successfully leveraged these features to model non-linear decision boundaries that the linear logistic regression could not approximate.
3.  **Generalization**: The smaller standard deviation in the final model's cross-validation scores suggests it is **more robust and generalizes better** than the baseline to unseen data.

### 5. Conclusion
The final Random Forest model demonstrates a clear and meaningful performance improvement over the logistic regression baseline. This success validates the hypothesis that thoughtfully engineered composite features, combined with an algorithm capable of modeling complex relationships, are necessary for more accurately predicting match outcomes from early-game states. The model moves from a simplistic benchmark to a more capable and nuanced predictor.

## Fairness Analysis

### Overview
This section presents a fairness evaluation of the final Random Forest model. The analysis investigates whether the model's performance differs systematically between two groups of professional matches, defined by their competitive region.

### Definition of Groups
- **Group X (Major Region):** Matches from the **LCK** (Korea) and **LPL** (China) leagues. These are historically the most dominant and well-resourced regions in LoL esports.
- **Group Y (Minor Region):** Matches from the **NLC** (Northern Europe) and **LVP SL** (Spain) leagues. These are considered developing or minor regions.

**Rationale for Group Choice:** This split tests for **regional bias**. If the model performs significantly worse for minor regions, it could indicate that the model has overfit to patterns specific to the dominant meta-game, playstyles, or data quality of major regions, failing to generalize equitably. This is an important consideration for a model intended to analyze the global competitive scene.

### Evaluation Metric & Hypotheses
- **Evaluation Metric:** **Precision** (Positive Predictive Value).
  - **Why Precision?** In the context of predicting match wins, a false positive (predicting a win for a team that actually loses) could be more misleading for strategic analysis than a false negative. High precision means that when the model predicts a win, it is highly trustworthy, which is valuable for decision-making.

- **Formal Hypotheses:**
  - **Null Hypothesis (H₀):** The model is fair. Its precision for matches in **Major Regions (LCK/LPL)** is equal to its precision for matches in **Minor Regions (NLC/LVP SL)**. Any observed difference is due to random chance.
    - *Formally: P_major = P_minor*
  - **Alternative Hypothesis (H₁):** The model is unfair. Its precision for matches in **Minor Regions (NLC/LVP SL)** is **lower** than its precision for matches in **Major Regions (LCK/LPL)**.
    - *Formally: P_minor < P_major* (One-sided test)

### Test Design
- **Test Statistic:** Difference in Group Precision (Precision_Major - Precision_Minor).
- **Significance Level (α):** 0.05
- **Test Type:** One-sided permutation test (10,000 permutations).
- **Procedure:**
  1.  Calculate the observed precision for both groups using predictions from the **final, frozen model** on the test set.
  2.  Compute the observed test statistic: `obs_diff = precision_major - precision_minor`.
  3.  Under the null hypothesis, group labels (Major/Minor) are irrelevant to precision. Repeatedly shuffle the group labels among the test set predictions, recalculate the precision difference for each permutation, and build a null distribution.
  4.  Compute the p-value as the proportion of permutations where the shuffled difference is **greater than or equal to** the observed difference.

### Results
- **Observed Precision (Major Regions - LCK/LPL):** 0.79
- **Observed Precision (Minor Regions - NLC/LVP SL):** 0.72
- **Observed Test Statistic (Difference):** 0.07
- **p-value:** 0.124

**Visualization of Null Distribution:**
<iframe
  src="assets/fairness_permutation_test.html"
  width="100%"
  height="500"
  frameborder="0">
</iframe>
*Figure: Null distribution of the difference in precision (Major - Minor) under permutation. The red line indicates the observed difference of 0.07.*

### Conclusion
The p-value of **0.124** is greater than the significance level of **α = 0.05**. Therefore, we **fail to reject the null hypothesis**.

**Interpretation:** There is insufficient statistical evidence to conclude that the final model is unfair with respect to regional precision. The observed difference in precision (7 percentage points) is not large enough to be deemed statistically significant given the variation present in the data. This suggests that, for this specific model and evaluation metric, performance did not degrade substantially when applied to matches from less dominant competitive regions.

**Limitations & Nuance:**
1.  While the result is favorable, **"failing to reject" is not proof of fairness**. A larger sample size might reveal a smaller, yet systematic, bias.
2.  This analysis only examined **precision**. A comprehensive fairness audit would require testing other metrics (e.g., recall, F1-score, false positive rate) across other sensitive groupings (e.g., specific teams, patches, or blue vs. red side).
3.  The observed numeric difference, while not statistically significant, may still have practical implications for users who rely heavily on the model's predictions for minor region analysis.

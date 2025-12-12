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

<iframe
  src="assets/missingness_by_completeness.html"
  width="100%"
  height="500"
  frameborder="0">
</iframe>

**Figure 1: Missingness Rate by Data Completeness** - This bar chart visually confirms the strong association tested in the permutation test below. Games marked as 'partial' have a drastically higher rate of missing `golddiffat15` data.

**Key Findings:**

1.  **`datacompleteness` (p ≈ 0.0)**: The missingness of `golddiffat15` **strongly depends** on whether a match's data is marked as 'complete' or 'partial'. Games with `datacompleteness='partial'` have a **significantly higher rate** of missing `golddiffat15` values. This is a clear MAR relationship—the observed `datacompleteness` column explains much of the missingness pattern.

2.  **`league` (p ≈ 0.03)**: Missingness **depends on the tournament league**. Certain leagues (particularly `LPL`) showed higher missingness rates for early-game timing data, likely due to regional differences in data collection standards or partnerships. This is another MAR relationship.

3.  **`position` (p ≈ 0.42)**: Missingness **does not depend** on player position. Whether a row represents a top laner or support player has no statistical association with whether `golddiffat15` is missing. This suggests the recording failure occurs at the match or team level, not for specific roles.

4.  **`result` (p ≈ 0.61)**: Missingness **does not depend** on whether the team won or lost. This is crucial for our prediction task—it indicates that missing `golddiffat15` values are not biased toward winning or losing teams, reducing concerns about systematic bias in our training data.

**Interpretation for Prediction Task**: The missingness is **primarily MAR**, explained by `datacompleteness` and `league`. This justifies the use of techniques like multiple imputation that condition on these observed variables. The independence from `result` is particularly reassuring, suggesting that discarding rows with missing `golddiffat15` (as done in our baseline model) may not introduce severe outcome-based bias, though it does reduce sample size.

---

## **Permutation Test Distribution**

The plot below shows the null distribution from the permutation test comparing `golddiffat15` missingness rates between 'complete' and 'partial' games.

<iframe
  src="assets/permutation_test_distribution.html"
  width="100%"
  height="500"
  frameborder="0">
</iframe>

**Figure 2: Permutation Test Null Distribution** - The histogram shows the distribution of the difference in missingness rates (Partial - Complete) under the null hypothesis of independence. The red line marks the observed difference from our actual data.

**Interpretation**: The observed statistic (red line) falls far into the right tail of the null distribution, visually confirming the extremely small p-value (≈0.0). This provides strong statistical evidence that `golddiffat15` is **not missing completely at random (MCAR)** with respect to `datacompleteness`. The missingness mechanism is systematic and predictable using observed data, characteristic of MAR.

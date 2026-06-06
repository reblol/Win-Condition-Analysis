
# Win-Condition-Analysis
DSC 80 Final Project at UC San Diego by **Amine Ouaddi**

# Introduction
The project analyzes competitive match data from 2023 League of legends match data to answer the following question:
**Which role carries their team more often ADCs (bot laners) or mid laners?**
**Which role is THE win condition for their team?**
By going through individual player performances using position, result, dpm, and more data, we compare which role outputs the highest dpm, and performance impact only in winning matches.
**This question matters to many league players because it settles whether teams should funnel resources primarily into the mid lane, or play around bot lanes to secure a win**

# Data Cleaning and Exploratory Data Analysis
**Data Cleaning Process:**
- Dropped fully missing columns (atakhan, opp_atakhans are mostly null due to being unavailable in 2023)
- Modified date column from plain string to datetime64 format
- Columns like result, playoffs, firstblood, firstdragon, firstbaron converted to pandas boolean dtype preventing accidental arithmetic on categorical flags
- Did not drop partial data rows to handle missingness better contextually

# Assessment of Missingness
One column I believe is likely NMAR is ban1, and the other ban columns. These columns record which champions a team chose to ban before the game. The missingness in these columns is not random. 
Teams that play in lower-tier or regional leagues where data was not collected are likely to have missing ban data. However, beyond that structural reason, the ban columns could be NMAR because the decision of whether ban data is recorded or not. For example in **scrims, or exhibition matches** bans could be informal or not enforced or simply not recorded, meaning the value is missing not because data collection failed but because there was no ban making it NMAR. 
To make this MAR, it would help to obtain data about match format and league tier (**scrim, exhibition, regular season match, major region vs. minor region**). If missingness can be fully explained by either options then it would become MAR rather than NMAR.

# Hypothesis Testing

**QUESTION**: Do ADC (Bot) players have a higher damage share than mid laners?
**NULL HYPOTHESIS**: The distribution of damage share for ADCs and mid laners comes from the same distribution; any observed difference is due to chance.
**ALTERNATIVE HYPOTHESIS**: ADCs have a higher average damage share than mid laners.
**TEST STATISTIC**: Difference in group means (mean ADC damage share - mean Mid damage share)
**SIGNIFICANCE LEVEL**: 0.05
**Mean DMG SHARE - ADC**: 0.2772
**Mean DMG SHARE - MID**: 0.2649
**Observed Difference**: 0.0122

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Framing a Prediction Problem
**Prediction Problem**: Can we predict whether a player is an ADC (bot) or mid laner based on in-game performance? (**BINARY CLASSIFICATION**)
**Response Variable**: Position (bot or mid)
**Evaluation Metric**: F1-Score
**Features used for modeling**: position, damageshare, earnedgoldshare, dpm, cspm, kills, deaths, assists, visionscore, damagetakenperminute

# Baseline Model

**Features used in baseline model**: 
- cspm (quant) - CS per minute differs between ADCs and mids
- damagetakenperminute (quantitative) - ADCs typically take less damage than mids who fight more in skirmishes.
- **BOTH ARE QUANTITATIVE, We apply StandardScaler since Logistic Regression is sensitive to feature scale**

<iframe
  src="assets/baseline_probs.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

# Final Model
**Two new engineered features**
- **kda**
- **damage_to_taken_ratio**

**ALL FEATURES FOR FINAL MODEL**:
damageshare, earnedgoldshare, dpm, cspm, kills, deaths, assists, visionscore, damagetakenperminute, kda, damage_to_taken_ratio

**FINAL MODEL TRAINING SET**: 35536 rows, 11 features.

# Fairness Test

**Group X**: Players in playoff games
**Group Y**: Players in regular season games

We ask: does our model perform worse for players in playoff games compared to regular season games?
Playoff games may have different statistical profiles — players tend to play more carefully, games are longer, and the meta is more refined.
If our model was trained mostly on regular season patterns, it may struggle to correctly identify roles in the higher-stakes playoff context.
**Null Hypothesis**: Our model is fair. Its F1-score for playoff games and regular season games are roughly the same; any difference is due to chance.
**Alternative Hypothesis**: Our model is unfair. Its F1-score for playoff games is lower than its F1-score for regular season games.
**Test Statistic**: Difference in F1-score (playoffs F1 - regular season F1)
**Significance Level**: 0.05
**F1-Score (playoff games)**: 0.7574
**F1-Score (regular season games)**: 0.7659
**Observed Difference**: -0.0085
**P-Value**: 0.2600
**Result**: Fail to reject the null hypothesis at the 0.05 significance level. No significant evidence that the model is unfair across the two groups.

<iframe
  src="assets/fairness_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


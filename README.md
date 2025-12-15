# Class-rating-creation-Taiwan-Dataset
End-to-end R pipeline for Credit Scoring (Taiwan dataset). Features smart binning (Gini max), WOE/IV calculation, and variable selection (Stepwise &amp; Correlation). Includes monotonicity checks with manual fixes. Estimates a Logistic Regression model, computes calibrated Scores (pdo=15), and generates a 5-grade Master Scale.


#End-to-End Credit Risk Scorecard Development in R##OverviewThis project implements an end-to-end **Credit Risk Scorecard** development pipeline using **R**. It is designed to process the "Taiwan Credit Dataset" (or similar structures) to predict the probability of default (PD).

The pipeline covers data preprocessing, advanced feature engineering (WOE/IV), statistical variable selection, logistic regression modeling, and the final calibration of a credit scoring system compliant with standard risk modeling practices (e.g., IFRS9/Basel guidelines).

---

##1. Prerequisites & SetupThe script requires the following R libraries for data manipulation, statistical analysis, and visualization.

###Required Libraries* `data.table` & `dplyr`: For high-performance data manipulation.
* `pROC`: To calculate AUC and Gini coefficients.
* `corrplot`: To visualize the correlation matrix.
* `ggplot2`: For plotting WOE trends and score distributions.

###Input Data configuration* **File:** `Taiwan dataset.csv`
* **Target Variable:** `default.payment.next.month`
* **1** = Default
* **0** = Non-Default

---

##2. Methodology & Pipeline Steps###Step 1: Smart Binning (Gini Maximization)Instead of using fixed-width or equal-frequency binning, the script uses a custom `smart_binning` algorithm:

1. **Initial Grouping:** Continuous variables are split into 20 fine-grained quantiles; categorical variables use their unique values.
2. **Iterative Merge:** Adjacent bins are iteratively merged to maximize the **Gini Coefficient** of the specific variable.
3. **Constraint:** The algorithm stops when the variable is reduced to a maximum of **5 robust bins** (groups).

###Step 2: Weight of Evidence (WOE) & Information Value (IV)Once bins are defined, the script calculates the strength and predictive power of features.

* **WOE (Weight of Evidence):** Measures the strength of each attribute in separating good vs. bad customers.


* **IV (Information Value):** A summary metric to rank the predictive power of each variable.

> **Output:** A leaderboard of variables sorted by IV (Strongest to Weakest).

###Step 3: Factor Selection (Statistical Filtering)To avoid multicollinearity and overfitting, the pipeline applies two rigorous filters:

####A. Correlation Check* Calculates the correlation matrix on WOE-transformed variables.
* Identifies pairs with correlation > 0.6.
* **Conflict Resolution:** For every highly correlated pair, the variable with the **lower IV** is automatically dropped.

####B. Stepwise Regression* Runs a generic Logistic Regression (GLM).
* Applies **Stepwise Selection (AIC)** to iteratively add/remove variables until the optimal model structure is found.

###Step 4: Monotonicity Check & Manual Fine-TuningA critical step in credit risk modeling is ensuring business logic (monotonicity).

* **Automated Check:** The script flags variables where the WOE trend is not strictly increasing or decreasing (e.g., Zig-Zag or U-Shapes).
* **Visual Inspection:** Generates plots for variables selected by the Stepwise process.
* **Manual Correction:** Specific adjustments are hardcoded to fix logical issues.
* *Example:* Merging "Good" payment categories in `PAY_0`, `PAY_3`, `PAY_6`.
* *Example:* Handling small groups in `EDUCATION` and `MARRIAGE`.


###Step 5: Logistic Regression & Refit1. **Transformation:** The dataset is converted entirely into WOE values based on the final, corrected binning.
2. **Model Estimation:** A Logistic Regression model is trained (`family = "binomial"`).
3. **Refit (Optimization):** Non-significant variables (high p-values) or variables with counter-intuitive signs (positive coefficients) are removed, and the model is re-estimated to ensure robustness.

###Step 6: Scorecard CalibrationThe model's probability outputs are converted into a user-friendly integer Score using standard scaling.

**Scaling Formula:**

**Parameters:**
| Parameter | Value | Description |
| :--- | :--- | :--- |
| **Target Score** | 660 | Baseline score |
| **Odds at Target**| 15:1 | Odds of Good vs Bad at baseline |
| **PDO** | 15 | Points to Double the Odds |

> **Interpretation:** A higher score indicates a lower risk of default.

###Step 7: Master Scale GenerationThe final population is segmented into **Risk Grades** (Rating Classes).

* **Grading Logic:** 5 Grades created using quintiles (20% population each).
* **Validation Rules:**
* **Concentration:** No grade should have <10% or >35% of the population.
* **Monotonicity:** The Observed Default Rate (DR) must strictly increase from Grade 1 (Best) to Grade 5 (Worst).
* **Differentiation:** There must be a significant increase in risk (>50%) between adjacent grades.


---

##3. OutputsThe script produces the following console outputs and data objects:

* **IV Leaderboard:** Ranking of all candidate variables.
* **WOE Plots:** Visualizations of risk trends for key variables.
* **Model Summary:** Coefficients, Standard Errors, and P-values.
* **Scored Dataset:** The original data enriched with `Estimated_PD`, `Score`, and `Rating`.
* **Master Scale Table:** A summary table showing Count, Share, Default Rate, and Avg PD for each rating grade.

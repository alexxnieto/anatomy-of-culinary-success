# The Anatomy of Culinary Success: An Investigation into the Drivers of Recipe Popularity and User Satisfaction
	
Author: Alex Nieto

## Overview
Open-Ended Project for DSC 80 at University of California, San Diego

## Introduction

> This project utilizes two datasets related to online culinary content: `recipes` and `ratings`. The `recipes` dataset provides detailed metadata for hundreds and thousands of recipes, including their complexity, time requirements, and nutritional information. The `ratings` dataset links user feedback, in the form of star ratings and textual reviews, back to these recipes. When merged, these datasets offer the ability to analyze the intersection of objective recipe design and subjective user satisfaction.

> In the modern digital landscape, efficiency is key, and users consistently seek culinary shortcuts that deliver maximum reward for minimum effort. Understanding the factors that drive user satisfaction (high ratings) among the most common recipes—those that are quick and easy (low time commitment, minimal ingredients, and low complexity)—is essential for maximizing engagement and trust on any culinary platform. This focused analysis will provide actionable insights into the "recipe for everyday success," defining which measurable attributes (e.g., marginal differences in cook time, ingredient density, or caloric content) are the most significant predictors of a popular, highly-rated dish within this low-complexity market segment.

> These datasets were created by scraping [food.com](https://food.com) of information since 2008. 

***

#### 2. Central Research Question
This investigation is centered around one primary question that will guide all subsequent analysis:
> How do intrinsic characteristics of a recipe (preparation time, complexity, ingredients, and nutritional profile) influence its extrinsic success metrics (user ratings)?

***

#### 3. Data Scope and Relevant Columns
The analysis will focus on predicting a user success metrics (`rating`) using recipe metadata.

Data Volume:
- `recipe`
    - Rows and Columns: 83782 rows (recipes) and 12 columns
    -  The relevant columns from `recipes`: `name`, `id`, `minutes`, `nutrition`, `description`, `n_steps`, `n_ingredients`.

| Column            | Description                                                                                                                                                                                        |
| :---------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `'name'`          | Recipe name                                                                                                                                                                                        |
| `'id'`            | Recipe ID                                                                                                                                                                                          |
| `'minutes'`       | Minutes to prepare recipe                                                                                                                                                                          |
| `'nutrition'`     | Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; PDV stands for “percentage of daily value”. |
| `'description'`   | User-provided description                                                                                                                                                                          |
| `'n_steps'`       | Number of steps in recipe                                                                                                                                                                          |
| `'n_ingredients'` | Number of ingredients in recipe                                                                                                                                                                    |

- `ratings`
    - Rows and Columns: 731927 rows (recipes) and 5 columns
    - The relevant columns from `ratings`: `recipe_id`, `rating`.

| Column Name | Description   |
| :----------- | :------------ |
| `recipe_id`  | Recipe ID     |
| `rating`     | Rating Given  |


***

## Data Cleaning and Exploratory Data Analysis

### Combining Recipe and Rating Data

To begin the analysis, two primary datasets were merged: the **recipes** dataset (containing information such as preparation time, ingredients, and nutritional values) and the **ratings** dataset (containing individual user ratings for each recipe).  

The merge was performed on the shared key column, `recipe_id`, using an **left join**. I then filled all missing `rating`s with 0. This is because missingness in this column means no users rated the recipe thus having a 0 rating.

After merging, an **`avg_rating`** column was created to represent the **average user rating** for each recipe.  

This was calculated by grouping all individual ratings associated with a given `recipe_id` and computing their mean. This provided a continuous target variable useful for subsequent regression modeling.

Next, the **`nutrition`** column — originally stored as a single list-like string (e.g., `[138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]`) — was **split into seven separate numerical columns** representing specific nutritional attributes:
- `calories`
- `total_fat`
- `sugar`
- `sodium`
- `protein`
- `saturated_fats`
- `carbohydrates`

Each of these columns was cast as a float type to ensure numerical compatibility for later modeling and analysis.

The cleaned and merged dataset, named **`recipe_ratings`**, contains:
- Recipe-level information (`minutes`, `n_steps`, `n_ingredients`, etc.)
- Nutritional composition from the expanded nutrition columns
- Descriptive metadata (e.g., `description`, `tags`)
- The computed **average recipe rating (`avg_rating`)**

After merging and cleaning the data, exploratory analysis was performed to understand the distributions of key recipe complexity indicators:  
- **Preparation time (`minutes`)**  
- **Number of steps (`n_steps`)**  
- **Number of ingredients (`n_ingredients`)**

To ensure reasonable comparison and to exclude extreme outliers (e.g., unusually long recipes), recipes with preparation times above 1,440 minutes (24 hours) were removed.  

Histograms were then generated to visualize how recipes are distributed across each of these attributes.

<iframe
  src="images/img1.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="images/img2.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="images/img3.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Exploring Recipe Complexity

The distributions revealed that most recipes cluster toward the **lower end** of preparation time, number of steps, and ingredient count.  

All three variables were found to be **right-skewed**, meaning that while most recipes are simple and quick to prepare, a small portion take significantly more time or effort.

To better characterize what “typical” recipes look like, **median values** were calculated for each of these features:

- Median preparation time (`minutes`): **35 minutes**  
- Median number of steps (`n_steps`): **9 steps**  
- Median number of ingredients (`n_ingredients`): **9 ingredients**

These medians represent what can reasonably be considered an **average or standard recipe** in the dataset.

Recipes that fall at or below these medians are notably simpler and faster to prepare.

### Defining a “Quick & Easy” Recipe Category

Using these thresholds, a new **Boolean indicator column** named `quick_easy` was created to identify recipes that meet all three of the following criteria:
- **Quick:** preparation time ≤ 35 minutes  
- **Simple (steps):** ≤ 9 steps  
- **Simple (ingredients):** ≤ 9 ingredients  

If a recipe satisfies all three conditions, `quick_easy = True`; otherwise, it is labeled `False`.

This classification serves as a way to distinguish **simpler, more accessible recipes** from those that are more time-consuming or complex.

By explicitly marking this subset of recipes, later analyses could explore whether *quick and easy* recipes differ meaningfully in their **average user ratings**, **nutritional values**, or **model fairness** — ultimately helping to reveal how convenience influences perceived recipe quality.

Below is the final `recipe_ratings` dataframe (with only relevant columns depicted) to be used for the rest of the analysis:

| name                                 |   minutes |   n_steps | description                                                                                                                                                                                                                                                                                                                                                                       |   n_ingredients |   rating |   avg_rating |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fats |   carbohydrates | quick_easy   |
|:-------------------------------------|----------:|----------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|---------:|-------------:|-----------:|------------:|--------:|---------:|----------:|-----------------:|----------------:|:-------------|
| 1 brownies in the world    best ever |        40 |        10 | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |               9 |        4 |            4 |      138.4 |          10 |      50 |        3 |         3 |               19 |               6 | False        |
| 1 in canada chocolate chip cookies   |        45 |        12 | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |              11 |        5 |            5 |      595.1 |          46 |     211 |       22 |        13 |               51 |              26 | False        |
| 412 broccoli casserole               |        40 |         6 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |               9 |        5 |            5 |      194.8 |          20 |       6 |       32 |        22 |               36 |               3 | False        |
| 412 broccoli casserole               |        40 |         6 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |               9 |        5 |            5 |      194.8 |          20 |       6 |       32 |        22 |               36 |               3 | False        |
| 412 broccoli casserole               |        40 |         6 | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |               9 |        5 |            5 |      194.8 |          20 |       6 |       32 |        22 |               36 |               3 | False        |


Let's look at the proportion of `quick_easy` recipes in our complete dataframe: 

| quick_easy   |   proportion |
|:-------------|-------------:|
| False        |     0.708701 |
| True         |     0.291299 |


### Distribution of "Quick & Easy" Recipes

After defining the `quick_easy` indicator using data-driven thresholds (≤ 35 minutes, ≤ 9 steps, ≤ 9 ingredients), we found that approximately **30% of recipes** meet these criteria, while **70% do not**.

This distribution indicates that:
- A substantial minority of recipes can be described as *Quick & Easy*, aligning with the right-skewed distributions seen in the EDA.  
- Most recipes are moderately or highly complex in preparation time, steps, or ingredients, suggesting that the dataset represents a wide range of cooking styles—from simple weekday meals to more involved dishes.

The 70/30 split confirms that the `quick_easy` feature effectively distinguishes a meaningful subgroup of recipes rather than labeling all recipes as simple.  

This distinction will be valuable in later analyses examining whether these simpler recipes receive higher ratings.

### Univariate Analysis

For this analysis, I looked at the distribution of average recipe rating, `avg_rating`, across all recipe types. Below, the distribution is heavily left-skewed, indicating that most recipes receive very high user ratings. The majority of ratings cluster tightly between 4.0 and 5.0, suggesting that users tend to rate recipes positively, with few low-scoring outliers.

<iframe
  src="images/img4.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Below, the histogram shows the distribution of recipe calorie content, capped at 3,000 calories to align with the recommended daily intake for an average adult. Most recipes fall well below this threshold, clustering heavily between 0 and 800 calories, indicating that the majority of dishes are relatively moderate in energy content. Only a small fraction of recipes approach or exceed 2,000 calories, suggesting that high-calorie recipes are rare outliers in the dataset. This cutoff provides a clearer view of the overall calorie distribution by reducing the visual skew caused by extreme values, allowing us to interpret nutritional trends more accurately.

<iframe
  src="images/img5.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis

Next, for this analysis, I examined the distribution of the average recipe rating and calories conditional on whether the recipe was labeled "Quick and Easy", `quick_easy = True` or labeled "Other Recipe", `quick_easy = False`. 

The first density plot compares the distributions of **average ratings** between *Quick & Easy* and *Other Recipes*. Both groups show a strong right-skew, with most ratings concentrated between **4.0 and 5.0**, indicating that users generally rate recipes positively regardless of preparation complexity. However, *Quick & Easy* recipes have a slightly higher peak near 5.0, suggesting that simpler, faster recipes may be marginally more appreciated by users.

<iframe
  src="images/imgavg_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The second plot compares the distributions of **calorie content** across the same two groups. Both distributions are heavily right-skewed, but *Quick & Easy* recipes show a sharper concentration near the lower end, indicating that these recipes tend to be lighter and less calorically dense. In contrast, *Other Recipes* exhibit a longer tail, implying that more complex recipes often involve richer or more calorie-heavy ingredients.

<iframe
  src="images/imgcalories.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Overall, these trends reinforce the idea that *Quick & Easy* recipes are not only simpler and faster but also **leaner** while maintaining **similarly high user satisfaction**.

### Interesting Aggregates

In this section, I created the pivot table below that compares *Quick & Easy* recipes to other recipes across key feedback and complexity metrics.  

| quick_easy   |   avg_rating |   minutes |   n_ingredients |   recipe_count |
|:-------------|-------------:|----------:|----------------:|---------------:|
| False        |         4.35 |     80.28 |           10.3  |         165043 |
| True         |         4.46 |     16.01 |            6.08 |          67838 |

On average, **Quick & Easy recipes receive slightly higher user ratings** while requiring **significantly less preparation time and fewer ingredients**.

For example, the mean cooking time and ingredient count are both markedly lower for *Quick & Easy* dishes, confirming that this category successfully captures simpler, more efficient recipes.  

Interestingly, although *Quick & Easy* recipes represent a smaller share of the dataset (fewer total entries), they maintain **comparable or better satisfaction levels**. This suggests that users appreciate recipes that balance convenience and quality—favoring dishes that deliver good results without excessive time or ingredient demands.  

Overall, the pivot table reinforces the finding that **simplicity correlates with positive user feedback**, highlighting *Quick & Easy* recipes as both accessible and well-received.

## Assessment of Missingness

Let's look at the `recipe_ratings` where three columns: `description`, `rating`, `review` had missing values. Thus, an assessment of missingness will be conducted below to determine the missingness mechanism for `description`.

### Assessing NMAR Missingness

The `description` column in the `recipe_ratings` dataset appears to contain missing values that may be **Not Missing At Random (NMAR)**.  

This is because the likelihood of a missing description could depend on the content itself—for instance, recipes that are simple or self-explanatory might be less likely to include written descriptions, whereas more complex or unique recipes may have detailed ones. If this is the case, the missingness depends on the unobserved variable (the *description text*), which is a defining characteristic of NMAR data.  

To better understand this missingness and potentially reclassify it as **Missing At Random (MAR)**, we would need additional information—such as the recipe author’s activity level, experience, or submission history—to see whether description completeness is influenced by user behavior rather than the unseen content itself.

### MAR Analysis

The missingness of the `recipe_ratings`'s `description` column is not due to the content of the description itself (i.e., whether the recipe is good or bad), but rather depends on other observed variables in the dataset, such as: either `minutes` or `avg_rating`. I tested whether missingness in description depends on `minutes` or `avg_rating` via a permutation test.

### Missingness Dependency (`minutes`)

I first examined whether the missingness of the `description` column in `recipe_ratings` depends on the `minutes` required for each recipe.

> **Null Hypothesis (H₀):**  
> The distribution of `minutes` is the same for recipes with missing and non-missing descriptions (MCAR).  
>
> **Alternative Hypothesis (H₁):**  
> The distribution of `minutes` is different for recipes with missing descriptions compared to those with non-missing descriptions.

**Test Statistic:** <code>x̄<sub>missing</sub> − x̄<sub>not missing</sub></code>

**Significance Level (α):** 0.05

<iframe src="images/kde_minutes.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="images/null_minutes.html" width="800" height="600" frameborder="0"></iframe>

### Missingness Dependency (`avg_rating`)

We examined whether the missingness of the `description` column in `recipe_ratings` depends on the `avg_rating` of each recipe.

> **Null Hypothesis (H₀):**  
> The distribution of `avg_rating` is the same for recipes with missing and non-missing descriptions.  
>
> **Alternative Hypothesis (H₁):**  
> The distribution of `avg_rating` is different between recipes with missing and non-missing descriptions.

**Test Statistic:** <code>x̄<sub>missing</sub> − x̄<sub>not missing</sub></code>

**Significance Level (α):** 0.05

<iframe src="images/kde_avg_rating.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="images/null_avg_rating.html" width="800" height="600" frameborder="0"></iframe>

The permutation tests reveal contrasting patterns in how missingness of the `description` column relates to other recipe attributes.  

For **`minutes`**, the observed difference in means between recipes with and without missing descriptions was small, and the p-value was high (*p* ≈ 0.31). This indicates that we **fail to reject the null hypothesis**, suggesting that the likelihood of a missing description is independent of the time required to prepare a recipe.  

This pattern is consistent with a **Missing Completely at Random (MCAR)** mechanism.  

In contrast, for **`avg_rating`**, the observed difference was more pronounced, and the permutation test yielded a low p-value (*p* ≈ 0.002). We therefore **reject the null hypothesis** and conclude that the missingness of `description` is **dependent on `avg_rating`**.  

Overall, these results indicate that while `minutes` is unrelated to missingness, `avg_rating` exhibits a **Missing At Random (MAR)** relationship with `description`.

## Hypothesis Testing

### Rating Differences Between "Quick & Easy" and Other Recipes

Understanding how recipe simplicity relates to user satisfaction is important for both recipe creators and online platforms. Recipes that are faster to prepare and use fewer ingredients may appeal to users who value convenience, while more time-intensive dishes might be preferred by users seeking complexity or richer flavor.  

To test this relationship, we use the data-driven **`quick_easy`** indicator, which identifies recipes that are both *quick to make* (≤ 35 minutes) and *simple to follow* (≤ 9 steps and ≤ 9 ingredients).

**Question:**  

>Do “Quick & Easy” recipes receive significantly different average ratings than more complex recipes?

This analysis helps reveal whether users tend to reward convenience and efficiency with higher satisfaction scores or whether longer, more involved recipes are rated more favorably.

**Hypotheses:**

> **Null Hypothesis (H₀):**  
> The distribution of `avg_rating` is the same for “Quick & Easy” and non–“Quick & Easy” recipes.  
>
> **Alternative Hypothesis (H₁):**  
> “Quick & Easy” recipes have higher average ratings than non–“Quick & Easy” recipes.

**Test Statistic:** <code>x̄<sub>Quick & Easy</sub> − x̄<sub>Other Recipes</sub></code>

**Significance Level (α):** 0.05

<iframe src="images/hyp_kde.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="images/hyp_null.html" width="800" height="600" frameborder="0"></iframe>

### Analysis of Hypothesis Test Results

The permutation test reveals a strong and statistically significant difference in user ratings between *Quick & Easy* and *Other* recipes.  
The observed difference in mean ratings <code>x̄<sub>Quick & Easy</sub> − x̄<sub>Other</sub></code> = 0.1133 is positive, indicating that *Quick & Easy* recipes, on average, receive higher ratings.  

The p-value from the one-sided permutation test is **p = 0.0000**, which is well below the significance threshold of 0.05. This means we **reject the null hypothesis** and conclude that the difference in average ratings is unlikely to have occurred by chance.  

In practical terms, users appear to **favor simpler and faster recipes**, suggesting that convenience plays a meaningful role in how positively recipes are rated. This finding aligns with the broader trend observed in exploratory analysis — recipes that are easier to prepare tend to attract higher user satisfaction, perhaps due to their accessibility, reliability, or lower cognitive effort required.


## Framing a Prediction Problem

### Estimating Recipe Ratings from Numeric Attributes

In this section, I aim to predict the **average rating (`avg_rating`)** of a recipe based on its **quantitative features**, such as preparation time, nutritional values, and recipe complexity measures. This prediction problem aligns with the broader goal of understanding what factors drive user satisfaction and how recipe characteristics relate to perceived quality.

Since `avg_rating` is a **continuous numerical variable**, this task is framed as a **regression problem**.  
The goal is to model the relationship between recipe-level predictors (e.g., `minutes`, `n_steps`, `n_ingredients`, `calories`, `protein`, `carbohydrates`, etc.) and the average rating users assign to each recipe.

**Response Variable:**  
`avg_rating` — the mean rating assigned to a recipe by all users.

**Predictor Variables (Features):**  
`minutes`, `n_steps`, `n_ingredients`, `calories`, `protein`, `carbohydrates`, `sugar`, `sodium`, `total_fat`, `saturated_fats`

**Type of Problem:**  
Regression — predicting a continuous value (`avg_rating`).

**Evaluation Metric:**  
We will evaluate model performance using **Root Mean Squared Error (RMSE)**, as it penalizes larger errors more strongly and is a common metric for continuous prediction tasks.  

Alternatively, **R² (coefficient of determination)** may also be used to assess how well the model explains variability in recipe ratings.

**Goal:**  
By training this model, we hope to uncover which recipe attributes most strongly influence user ratings and assess how accurately these quantitative factors can predict overall satisfaction.

## Baseline Model

### Multiple Linear Regression

To establish a baseline for predicting `avg_rating`, we fit a **Multiple Linear Regression (MLR)** model using the recipe-level quantitative features identified earlier.  

This model serves as a simple yet interpretable benchmark that captures linear relationships between recipe characteristics (e.g., preparation time, ingredient count, nutritional values) and user satisfaction.

The baseline model assumes that each feature contributes additively to the average rating and that the effect of each variable is constant across all recipes.  

While this assumption may oversimplify real-world patterns, it provides an important reference point for evaluating whether more complex models (e.g., polynomial regression or tree-based methods) improve prediction accuracy.

**Model Specification:**

>`avg_rating = β₀ + β₁(minutes) + β₂(n_steps) + β₃(n_ingredients) + β₄(calories) + β₅(total_fat) + β₆(sugar) + β₇(sodium) + β₈(protein) + β₉(saturated_fats) + β₁₀(carbohydrates) + ε`

**Goal:**  
Assess how strongly each numeric attribute influences user ratings and establish a baseline error metric (RMSE, R²) for comparison with future models.

### Baseline Model Results and Evaluation

The baseline model is a **Multiple Linear Regression** that predicts a recipe’s average user rating (`avg_rating`) based solely on **quantitative (numeric)** recipe attributes. These features capture measures of time, procedural complexity, and nutritional composition that may influence how users rate recipes.

**Features included:**
- **Quantitative (10):** `minutes`, `n_steps`, `n_ingredients`, `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fats`, `carbohydrates`  

These continuous variables represent core aspects of a recipe’s structure and nutritional profile. No categorical or boolean features were included in this baseline version.

All numeric features were passed through unchanged (`"passthrough"`) without scaling or transformation, and the model was trained using a standard `LinearRegression` estimator within a single `sklearn` `Pipeline`.

---

**Model Performance**
- **Root Mean Squared Error (RMSE):** 0.8459  
- **R² (Coefficient of Determination):** 0.0087  

---

**Interpretation:**  
The low R² value (≈0.01) indicates that the baseline model explains less than 1% of the variance in recipe ratings, suggesting that numeric features alone provide **limited predictive power** for user satisfaction. The relatively high RMSE (≈0.85), given that ratings range from 0 to 5, further suggests that the model’s predictions deviate substantially from observed ratings.  

Overall, this baseline model serves as a **minimal benchmark**, demonstrating that while numeric recipe attributes such as preparation time and nutritional content may correlate weakly with ratings, they are **insufficient on their own** to model subjective user preferences. Future models should incorporate additional contextual or categorical information—such as recipe type, description text, or contributor metadata—to better explain variations in user ratings.

## Final Model

### Random Forest with Engineered Complexity and Nutrient Features

To improve upon the baseline multiple linear regression model (which used only raw numeric features),  

I trained a **Random Forest Regressor** that incorporates two engineered features designed to capture recipe complexity and nutritional balance more directly.

**Base quantitative features (10):**

- `minutes`, `n_steps`, `n_ingredients`
- `calories`, `total_fat`, `sugar`, `sodium`
- `protein`, `saturated_fats`, `carbohydrates`

These describe time, procedural complexity, and the nutritional profile of each recipe.

**Engineered features (2):**

1. **Interaction term: `F_complexity = n_steps × minutes`**  
This feature measures how time and procedural complexity interact. Recipes that are both long and involve many steps will have very high `F_complexity`, reflecting a labor-intensive process that may negatively impact user satisfaction.

2. **Ratio term: `F_nutrient_ratio = protein / (carbohydrates + ε)`**  
   This feature captures the balance between protein and carbohydrates, a key aspect of nutritional “type.”  
   - High values indicate high-protein, lower-carb recipes (e.g., steak and vegetables).  
   - Low values indicate carb-heavy dishes (e.g., pasta, desserts).

    A small constant ε is added to the denominator to avoid division by zero.

Both engineered features are created **inside the sklearn Pipeline** via a custom transformer, so they are always computed consistently during training and testing.

**Hyperparameter Tuning**

To further improve performance and control model complexity, I tuned the following Random Forest hyperparameters using `GridSearchCV` with 5-fold cross-validation:

- `n_estimators`: number of trees in the forest (`[150]`)  
- `max_depth`: maximum depth of each tree (`[20, 25, None]`)  
- `max_features`: number of features considered when looking for the best split (`['sqrt']`)

All preprocessing (feature engineering) and model training are implemented in a single sklearn `Pipeline`, and the best model is selected based on validation R².

### Final Model Results and Evaluation

#### **Model and Hyperparameter Tuning**
The Random Forest model was optimized using **5-fold cross-validation** via `GridSearchCV`.  
The following hyperparameters were tuned:
- `max_depth ∈ [10, 15, 20, 25]` – controls tree depth and model complexity.  
- `max_features = "sqrt"` – ensures that only a subset of features is considered at each split, reducing overfitting.  
- `n_estimators = 150` – fixed to balance accuracy and runtime.

The **best configuration** selected by the grid search was:
                            
                `{'model__max_depth': None, 'model__max_features': 'sqrt', 'model__n_estimators': 150}`

---

**Model Performance**
- **Root Mean Squared Error (RMSE):** 0.5899  
- **R² (Coefficient of Determination):** 0.5179  

---

<iframe src="images/base_vs_final.html" width="800" height="600" frameborder="0"></iframe>


**Interpretation:**  

The final Random Forest model achieved a **Root Mean Squared Error (RMSE)** of **0.5899**, a significant improvement from the baseline RMSE of 0.8459. The **R² score** increased substantially from **0.0087** to **0.5179**, indicating that the model now explains approximately 51.8% of the variance in recipe ratings compared to less than 1% previously.

This improvement reflects how the engineered features captured latent relationships in the data:

>`F_Complexity` quantifies overall recipe effort — complex, time-intensive recipes tend to have lower ratings due to user fatigue or difficulty.

>`F_NutrientRatio` captures nutritional appeal — users tend to favor balanced, protein-rich recipes.

From the perspective of the data-generating process, these factors mirror real-world user behavior: satisfaction is influenced by both ease of preparation and nutritional balance. The Random Forest’s nonlinear nature also allows it to model these effects more flexibly than the linear regression baseline.

**Conclusion:**

The final Random Forest Regressor demonstrates a meaningful performance improvement over the baseline model by incorporating domain-informed feature engineering. While overall predictive power remains moderate—reflecting the inherent noise in user ratings—the model effectively captures complex, interpretable patterns related to recipe effort and nutritional content, validating the usefulness of the engineered features.

## Fairness Analysis

To test for fairness of our final model on different groups, I used the *trained Random Forest Regressor* (which excluded `quick_easy` as a feature) and computed the RMSE for each group:

- RMSE (Quick & Easy): 0.3493   
- RMSE (Other Recipes): 0.3858 

**Interpretation:**  
The model achieves **lower error for Quick & Easy recipes**, suggesting it predicts user ratings for these faster, simpler recipes more accurately than for more complex ones. This difference implies that the model generalizes better for straightforward, low-effort dishes, possibly because such recipes exhibit more consistent patterns in numeric features (e.g., minutes, steps, and calories).  

In contrast, the higher RMSE for **Other Recipes** indicates slightly reduced accuracy, potentially due to greater variability in ingredient counts, preparation times, or nutrient composition that the current numeric-only feature set does not fully capture.  

Overall, the model shows a **small but notable fairness gap** in favor of the Quick & Easy group, though both RMSE values remain within a reasonable range, indicating broadly consistent performance across recipe types.

### Model Performance Across Recipe Categories

To assess whether the final model performs fairly across recipe types, we conducted a **fairness analysis** comparing prediction errors for *Quick & Easy* recipes and *Other* recipes.

Because this is a regression problem, we evaluate fairness using **Root Mean Squared Error (RMSE)** instead of classification metrics.  

A large discrepancy in RMSE between the two groups would suggest the model predicts one group’s ratings more accurately than the other’s.

**Groups Tested:**
- **Group X (Quick & Easy):** Recipes labeled as quick and simple to make  
- **Group Y (Other Recipes):** All remaining recipes

**Hypotheses:**
>**Null Hypothesis (H₀):** The model is fair — its prediction error (RMSE) is the same for both groups, and any observed difference is due to chance.  
>**Alternative Hypothesis (H₁):** The model is unfair — its RMSE differs significantly between the two groups.  

**Test Statistic:**  
Difference in RMSE between groups (Quick & Easy − Other Recipes)

**Significance Level:** α = 0.05

<iframe src="images/fair_kde.html" width="800" height="600" frameborder="0"></iframe>

<iframe src="images/fair_null.html" width="800" height="600" frameborder="0"></iframe>

### Fairness Analysis Results: Quick & Easy vs. Other Recipes

#### Results
- **Observed RMSE difference:** −0.0365  
- **Permutation p-value:** 0.0050  

The **negative observed difference** indicates that the model’s RMSE is **lower for Quick & Easy recipes**, meaning it predicts these simpler, faster recipes more accurately than more complex ones.  
The **p-value of 0.0050** suggests this difference is statistically significant at the 0.05 level, providing strong evidence that the model performs differently across the two groups.

---

#### Interpretation
The fairness analysis shows that the final Random Forest model exhibits a **performance disparity favoring Quick & Easy recipes**.  

This implies that the model generalizes better to **low-effort, straightforward recipes**, which tend to have fewer ingredients, shorter preparation times, and simpler nutrient profiles. These characteristics align well with the numeric features used during training, allowing the model to capture their patterns more effectively.

In contrast, the slightly **higher RMSE for Other Recipes** suggests that the model struggles to predict ratings for **longer, more complex dishes**. These recipes likely involve more qualitative nuances—such as taste complexity, ingredient interactions, or cultural factors—that are not captured by the current numeric features.

---

#### Conclusion
The model demonstrates **a mild but measurable fairness concern**: it performs significantly better for the *Quick & Easy* subset than for more complex recipes. While this bias does not invalidate the model, it indicates that **simpler recipes are systematically easier for the model to predict accurately**.  

Future iterations could mitigate this disparity by incorporating **richer, non-numeric features**—for example, textual embeddings from recipe descriptions or categorical encodings for cuisine type—to better represent the complexity of diverse recipes and improve fairness across groups.




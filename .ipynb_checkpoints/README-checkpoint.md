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

The merge was performed on the shared key column, `recipe_id`, using an **inner join** to ensure that only recipes with at least one corresponding rating were included in the resulting dataset.

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

Below is the final `recipe_ratings` dataframe to be used for the rest of the analysis:

<iframe src="images/recipe_ratings.html" width="800" height="300" frameborder="0"></iframe>

Let's look at the proportion of `quick_easy` recipes in our complete dataframe: 

<iframe src="images/prop_quick_easy.html" width="800" height="300" frameborder="0"></iframe>

### Distribution of "Quick & Easy" Recipes

After defining the `quick_easy` indicator using data-driven thresholds (≤ 35 minutes, ≤ 9 steps, ≤ 9 ingredients),  
we found that approximately **30% of recipes** meet these criteria, while **70% do not**.

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


I also looked at the distribution of calories across all recipes. Below, the histogram shows the distribution of recipe calorie content, capped at 3,000 calories to align with the recommended daily intake for an average adult. Most recipes fall well below this threshold, clustering heavily between 0 and 800 calories, indicating that the majority of dishes are relatively moderate in energy content. Only a small fraction of recipes approach or exceed 2,000 calories, suggesting that high-calorie recipes are rare outliers in the dataset. This cutoff provides a clearer view of the overall calorie distribution by reducing the visual skew caused by extreme values, allowing us to interpret nutritional trends more accurately.

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

<iframe src="images/int_agg.html" width="800" height="300" frameborder="0"></iframe>

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

The missingness of the `recipe_ratings`'s `description` column is not due to the content of the description itself (i.e., whether the recipe is good or bad),     but rather depends on other observed variables in the dataset, such as: either `minutes` or `avg_rating`. I tested whether missingness in description depends on `minutes` or `avg_rating` via a permutation test.

### Missingness Dependency (`minutes`)

We first examined whether the missingness of the `description` column in `recipe_ratings` depends on the `minutes` required for each recipe.

> **Null Hypothesis (H₀):**  
> The distribution of `minutes` is the same for recipes with missing and non-missing descriptions (MCAR).  
>
> **Alternative Hypothesis (H₁):**  
> The distribution of `minutes` is different for recipes with missing descriptions compared to those with non-missing descriptions.

**Test Statistic:**  
$$ \bar{x}_{\text{missing}} - \bar{x}_{\text{not missing}} $$

**Significance Level (α):** 0.05





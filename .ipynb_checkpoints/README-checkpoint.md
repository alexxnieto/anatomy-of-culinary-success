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

The final cleaned and merged dataset, named **`recipe_ratings`**, contains:
- Recipe-level information (`minutes`, `n_steps`, `n_ingredients`, etc.)
- Nutritional composition from the expanded nutrition columns
- Descriptive metadata (e.g., `description`, `tags`)
- The computed **average recipe rating (`avg_rating`)**

This resulting DataFrame served as the **foundation for all exploratory analysis, hypothesis testing, and predictive modeling** conducted in subsequent steps.





print(recipe_ratings[['name', 'minutes', 'n_steps', 'description', 'n_ingredients', 'rating', 'avg_rating', 'calories', 'total_fat', 'sugar', 'sodium', 'protein', 'saturated_fats', 'carbohydrates', 'quick_easy]].head().to_markdown(index=False))



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
    -  The relevant columns from `recipes`: `name`, `id`, `minutes`, `nutrition`,`n_steps`, `n_ingredients`.
    | Column             | Description                                                                                                                                                                                       |
    | :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
    | `'name'`           | Recipe name                                                                                                                                                                                       |
    | `'id'`             | Recipe ID                                                                                                                                                                                         |
    | `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
    | `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
    | `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
    | `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
    | `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
    | `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
    | `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
    | `'description'`    | User-provided description                                                                                                                                                                         |
    | `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
    | `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |

- `ratings`
    - Rows and Columns: 731927 rows (recipes) and 5 columns
    - The relevant columns from `ratings`: `recipe_id`, `rating`.



***

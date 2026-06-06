# Are Healthier Recipes Rated Differently on Food.com?

Project for DSC 80 at UCSD

By Jay Ma

## Introduction

In this project, I examined a dataset of recipes and ratings from Food.com. Food.com is a recipe-sharing website where users can submit recipes and rate recipes posted by others. This dataset contains recipe-level information, such as preparation time, ingredients, nutrition, and recipe tags, as well as user interaction data, such as ratings and reviews.

The data originally comes in two CSV files. `RAW_recipes.csv` contains recipe-level information, such as recipe name, preparation time, tags, nutrition, steps, ingredients, and number of ingredients. `RAW_interactions.csv` contains user-submitted interactions with recipes, including user IDs, recipe IDs, dates, ratings, and review text. I use these two files together because the recipe file contains the nutritional and preparation information, while the interactions file contains the ratings needed to calculate each recipe’s average rating.

The main question I explore is: **Are healthier recipes rated differently from less healthy recipes on Food.com?**

This question is interesting because online ratings can influence which recipes users choose to make. If healthier recipes tend to receive different ratings than less healthy recipes, this could reveal something about how users respond to nutritional characteristics when evaluating food. In this project, I focus especially on protein and sugar as indicators of healthiness.

The original recipe dataset contains 83,782 recipes and 12 columns. The original interactions dataset contains 731,927 user interactions and 5 columns. After cleaning and feature extraction, I focus on the following columns for my analysis:

| Column              | Description                                                                                                                                                                                                                     |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`              | Recipe name                                                                                                                                                                                                                     |
| `id`                | Recipe ID                                                                                                                                                                                                                       |
| `minutes`           | Minutes to prepare the recipe                                                                                                                                                                                                   |
| `submitted`         | Date the recipe was submitted                                                                                                                                                                                                   |
| `tags`              | Food.com tags for the recipe                                                                                                                                                                                                    |
| `nutrition`         | Nutrition information stored as a list in the form `[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]`. Calories is a raw count, while the remaining nutrition values are percentages of daily value. |
| `n_steps`           | Number of steps in the recipe                                                                                                                                                                                                   |
| `ingredients`       | Ingredients used in the recipe                                                                                                                                                                                                  |
| `n_ingredients`     | Number of ingredients in the recipe                                                                                                                                                                                             |
| `rating`            | Rating given by a user in the interactions dataset                                                                                                                                                                              |
| `avg_rating`        | Average rating for each recipe after replacing ratings of 0 with missing values                                                                                                                                                 |
| `calories`          | Calories in the recipe, extracted from `nutrition`                                                                                                                                                                              |
| `sugar_pdv`         | Sugar as a percentage of daily value, extracted from `nutrition`                                                                                                                                                                |
| `protein_pdv`       | Protein as a percentage of daily value, extracted from `nutrition`                                                                                                                                                              |
| `carbohydrates_pdv` | Carbohydrates as a percentage of daily value, extracted from `nutrition`                                                                                                                                                        |

## Data Cleaning and Exploratory Data Analysis

The first step was to clean the data and prepare it for analysis. Since my project focuses on the relationship between nutrition and ratings, I needed to combine the recipe information with the user rating information.

### Cleaning

The data originally came in two CSV files: `RAW_recipes.csv` and `RAW_interactions.csv`. `RAW_recipes.csv` contains recipe-level information, such as the recipe name, preparation time, tags, nutrition, steps, ingredients, and number of ingredients. `RAW_interactions.csv` contains user interactions with recipes, including user IDs, recipe IDs, dates, ratings, and review text.

To combine the two datasets, I performed a left merge using the recipe ID. This kept every recipe from the recipes dataset, even if a recipe did not have a corresponding rating in the interactions dataset.

After merging, I replaced all ratings of 0 with `NaN`. This is reasonable because Food.com ratings are intended to be on a 1-to-5 scale, so a rating of 0 likely represents a missing or unsubmitted rating rather than a true rating. Keeping 0 ratings would artificially lower the average rating of some recipes.

Next, I grouped the merged dataset by recipe ID and calculated the average rating for each recipe. I then added this average rating back to the original recipes dataset as a new column called `avg_rating`. This recipe-level dataset, with one row per recipe and an added `avg_rating` column, is the main dataset I use for the rest of my analysis.

I also cleaned the `nutrition` column. In the raw data, nutrition information is stored as a list in the following order:

`[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]`

To make these values easier to analyze, I separated this list into individual columns:

* `calories`
* `total_fat_pdv`
* `sugar_pdv`
* `sodium_pdv`
* `protein_pdv`
* `saturated_fat_pdv`
* `carbohydrates_pdv`

This cleaned dataset allowed me to compare nutrition values directly with average recipe ratings.

The first few rows of the cleaned dataframe are shown below:

| name                               |     id | minutes | n_steps | n_ingredients | avg_rating | calories | sugar_pdv | protein_pdv |
| ---------------------------------- | -----: | ------: | ------: | ------------: | ---------: | -------: | --------: | ----------: |
| 1 brownies in the world best ever  | 333281 |      40 |      10 |             9 |        4.0 |    138.4 |      50.0 |         3.0 |
| 1 in canada chocolate chip cookies | 453467 |      45 |      12 |            11 |        5.0 |    595.1 |     211.0 |        13.0 |
| 412 broccoli casserole             | 306168 |      40 |       6 |             9 |        5.0 |    194.8 |       6.0 |        22.0 |
| millionaire pound cake             | 286009 |     120 |       7 |             7 |        5.0 |    878.3 |     326.0 |        20.0 |
| 2000 meatloaf                      | 475785 |      90 |      17 |            13 |        5.0 |    267.0 |      12.0 |        29.0 |

### Exploratory Data Analysis

In my exploratory data analysis, I first examined the distributions of important individual variables. Since my main question is about healthiness and ratings, I focused on `avg_rating`, `calories`, and `protein_pdv`.

### Univariate Analysis

First, I looked at the distribution of average recipe ratings.

<iframe
  src="assets/avg_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of `avg_rating` is heavily concentrated near 5 stars. This shows that most recipes in the dataset receive very positive ratings. Because ratings are so skewed toward 5, small differences in average rating may still be meaningful. This is important for my project because I am trying to compare ratings across different types of recipes, even though most ratings are already very high.

Next, I examined the distribution of calories.

<iframe
  src="assets/calories_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of calories is right-skewed. Most recipes have relatively moderate calorie counts, while a smaller number of recipes have very high calorie values. Because the dataset contains extreme nutrition outliers, I used a filtered version of the data for this visualization so the main distribution would be easier to interpret. This plot shows that calories vary substantially across recipes, making calories a useful nutrition feature to consider in later analysis.

I also examined the distribution of protein percentage of daily value.

<iframe
  src="assets/protein_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The distribution of `protein_pdv` is also right-skewed. Most recipes have relatively low to moderate protein values, while fewer recipes have very high protein values. Since protein is one way to describe a recipe’s nutritional profile, this column is important for my analysis of whether healthier recipes are rated differently from less healthy recipes.


### Bivariate Analysis

After looking at individual variables, I examined relationships between nutrition and average rating. Since calorie and protein values are numerical and have wide ranges, I grouped recipes into four quartile-based groups. This makes it easier to compare ratings across recipes with relatively low, medium, and high nutritional values.

First, I grouped recipes into calorie groups and compared their average ratings.

<iframe
  src="assets/rating_by_calorie_group.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The calorie group box plot shows that average ratings are generally high across all calorie groups. The median rating is close to 5 for each group, suggesting that calorie level alone does not strongly separate highly rated recipes from lower-rated recipes. However, lower-rated outliers appear in every calorie group, meaning that recipes with both low and high calorie counts can still receive lower ratings.

Next, I grouped recipes into protein groups and compared their average ratings.

<iframe
  src="assets/rating_by_protein_group.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The protein group box plot shows a similar pattern. Ratings are high across all protein groups, but the mean average rating slightly decreases as protein level increases. This pattern is small, but it suggests that protein content may be related to recipe ratings. Because the visual difference is not very large, I also used grouped summary statistics to examine the pattern more clearly.

### Interesting Aggregates

To summarize the relationship between protein and rating more clearly, I grouped recipes by protein level and calculated summary statistics.

| Protein Group       | Number of Recipes | Mean Average Rating | Median Average Rating | Mean Calories | Mean Sugar PDV |
| ------------------- | ----------------: | ------------------: | --------------------: | ------------: | -------------: |
| Low Protein         |             20248 |              4.6509 |                   5.0 |      158.1851 |        53.1789 |
| Medium-Low Protein  |             19641 |              4.6239 |                   5.0 |      287.1145 |        64.6373 |
| Medium-High Protein |             19811 |              4.6203 |                   5.0 |      416.6272 |        47.0612 |
| High Protein        |             19667 |              4.6068 |                   5.0 |      619.5255 |        39.1060 |

This table shows that all protein groups have a median average rating of 5.0, which confirms that Food.com ratings are generally very high. However, the mean average rating slightly decreases as protein level increases. Low-protein recipes have the highest mean average rating, while high-protein recipes have the lowest mean average rating. The difference is not large, but the pattern suggests that protein content may be related to recipe ratings and is worth investigating further in the hypothesis test.

I also created a pivot table showing the mean average rating for each combination of protein group and calorie group.

| Protein Group       | Low Calories | Medium-Low Calories | Medium-High Calories | High Calories |
| ------------------- | -----------: | ------------------: | -------------------: | ------------: |
| Low Protein         |       4.6473 |              4.6617 |               4.6509 |        4.6501 |
| Medium-Low Protein  |       4.6158 |              4.6190 |               4.6349 |        4.6336 |
| Medium-High Protein |       4.6023 |              4.5967 |               4.6267 |        4.6401 |
| High Protein        |       4.6179 |              4.6134 |               4.5940 |        4.6120 |

The pivot table shows that ratings remain high across nearly all combinations of calorie and protein groups. There is no dramatic rating difference across the table, but the values are not exactly the same. This supports the idea that nutrition may have some relationship with ratings, even though the effect appears small because most recipes are rated highly.


## Assessment of Missingness

I focused my missingness analysis on the `avg_rating` column, which is missing for 2,609 recipes. A recipe has missing `avg_rating` when it has no valid rating after ratings of 0 were replaced with `NaN`.

I do not believe that the missingness of `avg_rating` is clearly NMAR based only on the available data. A recipe may be missing ratings because of observable factors such as the year it was submitted, its nutritional content, or how complex the recipe is. However, missingness could also depend on unobserved factors, such as how many users viewed the recipe page but chose not to rate it. If I had additional data such as page views, clicks, saves, or impressions, I could better explain why some recipes are missing ratings and potentially make the missingness MAR instead of NMAR.

To test whether missingness in `avg_rating` depends on other columns, I performed permutation tests. First, I tested whether missingness in `avg_rating` depends on `protein_pdv`. The test statistic was the absolute difference in mean protein percentage of daily value between recipes with missing and non-missing average ratings. The observed statistic was 1.2873, and the p-value was about 0.173 in 1,000 simulations. Since this p-value is greater than 0.05, I fail to reject the null hypothesis. This suggests that missingness in `avg_rating` does not appear to depend on protein content.

Next, I tested whether missingness in `avg_rating` depends on `submitted_year`. The test statistic was the absolute difference in mean submitted year between recipes with missing and non-missing average ratings. The observed statistic was 0.7297, and the p-value was 0.0 in 1,000 simulations. Since this p-value is less than 0.05, I reject the null hypothesis. This suggests that missingness in `avg_rating` does appear to depend on the year the recipe was submitted.

<iframe
  src="assets/missingness_year_permutation.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The permutation distribution for `submitted_year` shows that the observed difference is far from what would typically occur by random chance if missingness were unrelated to submission year. This makes sense because recipes submitted in different years may have had different amounts of time to receive ratings. Older recipes may have had more time to accumulate ratings, while newer recipes may be more likely to have missing average ratings.


## Hypothesis Testing

For my hypothesis test, I investigated whether healthier recipes are rated differently from less healthy recipes.

I defined **healthier recipes** as recipes with **above-median protein** and **below-median sugar**. I defined **less healthy recipes** as recipes with **below-median protein** and **above-median sugar**.

My hypotheses were:

- **Null hypothesis:** Healthier recipes and less healthy recipes have the same average rating.
- **Alternative hypothesis:** Healthier recipes and less healthy recipes have different average ratings.

My test statistic was the difference in mean average rating between the two groups:

**mean average rating of healthier recipes − mean average rating of less healthy recipes**

The observed difference was **-0.026955**, meaning that healthier recipes had a slightly lower mean average rating than less healthy recipes in the observed data.

I then performed a **permutation test** using **1,000 simulations**. The resulting **p-value was 0.0** (in other words, none of the 1,000 simulated differences were as extreme as the observed difference). Because this p-value is less than **0.05**, I rejected the null hypothesis.

This suggests that healthier and less healthy recipes have **different average ratings** in this dataset. However, the size of the observed difference is quite small, so the practical difference between the two groups may not be very large. In addition, this result does **not** prove that healthiness causes different ratings; it only shows that the observed difference would be unlikely if the two groups truly had the same average rating.

<iframe
  src="assets/healthiness_hypothesis_test.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The histogram above shows the empirical distribution of the permutation test statistic under the null hypothesis. The red vertical line marks the observed difference in mean average rating. Because the observed statistic falls in the extreme tail of the distribution, it provides evidence against the null hypothesis and supports the conclusion that the two groups are rated differently.

## Framing a Prediction Problem

For my prediction problem, I will predict whether a recipe is **highly rated**. I define a highly rated recipe as one with an `avg_rating` equal to 5.0, and a not highly rated recipe as one with an `avg_rating` below 5.0. This is a **binary classification problem** because the response variable has two possible classes: highly rated or not highly rated.

I chose this prediction problem because the earlier parts of my project focused on the relationship between recipe nutrition and user ratings. The distribution of `avg_rating` is heavily concentrated near 5 stars, so predicting the exact numerical rating may not be very meaningful. Instead, predicting whether a recipe receives a perfect average rating allows me to keep the focus on recipe ratings while creating a clearer classification task.

The response variable is `high_rating`, which is `True` if a recipe’s `avg_rating` is equal to 5.0 and `False` otherwise.

At the time of prediction, I would know recipe-level information such as nutrition values, preparation time, number of steps, and number of ingredients. I would not use individual user ratings or review text as features because those are only available after users interact with the recipe.

I will evaluate my model using **F1-score**. I chose F1-score instead of accuracy because the classes are imbalanced: many recipes have very high ratings, especially ratings close to or equal to 5. Accuracy alone could be misleading if the model mostly predicts the majority class. F1-score is more appropriate because it balances precision and recall when evaluating how well the model identifies highly rated recipes.

## Baseline Model

For my baseline model, I used a **logistic regression classifier** to predict `high_rating`, which indicates whether a recipe has an average rating of exactly 5.0. I chose logistic regression because it is simple, interpretable, and appropriate for a binary classification problem.

The model used nine quantitative recipe-level features:

* `calories`
* `sugar_pdv`
* `protein_pdv`
* `carbohydrates_pdv`
* `total_fat_pdv`
* `sodium_pdv`
* `minutes`
* `n_steps`
* `n_ingredients`

All nine features are quantitative. There are **0 ordinal features** and **0 nominal features** in this baseline model, so no categorical encoding was needed. I standardized the numerical features using `StandardScaler` and trained the logistic regression model using a single `sklearn` pipeline.

To evaluate the model’s ability to generalize to unseen data, I used a train-test split and measured performance using accuracy and F1-score. The baseline model had a training accuracy of **0.5882** and a test accuracy of **0.5863**. It had a training F1-score of **0.7403** and a test F1-score of **0.7387**.

The train and test scores are very similar, which suggests that the model is not severely overfitting. However, I would not consider this baseline model very strong. Since about **58.9%** of the recipes in the modeling dataset are highly rated, the model’s accuracy is close to what could be achieved by often predicting the majority class. The F1-score is higher because the positive class, highly rated recipes, is the majority class. Overall, this model is a useful baseline, but it leaves room for improvement through feature engineering and more flexible modeling methods.

## Final Model

For my final model, I used Ridge regression with both numerical recipe features and text-based features from recipe names and tags. This model improves on the baseline because it includes engineered features and recipe text information that may capture recipe categories not represented by nutrition values alone.

I added four engineered features:

* `log_minutes`: the log-transformed preparation time
* `log_calories`: the log-transformed calorie value
* `health_score`: protein percentage of daily value minus sugar percentage of daily value
* `complexity_score`: number of steps plus number of ingredients

I created `log_minutes` and `log_calories` because both preparation time and calories are right-skewed and contain extreme values. Taking the log reduces the influence of very large values. I created `health_score` because my project focuses on whether healthier recipes are rated differently, and this feature combines two important nutrition variables: protein and sugar. I created `complexity_score` because recipes with more steps and more ingredients may be more complicated to make, which could affect user ratings.

In addition to these numerical features, I used TF-IDF vectorization on the `name` and `tags` columns. These text features can capture information about recipe categories, such as desserts, salads, soups, quick meals, or healthy recipes. This information may help predict ratings beyond what is available from nutrition columns alone.

I used GridSearchCV to tune the Ridge regression regularization parameter `alpha`. The best value was `alpha = 100`.

The final model had a training RMSE of 0.6322 and a test RMSE of 0.6291. Compared to the baseline model’s test RMSE of about 0.6360, the final model slightly improved performance on unseen data.

Although the improvement is small, the final model is more thoughtful than the baseline because it uses engineered features and text-based information from recipe names and tags. The small improvement may be partly due to the fact that average ratings are highly concentrated near 5, making it difficult for any model to predict meaningful differences between recipes.

## Fairness Analysis

This section will analyze whether the final model performs differently across two groups of recipes.

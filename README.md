# Recipe Rating Predictions Based on MEAT Inclusion
Authors: Evelyn Zhang & Haowen Zhang

## Overview
In this project, we decided to explore the relationship between recipe ratings and the nutrition information.

## Introduction
Over the past few years there has been a major change in dietary preferences and more people are paying attention to how the composition of recipes influences consumer perceptions and ratings. Specifically, the inclusion of meat in recipes has become a focal point, as trends toward plant-based eating has been popularized with our increasing health-conscious and environmental awareness.

Therefore, we are interested in investigating relationship between the meat inclusion of recipes and their user ratings. In particular, **we wonder if recipes with meat tagged will get lower ratings due to consumer’s expectations for healthier, more sustainable eating options**. 

The dataset we are using contains recipes and ratings from [food.com](food.com). You could download the two csv files [here](https://drive.google.com/drive/folders/15965zA-g3QbfFqEIzLzk5ICPn4yGzHmr?usp=sharing).

1. `RAW_recipes.csv` contains all recipes
2. `RAW_interactions.csv` contains all reviews and ratings submitted for recipes in `RAW_interactions.csv`


For the first `Recipe` Dataset, there are a total of 83782 rows and 13 columns containing all the recipes' information:

| **Column**       | **Description**                                                                                                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`           | Recipe name                                                                                                                                                                                         |
| `id`             | Recipe ID                                                                                                                                                                                           |
| `minutes`        | Minutes to prepare recipe                                                                                                                                                                           |
| `contributor_id` | User ID who submitted this recipe                                                                                                                                                                   |
| `submitted`      | Date recipe was submitted                                                                                                                                                                           |
| `tags`           | Food.com tags for recipe                                                                                                                                                                            |
| `nutrition`      | Nutrition information in the form `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`; PDV stands for “percentage of daily value” |
| `n_steps`        | Number of steps in recipe                                                                                                                                                                           |
| `steps`          | Text for recipe steps, in order                                                                                                                                                                     |
| `description`    | User-provided description                                                                                                                                                                           |
| `ingredients`    | Ingredients of the recipe                                                                                                                                                                           |
| `n_ingredients`  | Number of ingredients for the recipe                                                                                                                                                                |

The second dataset `interactions` contains 731927 rows and 5 columns, each row corresponds to a review to a recipe in `recipe`:

| **Column**  | **Description**     |
| ----------- | ------------------- |
| `user_id`   | User ID             |
| `recipe_id` | Recipe ID           |
| `date`      | Date of interaction |
| `rating`    | Rating given        |
| `review`    | Review text         |

**With the dataset provided above, we are planning on determining whether the recipe with meat are rated the same as those without.** Our methodology is to check the `tags` column for a given row/recipe and create a new column `contains_meat` with bool values: `True` if the specific `meat` tag is included in the corresponding `tags` column. After operating some data cleaning below, we will focus on `contains_meat`, `rating`, and the `average_rating` column, which contains the users' average rating for that specific row/recipe. 

In addition to investigate the relationship bewteen the two, we are also interested in determining the missingness of `rating` and prediction/categorization of `rating` based on relevamt recipe information like `contains_meat`, `minutes`, and so forth.

We will not use `review` column in this project as it is irrelevant for our current topic of investigation.

## Data Clearning and Exploratory Data Analysis
We used the following steps to clean our data:

1. Left merge `recipe` and `interactions` dataset. 

2. Check the data type for resulting columns:

   - This step allows us to evaluate following cleaning operations
   - | **Column**       | **Dtype** |
     | ---------------- | --------- |
     | `name`           | object    |
     | `id`             | int64     |
     | `minutes`        | int64     |
     | `contributor_id` | int64     |
     | `submitted`      | object    |
     | `tags`           | object    |
     | `nutrition`      | object    |
     | `n_steps`        | int64     |
     | `steps`          | object    |
     | `description`    | object    |
     | `ingredients`    | object    |
     | `n_ingredients`  | int64     |
     | `user_id`        | float64   |
     | `recipe_id`      | float64   |
     | `date`           | object    |
     | `rating`         | float64   |
     | `review`         | object    |


3.  Fill all ratings of **0** in `rating` with **np.nan** and convert th rest to integers.
    - All the ratings have scale from **1** to **5** with **1** meaning the lowest rating **5** means the highest rating; therefore, a rating of **0** suggests that the rating for that speicifc recipe is missing. Thus, to avoid bias in the `ratings` and false results during our permutation test, we filled the value **0** with **np.nan**. Since we plan to build classifier, we changed the type to interger.

4. Check the **null** values across columns. Drop one row with `name` of the recipe being **nan**.

5. Add column `average_rating` containing average ratings per recipe.

    - Since a recipe can have numerous ratings from different users while we are merging the data, we take the **mean** of all the ratings, which will give us a more comprehensive understanding of all the ratings for the given recipe.

6. Split and convert the `nutritions`, `ingredients`, `tags` column from `str` to `list`.

    - Since we will be doing multiple extractions and look up of items in each column for future analysis, we thought of converting these columns which are **objects** types, acting like strings, to **list** type. Speicifically, we applied a lambda function to perform simple strip() and split() then converted the columns to **list** of strings or floats depending on the content. It will allow us to conduct numerical calculations, item lookup, and so forth on the columns.

7. Split values in the `nutrition` column to individual columns of floats

    - After converting the column to **list**, we then seperate each into a column with thecorresponding nutrition information as **float**. This could help us look up for speicifc nutrition information more quickly.

8. Convert `submitted` and `date` to datetime.

    - Sinnce these two columns are both stored as **objects**, for our convenience, we decidded to convert them into datetime to allow us conduct analysis on selected period of time.

9. Add `contains_meat` to the dataframe

    - The `contains_meat`is a boolean column checking if the tags of recipes contain `meat`. This step separates the recipes into two groups, with recipes contain `meat` and those without. This provides us a convenient way to compare ratings of recipes of two distributitons: recipes with and without `meat`.

### Result

Finally, here is our cleaned dataframe with columns in the appropriate types and is indexed by the recipe `name`:

| **Column**     | **Dtype**      |
|----------------|----------------|
| id             | int64          |
| minutes        | int64          |
| contributor_id | int64          |
| submitted      | datetime64[ns] |
| tags           | object         |
| nutrition      | object         |
| n_steps        | int64          |
| steps          | object         |
| description    | object         |
| ingredients    | object         |
| n_ingredients  | int64          |
| user_id        | float64        |
| recipe_id      | float64        |
| date           | datetime64[ns] |
| review         | object         |
| rating         | int64          |
| average_rating | float64        |
| contains_meat  | bool           |
| calories       | float64        |
| total_fat      | float64        |
| sugar          | float64        |
| sodium         | float64        |
| protein        | float64        |
| saturated_fat  | float64        |
| carbohydrates  | float64        |


Our cleaned dataframe has 234429 rows and 25 columns. Notice that some rows have the same `name` meaning they contain the same information for that specific recipe. Here is the first five rows with relevant columns selected:

| name                                 |     id |   minutes |   n_steps |   n_ingredients |   calories |   rating |   protein |   total_fat | contains_meat   |
|:-------------------------------------|-------:|----------:|----------:|----------------:|-----------:|---------:|----------:|------------:|:----------------|
| 1 brownies in the world    best ever | 333281 |        40 |        10 |               9 |      138.4 |        4 |         3 |          10 | False           |
| 1 in canada chocolate chip cookies   | 453467 |        45 |        12 |              11 |      595.1 |        5 |        13 |          46 | False           |
| 412 broccoli casserole               | 306168 |        40 |         6 |               9 |      194.8 |        5 |        22 |          20 | False           |
| 412 broccoli casserole               | 306168 |        40 |         6 |               9 |      194.8 |        5 |        22 |          20 | False           |
| 412 broccoli casserole               | 306168 |        40 |         6 |               9 |      194.8 |        5 |        22 |          20 | False           | 


## Univariate Analysis
Since we would like to classify the ratings in the end, we would like to understand the distribution of our `average_ratings`. As we can observe in the graph, the recipe ratings are more concentrated towards 4 or 5, meaning there is a clear left skew. We suspect that the `average_ratings` might be biased and people tend to give ratings for review especially when they particular like the receipe.
<iframe
    src="assets/rating-distribution.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

Another distribution we are interested in visualizing is the distribution of `protein` across all recipes. We can observe a decreasing trend, indicating that the higher the `protein` PDV, there are fewer of those recipes.  

<iframe
    src="assets/protein-distribution.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

## Bivariate Analysis

To investigate our most interested question,**will the meat inclusion(with meat tag) affect recipes' ratings**, we used a KDE plot to show the distribution of average ratings for recipes with and without the meat tag. We can see that
both curves, with and without meat tags, have a major peak around a certain rating (around 4-5). However, in general, the close alignment of the two KDE curves implies that the presence or absence of meat **does not** significantly shift the overall rating distribution.
<iframe
    src="assets/KDE.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

## Interesting Aggregates
For our aggregating analysis, we would like to use pivot table to compare and contrast the difference in `rating`, `protein`, `minutes` and `total_fat ` between two distributions in `contains_meat` group (with and without meat tag)
 Interestingly, we can see that recipes with meat deliver higher protein but also come with higher fat content and longer preparation times.

| contains_meat   |   minutes |   protein |   rating |   total_fat |
|:----------------|----------:|----------:|---------:|------------:|
| False           |   103.197 |   22.8448 |  4.68433 |     28.045  |
| True            |   118.354 |   66.4226 |  4.66549 |     44.3924 |


Another analysis we performed is to observe the trend of distributions of nutrition information based on `rating`. Overall, the **nutritional** metrics (calories, carbohydrates, protein, saturated fat, sugar, and total fat) show a fairly clear negative trend with increasing rating—lower values tend to correspond to higher ratings (at least from ratings 1 to 4). The decrease in calories, carbs, and fats from ratings 1 to 4 might reflect a preference for recipes that are perceived as healthier or less indulgent. However, the slight reversal in rating 5 (longer minutes, higher `fat`/`sugar`) could suggest that the most highly rated recipes balance healthiness with complexity or flavor richness that requires more preparation time and slightly richer ingredients.

Most metrics **decrease** as the rating increases, suggesting that higher ratings (in this range) might be associated with healthier or less energy-dense options. There are some small increase sin several values at rating 5 indicates that while there’s an overall downward trend, the highest rating category might not follow the pattern as closely as ratings 1 to 4, so there might be some outliers existing.

|   rating |   calories |   carbohydrates |   minutes |   n_ingredients |   protein |   saturated_fat |   sugar |   total_fat |
|---------:|-----------:|----------------:|----------:|----------------:|----------:|----------------:|--------:|------------:|
|        1 |    486.595 |         16.3962 |   99.6725 |         8.91289 |   34.0589 |         46.6791 | 88.0094 |     37.0564 |
|        2 |    446.598 |         15.0405 |   98.0215 |         9.22973 |   34.307  |         42.8822 | 75.1664 |     32.7669 |
|        3 |    425.791 |         13.6813 |   87.4976 |         9.20092 |   34.8582 |         40.0881 | 65.552  |     31.6405 |
|        4 |    405.047 |         12.8306 |   91.585  |         9.10076 |   34.0466 |         36.4327 | 56.7858 |     29.9404 |
|        5 |    415.213 |         13.038  |  106.924  |         9.04675 |   32.6446 |         39.2268 | 63.0816 |     31.7924 |

## Assessment of Missingness
The three columns in the cleaned dataframe that have non-trivial number missing entries are `rating`(**15036**), `review`(**2777**), and `description`(**136**).

### NMAR Analysis
In these three columns, we think that **none** is **NMAR**. 

In fact, we believe that all three columns all could be **MAR** due to the fact that they tend to correlate with each other. For instance, people who like the recipe will likely to give `rating` and also leave some positive feedback and `review` or the `description` of the dish itself to express his or her enjoyment. Also, the activeness/number of the `review`, `description`, and the `rating` could be also dependent on the `contributor_id` and the `name` of the recipe. One user could be really proactive in delievering messages or leave comments. One recipe may be really special/seasonal/memroable or for is designed for a specific holiday, leading to more views and attention. 

Specifically, we believe that `rating` is strongly likely to be influneced/correlated with other columns.
### Missingness Dependency
First, we would like to investigate whether the number of ingredients has an impact on the missingness of `rating`.  

***`n_ingredients` and `rating`'s missing dependecy***

**Null Hypothesis:** The missingness of ratings does not depend on the number of ingredients(`n_ingredients`) in the recipe.

**Alternative Hypothesis:** The missingness of ratings does depend on the number of ingredients(`n_ingredients`) in the recipe.

**Test Statistic:** the absolute difference in mean `n_ingredients` between missing and non-missing rating groups.

**Significance level:** 0.01

Here is our **result**:
<iframe
    src="assets/n_ingredients_missing.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

Our **Observed Absolute Difference in N_ingredients** is **0.1607**.
We then get **p-value** of **0.0**, which is less than **0.01**.
As a result, we **reject** the null hypothesis and conclude that the missingness of `rating` **does** depend on the number of ingredients(`n_ingredients`)

Moving on, we would also like to see whether the duration of cooking has an impact on the missingness of `rating`.  

***`minutes` and `rating`'s missing dependecy***

**Null Hypothesis:** The missingness of ratings does not depend on the duration of cooking the recipe. <br>

**Alternative Hypothesis:** The missingness of ratings does depend on the duration of cookinghe recipe. <br>

**Test Statistic:** The absolute difference in mean `minutes` between missing and non-missing rating groups. <br>

**Significance level:** 0.01

Here is our **result**:
<iframe
    src="assets/minutes_missing.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

Our **Observed Absolute Difference in Minutes** is **51.4524**.
We then get **p-value** of **0.1050**, which is greater than **0.01**.
As a result, we **failed to reject** the null hypothesis and conclude that the missingness of `rating` **there is lack of evidence proving** that the missingness depends on the cooking time of the recipe


## Hypothesis Testing
From the beginning, we would like to predict the rating of a recipe and we suspect that having `meat` in the recipes' `tags` have lower `rating` on average thnn those do not have.

To investigate this question, we designed a **permutaiton test** where we shuffled the boolean column we created `contains_meat`. Our hypothesis, decision rule, statistics, prodcedures are as follows:

**Null Hypothesis:** People rate meat and non-meat tagged dishes the same <br>

**Alternative Hypothesis:** People rate recipes with **meat** tagged  lower than they rate non-meat tagged ones <br>

**Decision Rule:** We will reject the null hypothesis if our p-value is less than **significance level** 0.05

**Test Statistic:** We used **difference in mean** between ratings of non-meat dishes and meat dishes(with meat-without meat) as test stats since it is a directional test

**Prodcedures:**
We run a 10000-simulation permutaiton test in order to get the empirical distribution of the test statistics under the null hypothesis. Since we lack prior information about the underlying population, so instead of relying on assumptions about the data, the permutation test allows us to directly assess whether the two observed distributions (recipes with meat and without meat) are likely to have come from the same population. We randomly shuffled the bool values given by `contains_meat`column many times and recalculating the rating differences each time, then we build an empirical distribution of the difference under the null hypothesis (that meat inclusion does not affect ratings).

Here is our **result**:

<iframe
    src="assets/permutation.html"
    width = "800"
    height = "600"
    frameborder = "0"
    style="margin: 0; padding: 0; display: block;"
></iframe>

Our **Observed Difference in Rating** (With Meat - Without Meat) is **-0.0188**.

We then get **p-value** of **0.0**, which is less than **0.05**.

As a result, we **reject** the null hypothesis and conclude that people rate recipes with meat in their tag lower than recipes without meat in their tag.

## Framing a Prediction Problem

In this project, we will be **Predicting the Rating of a Recipe**, which is a **multi-class classification problem**, as `rating` consists only the integer from 1-5 (note we replace the **0** with **np.nan** at the beginning), which would be an catagorical, ordinal variable.

We chose `rating` as our response variable as it is the statistic that best represent user satisfaction and overall quality. Additionally, we conduct several visualizations and analysis with `rating` as one of the variables and observed several trends across the distributions of `rating`. For instance, our hypothesis shows people rate recipes with meat in their tag **lower** than recipes without meat in their tag. So we may be able to predict the rating through features like inclusion of meat.

At the time of prediction, some avaliable features include `contains_meat` and `protein`, since they were all avalible as soon as the recipes were created before the actual ratings were given, we can safely use them as suitable predictors for our model.

To evaluate our model's effectiveness, we will take a look at both the model's **accuracy** and **F1-score**. 

**Accuracy:** The accuracy will allow us to analyze overall how our model is doing in terms of creating predictions based on **given** parameters.

**F1-Score:**  The f1 socre will calculate harmonic mean of precision and recall for each class, weighted by the class frequency. As shown in the above analysis, we observed the imbalanced distribution for `rating`, which are heavily skewed left with most around 4 to 5. So, using such metric calculation ensures that the performance on less frequent rating categories is properly accounted for.

## Baseline Model
We used the **Random Forest Classifier** with two parameters(one **quantitative** one **nominal**) for the baseline model.

1. `contains_meat`: As in the hypothesis section mentioned, people are likely to rate dishes with meat lower than the dishes without meat. This is a **nominal** variable contains **bool** value. We transformed the booleans into integers using **OneHotEncoder()** with 1 means True and 0 means False.

2. `protein `: The `protein` column contains continuous, numerical values with **float** type, making it **quantitative**. Speicifcally, it represents the **percentage of daily value** of **protein** for the speicifc recipe. We used **StandardScaler()** to standardize the data.

Additionally, we seperate the data into training group and testing group. The test group contains 20% of the data while the train group contians 80% of the data.

The statistics produced by the baseline model is as follows:

**Accuracy** 0.7736046856127077 <br>

**F1-Score** 0.17489827929656016 <br>


Our baseline has a pretty high accuracy but low F1-score. The f1-score shows that our baseline model is not overconfident and does not miss too many actual positives. In other words while the model is often correct, it likely performs very poorly on one or more classes—usually the minority class—resulting in low precision, recall, or both when these are combined as the F1 score. This is most likely due to the **imbalanced** distribution, as mentioned above during our analysis in EDA.

To improve our model, we decided to take a look into the feature importance:

For two of our **raw features: protein, contains_meat_True**, we have correponding transformed feature importance:

**standardscaler__protein** 0.9789 <br>

**onehotencoder__contains_meat_True** 0.0211 <br>

This shows that the model relies heavily on the protein feature to make predictions, while the indicator for whether a recipe contains meat plays a much smaller role.

## Final Model

Based on previous experiement, given the high accuracy but low f1 score, we decided to add more parameters and features to train our model. Also, we utilized **GridSearchCV** to find optimal hyperparameters `max_depth`,`n_estimators`,`min_samples_split`,`min_samples_leaf`, and `max_features`. We set the **class_weight** to **balanced** to account for our imbalanced data. We also introduced **K-fold cross-validation** as it provides a more reliable estimate of your model's performance.

In order to build a final model, we decided to do another round of exploratory analysis of all the **numerical** columns. Speicifcally, we would like to examine and take a look at the **distribution** and **correlation** with target column `rating` of each.

We made some key observations after the visualizations:

1. Some distributions like `n_steps` and `n_ingredients` look fairly symmetric, indicating that the mean and median are likely similar. For these we can just apply some simple **StandardScaler()**.

2. Other distributions especially for those relevant to **nutrition**, such as `protein` and `saturated_fat`, show clear skewness (either right‐skewed or left‐skewed), which implies that a few extreme values are pulling the mean away from the median. For these columns, we might need to apply potential transoformers and scalers like **log or exponential transformation** or **RobustScaler()**.

3. Most of the **datetime** type columns like `day` and `month` appeared to be relatively even, uniform, and symmetric.

In general, our data seems to be quite **imbalanced**. Due to this consideration, for **model selection**, we decided to keep using **Random Forest Classifiers**, which is tend to be robust to outliers and skewed distributions because they split data based on thresholds and are less affected by extreme values.

To make full use of our **datetime** columns, we split the **submitted** columns based on `day`, `month`, and `year`, and created a column for each, and we convert `date` columns to **ordinal** numbers as well.

After generating correlation score, we observed that many variables and **nutritional metrics** (total_fat, saturated_fat, sodium, sugar, calories, protein, carbohydrates) have correlations near zero (ranging from about **0.02463** to **-0.012863**), indicates that these factors have **little** direct linear association with the rating in this dataset.

This further proves that in order to increase predictive power, we have to create more informative features by using techniques such as **creating interaction terms** and **apply transformations** that better capture the underlying relationships.

Here is our procedures:

**Numerical Features:**

`calories`, `protein`, `total fat`, `sugar`, `sodium`, `saturated fat`, and `carbohydrates`

According to the pivot table, we know that nutritional information generally are good indicator for `rating`.(As `rating` increases, in general, all nutritional features decrease)
To address skewness and redundancy as shown in the above analysis, a custom log transformation (np.log1p) is applied to nutrition features, and then we utilized **RobustScaler()** to scale them considering the high potential of outliers.

**Preparation Details:**

`minutes`,`n_ingredients` and  `n_steps`

Features that are normally distributed like number of steps, and ingredients are scaled using **StandardScaler()**. We also know from previous pivot table analysis that higher rated recipes are more time-consuming and require more comlex procedures.

**Categorical Features:**

`contains_meat`

We keep using the **OneHotEncode()** as shown in our baseline model as it helps with achieving decent accuracy and also, in our hypothesis, we see there is an effect of the presence of `meat` tag on `rating`

**Textual Features:**

`tags`

Since it contians a list of tags for a given recipe, we preprocessed them by cleaning, lowercasing, and concatenating individual tags. They are then transformed into numerical features using TF-IDF. We think that there might be certain tags that are associated with higher views, search, and attentions, such as those related to holidays or special diet plan.


**Date Features:**

`submitted` and `date`

Since these were columns containing datetime objects, we think that recipes rating might vary in certain holidays, seasons, culture, societal trend, and other timeframe. Also, in our previous analysis, we see that higher rates were given in earlier years like 2008. To account for these, we further extract day, month, year, day of week, and ordinal values from the submitted and recipe dates to capture temporal patterns. We then applied **StandardScaler()** to scale them.

Here is our **result**:

**Best Hyperparameters:** {'clf__max_depth': None, 'clf__max_features': None, 'clf__min_samples_leaf': 1, 'clf__min_samples_split': 10, 'clf__n_estimators': 200} <br>

**Best Cross-Validated f1_macro:** 0.31980640001565172 <br>

**Test Set Accuracy:** 0.7764122973065432 <br>

While maintain the similar **accuracy** score, we see a great improvement in our **f1_score**. This means that our model was able to predict reviews for minority groups, which are those recipes with lower rating more accuratly than before!

## Fairness Analysis

For our fairness analysis, we decided to use the column `contains_meat` to create the group.

**Group X :** Defined as all recipes where the boolean column `contains_meat` is True.
**Group Y :** Defined as all recipes where the boolean column `contains_meat` is False.

We used **precision/recall parity** as our evaluation metric, as seen before. The metric calculates the precision for each class and then takes the average, ensuring that performance across all classes is considered equally. In imbalanced datasets or multi-class settings, overall accuracy can be misleading. It’s more important to focus on **False Positives**, that is, for the model to correctly identify the rating of a recipe among all instances of that rating. False positives would not be good since it would mislead users with the incorrectly labeled ratings.

**Null Hypothesis:** The model is fair. That is, the difference in macro precision between recipes with meat (Group X) and recipes without meat (Group Y) is zero, and any observed difference is due to random chance. <br>

**Alternative Hypothesis:** The model is unfair. In other words, there is a systematic difference in macro precision between the two groups (specifically, that recipes without meat have a higher precision than those with meat). <br>

**Decision Rule:** We will reject the null hypothesis if our p-value is less than **significance level** 0.01 <br>

**Test Statistic:** We used **difference in macro precision** between between the two groups(without meat-with meat) as our test statistic.

**Procedure**:
We performed a permutation test by randomly shuffling the group labels `contains_meat` 10000 times. For each permutation, we recalculated the absolute difference in macro precision. The p-value was then determined as the proportion of permutations in which the permuted absolute difference was greater than or equal to the observed absolute difference.


Our **Observed Difference in scores** is **0.13079558064360503**.

We then get **p-value** of **0.071**, which is less than **0.01**.

As a result, we **fail to reject** the null hypothesis. This suggests that the observed difference in precision could reasonably be due to random chance, and we conclude that there is no statistically significant evidence proving that our model is unfair.


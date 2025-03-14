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
| name                                 |   minutes |   protein | contains_meat   |   calories |   n_steps |   n_ingredients | submitted           | date                |   rating |   average_rating |
|:-------------------------------------|----------:|----------:|:----------------|-----------:|----------:|----------------:|:--------------------|:--------------------|---------:|-----------------:|
| 1 brownies in the world    best ever |        40 |         3 | False           |      138.4 |        10 |               9 | 2008-10-27 00:00:00 | 2008-11-19 00:00:00 |        4 |                4 |
| 1 in canada chocolate chip cookies   |        45 |        13 | False           |      595.1 |        12 |              11 | 2011-04-11 00:00:00 | 2012-01-26 00:00:00 |        5 |                5 |
| 412 broccoli casserole               |        40 |        22 | False           |      194.8 |         6 |               9 | 2008-05-30 00:00:00 | 2008-12-31 00:00:00 |        5 |                5 |
| 412 broccoli casserole               |        40 |        22 | False           |      194.8 |         6 |               9 | 2008-05-30 00:00:00 | 2009-04-13 00:00:00 |        5 |                5 |
| 412 broccoli casserole               |        40 |        22 | False           |      194.8 |         6 |               9 | 2008-05-30 00:00:00 | 2013-08-02 00:00:00 |        5 |                5 |

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


Another analysis we performed is to observe the trend of distributions of nutrition information based on `rating`. Recipes **with meat** tend to have **higher** calories, protein, total fat, and saturated fat, while non-meat recipes show higher carbohydrate` and sugar values. By observing these trends, we can identify which features may have predictive/determining power for our classifier. 


|   rating |   ('calories', False) |   ('calories', True) |   ('carbohydrates', False) |   ('carbohydrates', True) |   ('minutes', False) |   ('minutes', True) |   ('protein', False) |   ('protein', True) |   ('saturated_fat', False) |   ('saturated_fat', True) |   ('sugar', False) |   ('sugar', True) |   ('total_fat', False) |   ('total_fat', True) |
|---------:|----------------------:|---------------------:|---------------------------:|--------------------------:|---------------------:|--------------------:|---------------------:|--------------------:|---------------------------:|--------------------------:|-------------------:|------------------:|-----------------------:|----------------------:|
|        1 |               452.611 |              602.204 |                    18.206  |                  10.2393  |              76.8377 |             177.353 |              22.1722 |             74.4954 |                    43.261  |                   58.3067 |           103.058  |           36.816  |                32.1186 |               53.8543 |
|        2 |               417.497 |              533.707 |                    16.5065 |                  10.6526  |              78.3414 |             156.929 |              22.5854 |             69.3929 |                    39.7893 |                   52.14   |            89.4124 |           32.5245 |                29.2338 |               43.3423 |
|        3 |               396.173 |              516.864 |                    14.8705 |                  10.0244  |              76.0423 |             122.723 |              24.214  |             67.5892 |                    37.1255 |                   49.1983 |            76.0222 |           33.3563 |                28.1778 |               42.2886 |
|        4 |               370.899 |              505.299 |                    13.7789 |                  10.0467  |              81.7744 |             120.387 |              23.7568 |             64.2552 |                    32.4152 |                   48.2271 |            65.4865 |           31.2427 |                26.0506 |               41.3598 |
|        5 |               382.066 |              524.49  |                    14.0343 |                   9.75371 |             105.537  |             111.495 |              22.4546 |             66.2381 |                    35.4745 |                   51.5969 |            72.0983 |           33.3563 |                27.91   |               44.5914 | 


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

**Null Hypothesis:**The missingness of ratings does not depend on the duration of cooking the recipe.

**Alternative Hypothesis:** The missingness of ratings does depend on the duration of cookinghe recipe.

**Test Statistic:** the absolute difference in mean `minutes` between missing and non-missing rating groups.

**Significance level:**0.01

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
As a result, we **reject** the null hypothesis and conclude that the missingness of `rating` **does not** depend on the cooking time of the recipe


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

At the time of prediction, some avaliable features include `contains_meat`, `protein`, and `minutes`, since they were all avalible as soon as the recipes were created before the actual ratings were given, we can safely use them as suitable predictors for our model.

To evaluate our model's effectiveness, we will take a look at both the model's **accuracy** and **F1-score**. 

**Accuracy:** The accuracy will allow us to analyze overall how our model is doing in terms of creating predictions based on **given** parameters, in this case being `minute`,`protein` and `contains_meat`

**F1-Score:**  The f1 socre will calculate harmonic mean of precision and recall for each class, weighted by the class frequency. As shown in the above analysis, we observed the imbalanced distribution for `rating`, which are heavily skewed left with most around 4 to 5. So, using such metric calculation ensures that the performance on less frequent rating categories is properly accounted for.

## Baseline Model
We used the Random Forest Classifier with two parameters in the baseline model.

1. `contains_meat`: As in the hypothesis section mentioned, people are likely to rate dishes with meat lower than the dishes without meat. This is a nominal variable that affact rating. We transformed the booleans into integers with 1 means True and 0 means False.

2. `minutes`: The minutes column is a continuous, numerical variable in the dataframe. We used StandardScaler() to standardize the data.
We used k-fold cross validation to seperate the data into training group and testing group. The test group contains 20% of the data while the train group contians 80% of the data.

The statistics produced by the baseline model is as follows:

**Accuracy** 0.7728642203628608
**F1-Score** 0.6747765701443748


Our baseline has a pretty high accuracy and F1-score. The f1-score shows that our baseline model is not overconfident and does not miss to many actual positives. 


## Final Model
In the final model, in added many more parameters. The list of features and how we transformed them is as follow:

1. `minutes`: This is the continuous, numerical variable that we used in the baseline model. In the final model, we did log transformation to the `minutes` column to reduce the skewness, then we used `StandardScalar()` to normalize it. 

2. `protein`: We get the amount of protein from the `nutrition` column. 

3. `contains_meat`: This is the nominal variable that we used in the baseline model. We still use `OneHotEncoder()` to transform the booleans into integers with 1 means True and 0 means False.

4. `calories`

5. `n_ingredients` and  `n_steps`: These columns cotained the number of steps and the number of ingredients in the recipte. We added the two columns together to get the recipte_complexity. Then we take log transformations of the recipte_complexity to reduce skewness. Then we used `StandardScalar()` to normalize it. 

6. `saturated_fat` and `total_fat`： This column is gained from the `nutrition` column. We created the `saturated_fat_ratio` column by dividing the value of `saturated_fat` by `total_fat` because saturated_fat is considered bad in the dishes . It causes obesity and type 2 diabetes. The proportion saturated_fat may affect the rating of the recipes. Finally, we used `StandardScalar()` to normalize the values.

We used the `RandomForestClassifier` in the final model. We also added the argument `class_weight=balanced`. This will allow us to account for the uneven distribution that was seen in the model.

Finally, we run a GridSearch to find optimal hyperparameters, recursivelu searching the number of estimators, the max depth, and the minimum split arguments. 


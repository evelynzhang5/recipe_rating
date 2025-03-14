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
| name                                 |     id |   minutes |   contributor_id | submitted           | tags                                                                                                                                                                                                                        | nutrition                                                        |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | description                                                                                                                                                                                                                                                                                                                                                                       | ingredients                                                                                                                                                                                              |   n_ingredients |          user_id |   recipe_id | date                |   rating | review                                                                                                                                                                                                                                                                                                                                           |   average_rating | contains_meat   |   calories |   total_fat |   sugar |   sodium |   protein |   saturated_fat |   carbohydrates |
|:-------------------------------------|-------:|----------:|-----------------:|:--------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------|----------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|-----------------:|------------:|:--------------------|---------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------:|:----------------|-----------:|------------:|--------:|---------:|----------:|----------------:|----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27 00:00:00 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] | ['138.4', ' 10.0', ' 50.0', ' 3.0', ' 3.0', ' 19.0', ' 6.0']     |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']                                                  | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              | ["'bittersweet chocolate'", " 'unsalted butter'", " 'eggs'", " 'granulated sugar'", " 'unsweetened cocoa powder'", " 'vanilla extract'", " 'brewed espresso'", " 'kosher salt'", " 'all-purpose flour'"] |               9 | 386585           |      333281 | 2008-11-19 00:00:00 |        4 | These were pretty good, but took forever to bake.  I would send it ended up being almost an hour!  Even then, the brownies stuck to the foil, and were on the overly moist side and not easy to cut.  They did taste quite rich, though!  Made for My 3 Chefs.                                                                                   |                4 | False           |      138.4 |          10 |      50 |        3 |         3 |              19 |               6 |
| 1 in canada chocolate chip cookies   | 453467 |        45 |          1848091 | 2011-04-11 00:00:00 | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                               | ['595.1', ' 46.0', ' 211.0', ' 22.0', ' 13.0', ' 51.0', ' 26.0'] |        12 | ['pre-heat oven the 350 degrees f', 'in a mixing bowl , sift together the flours and baking powder', 'set aside', 'in another mixing bowl , blend together the sugars , margarine , and salt until light and fluffy', 'add the eggs , water , and vanilla to the margarine / sugar mixture and mix together until well combined', 'add in the flour mixture to the wet ingredients and blend until combined', 'scrape down the sides of the bowl and add the chocolate chips', 'mix until combined', 'scrape down the sides to the bowl again', 'using an ice cream scoop , scoop evenly rounded balls of dough and place of cookie sheet about 1 - 2 inches apart to allow for spreading during baking', 'bake for 10 - 15 minutes or until golden brown on the outside and soft & chewy in the center', 'serve hot and enjoy !'] | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            | ["'white sugar'", " 'brown sugar'", " 'salt'", " 'margarine'", " 'eggs'", " 'vanilla'", " 'water'", " 'all-purpose flour'", " 'whole wheat flour'", " 'baking soda'", " 'chocolate chips'"]              |              11 | 424680           |      453467 | 2012-01-26 00:00:00 |        5 | Originally I was gonna cut the recipe in half (just the 2 of us here), but then we had a park-wide yard sale, & I made the whole batch & used them as enticements for potential buyers ~ what the hey, a free cookie as delicious as these are, definitely works its magic! Will be making these again, for sure! Thanks for posting the recipe! |                5 | False           |      595.1 |          46 |     211 |       22 |        13 |              51 |              26 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ["'frozen broccoli cuts'", " 'cream of chicken soup'", " 'sharp cheddar cheese'", " 'garlic powder'", " 'ground black pepper'", " 'salt'", " 'milk'", " 'soy sauce'", " 'french-fried onions'"]          |               9 |  29782           |      306168 | 2008-12-31 00:00:00 |        5 | This was one of the best broccoli casseroles that I have ever made.  I made my own chicken soup for this recipe. I was a bit worried about the tsp of soy sauce but it gave the casserole the best flavor. YUM!                                                                                                                                  |                5 | False           |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
|                                      |        |           |                  |                     |                                                                                                                                                                                                                             |                                                                  |           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                   |                                                                                                                                                                                                          |                 |                  |             |                     |          | The photos you took (shapeweaver) inspired me to make this recipe and it actually does look just like them when it comes out of the oven.                                                                                                                                                                                                        |                  |                 |            |             |         |          |           |                 |                 |
|                                      |        |           |                  |                     |                                                                                                                                                                                                                             |                                                                  |           |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |                                                                                                                                                                                                                                                                                                                                                                                   |                                                                                                                                                                                                          |                 |                  |             |                     |          | Thanks so much for sharing your recipe shapeweaver. It was wonderful!  Going into my family's favorite Zaar cookbook :)                                                                                                                                                                                                                          |                  |                 |            |             |         |          |           |                 |                 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ["'frozen broccoli cuts'", " 'cream of chicken soup'", " 'sharp cheddar cheese'", " 'garlic powder'", " 'ground black pepper'", " 'salt'", " 'milk'", " 'soy sauce'", " 'french-fried onions'"]          |               9 |      1.19628e+06 |      306168 | 2009-04-13 00:00:00 |        5 | I made this for my son's first birthday party this weekend. Our guests INHALED it! Everyone kept saying how delicious it was. I was I could have gotten to try it.                                                                                                                                                                               |                5 | False           |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
| 412 broccoli casserole               | 306168 |        40 |            50969 | 2008-05-30 00:00:00 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                        | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    |         6 | ['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray , set aside', 'in a large bowl mix together broccoli , soup , one cup of cheese , garlic powder , pepper , salt , milk , 1 cup of french onions , and soy sauce', 'pour into baking dish , sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly , about 10 more minutes']                                                                                                                                                                                                                                                                                                                              | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 | ["'frozen broccoli cuts'", " 'cream of chicken soup'", " 'sharp cheddar cheese'", " 'garlic powder'", " 'ground black pepper'", " 'salt'", " 'milk'", " 'soy sauce'", " 'french-fried onions'"]          |               9 | 768828           |      306168 | 2013-08-02 00:00:00 |        5 | Loved this.  Be sure to completely thaw the broccoli.  I didn&#039;t and it didn&#039;t get done in time specified.  Just cooked it a little longer though and it was perfect.  Thanks Chef.                                                                                                                                                     |                5 | False           |      194.8 |          20 |       6 |       32 |        22 |              36 |               3 |
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

- *`n_ingredients` and `rating`'s missing dependecy*

**Null Hypothesis:**The missingness of ratings does not depend on the number of ingredients(`n_ingredients`) in the recipe.

**Alternative Hypothesis:** The missingness of ratings does depend on the number of ingredients(`n_ingredients`) in the recipe.

**Test Statistic:** the absolute difference in mean `n_ingredients` between missing and non-missing rating groups.

**Significance level:**0.01

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

- *`minutes` and `rating`'s missing dependecy*

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

In this project, we will be **Predictring the Rating of a Recipe**, which is a **multi-class classification problem**, as `rating` consists only the integer from 1-5 (note we replace the **0** with **np.nan** at the beginning), which would be an catagorical, ordinal variable.

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


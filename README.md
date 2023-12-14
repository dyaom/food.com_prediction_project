# Project: Rating Predictions on food.com

**Author**: Derrick Yao

This is a project for DSC80: Practice and Application of Data Science at UC San Diego.

My exploratory data analysis on this dataset can be found [here](https://dyaom.github.io/food.com_recipe_project/).

## Framing the Problem

In this project, I aim to predict the average rating of a recipe posted on [food.com](food.com). Over the course of this project, I show that framing this task as a classification task (predicting ratings 1,2,3,4,5) is better than framing this as a regression task (predicting rating 1-5).

This will be a multiclass classification task. The response variable is the average rating. I chose this variable because it is determined after the recipe is posted; temporally, the recipe and its information appears first, then the rating appears later.

I will use accuracy to evaluate my model. I chose this metric over other metrics because other metrics we discussed (e.g. precision and recall, F1-score) are ill-suited to multiclass classification tasks.

I will use information contained in the recipe post itself to train my model.

## Baseline Model

Initially, I trained a linear regression model, and rounded the output to get the categorical rating. A linear regression model fits a line (or hyperplane) of best fit to the points in the dataset.

This baseline model contains seven features, which are all quantitative. The seven features are the nutritional details of the recipe: calorie, total fat, sugar, sodium, protein, saturated fat, and carbohydrate content.

I encoded all seven features. For each feature, I took the square root to remove the right skew, then applied `scikit-learn`'s `StandardScaler` so that the range of each feature would be normalized.

After training the regression model, I found that the numerical results were meaningless, as it always predicted a value around 4.6:

<iframe src="assets/fig_true.html" width=600 height=450 frameBorder=0></iframe>

<iframe src="assets/fig_pred.html" width=600 height=450 frameBorder=0></iframe>

After rounding the numerical results into categories, I found that the baseline model had an accuracy of 0.6880 on the training set, and an accuracy of 0.6956 on the test set.

I do not believe that this model is very good. The accuracy is reasonably high, but it suffers from the dataset's initial skew towards 5-star ratings. Additionally, I only use nutritional information to predict ratings, while a recipe also includes other factors such as complexity.

## Final Model

In my final model, I added the complexity of the recipe as features; minutes, ingredients, and steps. The number of minutes to prepare a recipe is a quantitative feature; I encoded it with the same square root transform. The list of ingredients and steps is a nominal feature; I encoded it by counting the number of ingredients and number of steps.

Since preparing a recipe is a major part of the experience of using it, including this complexity information helps gain a broader understanding of a rater's experience.

To categorize the recipes into rankings, I chose a random forest model. I used grid search to select hyperparameters. The hyperparameters that ended up performing the best were:

* Using 120 decision trees in the random forest
* Limiting trees to a max depth of 18
* Limiting tree nodes to a minimum size of 65 observations
* Using the `gini` criterion

After training, I found that the final model had an accuracy of 0.6917 on the training set, and an accuracy of 0.6962 on the test set. This was a minor improvement over the baseline model.

## Fairness Analysis

I asked the question, "Does this model perform worse for non-5 ratings than for 5 ratings?" I used a permutation test to answer this question.

My evaluation metric for the permutation test was accuracy. I noted that, if this were a binary classification instead of a multiclass classification, this method would be similar to finding the recall of the model.

My null and alternative hypotheses were:

 * Null hypothesis: The accuracy for 5-star and non-5-star ratings is the same.
 * Alternative hypothesis: The accuracy for non-5-star ratings is lower than the accuracy for 5-star ratings.

My choice of test statistic was the difference in accuracy between the two groups. I set a significance level of 0.05.

After running the permutation test over 100,000 iterations, I obtained a p-value of 0.00. This shows that the difference in accuracies is extremely significant, and that the accuracy of non-5-star ratings is almost certainly lower than the accuracy for 5-star ratings.

<iframe src="assets/fig_fair.html" width=600 height=450 frameBorder=0></iframe>
# Phase 2 Project

Another module down--you're almost half way there!

![awesome](https://raw.githubusercontent.com/learn-co-curriculum/dsc-phase-2-project-campus/master/halfway-there.gif)

All that remains in Phase 2 is to put our newfound data science skills to use with a large project! This project should take 20 to 30 hours to complete.

## Project Overview

For this project, you will use regression modeling to analyze house sales in a northwestern county.

### The Data

This project uses the King County House Sales dataset, which can be found in  `kc_house_data.csv` in the data folder in this repo. The description of the column names can be found in `column_names.md` in the same folder. As with most real world data sets, the column names are not perfectly described, so you'll have to do some research or use your best judgment if you have questions about what the data means.

It is up to you to decide what data from this dataset to use and how to use it. If you are feeling overwhelmed or behind, we recommend you ignore some or all of the following features:

* date
* view
* sqft_above
* sqft_basement
* yr_renovated
* zipcode
* lat
* long
* sqft_living15
* sqft_lot15

### Business Problem

We have had the house selling records for the last few years. With these data, I want to build a model in which I can use the features in the data about the house to predict the price. In this case, we can guide both the seller and buyer to their business. The seller can use the model to predict the selling price of their house and if they need to do any renovation before selling their home. The buyer can have some suggestions about which kind of house they can afford based on their budget. To the details goal：

1. polish the data which have no meaning or is null to the price.
2. remove the features which do not contribute to the house price.
3. check if there are some high correlated features in which some of them can be removed.
4. build the linear regression model.
5. check how the features can contribute to the house change.

## import all the necessary library
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import warnings
warnings.filterwarnings("ignore")

## Loading the Data
In the cell below, I load the house data into a dataframe and checked it's dimension and datatype.

df = pd.read_csv("data/kc_house_data.csv")
df.head()

I need to steply remove and polish most of the columns which is not contribute to the price of house
1. The id is not related to the price
2. Split the date file to month and year.
3. Since the lat and long data is high related to the zipcode, I need to remove them.
4. Remodle the zip column with only the first three number
5. Remove the sqft_living15 and sqft_lot15 from columns.
6. Change the yr_built to the age of house at sold time
7. Change the yr_renovated to if the house is renovated and is the renovated within 10 and 30 years at sold.
8. Remove the 'sqft_living15', 'sqft_lot15'

df = df.drop(['id', 'lat', 'long'], axis  = 1)
df.zipcode = [int(x/10) for x in df.zipcode ]
month = []
year = []
for date in df.date:
    month.append(int(date.split('/')[0]))
    year.append(int (date.split('/')[2]))
df['month'] = month
df['year'] = (year)

df.fillna(value = 0, inplace=True)

df['is_renovated'] = [1  if x > 0 else 0 for x in df['yr_renovated']  ]
df['renovated_age'] = df['year'] - df['yr_renovated']
df['renovated_age2'] = [0 if x >1000 else x for x in df['renovated_age']]
df['renovated_10'] = [1 if (x <10) & (x >0)  else 0 for x in df['renovated_age2']]
df['renovated_30'] = [1 if (x <30) & (x >0) else 0 for x in df['renovated_age2']]
df['age_sold'] = df['year'] - df['yr_built']

#df['price'] = df['price']/(df['price'].max())
df['sqft_basement'] = [float(x) if x != '?' else 0.0 for x in df['sqft_basement']]

to_drop = ['date', 'sqft_living15', 'sqft_lot15' ,  'yr_renovated','yr_built', 'renovated_age', 'renovated_age2'  ] #, 
df_precessed = df.drop(to_drop, axis  = 1)
df_precessed = df_precessed[(df_precessed.bedrooms<33) & (df_precessed.age_sold >0)]
df_precessed.head()

## I checked the number of unique value for each of the columns
df_precessed.nunique()


### I then drawed the distribution of each of the columns  which had more than 10 unique value to check if there is any outlier values.
fig, axs = plt.subplots(2,5, figsize = (15,6))
plt1 = sns.boxplot(df_precessed['price'], ax = axs[0,0])
plt2 = sns.boxplot(df_precessed['bedrooms'], ax = axs[0,1])
plt3 = sns.boxplot(df_precessed['bathrooms'], ax = axs[0,2])
plt4 = sns.boxplot(df_precessed['sqft_living'], ax = axs[0,3])
plt5 = sns.boxplot(df_precessed['sqft_lot'], ax = axs[0,4])
plt1 = sns.boxplot(df_precessed['floors'], ax = axs[1,0])
plt2 = sns.boxplot(df_precessed['sqft_above'], ax = axs[1,1])
plt3 = sns.boxplot(df_precessed['sqft_basement'], ax = axs[1,2])
plt4 = sns.boxplot(df_precessed['age_sold'], ax = axs[1,3])

#The above figures show that there are multipal columns contain some outlier data. I then collected all the columns and remove them 
to_modify = ['price', 'bedrooms', 'bathrooms', 'sqft_living', 'sqft_lot','sqft_above','sqft_basement']
for col in to_modify:
    Q1 = df_precessed[col].quantile(0.25)
    Q3 = df_precessed[col].quantile(0.75)
    IQR = Q3 - Q1
    df_precessed = df_precessed[(df_precessed[col] >= Q1 - 1.5*IQR) & (df_precessed[col] <= Q3 + 1.5*IQR)]

### check the data after modification
fig, axs = plt.subplots(2,5, figsize = (15,6))
plt1 = sns.boxplot(df_precessed['price'], ax = axs[0,0])
plt2 = sns.boxplot(df_precessed['bedrooms'], ax = axs[0,1])
plt3 = sns.boxplot(df_precessed['bathrooms'], ax = axs[0,2])
plt4 = sns.boxplot(df_precessed['sqft_living'], ax = axs[0,3])
plt5 = sns.boxplot(df_precessed['sqft_lot'], ax = axs[0,4])
plt1 = sns.boxplot(df_precessed['floors'], ax = axs[1,0])
plt2 = sns.boxplot(df_precessed['sqft_above'], ax = axs[1,1])
plt3 = sns.boxplot(df_precessed['sqft_basement'], ax = axs[1,2])
plt4 = sns.boxplot(df_precessed['age_sold'], ax = axs[1,3])

The data looks much better now with very few of outlier numbers.


#  In order to check the relationship between the price with most of the columns with few unique numbers, 
# I plot their relations in seperate figures.
plt.figure(figsize=(20, 12))
plt.subplot(4,3,1)
sns.boxplot(x = 'bedrooms', y = 'price', data = df_precessed)
plt.subplot(4,3,2)
sns.boxplot(x = 'floors', y = 'price', data = df_precessed)
plt.subplot(4,3,3)
sns.boxplot(x = 'waterfront', y = 'price', data = df_precessed)
plt.subplot(4,3,4)
sns.boxplot(x = 'view', y = 'price', data = df_precessed)
plt.subplot(4,3,5)
sns.boxplot(x = 'condition', y = 'price', data = df_precessed)
plt.subplot(4,3,6)
sns.boxplot(x = 'grade', y = 'price', data = df_precessed)
plt.subplot(4,3,7)

sns.boxplot(x = 'is_renovated', y = 'price', data = df_precessed)
plt.subplot(4,3,8)
sns.boxplot(x = 'renovated_10', y = 'price', data = df_precessed)
plt.subplot(4,3,9)
sns.boxplot(x = 'renovated_30', y = 'price', data = df_precessed)
plt.subplot(4,3,10)
sns.boxplot(x = 'bathrooms', y = 'price', data = df_precessed)
plt.subplot(4,3,11)

sns.boxplot(x = 'month', y = 'price', data = df_precessed)
plt.show()

### The scatter plot of each two columns shows in general how the feature realated to each other and if there is any obvious correlation between them.
scatter_matrix = pd.plotting.scatter_matrix(
    df_precessed,
    figsize  = [20, 20],
    marker   = ".",
    s        = 0.2,
    diagonal = "kde"
)

for ax in scatter_matrix.ravel():
    ax.set_xlabel(ax.get_xlabel(), fontsize = 10, rotation = 90)
    ax.set_ylabel(ax.get_ylabel(), fontsize = 10, rotation = 0)

Base on the scatter figure above, there are several features correlated with each other. However, visual approach to finding correlation cannot be automated, so a numeric approach is a good next step.


### I tested the pairs of feature with correlation more than 0.75.
df = df_precessed.corr().abs().stack().reset_index().sort_values(0, ascending = False)
df['pairs'] = list(zip(df.level_0, df.level_1))
df.set_index(['pairs'], inplace = True)
df.drop(columns = ['level_0', "level_1"], inplace  = True)
df.columns = ['cc']
df.drop_duplicates(inplace = True)
df[(df.cc>.7) & (df.cc<1)]


There are three pairs of features high related with each other. I need to remove at least one of the features in each pair. Comparing the last list, I decided to delete the columns sqft_above, renovated_30, year, month. 

to_drop = ['sqft_above', 'renovated_30', 'year', 'month' ]
df_precessed = df_precessed.drop(to_drop,axis  = 1 )


# Regression
Until now, I finished the polish of the all the features and then I will split the data to trainning and testing parts to do the fitting.

### split the data to training and testing part
from sklearn.model_selection import train_test_split
y = df_precessed['price']
X = df_precessed.drop('price', axis  = 1)
X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=42)
print(len(X_train), len(X_test), len(y_train), len(y_test))

#check heatmap of the data to find out the most correlated feature and make the base line

heatmap_data = pd.concat([y_train, X_train], axis = 1)
corr = heatmap_data.corr()

#setup figure for heatmap
fig, ax = plt.subplots(figsize = (15, 15))

### Plot a heatmap of the correlation matrix, with both numbers and colors indicating the correlations
sns.heatmap(
    # Specifies the data to be plotted
    data=corr,
    # The mask means we only show half the values,
    # instead of showing duplicates. It's optional.
    mask=np.triu(np.ones_like(corr, dtype=bool)),
    # Specifies that we should use the existing axes
    ax=ax,
    # Specifies that we want labels, not just colors
    annot=True,
    # Customizes colorbar appearance
    cbar_kws={"label": "Correlation", "orientation": "horizontal", "pad": .2, "extend": "both"}
    )

    #Customize the plot appearance
    ax.set_title("Heatmap of Correlation Between Attributes (Including Price)");
    
most_correlated_feature = "grade"


from sklearn.linear_model import LinearRegression
from sklearn.model_selection import cross_validate, ShuffleSplit

baseline_model = LinearRegression()


splitter = ShuffleSplit(n_splits=3, test_size=0.25, random_state=0)

baseline_scores = cross_validate(
    estimator=baseline_model,
    X=X_train[[most_correlated_feature]],
    y=y_train,
    return_train_score=True,
    cv=splitter
)

print("Train score:     ", baseline_scores["train_score"].mean())
print("Validation score:", baseline_scores["test_score"].mean())

Because we are using the .score method of LinearRegression, these are r-squared scores. That means that each of them represents the amount of variance of the target ( price) that is explained by the model's features (currently just the number of grade) and parameters (intercept value and coefficient values for the features).

In general this seems like not a very strong model. However, it is getting nearly identical performance on training subsets compared to the validation subsets, explaining around 50% of the variance both times.

We will need to add more features to the model to check if there is any improvement.

## Build a Model with All Numeric Features 

second_model = LinearRegression()

second_model_scores = cross_validate(
    estimator=second_model,
    X=X_train,
    y=y_train,
    return_train_score=True,
    cv=splitter
)

print("Current Model")
print("Train score:     ", second_model_scores["train_score"].mean())
print("Validation score:", second_model_scores["test_score"].mean())
print()
print("Baseline Model")
print("Train score:     ", baseline_scores["train_score"].mean())
print("Validation score:", baseline_scores["test_score"].mean())

Our second model got better scores on the training data, and better scores on the validation data. However, I still want to continue to check how each feature work in general. Then I choose to check the coef value of the regression

###  Select the Best Combination of Features
import statsmodels.api as sm

sm.OLS(y_train, sm.add_constant(X_train)).fit().summary()

### Base on the p value, I temperaly select 10 columns in which p<0.05
select_cat = ['bedrooms', 'bathrooms','sqft_living', 'sqft_lot', 'floors', 'waterfront', 'view', 'condition', 'grade',
              'sqft_basement', 'renovated_10', 'age_sold']
X_train_third = X_train[select_cat]

third_model = LinearRegression()

third_model_scores = cross_validate(
    estimator=third_model,
    X=X_train_third,
    y=y_train,
    return_train_score=True,
    cv=splitter
)

print("current Model")
print("Train score:     ", third_model_scores["train_score"].mean())
print("Validation score:", third_model_scores["test_score"].mean())

print("second Model")
print("Train score:     ", second_model_scores["train_score"].mean())
print("Validation score:", second_model_scores["test_score"].mean())
print()
print("Baseline Model")
print("Train score:     ", baseline_scores["train_score"].mean())
print("Validation score:", baseline_scores["test_score"].mean())

There is a little bit improve on the prediction, but very little.
 I tried to selecting Features with sklearn.feature_selection

from sklearn.feature_selection import RFECV
from sklearn.preprocessing import StandardScaler

#Importances are based on coefficient magnitude, so
#we need to scale the data to normalize the coefficients
X_train_for_RFECV = StandardScaler().fit_transform(X_train)

model_for_RFECV = LinearRegression()

#Instantiate and fit the selector
selector = RFECV(model_for_RFECV, cv=splitter)
selector.fit(X_train_for_RFECV, y_train)

#Print the results
print("Was the column selected?")
for index, col in enumerate(X_train.columns):
    print(f"{col}: {selector.support_[index]}")
Was the column selected?
bedrooms: True
bathrooms: True
sqft_living: True
sqft_lot: True
floors: True
waterfront: True
view: True
condition: True
grade: True
sqft_basement: True
zipcode: False
is_renovated: False
renovated_10: True
age_sold: True

The RFE methods give me the same selection of features above.

The results showed that the auto sedlected features did not give better score than the third model.

Now, I remade the third model features to best_features to validate the final model.


#Base on the train score and validation score, the best columns until now is the third model. 


X_train_final = X_train[select_cat]
X_test_final = X_test[select_cat]


final_model = LinearRegression()

#Fit the model on X_train_final and y_train
final_model.fit(X_train_final, y_train)

#Score the model on X_test_final and y_test
#use the built-in .score method
final_model.score(X_test_final, y_test)


# Validation
## import the mse to check the mse value
from sklearn.metrics import mean_squared_error

mean_squared_error(y_test, final_model.predict(X_test_final), squared=False)

#check the distribution of price in test data
y_test.hist(bins = 100)
y_test.mean()

""" This means that for an average house price, this algorithm will be off by about $130331 thousands. Given that the mean value of house price is 445683, the algorithm can patially set the price. However, we still want to have a human double-check and adjust these prices rather than just allowing the algorithm to set them. """

print(pd.Series(final_model.coef_, index=X_train_final.columns, name="Coefficients"))
print()
print("Intercept:", final_model.intercept_)



preds = final_model.predict(X_test_final)
fig, ax = plt.subplots(figsize =(5,5))

perfect_line = np.arange(y_test.min(), y_test.max())
#perfect_x = [0, 1]
#perfect_y = [0, 1]

#ax.plot(perfect_x, perfect_y, linestyle="--", color="black", label="Perfect Fit")
ax.plot(perfect_line,perfect_line, linestyle="--", color="black", label="Perfect Fit")
ax.scatter(y_test, preds, alpha=0.5)
ax.set_xlabel("Actual Price")
ax.set_ylabel("Predicted Price")
ax.legend();

import scipy.stats as stats
residuals = (y_test - preds)
sm.graphics.qqplot(residuals, dist=stats.norm, line='45', fit=True);

fig, ax = plt.subplots()

ax.scatter(preds, residuals, alpha=0.5)
ax.plot(preds, [0 for i in range(len(X_test))])
ax.set_xlabel("Predicted Value")
ax.set_ylabel("Actual - Predicted Value");

The validation of prediction and real data shows that the prediction price for most house whose price is low (20% of the max price) is close to the real price. qqplot showes that the house price is not well normal distributed but peaked in the middle. There is a lot of shift of prediction price when the house value increase especialy when house price is more than 2 million.


## Summary

Our model predicted well the house price on many of the features. The Coefficients are like between bedrooms -17734, bathrooms 22333, sqft_living 104, sqft_lot -7, floors 17895, waterfront 140605 , view 30502, condition 20867 , grade 104396, sqft_basement 10 , renovated_10 46690, age_sold 2655,

To the buyer, they can estimate the price of the house base on the features of the house. To the seller, if they want to sell the house in a better value, they can try to renovate the house and make water front if possible.

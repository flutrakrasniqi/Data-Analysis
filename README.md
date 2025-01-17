# Data Analyses: Bugs

A somewhat silly psychology experiment conducted in 2013 measured subjects’ reactions to
pictures of bugs. Independent raters had deemed the bugs to be either disgusting or non-disgusting,
and either frightening or non-frightening. The subjects had to rate how much they wanted to kill
each bug, on a scale from 1 to 10. You can read more about the experiment here:

https://doi.org/10.1016/j.chb.2013.01.024

And you can obtain a csv file of the data from the experiment here:
https://raw.githubusercontent.com/luketudge/stats-tutorials/master/tutorials/data/bugs.csv

Write a Python program that produces an analysis of these data. Your program should print some
numerical summaries and produce a plot:
- Summary statistics of the kill ratings for each type of bug:
  - minimum
  - maximum
  - median
  - mean
  - Standard Deviation
- The results of a linear model with kill rating as the outcome variable and the categories of
bug as the predictor variables.
- Boxplots with overlaid points showing the distribution of kill ratings for each category of
bug. 

# Contents
- [Load data](https://github.com/flutrakrasniqi/flutra/blob/master/README.md#load-data)
- [Preparing the data](https://github.com/flutrakrasniqi/flutra/blob/master/README.md#preparing-the-data)
- [Summary statistics](https://github.com/flutrakrasniqi/flutra/blob/master/README.md#summary-statistics)
- [Linear model](https://github.com/flutrakrasniqi/flutra/blob/master/README.md#linear-model)
- [Visualizing data](https://github.com/flutrakrasniqi/flutra/blob/master/README.md#visualizing-data)


## Load data

```python
url = 'https://raw.githubusercontent.com/luketudge/stats-tutorials/master/tutorials/data/bugs.csv'
```

We save our url file into a variable. This file stores a table of data on the desire of subjects to kill a bug. Each row represents one bug, with the following columns:
- Subject: An ID code for the person rating their desire to kill the bug
- Sex: The sex of the subject
- Disgust: The level of how disgusting is the bug (low/high)
- Fear: The level of how frightening is the bug (low/high)
- KillRating: The subject’s desire to kill the bug (1-10).

Let's import `pandas` and use its `read_csv()` function to read this file:
```python
import pandas
df = pandas.read_csv(url)
```
Taking into account that we may mistakenly give a wrong url we should raise an error that will stop the program and inform us that the url file is not found:
```python
try:
    df = pandas.read_csv(url)
except IOError:
    raise FileNotFoundError('URL is not found!')
```

```python
df
```
```
     Subject     Sex Disgust  Fear  KillRating
0          1  Female     low   low         6.0
1          3  Female     low   low         5.0
2          4  Female     low   low         6.0
3          5  Female     low   low         3.0
4          6  Female     low   low         2.0
..       ...     ...     ...   ...         ...
343       96    Male    high  high        10.0
344       97  Female    high  high        10.0
345       98  Female    high  high        10.0
346       99  Female    high  high        10.0
347      100  Female    high  high         3.0

[348 rows x 5 columns]
```
## Preparing the data

Before proceeding to analyzing the data let’s first define our variables in terms of predictors and outcome, in order to make our program more flexible. In the experiment the kill rating supposedly depends on the category of bug which is determined by its level of disgust and fear. So, we have:
```python
predictor1 = 'Disgust'
predictor2 = 'Fear'
outcome = 'KillRating'
```

This way, if in the future we want to make changes, we don’t have to find each place where the variables are written in the program to rewrite them.

As mentioned above the kill rating is supposed to depend on the category of the bug. So, we want to have a new column indicating the bugs' category. 
To assign each bug to a category, we will define a function that looks at the value of the disgust and fear column of a row and by combining them returns one of four possible categories (combinations). In other instances (e.g. there is a missing value in one of the columns) the function returns `None`.

```python
def label_bug(row):
  if row[predictor1] == 'low' and row[predictor2] == 'low' :
      return '1. Low {} and Low {}'.format(predictor1, predictor2)
  elif row[predictor1] == 'low' and row[predictor2] == 'high' :
      return '2. Low {} and High {}'.format(predictor1, predictor2)
  elif row[predictor1] == 'high' and row[predictor2] == 'low' :
      return '3. High {} and Low {}'.format(predictor1, predictor2)
  elif row[predictor1] == 'high' and row[predictor2] == 'high' :
      return '4. High {} and High {}'.format(predictor1, predictor2)
  else:
      return None
```

Now we want to apply this function to each row of the data frame and save its results into new column called ‘bug_category’. The `.apply` function applies a function along an axis of the DataFrame. As arguments we specify:
- `the function` which will be applied; and 
- `the axis` (when axis= 1, the function is applied to each row).
```python
df['bug_category'] = df.apply (label_bug, axis=1)
```
Now we can also drop the rows that have missing values in our new column since it doesn’t make sense to keep a row that doesn’t contain the predictors value. However, statistically I am not sure if this is the right thing to do.

`pandas`'s method `dropna()` removes the rows (specified by the argument: axis=0) that have missing values in one column (specified by: subset=[‘bug_category’]).

```python
df = df.dropna(axis=0, subset=['bug_category'])
```
This is what df includes now:
```python
df
```
```
	Subject	Sex	Disgust	Fear	KillRating	bug_category
0	1	Female	low	low	6.0		1. Low Disgust and Low Fear
1	3	Female	low	low	5.0		1. Low Disgust and Low Fear
2	4	Female	low	low	6.0		1. Low Disgust and Low Fear
3	5	Female	low	low	3.0		1. Low Disgust and Low Fear
4	6	Female	low	low	2.0		1. Low Disgust and Low Fear
...	...	...	...	...	...		...
343	96	Male	high	high	10.0		4. High Disgust and High Fear
344	97	Female	high	high	10.0		4. High Disgust and High Fear
345	98	Female	high	high	10.0		4. High Disgust and High Fear
346	99	Female	high	high	10.0		4. High Disgust and High Fear
347	100	Female	high	high	3.0		4. High Disgust and High Fear
348 rows × 6 columns
```
## Summary statistics

Often one of the first steps when analyzing new data is to get an idea of the spread or 'distribution' of the different values in some columns. We might want to know what the lowest and highest values are, what the most common values are, and so on. Summarizing multiple values in one number is also known as 'aggregating' the values. The `aggregate()` function of a `pandas.DataFrame` returns a new shorter data frame containing only the aggregated values. The first argument to aggregate() specifies what function or functions to use to aggregate the data.

If we want to apply multiple aggregation functions, then it is a little clearer to first gather the functions in a list variable, and then pass this list variable into the `aggregate()` function. 
```python
from statistics import median, mean, stdev
summary_stats = [min, median, mean, stdev, max]
```

So, let’s check the summary statistics of the kill ratings (our outcome):
```python
summary = df.aggregate({outcome: summary_stats})
print(summary)
```
```
        KillRating
min       0.000000
median    7.500000
mean      6.859195
stdev     2.840425
max      10.000000
```

Since we want this information separately for each category of bug, we need a ’grouped summary’.

`pandas` makes grouping our data pretty easy. The `groupby()` function groups the rows of a data frame according to the categories in one or more columns. The first argument is a list of column names to group by.

```python
summary = df.groupby(['bug_category']).aggregate({outcome: summary_stats})
print(summary)
```
```
	                            KillRating
                              	min  	 median   mean	        stdev	        max
bug_category					
1. Low Disgust and Low Fear	0.0	 6.0	  5.632184	2.680094	10.0
2. Low Disgust and High Fear	0.5	 8.0	  7.339080	2.551213	10.0
3. High Disgust and Low Fear	0.0	 7.5	  6.660920	3.151683	10.0
4. High Disgust and High Fear	0.0	 8.5	  7.804598	2.495760	10.0
```

## Linear model

To model the relationship between our dependent variable (kill rating) and independent variables (categories of bugs) we are going to use the linear regression model. The regression can only use numerical variables as its input data. Due to this, the categorical variables need to be encoded as dummy variables. Dummy coding encodes the categorical variables as 0 and 1 respectively if the observation does not or does belong to the group.

This encoding can easily be done with `pandas`'s function `get_dummies()`, where the first argument specifies data from which to get dummy indicators.

In linear regression with categorical variables we should be careful of the Dummy Variable Trap. The Dummy Variable trap is a scenario in which the independent variables are multicollinear - a scenario in which two or more variables are highly correlated; in simple terms one variable can be predicted from the others. This can produce singularity of a model, meaning our model just won't work. 

The idea is to use `get_dummies()` with a second argument drop_first=True; this will omit one column after converting categorical variables into dummy/indicator variables. We will not lose any relevant information by doing that simply because all points in dataset can fully be explained by rest of the features.

```python
embarked_dummies=pandas.get_dummies(df.bug_category, drop_first=True)
```

Now we join these dummy variable columns with the main dataset, using `pandas.concat()`. The first argument here is a list of datasets that we want to join, and the second is the axis to concatenate along:
```python
df = pandas.concat([df, embarked_dummies], axis=1)
```

This is how our df looks now:
```python
df
```
```
Subject	Sex	Disgust	Fear	KillRating	bug_category		        2. Low Disgust and High Fear	3. High Disgust and Low Fear	4. High Disgust and High Fear	
0	1	Female	low	low	6.0	1. Low Disgust and Low Fear	0				0			0
1	3	Female	low	low	5.0	1. Low Disgust and Low Fear	0				0			0
2	4	Female	low	low	6.0	1. Low Disgust and Low Fear	0				0			0
3	5	Female	low	low	3.0	1. Low Disgust and Low Fear	0				0			0
4	6	Female	low	low	2.0	1. Low Disgust and Low Fear	0				0			0
...	...	...	...	...	...	...				...				...			...	
343	96	Male	high	high	10.0	4. High Disgust and High Fear	0				0			1	
344	97	Female	high	high	10.0	4. High Disgust and High Fear	0				0			1	
345	98	Female	high	high	10.0	4. High Disgust and High Fear	0				0			1	
346	99	Female	high	high	10.0	4. High Disgust and High Fear	0				0			1	
347	100	Female	high	high	3.0	4. High Disgust and High Fear	0				0			1	
348 rows × 15 columns
```

We can now continue to use them in our linear model. But first, we split our data frame into two sets. We will do this by using four fifths of the data for the model fitting (the so-called 'training' data), and reserving the remaining fifth as the imagined new data against which we will test the fitted model's predictions (the so-called 'test' data).

We split the data by using `train_test_split()` from `sklearn.model_selection`. Here the first parameter represents the datasets we are selecting to use (predictors and outcome); the second parameter sets the size of the testing dataset, and about a third one: `random_state`, we set it to some fixed number since want reproducible results (if we do not use a randomstate , every time we make the split/run the program we might get a different set of train and test data points).
```python
from sklearn.model_selection import train_test_split
X = embarked_dummies
y = df[outcome]
X_train,X_test,y_train,y_test = train_test_split(X, y, test_size = .20, random_state=1)
```

We have split our data into training and testing sets, and now is finally the time to fit the linear model to our data.

With `Scikit-Learn` it is extremely straight forward to implement linear regression models, as all we really need to do is import the `LinearRegression` class, instantiate it, and call the `fit()` method along with our training data.
```python
from sklearn.linear_model import LinearRegression
reg = LinearRegression().fit(X_train,y_train)
```

Theoretically linear regression model basically finds the best value for the intercept and coificients, which result in a line that best fits the data. To see the values of the intercept and coificients calculated by the linear regression algorithm for our dataset, we execute the following code:
```python
print('Intercept: ', reg.intercept_)
print('Coefficients: ', reg.coef_)
```
```
Intercept:  5.691780821917809
Coefficients:  [1.43321918 0.95107632 2.13216284]
```

After we have trained our model, we can make predictions on the test data by using the function `predict()`. This accepts one argument; the new data concerning the predictors (in our case the data that we split to test out model):
```python
y_pred=reg.predict(X_test)
```

The y_pred is an array that contains all the predicted values for the input values in the X_test series. 

Now we can compare the actual outcome’s values and the predicted ones:
```python
df1 = pandas.DataFrame({'Actual': y_test, 'Predicted': y_pred})
print(df1)
```
```
	actual	Predicted
85	9.0	5.691781
191	8.5	6.642857
107	10.0	7.125000
105	4.5	7.125000
123	2.0	7.125000
...	...	...
347	3.0	7.823944
92	10.0	7.125000
29	4.5	5.691781
58	2.0	5.691781
192	5.0	6.642857
70 rows × 2 columns
```

And we can also evaluate the performance of our model by finding the values of Mean squared error and R squared. To do this, from `sklearn.metrics`, we import `mean_squared_error`, `r2_score` functions which as arguments take the actual outcome values and the predicted outcome values:
```python
from sklearn.metrics import mean_squared_error, r2_score
print('Mean squared error: ', mean_squared_error(y_test, y_pred))
print('R2:', r2_score(y_test,y_pred))
```
```
Mean squared error:  7.847481505967478
R2: 0.08544037628158796
```

So in this model the average squared difference between the estimated values and true values of kill ratings is about 8 numbers and the model explains only 8% of the variation on kill ratings.

## Visualizing data

To show the relationship between multiple variables in a dataset we also can use different visual representations. A box plot (or box-and-whisker plot) shows the distribution of quantitative data in a way that facilitates comparison between variables or across levels of a categorical variable. The box shows the quartiles of the dataset while the whiskers extend to show the rest of the distribution, except for points that are determined to be “outliers”.

We can create box plots using seaborn's sns.boxplot method, but first we specify the size and the style we want our figure to have:
```python
import seaborn as sns
import matplotlib.pyplot as plt

plt.figure(figsize=(7,5))
sns.set_style('darkgrid')
```

Another thing we have to take care beforehead, if we want our box plots to be in some specific colors, is creating a palette. xkcd_palette() from seaborn makes it possible for us to create a palette with color names [from xkcd color survey](https://xkcd.com/color/rgb/). As argument it excepts a list of strings (color names): 
```python
our_palette = sns.xkcd_palette(["salmon", "aquamarine"])
```

Now we can plot the data in a box and whisker plot using boxplot() method. Two first parameters in this method specify the columns that will be represented in x and y axis. The hue parameter determines which column should be used for color encoding. Then we specify the dataset for plotting and the palette to use for different levels of the hue variable.
```python
bp = sns.boxplot(x=predictor1, y=outcome, hue=predictor2, data=df, 
                 palette = our_palette)
```

To add the overlaid points that show the distribution of kill ratings for each category of bug, we use another method from seaborn called `stripplot()`. Except the common parameters, here we also specify:

> `jitter=True`: to add some random noise (“jitter”) to discrete values, to make the distribution of those values clearer; <br /> `dodge = True`: to separate the strips for different hue levels along the categorical axis; and <br /> `linewidth = 1`: to add gray lines that frame the points, since were using the same palette as for the boxes.
```python
bp = sns.stripplot(x=predictor1, y=outcome, hue=predictor2, data=df,
                   jitter=True, dodge=True, linewidth=1, palette = our_palette)
```

And this is the plot that we get: <br />
![](image/graph.png)

After we have plotted the data, if we want to add grid for both axis and customize the y axis grid so it shows lines for each (every 1) integer, we execute the following code:
```python
from matplotlib.ticker import MultipleLocator
bp.grid(True, which='major')
bp.yaxis.set_major_locator(MultipleLocator(1))
```

To specify the locator which is the argument that `set_major_locator()` accepts, we need to import `MultipleLocator()` from matplotlib.ticker. The parameter in MultipleLocator specifies after how many units a line will show up.

Other details that we can handle regarding the plot are the y axis label which would be clearer if it shows ‘Desire to kill’, and also the legend where we have doubled handles and labels.

To make the clearer y axis label we use pyplot and execute:
```python
plt.ylabel('Desire to kill')
```

To deal with the legend, we use pyplot as well but first we get the legend’s handles and labels from the plot and save them as variables:
```python
handles, labels = bp.get_legend_handles_labels()
```

Next, we use the legend method from `pyplot` to kind of build a new legend, where we specify:
>`handles[0:2]` – we only want the first 2 handles to be shown; labels[0:2] – we want only the first 2 labels to be shown; <br />
`loc=2` – we place the legend on the upper left corner; bbox_to_anchor=(x,y) – in conjunction with loc places the legend on x and y axis giving a great degree of control for manual legend placement; <br />
`Frameon=False` – to get rid of the legend's frame; and title=predictor2.
```python
lg = plt.legend(handles[0:2], labels[0:2], loc=2,
           bbox_to_anchor=(1, .5), frameon=False, title=predictor2)
```
![](image/graph_final.png)

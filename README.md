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
bug. Try to recreate the main features of the plot shown below.

## Load data

url = https://raw.githubusercontent.com/luketudge/stats-tutorials/master/tutorials/data/bugs.csv

We save our url file into a variable. This file stores a table of data on the desire of subjects to kill a bug. Each row represents one bug, with the following columns:
- Subject: An ID code for the person rating their desire to kill the bug
- Sex: The sex of the subject
- Disgust: The level of how disgusting is the bug (low/high)
- Fear: The level of how frightening is the bug (low/high)
- KillRating: The subject’s desire to kill the bug (1-10).

Let's import pandas and use its read_csv() function to read this file:
```python
import pandas
df = pandas.read_csv(url)
```
Taking into account that we may mistakenly give a wrong url, file we should raise an error that will stop the program and inform us that the url file is not found:
```python
try:
    df = pandas.read_csv(url)
except IOError:
    raise FileNotFoundError('URL is not found!')
```

```python
df
```
## Preparing the data

Before proceeding to analyze the data let’s first define our variables in terms of predictors and outcome, in order to make our program more flexible. In the experiment the kill rating supposedly depends on the category of bug which is determined by its level of disgust and fear. So, we have:
- predictor1 = 'Disgust'
- predictor2 = 'Fear'
- outcome = 'KillRating'.

This way, if in the future we want to make changes, we don’t have to find each place where the variables are written in the program to rewrite them.
As mentioned above the kill rating is supposed to depend on the category of the bug. So, we want to have a new column indicating the bugs category. 
To assign each bug to a category, we will define a function that looks at the value of the disgust and fear column for each row and by combining them returns one of four possible categories (combinations) or None:
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
Now we want to apply this function to each row and save its results into new column called ‘bug_category’. The df.apply function applies a function along an axis of the DataFrame. As arguments we specify the function which will be applied and the axis (when axis= 1 or ‘columns’, the function is applied to each row).
```python
df['bug_category'] = df.apply (label_bug, axis=1)
```
Now we can also drop rows that have missing values in our new column since it doesn’t make much sense to keep a row that doesn’t contain the predictors value. However, statistically I am not sure if this is the right thing to do.

`df.dropna()` removes the rows (specified by the argument: axis=0) that have missing values in one column (specified by: subset=[‘bug_category’]).

```python
df = df.dropna(axis=0, subset=['bug_category'])
```
This is what df includes now:
```python
df
```
## Summary statistics

Often one of the first steps when analyzing some new data is to get an idea of the spread or 'distribution' of the different values in each column. We might want to know what the lowest and highest values are, what the most common values are, and so on Summarizing multiple values in one number is also known as 'aggregating' the values. The aggregate() method of a pandas.DataFrame returns a new shorter data frame containing only aggregated values. The first argument to aggregate() specifies what function or functions to use to aggregate the data.

If we want to apply multiple aggregation functions, then it is a little clearer to first gather the functions in a list variable, and then pass this list variable into the aggregate() method. 
```python
from statistics import median, mean, stdev

summary_stats = [min, median, mean, stdev
```

So, let’s check the summary statistics of the kill ratings (our outcome):
```python
summary = df.aggregate({outcome: summary_stats})
```



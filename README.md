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

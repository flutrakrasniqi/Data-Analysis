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

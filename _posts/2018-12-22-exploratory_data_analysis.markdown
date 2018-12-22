---
layout: post
title:      "Exploratory Data Analysis"
date:       2018-12-22 13:44:56 +0000
permalink:  exploratory_data_analysis
---


When you start a new project it can often be daunting. You know the intended outcome of the project and it is easy to narrow your focus on the task at hand. It is however important and exciting to spend more time in the Exploratory data analysis (EDA) stage as you get to know your data or even come back for more after you have completed your task.

EDA is simply an approach to summarize your variables through simple statics and visualizations. Your goal is develop a deeper understanding of the distribution of your data. Ideally this process would lead to some additional research questions or points of interest in the data set.  In a straight forward processing model like OSEMiN, the exploratory stage comes after you have scrubbed your data but before you start your modeling. Like many things in life, there is a lot of overlap between these stages and to be successful you may need to bounce between the stages often. 

I like to start my EDA process by computing histograms for my variables so I can get an ideas of their shape. Since your goal is to give you the best understanding of the data, I find it useful to look at the graphs first. You can then call .describe() on your data set which will return a table with your 5-point statistics, as well as the mean and standard deviation.

* 5-point statistics or five number summary refers to:
* Min: minimum value in the data set
* 1st Quartile: 25% of the data falls below this value.
* Median  (2nd Quartile): 50% of the data falls below this value
* 3rd Quartile: 75% of the data falls below this value
* Max: largest value in  the dataset

These statistics are commonly used to graph box plots which can give another look at the data and is an easy way to identify outliers.

I find scatter plots to be helpful to give me ideas for relationships between variable and what might be interested for further analysis. This is also a good way to start tying your EDA into your project, look for possible predictors, and if you have not done so already, remove any outliers. Seaborn has two great tools to use to analyse scatter plots. The first A is the seaborn pairplot. It returns a matrix of scatter plots for all data, which can get overwhelming but it allows you to look at all the variable at one time and how they interact with each other. The second tool is the seaborn jointplot which consists of a scatter, KDE, and distribution. It also plots a regression line.

Another great way to dive deeper into a set is to group the data by a categorical variable or create meaningful bins for a discrete variable. You can then simply return the mean for this categories or you can run the 5- number summary. Either way, grouping by a variable and graphing the results is a good way to see if there is anything of interest, it is also a good check from your scrubbing stage. 

I believe that EDA boils down mostly to an open approach when looking at a dataset. It should encourage you to think critically about what might be relevant but be open to interesting things that come along the way and willing to seek them out. 


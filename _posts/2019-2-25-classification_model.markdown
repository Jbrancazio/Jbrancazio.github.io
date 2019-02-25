---
layout: post
title:      Classification Model - Marketing for Term Deposit
date:       2019-2-25 11:11:45 +0000
permalink:  classification_model
---

> Classification uses a model of inputs to accurately predict the value of one or more outcomes. These outcomes are labels in a dataset and can either be binary or multi-categorical. 


There are multiple ways to organize a data project but one of the most straight forward processes is to use OSMIN. I will use this process to develop a model using marketing bank data maintained by [UCI](//archive.ics.uci.edu/ml/datasets/bank+marketing#), to predict whether or not a customer will subscribe to Term Deposit. My technical jupyter notebook can be found on my [GitHub](). 

## Obtain
I started my process by obtaining the data from the UCI website and doing some initial standard analysis to understand the makeup of the dataset. Some detailed notes about the features used are below:

* age: [numerical]
* job: Type of Job [categorical: "admin.","blue-collar","entrepreneur","housemaid","management","retired","self-employed","services","student","technician","unemployed","unknown"]
* marital: Marital status [categorical: "divorced","married","single","unknown"]
* education: Level of education [categorical: "primary","secondary","tertiary","unknown"]
* default: Has credit in default? [binary: "no","yes"]
* balance: Average yearly balance [numerical: in euros]
* housing: has housing loan? [binary: "no","yes"]
* loan: has personal loan? [binary: "no","yes"]
* contact: contact communication type [categorical: "cellular","telephone"]
* day: last contact day of the week [categorical: "mon","tue","wed","thu","fri"]
* month: last contact month of year [categorical: "jan", "feb", ..., "nov", "dec"]
* duration: last contact duration [numerical: in seconds]
* campaign: number of contacts performed during this campaign and for this client [numerical]
* pdays: number of days that passed by after the client was last contacted from a previous campaign [numerical]
* previous: number of contacts performed before this campaign and for this client [numerical]
* poutcome: outcome of the previous marketing campaign [categorical: "failure","nonexistent","success"]
* y: Whether or not the customer has term deposit [binary: "no","yes"]


## Scrub

This dataset was fairly neat so the scrub process was pretty straight forward. I started by turning the binary categories into values of 0:No and 1:Yes. I then one_hot_coded the categorical data and standardized all values with a mean of 0 and standard deviation of 1. 

One of the important aspects of following the OSMIN process is that it is a fluid process. After complete my initial model and looking at the important features I noticed that duration was by far the most important feature used in the model. 

![Feature Importance for Base Model](img/model1_feature_importance.png)

Further understanding of the duration variable however led me to ultimately remove it from further modeling. This is backed up by the research done by [Moro et al., 2014] which indicated that 

>this attribute highly affects the output target (e.g., if duration=0 then y='no'). Yet, the duration is not known before a call is performed. Also, after the end of the call y is obviously known. Thus, this input should only be included for benchmark purposes and should be discarded if the intention is to have a realistic predictive model.<sup id="a1">[1](#f1)</sup>

## Exploratory Data Analysis

In my mind, this step is more important than actually modeling because it is the work here that will help give you an understanding of the data that will allow you to interpret the results. 

## Model

When determining my model I decided to use a narrowing approach. I first created a variety of basic models including XGBoost Decision Tree, Random Forest, Logistical Regression and Support Vector Machine algorithms and compared their Accuracy and f1 scores. The highest accuracy was achieved by the SVM algorithm however the model was significantly slower than other options. For that reason I decided to use The XGBoost model as the accuracy was only slightly worse and the algorithm was significantly faster. Despite the high accuracy of 89.33%, the F1 score for the XGBoost model was 0.26. Our random forest model scored better in F1 with a score of 0.2921 with an accuracy of 88.96%

The accuracy of these models are promising however with such low F1 scores, there seem to be an issue. Upon further investigation, it seems that the fact that this dataset is so imbalanced in predictors is likely the culprit (no: , yes: ). 

I have solved this problem through using SMOTE: Synthetic Minority Over-Sampling Technique. This is an approach creates new "synthetic" observations that can be used to fit the model. I struggled with how to approach this step because I wanted to be able to train our data set without affecting the predictability of the algorithm. Ultimately, I decided to use the SMOTE process on our training data and keep out test data "pure". 

I then fit both the XGBoost model and Random Forest models using the new synthetic dataset and compared the F1 and accuracy scores again. This synthetic data raised the test data F1 score for both the Random Forest and XGBoost algorithms from 0.2921 to .3479 and 0.2696 to 0.4035 respectively. Based on this improvement, I decided to move forward with the XGBoost model and try to tune the parameters to improve both my Accuracy and F1 score. 









~~~
h3 = pd.read_sql_query('''SELECT c.categoryname categories, p.id as product, od.quantity quantity,
                            r.regiondescription regions, od. unitprice unit_price, od.Discount discount
                            FROM product p 
                            JOIN category c ON p.categoryId = c.Id
                            JOIN orderdetail od ON p.Id = od.productId
                            JOIN [order] o ON od.OrderId = o.Id
                            JOIN employee e ON o.EmployeeId = e.Id
                            JOIN employeeterritory et ON e.Id = et.employeeid
                            JOIN territory t ON et.territoryId = t.id
                            JOIN region r ON t.regionId = r.id
                            ;''', con)
~~~


<b id="f1">1</b> [Moro et al., 2014] S. Moro, P. Cortez and P. Rita. A Data-Driven Approach to Predict the Success of Bank Telemarketing. Decision Support Systems, Elsevier, 62:22-31, June 2014 [↩](#a1)
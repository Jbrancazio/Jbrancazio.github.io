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

* **age**: [numerical]
* **job**: Type of Job [categorical: "admin.","blue-collar","entrepreneur","housemaid","management","retired","self-employed","services","student","technician","unemployed","unknown"]
* **marital**: Marital status [categorical: "divorced","married","single","unknown"]
* **education**: Level of education [categorical: "primary","secondary","tertiary","unknown"]
* **default**: Has credit in default? [binary: "no","yes"]
* **balance**: Average yearly balance [numerical: in euros]
* **housing**: has housing loan? [binary: "no","yes"]
* **loan**: has personal loan? [binary: "no","yes"]
* **contact**: contact communication type [categorical: "cellular","telephone"]
* **day**: last contact day of the week [categorical: "mon","tue","wed","thu","fri"]
* **month**: last contact month of year [categorical: "jan", "feb", ..., "nov", "dec"]
* **duration**: last contact duration [numerical: in seconds]
* **campaign**: number of contacts performed during this campaign and for this client [numerical]
* **pdays**: number of days that passed by after the client was last contacted from a previous campaign [numerical]
* **previous**: number of contacts performed before this campaign and for this client [numerical]
* **poutcome**: outcome of the previous marketing campaign [categorical: "failure","nonexistent","success"]
* **y**: Whether or not the customer has term deposit [binary: "no","yes"]


## Scrub

This dataset was fairly neat so the scrub process was pretty straight forward. I started by turning the binary categories into values of 0:No and 1:Yes. I then one_hot_coded the categorical data and standardized all values with a mean of 0 and standard deviation of 1. 

One of the important aspects of following the OSMIN process is that it is a fluid process. After complete my initial model and looking at the important features I noticed that duration was by far the most important feature used in the model. Further understanding of the duration variable however led me to ultimately remove it from further modeling. This is backed up by the research done by [Moro et al., 2014] which indicated that this attribute highly affects the output target (e.g., if duration=0 then y='no'). Yet, the duration is not known before a call is performed. Also, after the end of the call y is obviously known. Thus, this input should only be included for benchmark purposes and should be discarded if the intention is to have a realistic predictive model.<sup id="a1">[1](#f1)</sup>

## Exploratory Data Analysis

In my mind, this step is more important than actually modeling because it is the work here that will help give you an understanding of the data that will allow you to interpret the results. 

## Model

When determining my model I decided to use a narrowing approach. I first created a variety of basic models including XGBoost Decision Tree, Random Forest, Logistical Regression and Support Vector Machine algorithms and compared their Accuracy and f1 scores. The highest accuracy was achieved by the SVM algorithm however the model was significantly slower than other options. For that reason I decided to use The XGBoost model as the accuracy was only slightly worse and the algorithm was significantly faster. Despite the high accuracy of 89.33%, the F1 score for the XGBoost model was 0.26. Our random forest model scored better in F1 with a score of 0.2921 with an accuracy of 88.96%

The accuracy of these models are promising however with such low F1 scores, there seem to be an issue. Upon further investigation, it seems that the fact that this dataset is so imbalanced in predictors is likely the culprit (no: 88%, yes: 12%). 

I have solved this problem through using SMOTE: Synthetic Minority Over-Sampling Technique. This is an approach creates new "synthetic" observations that can be used to fit the model. I struggled with how to approach this step because I wanted to be able to train our data set without affecting the predictability of the algorithm. Ultimately, I decided to use the SMOTE process on our training data and keep out test data "pure". 

I then fit both the XGBoost model and Random Forest models using the new synthetic dataset and compared the F1 and accuracy scores again. This synthetic data raised the test data F1 score for both the Random Forest and XGBoost algorithms from 0.2921 to .3479 and 0.2696 to 0.4035 respectively. Based on this improvement, I decided to move forward with the XGBoost model and try to tune the parameters to improve both my Accuracy and F1 score. 

My approach to tuning the parameters is to adjust two parameters at a time. This process takes some time to run especially if a lot of options are given so by splitting up the tuning parameters it allows me to analyze the data in-between runs. It was this approach that lead me to believe that I was over-fitting my model. 

During my first pass I used the following parameters to tune my model. 
~~~
#create paramater grid to loop through
param_grid = {
    'max_depth': [3,4,5,6],
    'n_estimators': [50, 100, 350, 500, 1000],
}

#use Gridsearch to loop through
grid_clf = GridSearchCV(clf_xgb, param_grid, scoring='roc_auc', cv=5, n_jobs=1)
grid_clf.fit(X_train_resampled, y_train_resampled)
~~~
This resulted in an increase of my testing accuracy and AUC <sub><sup>(Accuracy:95.97%, AUC: 0.96, F1: 0.9584)</sup></sub> as well as my training accuracy and AUC. There was however a decrease of my training F1 score <sub><sup>(Accuracy:89.32%, AUC: 0.74, F1: 0.3807)</sup></sub>.

From here I decided to continue tuning my parameters. I picked two more parameters I wanted to tune and  kept the last two constant. 
~~~
#adding some additional tuning parameters while keeping those we already found constant.
param_grid = {
    'gamma':[i/10.0 for i in range(0,3)],
    'max_depth': [6],
    'subsample':[i/10.0 for i in range(6,9)],
    'n_estimators': [500],
}

#Using GridsearchCV to loop
grid_clf = GridSearchCV(clf_xgb, param_grid, scoring='roc_auc', cv=5, n_jobs=1)
grid_clf.fit(X_train_resampled, y_train_resampled)
~~~
This resulted in another increase of my training scores <sub><sup>(Accuracy:96.17%, AUC: 0.96, F1: 0.9605)</sup></sub> but actually a decrease in my testing scores <sub><sup>(Accuracy:89.28%, AUC: 0.74, F1: 0.3765)</sup></sub>. This is where I started to suspect that I was over-fitting my model.

Despite my suspicions I decided to try to tune my model one more time using the following parameters.
~~~
#adding a few more parameters
param_grid = {
    'gamma':[0.1],
    'learning_rate': [0.001,0.01,0.1,0.2],
    'max_depth': [6],
    'min_child_weight': [4,6,8,10],
    'subsample':[0.6],
    'n_estimators': [500],
}

#loop through the fitting process
grid_clf = GridSearchCV(clf_xgb, param_grid, scoring='roc_auc', cv=5, n_jobs=1)
grid_clf.fit(X_train_resampled, y_train_resampled)
~~~
This resulted in another increase of my training scores <sub><sup>(Accuracy:96.55%, AUC: 0.97, F1: 0.9646)</sup></sub> but acnother decrease in my testing scores <sub><sup>(Accuracy:88.86%, AUC: 0.72, F1: 36.75)</sup></sub>. This is where I started to suspect that I was over-fitting my model.


At this point I had a final model that I was happy with in terms of my modeling ability so I decided to graph my ROC Curve and see if I could figure out what was going on. 
![ROC Curve of Training and Testing Predicitons for Model 1](img/ROC_Model_1.png)
As you can see by the ROC Curve, our model is strong when predicting our training data however is significantly weaker when predicting our testing data and only slightly better than chance. In other words, if we tune our training model to predict 95% accuracy in predicting true positives we will have a 12% chance of a false positive. When using new information, our model is much less impressive. If we tune our model to 50% true positive rate, we will have a 33% false positive rate. At this point, we have a good model for modeling purposes but it is a poor predictor.

I believe that this is due to overfitting problem I mentioned earlier and one way to potentially fix that issue is to use PCA(Principal Component Analysis). This will allow us to compress or reduce the dimensionality of our dataset which will speed up our process as well as potentially help our overfitting issue.

For this model, I will used PCA to reduce the dimensionality but retain 80% of the variance in the dataset. In order to do this I could shrink my model into 27 components. 
![ROC Curve of Training and Testing Predicitons for Model 2](img/ROC_Model_2.png)
As you can see from the graph above, the AUC decrease a little for the training data but increase for the testing data. Using the same points we used before. For a 95% true positive rate in the training data we actually see a decrease to only a 6% false positive rate. When it comes to testing the data, for a 50% true positive rate we see a decrease to 21% false positive rate.

Because we made progress using this method, I decided to retune my parameters to see if I can improved the ROC_AUC for my testing data.


<b id="f1">1</b> [Moro et al., 2014] S. Moro, P. Cortez and P. Rita. A Data-Driven Approach to Predict the Success of Bank Telemarketing. Decision Support Systems, Elsevier, 62:22-31, June 2014 [↩](#a1)

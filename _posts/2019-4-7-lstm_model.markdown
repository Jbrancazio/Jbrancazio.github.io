---
layout: post
title:      LSTM Model - Predicting Reddit Upvotes
date:       2019-4-7 11:11:45 +0000
permalink:  lstm_model
---

For my final module 4 project I have decided to try to use Natural Language Processing (NLP) and a Deep Neural Network to try to predict the "final score" of a reddit comment, i.e. the number of upvotes or downvotes. I have based this project off of a Kaggle project submission which can be found [here](https://www.kaggle.com/ehallmar/reddit-comment-score-prediction).

The data that was provided consists of two csv files, positive and negative scores, each containing 2,000,000 comments. For the purpose of this project, i have decided to randomly choose 100,000 comments from both files for my dataset. This will allow me to get a decent representation of the comments without overloading my system when cleaning data and running models.

Because I am working with NLP, one of the most important steps is to prepare the data for modeling. Normally the Scrubbing/Exploration steps will happen relatively consecutively however because of the nature of this data, I performed these steps concurently with an iterative process:

1. observe the data
2. find characters that need to be removed
3. add those characters to my cleaner function
4. rerun the cleaner on the text
5. repeat

Through this process, I also decided to clean and observe the parent comment data. This allowed me to explore more text data without needing to import more rows, and it also allowed me to compare the two dataset to see if there were differences. I made a few observations, found some characters that needed further exploration and made some decisions about leaving the data untouched.

I decided to leave "r/" in the data as it refers to other subreddits that might be useful information. Below is a word cloud of the most frequent subreddits used in the comment text. Since we are working with Reddit comments, I did try to hide some of the more offensive ones but you might still find some in the cloud. 
![Wordcould of most common subreddits](img/subreddit_wordcloud.png)

We can compare this to the parent wordcloud to get an idea of whether or not the data is similar.
![Wordcould of most common parent text subreddits](img/parent_subreddit_wordcloud.png)

Some other "interesting" things I found were:
* I explored the character "&gt" as it showed up as one of the most common characters. It turns out to just be a symbol for greater than in reddit so I removed it along with "&lt".

* There were a lot of url's in the data so I attempted to write a regex parser to find/replace those.

* There were some foreign characters so I attempted to write a regex parser for those as well.


# Modeling

There are a few different models I used during this process in the hopes of improving the accuracy of the data. I decided to predict the score of the comment. This means that the output could range from - ∞  to  ∞ . I decided to use an LSTM model with my Word2Vec embedding as the first layer. The following is a list of models I created:

1. Linear Model
  1. shallow model with fixed embedding layer
  2. shallow model with trainable embedding layer
  3. deeper model with one Dense(activation="Relu") and two Dropout layers
  4. deeper model with scaled output

2. Categorical Model
  1. deeper model with 6 categorical outputs and batch size 32
  2. deeper model with 6 categorical outputs and batch size 100
  3. added one more Dense(activation="Relu") and Dropout Layer

### Linear Model
My initial linear models all returned very disappointing results. The best of the 4 models was the deeper model with an added Dense Layer and two dropout layers although this model still returned a validation accuracy of 0.34%. 

![loss and accuracy of best linear model](img/1C_LSTM_Model.png)

### Categorical Model
I decided to change my model from a linear model to a categorical. This would hopefully help my performance as we would not have an  ∞  range of possibilities.

I decided to group my data into the following 6 categories:

High Negative: 100 ≤ x downvotes
Negative: 20 ≤ x ≤ 100 downvotes
Low Negative: 0 ≤ x ≤ 20 downvotes
Low Positive: 0 ≤ x ≤ 20 upvotes
Positive: 20 ≤ x ≤ 100 upvotes
High Positive: 100 ≤ x upvotes

# Interpret results

The final model is shown below. 
![Final Model](img/LSTM_Final_model.png)

With this model we achieved the following results. 
* Model Loss: 1.1824
* Model Accuracy: 48.23%
* Validation Loss: 1.211
* Validation Accuracy: 46.20%
* Test Loss: 1.204
* Test Accuracy: 47.14%

We can further test our results bu importing some current comments from Reddit and feeding them into our model, predicting the outcome. Two such examples are below.

>>Canada is in the data, but not on your chart. -2 internet points for you!

**Current score = 125**

* High Negative: 0.18%
* Negative: 7.06%
* **Low Negative: 36.3%**
* Low Positive: 0%
* Positive: 22.5%
* *High Positive: 33.8%*

According to our model, the most probible outcome is the low Negative group but in reality this comment belongs into the high positive group which has a probability only 3% behind Low Negative.

>>Could there be a more useless, confusing, and stupid, representation of 'whatever the OP was attempting to display? Yeah, no.

**Current Score= -25**

* High Negative: 0.15%
* *Negative: 5.9%*
* Low Negative: 30%
-----
* Low Positive: 0%
* Positive: 24.7%
* **High Positive: 38.7%**

In this example, our model predicts this comment would achieve a high positive rating however it achieved a negative rating. This example show the bias of our data at work. 



Our final model still has some room to improve. As you can see, from the graph below, our labels are not standardly distributed which biases our results. 
![label countplot](img/LSTM_Label_countplot.png)

This improvement can be done through improving the model but also can be done through improving our data. As you can see from the word clouds below, some of the most common words in the two groups are not actualy english words which will effect the accuracy of our model.
![Parent comments, most common words](img/parent_text_wordcloud.png) Another option is to import more than 200,000 examples to train the initial embedding and model.

## Next Steps
1. Work to improve our cleaner function to improve our data.
2. Add more inputs to our model such as parent comments, subreddit and maybe parent score
3. Combine the texts for word embedding to improve my weights
4. Import more data



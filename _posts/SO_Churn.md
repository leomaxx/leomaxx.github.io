---
layout: post
title: Stack Overflow Responder Churn Analysis
---

## Backgound:
If you ask me what is the most useful website? I would say, without hesitation, Stack Overflow! As a data scientist, I visit this website multiple times everyday for programming and data science questions. It has been a reliable source of answers to ALL my questions, until recently I came across a few instances where I cannot find the exact answer.

Did my questions become harder, or did Stack Overflow become less effective in atrracting answers? I have to find out...

## Dataset:
Stack Overflow is pretty open about its data and you can find thw whole site's user and post information in multiple locations: [Kaggle](https://www.kaggle.com/stackoverflow),  [Internet Archive](https://archive.org/download/stackexchange), and [Google BigQuery](https://cloud.google.com/bigquery/public-data/stackoverflow). The dataset includes all the public data on the site including:
  - User demographic information: id, age, about me, homepage, profile image
  - Information of all questions, answers, and comments including timestamp, id, and scores
  - Votes, tags, and wikis <br>
However, it does not contain the user browing activity data. It would be very informational but it comprimises user privacy.

## EDA:
Naturally, the first thing I checked is the monthly activity trend on the website. In particular, number of questions and answers posted every month in the last 5 years.

It it clear that the number of questions stayed stable, the number of answers has been dropping over the last 5 years. What's happending?

A common analysis technique for customer retention is **Cohort Analysis**. In this case, we are interested in finding out retention of responders. 

There are two observations in Cohort Analysis:
  - Less and less new users are answering question in the same year they signed up
  - Over the lifetime of a cohort, active responders stop answering questions

The reasons for the first observation may be complicated. It could be new users having troubles finding questions to answer, or they find it intimidating to provide an imperfect answer, or they do not have the motivation to answer questions. In order to improve the new user answering rate, SO could conduct user experience research, focus group study or send surveys to pinpoints the possible causes.

Though the second observation is expected (In a cohort analysis, retained user always reduce in a cohort), it is a bit subtle that the churn rate has increased. Take the first two years of cohort 2018 and 2016 for example, 87% responders stayed active in year 1, while only 53% of responders stayed active in year 1. 

**If we can effectively predict the responder churn, then the Site can do something to retain the users.**



## Problem Formalization:
In order to predict responder churn, there are a few things we need to agree on first:
  - Which subset of responder population do we want to retain the most?
  - What is the window size for responder churn?
  - How many months of activity data do we plan to use for the prediction?

### Focus Responder Group
If we plot the histogram of number of answers per responder between 2015 and 2017, we would see most responders have only 1 or 2 answers. In fact, 45% of the answers are contributed by 2% of the responder population. These are the savant responders who post more than 2 questions every month. Instead of predicting all the responders, we will focus on predicting this group.

### Churn Window Size
Churn window refers to the duration of time when the user is inactive, after which the user stays inactive. Of course this is an over-simplification of the real user patterns. There are users who visit the website in a cyclic manner，for example, every December (Santa Claus, is that you?). If we define the churn window as 6 months, then we would falsely announce in June that the user has churned. However, we cannot define churn window size as infinity as we would not have any reliable data to train the model. In addition, as window size increases, it becomes harder to predict if the user is going to churn. Therefore we need to strike a balance between accuracy and prediction power. Looking at the responder activity data between 2015 and 2017, here are the window size and accuracy rate:

| **Window Size** | **Accuracy Rate** |  
| --- | --- |
| 3 Months | 32% |   
| 4 Months | 44% |
| 5 Months | 55% |  
| **6 Months** | **67%** |  
| 7 Months | 72% |  
| 8 Months | 78% | 

It is up to business to decide which accuracy rate is adequate. In this project, we will choose churn window size to be 6 months. 

### Observation Window Size
Observation window refers to time duration that is observable to us, during which the user activity can be collected and used for prediction of their status in the following churn window. With a larger observation window, we will be able to collect more data and have stronger prediction power. One downside is that new user who joins the site shorter than the observation window will not be covered by the window. In this project, we select 12 months as the observation window.

### Churn Analysis Setup
I used user activity data between 2015 and 2017 for analysis. A sliding window of 18 months was applied to each user since they sign up. The data was further break down into 2 parts: The answering activity in the last 6 months is converted to a binary class: churn or not churn; Activities from the first 12 months are collected as features. These features include:
  - Number of questions posted by the user each month
  - Number of answers posted by the user each month
  - Number of comments posted by the user each month
  - Avg time taken by users to provide answers after question is posted
  - User demographics information (signup year, about me, personal website, profile image ..etc)
  
### Model Building
Multiple classifiers were trained and hyper-parameters were tuned to maximize the AUC Score. Specifically the following models were built:
| **Model Name** | **AUC Score** |  
| --- | --- |
| Random Forest | 0.82 |   
| Decision Tree | 0.85 |
| Logistic Regression | 0.87 |  
| Gradient Boosting Trees | 0.87 |  
| XGBoost | 0.87 |  
As a common practise to improve accuracy, a voting classifier was built to consolidate the results of all models. The voting classifier has a slightly higher AUC score of 0.871.

To convert the probability score to a binary class (churn/not churn), we need to define a threshold. In this business case, we are interested in predicting users who are going to churn as many as possible, and we are less concerned about unnecessarily reaching out to the users who wouls not churn.
Therefore, F2 score is chosen as the optmizing metric. A threshold (0.33) was chosen to maximize the F2 score and tranform the probability scores to binary class.

### User Retention Strategies
Once we have the list of users who are going to churn, we can take proactive measures to retain them. Some of the ideas include:
  - Send them targeted emails and updates
  - Invite them to answer high quality questions from their domain of expertise
  - Send them surveys to find out why they became inactive

### Future Improvements
Unfortunataly, user browsing history is not part of this public dataset. If such data is available, we will have much more information about each user and a lot more features can be extracted and improve the model accuracy.
Nature language processing is difficult for large volume of texts. Given more computational power, it would be interesting to see if there is any change in writing style or sentiment in the user posts before they churn.


# Babies: Fur and Otherwise #

## Problem Statement ##

The marketing mavens at ABC Toy Co. are dealing with a surprising challenge. It turns out that people who have small children (ABC's target demographic), and people who have dogs tend to talk about their... dependents in similar ways. As a result, ABC has wasted a fortune in internet advertising that turned out to be targeted towards people who don't have kids at all. In order to eliminate this wasteful spending, and better target their advertising towards the people most likely to buy their children's products, ABC has commissioned a data scientist to build a machine learning model that can distinguish between those who have actual human children, and those whose babies are of the fur variety.

While ABC is concerned about wasteful spending, they still view missing out on reaching potential customers as the greater evil. The ideal model, therefore, will optimize for sensitivity, while also taking accuracy into consideration.

### Methodology ###

#### Data ####

In order to train my models, I'll be looking at posts from two subreddits, one each on children and dogs. Specifically: 
- r/parenting
- r/dogs

I chose these two subreddits because their tone is similar, with posts largely concerning practical issues and advice related to the raising and rearing of children and dogs, respectively.

By accessing the **pushshift/api**, which is an api for **Reddit** posts and comments, I will assemble a sample dataset of 4000 total posts, 2000 from each subreddit. My final dataset will consist of these posts after culling duplicates.

The goal of my models will be to correctly identify which of the two subreddits each post comes from. Specifically, I'm interested in correctly identifying posts from the **r/parenting** subreddit, as those posting to this subreddit are potential customers that my client is interested in reaching.

#### Exploratory Data Analysis ####

During EDA I'll look into some of the characteristics of my data. Some things I'll look into:
- Post length:
    - How long is the average post?
    - Is there a relationship between post length and which subreddit the post comes from?
- Common Words:
    - After removing some meaningless words, what are some of the most common words used, both overall and by subreddit?
    - How often do words that are obvious differentiators appear?
        - *eg. how often does the word 'dog' appear in posts in the **r/dogs** subreddit*
        - I'll be looking to build a custom list of **crutch words** out of these common differentiators, for use in my models
        
#### Custom Stop Words - Crutch Words ####

I'm going to define a custom class of stop words that I'll call **'Crutch Words'**. This list includes some of the most popular and obvious differentiator words from each subreddit, such as 'dog' and 'kids'. I'll do versions of each model with and without the list of **'Crutch Words'**.

I expect removing the **'Crutch Words'** to negatively impact model performance, but I am interested to see to what extent that is the case

## Modelling ##

I'll build four different types of models:
1) Logistic Regression
2) Random Forest
3) Gradient Boost
4) Support Vector Machine

For each model type I'll build two sets of models:
1) The first set will be given the corpus of data with the **'Crutch Words'** still included
2) The second will be given data with the **'Crutch Words'** removed

For each model I'll use **GridSearch** to determine best parameters

## Model Performance Analysis and Visualizations ##

As I ran various iterations of models, I ran a function on each that computed various performance metrics, then saved these metrics in a dataframe

#### Data Dictionary - Model Performance Metrics DataFrame ####

|Feature|Type|Range of Values|Description|
|---|---|---|---|
|model_type|object|Logistic Regression, Random Forest, Gradient Boost, or Service Vector Machine|The type of model|
|vectorizer|object|CountVectorizer or Tfidf Vectorizer|The vectorizer used to transform language into model readable data|
|transform_type|object|Original (No Transformation), Stemmed, Lemmatized|Type of transformation done to the text to reduce words to roots|
|crutch_words_removed|int|Yes, No|Whether or not the list of common **crutch words** was removed from the data|
|true_positives|int|Countable|Posts from the test data correctly identified by the model as from **r/parenting**|
|true_negatives|int|Countable|Posts from  the test data correctly identified by the model as from **r/dogs**|
|false_positives|int|Countable|Posts from the test data *incorrectly* identified by the model as from **r/parenting**l|
|false_negatives|int|Countable|Posts from  the test data *incorrectly* identified by the model as from **r/dogs**|
|train_accuracy|float|between 0 and 1|Accuracy score on the training data|
|accuracy|float|between 0 and 1|Ratio of total correctly identified posts to total posts|
|sensitivity|float|between 0 and 1|Ratio of total correctly identified posts from **r/parenting** to total posts from **r/parenting**|
|specificity|float|between 0 and 1|Ratio of total correctly identified posts from **r/dogs** to total posts from **r/dogs**|
|precision|float|between 0 and 1|Ratio of total correctly identified posts from **r/parenting** to total posts identified as from **r/parenting**|

### Production Model - Gradient Boost ###

For my production model I've chosen the **Gradient Boost** model with **crutch words** allowed.

The client has made clear that they would rather incorrectly target some advertising towards childless adults then to miss out on reaching potential customers. I've therefore selected the model that best optimizes for **sensitivity**, which is **Gradient Boost**.

## Conclusions ##

Using advanced machine learning techniques, I was able to build a model capable of differentiating between social media posts from parents talking about their children, and pet lovers talking about their dogs. This model should greatly aid **ABC Toy Co.** in targeting their online advertising towards the former group, eliminating wasteful spending while also ensuring that their ads are reaching their target demographic.

Although some models showed better performance on other metrics, I selected **Gradient Boost** as the best candidate for a production model. **Gradient Boost** displayed the best performance on the key **sensitivity** metric, correctly identifying 98.1% of the posts from the **r/parenting** subreddit in the test data.

While it would not make business sense to implement a model while denying it access to the best data, the **Support Vector Machine** model's performance on the data with **crutch words** removed was still notable. Even without access to some of the best differentiator words in the corpus of data, it was still able to correctly identify 91.4% of the posts from **r/parenting** in the test data, the only model to achieve a sensitivity score over 90% on this data.

## Recommendations ##

The high performing **Gradient Boost** model could be put into production in it's current form, and it would provide a significant boost to the effectiveness of ABC's ad targeting. That said, there are several ways in which the model could be further refined to improve performance even more:
- **More Data**: Model performance improves the more data it has to train on. This model was trained using a little more than 3,000 **Reddit** posts covering a time period of 14 days from 2/17/2023 to 3/2/2023. **Reddit** is a treasure trove of this kind of data, and those numbers could easily be expanded to hundreds of thousands or even millions of posts over several years. Comments on posts could be added to the mix as well
- **Ensembling**: Though **Gradient Boost** was the highest performing model on the **sensitivity** metric, many of the other models performed nearly as well. These models could be combined together to achieve even greater performance
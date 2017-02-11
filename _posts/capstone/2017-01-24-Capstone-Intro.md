---
layout: post
title: Introducing the Capstone Project
category: capstone
---
# Categorizing News Articles Using NLP

This is first installment of a series of posts documenting my progress on my longest and most in-depth analysis to date.

## Backstory
For this analysis, I will take a deep dive into the world of Natural Language Processing in order to classify news articles based on the type of language they use. Given the rise of so-called "fake news" and "alternative facts", the Facebook algorithms that preferentially show a user  content that aligns with their viewing preferences, and resulting "echo chamber" effect on users' newsfeeds, I've been thinking a lot about the intersection of machine learning, text classification, and journalism.

Initially, I was curious to see if there were readily detectable linguistic differences in news articles from reputable sources and those from the onslaught of .co look-alike sites that have been created with the goal of deceiving readers. However, I found myself entering a gray area - what to do about sites like Breitbart.com? Although I personally disagree with essentially any piece published there, the site falls into a strange middle zone between opinion and actual news. Some of the facts may be true, but they're presented through a very extreme lens. Additionally, many of the knock-off news sites offer a relatively sparse article list - they just don't publish a large enough corpus. This problem is certainly a challenging one, probably beyond the scope of this project, so I adjusted my sights to a simpler task.

## Problem Statement

My goal for this project is to classify news articles by subject matter. At the most basic level, I hope to build a classifier that can differentiate between factual reporting and opinion pieces when trained on a corpus of articles published by the *New York Times*. If this is successful, I hope to increase predictive power in order to differentiate between business, world news, technology, science, sports, or arts articles.

## Risks and Assumptions

I've chosen my source based on its journalistic merit and large corpus, but I recognize that this model may not generalize to news from other sources. My results will likely also be particular to this particular moment in time, since I'm utilizing articles from January 2017 onwards.

## Data Acquisition

The first step of this project is to acquire and store the articles I'll use to build my corpus. I've web-scraped article titles and text, as well as meta-data such as author, date of publication, and section (business, science, etc). Since I'm restricting this task to actual article content, I've ignored comments sections, clicks and shares, and publication metadata. These data are stored in a local postgres database before processing and analysis in Pandas.


![png](../../../images/database_screenshot.png)


## Exploratory Data Analysis

Borrowing heavily from a [helpful tutorial](http://bbengfort.github.io/tutorials/2016/05/19/text-classification-nltk-sckit-learn.html) by Ben Bengfort, I've built a preprocessing pipeline that uses lemmatization and word tokenization from the NLTK library and prepares data for use in Scikit-Learn's Naive Bayes estimator.

Of the 751 articles I have so far, 17% of them are opinion pieces. This means 83% of them are non-opinion and my model will have to exceed the baseline of 83% accuracy.

![png](../../../images/titles_class_report.png)

The classification report for the title text only has an accuracy rate of 61%, but it looks as though the model is performing very poorly on categorizing opinion pieces.

![png](../../../images/bodies_class_report.png)

The articles bodies fared slightly better, with 74% accuracy, though opinion pieces are still a challenge.

## Word Clouds

I constructed word clouds from opinion and non-opinion article titles using a [word cloud library](https://github.com/amueller/word_cloud).

Here are the words from opinion titles:

![png](../../../images/opinion_big.png)

And here are the words from non-opinion titles:

![png](../../../images/non_opinion_big.png)

Obviously, "Trump" occurs with great frequency in both categories, but "Donald" appears more "bigly" in the opinion pieces. This may be because non-opinion pieces typically refer to him as Mr. Trump or President Trump, and opinion pieces are more informal. Additionally, "health" seems to appear more frequently among opinion titles, and "immigration" more frequently among non-opinion titles. Perhaps there are more opinion pieces relating to health or healthcare, and more reporting on immigration issues.

## Next Steps

As I continue to acquire more articles, I'm considering expanding to bring in another news source, such as the *Washington Post*. I am also interested in perhaps looking at some articles from *The Onion* to see if there are linguistic differences between satire and genuine news.

My next steps will be to construct n-grams to see if any relevant features arise that can improve model accuracy, and explore different model types.

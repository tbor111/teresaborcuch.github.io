---
layout: post
title: Sentiment Analysis
category: capstone
tags: [sentiment analysis, capstone, NLP]
---
# EDA: Sentiment Analysis

After about a week and a half of dedicated web-scraping, I've amassed a corpus of articles from three sources: *The New York Times* (1,425 articles), *The Washington Post* (447 articles), and FoxNews.com (309 articles). I'm aiming to have at least 1,000 articles from each source, but I'll start examining trends in the data I currently have.

Sentiment analysis is a natural language processing technique that seeks to identify the polarity of written text - whether the document expresses positive, negative, or neutral feelings. This technique is commonly applied to text that may express strong opinions, such as reviews or social media posts, but I'm going to apply it to news articles and see if there are any differences in the sentiments expressed by different publications or in different sections.

The goal for this analysis is to derive scores for each article's body and title based on the positivity or negativity of the words they contain, and see if these score might be useful features in predicting what section the article is from. Sentiment scores range from 1 (very positive) to -1 (very negative), and I'm expecting a relatively small range for articles in this corpus, since news is presumably objective. [SentiWordNet](http://sentiwordnet.isti.cnr.it/) is a resource that maps tens of thousands of English words to a sentiment score. Since language is complex, and the meaning of a given word can vary significantly depending on the context in which it's used, these scores are imperfect, but they'll provide a general picture of the sentiment behind each article.

## ArticleData Class

I've been scraping each of my three sources at least twice a day to get fresh news and storing articles in a local postgres database. To facilitate the process of retrieving data and automate some of the cleaning and formatting, I've created a service class ArticleData. Objects of this class have a call method that creates a dataframe of article titles, dates, bodies, sections, and sources. Any articles that have a body of under 200 characters in length are dropped, all dates converted to datetime format, and the various sections from each source are condensed into business, world, politics, science/health, technology, sports, education, entertainment, or opinion. (To see code for this class and anything else related to this project, see my [github](https://github.com/teresaborcuch/capstone_project).)

## Sentiment Scores

Since I'll be looking at sentiment scores, ArticleData objects also calculate scores for article bodies and titles, and store them in columns in the dataframes they return when called. I've borrowed the compute_score function from [this Github gist](https://gist.github.com/rtkgupta/da5a7c5b66383d8db2b1) and written it into the call method. This function works by using NLTK's WordNetLemmatizer to lemmatize each word in a text, and the PerceptronTagger to tag the part of speech of each word in the text as a noun, verb, adjective, or adverb. This helps SentiwordNet to assign the word to a synset, or synonym set, for which it can assign a positive or negative value. Since a word can belong to multiple synsets, the function selects the most common one.


```python
from articledata import *
```

```python
data = ArticleData().call()

data.shape
```




    (2322, 9)




```python
data.head(1)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>date</th>
      <th>body</th>
      <th>section</th>
      <th>source</th>
      <th>condensed_section</th>
      <th>SA_body</th>
      <th>SA_title</th>
      <th>SA_diff</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>$5 Million for a Super Bowl Ad. Another Millio...</td>
      <td>2017-01-29</td>
      <td>This month, Anheuser-Busch InBev hosted a doze...</td>
      <td>business</td>
      <td>NYT</td>
      <td>business</td>
      <td>0.01624</td>
      <td>-0.023148</td>
      <td>0.039388</td>
    </tr>
  </tbody>
</table>
</div>



## Trends in Sentiment Scores


```python
plt.scatter(data['SA_title'], data['SA_body'])
plt.xlabel('Sentiment Scores of Article Titles')
plt.ylabel('Sentiment Scores of Article Bodies')
plt.xlim(-0.35, 0.35)
plt.ylim(-0.35, 0.35)
plt.show()
```


![png](../../../images/second_blog_post_files/second_blog_post_9_0.png)


This plot shows the relationship between title scores and body scores for all articles. Interestingly, there seems to be little relationship between the sentiment score of the title and that of the body. The other feature of note is that article titles have a much wider score range than the bodies. This may be because the compute_score function averages the score of each word over the entire length of the document. Since article bodies contain many more words, and therefore more neutral words, their scores will be lower.


```python
mask1 = (data['source'] == 'Fox')
mask2 = (data['source'] == 'WP')
mask3 = (data['source'] == 'NYT')

score_dict = {'Fox': data[mask1]['SA_body'],
           'WP': data[mask2]['SA_body'],
           'NYT' : data[mask3]['SA_body']}

sns.set_style("whitegrid", {'axes.grid' : False})
fig, ax = plt.subplots(figsize = (10,6))
for x in score_dict.keys():
    ax = sns.distplot(score_dict[x], kde = False, label = x)
    ax.set_xlabel("Distribution of Sentiment Scores by Publication")
ax.legend()
ax.set_xlim(-0.1, 0.1)
plt.show()
```


![png](../../../images/second_blog_post_files/second_blog_post_11_0.png)


This histogram shows the distribution of article bodies' sentiment scores for all three publications. As I suspected, they are normally distributed tightly around zero.


```python
pd.DataFrame(data.pivot_table(
        values = ['SA_title', 'SA_body'],
        index = 'condensed_section',
        aggfunc =np.mean)).sort_values('SA_body', ascending = False)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>SA_body</th>
      <th>SA_title</th>
    </tr>
    <tr>
      <th>condensed_section</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>entertainment</th>
      <td>0.009675</td>
      <td>0.005642</td>
    </tr>
    <tr>
      <th>education</th>
      <td>0.007797</td>
      <td>0.005974</td>
    </tr>
    <tr>
      <th>technology</th>
      <td>0.005968</td>
      <td>0.001077</td>
    </tr>
    <tr>
      <th>sci_health</th>
      <td>0.005271</td>
      <td>0.000586</td>
    </tr>
    <tr>
      <th>business</th>
      <td>0.004649</td>
      <td>0.005713</td>
    </tr>
    <tr>
      <th>sports</th>
      <td>0.001776</td>
      <td>-0.004155</td>
    </tr>
    <tr>
      <th>other</th>
      <td>0.000342</td>
      <td>0.003976</td>
    </tr>
    <tr>
      <th>opinion</th>
      <td>0.000138</td>
      <td>-0.011153</td>
    </tr>
    <tr>
      <th>politics</th>
      <td>-0.000470</td>
      <td>-0.002535</td>
    </tr>
    <tr>
      <th>world</th>
      <td>-0.006127</td>
      <td>-0.011486</td>
    </tr>
  </tbody>
</table>
</div>



Let's see which sections are the most positive and negative. This table contains the average sentiment scores of bodies and titles for each section, sorted by highest-scoring body to lowest. The averages are clustered around zero, but entertainment and education have the most positive content in their article bodies, and articles from the world news setion have the most negative.


```python
mask1 = (data['condensed_section'] == 'opinion')
mask2 = (data['condensed_section'] == 'entertainment')
mask3 = (data['condensed_section'] == 'world')

score_dict = {'opinion': data[mask1]['SA_body'],
           'entertainment': data[mask2]['SA_body'],
           'world' : data[mask3]['SA_body']}

sns.set_style("whitegrid", {'axes.grid' : False})
fig, ax = plt.subplots(figsize = (10,6))
for x in score_dict.keys():
    ax = sns.distplot(score_dict[x], kde = False, label = x)
    ax.set_xlabel("Distribution of Sentiment Scores by Section")
ax.legend()
ax.set_xlim(-0.1, 0.1)
plt.show()
```


![png](../../../images/second_blog_post_files/second_blog_post_15_0.png)


This histogram shows the distribution of scores for the article bodies from three sections: opinion, world, and entertainment. The entertainment section has more articles scoring above zero than the other two sections, and the world news section has fewer. This makes sense, since the entertainment section is composed of stores about "lighter" topics such as the arts, dining, or travel, and the world news section might be a bit "heavier". It also seems like the opinion section has a wider range of scores than the other two.

## Evaluating Sentiment For Particular Topics

It might be interesting to investigate whether certain publications differ in the sentiment of their reporting on particular topics, or whether the sentiment for a particular topic varies among sections of the same publication. I've written a function that aggregates the sentiment scores for articles from a particular publication or section into a dictionary with two keys: topic and non-topic. These dictionaries are readily plottable, and I can compare among sections and sources.


```python
# create dictionaries for each publication
nyt_trump = evaluate_topic(data = data, section = 'opinion', source = 'NYT', topic = 'Trump')
fox_trump = evaluate_topic(data = data, section = 'opinion', source = 'Fox', topic = 'Trump')
wp_trump = evaluate_topic(data = data, section = 'opinion', source = 'WP', topic = 'Trump')
```


```python
# plot dictionaries
sns.set_style("whitegrid", {'axes.grid' : False})
fig = plt.figure(figsize = (12, 5))
count = 1
label_dict = {1: "NYT", 2: "Fox", 3: "Washington Post"}

for score_dict in [nyt_trump, fox_trump, wp_trump]:
    ax = fig.add_subplot(1, 3, count)
    for key in score_dict.keys():
        ax.hist(score_dict[key], label = key)
        ax.set_xlabel(label_dict[count])
    ax.legend()
    ax.set_xlim(-0.1, 0.1)

    count +=1

plt.suptitle('Sentiment Scores for Opinion Articles Mentioning "Trump"')
plt.show()
```


![png](../../../images/second_blog_post_files/second_blog_post_19_0.png)


The obvious observation from these graphs is that more opinion pieces mention Trump than not for all three publications, and the sentiment of the Trump-related articles spans a wider range than non-Trump-related articles.


```python
# Compare sentiment scores for opinion vs non-opinion pieces that mention education
ed_dict = evaluate_topic(data = data, section = 'opinion', topic = 'education')

for key in ed_dict.keys():
    sns.distplot(ed_dict[key], kde = False, label = key)
plt.legend()
plt.show()

print 'Mean Score for Opinion Pieces Mentioning "education": ', np.mean(ed_dict['topic'])
print 'Mean Score for Opinion Pieces Not Mentioning "education": ', np.mean(ed_dict['nontopic'])
```


![png](../../../images/second_blog_post_files/second_blog_post_21_0.png)


    Mean Score for Opinion Pieces Mentioning "education":  0.00412250407705
    Mean Score for Opinion Pieces Not Mentioning "education":  -0.000467516117571


Here, I'm comparing opinion pieces from all three publications that mention education to those that don't. Opinion articles about education have a higher rating on average than those that don't, across all publications.

## EvaluateTime Class

I thought it might also be informative to look at how sentiment surrounding a particular topic changes over time. If I can plot the ups and downs, I might be able to pinpoint particular events from headlines or articles that correspond to the fluctuations in sentiment. I've written another service class, that when called, yields a dictionary of sentiment scores for all article bodies that mention a particular topic. You can also specify a source, section, and date to narrow the search, and then plot the results with min/max error bars with the plot_time method.

```python
et = EvaluateTime(data = data, source = 'Fox', section = 'politics', topic = 'Trump', date = datetime(2017, 1, 24)).call()
et.plot_time()
```


![png](../../../images/second_blog_post_files/second_blog_post_26_0.png)



```python
et = EvaluateTime(data = data, source = 'NYT', section = 'politics', topic = 'Trump', date = datetime(2017, 1, 24)).call()
et.plot_time()
```


![png](../../../images/second_blog_post_files/second_blog_post_27_0.png)


These two graphs show the fluctuation in sentiment in articles that contain the word 'Trump' after January 1, 2017. The first shows politics articles from FoxNews.com, and the second from *The New York Times* politics section. Although the Fox library of political Trump articles is a little sparse to make comparisons between the two, we can see on *The New York Times* graph that January 31st marked a spike of positive articles mentioning Trump.


## Next Steps

Next, I'm hoping to work on interactive graphs using D3 that would allow me to show which titles are associated with highs and lows on an EvaluateTime plot, and also see if Named Entity Recognition yields any interesting features of my data.

---
layout: post
title: Categorizing News Articles with NLP
category: capstone
tags: [NLP, capstone, pipeline, sentiment analysis, named entity recognition, Naive Bayes]
---
# The Final Capstone Write-up

# I. Introduction

In this post, I'll summarize the exploratory data analyses I performed, explain the feature engineering and reduction steps I utilized, and present my final models to classify news articles.

## A. Problem Statement

The goal of this project was to classify news articles into business/technology, entertainment, opinion, politics, science/health, sports, or world news categories using only features derived from text.

## B. Risks and Assumptions
To make a general classifier that would be effective on an article from any source from any era is a huge undertaking. Given the available training data, this one is specific to the topics in the news from three sources in early 2017. A very general classifier would require frequent re-training to combat "subject drift" [1]. For instance, given the recent presidential election, Donald Trump is featured prominently in almost every newspaper section, but this almost certainly wasn't the case one or two years ago, and may not be the case in the future. As new political figures rise in prominence, Donald Trump may become a more or less informative person for classifying a politics or world news article.

I also relied heavily on tagged datasets and tools developed by the Stanford NLP group. Any inaccuracies that are built into the out-of-the-box versions of the WordNetLemmatizer, Part-of-Speech-Tagger, or NERTagger are therefore built into my own analysis. I believe it is possible to make case-specific modifications to how certain names or locations are tagged, but that was beyond the scope of this project.

## C. Overview
From a corpus of 4,846 news articles from three sources, I extracted a vocabulary of 5,000 terms, derived named entity counts and sentiment scores, and used these to train Naive Bayes and Stochastic Gradient Descent classifiers to perform with ~85% accuracy in a seven-class discrimination task.

# II. Data Acquisition and Cleaning

## A. Webscraping
Between January 23 - February 18, 2017, I web-scraped full-text articles from *The New York Times* (2,646), *The Washington Post* (1,357), and FoxNews.com (843). I would typically run the scrapers at least twice a day in order to get a fresh batch of articles, since the sites seemed to use Javascript to continuously refresh new articles at the end of the page rather than listing sequential pages going backward in time, and archives were generally inaccessible. The FoxNews site was particularly difficult to work with, as its 'Opinion' page and 'Latest' page were structured differently from general article pages, necessitating three different scrapers to get articles in each category. In addition, the *New York Times* site required cookie storage, so I implemented a CookieJar object from cookielib as a workaround. For each article, I recorded the title, author, date of publication, source, whatever section the source designated it to come from, and the url.

I stored the articles in a local PostgreSQL database. Each scraper would deposit articles into a staging table, from which only unique articles would be moved into a particular table for that source. I used titles to determine whether or not a particular article had been already scraped and stored, as I assumed a source wouldn't publish two stories with the same title in timespan of this project.


```python
# NytArticleGrabber.py
# example webscraper for New York Times articles
#!/usr/bin/env python

import sys
import pandas as pd
from bs4 import BeautifulSoup
import cookielib
import urllib2
import requests
from sqlalchemy import create_engine
from sqlalchemy.orm.session import sessionmaker
import re

def make_soup(url):
    #request = urllib2.Request(url)
    #response = urllib2.urlopen(url)
    response = requests.get(url)
    html = response.content
    soup = BeautifulSoup(html.decode('utf-8', 'ignore'), 'html.parser')

    #soup = BeautifulSoup(urlopen(url).read().decode('utf-8', 'ignore'), 'html.parser')
    results = soup.find_all('div', attrs = {'class': 'story-body'})
    print response.status_code
    return results

def make_dict(results):

    # initialize lists
    titles = []
    dates = []
    links = []
    full_texts = []
    authors = []
    sections = []

    # scrape results into lists
    for x in results:
        # get link
        link = x.find('a')['href']

        # set regex to eliminate interactive features
        match = re.search('^https://www.nytimes.com/20', link)
        if match:
            links.append(link)

    # resoup it
    for link in links:
        all_p = ''
        cj = cookielib.CookieJar()
        opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cj))
        new_soup = BeautifulSoup(opener.open(link).read().decode('utf-8', 'ignore'), 'html.parser')

        # get the article content
        body = new_soup.find_all('p', attrs = {'class': 'story-body-text story-content'})
        for p in body:
            new_p = p.text.strip()
            all_p = all_p + new_p
        full_texts.append(all_p)

        # get titles
        title = new_soup.find('meta', attrs = {'property': 'og:title'})['content']
        titles.append(title)

        # get authors
        author = new_soup.find('meta', attrs = {'name': 'author'})['content']
        authors.append(author)

        # get sections
        section = new_soup.find('meta', attrs = {'name': 'CG'})['content']
        sections.append(section)

        # get dates
        date = new_soup.find('meta', attrs = {'name': 'pdate'})['content']
        dates.append(date)

    data_dict = {
        'title': titles, 'link': links, 'author': authors, 'body': full_texts,
        'section': sections, 'date': dates
    }
    return data_dict

def make_dataframe(data_dict):
    df = pd.DataFrame(data_dict)
    return df


def main():
    section_list = [
    'us','health','opinion','science','sports',
    'arts','world','technology']
    for x in section_list:
        url = 'http://www.nytimes.com/section/' + x
        results = make_soup(url)
        data_dict = make_dict(results)
        df = make_dataframe(data_dict)

        # create engine
        engine = create_engine('postgresql://teresaborcuch@localhost:5433/capstone')
        Session = sessionmaker(bind=engine)
        session = Session()

        # clear staging
        clear_staging_query = 'DELETE FROM nyt_staging *;'
        engine.execute(clear_staging_query)
        session.commit()

        # add df to staging
        df.to_sql('nyt_staging', engine, if_exists = 'append', index = False)

        # move unique rows from staging to ny_times
        move_unique_query = '''
        INSERT INTO ny_times (title, date, author, body, link, section)
        SELECT title, date, author, body, link, section
        FROM nyt_staging
        WHERE NOT EXISTS (SELECT title, date, author, body, link, section
        FROM ny_times
        WHERE ny_times.title = nyt_staging.title);
        '''
        engine.execute(move_unique_query)
        session.commit()
        session.close()

        print "{} done".format(x)


if __name__ == "__main__":
    main()

```

## B. Data Cleaning and Standardization

Since each source has its own uniquely named sections with multiple subsections, I had to judge which articles should be grouped together. Initially, I had eleven categories: world news, politics, entertainment, business, technology, science, health, sports, opinion, education, and other. I decided to combine science and health, as many of the science articles were related to the human body anyways, and merged business and technology, as most of the business articles were about technology companies. I dropped 'other' from analysis, since this was an amalgam of local news, obituaries, and other miscellany, as well as education, due to low numbers. This left me with the final seven categories: world, politics, entertainment, science/health, business/technology, sports, and opinion.

To handle this operation, merge the three tables from the database, and to conduct other maintenance cleaning tasks, like standardizing encoding, dropping the "| Fox News" suffix from certain titles, resetting the index, and dropping articles that were too short, I created a utility class **ArticleData**. Upon instantiation, an **ArticleData** object pulls all articles currently in the database, formatted their sections as described, and returns a dataframe containing title, date, condensed source, full-text article body, and the original source.


```python
class ArticleData():

    '''
    Usage:
    >>> from articledata import *
    >>> data = ArticleData().call()
    >>> data = get_sent_scores(data = data)
    >>> topic_data = evaluate_topic(data = data, section = 'opinion', source = 'NYT', topic = 'healthcare')
    >>> data = count_entities(data = data, title = True)
    >>> people_dict, place_dict = evaluate_entities(data = data, section = 'opinion', source = 'NYT')
    '''


    def __init__(self): pass

    def call(self):
        engine = create_engine('postgresql://teresaborcuch@localhost:5433/capstone')

        # get data from all three tables and add source column
        query1 = "SELECT DISTINCT ON(title) title, date, body, section FROM fox_news;"
        fox_data = pd.read_sql(query1, engine)
        fox_data['source'] = ['Fox']*len(fox_data)

        query2 = "SELECT DISTINCT ON(title) title, date, body, section FROM ny_times;"
        nyt_data = pd.read_sql(query2, engine)
        nyt_data['source'] = ['NYT'] * len(nyt_data)

        query3 = "SELECT DISTINCT ON(title) title, date, body, section FROM washington_post;"
        wp_data = pd.read_sql(query3, engine)
        wp_data['source'] = ['WP']*len(wp_data)

        # merge the dataframes into one big one
        data = pd.concat([nyt_data, fox_data, wp_data], axis = 0)

        # drop those with empty or suspiciously short bodies
        problem_rows = []
        for i, row in data.iterrows():
            try:
                if len(row[2]) < 200:
                    problem_rows.append(row.name)
            except TypeError:
                problem_rows.append(row.name)

        data = data.drop(data.index[problem_rows])

        # fix the dates
        new_dates = []
        for x in data['date']:
            if type(x) == int:
                x = str(x)
                x = (x[:4] + '-' + x[4:6] + '-' + x[6:8]).replace(' 00:00:00','')
                x = datetime.strptime(x, '%Y-%m-%d')
                new_dates.append(x)
            else:
                x = str(x).replace(' 00:00:00','')
                x = datetime.strptime(x, '%Y-%m-%d')
                new_dates.append(x)

        data['date'] = new_dates

        # eliminate | Fox News from titles

        clean_titles = []
        for x in data['title']:
            match = re.search('\|.*$', x)
            if match:
                clean_x = re.sub('\|.*$','',x)
                clean_titles.append(clean_x)
            else:
                clean_titles.append(x)
        data['title'] = clean_titles


        # create the condensed sections

        def condense_section(x):
            if 'world' in x:
                section = 'world'
            elif 'pinion' in x:
                section = 'opinion'
            elif ('business' in x) or ('tech' in x):
                section = 'bus_tech'
            elif ('entertain' in x) or ('art' in x) or ('theater' in x) or ('book' in x) or ('movie' in x) or ('travel' in x) or ('fashion' in x) or ('style' in x) or ('dining' in x):
                section = 'entertainment'
            elif 'sport' in x:
                section = 'sports'
            elif ('health' in x) or ('science' in x) or ('well' in x):
                section = 'sci_health'
            elif ('education' in x):
                section = 'education'
            elif ('politic' in x) or ('us' in x):
                section = 'politics'
            else:
                section = 'other'
            return section

        data['condensed_section'] = [condense_section(x) for x in data['section']]

        # drop other and education sections

        mask1 = data['condensed_section'] != 'other'
        mask2 = data['condensed_section'] != 'education'
        data = data[mask1 & mask2]

        return data

```


```python
# Example usage of ArticleData
from articledata import ArticleData
data = ArticleData().call()
```


```python
data.head()
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>#GrammysSoWhite Came to Life. Will the Awards ...</td>
      <td>2017-02-13</td>
      <td>Before the rapper Q-Tip of A Tribe Called Ques...</td>
      <td>arts</td>
      <td>NYT</td>
      <td>entertainment</td>
    </tr>
    <tr>
      <th>1</th>
      <td>$5 Million for a Super Bowl Ad. Another Millio...</td>
      <td>2017-01-29</td>
      <td>This month, Anheuser-Busch InBev hosted a doze...</td>
      <td>business</td>
      <td>NYT</td>
      <td>bus_tech</td>
    </tr>
    <tr>
      <th>2</th>
      <td>$60,000 in Tuition, and My Son Wants to Become...</td>
      <td>2017-01-12</td>
      <td>My wife and I are spending a fortune to send o...</td>
      <td>fashion</td>
      <td>NYT</td>
      <td>entertainment</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1 Patient, 7 Tumors and 100 Billion Cells Equa...</td>
      <td>2016-12-07</td>
      <td>The remarkable recovery of a woman with advanc...</td>
      <td>health</td>
      <td>NYT</td>
      <td>sci_health</td>
    </tr>
    <tr>
      <th>6</th>
      <td>14 TV Shows That Broke Ground With Gay and Tra...</td>
      <td>2017-02-16</td>
      <td>Last year was a remarkable time when it came t...</td>
      <td>arts</td>
      <td>NYT</td>
      <td>entertainment</td>
    </tr>
  </tbody>
</table>
</div>



# III. Feature Engineering

## A. Sentiment Analysis

Sentiment analysis approximates the overall polarity of a text by scoring its positive, negative and neutral words on a scale from -1 to 1. To score each article title or body, I used a compute_score function that lemmatizes each word using NLTK's WordNetLemmatizer, and then tags it with its part of speech using the PerceptronTagger. This allows each word to be matched to a synset, or synonym set in SentiwordNet, a resource that maps thousands of English words to their sentiment scores. Although a particular word can potentially belong to multiple synsets depending on its usage in a sentence, I opted to just select the first one.

The compute_score function can be applied to data directly for exploratory data analysis of sentiment, or the data can be transferred to a SentimentPlotter object (see Section IV.C Plotting with Highcharts, below).


```python
def compute_score(sentence):
        tagger = PerceptronTagger()
        taggedsentence = []
        sent_score = []
        taggedsentence.append(tagger.tag(sentence.split()))
        wnl = nltk.WordNetLemmatizer()
        for idx, words in enumerate(taggedsentence):
            for idx2, t in enumerate(words):
                newtag = ''
                lemmatizedsent = wnl.lemmatize(t[0])
                if t[1].startswith('NN'):
                    newtag = 'n'
                elif t[1].startswith('JJ'):
                    newtag = 'a'
                elif t[1].startswith('V'):
                    newtag = 'v'
                elif t[1].startswith('R'):
                    newtag = 'r'
                else:
                    newtag = ''
                if (newtag != ''):
                    synsets = list(swn.senti_synsets(lemmatizedsent, newtag))
                    score = 0.0
                    if (len(synsets) > 0):
                        for syn in synsets:
                            score += syn.pos_score() - syn.neg_score()
                        sent_score.append(score / len(synsets))
            if (len(sent_score)==0 or len(sent_score)==1):
                return (float(0.0))
            else:
                return (sum([word_score for word_score in sent_score]) / (len(sent_score)))
    data['SA_body'] = [compute_score(x) for x in data['body']]
    data['SA_title'] = [compute_score(x) for x in data['title']]

    return data
```

## B. Named Entity Recognition

Named entity recognition seeks to identify people, places, times, dates, or other elements from text. I utilized the Stanford NLP group's three-class model and an NLTK wrapper to identify people and places in each article. Additionally, NLTK comes with a corpus of tagged male and female names, so I counted how many men and women were mentioned in article titles and bodies. I wrote a count_entities function that tokenizes a text, uses Stanford's NERTagger to tag all people and places, and then counts them. However, I have to note that if a name isn't within the name corpus, it's possible for the function recognize and count people, but not know whether they are male or female, so these counts have varying levels of accuracy.

Additionally, to track which people and places were most common in each section, I wrote an evaluate_entities function that returns a dictionary of each entity and its frequency.


```python
def count_entities(data = None, title = True):
    # set up tagger
    os.environ['CLASSPATH'] = "../../stanford-ner-2013-11-12/stanford-ner.jar"
    os.environ['STANFORD_MODELS'] = '../../stanford-ner-2013-11-12/classifiers'
    st = StanfordNERTagger('english.all.3class.distsim.crf.ser.gz')

    tagged_titles = []
    persons = []
    places = []

    if title:

        for x in data['title']:
            tokens = word_tokenize(x)
            tags = st.tag(tokens)
            tagged_titles.append(tags)

        for pair_list in tagged_titles:
            person_count = 0
            place_count = 0
            for pair in pair_list:
                if pair[1] == 'PERSON':
                    person_count +=1
                elif pair[1] == 'LOCATION':
                    place_count +=1
                else:
                    continue
            persons.append(person_count)
            places.append(place_count)


        data['total_persons_title'] = persons
        data['total_places_title'] = places

    else:
        for x in data['body']:
            tokens = word_tokenize(x)
            tags = st.tag(tokens)
            tagged_titles.append(tags)

        for pair_list in tagged_titles:
            person_count = 0
            place_count = 0
            for pair in pair_list:
                if pair[1] == 'PERSON':
                    person_count +=1
                elif pair[1] == 'LOCATION':
                    place_count +=1
                else:
                    continue
            persons.append(person_count)
            places.append(place_count)

        data['total_persons_body'] = persons
        data['total_places_body'] = places

    return data

def evaluate_entities(data = None, section = None, source = None):
    section_mask = (data['condensed_section'] == section)
    source_mask = (data['source'] == source)

    if section and source:
        masked_data = data[section_mask & source_mask]

    elif section:
        masked_data = data[section_mask]

    elif source:
        masked_data = data[source_mask]

    else:
        masked_data = data

    # set up tagger
    os.environ['CLASSPATH'] = "/Users/teresaborcuch/stanford-ner-2013-11-12/stanford-ner.jar"
    os.environ['STANFORD_MODELS'] = '/Users/teresaborcuch/stanford-ner-2013-11-12/classifiers'
    st = StanfordNERTagger('english.all.3class.distsim.crf.ser.gz')
    # dictionaries to hold counts of entities
    person_dict = {}
    place_dict = {}

    for x in masked_data['body']:
        tokens = word_tokenize(x)
        tags = st.tag(tokens)
        for pair in tags:
            if pair[1] == 'PERSON':
                if pair[0] not in person_dict.keys():
                    person_dict[pair[0]] = 1
                else:
                    person_dict[pair[0]] +=1
            elif pair[1] == 'LOCATION':
                if pair[0] not in place_dict.keys():
                    place_dict[pair[0]] = 1
                else:
                    place_dict[pair[0]] += 1

    return person_dict, place_dict
```


```python
# Example usage of evaluate_entities function
world_people, world_places = evaluate_entities(data = data, section = 'world', source = None)
```


```python
# function to find the top 200 places
def get_top_200(my_dict):
    sorted_dict = sorted(my_dict.items(), key = operator.itemgetter(1))
    sorted_dict.reverse()
    top_entries = sorted_dict[:200]
    return top_entries
```

```python
top_200 = get_top_200(world_places)
top_200[:5]
```

    [(u'United', 1081),
     (u'States', 991),
     (u'U.S.',675),
     (u'Iran', 422),
     (u'Russia', 414)]




## C. Custom Transformers

To actually derive these features for article titles or bodies, I wrote two transformer classes, **TitleTransformer** and **BodyTransformer**. Each of these classes inherits the methods from Scikit-Learn's BaseEstimator and TransformerMixin classes, which allows them to be utilized in pipelines. Using the fit_transform method from one of these scalers on an **ArticleData** object returns a dataframe containing sentiment scores, as well as the number of people, number of men and women, and number of places in each article.


```python
class TitleTransformer(BaseEstimator, TransformerMixin):
    def fit(self, X, y = None):
        return self

    def transform(self, X):
        persons, places, males, females = count_entities(X['title'])
        ss = np.asarray([compute_score(x) for x in X['title']])

        title_array = map(list,(zip(persons, places, males, females, ss))) #ss

        return pd.DataFrame(title_array, columns = ['title_persons', 'title_places',
                                                    'title_men', 'title_women', 'title_ss'])

class BodyTransformer(BaseEstimator, TransformerMixin):
    def fit(self, X, y = None):
        return self

    def transform(self, X):

        ss = [compute_score(x) for x in X['body']]
        persons, places, males, females = count_entities(X['body'])

        body_array = map(list,(zip(persons, places, males, females,ss)))
        return pd.DataFrame(body_array, columns = ['body_persons', 'body_places',
                                                   'body_men', 'body_women', 'body_ss'])
```


```python
# Example of TitleTransformer in action
from newsanalyzer import TitleTransformer, BodyTransformer

test_data = data[:5]

tt = TitleTransformer()
test_data_tt = tt.fit_transform(test_data)
test_data_tt
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title_persons</th>
      <th>title_places</th>
      <th>title_men</th>
      <th>title_women</th>
      <th>title_ss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.052484</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.023148</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.041667</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>-0.034722</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.084028</td>
    </tr>
  </tbody>
</table>
</div>




```python
# BodyTransformer in action
bt = BodyTransformer()
test_data_bt = bt.fit_transform(test_data)
test_data_bt
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>body_persons</th>
      <th>body_places</th>
      <th>body_men</th>
      <th>body_women</th>
      <th>body_ss</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>42</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>-0.000025</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17</td>
      <td>8</td>
      <td>0</td>
      <td>0</td>
      <td>0.016240</td>
    </tr>
    <tr>
      <th>2</th>
      <td>13</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>0.020668</td>
    </tr>
    <tr>
      <th>3</th>
      <td>24</td>
      <td>9</td>
      <td>0</td>
      <td>0</td>
      <td>0.000946</td>
    </tr>
    <tr>
      <th>4</th>
      <td>80</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0.027508</td>
    </tr>
  </tbody>
</table>
</div>



## D. Tf-Idf Vectors

The vast majority of the feature set are Tf-Idf vectors from Scikit-Learn's TfidfVectorizer. Tf-Ddf stands for "Term Frequency-Inverse Document Frequency" and weighs the frequency with which a word appears in a given document against how common it is in the entire corpus. So a word that occurs very often in one document, but is rare in the corpus would have a higher Tf-Idf value.

The TfidfVectorizer has parameters for preprocessing, stopwords, and a n-gram range. I modified a custom preprocessor, the 'tokenize' function, that utilizes NLTK's lemmatizing capabilities, eliminates punctuation and stopwords, and strips accents [2]. Lemmatization is a process that reduces words to their lemmas, or base forms. For instance, lemmatization would reduce "is", "am" and "were" to "be", as they are all diverse forms of the same root verb "be".

I used ngram_range = (1,2) to include both bigrams and unigrams, and min_df = 10 to exclude words that appear fewer than 10 times in the corpus.


```python
# preprocessor function
def tokenize(text):
    text = text.encode('ascii','ignore')
    lemmas = []

    def lemmatize(token, tag):
        tag = {
            'N': wn.NOUN,
            'V': wn.VERB,
            'R': wn.ADV,
            'J': wn.ADJ
        }.get(tag[0], wn.NOUN)
        wnl = WordNetLemmatizer()
        return wnl.lemmatize(token, tag)

    for token, tag in pos_tag(wordpunct_tokenize(text)):
        token = token.lower()
        token = token.strip()
        token = token.strip('_')
        token = token.strip('*')

        if token in sw.words('english'):
            continue
        if all(char in string.punctuation for char in token):
            continue

        lemma = lemmatize(token, tag)
        lemmas.append(lemma)
        lemma_string = ' '.join(lemmas)

    return lemma_string

# parameters used for TfidfVectorizer
tfidf = TfidfVectorizer(preprocessor = tokenize, ngram_range = (1,2), min_df = 10)
```

# IV. Exploratory Data Analysis

## A. Sentiment Scores

Although sentiment scores can range from -1 to 1, I found that the scores of articles from each section only ranged from about -0.1 to 0.1. This makes sense, since the news aims for impartial reporting. Within this range, world news had the lowest average score (-0.005), while entertainment had the highest (0.009). A t-test indicated that this difference, though minuscule, was statistically significant (t = -21.60, p << 0.005, df = 1702).



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_25_0.png)



## B. Named Entities

Overall, every section had more total (not unique) mentions of people than places, and of those, vastly more mentions of men than women. Sports, politics, and entertainment had the most person mentions, while world news had the most places. Entertainment had the most mentions of women.

By far, variants of America, the United States, and the U.S. were the most common place mentioned, and Donald Trump was the most common person in every section but sports.



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_27_0.png)




![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_28_0.png)



![png](../../../images/trump_count.png)



## C. Plotting with HighCharts

I thought it might be interesting to plot the fluctuation in sentiment of a particular topic over time. The **SentimentPlotter** class extracts all scores of articles mentioning a topic for the selected source and section, and returns them as Javascript arrays ready for insertion into a Highcharts script. The resulting graph shows the mean score for articles by day, as well as error bars for the minimum and maximum scores on that day. Mousing over the end of an error bar will reveal the title of the article that earned the high or low score. In addition the plot_topic function does the same thing, but produces a static image using the Seaborn plotting library.


```python
#!/usr/bin/env python

import numpy as np
import pandas as pd
import json
from sqlalchemy import create_engine
from datetime import datetime, timedelta
import psycopg2
import seaborn as sns
import matplotlib.pyplot as plt
from nltk.sentiment.util import *
from nltk.corpus import sentiwordnet as swn
from nltk.tag.perceptron import PerceptronTagger
from nltk.tag import StanfordNERTagger
from nltk.tokenize import word_tokenize
import os
import re

class SentimentPlotter():
    def __init__(self, data = None):
        self.data = data

    def call(self):

        self.data['SA_body'] = [self.compute_score(x) for x in self.data['body']]
        self.data['SA_title'] = [self.compute_score(x) for x in self.data['title']]

        return self

    def get_highcharts_series(self, section = None, source = None, topic = None, date = None):
        self.section = section
        self.source = source
        self.topic = topic
        self.date = date

        self.range_date_dict, self.groupings = self.make_dict()

        self.x_dates, self.y_means, self.error_pairs, self.date_list = self.get_plotting_data()

        self.spline_series = self.get_spline_series()

        self.error_bar_series = self.get_error_bar_series()

        self.min_titles, self.max_titles = self.get_titles()

        self.min_scatter_series, self.max_scatter_series = self.get_scatter_points()

        return self.spline_series, self.error_bar_series, self.min_scatter_series, self.max_scatter_series


    def compute_score(self, sentence):
        tagger = PerceptronTagger()
        taggedsentence = []
        sent_score = []
        taggedsentence.append(tagger.tag(sentence.split()))
        wnl = nltk.WordNetLemmatizer()
        for idx, words in enumerate(taggedsentence):
            for idx2, t in enumerate(words):
                newtag = ''
                lemmatizedsent = wnl.lemmatize(t[0])
                if t[1].startswith('NN'):
                    newtag = 'n'
                elif t[1].startswith('JJ'):
                    newtag = 'a'
                elif t[1].startswith('V'):
                    newtag = 'v'
                elif t[1].startswith('R'):
                    newtag = 'r'
                else:
                    newtag = ''
                if (newtag != ''):
                    synsets = list(swn.senti_synsets(lemmatizedsent, newtag))
                    score = 0.0
                    if (len(synsets) > 0):
                        for syn in synsets:
                            score += syn.pos_score() - syn.neg_score()
                        sent_score.append(score / len(synsets))
            if (len(sent_score)==0 or len(sent_score)==1):
                return (float(0.0))
            else:
                return (sum([word_score for word_score in sent_score]) / (len(sent_score)))


    def make_dict(self):
        # define masks
        section_mask = (self.data['condensed_section'] == self.section)
        source_mask = (self.data['source'] == self.source)
        date_mask = (self.data['date'] > self.date)

        # initialize lists for range_date_dict
        topic_scores = []
        dates = []
        groupings = []

        # initialize dict
        range_date_dict = {}

        if not self.date:
            print "Please select a start date."

        # make plot_date_dict from appropriate subset of data
        else:
            if self.section and self.source:
                masked_data = self.data[section_mask & source_mask & date_mask]

            elif self.section and (not self.source):
                masked_data = self.data[section_mask & date_mask]

            elif self.source and (not self.section):
                masked_data = self.data[source_mask & date_mask]

            else:
                masked_data = self.data[date_mask]

            for i, row in masked_data.iterrows():

                if self.topic in row[2]:
                    topic_scores.append(row[6]) #body score
                    dates.append(row[1])
                    # score_title_date is tuple containing the title, date, and score
                    score_title_date = (row[0], row[1], row[6])
                    #groupings is a list of all the tuples
                    groupings.append(score_title_date)


                    # add to range_date_dict where keys are the dates and the values are a list of scores
                    if row[1] not in range_date_dict.keys():
                        range_date_dict[row[1]] = [row[6]]

                    elif row[1] in range_date_dict.keys():
                        (range_date_dict[row[1]]).append(row[6])

        return range_date_dict, groupings




    def get_plotting_data(self):
        # get dates for x-axis
        date_list = self.range_date_dict.keys()
        date_list.sort()
        # format date for javascript
        x_dates = [x.value// 10 ** 6 for x in date_list]

        # y-values
        y_values = [np.mean(self.range_date_dict[x]) for x in date_list] # mean score for the spline series

        # error bars
        error_min_max = []
        for x in date_list:
            temp_list = []
            minimum = min(self.range_date_dict[x])
            maximum = max(self.range_date_dict[x])
            temp_list.append(minimum)
            temp_list.append(maximum)
            error_min_max.append(temp_list) #error_min_max is a list of lists containing minimum and maximum

        return x_dates, y_values, error_min_max, date_list

    def get_spline_series(self):
        # format splines for Highcharts
        d = []
        series = {'name': 'Mean Score', 'type': 'spline'}
        for x in range(len(self.date_list)):
            data_point = [self.x_dates[x], self.y_means[x]]
            d.append(data_point)
        series['data'] = d
        spline_series = json.dumps(series)
        return spline_series

    def get_error_bar_series(self):
        d = []
        series = {'color': '#FF0000', 'name': 'Range', 'type': 'errorbar', 'stemWidth': 3, 'whiskerLength': 0}
        for x in range(len(self.date_list)):
            data_point = [self.x_dates[x], self.error_pairs[x][0], self.error_pairs[x][1]]
            d.append(data_point)
        series['data'] = d
        error_series = json.dumps(series)
        return error_series

    def get_titles(self):
        min_score_titles = {}
        max_score_titles = {}

        # min scores
        for x in self.groupings:
            if x[1] not in min_score_titles.keys():
                min_score_titles[x[1]] = (x[2], x[0])
            elif x[1] in min_score_titles.keys():
                if x[2] < min_score_titles[x[1]][0]:
                    min_score_titles[x[1]] = (x[2], x[0])
                elif x[2] >= min_score_titles[x[1]][0]:
                    continue

        # max scores
        for x in self.groupings:
            if x[1] not in max_score_titles.keys():
                max_score_titles[x[1]] = (x[2], x[0])
            elif x[1] in max_score_titles.keys():
                if x[2] > max_score_titles[x[1]][0]:
                    max_score_titles[x[1]] = (x[2], x[0])
                elif x[2] <= max_score_titles[x[1]][0]:
                    continue

        min_titles = [min_score_titles[x][1].encode('ascii', 'ignore') for x in self.date_list]
        max_titles = [max_score_titles[x][1].encode('ascii','ignore') for x in self.date_list]

        return min_titles, max_titles

    def get_scatter_points(self):
        max_series = []

        for x in range(len(self.date_list)):
            data_point = {'showInLegend': False, 'type': 'scatter', 'color': '#FF0000',
                          'marker': {'symbol': 'circle', 'enabled': True, 'color': '#FF0000'},
                          'tooltip': {'pointFormat': '{point.y}'}}

            data_point['name'] = self.max_titles[x]

            data_list = [[self.x_dates[x], self.error_pairs[x][1]]]

            data_point['data'] = data_list
            max_series.append(data_point)

        max_series = json.dumps(max_series)

        # return minimum scatter points series
        min_series = []

        for x in range(len(self.date_list)):
            data_point = {'showInLegend': False, 'type': 'scatter', 'color': '#FF0000',
                          'marker': {'symbol': 'circle', 'enabled': True, 'color': '#FF0000'},
                          'tooltip': {'pointFormat': '{point.y}'}}
            # get title
            data_point['name'] = self.min_titles[x]

            data_list = [[self.x_dates[x], self.error_pairs[x][0]]]

            data_point['data'] = data_list

            min_series.append(data_point)

        min_series = json.dumps(min_series)

        return min_series, max_series

    def plot_topic(self, topic = None, section = None, source = None, date = None):
        self.topic = topic
        self.section = section
        self.source = source
        self.date = date

        self.range_date_dict, self.groupings = self.make_dict()

        x = self.range_date_dict.keys()
        x.sort()
        ordered_x = []
        y = []
        for val in x:
            ordered_x.append(val)
            values = self.range_date_dict[val]
            mean = np.mean(values)
            y.append(mean)

        # define upper and lower boundaries for error bars
        upper_bounds = [max(self.range_date_dict[x]) for x in ordered_x]
        lower_bounds = [min(self.range_date_dict[x]) for x in ordered_x]

        # define distance for upper error bar
        y_upper = zip(y, upper_bounds)
        upper_error = [abs(pair[0] - pair[1]) for pair in y_upper]

        # define distance for lower error bar
        y_lower = zip(y, lower_bounds)
        lower_error = [abs(pair[0] - pair[1]) for pair in y_lower]

        asymmetric_error = [lower_error, upper_error]

        plt.plot(ordered_x, y, c = 'r', marker = 'o')
        plt.errorbar(ordered_x, y, yerr = asymmetric_error, ecolor = 'r', capthick = 1)
        plt.xlim(min(ordered_x) + timedelta(days = -1), max(ordered_x) + timedelta(days = 1))
        plt.xticks(rotation = 70)
        plt.show()

```


```python
# Example of SentimentPlotter object
# Returns spline series, error bar series, and scatter series
# ready for insertion into a script for plotting in Highcharts
sp = SentimentPlotter(data = data).call()
sp.get_highcharts_series(topic = 'Trump', date = datetime(2017,1,1))
```




    ('{"data": [[1483920000000, 0.00356272085255046], [1484179200000, 0.0056750113931406769], [1484265600000, 0.0028177523013720655], [1484611200000, 0.00089692887829575141], [1484697600000, 0.0016938879181849532], [1484784000000, -0.0048885450132428055], [1485043200000, 0.0012365967584631883], [1485216000000, 0.0043572349995608349], [1485302400000, 0.0036356197931765705], [1485388800000, -0.0026853165943573178], [1485475200000, 0.0022216736935552475], [1485561600000, -0.0018449943424716717], [1485648000000, -0.0022071860108271025], [1485734400000, -0.001529851958070017], [1485820800000, 0.00033406832864152618], [1485907200000, -0.0021746902284161683], [1485993600000, 0.0019484273186078137], [1486080000000, -0.00054120343005551808], [1486166400000, 0.00064082078054331093], [1486252800000, 0.00012753565516900202], [1486339200000, -1.748578231061307e-06], [1486425600000, 0.001551793808283743], [1486512000000, 0.00050055047068814135], [1486598400000, 0.0024633171591991222], [1486684800000, -1.16813077454858e-05], [1486771200000, 0.00044292252702777597], [1486857600000, 0.0039779527046775341], [1486944000000, 0.0015299931949532308], [1487030400000, -0.0015147819383065882], [1487116800000, 0.0007730856110698183], [1487203200000, 0.00047831026709830226], [1487289600000, -0.00098409976870834177], [1487376000000, 0.0036847395534617107], [1487462400000, -0.00061544716371609751]], "type": "spline", "name": "Mean Score"}',
     '{"whiskerLength": 0, "name": "Range", "color": "#FF0000", "data": [[1483920000000, 0.00356272085255046, 0.00356272085255046], [1484179200000, -0.00316513079969771, 0.014515153585979063], [1484265600000, 0.0028177523013720655, 0.0028177523013720655], [1484611200000, 0.0008969288782957514, 0.0008969288782957514], [1484697600000, -0.01406176632325732, 0.01582098300399639], [1484784000000, -0.0048885450132428055, -0.0048885450132428055], [1485043200000, 0.0012365967584631883, 0.0012365967584631883], [1485216000000, -0.004563800772505719, 0.011479183330534931], [1485302400000, -0.008473489645253119, 0.017641738988055614], [1485388800000, -0.03759215114091276, 0.01565565850590797], [1485475200000, -0.022690103242414637, 0.025077837397613833], [1485561600000, -0.015022964961774108, 0.02031316270058932], [1485648000000, -0.05419147153643498, 0.05242308555732057], [1485734400000, -0.07546673756351176, 0.0420177890622497], [1485820800000, -0.020138447055517678, 0.04326670335292343], [1485907200000, -0.022384127718062474, 0.021054953967588108], [1485993600000, -0.02332528907958725, 0.031418040040256626], [1486080000000, -0.030780827943724907, 0.03606674158261045], [1486166400000, -0.021891711652034677, 0.023102555090620785], [1486252800000, -0.03789034091473219, 0.03494342932466825], [1486339200000, -0.043060092097680726, 0.06012818901788827], [1486425600000, -0.028583708524322453, 0.0396643728080506], [1486512000000, -0.030349070877080105, 0.03757202154187034], [1486598400000, -0.036716269864498996, 0.028079521551545566], [1486684800000, -0.03715878315951397, 0.03281162943269168], [1486771200000, -0.06518599914981495, 0.02625167563685734], [1486857600000, -0.025044570265700627, 0.0685440626390607], [1486944000000, -0.05057232405623877, 0.025571849913560984], [1487030400000, -0.03067282594768871, 0.04311561239882547], [1487116800000, -0.03212079380292969, 0.02762014760358772], [1487203200000, -0.033716166645854154, 0.04936556024770029], [1487289600000, -0.032768404880582525, 0.029091922449511618], [1487376000000, -0.021484298781604537, 0.0310567626295596], [1487462400000, -0.021307848189584307, 0.027127567865876424]], "stemWidth": 3, "type": "errorbar"}',
     '[{"name": "Betsy DeVos, Trumps Education Pick, Plays Hardball With Her Wealth", "color": "#FF0000", "data": [[1483920000000, 0.00356272085255046]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Pick for Education Could Face Unusually Stiff Resistance", "color": "#FF0000", "data": [[1484179200000, -0.00316513079969771]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Donations Pour In to Band From Black College That Will Play at Inauguration", "color": "#FF0000", "data": [[1484265600000, 0.0028177523013720655]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Betsy DeVoss Education Hearing Erupts Into Partisan Debate", "color": "#FF0000", "data": [[1484611200000, 0.0008969288782957514]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Student Loan Collector Cheated Millions, Lawsuits Say", "color": "#FF0000", "data": [[1484697600000, -0.01406176632325732]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Obama decision to free FALN terror group leader sparks outrage ", "color": "#FF0000", "data": [[1484784000000, -0.0048885450132428055]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Undersea Museum Keeps Fish Feeding and Its Social Commentary Biting", "color": "#FF0000", "data": [[1485043200000, 0.0012365967584631883]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Climate Change Activists Barefoot Walk Across America Ends in Tragedy", "color": "#FF0000", "data": [[1485216000000, -0.004563800772505719]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "That Old Phone Trump Uses for Twitter Could Be an Opening to Security Threats", "color": "#FF0000", "data": [[1485302400000, -0.008473489645253119]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Democrats, Play Hardball on a Supreme Court Nominee", "color": "#FF0000", "data": [[1485388800000, -0.03759215114091276]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trumps Immigration Ban Is Illegal", "color": "#FF0000", "data": [[1485475200000, -0.022690103242414637]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Anxiety About Muslim Refugees Is Stoked Online by the Far-Right Media", "color": "#FF0000", "data": [[1485561600000, -0.015022964961774108]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Outcry Over Trumps Refugee Ban", "color": "#FF0000", "data": [[1485648000000, -0.05419147153643498]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Most Thankless Job: Trumps Press Secretary", "color": "#FF0000", "data": [[1485734400000, -0.07546673756351176]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Republican Fausts", "color": "#FF0000", "data": [[1485820800000, -0.020138447055517678]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Everyone Is Welcome Here", "color": "#FF0000", "data": [[1485907200000, -0.022384127718062474]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Foreigners Trapped in the United States by New Policy", "color": "#FF0000", "data": [[1485993600000, -0.02332528907958725]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "A Real and Present Danger: Lawyers Protest Against Trump", "color": "#FF0000", "data": [[1486080000000, -0.030780827943724907]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Chelsea Nears a Championship by Excelling at the Predictable", "color": "#FF0000", "data": [[1486166400000, -0.021891711652034677]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Five Reasons the Opposition Is in Good Shape to Fight Trump", "color": "#FF0000", "data": [[1486252800000, -0.03789034091473219]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "President Trumps Real Fear: The Courts", "color": "#FF0000", "data": [[1486339200000, -0.043060092097680726]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "If Betsy DeVos is not confirmed it will be a national embarrassment  ", "color": "#FF0000", "data": [[1486425600000, -0.028583708524322453]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "She Showed Up Yearly to Meet Immigration Agents. Now Theyve Deported Her.", "color": "#FF0000", "data": [[1486512000000, -0.030349070877080105]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Hateful Threats Against a Jewish Blogger", "color": "#FF0000", "data": [[1486598400000, -0.036716269864498996]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Shutting down fake news could move us closer to a modern-day 1984", "color": "#FF0000", "data": [[1486684800000, -0.03715878315951397]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Apple CEO Tim Cook: \'Fake news\' is \'killing people\'s minds\' ", "color": "#FF0000", "data": [[1486771200000, -0.06518599914981495]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Review: A Hairy Visionary Who Has the Rulers Ear", "color": "#FF0000", "data": [[1486857600000, -0.025044570265700627]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Michael Flynns Phone Call With Russia", "color": "#FF0000", "data": [[1486944000000, -0.05057232405623877]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Justice must pass: French president makes rare appearance in Paris suburbs to address riots", "color": "#FF0000", "data": [[1487030400000, -0.03067282594768871]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Is fear the Trump administrations immigration enforcement goal?", "color": "#FF0000", "data": [[1487116800000, -0.03212079380292969]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Our Polarized Politics", "color": "#FF0000", "data": [[1487203200000, -0.033716166645854154]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Veterans have a problem that no amount of parades or VA funding can solve", "color": "#FF0000", "data": [[1487289600000, -0.032768404880582525]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Republicans Charge Into Resistance at Tumultuous Town Halls", "color": "#FF0000", "data": [[1487376000000, -0.021484298781604537]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump says remark about Sweden referred to something on TV", "color": "#FF0000", "data": [[1487462400000, -0.021307848189584307]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}]',
     '[{"name": "Betsy DeVos, Trumps Education Pick, Plays Hardball With Her Wealth", "color": "#FF0000", "data": [[1483920000000, 0.00356272085255046]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Janet Yellen Gives an Economic Short Course, Beyond Interest Rates", "color": "#FF0000", "data": [[1484179200000, 0.014515153585979063]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Donations Pour In to Band From Black College That Will Play at Inauguration", "color": "#FF0000", "data": [[1484265600000, 0.0028177523013720655]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Betsy DeVoss Education Hearing Erupts Into Partisan Debate", "color": "#FF0000", "data": [[1484611200000, 0.0008969288782957514]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Nominee Betsy DeVoss Knowledge of Education Basics Is Open to Criticism", "color": "#FF0000", "data": [[1484697600000, 0.01582098300399639]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Obama decision to free FALN terror group leader sparks outrage ", "color": "#FF0000", "data": [[1484784000000, -0.0048885450132428055]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Undersea Museum Keeps Fish Feeding and Its Social Commentary Biting", "color": "#FF0000", "data": [[1485043200000, 0.0012365967584631883]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Mens Fashion Designers Grapple With the Rise of Trump", "color": "#FF0000", "data": [[1485216000000, 0.011479183330534931]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Repeal and Compete", "color": "#FF0000", "data": [[1485302400000, 0.017641738988055614]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Elon Musk Has Trumps Ear, and Wall Street Takes Note", "color": "#FF0000", "data": [[1485388800000, 0.01565565850590797]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Homespun Kalashnikovs", "color": "#FF0000", "data": [[1485475200000, 0.025077837397613833]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "The Normalization Trap", "color": "#FF0000", "data": [[1485561600000, 0.02031316270058932]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "La La Land Takes Top Prize at the Producers Guild Awards", "color": "#FF0000", "data": [[1485648000000, 0.05242308555732057]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Dont Cut Back on the Arts", "color": "#FF0000", "data": [[1485734400000, 0.0420177890622497]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Full Transcript and Video: Trump Picks Neil Gorsuch for Supreme Court", "color": "#FF0000", "data": [[1485820800000, 0.04326670335292343]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Runner-Up Didnt Make It to Supreme Court, but He Did Get to Altoona", "color": "#FF0000", "data": [[1485907200000, 0.021054953967588108]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump Rude Toward Australia", "color": "#FF0000", "data": [[1485993600000, 0.031418040040256626]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "CBS Bets on The Good Fight in the Streaming Wars", "color": "#FF0000", "data": [[1486080000000, 0.03606674158261045]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump Picks Wall Street Over Main Street", "color": "#FF0000", "data": [[1486166400000, 0.023102555090620785]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "La La Lands Damien Chazelle Wins Directors Guild Award", "color": "#FF0000", "data": [[1486252800000, 0.03494342932466825]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Buy Less, Pay More", "color": "#FF0000", "data": [[1486339200000, 0.06012818901788827]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Tom Price, a Bigger Problem Than Betsy DeVos", "color": "#FF0000", "data": [[1486425600000, 0.0396643728080506]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump, Socrates and the Laughter Effect", "color": "#FF0000", "data": [[1486512000000, 0.03757202154187034]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Trump should have Belichick spend one day a week at the White House. Heres why ", "color": "#FF0000", "data": [[1486598400000, 0.028079521551545566]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "In Welcoming Shinzo Abe, Trump Affirms U.S. Commitment to Defending Japan", "color": "#FF0000", "data": [[1486684800000, 0.03281162943269168]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Calls for Protests for and Against Planned Parenthood", "color": "#FF0000", "data": [[1486771200000, 0.02625167563685734]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "La La Land Wins Five Baftas, Continuing Its Oscar Momentum", "color": "#FF0000", "data": [[1486857600000, 0.0685440626390607]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Newt Gingrich: Why Trumps presidency will be great not just for America but Canada, too ", "color": "#FF0000", "data": [[1486944000000, 0.025571849913560984]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Lindsay Lohan defends Trump: \'We have to join him\' ", "color": "#FF0000", "data": [[1487030400000, 0.04311561239882547]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "No Love Lost for Michael Flynn on Valentines Day Shows", "color": "#FF0000", "data": [[1487116800000, 0.02762014760358772]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Review: \'The Good Fight\' \'can stand on its own\' ", "color": "#FF0000", "data": [[1487203200000, 0.04936556024770029]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Lincoln, Washington and Roosevelts remain historys best presidents in survey", "color": "#FF0000", "data": [[1487289600000, 0.029091922449511618]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Want to understand Trump? It\'s as simple as \'The Art of the Deal\' for dummies ", "color": "#FF0000", "data": [[1487376000000, 0.0310567626295596]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}, {"name": "Might Ivanka Trump Speak up if Her Father Guts the Arts?", "color": "#FF0000", "data": [[1487462400000, 0.027127567865876424]], "tooltip": {"pointFormat": "{point.y}"}, "marker": {"color": "#FF0000", "symbol": "circle", "enabled": true}, "type": "scatter", "showInLegend": false}]')



```python
# Example of plot_topic
sp.plot_topic(topic = 'Ivanka', date = datetime(2017,1,1))
```


![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_33_0.png)


# V. Feature Selection

Since the TfidfVectorizer yielded approximately 119,000 unigrams and bigrams, feature reduction was a necessity. Every unique word in a document can represent a new feature, so it's important to identify which words are the most informative and train a model on those, discarding the less informative words in order to reduce dimensionality and save time transforming data and fitting models. In addition, having a set vocabulary allows a classifier to work on new articles that may contain words it's never seen before. Since the news reports on new topics daily, this vocabulary would have to be continuously updated and optimized to account for content drift over time and to maintain a high level of accuracy.

## A. PCA

I tried a Principal Component Analysis and found that 80% of variance in the data could be explained with around 500 principal components, and over 90% could be explained with 800 principal components. However, when I fit a Naive Bayes classifier using 800 principal components, accuracy was only 56%. This low score, plus the loss of interpretability of the principal components made PCA less appealing than other dimensionality reduction methods.


![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_35_0.png)



## B. SelectKBest

The chi2 criterion is useful in feature selection for text classification tasks [3], so I compared the performance of Scikit-Learn's SelectKBest selector with k ranging from 100 to the full feature set with the chi2 scoring function, and assessed performance using a Bernoulli Naive Bayes classifier. Accuracy began to plateau with around 5000 features, so I built a vocabulary based on the top 5000 most informative features.



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_37_0.png)



This table lists the 15 most informative unigrams and bigrams, sorted by p-value. The chi2 score function assesses how unlikely the existing Tf-Idf scores would be if terms were distributed among the newspaper sections by chance. The most section-specific terms seem to be sports-related, though "researcher" and "company" may be specific to the science/health and business/tech sections respectively.



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_39_0.png)



# VI. Pipeline

This pipeline can be used to format new articles for classification. It includes all necessary transformations on an **ArticleData** object, as well as a scaling step. However, the entire pipeline does not preserve column names, so it is unfit for exploratory data analysis. If column names need to be preserved, **TitleTransformer** and **BodyTransformer** can be applied separately, and their results concatenated with the 'condensed_section' column for analysis.


## A. ItemSelector

**TitleTransformer** and **BodyTransformer** are written to only perform operations on the 'title' and 'body' columns of an **ArticleData** object. However, since the TfidfVectorizer need only be applied to the article bodies, I needed another transformer that could select a particular column and pass it on. ItemSelector can be combined with another more general Transformer in a sub-pipeline to apply a transformation to a subset of the total data.


```python
class ItemSelector(BaseEstimator, TransformerMixin):
    def __init__(self, key):
        self.key = key

    def fit(self, X, y = None):
        return self

    def transform(self,X):
        return(X[self.key])
```

## B. The Complete Pipeline

All features are derived and combined in a FeatureUnion during the 'features' step: applying title and body transformations, Tf-Idf vectorizing with a vocabulary of the top 5000 words derived from K-Best selection ('bigram_vocab'), and a transformation to dense.


```python
from sklearn.preprocessing import MinMaxScaler
from sklearn.pipeline import Pipeline, FeatureUnion
from sklearn.base import TransformerMixin, BaseEstimator
from mlxtend.preprocessing import DenseTransformer

pipe = Pipeline([('features', FeatureUnion([('t',TitleTransformer()),
                                            ('b', BodyTransformer()),
                                            ('tf-pipe', Pipeline([
                                            ('items', ItemSelector(key = 'body')),
                                           ('tfidf', TfidfVectorizer(preprocessor = tokenize, ngram_range = (1,2),
                                                                     min_df = 10, vocabulary = bigram_vocab)),
                                           ('dense', DenseTransformer())]))
                    ])),
                 ('scale', MinMaxScaler())
                 ])

# use fit_transform to prepare data for model fitting
# Warning: takes several hours to run for 4000 articles
Xt = pipe.fit_transform(data)
```

# VII. Models

Naive Bayes are the go-to models for text classification tasks, as they are reasonably efficient and robust to errors, especially when the sample size is relatively small [1]. However, other viable options include decision trees, K-Nearest Neighbors, and Support Vector Machines.

I tested six models in total using Scikit-Learn: KNN (with k ranging from 5-50, here reporting the highest score at k = 10), a random forest classifier, Adaboost, SGDClassifier, BernoulliNB, and MultinomialNB.

I witheld 30% of the data as a test set, and used a 5-fold cross-validated grid search to optimize hyperparameters for each model. The top two performers were Bernoulli Naive Bayes and Stochastic Gradient Descent Classifier at about 85% accuracy each. Interestingly, Multinomial Naive Bayes performed the worst, at only 32% accuracy. Multinomial Naive Bayes differs from Bernoulli Naive Bayes in that it assumes positional independence of tokens, which the data almost certainly violates, as words in written language have a meaningful and non-random sequence.



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_45_0.png)



# VIII. Model Evaluation

The confusion matrices for the two top-performing models contain diagonal bands of high values indicating the correctly categorized articles. The most difficult section to classify appears to be business/technology, as it has the lowest f1-score for both classifiers, though the SGDClassifier slightly out-performs the Naive Bayes. The opinion section also appears to pose some challenges for both classifiers, as both have low recall scores for this section, meaning that many opinion pieces are misclassified into other sections. This seems like a natural mistake to make, as an opinion piece could be written about any topic, and therefore, topical feature words might be less informative.

However, it's worth noting that the Naive Bayes model runs much, much faster than the SGDClassifier (a matter of minutes rather than hours).

## A. Bernoulli Naive Bayes



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_47_0.png)





![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_48_0.png)



## B. SGDClassifier



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_50_0.png)





![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_51_0.png)



For the SGDClassifier, I investigated the coefficient matrix to see which terms were the most informative for each class. This bubble chart shows the most features with the highest positive coefficient for each section. The sports section is defined strongly by terms like 'game', 'injury', or 'team', while the entertainment section is defined by 'novel', 'performance', or the number of people in the article title. Interestingly, opinion articles frequently the terms 'read post' or 'read topic', which may indicate that they are responses to other articles.

<meta charset="utf-8">
<style>

text {
  font: 10px sans-serif;
  text-anchor: middle;
}

</style>
<svg width="800" height="800"></svg>
<script src="https://d3js.org/d3.v4.min.js"></script>

<script>
var svg = d3.select("svg"),
    width = +svg.attr("width");

var format = d3.format(",d");

var color = d3.scaleOrdinal(d3.schemeCategory20);

var pack = d3.pack()
    .size([width, width])
    .padding(1.5);

d3.csv("../coef_df.csv", function(d) {
  d.value = +d.value;
  if (d.value) return d;
}, function(error, classes) {
  if (error) throw error;

  var root = d3.hierarchy({children: classes})
      .sum(function(d) { return d.value; })
      .each(function(d) {
        if (id = d.data.id) {
          var id, i = id.lastIndexOf(".");
          d.id = id;
          d.package = id.slice(0, i);
          d.class = id.slice(i + 1);
        }
      });

  var node = svg.selectAll(".node")
    .data(pack(root).leaves())
    .enter().append("g")
      .attr("class", "node")
      .attr("transform", function(d) { return "translate(" + d.x + "," + d.y + ")"; });

  node.append("circle")
      .attr("id", function(d) { return d.id; })
      .attr("r", function(d) { return d.r; })
      .style("fill", function(d) { return color(d.package); });

  node.append("clipPath")
      .attr("id", function(d) { return "clip-" + d.id; })
    .append("use")
      .attr("xlink:href", function(d) { return "#" + d.id; });

  node.append("text")
      .attr("clip-path", function(d) { return "url(#clip-" + d.id + ")"; })
    .selectAll("tspan")
    .data(function(d) { return d.class.split(' '); })
    .enter().append("tspan")
      .attr("x", 0)
      .attr("y", function(d, i, nodes) { return 13.0 + (i - nodes.length / 2.0 - 0.5) * 10.0; })
      .text(function(d) { return d; });

  node.append("title")
      .text(function(d) { return d.id + "\n" + d.value; });
});

</script>


# IX. Learning Curves

To investigate whether or not the classifiers might benefit from more data, I plotted the learning curves of the Bernoulli Naive Bayes and the SGDClassifiers. The training and cross-validation scores of the Naives Bayes classifier seem to be converging around 85%, so this model may not improve much with a larger sample size, but might benefit from further feature engineering.

```python
from sklearn.learning_curve import learning_curve
from sklearn.cross_validation import ShuffleSplit

cv = ShuffleSplit(n = 10, test_size = 0.3)
train_sizes, train_scores, valid_scores = learning_curve(sgd_model, X_select, y, cv = cv)

train_scores_mean = np.mean(train_scores, axis = 1)
test_scores_mean = np.mean(valid_scores, axis = 1)
```

![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_55_0.png)



However, the cross-validation score of the SGDClassifier does not seem to have plateaued at 4,000 samples, and may improve with more data.



![png](../../../images/capstone_final_writeup_files/capstone_final_writeup_57_0.png)



# X. Conclusions and Future Directions

This analysis has yielded two models that can classify a news article into the appropriate newspaper section with 85% accuracy. The primary use case for these classifiers is for online publishers or aggregators of news stories  to tag articles for easy searching or sorting, though they might also be useful to archivists, librarians, or any other business seeking to organize and categorize text.

There are several improvements that could be made to the classification process. Learning curves indicated that the Stochastic Gradient Descent classifier would benefit from a larger corpus, which is easily obtainable through web-scraping. In addition, including word counts from Scikit-Learn's CountVectorizer or counts of percents, dates, or numbers from a more complex Named-Entity Recognizer might help to boost accuracy for both SGDClassifier and Bernoulli Naive Bayes. 

However, including these features and more data would add processing time to an already slow transformation pipeline, so parallelizing this process on a framework like Spark or Hadoop would greatly facilitate the speed at which predictions could be made.

# XI. References

[1] Manning, C.D., Raghavan, P., Schutze, H. 2009. Introduction to Information Retrieval Online Edition. Cambridge (UK). Cambridge University Press.

[2] Bengfort, Ben. Text Classification with NLTK and Scikit-Learn. Blog Post. *Libelli*, 19 May 2016. Web. Accessed 2017.

[3] Fuka, K, Hanka, R. (2001). *Feature Set Reduction for Document Classification Problems*. Conference Proceedings. IJCAI-01 Workshop: Text Learning: Beyond Supervision. Seattle. University of Cambridge.

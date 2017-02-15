

```python
from articledata import *
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from nltk.corpus import names
from nltk.tag import StanfordNERTagger
import os
from nltk.tokenize import word_tokenize
```


```python
data = pd.read_pickle('/Users/teresaborcuch/capstone_project/notebooks/ss_entity_data.pkl')
```


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
      <th>total_persons_title</th>
      <th>total_places_title</th>
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
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
labeled_names = ([(name, 'male') for name in names.words("male.txt")] + [(name, 'female') for name in names.words('female.txt')])
```


```python
male_names = names.words("male.txt")
female_names = names.words("female.txt")
```


```python
male_names[:5]
```




    [u'Aamir', u'Aaron', u'Abbey', u'Abbie', u'Abbot']




```python
people, _ = evaluate_entities(data = data, section = 'politics', source = 'Fox')
```


```python
count = 0
for person in people:
    if person in male_names:
        count +=1

print count
```

    79



```python
count = 0
for person in people:
    if person in female_names:
        count +=1

print count
```

    36



```python
male_counts = []
female_counts = []
os.environ['CLASSPATH'] = "/Users/teresaborcuch/stanford-ner-2013-11-12/stanford-ner.jar"
os.environ['STANFORD_MODELS'] = '/Users/teresaborcuch/stanford-ner-2013-11-12/classifiers'
st = StanfordNERTagger('english.all.3class.distsim.crf.ser.gz')
for x in data['body']:
    m_count = 0
    f_count = 0
    tokens = word_tokenize(x)
    tags = st.tag(tokens)
    for pair in tags:
        if pair[1] == 'PERSON':
            if pair[0] in male_names:
                m_count += 1
            elif pair[0] in female_names:
                f_count += 1
            else:
                continue
    male_counts.append(m_count)
    female_counts.append(f_count)
```


```python
data['f_count'] = female_counts
data['m_count'] = male_counts
```


```python
data.to_pickle('/Users/teresaborcuch/capstone_project/notebooks/ss_entity_data.pkl')
```


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
      <th>total_persons_title</th>
      <th>total_places_title</th>
      <th>f_count</th>
      <th>m_count</th>
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
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>5</td>
    </tr>
  </tbody>
</table>
</div>




```python
data.pivot_table(index = ['condensed_section'], values = ['f_count', 'm_count']).sort_values('f_count', ascending = False)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>f_count</th>
      <th>m_count</th>
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
      <td>3.590985</td>
      <td>10.497496</td>
    </tr>
    <tr>
      <th>politics</th>
      <td>1.786932</td>
      <td>10.514205</td>
    </tr>
    <tr>
      <th>sports</th>
      <td>1.758621</td>
      <td>15.737069</td>
    </tr>
    <tr>
      <th>education</th>
      <td>1.692308</td>
      <td>8.846154</td>
    </tr>
    <tr>
      <th>other</th>
      <td>1.407960</td>
      <td>9.134328</td>
    </tr>
    <tr>
      <th>world</th>
      <td>1.175589</td>
      <td>5.083512</td>
    </tr>
    <tr>
      <th>business</th>
      <td>1.097087</td>
      <td>6.087379</td>
    </tr>
    <tr>
      <th>sci_health</th>
      <td>1.082251</td>
      <td>3.376623</td>
    </tr>
    <tr>
      <th>opinion</th>
      <td>0.973684</td>
      <td>6.172249</td>
    </tr>
    <tr>
      <th>technology</th>
      <td>0.333333</td>
      <td>3.404040</td>
    </tr>
  </tbody>
</table>
</div>



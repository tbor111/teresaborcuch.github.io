
# Project 2
## Step 1: Exploring your data.

##### Load your data in using Pandas and start to explore. Save all of your early exploration code here and include in your final submission.


```python
import pandas as pd
import numpy as np
import scipy
import matplotlib.pyplot as plt
```


```python
# Read in billboard.csv as a data frame
billboard = pd.read_csv("/Users/teresaborcuch/DSI-course-materials/curriculum/04-lessons/week-02/2.4-lab/assets/datasets/billboard.csv")
billboard.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>artist.inverted</th>
      <th>track</th>
      <th>time</th>
      <th>genre</th>
      <th>date.entered</th>
      <th>date.peaked</th>
      <th>x1st.week</th>
      <th>x2nd.week</th>
      <th>x3rd.week</th>
      <th>...</th>
      <th>x67th.week</th>
      <th>x68th.week</th>
      <th>x69th.week</th>
      <th>x70th.week</th>
      <th>x71st.week</th>
      <th>x72nd.week</th>
      <th>x73rd.week</th>
      <th>x74th.week</th>
      <th>x75th.week</th>
      <th>x76th.week</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2000</td>
      <td>Destiny's Child</td>
      <td>Independent Women Part I</td>
      <td>3:38</td>
      <td>Rock</td>
      <td>2000-09-23</td>
      <td>2000-11-18</td>
      <td>78</td>
      <td>63.0</td>
      <td>49.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2000</td>
      <td>Santana</td>
      <td>Maria, Maria</td>
      <td>4:18</td>
      <td>Rock</td>
      <td>2000-02-12</td>
      <td>2000-04-08</td>
      <td>15</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>Savage Garden</td>
      <td>I Knew I Loved You</td>
      <td>4:07</td>
      <td>Rock</td>
      <td>1999-10-23</td>
      <td>2000-01-29</td>
      <td>71</td>
      <td>48.0</td>
      <td>43.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2000</td>
      <td>Madonna</td>
      <td>Music</td>
      <td>3:45</td>
      <td>Rock</td>
      <td>2000-08-12</td>
      <td>2000-09-16</td>
      <td>41</td>
      <td>23.0</td>
      <td>18.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2000</td>
      <td>Aguilera, Christina</td>
      <td>Come On Over Baby (All I Want Is You)</td>
      <td>3:38</td>
      <td>Rock</td>
      <td>2000-08-05</td>
      <td>2000-10-14</td>
      <td>57</td>
      <td>47.0</td>
      <td>45.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 83 columns</p>
</div>




```python
print(billboard['date.entered'].max())
print(billboard['date.entered'].min())

```

    2000-12-30
    1999-06-05


##### Write a brief description of your data, and any interesting observations you've made thus far. 

The data contains 317 songs that ranked in Billboard's Top 100 list in the year 2000. Each row includes song title, artist, duration, genre, the date it first entered the list, the date it reached its highest ranking, and then its ranking for 76 weeks ranging from June 1999 - December 2000. (If a song's entry date was in 1999, but it remained on the list into 2000, it is still included.)

## Step 2: Clean your data.

##### Do some rudimentary cleaning. Rename any columns that are poorly named, shorten any strings that may be too long, check for missing values (and replace them if it makes sense to do so). Explain your rationale for the way you choose to "impute" the missing data.


```python
#rename artist.inverted
billboard = billboard.rename(columns = {'artist.inverted': 'artist'})
```


```python
# Convert time to seconds
billboard['time'] = billboard['time'].str.split(':')
time = billboard['time'].tolist()
seconds_list = []
for pair in time:
    minutes = int(pair[0])
    seconds = int(pair[1])
    total_sec = (minutes * 60)+ seconds
    seconds_list.append(total_sec)
billboard['time_in_sec'] = seconds_list
```


```python
# Count number of weeks each track spent in top 100
# Report summary stats
from scipy import stats
num_rows = range(317)
weeks_in_top_hun = []
for num in num_rows:
    row = billboard.iloc[num][7:]
    not_null = pd.notnull(billboard.iloc[num][7:])
    week_list = billboard.iloc[num][7:][not_null]
    num_weeks = len(week_list)
    weeks_in_top_hun.append(num_weeks)

billboard['weeks_in_top_hun'] = weeks_in_top_hun
```


```python
# isolate month from date.peaked column and save astype('category') in its own column
billboard['date.peaked'] = billboard['date.peaked'].astype('string')
peak_month_split = billboard['date.peaked'].str.split('-')
peak_months = []
for item in peak_month_split:
    month = item[1]
    peak_months.append(month)

billboard['month.peaked'] = peak_months
```


```python
billboard.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>artist</th>
      <th>track</th>
      <th>time</th>
      <th>genre</th>
      <th>date.entered</th>
      <th>date.peaked</th>
      <th>x1st.week</th>
      <th>x2nd.week</th>
      <th>x3rd.week</th>
      <th>...</th>
      <th>x70th.week</th>
      <th>x71st.week</th>
      <th>x72nd.week</th>
      <th>x73rd.week</th>
      <th>x74th.week</th>
      <th>x75th.week</th>
      <th>x76th.week</th>
      <th>time_in_sec</th>
      <th>weeks_in_top_hun</th>
      <th>month.peaked</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2000</td>
      <td>Destiny's Child</td>
      <td>Independent Women Part I</td>
      <td>[3, 38]</td>
      <td>Rock</td>
      <td>2000-09-23</td>
      <td>2000-11-18</td>
      <td>78</td>
      <td>63.0</td>
      <td>49.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>218</td>
      <td>29</td>
      <td>11</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2000</td>
      <td>Santana</td>
      <td>Maria, Maria</td>
      <td>[4, 18]</td>
      <td>Rock</td>
      <td>2000-02-12</td>
      <td>2000-04-08</td>
      <td>15</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>258</td>
      <td>27</td>
      <td>04</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>Savage Garden</td>
      <td>I Knew I Loved You</td>
      <td>[4, 07]</td>
      <td>Rock</td>
      <td>1999-10-23</td>
      <td>2000-01-29</td>
      <td>71</td>
      <td>48.0</td>
      <td>43.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>247</td>
      <td>34</td>
      <td>01</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2000</td>
      <td>Madonna</td>
      <td>Music</td>
      <td>[3, 45]</td>
      <td>Rock</td>
      <td>2000-08-12</td>
      <td>2000-09-16</td>
      <td>41</td>
      <td>23.0</td>
      <td>18.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>225</td>
      <td>25</td>
      <td>09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2000</td>
      <td>Aguilera, Christina</td>
      <td>Come On Over Baby (All I Want Is You)</td>
      <td>[3, 38]</td>
      <td>Rock</td>
      <td>2000-08-05</td>
      <td>2000-10-14</td>
      <td>57</td>
      <td>47.0</td>
      <td>45.0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>218</td>
      <td>22</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 86 columns</p>
</div>




```python
cols_reordered = ['year','artist','track','time','time_in_sec','genre','date.entered','date.peaked','month.peaked', 
                  'weeks_in_top_hun','x1st.week','x2nd.week','x3rd.week','x4th.week','x5th.week','x6th.week',
                  'x7th.week','x8th.week','x9th.week','x10th.week','x11th.week','x12th.week','x13th.week','x14th.week','x15th.week','x16th.week',
                  'x17th.week','x18th.week','x19th.week','x20th.week','x21st.week','x22nd.week','x23rd.week',
                  'x24th.week','x25th.week','x26th.week','x27th.week','x28th.week','x29th.week','x30th.week',
                  'x31st.week','x32nd.week','x33rd.week','x34th.week','x35th.week','x36th.week','x37th.week',
                  'x38th.week','x39th.week','x40th.week','x41st.week','x42nd.week','x43rd.week','x44th.week',
                  'x45th.week','x46th.week','x47th.week','x48th.week','x49th.week','x50th.week','x51st.week',
                  'x52nd.week','x53rd.week','x54th.week','x55th.week','x56th.week','x57th.week','x58th.week',
                  'x59th.week','x60th.week','x61st.week','x62nd.week','x63rd.week','x64th.week','x65th.week',
                  'x66th.week','x67th.week','x68th.week','x69th.week','x70th.week','x71st.week','x72nd.week',
                  'x73rd.week','x74th.week','x75th.week','x76th.week']
```


```python
billboard = billboard[cols_reordered]
```


```python
billboard.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>year</th>
      <th>artist</th>
      <th>track</th>
      <th>time</th>
      <th>time_in_sec</th>
      <th>genre</th>
      <th>date.entered</th>
      <th>date.peaked</th>
      <th>month.peaked</th>
      <th>weeks_in_top_hun</th>
      <th>...</th>
      <th>x67th.week</th>
      <th>x68th.week</th>
      <th>x69th.week</th>
      <th>x70th.week</th>
      <th>x71st.week</th>
      <th>x72nd.week</th>
      <th>x73rd.week</th>
      <th>x74th.week</th>
      <th>x75th.week</th>
      <th>x76th.week</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2000</td>
      <td>Destiny's Child</td>
      <td>Independent Women Part I</td>
      <td>[3, 38]</td>
      <td>218</td>
      <td>Rock</td>
      <td>2000-09-23</td>
      <td>2000-11-18</td>
      <td>11</td>
      <td>29</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2000</td>
      <td>Santana</td>
      <td>Maria, Maria</td>
      <td>[4, 18]</td>
      <td>258</td>
      <td>Rock</td>
      <td>2000-02-12</td>
      <td>2000-04-08</td>
      <td>04</td>
      <td>27</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2000</td>
      <td>Savage Garden</td>
      <td>I Knew I Loved You</td>
      <td>[4, 07]</td>
      <td>247</td>
      <td>Rock</td>
      <td>1999-10-23</td>
      <td>2000-01-29</td>
      <td>01</td>
      <td>34</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2000</td>
      <td>Madonna</td>
      <td>Music</td>
      <td>[3, 45]</td>
      <td>225</td>
      <td>Rock</td>
      <td>2000-08-12</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>25</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2000</td>
      <td>Aguilera, Christina</td>
      <td>Come On Over Baby (All I Want Is You)</td>
      <td>[3, 38]</td>
      <td>218</td>
      <td>Rock</td>
      <td>2000-08-05</td>
      <td>2000-10-14</td>
      <td>10</td>
      <td>22</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 86 columns</p>
</div>



##### Using Pandas' built in `melt` function, pivot the weekly ranking data to be long rather than wide. As a result, you will have removed the 72 'week' columns and replace it with two: Week and Ranking. There will now be multiple entries for each song, one for each week on the Billboard rankings.


```python
billboard1 = pd.melt(billboard, id_vars = ['track', 'artist', 'year', 'time', 'time_in_sec','genre', 'date.entered','date.peaked',
                              'month.peaked', 'weeks_in_top_hun'], var_name = 'week', value_name = 'ranking')
```


```python
billboard1.sort_values(['track','week'])
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>track</th>
      <th>artist</th>
      <th>year</th>
      <th>time</th>
      <th>time_in_sec</th>
      <th>genre</th>
      <th>date.entered</th>
      <th>date.peaked</th>
      <th>month.peaked</th>
      <th>weeks_in_top_hun</th>
      <th>week</th>
      <th>ranking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2900</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x10th.week</td>
      <td>36.0</td>
    </tr>
    <tr>
      <th>3217</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x11th.week</td>
      <td>37.0</td>
    </tr>
    <tr>
      <th>3534</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x12th.week</td>
      <td>30.0</td>
    </tr>
    <tr>
      <th>3851</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x13th.week</td>
      <td>23.0</td>
    </tr>
    <tr>
      <th>4168</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x14th.week</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>4485</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x15th.week</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>4802</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x16th.week</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>5119</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x17th.week</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>5436</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x18th.week</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>5753</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x19th.week</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>47</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x1st.week</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>6070</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x20th.week</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>6387</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x21st.week</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>6704</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x22nd.week</td>
      <td>7.0</td>
    </tr>
    <tr>
      <th>7021</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x23rd.week</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>7338</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x24th.week</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>7655</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x25th.week</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>7972</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x26th.week</td>
      <td>12.0</td>
    </tr>
    <tr>
      <th>8289</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x27th.week</td>
      <td>12.0</td>
    </tr>
    <tr>
      <th>8606</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x28th.week</td>
      <td>17.0</td>
    </tr>
    <tr>
      <th>8923</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x29th.week</td>
      <td>19.0</td>
    </tr>
    <tr>
      <th>364</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x2nd.week</td>
      <td>99.0</td>
    </tr>
    <tr>
      <th>9240</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x30th.week</td>
      <td>23.0</td>
    </tr>
    <tr>
      <th>9557</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x31st.week</td>
      <td>33.0</td>
    </tr>
    <tr>
      <th>9874</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x32nd.week</td>
      <td>37.0</td>
    </tr>
    <tr>
      <th>10191</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x33rd.week</td>
      <td>36.0</td>
    </tr>
    <tr>
      <th>10508</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x34th.week</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>10825</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x35th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11142</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x36th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11459</th>
      <td>(Hot S**t) Country Grammar</td>
      <td>Nelly</td>
      <td>2000</td>
      <td>[4, 17]</td>
      <td>257</td>
      <td>Rap</td>
      <td>2000-04-29</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>35</td>
      <td>x37th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>16333</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x52nd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16650</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x53rd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16967</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x54th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17284</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x55th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17601</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x56th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17918</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x57th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18235</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x58th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18552</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x59th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1434</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x5th.week</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>18869</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x60th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19186</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x61st.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19503</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x62nd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19820</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x63rd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20137</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x64th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20454</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x65th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20771</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x66th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21088</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x67th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21405</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x68th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21722</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x69th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1751</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x6th.week</td>
      <td>49.0</td>
    </tr>
    <tr>
      <th>22039</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x70th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22356</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x71st.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22673</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x72nd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22990</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x73rd.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23307</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x74th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23624</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x75th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23941</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x76th.week</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2068</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x7th.week</td>
      <td>47.0</td>
    </tr>
    <tr>
      <th>2385</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x8th.week</td>
      <td>45.0</td>
    </tr>
    <tr>
      <th>2702</th>
      <td>www.memory</td>
      <td>Jackson, Alan</td>
      <td>2000</td>
      <td>[2, 36]</td>
      <td>156</td>
      <td>Country</td>
      <td>2000-11-04</td>
      <td>2000-12-23</td>
      <td>12</td>
      <td>16</td>
      <td>x9th.week</td>
      <td>49.0</td>
    </tr>
  </tbody>
</table>
<p>24092 rows × 12 columns</p>
</div>




```python
#change week to number value
weeks = billboard1['week'].tolist()
new_weeks = []
for week in weeks:
    new_week = week.replace('th.week', '').replace('st.week', '').replace('nd.week', '').replace('rd.week','').replace('x','')
    int_week = int(new_week)
    new_weeks.append(int_week)
    
billboard1['week'] = new_weeks
```


```python
# drop NaN's!
billboard1 = billboard1.dropna()
billboard1.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>track</th>
      <th>artist</th>
      <th>year</th>
      <th>time</th>
      <th>time_in_sec</th>
      <th>genre</th>
      <th>date.entered</th>
      <th>date.peaked</th>
      <th>month.peaked</th>
      <th>weeks_in_top_hun</th>
      <th>week</th>
      <th>ranking</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Independent Women Part I</td>
      <td>Destiny's Child</td>
      <td>2000</td>
      <td>[3, 38]</td>
      <td>218</td>
      <td>Rock</td>
      <td>2000-09-23</td>
      <td>2000-11-18</td>
      <td>11</td>
      <td>29</td>
      <td>1</td>
      <td>78.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Maria, Maria</td>
      <td>Santana</td>
      <td>2000</td>
      <td>[4, 18]</td>
      <td>258</td>
      <td>Rock</td>
      <td>2000-02-12</td>
      <td>2000-04-08</td>
      <td>04</td>
      <td>27</td>
      <td>1</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>I Knew I Loved You</td>
      <td>Savage Garden</td>
      <td>2000</td>
      <td>[4, 07]</td>
      <td>247</td>
      <td>Rock</td>
      <td>1999-10-23</td>
      <td>2000-01-29</td>
      <td>01</td>
      <td>34</td>
      <td>1</td>
      <td>71.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Music</td>
      <td>Madonna</td>
      <td>2000</td>
      <td>[3, 45]</td>
      <td>225</td>
      <td>Rock</td>
      <td>2000-08-12</td>
      <td>2000-09-16</td>
      <td>09</td>
      <td>25</td>
      <td>1</td>
      <td>41.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Come On Over Baby (All I Want Is You)</td>
      <td>Aguilera, Christina</td>
      <td>2000</td>
      <td>[3, 38]</td>
      <td>218</td>
      <td>Rock</td>
      <td>2000-08-05</td>
      <td>2000-10-14</td>
      <td>10</td>
      <td>22</td>
      <td>1</td>
      <td>57.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
week_rank = billboard1.pivot_table(index = ['week'], values = 'ranking', columns = 'genre')
```


```python
genre_weeks_in_hun = billboard1.pivot_table(index = 'genre', values = 'weeks_in_top_hun', aggfunc = np.mean)
```


```python
genre_avg_rank = billboard1.pivot_table(index = 'genre', values = 'ranking', aggfunc = np.mean)
```


```python
peak_months = billboard1['month.peaked'].value_counts()
peak_months
```




    01    643
    12    596
    07    552
    04    521
    09    502
    08    449
    05    358
    10    357
    02    344
    03    343
    06    336
    11    306
    Name: month.peaked, dtype: int64




```python
peak_month_df = pd.DataFrame(columns = ['month', 'num_songs'])
months = ['Jan', 'Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']
num_songs = [643,344,343,521,358,336,552,449,502,357,306,596]
peak_month_df['month'] = months
peak_month_df['num_songs'] = num_songs
```

## Step 3: Visualize your data.

##### Using a plotting utility of your choice (Tableau or python modules or both), create visualizations that will provide context to your data. There is no minimum or maximum number of graphs you should generate, but there should be a clear and consistent story being told. Give insights to the distribution, statistics, and relationships of the data. 


```python
# Graph ranking over time
wk_rk_plot = week_rank.plot(figsize = (30,10), xticks = (range(0,70,5)), yticks = (range(0,100,10)), 
                            colormap = 'nipy_spectral', lw = 4)
wk_rk_plot.legend(fontsize = 'xx-large')
plt.show()
```


![png](project_2_files/project_2_28_0.png)



```python
#graph of weeks in top 100 by genre
genre_week_plot = genre_weeks_in_hun.plot(kind = "bar", colormap = 'nipy_spectral')
genre_week_plot.set_title("Average Time in Top 100")
genre_week_plot.set_ylabel("Weeks")
genre_week_plot.set_xlabel("Genre")
plt.show()
```


![png](project_2_files/project_2_29_0.png)



```python
gen_rank_plot = genre_avg_rank.plot(kind = "bar", colormap = 'nipy_spectral')
gen_rank_plot.set_ylabel("Average Song Rank")
gen_rank_plot.set_xlabel("Genre")
gen_rank_plot.set_title("Average Rank of Songs by Genre")
plt.show()
```


![png](project_2_files/project_2_30_0.png)



```python
plt.bar(range(len(peak_month_df['month'])), peak_month_df['num_songs'], align = 'center')
plt.xticks(range(len(peak_month_df['month'])), peak_month_df['month'])
plt.xlabel('Month')
plt.ylabel('Song Count')
plt.title('Number of Songs Peaking per Month')
plt.show()
```


![png](project_2_files/project_2_31_0.png)



```python
# At a glance descriptive stats:
# Mean # of weeks for a song to stay on the Billboard Top 100
print "Maximum: ", max(weeks_in_top_hun)
print "Minumum: ", min(weeks_in_top_hun)
print "Mean: ", np.mean(weeks_in_top_hun)
print "Median: ", np.median(weeks_in_top_hun)
print "Mode: ", scipy.stats.mode(np.array(weeks_in_top_hun))[0][0]
print "Standard Deviation: ", np.std(weeks_in_top_hun)
print "Variance: ", np.var(weeks_in_top_hun)
```

    Maximum:  58
    Minumum:  2
    Mean:  17.7413249211
    Median:  19.0
    Mode:  21
    Standard Deviation:  9.06944616639
    Variance:  82.2548537651


## Step 4: Create a Problem Statement.

##### Having explored the data, come up with a problem statement for this data set. You can feel free to introduce data from any other source to support your problem statement, just be sure to provide a link to the origin of the data. Once again- be creative!

The Billboard Top 100 is a list of the most popular songs based on sales and airplay time. Given a dataset of the Billboard Top 100 list for the year 2000, I propose that popularity of a given track or genre can be assessed by measuring how high it ranks during its time on the list, how fast it reaches its peak, and how many weeks it stays in the top 100. Certain tracks may peak quickly, but have little staying power on the list, while others may not rank significantly higher than average, but have a long tenure in the top 100. Identifying which genres fall into which of these categories will provide insight into the tastes of music listeners in the year 2000.

## Step 5: Brainstorm your Approach.
##### In bullet-list form, provide a proposed approach for evaluating your problem statement. This can be somewhat high-level, but start to think about ways you can massage the data for maximum efficacy. 

- Calculate the mean ranking of tracks overall and identify if any genres rank higher on average than others
- Calculate mean # of weeks a track spends in the top 100, and identify if any genres spend significantly longer or shorter time on the list
- Plot weeks on the list vs average ranking by genres to determine if there's a relationship between popularity and staying power

## Step 6: Create a blog post with your code snippets and visualizations.
##### Data Science is a growing field, and the Tech industry thrives off of collaboration and sharing of knowledge. Blogging is a powerful means for pushing the needle forward in our field. Using your blogging platform of choice, create a post describing each of the 5 steps above. Rather than writing a procedural text, imagine you're describing the data, visualizations, and conclusions you've arrived at to your peers. Aim for a minimum of 500 words. 


```python

```

## BONUS: The Content Managers working for the Podcast Publishing Company have recognized you as a thought leader in your field. They've asked you to pen a white paper (minimum 500 words) on the subject of 'What It Means To Have Clean Data'. This will be an opinion piece read by a wide audience, so be sure to back up your statements with real world examples or scenarios.

##### Hint: To get started, look around on the internet for articles, blog posts, papers, youtube videos, podcasts, reddit discussions, anything that will help you understand the challenges and implications of dealing with big data. This should be a personal reflection on everything you've learned this week, and the learning goals that have been set out for you going forward. 


```python

```

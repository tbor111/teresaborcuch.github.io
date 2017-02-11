---
layout: post
title: Principal Component Analysis of FAA Data
category: blog
tags: [pandas, PCA, 3D plots]
---
# Using Data to Reduce Flight Delays

Given operational data for 74 airports over a 10 year period, I conducted a principal component analysis to examine underlying factors impacting flight delays. The data contains durations of gate, taxiing, and total delays for both arrivals and departures, volume of arrivals, departures, cancellations, and diversions, as well as the percent of flights that were delayed arriving or departing. PCA revealed three components that roughly correspond to overall delays and flight volume, gate and arrival delays, and airborne and taxi delays. It appears that gate delays are the most related to overall departure delay, and that busiest airports are not necessarily the most congested.

# Data Cleaning and Merging

The flight data originally came in three separate csv files. The operations file contained all data on delays for each airport in a particular year. The cancellations file contained the number of cancellations and diversions for an airport in that year, and the airport file contained data on an airport's location. I merged these into a single dataframe using Pandas' merge function and dropped all NaN rows.




```python
cancellations = pd.read_csv("../assets/airport_cancellations.csv")
airports = pd.read_csv("../assets/airports.csv")
operations = pd.read_csv("../assets/Airport_operations.csv")
```


```python
# merge the datasets
data = operations.merge(airports, left_on = "airport", right_on = "LocID", how = "left")
```



```python
data = data.merge(cancellations, left_on = ["airport","year"], right_on = ['Airport','Year'], how = "inner")
```


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>airport</th>
      <th>year</th>
      <th>departures for metric computation</th>
      <th>arrivals for metric computation</th>
      <th>percent on-time gate departures</th>
      <th>percent on-time airport departures</th>
      <th>percent on-time gate arrivals</th>
      <th>average_gate_departure_delay</th>
      <th>average_taxi_out_time</th>
      <th>average taxi out delay</th>
      <th>...</th>
      <th>AP Type</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Boundary Data Available</th>
      <th>Airport</th>
      <th>Year</th>
      <th>Departure Cancellations</th>
      <th>Arrival Cancellations</th>
      <th>Departure Diversions</th>
      <th>Arrival Diversions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ABQ</td>
      <td>2004</td>
      <td>53971</td>
      <td>53818</td>
      <td>0.8030</td>
      <td>0.7809</td>
      <td>0.7921</td>
      <td>10.38</td>
      <td>9.89</td>
      <td>2.43</td>
      <td>...</td>
      <td>Federalized/Commercial</td>
      <td>35.040194</td>
      <td>-106.609194</td>
      <td>Yes</td>
      <td>ABQ</td>
      <td>2004.0</td>
      <td>242.0</td>
      <td>235.0</td>
      <td>71.0</td>
      <td>46.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ABQ</td>
      <td>2005</td>
      <td>51829</td>
      <td>51877</td>
      <td>0.8140</td>
      <td>0.7922</td>
      <td>0.8001</td>
      <td>9.60</td>
      <td>9.79</td>
      <td>2.29</td>
      <td>...</td>
      <td>Federalized/Commercial</td>
      <td>35.040194</td>
      <td>-106.609194</td>
      <td>Yes</td>
      <td>ABQ</td>
      <td>2005.0</td>
      <td>221.0</td>
      <td>190.0</td>
      <td>61.0</td>
      <td>33.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ABQ</td>
      <td>2006</td>
      <td>49682</td>
      <td>51199</td>
      <td>0.7983</td>
      <td>0.7756</td>
      <td>0.7746</td>
      <td>10.84</td>
      <td>9.89</td>
      <td>2.16</td>
      <td>...</td>
      <td>Federalized/Commercial</td>
      <td>35.040194</td>
      <td>-106.609194</td>
      <td>Yes</td>
      <td>ABQ</td>
      <td>2006.0</td>
      <td>392.0</td>
      <td>329.0</td>
      <td>71.0</td>
      <td>124.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ABQ</td>
      <td>2007</td>
      <td>53255</td>
      <td>53611</td>
      <td>0.8005</td>
      <td>0.7704</td>
      <td>0.7647</td>
      <td>11.29</td>
      <td>10.34</td>
      <td>2.40</td>
      <td>...</td>
      <td>Federalized/Commercial</td>
      <td>35.040194</td>
      <td>-106.609194</td>
      <td>Yes</td>
      <td>ABQ</td>
      <td>2007.0</td>
      <td>366.0</td>
      <td>304.0</td>
      <td>107.0</td>
      <td>45.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ABQ</td>
      <td>2008</td>
      <td>49589</td>
      <td>49512</td>
      <td>0.8103</td>
      <td>0.7844</td>
      <td>0.7875</td>
      <td>10.79</td>
      <td>10.41</td>
      <td>2.41</td>
      <td>...</td>
      <td>Federalized/Commercial</td>
      <td>35.040194</td>
      <td>-106.609194</td>
      <td>Yes</td>
      <td>ABQ</td>
      <td>2008.0</td>
      <td>333.0</td>
      <td>300.0</td>
      <td>79.0</td>
      <td>42.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 34 columns</p>
</div>



# Exploratory Data Analysis

These data seem deceptively straightforward. Longer delays mean fewer on-time flights, right? There's a bit more to it. First, I want to differentiate between how frequently an airport has a late flight (the percent on-time columns for arrivals and departures), and how late it is (the average delay columns). Having every flight arrive one minute late may not have the same effect as having 5% of the flights arrive one hour late.

It's also important to understand how each variable is defined. [This website](http://aspmhelp.faa.gov/index.php/ASPM_Airport_Analysis:_Definitions_of_Variables) provides the definitions of the Aviation System Performance Metrics. One distinction that needs to be made is that all of the average delay times only include delays > 1 minute, so a flight that is on time is not included. Therefore, the average delays are not truly averaged across all flights. It's more accurate to think of them as the average delay, when there is a delay.


```python
# plot histograms for departure delays
fig = plt.figure(figsize = (14,6))
count = 1
for col in ['average_gate_departure_delay','average taxi out delay','average airport departure delay']:
    ax = fig.add_subplot(1,3,count)
    ax.hist(data[col], color = "navy")
    ax.set_xlabel(col)
    count +=1
fig.suptitle('Distributions of Departure Delays', fontsize = 20)
plt.show()
fig.tight_layout()

```


![png](../../../images/project_7_faa_files/project_7_faa_8_0.png)


From these plots, I can see that the average flight is delayed  around 12 minutes at the gate, and only about 3 minutes in taxi-ing out. It seems that the majority of the total average departure delay comes from sitting at the gate rather than slow taxi-ing out.


![png](../../../images/project_7_faa_files/project_7_faa_10_0.png)


From these histograms, I can see the average airborne and taxi-in delays are between 2-3 and 1-2 minutes respectively, but the average gate arrival delay is significantly longer, between 10-15 minutes. Average gate arrival is defined as the difference between scheduled and actual arrival times, and I hypothesize that this difference between average gate arrival delay and the other arrival delays actually spills over from departure delays.



![png](../../../images/project_7_faa_files/project_7_faa_12_0.png)


Here, I can see that in fact, most flights do depart and arrive on time. The majority of airports had between 70 and 80% of their flights leave and arrive on time. Average block delay is difference in minutes between scheduled and actual gate-to-gate time. The fact that this is lower than the average departure and arrival times may be explained by the fact that it includes on-time flights, and possibly flights that arrive early.


![png](../../../images/project_7_faa_files/project_7_faa_14_0.png)


The arrival and departure data are heavily left skewed. The majority of airports had fewer than 100,000 flights pass through in a given year, but there are a few that had over 400,000 flights.

### Example Airports


```python
# plot on-time gate departures for select airports
mask = (data['airport'] == 'BOS')|(data['airport'] == 'ATL') |(data['airport'] == 'BUF')|(data['airport'] == 'HPN')|(data['airport'] == 'JFK')| (data['airport'] == 'ORD')| (data['airport'] == 'OGG')|(data['airport'] == 'MDW')|(data['airport'] == 'LAX')|(data['airport'] == 'SDF')
dept_data = data[mask].groupby(['airport'])['percent on-time gate departures'].mean()

plt.figure(figsize = (12,6))

dept_bar_positions = [i + 0.65 for i in range(len(dept_data))]
dept_bar_heights = dept_data.values * 100
# plot on-time departures
plt.bar(dept_bar_positions, dept_bar_heights, 0.45, color = "navy")
plt.xticks([i + 0.5 for i in dept_bar_positions], dept_data.index.values, rotation = 40)

# plot departure delays
delay_data = data[mask].groupby(['airport'])['average airport departure delay'].mean()
delay_bar_positions = [i + 1.1 for i in range(len(delay_data))]
delay_bar_heights = delay_data.values
plt.bar(delay_bar_positions, delay_bar_heights, 0.45, color = 'darkorange')

plt.ylabel('Percent On-Time Gate Departures/Average Departure Delay (min)')
plt.legend(labels = ['percent on-time departures', 'average departure delay time'], loc = 'best')
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_17_0.png)


This graph contains the percent of on time departures and the average delay time for a few select airports of varying sizes and locations to demonstrate that there isn't necessarily a clear relationship between the two. Among the airports with the longest delay times are JFK, John F. Kennedy International Airport, and HPN, the Westchester County Airport. Although they are both in the same Northeast region promixal to NYC and have long average departure delays, JFK is a busy international airport whereas HPN is serviced by only seven airlines with no international flights. In contrast, OGG (the Kahului Airport in Hawaii) has the shortest average departure delay, and a very high on-time departure rate.

### Select Data Summary at a Glance



    Mean Percent On-Time Departures:  72.6957571965 %
    Mean Departure Delay:  15.7035043805 minutes
    Max Departure Delay:  40.51 minutes (JFK in 2007)
    Min Departure Delay:  6.29 minutes (OGG in 2010)


### Correlations Between Features


```python
# select relevantfeatures from data
X = data[[u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'percent on-time gate departures',
          'percent on-time airport departures', u'average_gate_departure_delay',
          u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay',
          u'average airborne delay', u'arrivals for metric computation',
          u'Arrival Cancellations', u'Arrival Diversions', u'percent on-time gate arrivals', u'average taxi in delay',
       u'average block delay', u'average gate arrival delay', 'Latitude','Longitude']]
```


```python
plt.figure(figsize = (6,6))
sns.heatmap(X.corr())
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_23_0.png)


This heatmap is arranged with all departure-related features on its top and left halves and all arrival-related features on the bottom and right halves. The blue bands are percent on-time arrivals, departures, and gate departures, which are obviously negatively correlated with all delay metrics. In general, all delay features seem to be positively correlated with each other, there are a few specific observations to make.

Average gate departure delays are strongly correlated with both average airport departure delays, as well as gate arrival delays. This makes sense, because if a plane is late leaving its gate, it will be behind schedule taking off, and an arriving plane can't pull into the gate until the first plane has left. In fact, average gate and aiport departure delays are more strongly correlated with average gate arrival delay than taxi-in delays.

There are strong correlations between total arrivals, total departures, and number of cancellations and diversions. This is all a consequence of simply having more flights coming and going to busy airports. However, there is one delay-related factor that seems to be correlated with all of these: taxi-in delay. Taxi-in delay is weakly correlated with percent on-time departures, so this may be a problem only at airports of a certain size.

Interestingly, longitude is negatively correlated with percent on-time departures and arrivals, so the greater the longitude (or the further east), the more unlikely flights are to be on time.

These correlations point to delays originating with departing flights, as the effect of staying longer than anticipated at the gate makes the flight itself late to take off, but also prevents incoming flights from pulling into the gate at their scheduled times.

# Principal Component Analysis


```python
# normalize the data and remove percent columns
Xt = scale(X[[u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'average_gate_departure_delay', u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay', u'average airborne delay',  u'arrivals for metric computation',
          u'Arrival Cancellations', u'Arrival Diversions', u'average taxi in delay', u'average block delay',
              u'average gate arrival delay','Latitude','Longitude']])
```


```python
pca = PCA(n_components = 16)
pca = pca.fit(Xt)
```


```python
# make dataframe of principal components
Y = pd.DataFrame(pca.fit_transform(Xt), columns = ['PC{}'.format(str(i)) for i in range(1,17)])
Y['region'] = data['FAA REGION']
Y['busy'] = [
    2 if x >= 300000 else 1 if (x < 300000 and x > 100000) else 0 for x in data['departures for metric computation']
]
Y['airport'] = data['airport']
```


```python
# Use cumulative sum to get total explained variances
exp_variances = np.cumsum(pca.explained_variance_ratio_*100)
```


```python
# plot cumulative explained variance
plt.figure(figsize = (6,6))
plt.plot(range(1, 17), exp_variances, '-o')
plt.xlabel("Number of components")
plt.ylabel("Cumulative % Explained Variance")
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_30_0.png)



```python
# scree plot of eigenvalues
plt.figure(figsize = (8,6))
plt.plot(range(1,17), pca.explained_variance_, "-o")
plt.xticks(range(1,17))
plt.ylabel("Eigenvalues")
plt.xlabel("PCA Components")
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_31_0.png)


From examining the cumulative explained variance curve as well as the eigenvalues plot, I will select the first three principle components, as these will account for approximately 85% of the variance in the data and this seems to be where an "elbow" occurs in the eigenvalue plot.


```python
# Make a dataframe of eigenvectors and examine top 3
eigenvectors = pd.DataFrame(pca.components_, columns = [u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'average_gate_departure_delay', u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay', u'average airborne delay',
          u'arrivals for metric computation', u'Arrival Cancellations',
          u'Arrival Diversions', u'percent on-time gate arrivals',
          u'average taxi in delay', u'average block delay',
          u'average gate arrival delay', 'Latitude','Longitude'])
eigenvectors.head(3)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>departures for metric computation</th>
      <th>Departure Cancellations</th>
      <th>Departure Diversions</th>
      <th>average_gate_departure_delay</th>
      <th>average_taxi_out_time</th>
      <th>average taxi out delay</th>
      <th>average airport departure delay</th>
      <th>average airborne delay</th>
      <th>arrivals for metric computation</th>
      <th>Arrival Cancellations</th>
      <th>Arrival Diversions</th>
      <th>average taxi in delay</th>
      <th>average block delay</th>
      <th>average gate arrival delay</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.296668</td>
      <td>0.298337</td>
      <td>0.285840</td>
      <td>0.165382</td>
      <td>0.286638</td>
      <td>0.288277</td>
      <td>0.260309</td>
      <td>0.206061</td>
      <td>0.296403</td>
      <td>0.297720</td>
      <td>0.283809</td>
      <td>0.294027</td>
      <td>0.216516</td>
      <td>0.180060</td>
      <td>0.064057</td>
      <td>0.107449</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.254314</td>
      <td>0.143402</td>
      <td>0.268331</td>
      <td>-0.415843</td>
      <td>-0.087365</td>
      <td>-0.098885</td>
      <td>-0.347215</td>
      <td>-0.181191</td>
      <td>0.255471</td>
      <td>0.158649</td>
      <td>0.191832</td>
      <td>0.148109</td>
      <td>-0.205680</td>
      <td>-0.434799</td>
      <td>-0.161557</td>
      <td>-0.303987</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.035789</td>
      <td>0.097352</td>
      <td>0.081780</td>
      <td>0.284030</td>
      <td>-0.199692</td>
      <td>-0.198152</td>
      <td>0.106102</td>
      <td>-0.500504</td>
      <td>-0.035115</td>
      <td>0.094163</td>
      <td>0.137457</td>
      <td>0.116878</td>
      <td>-0.056067</td>
      <td>0.197648</td>
      <td>-0.640200</td>
      <td>0.257408</td>
    </tr>
  </tbody>
</table>
</div>



**PC1:** The first principal component seems to be composed of the factors reflecting how busy an airport is. It weights number of departures, arrivals, diversions, and cancellations relatively equally, as well as factors related to taxiing. An airport high on PC1 likely has longer delays at any point in an airplane's journey.

**PC2:** The second principal component negatively weights gate and arrival delays. An airport high on PC2 is more likely to have shorter delays. Since PC2 also negatively weights longitude, high PC2 score may correspond to being further east.

**PC3:** The third principal component negatively weights airborne delays and taxi out time and delays. An airport high in PC3 has shorter airborne delays and takes longer to taxi to the runway. This component strongly accounts for latitude, so an airport high in PC3 is more likely to be in the south of the U.S.


```python
# plot against percent on-time departures
fig = plt.figure(figsize = (10,8))
ax = fig.add_subplot(111, projection='3d')
p = ax.scatter(
    Y['PC1'], Y['PC2'], Y['PC3'],
    zdir='x', s = 10, c = X['percent on-time airport departures'],
    cmap = 'gist_heat', depthshade=True)
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")
ax.set_zlabel("PC3")
ax.set_title("Percent On-Time Departures vs. Principal Components")
fig.colorbar(p, shrink=0.5, aspect=5)
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_35_0.png)


This shows the airports plotted in the dimension of the three principal components. The colors represent the percent on-time departures. Airports with the darkest color are those with the most frequent delays. These seem to be clustered in the high end of PC1 and the lower range of PC3.



![png](../../../images/project_7_faa_files/project_7_faa_37_0.png)


In this plot, the color represents average duration of gate delays. It's clear that the longest delays correspond to the edges of the clusters that also had the lowest percent of on-time departures.


```python
# Show 2 example airports
fig = plt.figure(figsize = (10,8))
ax = fig.add_subplot(111, projection='3d')
p = ax.scatter(
    Y['PC1'], Y['PC2'], Y['PC3'],
    zdir='x', s = 10, c = X['percent on-time airport departures'],
    cmap = 'gist_heat', depthshade=True)

# make a mask for JFK and OGG and plot them in different colors
mask = ((Y['airport'] == 'ATL') | (Y['airport'] == 'OGG') | (Y['airport'] == 'HPN'))

color_dict = {'ATL':'green', 'OGG': 'blue', 'HPN': 'black'}
color_map = [color_dict[i] for i in Y[mask]['airport']]

ax.scatter(Y[mask]['PC1'], Y[mask]['PC2'], Y[mask]['PC3'], s = 10, c = color_map, zdir='x')
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")
ax.set_zlabel("PC3")
ax.set_title("Percent On-Time Departures vs. Principal Components")
fig.colorbar(p, shrink=0.5, aspect=5)
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_39_0.png)


This is the same graph, but I've indicated three example airports, ATL, OGG, and HPN in green, blue, and black, respectively. ATL is one of the busiest airports in the U.S., but has an above on-time departure rate and a higher than average departure delay time. OGG, in contrast, is located on the island of Maui and is much less busy, but with the highest on-time departure rate and one of the lowest average departure delays. HPN, though also a small airport, has a low on-time departure rate and long average departure delays.


```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

color_dict = {0:'blue', 1: 'green', 2: 'red'}
color_map = [color_dict[i] for i in Y['busy']]
fig = plt.figure(figsize = (10,8))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(Y['PC1'], Y['PC2'], Y['PC3'], zdir='x', s = 10, c=color_map, depthshade=True)
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")
ax.set_zlabel("PC3")
ax.set_title("Flight Volume vs Principal Components")
plt.show()
```


![png](../../../images/project_7_faa_files/project_7_faa_41_0.png)


In this graph, the colors represent arrival and departing flight volume. Red corresponds to over 300,000 flights in a year, green is between 300,000 and 100,000, and blue represents those airports with fewer than 100,000 flights. These flight volumes roughly seem to correspond to three clusters in the points. Contrary to what I might have predicted, the flights with the lowest percent of on-time departures are located in the medium and low-volume airport clusters.

# Conclusions and Recommendations

Clearly, reducing flight delays is a complex challenge! I would recommend that the FAA focus efforts on reducing delays at the gate for departing flights. This factor seems to have a cascading effect on airport departure delays, as well as arrival times for incoming flights that can't access their gates. For larger airports, it may be worth investigating taxi-in delays, but this effect is probably less important at smaller ones. Surprisingly, the majority of those airports with the most frequent and longest delays are not the busiest, so it may be worthwhile to direct some attention to these locations. Overall, the key to reducing both frequency and duration of departure delays seems to be in the reduction of gate departure delays.

---
layout: post
title: Principal Component Analysis of FAA Flight Delays
---
# Using Data to Reduce Delays

Given operational data for 74 airports over a 10 year period, I conducted a principal component analysis to examine underlying factors impacting flight delays. The data contains durations of gate, taxiing, and total delays for both arrivals and departures, volume of arrivals, departures, cancellations, and diversions, as well as the percent of flights that were delayed arriving or departing.

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

These data seem deceptively straightforward. Longer delays mean fewer on-time flights, right? There's a bit more to it. First, I want to differentiate between how frequently an airport has a late flight (the percent on-time columns for arrivals and departures), and how late it is (the average delay columns). Having every flight arrive one minute late may not have the same effect as having 5% of the flights arrive 1 hour late.

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


```python
# plot histograms for arrival delays
fig = plt.figure(figsize = (14,6))
count = 1
for col in ['average airborne delay', 'average taxi in delay','average gate arrival delay']:
    ax = fig.add_subplot(1,3,count)
    ax.hist(data[col], color = "navy")
    ax.set_xlabel(col)
    count +=1
fig.suptitle('Distributions of Arrival Delays', fontsize = 20)
plt.show()
fig.tight_layout()
```


![png](../../../images/project_7_faa_files/project_7_faa_10_0.png)


From these histograms, I can see the average airborne and taxi-in delays are between 2-3 and 1-2 minutes respectively, but the average gate arrival delay is significantly longer, between 10-15 minutes. Average gate arrival is defined as the difference between scheduled and actual arrival times, and I hypothesize that this difference between average gate arrival delay and the other arrival delays actually spills over from departure delays.


```python
# plot histograms for on time departures and arrivals
fig = plt.figure(figsize = (14,6))
count = 1
for col in ['percent on-time airport departures', 'percent on-time gate arrivals', 'average block delay']:
    ax = fig.add_subplot(1,3,count)
    ax.hist(data[col], color = "navy")
    ax.set_xlabel(col)
    count +=1
fig.suptitle('Distributions of Percent On-Time Departures and Arrivals', fontsize = 20)
plt.show()
fig.tight_layout()


```


![png](../../../images/project_7_faa_files/project_7_faa_12_0.png)


Here, I can see that in fact, most flights do depart and arrive on time. The majority of airports had between 70 and 80% of their flights leave and arrive on time. Average block delay is difference in minutes between scheduled and actual gate-to-gate time. The fact that this is lower than the average departure and arrival times may be explained by the fact that it includes on-time flights, and possibly flights that arrive early.


```python
# plot histograms for on time departures and arrivals
fig = plt.figure(figsize = (14,6))
count = 1
for col in ['departures for metric computation', 'arrivals for metric computation']:
    ax = fig.add_subplot(1,2,count)
    ax.hist(data[col], color = "navy")
    ax.set_xlabel(col)
    count +=1
fig.suptitle('Distributions of Total Arrivals and Departures', fontsize = 20)
plt.show()
fig.tight_layout()


```


![png](../../../images/project_7_faa_files/project_7_faa_14_0.png)


The arrival and departure data are heavily left skewed. The majority of airports had fewer than 100,000 flights pass through in a given year, but there are a few that had over 400,000 flights.


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


![png](../../../images/project_7_faa_files/project_7_faa_16_0.png)


This graph contains the percent of on time departures and the average delay time for a few select airports of varying sizes and locations to demonstrate that there isn't necessarily a clear relationship between the two. The airports with the longest delay times are JFK, John F. Kennedy International Airport and HPN, the Westchester County Airport. Although they are both in the same Northeast region promixal to NYC, JFK is a busy international airport whereas HPN is serviced by only seven airlines with no international flights. Although both JFK and HPN have high average delay times, OGG (the Kahului Airport in Hawaii) has a very short


```python
X = data[[u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'percent on-time gate departures','percent on-time airport departures',
       u'average_gate_departure_delay', u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay', u'average airborne delay',  u'arrivals for metric computation',
          u'Arrival Cancellations', u'Arrival Diversions', u'percent on-time gate arrivals', u'average taxi in delay',
       u'average block delay', u'average gate arrival delay']]
```


```python
plt.figure(figsize = (6,6))
sns.heatmap(X.corr())
plt.show()
```


![png](project_7_faa_files/project_7_faa_19_0.png)


This heatmap is arranged with all departure-related features on its top and left halves and all arrival-related features on the bottom and right halves. The blue bands are percent on-time arrivals, departures, and gate departures, which are obviously negatively correlated with all delay metrics. In general, all delay features seem to be positively correlated with each other, there are a few specific observations to make.

Average gate departure delays are strongly correlated with both average airport departure delays, as well as gate arrival delays. This makes sense, because if a plane is late leaving its gate, it will be behind schedule taking off, and an arriving plane can't pull into the gate until the first plane has left. In fact, average gate and aiport departure delays are more strongly correlated with average gate arrival delay than taxi-in delays.

There are strong correlations between total arrivals, total departures, and number of cancellations and diversions. This is all a consequence of simply having more flights coming and going to busy airports. However, there is one delay-related factor that seems to be correlated with all of these: taxi-in delay. Taxi-in delay is weakly correlated with percent on-time departures, so this may be a problem only at airports of a certain size.

These correlations point to delays originating with departing flights, as the effect of staying longer than anticipated at the gate makes the flight itself late to take off, but also prevents incoming flights from pulling into the gate at their scheduled times.

# Principal Component Analysis


```python
# normalize the data and remove percent columns
Xt = scale(X[[u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'average_gate_departure_delay', u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay', u'average airborne delay',  u'arrivals for metric computation',
          u'Arrival Cancellations', u'Arrival Diversions', u'average taxi in delay', u'average block delay',
              u'average gate arrival delay']])
```


```python
pca = PCA(n_components = 14)
pca = pca.fit(Xt)
```


```python
# make dataframe of principal components
Y = pd.DataFrame(pca.fit_transform(Xt), columns = ['PC{}'.format(str(i)) for i in range(1,15)])
Y['region'] = data['FAA REGION']
Y['busy'] = [
    2 if x >= 300000 else 1 if (x < 300000 and x > 100000) else 0 for x in data['departures for metric computation']
]
Y['airport'] = data['airport']
```


```python
exp_variances = np.cumsum(pca.explained_variance_ratio_*100)
```


```python
plt.figure(figsize = (6,6))
plt.plot(range(1, 15), exp_variances, '-o')
plt.xlabel("Number of components")
plt.ylabel("Cumulative % Explained Variance")
plt.show()
```


![png](project_7_faa_files/project_7_faa_26_0.png)



```python
# scree plot of eigenvalues from pca.explained_variance
plt.figure(figsize = (8,6))
plt.plot(range(1,15), pca.explained_variance_, "-o")
plt.xticks(range(1,15))
plt.ylabel("Eigenvalues")
plt.xlabel("PCA Components")
plt.show()
```


![png](project_7_faa_files/project_7_faa_27_0.png)


From examining the cumulative explained variance curve as well as the eigenvalues plot, I will select the first three principle components, as these will account for approximately 85% of the variance in the data and this seems to be where an "elbow" occurs in the eigenvalue plot.


```python
# eigenvectors are in pca.components_
eigenvectors = pd.DataFrame(pca.components_, columns = [u'departures for metric computation', u'Departure Cancellations',
          u'Departure Diversions', u'average_gate_departure_delay', u'average_taxi_out_time', u'average taxi out delay',
          u'average airport departure delay', u'average airborne delay',  u'arrivals for metric computation',
          u'Arrival Cancellations', u'Arrival Diversions', u'average taxi in delay', u'average block delay',
              u'average gate arrival delay'])
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
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.302891</td>
      <td>0.302562</td>
      <td>0.292481</td>
      <td>0.158552</td>
      <td>0.286629</td>
      <td>0.288579</td>
      <td>0.255493</td>
      <td>0.203416</td>
      <td>0.302640</td>
      <td>0.302210</td>
      <td>0.289626</td>
      <td>0.299519</td>
      <td>0.217245</td>
      <td>0.173862</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.255345</td>
      <td>0.138365</td>
      <td>0.268606</td>
      <td>-0.444343</td>
      <td>-0.106630</td>
      <td>-0.123655</td>
      <td>-0.379182</td>
      <td>-0.197965</td>
      <td>0.256646</td>
      <td>0.154566</td>
      <td>0.182089</td>
      <td>0.130730</td>
      <td>-0.264484</td>
      <td>-0.475898</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-0.007805</td>
      <td>0.234070</td>
      <td>0.109756</td>
      <td>0.432053</td>
      <td>-0.394528</td>
      <td>-0.372579</td>
      <td>0.129877</td>
      <td>-0.506134</td>
      <td>-0.006141</td>
      <td>0.234982</td>
      <td>0.150449</td>
      <td>0.017603</td>
      <td>-0.188877</td>
      <td>0.255056</td>
    </tr>
  </tbody>
</table>
</div>



**PC1:** The first principal component seems to be composed of the factors reflecting how busy an airport is. It weights number of departures, arrivals, diversions, and cancellations relatively equally, as well as factors related to taxiing. An airport high on PC1 likely has longer delays at any point in an airplane's journey.

**PC2:** The second principal component negatively weights departure, arrival, gate, and block delays, so an airport high on PC2 likely has shorter delays.

**PC3:** The third principal component negatively weights airborne delays and taxi out time and delays. An airport high in PC3 has longer airborne delays and takes longer to taxi to the runway.


```python
# plot against percent departure delays
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


![png](project_7_faa_files/project_7_faa_31_0.png)



```python
# Show 2 example airports
fig = plt.figure(figsize = (10,8))
ax = fig.add_subplot(111, projection='3d')
p = ax.scatter(
    Y['PC1'], Y['PC2'], Y['PC3'],
    zdir='x', s = 10, c = X['percent on-time airport departures'],
    cmap = 'gist_heat', depthshade=True)

# make a mask for JFK and OGG and plot them in different colors
mask = ((Y['airport'] == 'HPN') | (Y['airport'] == 'OGG'))

color_dict = {'HPN':'green', 'OGG': 'blue'}
color_map = [color_dict[i] for i in Y[mask]['airport']]

ax.scatter(Y[mask]['PC1'], Y[mask]['PC2'], Y[mask]['PC3'], s = 10, c = color_map)
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")
ax.set_zlabel("PC3")
ax.set_title("Percent On-Time Departures vs. Principal Components")
fig.colorbar(p, shrink=0.5, aspect=5)
plt.show()
```


![png](project_7_faa_files/project_7_faa_32_0.png)



```python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D

color_dict = {
    'ASW': u'orchid', 'AAL': u'darkcyan', 'ASO': u'grey', 'ANE': u'dodgerblue',
    'AEA': u'turquoise', 'AWP': u'darkviolet', 'AGL': '#624ea7', 'ANM': 'maroon', 'ACE': 'k'
}
color_map = [color_dict[i] for i in Y['region']]
fig = plt.figure(figsize = (10,8))
ax = fig.add_subplot(111, projection='3d')
ax.scatter(Y['PC1'], Y['PC2'], Y['PC3'], zdir='x', s = 10, c=color_map, depthshade=False)
ax.set_xlabel("PC1")
ax.set_ylabel("PC2")
ax.set_zlabel("PC3")
ax.set_title("Flight Volume vs Principal Components")
ax.legend(labels = color_dict.keys())
plt.show()
```


![png](project_7_faa_files/project_7_faa_33_0.png)


This plot of


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


![png](project_7_faa_files/project_7_faa_35_0.png)



```python
# try dbscan to find best clusters
Y_cluster = Y[[u'PC1', u'PC2', u'PC3', u'PC4', u'PC5', u'PC6', u'PC7', u'PC8', u'PC9',
       u'PC10', u'PC11', u'PC12', u'PC13', u'PC14']]
db = DBSCAN(eps = 3, min_samples = 5)
db.fit(Y_cluster)
labels = db.labels_

fig, ax = plt.subplots(figsize = (6,6))
ax.scatter(Y_cluster['PC1'], Y_cluster['PC2'], c = labels)
plt.show()

print "Silhouette score: ", silhouette_score(Y_cluster[['PC1','PC2']], labels, metric = 'euclidean')
```


![png](project_7_faa_files/project_7_faa_36_0.png)


    Silhouette score:  0.587743805013



```python

```



```python
# import BWI passenger data as numpy array
import csv
import numpy as np
f = open('BWI_Passenger_Data.csv', 'rU')
reader = csv.reader(f)
BWI_list= list(reader)
BWI_data = np.array(BWI_list)
BWI_data.shape
```




    (44, 77)




```python
#get rid of first row
pos = (0,1,21,3,6,18,24)
BWI_data_full = BWI_data[:,pos]
headers = BWI_data_full[0]
headers
BWI_data = BWI_data_full[1:,]
```


```python
import matplotlib.pyplot as plt
year = BWI_data[:,0]
year = year[1: len(year)]
year
```

    /Users/teresaborcuch/anaconda2/lib/python2.7/site-packages/matplotlib/font_manager.py:273: UserWarning: Matplotlib is building the font cache using fc-list. This may take a moment.
      warnings.warn('Matplotlib is building the font cache using fc-list. This may take a moment.')





    array(['2010', '2010', '2010', '2010', '2010', '2010', '2010', '2010',
           '2010', '2010', '2010', '2010', '2011', '2011', '2011', '2011',
           '2011', '2011', '2011', '2011', '2011', '2011', '2011', '2011',
           '2012', '2012', '2012', '2012', '2012', '2012', '2012', '2012',
           '2012', '2012', '2012', '2012', '2013', '2013', '2013', '2013',
           '2013', '2013', '2013'], 
          dtype='|S26')




```python
month = BWI_data[:,1]
month = month[1: len(month)]
month
```




    array(['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep',
           'Oct', 'Nov', 'Dec', 'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun',
           'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec', 'Jan', 'Feb', 'Mar',
           'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec',
           'Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul'], 
          dtype='|S26')




```python
southwest = BWI_data[:,2]
southwest = southwest[1: len(southwest)]
southwest = [int(x) for x in southwest]
southwest

```




    [789954,
     586606,
     973477,
     990728,
     1041684,
     1111896,
     1171494,
     1128231,
     936313,
     1085361,
     980593,
     946909,
     856223,
     792089,
     1085356,
     1083186,
     1171942,
     1185864,
     1211951,
     1111306,
     995803,
     1114191,
     993604,
     964545,
     875519,
     887581,
     1124314,
     1149702,
     1208085,
     1263110,
     1294533,
     1224041,
     1005417,
     1056730,
     1023471,
     977816,
     863744,
     838814,
     1124850,
     1093212,
     1225818,
     1248484,
     1267677]




```python
# get total monthly average
months = ['Jan', 'Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec']
sw_month_avg = {}
for m in months:
    sw_month_avg[m] = []
    for row in BWI_data:
        if row[1] == m:
            sw_month_avg[m].append(int(row[2]))
sw_month_avg
```




    {'Apr': [990728, 1083186, 1149702, 1093212],
     'Aug': [1128231, 1111306, 1224041],
     'Dec': [946909, 964545, 977816],
     'Feb': [586606, 792089, 887581, 838814],
     'Jan': [789954, 856223, 875519, 863744],
     'Jul': [1171494, 1211951, 1294533, 1267677],
     'Jun': [1111896, 1185864, 1263110, 1248484],
     'Mar': [973477, 1085356, 1124314, 1124850],
     'May': [1041684, 1171942, 1208085, 1225818],
     'Nov': [980593, 993604, 1023471],
     'Oct': [1085361, 1114191, 1056730],
     'Sep': [936313, 995803, 1005417]}




```python
sw_month_avg[]
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-60-cf182fa1f0e9> in <module>()
    ----> 1 plt.bar(sw_month_avg)
    

    TypeError: bar() takes at least 2 arguments (1 given)



```python

```


```python

```

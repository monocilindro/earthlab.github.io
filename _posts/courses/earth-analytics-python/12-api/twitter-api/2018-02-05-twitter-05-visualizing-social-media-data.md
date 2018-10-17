---
layout: single
title: "Visualize Social Media Data in Python"
excerpt: "This lesson provides an example of visualizing twitter in Python. "
authors: ['Martha Morrissey', 'Leah Wasser','Carson Farmer']
modified: 2018-10-08
category: [courses]
class-lesson: ['social-media-Python']
permalink: /courses/earth-analytics/get-data-using-apis/map-tweet-locations-over-time-python/
nav-title: 'Map Tweet Locations'
week: 12
sidebar:
    nav:
author_profile: false
comments: true
order: 5
course: "earth-analytics-python"
topics:
    find-and-manage-data: ['apis']
---
{% include toc title = "In This Lesson" icon="file-text" %}

<div class='notice--success' markdown="1">

## <i class="fa fa-graduation-cap" aria-hidden="true"></i> Learning Objectives

After completing this tutorial, you will be able to:

* Use  to create a static map of social media activity.
* Use `folium` to create an interactive map of social media activity.
* Use  to create an antimated gif file of social media activity.

## <i class="fa fa-check-square-o fa-2" aria-hidden="true"></i> What You Need

You will need a computer with internet access to complete this lesson.

</div>


In the previous lesson, you used text mining approaches to understand what people
were tweeting about during the flood. Here, you will create a map that shows the
location from where people were tweeting during the flood.

Keep in mind that these data have already been filtered to only include tweets that
at the time of the flood event, had an x, y location associated with them.
Thus this map doesn't represent all of the tweets that may be related to the flood
event.

You will work with the `Python` package `folium` to create visualizations. 

{:.input}
```python
import tweepy 
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import nltk
from nltk.tokenize import word_tokenize
from nltk.tokenize import RegexpTokenizer
from nltk.corpus import stopwords
import re

import folium
import json
from pandas.io.json import json_normalize
```

{:.input}
```python
import os
import earthpy as et
os.chdir(os.path.join(et.io.HOME, 'earth-analytics'))
```

## Load and Clean Data

{:.input}
```python
tweets = []
for line in open('data/twitter_data/boulder_flood_geolocated_tweets.json', 'r'):
    tweets.append(json.loads(line))
```

{:.input}
```python
df = json_normalize(tweets)
```

{:.input}
```python
df.columns
```

{:.output}
{:.execute_result}



    Index(['contributors', 'coordinates', 'coordinates.coordinates',
           'coordinates.type', 'created_at', 'entities.hashtags', 'entities.media',
           'entities.symbols', 'entities.urls', 'entities.user_mentions',
           ...
           'user.profile_text_color', 'user.profile_use_background_image',
           'user.protected', 'user.screen_name', 'user.statuses_count',
           'user.time_zone', 'user.translator_type', 'user.url', 'user.utc_offset',
           'user.verified'],
          dtype='object', length=241)





{:.input}
```python
df['coordinates.coordinates']
```

{:.output}
{:.execute_result}



    0        [-118.10041174, 34.14628356]
    1                                 NaN
    2           [0.13429814, 52.22500698]
    3                                 NaN
    4        [144.98467167, -37.80312131]
    5                                 NaN
    6                                 NaN
    7                                 NaN
    8                                 NaN
    9                                 NaN
    10                                NaN
    11                                NaN
    12       [-105.27725101, 40.01544423]
    13        [-105.2769568, 40.01579474]
    14                                NaN
    15       [-105.14533765, 39.98767028]
    16       [-105.23082163, 39.99946795]
    17       [-118.36457491, 33.90057323]
    18                                NaN
    19       [-105.22207764, 39.98664341]
    20                                NaN
    21                                NaN
    22                                NaN
    23        [-74.12520647, 40.10432876]
    24       [-105.03197389, 39.93687082]
    25                                NaN
    26                                NaN
    27                                NaN
    28                                NaN
    29                                NaN
                         ...             
    18791                             NaN
    18792                             NaN
    18793                             NaN
    18794                             NaN
    18795                             NaN
    18796                             NaN
    18797                             NaN
    18798                             NaN
    18799                             NaN
    18800    [-104.96650623, 39.65430591]
    18801                             NaN
    18802                             NaN
    18803                             NaN
    18804                             NaN
    18805                             NaN
    18806                             NaN
    18807                             NaN
    18808                             NaN
    18809                             NaN
    18810                             NaN
    18811                             NaN
    18812                             NaN
    18813                             NaN
    18814                             NaN
    18815                             NaN
    18816                             NaN
    18817                             NaN
    18818                             NaN
    18819                             NaN
    18820                             NaN
    Name: coordinates.coordinates, Length: 18821, dtype: object





{:.input}
```python
df_small = df[['text', 'coordinates.coordinates', 'created_at']]
```

{:.input}
```python
df_small.shape
```

{:.output}
{:.execute_result}



    (18821, 3)





{:.input}
```python
del df
```

{:.input}
```python
df_small = df_small.dropna()
```

{:.input}
```python
df_small.shape
```

{:.output}
{:.execute_result}



    (3943, 3)





{:.input}
```python
df_small.head()
```

{:.output}
{:.execute_result}



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
      <th>coordinates.coordinates</th>
      <th>created_at</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Boom bitch get out the way! #drunk #islands #g...</td>
      <td>[-118.10041174, 34.14628356]</td>
      <td>Tue Dec 31 07:14:22 +0000 2013</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Story of my life! 😂 #boulder http://t.co/ZMfNK...</td>
      <td>[0.13429814, 52.22500698]</td>
      <td>Mon Dec 30 20:29:20 +0000 2013</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Happy New Year #Boulder !!!! What are some of ...</td>
      <td>[144.98467167, -37.80312131]</td>
      <td>Wed Jan 01 06:12:15 +0000 2014</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Last sunset here in #boulder #flatirons @ Dush...</td>
      <td>[-105.27725101, 40.01544423]</td>
      <td>Tue Dec 31 22:48:10 +0000 2013</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Ringing in the #NewYear @BMoCA for their NYE a...</td>
      <td>[-105.2769568, 40.01579474]</td>
      <td>Wed Jan 01 04:57:01 +0000 2014</td>
    </tr>
  </tbody>
</table>
</div>





{:.input}
```python
longitude = [x[0] for x in df_small['coordinates.coordinates']]
```

{:.input}
```python
df_small['coordinates.coordinates'][0][0]
```

{:.input}
```python
latitude = [x[1] for x in df_small['coordinates.coordinates']]
```

{:.input}
```python
df_small['latitude'] = latitude
df_small['longitude'] = longitude
```

{:.input}
```python
type(latitude[0])
```

{:.output}
{:.execute_result}



    float





{:.input}
```python
locations = df_small[['latitude', 'longitude']]
coords = locations.values.tolist()
```

## Cluster and Visualize Tweets Spatially

{:.input}
```python

icon_create_function = """\
function(cluster) {
    return L.divIcon({
    html: '<b>' + cluster.getChildCount() + '</b>',
    className: 'marker-cluster marker-cluster-large',
    iconSize: new L.Point(20, 20)
    });
}"""
```

{:.input}
```python
from folium.plugins import MarkerCluster


m = folium.Map(
    location= [40.01, -105.27],
    tiles='Cartodb Positron',
    zoom_start=1
)

marker_cluster = MarkerCluster(
    locations= list(zip(latitude, longitude)),
    name='1000 clustered icons',
    overlay=True,
    control=True,
    icon_create_function=icon_create_function
)

marker_cluster.add_to(m)

folium.LayerControl().add_to(m)
m
```

{:.output}
{:.execute_result}








### Reflecting about your map

* Do any of the clusters in the maps suprise you? 
* Where is the largest cluster of tweets?

{:.input}
```python
location= list(zip(latitude, longitude))
```

{:.input}
```python
location[0]
```

{:.output}
{:.execute_result}



    (34.14628356, -118.10041174)





## Visualize Spatially and Temporally

To make an animated visualization you'll need to work with the twitter data field created at and use the `folium` plugin `Timestamped GeoJSON`.

Check the format of the created_at column in your `DataFrame` to make sure it is in an appropriate time format and not a `string`!

{:.input}
```python
df_small['created_at'][0]
```

{:.output}
{:.execute_result}



    'Tue Dec 31 07:14:22 +0000 2013'





{:.input}
```python
from datetime import datetime
#timestr = 'Mon Oct 27 23:00:03 +0000 2014'
datetime.strptime(timestr, '%a %b %d %X %z %Y')
```

{:.output}
{:.execute_result}



    datetime.datetime(2014, 10, 27, 23, 0, 3, tzinfo=datetime.timezone.utc)





{:.input}
```python
from datetime import datetime
datetime.strptime('Tue Dec 31 07:14:22 +0000 2013', '%a %b %d %X %z %Y')
```

{:.output}
{:.execute_result}



    datetime.datetime(2013, 12, 31, 7, 14, 22, tzinfo=datetime.timezone.utc)





{:.input}
```python
dt_list = [datetime.strptime(x, '%a %b %d %X %z %Y') for x in df_small['created_at']]
```

{:.input}
```python
dt_list[:5]
```

{:.output}
{:.execute_result}



    [datetime.datetime(2013, 12, 31, 7, 14, 22, tzinfo=datetime.timezone.utc),
     datetime.datetime(2013, 12, 30, 20, 29, 20, tzinfo=datetime.timezone.utc),
     datetime.datetime(2014, 1, 1, 6, 12, 15, tzinfo=datetime.timezone.utc),
     datetime.datetime(2013, 12, 31, 22, 48, 10, tzinfo=datetime.timezone.utc),
     datetime.datetime(2014, 1, 1, 4, 57, 1, tzinfo=datetime.timezone.utc)]




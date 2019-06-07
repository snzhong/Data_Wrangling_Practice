
# Data Wrangling Practice

## Table of Contents
<ul>
<li><a href="#intro">Introduction</a></li>
<li><a href="#gather">Data Gathering</a></li>
<li><a href="#assess">Data Assessment</a></li>
- <a href="#visual_assess">Visual Data Assessment</a><br>
- <a href="#prog_assess">Programmatic Data Assessment</a>
<li><a href="#issues">Identified Data Issues</a></li>
<li><a href="#clean">Data Cleaning</a></li>
<li><a href="#analysis">Data Analysis</a></li>
</ul>

<a id='intro'></a>
## Introduction

Below, I practiced data wrangling techniques on the a WeRateDogs tweet archive starting from their first tweet to 8/1/2017. The data came in three sets: a twitter archive csv file on hand (given by Udacity), a tweet image predictions file programmatically downloaded from Udacity servers, and corresponding tweet json data downloaded from Twitter via their API.

In addition to data wrangling, I also did some light analysis and visualization.

> **Note:** I will not be completely cleaning the data. I will only be cleaning 8 data quality issues and 2 tidiness issues. Additionally, instead of analyzing all of their tweets, I will only be analyzing the tweets that contained original WeRateDog dog ratings, while also leaving out retweets, etc.

<a id='gather'></a>
## Data Gathering


```python
import requests
import os
import pandas as pd
import tweepy
import time
import json
from pandas.io.json import json_normalize
import numpy as np
import matplotlib.pyplot as plt
% matplotlib inline
```


```python
# Creating manually downloaded WeRateDogs twitter archive dataframe
twitter_archive = pd.read_csv('twitter-archive-enhanced.csv')
twitter_archive.head()
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421...</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Downloading 'image_predictions.tsv' file from Udacity servers
url_file = 'https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv'
response = requests.get(url_file)

with open('image_predictions.tsv', mode='wb') as file:
    file.write(response.content)
```


```python
# Creating programmatically downloaded 'image_predictions.tsv' dataframe
image_predictions = pd.read_csv('image_predictions.tsv', sep='\t')
image_predictions.head()
```




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
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.061428</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.074192</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.072010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.138584</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.116197</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Creating a Twitter API object for gathering Twitter data
consumer_key = 'CONSUMER KEY'
consumer_secret = 'CONSUMER SECRET'
access_token = 'ACCESS TOKEN'
access_secret = 'ACCESS SECRET'

auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

api = tweepy.API(auth, wait_on_rate_limit=True, wait_on_rate_limit_notify = True)
```


```python
# Creating list of Tweet ID's from WeRateDogs twitter archive
tweet_id_list = twitter_archive['tweet_id'].tolist()
print(tweet_id_list[:5])
```

    [892420643555336193, 892177421306343426, 891815181378084864, 891689557279858688, 891327558926688256]
    


```python
# Gathering Tweets from available Tweet ID's
start = time.time()
count = 0
failed_tweet_list = []
with open('tweet_json.txt', 'w') as text_file:
    for tweet_id in tweet_id_list:
        count += 1
        try:
            tweet = api.get_status(tweet_id, tweet_mode='extended')
            json.dump(tweet._json, text_file)
            text_file.write('\n')
        except tweepy.TweepError:
            print('Tweep Error')
            failed_tweet_list.append(tweet_id)
            pass
end = time.time()
print(end - start)
print(failed_tweet_list)
```

    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    Tweep Error
    1923.9752163887024
    [888202515573088257, 873697596434513921, 872668790621863937, 872261713294495745, 869988702071779329, 866816280283807744, 861769973181624320, 856602993587888130, 845459076796616705, 844704788403113984, 842892208864923648, 837012587749474308, 827228250799742977, 812747805718642688, 802247111496568832, 775096608509886464, 770743923962707968, 758740312047005698, 754011816964026368, 680055455951884288, 676957860086095872]
    


```python
# Checking number of failed tweet retrievals
print(len(failed_tweet_list))
```

    21
    


```python
# Reading Twitter json data to a Pandas dataframe
with open('tweet_json.txt') as json_file:
    data = [json.loads(line) for line in json_file]
tweet_json = pd.DataFrame.from_dict(json_normalize(data), orient='columns')
tweet_json.head()
```




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
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>display_text_range</th>
      <th>entities.hashtags</th>
      <th>entities.media</th>
      <th>entities.symbols</th>
      <th>entities.urls</th>
      <th>entities.user_mentions</th>
      <th>extended_entities.media</th>
      <th>...</th>
      <th>user.profile_text_color</th>
      <th>user.profile_use_background_image</th>
      <th>user.protected</th>
      <th>user.screen_name</th>
      <th>user.statuses_count</th>
      <th>user.time_zone</th>
      <th>user.translator_type</th>
      <th>user.url</th>
      <th>user.utc_offset</th>
      <th>user.verified</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Aug 01 16:23:56 +0000 2017</td>
      <td>[0, 85]</td>
      <td>[]</td>
      <td>[{'id': 892420639486877696, 'id_str': '8924206...</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[{'id': 892420639486877696, 'id_str': '8924206...</td>
      <td>...</td>
      <td>000000</td>
      <td>False</td>
      <td>False</td>
      <td>dog_rates</td>
      <td>10339</td>
      <td>None</td>
      <td>none</td>
      <td>https://t.co/N7sNNHAEXS</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>None</td>
      <td>None</td>
      <td>Tue Aug 01 00:17:27 +0000 2017</td>
      <td>[0, 138]</td>
      <td>[]</td>
      <td>[{'id': 892177413194625024, 'id_str': '8921774...</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[{'id': 892177413194625024, 'id_str': '8921774...</td>
      <td>...</td>
      <td>000000</td>
      <td>False</td>
      <td>False</td>
      <td>dog_rates</td>
      <td>10339</td>
      <td>None</td>
      <td>none</td>
      <td>https://t.co/N7sNNHAEXS</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>None</td>
      <td>None</td>
      <td>Mon Jul 31 00:18:03 +0000 2017</td>
      <td>[0, 121]</td>
      <td>[]</td>
      <td>[{'id': 891815175371796480, 'id_str': '8918151...</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[{'id': 891815175371796480, 'id_str': '8918151...</td>
      <td>...</td>
      <td>000000</td>
      <td>False</td>
      <td>False</td>
      <td>dog_rates</td>
      <td>10339</td>
      <td>None</td>
      <td>none</td>
      <td>https://t.co/N7sNNHAEXS</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>None</td>
      <td>None</td>
      <td>Sun Jul 30 15:58:51 +0000 2017</td>
      <td>[0, 79]</td>
      <td>[]</td>
      <td>[{'id': 891689552724799489, 'id_str': '8916895...</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[{'id': 891689552724799489, 'id_str': '8916895...</td>
      <td>...</td>
      <td>000000</td>
      <td>False</td>
      <td>False</td>
      <td>dog_rates</td>
      <td>10339</td>
      <td>None</td>
      <td>none</td>
      <td>https://t.co/N7sNNHAEXS</td>
      <td>None</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>None</td>
      <td>None</td>
      <td>Sat Jul 29 16:00:24 +0000 2017</td>
      <td>[0, 138]</td>
      <td>[{'text': 'BarkWeek', 'indices': [129, 138]}]</td>
      <td>[{'id': 891327551943041024, 'id_str': '8913275...</td>
      <td>[]</td>
      <td>[]</td>
      <td>[]</td>
      <td>[{'id': 891327551943041024, 'id_str': '8913275...</td>
      <td>...</td>
      <td>000000</td>
      <td>False</td>
      <td>False</td>
      <td>dog_rates</td>
      <td>10339</td>
      <td>None</td>
      <td>none</td>
      <td>https://t.co/N7sNNHAEXS</td>
      <td>None</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 322 columns</p>
</div>



Since I will only be retaining 3 out of the 322 columns in the 'tweet_json' dataframe for the final master dataset (id, retweet_count, and favorite_count), I will drop the ones that won't be needed. I looked up the relevant column names from the twitter developer page.


```python
# Dropping non-relevant columns
tweet_json.drop(tweet_json.columns.difference(['id', 'retweet_count', 'favorite_count']), 1, inplace=True)
tweet_json.head()
```




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
      <th>favorite_count</th>
      <th>id</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>37481</td>
      <td>892420643555336193</td>
      <td>8163</td>
    </tr>
    <tr>
      <th>1</th>
      <td>32217</td>
      <td>892177421306343426</td>
      <td>6043</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24282</td>
      <td>891815181378084864</td>
      <td>3999</td>
    </tr>
    <tr>
      <th>3</th>
      <td>40807</td>
      <td>891689557279858688</td>
      <td>8309</td>
    </tr>
    <tr>
      <th>4</th>
      <td>39021</td>
      <td>891327558926688256</td>
      <td>9012</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Rearranging tweet_json dataframe to have 'id' first
tweet_json = tweet_json[['id', 'favorite_count', 'retweet_count']]
tweet_json.head()
```




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
      <th>id</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>37481</td>
      <td>8163</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>32217</td>
      <td>6043</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>24282</td>
      <td>3999</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>40807</td>
      <td>8309</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>39021</td>
      <td>9012</td>
    </tr>
  </tbody>
</table>
</div>



<a id='assess'></a>
## Data Assessment
#### Running various routine checks to look for data quality issues both programmatically and visually

<a id='visual_assess'></a>
### Visual Data Assessment
#### Additional visual data assessment was done in 3rd party spreadsheet apps


```python
twitter_archive.sample(5)
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1753</th>
      <td>678800283649069056</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-21 04:52:53 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here's a pupper with some mean tan lines. Snazzy sweater though 12/10 https://t.co/DpCSVsl6vu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/678800283649069056/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1864</th>
      <td>675362609739206656</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-11 17:12:48 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Daisy. She loves that shoe. Still no seat belt. Super churlish. 12/10 the dogs are killing it today https://t.co/cZlkvgRPdn</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675362609739206656/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Daisy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>372</th>
      <td>828381636999917570</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-02-05 23:15:47 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Meet Doobert. He's a deaf doggo. Didn't stop him on the field tho. Absolute legend today. 14/10 would pat head approvingly https://t.co/iCk7zstRA9</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/828381636999917570/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>Doobert</td>
      <td>doggo</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>518</th>
      <td>810657578271330305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-12-19 01:26:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Pavlov. His floatation device has failed him. He's quite pupset about it. 11/10 would rescue https://t.co/MXd0AGDsRJ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/810657578271330305/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>Pavlov</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1069</th>
      <td>740365076218183684</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-08 02:09:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>When the photographer forgets to tell you where to look... 10/10 https://t.co/u1GHWxhC85</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/740365076218183684/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
twitter_archive.sample(5)
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1708</th>
      <td>680798457301471234</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-26 17:12:55 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Say hello to Moofasa. He must be a powerful do...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/680798457...</td>
      <td>6</td>
      <td>10</td>
      <td>Moofasa</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1200</th>
      <td>716730379797970944</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-03 20:53:33 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>There has clearly been a mistake. Pup did noth...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/chpsanfrancisco/status/716...</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>861</th>
      <td>763103485927849985</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-08-09 20:03:43 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Belle. She's a Butterflop Hufflepoof. ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/763103485...</td>
      <td>10</td>
      <td>10</td>
      <td>Belle</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>616</th>
      <td>796484825502875648</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-11-09 22:49:15 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here's a sleepy doggo that requested some assi...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/796484825...</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>doggo</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1102</th>
      <td>735274964362878976</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-05-25 01:03:06 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>We only rate dogs. Please stop sending in your...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/735274964...</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
image_predictions.sample(5)
```




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
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>884</th>
      <td>698953797952008193</td>
      <td>https://pbs.twimg.com/media/CbMuxV5WEAAIBjy.jpg</td>
      <td>1</td>
      <td>Italian_greyhound</td>
      <td>0.382378</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.102255</td>
      <td>True</td>
      <td>shower_cap</td>
      <td>0.076834</td>
      <td>False</td>
    </tr>
    <tr>
      <th>789</th>
      <td>690597161306841088</td>
      <td>https://pbs.twimg.com/media/CZV-c9NVIAEWtiU.jpg</td>
      <td>1</td>
      <td>Lhasa</td>
      <td>0.097500</td>
      <td>True</td>
      <td>koala</td>
      <td>0.091934</td>
      <td>False</td>
      <td>sunglasses</td>
      <td>0.091505</td>
      <td>False</td>
    </tr>
    <tr>
      <th>857</th>
      <td>696877980375769088</td>
      <td>https://pbs.twimg.com/media/CavO0uuWEAE96Ed.jpg</td>
      <td>1</td>
      <td>space_heater</td>
      <td>0.206876</td>
      <td>False</td>
      <td>spatula</td>
      <td>0.123450</td>
      <td>False</td>
      <td>vacuum</td>
      <td>0.119218</td>
      <td>False</td>
    </tr>
    <tr>
      <th>456</th>
      <td>674774481756377088</td>
      <td>https://pbs.twimg.com/media/CV1HztsWoAAuZwo.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.407016</td>
      <td>True</td>
      <td>French_bulldog</td>
      <td>0.309978</td>
      <td>True</td>
      <td>Siamese_cat</td>
      <td>0.227677</td>
      <td>False</td>
    </tr>
    <tr>
      <th>27</th>
      <td>666396247373291520</td>
      <td>https://pbs.twimg.com/media/CT-D2ZHWIAA3gK1.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.978108</td>
      <td>True</td>
      <td>toy_terrier</td>
      <td>0.009397</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.004577</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
tweet_json.sample(5)
```




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
      <th>id</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1885</th>
      <td>674468880899788800</td>
      <td>6276</td>
      <td>2105</td>
    </tr>
    <tr>
      <th>773</th>
      <td>773985732834758656</td>
      <td>11233</td>
      <td>4145</td>
    </tr>
    <tr>
      <th>1819</th>
      <td>675888385639251968</td>
      <td>2416</td>
      <td>996</td>
    </tr>
    <tr>
      <th>1453</th>
      <td>693942351086120961</td>
      <td>1794</td>
      <td>385</td>
    </tr>
    <tr>
      <th>1375</th>
      <td>700002074055016451</td>
      <td>3416</td>
      <td>1403</td>
    </tr>
  </tbody>
</table>
</div>



<a id='prog_assess'></a>
### Programmatic Data Assessment


```python
twitter_archive.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
    tweet_id                      2356 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    doggo                         2356 non-null object
    floofer                       2356 non-null object
    pupper                        2356 non-null object
    puppo                         2356 non-null object
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB
    


```python
image_predictions.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2075 entries, 0 to 2074
    Data columns (total 12 columns):
    tweet_id    2075 non-null int64
    jpg_url     2075 non-null object
    img_num     2075 non-null int64
    p1          2075 non-null object
    p1_conf     2075 non-null float64
    p1_dog      2075 non-null bool
    p2          2075 non-null object
    p2_conf     2075 non-null float64
    p2_dog      2075 non-null bool
    p3          2075 non-null object
    p3_conf     2075 non-null float64
    p3_dog      2075 non-null bool
    dtypes: bool(3), float64(3), int64(2), object(4)
    memory usage: 152.1+ KB
    


```python
tweet_json.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2335 entries, 0 to 2334
    Data columns (total 3 columns):
    id                2335 non-null int64
    favorite_count    2335 non-null int64
    retweet_count     2335 non-null int64
    dtypes: int64(3)
    memory usage: 54.8 KB
    


```python
twitter_archive.describe()
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.356000e+03</td>
      <td>7.800000e+01</td>
      <td>7.800000e+01</td>
      <td>1.810000e+02</td>
      <td>1.810000e+02</td>
      <td>2356.000000</td>
      <td>2356.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.427716e+17</td>
      <td>7.455079e+17</td>
      <td>2.014171e+16</td>
      <td>7.720400e+17</td>
      <td>1.241698e+16</td>
      <td>13.126486</td>
      <td>10.455433</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.856705e+16</td>
      <td>7.582492e+16</td>
      <td>1.252797e+17</td>
      <td>6.236928e+16</td>
      <td>9.599254e+16</td>
      <td>45.876648</td>
      <td>6.745237</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>6.658147e+17</td>
      <td>1.185634e+07</td>
      <td>6.661041e+17</td>
      <td>7.832140e+05</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.783989e+17</td>
      <td>6.757419e+17</td>
      <td>3.086374e+08</td>
      <td>7.186315e+17</td>
      <td>4.196984e+09</td>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.196279e+17</td>
      <td>7.038708e+17</td>
      <td>4.196984e+09</td>
      <td>7.804657e+17</td>
      <td>4.196984e+09</td>
      <td>11.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.993373e+17</td>
      <td>8.257804e+17</td>
      <td>4.196984e+09</td>
      <td>8.203146e+17</td>
      <td>4.196984e+09</td>
      <td>12.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>8.862664e+17</td>
      <td>8.405479e+17</td>
      <td>8.874740e+17</td>
      <td>7.874618e+17</td>
      <td>1776.000000</td>
      <td>170.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Looking for abnormal numerator values
twitter_archive.rating_numerator.value_counts()
```




    12      558
    11      464
    10      461
    13      351
    9       158
    8       102
    7        55
    14       54
    5        37
    6        32
    3        19
    4        17
    1         9
    2         9
    420       2
    0         2
    15        2
    75        2
    80        1
    20        1
    24        1
    26        1
    44        1
    50        1
    60        1
    165       1
    84        1
    88        1
    144       1
    182       1
    143       1
    666       1
    960       1
    1776      1
    17        1
    27        1
    45        1
    99        1
    121       1
    204       1
    Name: rating_numerator, dtype: int64




```python
# Looking for abnormal denominator values
twitter_archive.rating_denominator.value_counts()
```




    10     2333
    11        3
    50        3
    80        2
    20        2
    2         1
    16        1
    40        1
    70        1
    15        1
    90        1
    110       1
    120       1
    130       1
    150       1
    170       1
    7         1
    0         1
    Name: rating_denominator, dtype: int64



**There are some suspicious denominators, as they should all be 10.**


```python
image_predictions.describe()
```




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
      <th>tweet_id</th>
      <th>img_num</th>
      <th>p1_conf</th>
      <th>p2_conf</th>
      <th>p3_conf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.075000e+03</td>
      <td>2075.000000</td>
      <td>2075.000000</td>
      <td>2.075000e+03</td>
      <td>2.075000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.384514e+17</td>
      <td>1.203855</td>
      <td>0.594548</td>
      <td>1.345886e-01</td>
      <td>6.032417e-02</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.785203e+16</td>
      <td>0.561875</td>
      <td>0.271174</td>
      <td>1.006657e-01</td>
      <td>5.090593e-02</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>1.000000</td>
      <td>0.044333</td>
      <td>1.011300e-08</td>
      <td>1.740170e-10</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.764835e+17</td>
      <td>1.000000</td>
      <td>0.364412</td>
      <td>5.388625e-02</td>
      <td>1.622240e-02</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.119988e+17</td>
      <td>1.000000</td>
      <td>0.588230</td>
      <td>1.181810e-01</td>
      <td>4.944380e-02</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.932034e+17</td>
      <td>1.000000</td>
      <td>0.843855</td>
      <td>1.955655e-01</td>
      <td>9.180755e-02</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>4.880140e-01</td>
      <td>2.734190e-01</td>
    </tr>
  </tbody>
</table>
</div>




```python
tweet_json.describe()
```




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
      <th>id</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.335000e+03</td>
      <td>2335.000000</td>
      <td>2335.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.420899e+17</td>
      <td>7831.524197</td>
      <td>2871.640257</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.825749e+16</td>
      <td>12139.567126</td>
      <td>4850.100318</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.783378e+17</td>
      <td>1355.000000</td>
      <td>577.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.185406e+17</td>
      <td>3406.000000</td>
      <td>1341.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.986846e+17</td>
      <td>9594.000000</td>
      <td>3347.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>161883.000000</td>
      <td>82258.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
image_predictions.p1.tolist()
```




    ['Welsh_springer_spaniel',
     'redbone',
     'German_shepherd',
     'Rhodesian_ridgeback',
     'miniature_pinscher',
     'Bernese_mountain_dog',
     'box_turtle',
     'chow',
     'shopping_cart',
     'miniature_poodle',
     'golden_retriever',
     'Gordon_setter',
     'Walker_hound',
     'pug',
     'bloodhound',
     'Lhasa',
     'English_setter',
     'hen',
     'desktop_computer',
     'Italian_greyhound',
     'Maltese_dog',
     'three-toed_sloth',
     'ox',
     'golden_retriever',
     'malamute',
     'guinea_pig',
     'soft-coated_wheaten_terrier',
     'Chihuahua',
     'black-and-tan_coonhound',
     'coho',
     'toy_terrier',
     'Blenheim_spaniel',
     'Pembroke',
     'llama',
     'Chesapeake_Bay_retriever',
     'Chihuahua',
     'curly-coated_retriever',
     'dalmatian',
     'Ibizan_hound',
     'Border_collie',
     'German_shepherd',
     'Labrador_retriever',
     'miniature_poodle',
     'seat_belt',
     'Italian_greyhound',
     'snail',
     'English_setter',
     'miniature_schnauzer',
     'Maltese_dog',
     'Airedale',
     'triceratops',
     'swab',
     'hay',
     'hyena',
     'golden_retriever',
     'Chesapeake_Bay_retriever',
     'jigsaw_puzzle',
     'Chihuahua',
     'Chihuahua',
     'Pembroke',
     'West_Highland_white_terrier',
     'toy_poodle',
     'golden_retriever',
     'miniature_pinscher',
     'giant_schnauzer',
     'toy_poodle',
     'soft-coated_wheaten_terrier',
     'vizsla',
     'golden_retriever',
     'vacuum',
     'Rottweiler',
     'Siberian_husky',
     'golden_retriever',
     'teddy',
     'papillon',
     'Saint_Bernard',
     'Rottweiler',
     'porcupine',
     'goose',
     'Labrador_retriever',
     'Tibetan_terrier',
     'toy_poodle',
     'borzoi',
     'Chihuahua',
     'Labrador_retriever',
     'beagle',
     'Italian_greyhound',
     'hare',
     'golden_retriever',
     'Pembroke',
     'Yorkshire_terrier',
     'Pomeranian',
     'toy_poodle',
     'electric_fan',
     'web_site',
     'web_site',
     'ibex',
     'kuvasz',
     'fire_engine',
     'West_Highland_white_terrier',
     'lorikeet',
     'dalmatian',
     'flat-coated_retriever',
     'toyshop',
     'miniature_pinscher',
     'malamute',
     'jigsaw_puzzle',
     'common_iguana',
     'seat_belt',
     'malamute',
     'Pomeranian',
     'Norwegian_elkhound',
     'frilled_lizard',
     'leatherback_turtle',
     'Labrador_retriever',
     'hamster',
     'Pembroke',
     'Angora',
     'Arctic_fox',
     'Chihuahua',
     'Blenheim_spaniel',
     'Labrador_retriever',
     'chow',
     'trombone',
     'chow',
     'Chihuahua',
     'canoe',
     'Pembroke',
     'soft-coated_wheaten_terrier',
     'golden_retriever',
     'web_site',
     'king_penguin',
     'shopping_basket',
     'Arctic_fox',
     'standard_poodle',
     'Staffordshire_bullterrier',
     'basenji',
     'Labrador_retriever',
     'Lakeland_terrier',
     'American_Staffordshire_terrier',
     'bearskin',
     'Shih-Tzu',
     'bustard',
     'crash_helmet',
     'Chihuahua',
     'ox',
     'French_bulldog',
     'miniature_schnauzer',
     'Pekinese',
     'komondor',
     'vacuum',
     'common_iguana',
     'ski_mask',
     'beagle',
     'Chesapeake_Bay_retriever',
     'redbone',
     'malinois',
     'golden_retriever',
     'teddy',
     'kelpie',
     'Brittany_spaniel',
     'standard_poodle',
     'cocker_spaniel',
     'shower_curtain',
     'basset',
     'golden_retriever',
     'jellyfish',
     'doormat',
     'Arabian_camel',
     'pug',
     'lynx',
     'hog',
     'Pembroke',
     'Chihuahua',
     'comic_book',
     'Chihuahua',
     'pug',
     'minivan',
     'golden_retriever',
     'seashore',
     'golden_retriever',
     'Siberian_husky',
     'cuirass',
     'teddy',
     'Chihuahua',
     'Labrador_retriever',
     'golden_retriever',
     'basset',
     'Brabancon_griffon',
     'Airedale',
     'cocker_spaniel',
     'toy_poodle',
     'Chihuahua',
     'minivan',
     'West_Highland_white_terrier',
     'candle',
     'Eskimo_dog',
     'Maltese_dog',
     'seat_belt',
     'weasel',
     'dalmatian',
     'Christmas_stocking',
     'Pomeranian',
     'washbasin',
     'Pembroke',
     'car_mirror',
     'Pomeranian',
     'vizsla',
     'miniature_pinscher',
     'teddy',
     'piggy_bank',
     'beagle',
     'pot',
     'web_site',
     'Blenheim_spaniel',
     'snail',
     'Italian_greyhound',
     'boathouse',
     'malamute',
     'mud_turtle',
     'German_short-haired_pointer',
     'toy_poodle',
     'Chihuahua',
     'Shetland_sheepdog',
     'Irish_terrier',
     'cairn',
     'platypus',
     'English_springer',
     'whippet',
     'pug',
     'ping-pong_ball',
     'Maltese_dog',
     'sea_urchin',
     'bow_tie',
     'Chihuahua',
     'seat_belt',
     'chow',
     'window_shade',
     "jack-o'-lantern",
     'sorrel',
     'Sussex_spaniel',
     'English_springer',
     'peacock',
     'Arctic_fox',
     'axolotl',
     'minivan',
     'wool',
     'banana',
     'Dandie_Dinmont',
     'Ibizan_hound',
     'Shetland_sheepdog',
     'Norwich_terrier',
     'Pomeranian',
     'wood_rabbit',
     'dhole',
     'keeshond',
     'Norfolk_terrier',
     'pug',
     'Labrador_retriever',
     'Chihuahua',
     'lacewing',
     'dingo',
     'beagle',
     'brown_bear',
     'Pembroke',
     'basenji',
     'Pomeranian',
     'Old_English_sheepdog',
     'basset',
     'American_Staffordshire_terrier',
     'web_site',
     'Labrador_retriever',
     'scorpion',
     'malinois',
     'Pekinese',
     'flamingo',
     'Shih-Tzu',
     'microphone',
     'redbone',
     'basenji',
     'malinois',
     'goose',
     'Shih-Tzu',
     'Samoyed',
     'guinea_pig',
     'Yorkshire_terrier',
     'Rottweiler',
     'Labrador_retriever',
     'pitcher',
     'African_hunting_dog',
     'refrigerator',
     'Rottweiler',
     'Chihuahua',
     'picket_fence',
     'miniature_poodle',
     'Italian_greyhound',
     'tub',
     'zebra',
     'Samoyed',
     'German_shepherd',
     'hermit_crab',
     'swing',
     'toy_poodle',
     'hermit_crab',
     'Lakeland_terrier',
     'Pomeranian',
     'Doberman',
     'hen',
     'pug',
     'park_bench',
     'Shetland_sheepdog',
     'feather_boa',
     'Loafer',
     'Gordon_setter',
     'kuvasz',
     'stone_wall',
     'toy_poodle',
     'ice_bear',
     'prayer_rug',
     'Chihuahua',
     'dalmatian',
     'chimpanzee',
     'Labrador_retriever',
     'china_cabinet',
     'bee_eater',
     'ski_mask',
     'Labrador_retriever',
     'chow',
     'pug',
     'hamster',
     'pug',
     'tennis_ball',
     'Rottweiler',
     'pug',
     'cocker_spaniel',
     'carton',
     'beagle',
     'killer_whale',
     'pug',
     'Chihuahua',
     'Irish_terrier',
     'pug',
     'ostrich',
     'Chihuahua',
     'Irish_terrier',
     'pug',
     'cocker_spaniel',
     'terrapin',
     'Border_collie',
     'West_Highland_white_terrier',
     'Doberman',
     'golden_retriever',
     'Siamese_cat',
     'gondola',
     'kuvasz',
     'Great_Pyrenees',
     'toy_poodle',
     'microwave',
     'starfish',
     'golden_retriever',
     'Chesapeake_Bay_retriever',
     'sandbar',
     'Pembroke',
     'Chihuahua',
     'tusker',
     'motor_scooter',
     'ram',
     'Chihuahua',
     'toy_poodle',
     'leaf_beetle',
     'pug',
     'car_mirror',
     'wombat',
     'Lakeland_terrier',
     'French_bulldog',
     'chow',
     'tub',
     'schipperke',
     'Airedale',
     'Shih-Tzu',
     'golden_retriever',
     'miniature_pinscher',
     'Samoyed',
     'ski_mask',
     'Chihuahua',
     'hamster',
     'West_Highland_white_terrier',
     'golden_retriever',
     'Rottweiler',
     'English_setter',
     'ox',
     'teddy',
     'beagle',
     'Arctic_fox',
     'Newfoundland',
     'wombat',
     'bull_mastiff',
     'Labrador_retriever',
     'Samoyed',
     'Pekinese',
     'soft-coated_wheaten_terrier',
     'Chesapeake_Bay_retriever',
     'porcupine',
     'water_bottle',
     'Shih-Tzu',
     'German_short-haired_pointer',
     'golden_retriever',
     'Chihuahua',
     'Maltese_dog',
     'suit',
     'Brabancon_griffon',
     'toilet_seat',
     "jack-o'-lantern",
     'pug',
     'jigsaw_puzzle',
     'Pembroke',
     'collie',
     'Pomeranian',
     'toy_poodle',
     'Eskimo_dog',
     'toy_poodle',
     'robin',
     'Shih-Tzu',
     'Cardigan',
     'ostrich',
     'Airedale',
     'Pomeranian',
     'Eskimo_dog',
     'Greater_Swiss_Mountain_dog',
     'slug',
     'pug',
     'German_shepherd',
     'Cardigan',
     'porcupine',
     'seashore',
     'toilet_tissue',
     'brown_bear',
     'Chihuahua',
     'golden_retriever',
     'acorn_squash',
     'Brabancon_griffon',
     'chow',
     'jellyfish',
     'Pomeranian',
     'soccer_ball',
     'flat-coated_retriever',
     'African_crocodile',
     'malamute',
     'tick',
     'Pomeranian',
     'Dandie_Dinmont',
     'Bernese_mountain_dog',
     'vizsla',
     'seashore',
     'Samoyed',
     'shower_curtain',
     'Chihuahua',
     'ocarina',
     'miniature_poodle',
     'Labrador_retriever',
     'giant_schnauzer',
     'Pembroke',
     'English_springer',
     'Rottweiler',
     'Pembroke',
     'boxer',
     'street_sign',
     'Samoyed',
     'dalmatian',
     'Labrador_retriever',
     'bow',
     'stove',
     'Labrador_retriever',
     'dingo',
     'golden_retriever',
     'chow',
     'paper_towel',
     'llama',
     'Pembroke',
     'borzoi',
     'upright',
     'Labrador_retriever',
     'Chihuahua',
     'Labrador_retriever',
     'box_turtle',
     'West_Highland_white_terrier',
     'vizsla',
     'dough',
     'Scottish_deerhound',
     'cocker_spaniel',
     'Pembroke',
     'chow',
     'English_springer',
     'bath_towel',
     'standard_schnauzer',
     'golden_retriever',
     'Maltese_dog',
     'beagle',
     'basset',
     'Pomeranian',
     'pug',
     'Labrador_retriever',
     'golden_retriever',
     'wood_rabbit',
     'West_Highland_white_terrier',
     'Italian_greyhound',
     'Labrador_retriever',
     'bull_mastiff',
     'walking_stick',
     'Shih-Tzu',
     'Irish_water_spaniel',
     'Chihuahua',
     'hamster',
     'bubble',
     'French_bulldog',
     'teddy',
     'golden_retriever',
     'Maltese_dog',
     'golden_retriever',
     'pug',
     'chow',
     'feather_boa',
     'seat_belt',
     'Boston_bull',
     'whippet',
     'Loafer',
     'book_jacket',
     'Chihuahua',
     'doormat',
     'Chihuahua',
     'rain_barrel',
     'Great_Pyrenees',
     'Chesapeake_Bay_retriever',
     'hamster',
     'Chesapeake_Bay_retriever',
     'black-footed_ferret',
     'Pekinese',
     'guenon',
     'Welsh_springer_spaniel',
     'Labrador_retriever',
     'malamute',
     'English_setter',
     'common_iguana',
     'Shetland_sheepdog',
     'Japanese_spaniel',
     'Blenheim_spaniel',
     'water_buffalo',
     'beagle',
     'Lakeland_terrier',
     'Staffordshire_bullterrier',
     'American_Staffordshire_terrier',
     'seat_belt',
     'basset',
     'patio',
     'Chihuahua',
     'cowboy_hat',
     'Maltese_dog',
     'Pembroke',
     'Shih-Tzu',
     'Siberian_husky',
     'teddy',
     'dalmatian',
     'Eskimo_dog',
     'miniature_pinscher',
     'Chihuahua',
     'Maltese_dog',
     'Norfolk_terrier',
     'golden_retriever',
     'dogsled',
     'miniature_pinscher',
     'Pembroke',
     'swing',
     'schipperke',
     'Maltese_dog',
     'Staffordshire_bullterrier',
     'Labrador_retriever',
     'maze',
     'seat_belt',
     'malamute',
     'Irish_terrier',
     'harp',
     'Pembroke',
     'Airedale',
     'Labrador_retriever',
     'Labrador_retriever',
     'Eskimo_dog',
     'panpipe',
     'cash_machine',
     'kelpie',
     'Scottish_deerhound',
     'Italian_greyhound',
     'pug',
     'toy_poodle',
     'Maltese_dog',
     'porcupine',
     'Chihuahua',
     'mailbox',
     'dalmatian',
     'boxer',
     'wallaby',
     'French_bulldog',
     'bloodhound',
     'Chihuahua',
     'Airedale',
     'llama',
     'EntleBucher',
     'hog',
     'Samoyed',
     'earthstar',
     'pillow',
     'golden_retriever',
     'pug',
     'Maltese_dog',
     'miniature_poodle',
     'bluetick',
     'Christmas_stocking',
     'Brittany_spaniel',
     'Christmas_stocking',
     'bubble',
     'space_heater',
     'Lakeland_terrier',
     'kuvasz',
     'Chihuahua',
     'tub',
     'French_bulldog',
     'ram',
     'pug',
     'Pembroke',
     'Shetland_sheepdog',
     'Pomeranian',
     'Labrador_retriever',
     'soft-coated_wheaten_terrier',
     'carousel',
     'pug',
     'Pomeranian',
     'Irish_setter',
     'motor_scooter',
     'Tibetan_terrier',
     'Saint_Bernard',
     'Lhasa',
     'frilled_lizard',
     'Samoyed',
     'seat_belt',
     'Norfolk_terrier',
     'Rottweiler',
     'toy_poodle',
     'Pomeranian',
     'birdhouse',
     'toy_poodle',
     'Chihuahua',
     'Labrador_retriever',
     'triceratops',
     'schipperke',
     'teddy',
     'jigsaw_puzzle',
     'snorkel',
     'Siberian_husky',
     'seat_belt',
     'curly-coated_retriever',
     'vizsla',
     'wombat',
     'whippet',
     'English_springer',
     'beagle',
     'bald_eagle',
     'English_setter',
     'toyshop',
     'dingo',
     'boxer',
     'koala',
     'golden_retriever',
     'cocker_spaniel',
     'Leonberg',
     'Pembroke',
     'French_bulldog',
     'Lakeland_terrier',
     'Italian_greyhound',
     'keeshond',
     'Airedale',
     'Pembroke',
     'miniature_pinscher',
     'malamute',
     'Maltese_dog',
     'hog',
     'toy_poodle',
     'bluetick',
     'Labrador_retriever',
     'cheetah',
     'chow',
     'kelpie',
     'Chihuahua',
     'llama',
     'soft-coated_wheaten_terrier',
     'golden_retriever',
     'Pembroke',
     'Labrador_retriever',
     'Chihuahua',
     'Chihuahua',
     'minibus',
     'Weimaraner',
     'miniature_schnauzer',
     'clog',
     'Pembroke',
     'shopping_cart',
     'Labrador_retriever',
     'Border_collie',
     'llama',
     'Bernese_mountain_dog',
     'dishwasher',
     'pug',
     'Pomeranian',
     'chow',
     'Boston_bull',
     'golden_retriever',
     'white_wolf',
     'web_site',
     'Lakeland_terrier',
     'Chesapeake_Bay_retriever',
     'sliding_door',
     'Yorkshire_terrier',
     'papillon',
     'Siberian_husky',
     'damselfly',
     'Rhodesian_ridgeback',
     'Great_Dane',
     'Pomeranian',
     'pug',
     'German_shepherd',
     'Yorkshire_terrier',
     'Labrador_retriever',
     'Rottweiler',
     'Norwich_terrier',
     'Tibetan_mastiff',
     'cheeseburger',
     'refrigerator',
     'Labrador_retriever',
     'fiddler_crab',
     'borzoi',
     'car_mirror',
     'pug',
     'seat_belt',
     'Shih-Tzu',
     'Boston_bull',
     'wood_rabbit',
     'English_springer',
     'Rottweiler',
     'pug',
     'miniature_pinscher',
     'Staffordshire_bullterrier',
     'Lakeland_terrier',
     'Pomeranian',
     'Eskimo_dog',
     'pug',
     'sorrel',
     'bannister',
     'golden_retriever',
     'Pembroke',
     'papillon',
     'American_Staffordshire_terrier',
     'malinois',
     'Cardigan',
     'hog',
     'Ibizan_hound',
     'crane',
     'Pembroke',
     'English_springer',
     'cocker_spaniel',
     'American_Staffordshire_terrier',
     'Chihuahua',
     'Scotch_terrier',
     'snowmobile',
     'Pembroke',
     'Eskimo_dog',
     'dogsled',
     'toy_poodle',
     'bustard',
     'Italian_greyhound',
     'collie',
     'Old_English_sheepdog',
     'Pomeranian',
     'cowboy_hat',
     'standard_poodle',
     'Samoyed',
     'malamute',
     'badger',
     'motor_scooter',
     'pug',
     'kuvasz',
     'Pembroke',
     'Lhasa',
     'bighorn',
     'bath_towel',
     'kuvasz',
     'golden_retriever',
     'snorkel',
     'geyser',
     'golden_retriever',
     'barrow',
     'borzoi',
     'Rottweiler',
     'Lakeland_terrier',
     'Bernese_mountain_dog',
     'Shetland_sheepdog',
     'bloodhound',
     'Chihuahua',
     'Saint_Bernard',
     'minivan',
     'Tibetan_terrier',
     'toy_poodle',
     'pug',
     'Siberian_husky',
     'bison',
     'Siberian_husky',
     'pug',
     'golden_retriever',
     'Samoyed',
     'Samoyed',
     'German_short-haired_pointer',
     'soft-coated_wheaten_terrier',
     'Mexican_hairless',
     'Saint_Bernard',
     'ice_lolly',
     'cocker_spaniel',
     'Shih-Tzu',
     'vizsla',
     'golden_retriever',
     'Labrador_retriever',
     'sea_lion',
     'dining_table',
     'malamute',
     'pug',
     'Scottish_deerhound',
     'washbasin',
     'groenendael',
     'Pembroke',
     'teddy',
     'Shih-Tzu',
     'boxer',
     'Australian_terrier',
     'hamster',
     'beaver',
     'bow_tie',
     'Boston_bull',
     'seat_belt',
     'Shih-Tzu',
     'papillon',
     'Maltese_dog',
     'pug',
     'basenji',
     'Old_English_sheepdog',
     'soft-coated_wheaten_terrier',
     'Samoyed',
     'Pomeranian',
     'borzoi',
     'briard',
     'car_mirror',
     'weasel',
     'space_heater',
     'kuvasz',
     'Appenzeller',
     'Chihuahua',
     'grey_fox',
     'Great_Dane',
     'Great_Dane',
     'toy_poodle',
     'Labrador_retriever',
     'bath_towel',
     'Siamese_cat',
     'Shetland_sheepdog',
     'Lhasa',
     'washbasin',
     'Great_Dane',
     'Pembroke',
     'Staffordshire_bullterrier',
     'Chesapeake_Bay_retriever',
     'Labrador_retriever',
     'Italian_greyhound',
     'boxer',
     'pug',
     'French_bulldog',
     'Samoyed',
     'Brittany_spaniel',
     'Samoyed',
     'German_short-haired_pointer',
     'Italian_greyhound',
     'Norfolk_terrier',
     'Chihuahua',
     'Shih-Tzu',
     'schipperke',
     'mousetrap',
     'Labrador_retriever',
     'cairn',
     'Samoyed',
     'pug',
     'golden_retriever',
     'Pembroke',
     'hippopotamus',
     'Dandie_Dinmont',
     'malinois',
     'Border_terrier',
     'golden_retriever',
     'Chihuahua',
     'West_Highland_white_terrier',
     'hummingbird',
     'golden_retriever',
     'tennis_ball',
     'beagle',
     'hamster',
     'bath_towel',
     'American_Staffordshire_terrier',
     'Great_Pyrenees',
     'tailed_frog',
     'Rhodesian_ridgeback',
     'kuvasz',
     'white_wolf',
     'Chihuahua',
     'Border_collie',
     'Yorkshire_terrier',
     'Chihuahua',
     'French_bulldog',
     'toy_poodle',
     'Pomeranian',
     'golden_retriever',
     'Boston_bull',
     'West_Highland_white_terrier',
     'Pomeranian',
     'kelpie',
     'bloodhound',
     'golden_retriever',
     'wallaby',
     'hippopotamus',
     'Pembroke',
     'wool',
     'Border_collie',
     'golden_retriever',
     'doormat',
     'miniature_pinscher',
     'Pembroke',
     'window_shade',
     'boxer',
     'Labrador_retriever',
     'Great_Pyrenees',
     'otter',
     'teddy',
     'Pembroke',
     'Samoyed',
     'Chihuahua',
     'Siamese_cat',
     'guinea_pig',
     'golden_retriever',
     'pug',
     'Norfolk_terrier',
     'Airedale',
     'collie',
     'Egyptian_cat',
     'Chihuahua',
     'Chihuahua',
     'Great_Pyrenees',
     'golden_retriever',
     'basenji',
     'web_site',
     'collie',
     'golden_retriever',
     'Staffordshire_bullterrier',
     'Samoyed',
     'papillon',
     'Border_terrier',
     'Pembroke',
     'Tibetan_mastiff',
     'golden_retriever',
     'chow',
     'four-poster',
     'space_heater',
     'toy_poodle',
     'wild_boar',
     'Chihuahua',
     'cocker_spaniel',
     'pug',
     'Samoyed',
     'Blenheim_spaniel',
     'Bernese_mountain_dog',
     'golden_retriever',
     'Chihuahua',
     'Lakeland_terrier',
     'ram',
     'golden_retriever',
     'bathtub',
     'whippet',
     'miniature_pinscher',
     'toy_poodle',
     'agama',
     'malinois',
     'Staffordshire_bullterrier',
     'Norwich_terrier',
     'French_bulldog',
     'Cardigan',
     'muzzle',
     'pug',
     'Chihuahua',
     'Shih-Tzu',
     ...]




```python
# Looking for abnormal predictions
image_predictions.p1.value_counts()
```




    golden_retriever             150
    Labrador_retriever           100
    Pembroke                      89
    Chihuahua                     83
    pug                           57
    chow                          44
    Samoyed                       43
    toy_poodle                    39
    Pomeranian                    38
    cocker_spaniel                30
    malamute                      30
    French_bulldog                26
    miniature_pinscher            23
    Chesapeake_Bay_retriever      23
    seat_belt                     22
    Siberian_husky                20
    Staffordshire_bullterrier     20
    German_shepherd               20
    web_site                      19
    Cardigan                      19
    Shetland_sheepdog             18
    beagle                        18
    Maltese_dog                   18
    Eskimo_dog                    18
    teddy                         18
    Rottweiler                    17
    Shih-Tzu                      17
    Lakeland_terrier              17
    kuvasz                        16
    Italian_greyhound             16
                                ... 
    hummingbird                    1
    cliff                          1
    panpipe                        1
    sliding_door                   1
    standard_schnauzer             1
    flamingo                       1
    piggy_bank                     1
    groenendael                    1
    polecat                        1
    king_penguin                   1
    bakery                         1
    ping-pong_ball                 1
    bonnet                         1
    pool_table                     1
    microwave                      1
    school_bus                     1
    grey_fox                       1
    water_bottle                   1
    crane                          1
    cheetah                        1
    pillow                         1
    platypus                       1
    convertible                    1
    zebra                          1
    traffic_light                  1
    book_jacket                    1
    clog                           1
    leaf_beetle                    1
    tailed_frog                    1
    pitcher                        1
    Name: p1, Length: 378, dtype: int64




```python
# Looking for abnormal names
twitter_archive.name.value_counts()
```




    None         745
    a             55
    Charlie       12
    Oliver        11
    Cooper        11
    Lucy          11
    Penny         10
    Tucker        10
    Lola          10
    Bo             9
    Winston        9
    Sadie          8
    the            8
    an             7
    Toby           7
    Daisy          7
    Buddy          7
    Bailey         7
    Oscar          6
    Scout          6
    Koda           6
    Milo           6
    Leo            6
    Dave           6
    Jax            6
    Bella          6
    Stanley        6
    Rusty          6
    Jack           6
    George         5
                ... 
    Doobert        1
    Grey           1
    Dixie          1
    Sully          1
    Carper         1
    Asher          1
    Oddie          1
    Charl          1
    Creg           1
    Ashleigh       1
    Lenox          1
    Ralphy         1
    Harlso         1
    Snickers       1
    Bookstore      1
    Fynn           1
    Billl          1
    Skittles       1
    Lassie         1
    Newt           1
    Mitch          1
    Livvie         1
    Eazy           1
    Al             1
    Keet           1
    Arya           1
    Jockson        1
    Theo           1
    Goose          1
    Grizzwald      1
    Name: name, Length: 957, dtype: int64



**Several suspicious names, such as 'a' and 'an'.**


```python
# Increasing the column diplay width to better visually inspect the 'text column'
pd.set_option('display.max_colwidth', -1)
```


```python
# Checking rows where name was equal to 'a'
twitter_archive.query('name == "a"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>56</th>
      <td>881536004380872706</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-02 15:32:16 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a pupper approaching maximum borkdrive. Zooming at never before seen speeds. 14/10 paw-inspiring af \n(IG: puffie_the_chow) https://t.co/ghXBIIeQZF</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/881536004380872706/video/1</td>
      <td>14</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>649</th>
      <td>792913359805018113</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-10-31 02:17:31 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a perfect example of someone who has their priorities in order. 13/10 for both owner and Forrest https://t.co/LRyMrU7Wfq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/792913359805018113/photo/1,https://twitter.com/dog_rates/status/792913359805018113/photo/1,https://twitter.com/dog_rates/status/792913359805018113/photo/1,https://twitter.com/dog_rates/status/792913359805018113/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>801</th>
      <td>772581559778025472</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-09-04 23:46:12 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Guys this is getting so out of hand. We only rate dogs. This is a Galapagos Speed Panda. Pls only send dogs... 10/10 https://t.co/8lpAGaZRFn</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/772581559778025472/photo/1,https://twitter.com/dog_rates/status/772581559778025472/photo/1,https://twitter.com/dog_rates/status/772581559778025472/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1002</th>
      <td>747885874273214464</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-28 20:14:22 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a mighty rare blue-tailed hammer sherk. Human almost lost a limb trying to take these. Be careful guys. 8/10 https://t.co/TGenMeXreW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/747885874273214464/photo/1,https://twitter.com/dog_rates/status/747885874273214464/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1004</th>
      <td>747816857231626240</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-28 15:40:07 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Viewer discretion is advised. This is a terrible attack in progress. Not even in water (tragic af). 4/10 bad sherk https://t.co/L3U0j14N5R</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/747816857231626240/photo/1</td>
      <td>4</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1017</th>
      <td>746872823977771008</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-26 01:08:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a carrot. We only rate dogs. Please only send in dogs. You all really should know this by now ...11/10 https://t.co/9e48aPrBm2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/746872823977771008/photo/1,https://twitter.com/dog_rates/status/746872823977771008/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1049</th>
      <td>743222593470234624</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-15 23:24:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a very rare Great Alaskan Bush Pupper. Hard to stumble upon without spooking. 12/10 would pet passionately https://t.co/xOBKCdpzaa</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/743222593470234624/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1193</th>
      <td>717537687239008257</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-06 02:21:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>People please. This is a Deadly Mediterranean Plop T-Rex. We only rate dogs. Only send in dogs. Thanks you... 11/10 https://t.co/2ATDsgHD4n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/717537687239008257/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1207</th>
      <td>715733265223708672</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-01 02:51:22 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a taco. We only rate dogs. Please only send in dogs. Dogs are what we rate. Not tacos. Thank you... 10/10 https://t.co/cxl6xGY8B9</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/715733265223708672/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1340</th>
      <td>704859558691414016</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-03-02 02:43:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a heartbreaking scene of an incredible pupper being laid to rest. 10/10 RIP pupper https://t.co/81mvJ0rGRu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/704859558691414016/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1351</th>
      <td>704054845121142784</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-28 21:25:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a whole flock of puppers.  60/50 I'll take the lot https://t.co/9dpcw6MdWa</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/704054845121142784/photo/1</td>
      <td>60</td>
      <td>50</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1361</th>
      <td>703079050210877440</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-26 04:48:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Butternut Cumberfloof. It's not windy they just look like that. 11/10 back at it again with the red socks https://t.co/hMjzhdUHaW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/703079050210877440/photo/1,https://twitter.com/dog_rates/status/703079050210877440/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1368</th>
      <td>702539513671897089</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-24 17:04:07 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Wild Tuscan Poofwiggle. Careful not to startle. Rare tongue slip. One eye magical. 12/10 would def pet https://t.co/4EnShAQjv6</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/702539513671897089/photo/1,https://twitter.com/dog_rates/status/702539513671897089/photo/1,https://twitter.com/dog_rates/status/702539513671897089/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1382</th>
      <td>700864154249383937</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-20 02:06:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>"Pupper is a present to world. Here is a bow for pupper." 12/10 precious as hell https://t.co/ItSsE92gCW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/700864154249383937/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1499</th>
      <td>692187005137076224</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-27 03:26:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a rare Arctic Wubberfloof. Unamused by the happenings. No longer has the appetites. 12/10 would totally hug https://t.co/krvbacIX0N</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/692187005137076224/photo/1,https://twitter.com/dog_rates/status/692187005137076224/photo/1,https://twitter.com/dog_rates/status/692187005137076224/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1737</th>
      <td>679530280114372609</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-23 05:13:38 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Guys this really needs to stop. We've been over this way too many times. This is a giraffe. We only rate dogs.. 7/10 https://t.co/yavgkHYPOC</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/679530280114372609/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1785</th>
      <td>677644091929329666</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-18 00:18:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a dog swinging. I really enjoyed it so I hope you all do as well. 11/10 https://t.co/Ozo9KHTRND</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/677644091929329666/video/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1853</th>
      <td>675706639471788032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-12 15:59:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Sizzlin Menorah spaniel from Brooklyn named Wylie. Lovable eyes. Chiller as hell. 10/10 and I'm out.. poof https://t.co/7E0AiJXPmI</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675706639471788032/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1854</th>
      <td>675534494439489536</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-12 04:35:48 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Seriously guys?! Only send in dogs. I only rate dogs. This is a baby black bear... 11/10 https://t.co/H7kpabTfLj</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675534494439489536/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1877</th>
      <td>675109292475830276</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-11 00:26:12 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>C'mon guys. We've been over this. We only rate dogs. This is a cow. Please only submit dogs. Thank you...... 9/10 https://t.co/WjcELNEqN2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675109292475830276/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1878</th>
      <td>675047298674663426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-10 20:19:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a fluffy albino Bacardi Columbia mix. Excellent at the tweets. 11/10 would hug gently https://t.co/diboDRUuEI</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675047298674663426/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1923</th>
      <td>674082852460433408</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-08 04:27:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Sagitariot Baklava mix. Loves her new hat. 11/10 radiant pup https://t.co/Bko5kFJYUU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/674082852460433408/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1941</th>
      <td>673715861853720576</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-07 04:09:13 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a heavily opinionated dog. Loves walls. Nobody knows how the hair works. Always ready for a kiss. 4/10 https://t.co/dFiaKZ9cDl</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/673715861853720576/photo/1</td>
      <td>4</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1955</th>
      <td>673636718965334016</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-06 22:54:44 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Lofted Aphrodisiac Terrier named Kip. Big fan of bed n breakfasts. Fits perfectly. 10/10 would pet firmly https://t.co/gKlLpNzIl3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/673636718965334016/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1994</th>
      <td>672604026190569472</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-04 02:31:10 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a baby Rand Paul. Curls for days. 11/10 would cuddle the hell out of https://t.co/xHXNaPAYRe</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/672604026190569472/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2034</th>
      <td>671743150407421952</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-01 17:30:22 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Tuscaloosa Alcatraz named Jacob (Yacōb). Loves to sit in swing. Stellar tongue. 11/10 look at his feet https://t.co/2IslQ8ZSc7</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/671743150407421952/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2066</th>
      <td>671147085991960577</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-30 02:01:49 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Helvetica Listerine named Rufus. This time Rufus will be ready for the UPS guy. He'll never expect it 9/10 https://t.co/34OhVhMkVr</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/671147085991960577/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2116</th>
      <td>670427002554466305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-28 02:20:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Deciduous Trimester mix named Spork. Only 1 ear works. No seat belt. Incredibly reckless. 9/10 still cute https://t.co/CtuJoLHiDo</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/670427002554466305/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2125</th>
      <td>670361874861563904</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-27 22:01:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Rich Mahogany Seltzer named Cherokee. Just got destroyed by a snowball. Isn't very happy about it. 9/10 https://t.co/98ZBi6o4dj</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/670361874861563904/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2128</th>
      <td>670303360680108032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-27 18:09:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Speckled Cauliflower Yosemite named Hemry. He's terrified of intruder dog. Not one bit comfortable. 9/10 https://t.co/yV3Qgjh8iN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/670303360680108032/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2146</th>
      <td>669923323644657664</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-26 16:59:01 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a spotted Lipitor Rumpelstiltskin named Alphred. He can't wait for the Turkey. 10/10 would pet really well https://t.co/6GUGO7azNX</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/669923323644657664/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2153</th>
      <td>669661792646373376</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-25 23:39:47 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a brave dog. Excellent free climber. Trying to get closer to God. Not very loyal though. Doesn't bark. 5/10 https://t.co/ODnILTr4QM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/669661792646373376/photo/1</td>
      <td>5</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2161</th>
      <td>669564461267722241</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-25 17:13:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Coriander Baton Rouge named Alfredo. Loves to cuddle with smaller well-dressed dog. 10/10 would hug lots https://t.co/eCRdwouKCl</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/669564461267722241/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2191</th>
      <td>668955713004314625</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-24 00:54:05 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Slovakian Helter Skelter Feta named Leroi. Likes to skip on roofs. Good traction. Much balance. 10/10 wow! https://t.co/Dmy2mY2Qj5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668955713004314625/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2198</th>
      <td>668815180734689280</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-23 15:35:39 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a wild Toblerone from Papua New Guinea. Mouth always open. Addicted to hay. Acts blind. 7/10 handsome dog https://t.co/IGmVbz07tZ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668815180734689280/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2211</th>
      <td>668614819948453888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-23 02:19:29 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a horned dog. Much grace. Can jump over moons (dam!). Paws not soft. Bad at barking. 7/10 can still pet tho https://t.co/2Su7gmsnZm</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668614819948453888/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2218</th>
      <td>668507509523615744</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-22 19:13:05 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Birmingham Quagmire named Chuk. Loves to relax and watch the game while sippin on that iced mocha. 10/10 https://t.co/HvNg9JWxFt</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668507509523615744/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2222</th>
      <td>668466899341221888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-22 16:31:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a mother dog caring for her pups. Snazzy red mohawk. Doesn't wag tail. Pups look confused. Overall 4/10 https://t.co/YOHe6lf09m</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668466899341221888/photo/1</td>
      <td>4</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2235</th>
      <td>668171859951755264</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-21 20:59:20 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Trans Siberian Kellogg named Alfonso. Huge ass eyeballs. Actually Dobby from Harry Potter. 7/10 https://t.co/XpseHBlAAb</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668171859951755264/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2249</th>
      <td>667861340749471744</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-21 00:25:26 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Shotokon Macadamia mix named Cheryl. Sophisticated af. Looks like a disappointed librarian. Shh (lol) 9/10 https://t.co/J4GnJ5Swba</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667861340749471744/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2255</th>
      <td>667773195014021121</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-20 18:35:10 +0000</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Twitter Web Client&lt;/a&gt;</td>
      <td>This is a rare Hungarian Pinot named Jessiga. She is either mid-stroke or got stuck in the washing machine. 8/10 https://t.co/ZU0i0KJyqD</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667773195014021121/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2264</th>
      <td>667538891197542400</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-20 03:04:08 +0000</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Twitter Web Client&lt;/a&gt;</td>
      <td>This is a southwest Coriander named Klint. Hat looks expensive. Still on house arrest :(\n9/10 https://t.co/IQTOMqDUIe</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667538891197542400/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2273</th>
      <td>667470559035432960</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-19 22:32:36 +0000</td>
      <td>&lt;a href="http://twitter.com" rel="nofollow"&gt;Twitter Web Client&lt;/a&gt;</td>
      <td>This is a northern Wahoo named Kohl. He runs this town. Chases tumbleweeds. Draws gun wicked fast. 11/10 legendary https://t.co/J4vn2rOYFk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667470559035432960/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2287</th>
      <td>667177989038297088</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-19 03:10:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a Dasani Kingfisher from Maine. His name is Daryl. Daryl doesn't like being swallowed by a panda. 8/10 https://t.co/jpaeu6LNmW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667177989038297088/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2304</th>
      <td>666983947667116034</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 14:18:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a curly Ticonderoga named Pepe. No feet. Loves to jet ski. 11/10 would hug until forever https://t.co/cyDfaK8NBc</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666983947667116034/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2311</th>
      <td>666781792255496192</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 00:55:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a purebred Bacardi named Octaviath. Can shoot spaghetti out of mouth. 10/10 https://t.co/uEvsGLOFHa</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666781792255496192/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2314</th>
      <td>666701168228331520</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 19:35:19 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a golden Buckminsterfullerene named Johm. Drives trucks. Lumberjack (?). Enjoys wall. 8/10 would hug softly https://t.co/uQbZJM2DQB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666701168228331520/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2327</th>
      <td>666407126856765440</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 00:06:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a southern Vesuvius bumblegruff. Can drive a truck (wow). Made friends with 5 other nifty dogs (neat). 7/10 https://t.co/LopTBkKa8h</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666407126856765440/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2334</th>
      <td>666293911632134144</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 16:37:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a funny dog. Weird toes. Won't come down. Loves branch. Refuses to eat his food. Hard to cuddle with. 3/10 https://t.co/IIXis0zta0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666293911632134144/photo/1</td>
      <td>3</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2347</th>
      <td>666057090499244032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:55:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>My oh my. This is a rare blond Canadian terrier on wheels. Only $8.98. Rather docile. 9/10 very rare https://t.co/yWBqbrzy8O</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666057090499244032/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2348</th>
      <td>666055525042405380</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:49:46 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a Siberian heavily armored polar bear mix. Strong owner. 10/10 I would do unspeakable things to pet this dog https://t.co/rdivxLiqEt</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666055525042405380/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2350</th>
      <td>666050758794694657</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:30:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a truly beautiful English Wilson Staff retriever. Has a nice phone. Privileged. 10/10 would trade lives with https://t.co/fvIbQfHjIe</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666050758794694657/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>666044226329800704</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:04:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a purebred Piers Morgan. Loves to Netflix and chill. Always looks like he forgot to unplug the iron. 6/10 https://t.co/DWnyCjf2mx</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666044226329800704/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>666033412701032449</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 23:21:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is a very happy pup. Big fan of well-maintained decks. Just look at that tongue. 9/10 would cuddle af https://t.co/y671yMhoiR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666033412701032449/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>666029285002620928</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 23:05:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is a western brown Mitsubishi terrier. Upset about leaf. Actually 2 dogs here. 7/10 would walk the shit out of https://t.co/r7mOb2m0UI</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666029285002620928/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Checking rows where name was equal to 'an'
twitter_archive.query('name == "an"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>759</th>
      <td>778396591732486144</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-09-21 00:53:04 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @dog_rates: This is an East African Chalupa Seal. We only rate dogs. Please only send in dogs. Thank you... 10/10 https://t.co/iHe6liLwWR</td>
      <td>7.030419e+17</td>
      <td>4.196984e+09</td>
      <td>2016-02-26 02:20:37 +0000</td>
      <td>https://twitter.com/dog_rates/status/703041949650034688/photo/1,https://twitter.com/dog_rates/status/703041949650034688/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1025</th>
      <td>746369468511756288</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-24 15:48:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an Iraqi Speed Kangaroo. It is not a dog. Please only send in dogs. I'm very angry with all of you ...9/10 https://t.co/5qpBTTpgUt</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/746369468511756288/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1362</th>
      <td>703041949650034688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-26 02:20:37 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an East African Chalupa Seal. We only rate dogs. Please only send in dogs. Thank you... 10/10 https://t.co/iHe6liLwWR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/703041949650034688/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2204</th>
      <td>668636665813057536</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-23 03:46:18 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an Irish Rigatoni terrier named Berta. Completely made of rope. No eyes. Quite large. Loves to dance. 10/10 https://t.co/EM5fDykrJg</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/668636665813057536/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2333</th>
      <td>666337882303524864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 19:31:45 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an extremely rare horned Parthenon. Not amused. Wears shoes. Overall very nice. 9/10 would pet aggressively https://t.co/QpRjllzWAL</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666337882303524864/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 16:11:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666287406224695296/photo/1</td>
      <td>1</td>
      <td>2</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2349</th>
      <td>666051853826850816</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:35:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is an odd dog. Hard on the outside but loving on the inside. Petting still fun. Doesn't play catch well. 2/10 https://t.co/v5A4vzSDdc</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666051853826850816/photo/1</td>
      <td>2</td>
      <td>10</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Checking rows where name was equal to 'the'
twitter_archive.query('name == "the"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1527</th>
      <td>690360449368465409</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-22 02:28:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Stop sending in lobsters. This is the final warning. We only rate dogs. Thank you... 9/10 https://t.co/B9ZXXKJYNx</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/690360449368465409/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1603</th>
      <td>685943807276412928</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-09 21:58:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the newly formed pupper a capella group. They're just starting out but I see tons of potential. 8/10 for all https://t.co/wbAcvFoNtn</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/685943807276412928/video/1</td>
      <td>8</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1797</th>
      <td>677269281705472000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-16 23:29:14 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the happiest pupper I've ever seen. 10/10 would trade lives with https://t.co/ep8ATEJwRb</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/677269281705472000/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1815</th>
      <td>676613908052996102</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-15 04:05:01 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the saddest/sweetest/best picture I've been sent. 12/10 😢🐶 https://t.co/vQ2Lw1BLBF</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/676613908052996102/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2037</th>
      <td>671561002136281088</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-01 05:26:34 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the best thing I've ever seen so spread it like wildfire &amp;amp; maybe we'll find the genius who created it. 13/10 https://t.co/q6RsuOVYwU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/671561002136281088/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2212</th>
      <td>668587383441514497</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-23 00:30:28 +0000</td>
      <td>&lt;a href="http://vine.co" rel="nofollow"&gt;Vine - Make a Scene&lt;/a&gt;</td>
      <td>Never forget this vine. You will not stop watching for at least 15 minutes. This is the second coveted.. 13/10 https://t.co/roqIxCvEB3</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://vine.co/v/ea0OwvPTx9l</td>
      <td>13</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2345</th>
      <td>666063827256086533</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:22:45 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is the happiest dog you will ever see. Very committed owner. Nice couch. 10/10 https://t.co/RhUEAloehK</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666063827256086533/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2346</th>
      <td>666058600524156928</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:01:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here is the Rand Paul of retrievers folks! He's probably good at poker. Can drink beer (lol rad). 8/10 good dog https://t.co/pYAJkAe76p</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666058600524156928/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>the</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Checking rows where name was equal to 'this'
twitter_archive.query('name == "this"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1120</th>
      <td>731156023742988288</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-05-13 16:15:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Say hello to this unbelievably well behaved squad of doggos. 204/170 would try to pet all at once https://t.co/yGQI3He3xv</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/731156023742988288/photo/1</td>
      <td>204</td>
      <td>170</td>
      <td>this</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Checking rows where name was equal to 'space'
twitter_archive.query('name == "space"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2030</th>
      <td>671789708968640512</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-01 20:35:22 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is space pup. He's very confused. Tries to moonwalk at one point. Super spiffy uniform. 13/10 I love space pup https://t.co/SfPQ2KeLdq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/671789708968640512/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>space</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Checking rows where name was equal to 'None'
twitter_archive.query('name == "None"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>5</th>
      <td>891087950875897856</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 00:08:17 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a majestic great white breaching off South Africa's coast. Absolutely h*ckin breathtaking. 13/10 (IG: tucker_marlo) #BarkWeek https://t.co/kQ04fDDRmh</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891087950875897856/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>7</th>
      <td>890729181411237888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-28 00:22:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>When you watch your owner call another dog a good boy but then they turn back to you and say you're a great boy. 13/10 https://t.co/v0nONBcwxq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/890729181411237888/photo/1,https://twitter.com/dog_rates/status/890729181411237888/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>12</th>
      <td>889665388333682689</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-25 01:55:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here's a puppo that seems to be on the fence about something haha no but seriously someone help her. 13/10 https://t.co/BxvuXk0UCm</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/889665388333682689/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>puppo</td>
    </tr>
    <tr>
      <th>24</th>
      <td>887343217045368832</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-18 16:08:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>You may not have known you needed to see this today. 13/10 please enjoy (IG: emmylouroo) https://t.co/WZqNqygEyV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887343217045368832/video/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>25</th>
      <td>887101392804085760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-18 00:07:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This... is a Jubilant Antarctic House Bear. We only rate dogs. Please only send dogs. Thank you... 12/10 would suffocate in floof https://t.co/4Ad1jzJSdp</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/887101392804085760/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>30</th>
      <td>886267009285017600</td>
      <td>8.862664e+17</td>
      <td>2.281182e+09</td>
      <td>2017-07-15 16:51:35 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>@NonWhiteHat @MayhewMayhem omg hello tanner you are a scary good boy 12/10 would pet with extreme caution</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>32</th>
      <td>886054160059072513</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-15 02:45:48 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @Athletics: 12/10 #BATP https://t.co/WxwJmvjfxo</td>
      <td>8.860537e+17</td>
      <td>19607400.0</td>
      <td>2017-07-15 02:44:07 +0000</td>
      <td>https://twitter.com/dog_rates/status/886053434075471873,https://twitter.com/dog_rates/status/886053434075471873</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>35</th>
      <td>885518971528720385</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-13 15:19:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>I have a new hero and his name is Howard. 14/10 https://t.co/gzLHboL7Sk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/4bonds2carbon/status/885517367337512960</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>37</th>
      <td>885167619883638784</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-12 16:03:00 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a corgi undercover as a malamute. Pawbably doing important investigative work. Zero control over tongue happenings. 13/10 https://t.co/44ItaMubBf</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/885167619883638784/photo/1,https://twitter.com/dog_rates/status/885167619883638784/photo/1,https://twitter.com/dog_rates/status/885167619883638784/photo/1,https://twitter.com/dog_rates/status/885167619883638784/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>41</th>
      <td>884441805382717440</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-10 15:58:53 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>I present to you, Pup in Hat. Pup in Hat is great for all occasions. Extremely versatile. Compact as h*ck. 14/10 (IG: itselizabethgales) https://t.co/vvBOcC2VdC</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/884441805382717440/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>42</th>
      <td>884247878851493888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-10 03:08:17 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>OMG HE DIDN'T MEAN TO HE WAS JUST TRYING A LITTLE BARKOUR HE'S SUPER SORRY 13/10 WOULD FORGIVE IMMEDIATE https://t.co/uF3pQ8Wubj</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/kaijohnson_19/status/883965650754039809</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>47</th>
      <td>883117836046086144</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-07 00:17:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Please only send dogs. We don't rate mechanics, no matter how h*ckin good. Thank you... 13/10 would sneak a pat https://t.co/Se5fZ9wp5E</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/883117836046086144/photo/1,https://twitter.com/dog_rates/status/883117836046086144/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>55</th>
      <td>881633300179243008</td>
      <td>8.816070e+17</td>
      <td>4.738443e+07</td>
      <td>2017-07-02 21:58:53 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>@roushfenway These are good dogs but 17/10 is an emotional impulse rating. More like 13/10s</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>17</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>59</th>
      <td>880872448815771648</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-30 19:35:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Ugh not again. We only rate dogs. Please don't send in well-dressed  floppy-tongued street penguins. Dogs only please. Thank you... 12/10 https://t.co/WiAMbTkDPf</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/880872448815771648/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>62</th>
      <td>880095782870896641</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-28 16:09:20 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Please don't send in photos without dogs in them. We're not @porch_rates. Insubordinate and churlish. Pretty good porch tho 11/10 https://t.co/HauE8M3Bu4</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/880095782870896641/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>64</th>
      <td>879674319642796034</td>
      <td>8.795538e+17</td>
      <td>3.105441e+09</td>
      <td>2017-06-27 12:14:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>@RealKentMurphy 14/10 confirmed</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>72</th>
      <td>878604707211726852</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-24 13:24:20 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Martha is stunning how h*ckin dare you. 13/10 https://t.co/9uABQXgjwa</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/bbcworld/status/878599868507402241</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>78</th>
      <td>877611172832227328</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-21 19:36:23 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @rachel2195: @dog_rates the boyfriend and his soaking wet pupper h*cking love his new hat 14/10 https://t.co/dJx4Gzc50G</td>
      <td>8.768508e+17</td>
      <td>512804507.0</td>
      <td>2017-06-19 17:14:49 +0000</td>
      <td>https://twitter.com/rachel2195/status/876850772322988033/photo/1,https://twitter.com/rachel2195/status/876850772322988033/photo/1,https://twitter.com/rachel2195/status/876850772322988033/photo/1,https://twitter.com/rachel2195/status/876850772322988033/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>83</th>
      <td>876537666061221889</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-18 20:30:39 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>I can say with the pupmost confidence that the doggos who assisted with this search are heroic as h*ck. 14/10 for all https://t.co/8yoc1CNTsu</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/mpstowerham/status/876162994446753793</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>88</th>
      <td>875097192612077568</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-14 21:06:43 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>You'll get your package when that precious man is done appreciating the pups. 13/10 for everyone https://t.co/PFp4MghzBW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/drboondoc/status/874413398133547008</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>89</th>
      <td>875021211251597312</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-14 16:04:48 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Guys please stop sending pictures without any dogs in th- oh never mind hello excuse me sir. 12/10 stealthy as h*ck https://t.co/brCQoqc8AW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/875021211251597312/photo/1,https://twitter.com/dog_rates/status/875021211251597312/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>93</th>
      <td>874057562936811520</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-12 00:15:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>I can't believe this keeps happening. This, is a birb taking a bath. We only rate dogs. Please only send dogs. Thank you... 12/10 https://t.co/pwY9PQhtP2</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/874057562936811520/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>96</th>
      <td>873580283840344065</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-10 16:39:04 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>We usually don't rate Deck-bound Saskatoon Black Bears, but this one is h*ckin flawless. Sneaky tongue slip too. 13/10 would hug firmly https://t.co/mNuMH9400n</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/873580283840344065/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>99</th>
      <td>872967104147763200</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-09 00:02:31 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here's a very large dog. He has a date later. Politely asked this water person to check if his breath is bad. 12/10 good to go doggo https://t.co/EMYIdoblMR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/872967104147763200/photo/1,https://twitter.com/dog_rates/status/872967104147763200/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>doggo</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>100</th>
      <td>872820683541237760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-08 14:20:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here are my favorite #dogsatpollingstations \nMost voted for a more consistent walking schedule and to increase daily pats tenfold. All 13/10 https://t.co/17FVMl4VZ5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/872820683541237760/photo/1,https://twitter.com/dog_rates/status/872820683541237760/photo/1,https://twitter.com/dog_rates/status/872820683541237760/photo/1,https://twitter.com/dog_rates/status/872820683541237760/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>101</th>
      <td>872668790621863937</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-08 04:17:07 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>RT @loganamnosis: Penelope here is doing me quite a divertir. Well done, @dog_rates! Loving the pupdate. 14/10, je jouerais de nouveau. htt…</td>
      <td>8.726576e+17</td>
      <td>154767397.0</td>
      <td>2017-06-08 03:32:35 +0000</td>
      <td>https://twitter.com/loganamnosis/status/872657584259551233/photo/1</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>103</th>
      <td>872486979161796608</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-07 16:14:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>We. Only. Rate. Dogs. Do not send in other things like this fluffy floor shark clearly ready to attack. Get it together guys... 12/10 https://t.co/BZHiKx3FpQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/872486979161796608/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>110</th>
      <td>871102520638267392</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-03 20:33:19 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Never doubt a doggo 14/10 https://t.co/AbBLh2FZCH</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/animalcog/status/871075758080503809</td>
      <td>14</td>
      <td>10</td>
      <td>None</td>
      <td>doggo</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>112</th>
      <td>870804317367881728</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-06-03 00:48:22 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Real funny guys. Sending in a pic without a dog in it. Hilarious. We'll rate the rug tho because it's giving off a very good vibe. 11/10 https://t.co/GCD1JccCyi</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/870804317367881728/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>113</th>
      <td>870726314365509632</td>
      <td>8.707262e+17</td>
      <td>1.648776e+07</td>
      <td>2017-06-02 19:38:25 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>@ComplicitOwl @ShopWeRateDogs &amp;gt;10/10 is reserved for dogs</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
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
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2294</th>
      <td>667138269671505920</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-19 00:32:12 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Extremely intelligent dog here. Has learned to walk like human. Even has his own dog. Very impressive 10/10 https://t.co/0DvHAMdA4V</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667138269671505920/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2298</th>
      <td>667070482143944705</td>
      <td>6.670655e+17</td>
      <td>4.196984e+09</td>
      <td>2015-11-18 20:02:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>After much debate this dog is being upgraded to 10/10. I repeat 10/10</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2299</th>
      <td>667065535570550784</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 19:43:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a Hufflepuff. Loves vest. Eyes wide af. Flaccid tail. Matches carpet. Always a little blurry. 8/10 https://t.co/7JdgVqDnvR</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667065535570550784/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2301</th>
      <td>667044094246576128</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 18:17:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>12/10 gimme now https://t.co/QZAnwgnOMB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/667044094246576128/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2305</th>
      <td>666837028449972224</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 04:35:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>My goodness. Very rare dog here. Large. Tail dangerous. Kinda fat. Only eats leaves. Doesn't come when called 3/10 https://t.co/xYGdBrMS9h</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666837028449972224/photo/1</td>
      <td>3</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2306</th>
      <td>666835007768551424</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 04:27:09 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>These are Peruvian Feldspars. Their names are Cupit and Prencer. Both resemble Rand Paul. Sick outfits 10/10 &amp;amp; 10/10 https://t.co/ZnEMHBsAs1</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666835007768551424/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2307</th>
      <td>666826780179869698</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 03:54:28 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>12/10 simply brilliant pup https://t.co/V6ZzG45zzG</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666826780179869698/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2310</th>
      <td>666786068205871104</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-18 01:12:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Unfamiliar with this breed. Ears pointy af. Won't let go of seashell. Won't eat kibble. Not very fast. Bad dog 2/10 https://t.co/EIn5kElY1S</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666786068205871104/photo/1</td>
      <td>2</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2316</th>
      <td>666649482315059201</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 16:09:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Cool dog. Enjoys couch. Low monotone bark. Very nice kicks. Pisses milk (must be rare). Can't go down stairs. 4/10 https://t.co/vXMKrJC81s</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666649482315059201/photo/1</td>
      <td>4</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2320</th>
      <td>666437273139982337</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 02:06:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we see a lone northeastern Cumberbatch. Half ladybug. Only builds with bricks. Very confident with body. 7/10 https://t.co/7LtjBS0GPK</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666437273139982337/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2321</th>
      <td>666435652385423360</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 02:00:15 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>"Can you behave? You're ruining my wedding day"\nDOG: idgaf this flashlight tastes good as hell\n\n10/10 https://t.co/GlFZPzqcEU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666435652385423360/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2322</th>
      <td>666430724426358785</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 01:40:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Oh boy what a pup! Sunglasses take this one to the next level. Weirdly folds front legs. Pretty big. 6/10 https://t.co/yECbFrSArM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666430724426358785/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2323</th>
      <td>666428276349472768</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 01:30:57 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have an Austrian Pulitzer. Collectors edition. Levitates (?). 7/10 would garden with https://t.co/NMQq6HIglK</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666428276349472768/photo/1</td>
      <td>7</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2324</th>
      <td>666421158376562688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-17 01:02:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>*internally screaming* 12/10 https://t.co/YMcrXC2Y6R</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666421158376562688/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2328</th>
      <td>666396247373291520</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 23:23:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Oh goodness. A super rare northeast Qdoba kangaroo mix. Massive feet. No pouch (disappointing). Seems alert. 9/10 https://t.co/Dc7b0E8qFE</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666396247373291520/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2329</th>
      <td>666373753744588802</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 21:54:18 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Those are sunglasses and a jean jacket. 11/10 dog cool af https://t.co/uHXrPkUEyl</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666373753744588802/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2330</th>
      <td>666362758909284353</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 21:10:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Unique dog here. Very small. Lives in container of Frosted Flakes (?). Short legs. Must be rare 6/10 would still pet https://t.co/XMD9CwjEnM</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666362758909284353/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2331</th>
      <td>666353288456101888</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 20:32:58 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a mixed Asiago from the Galápagos Islands. Only one ear working. Big fan of marijuana carpet. 8/10 https://t.co/tltQ5w9aUO</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666353288456101888/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2332</th>
      <td>666345417576210432</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 20:01:42 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Look at this jokester thinking seat belt laws don't apply to him. Great tongue tho 10/10 https://t.co/VFKG1vxGjB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666345417576210432/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2336</th>
      <td>666273097616637952</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 15:14:19 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Can take selfies 11/10 https://t.co/ws2AMaNwPW</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666273097616637952/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2337</th>
      <td>666268910803644416</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 14:57:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Very concerned about fellow dog trapped in computer. 10/10 https://t.co/0yxApIikpk</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666268910803644416/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2338</th>
      <td>666104133288665088</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 04:02:55 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Not familiar with this breed. No tail (weird). Only 2 legs. Doesn't bark. Surprisingly quick. Shits eggs. 1/10 https://t.co/Asgdc6kuLX</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666104133288665088/photo/1</td>
      <td>1</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2339</th>
      <td>666102155909144576</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:55:04 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Oh my. Here you are seeing an Adobe Setter giving birth to twins!!! The world is an amazing place. 11/10 https://t.co/11LvqN4WLq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666102155909144576/photo/1</td>
      <td>11</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2340</th>
      <td>666099513787052032</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:44:34 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Can stand on stump for what seems like a while. Built that birdhouse? Impressive. Made friends with a squirrel. 8/10 https://t.co/Ri4nMTLq5C</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666099513787052032/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2341</th>
      <td>666094000022159362</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 03:22:39 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This appears to be a Mongolian Presbyterian mix. Very tired. Tongue slip confirmed. 9/10 would lie down with https://t.co/mnioXo3IfP</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666094000022159362/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2342</th>
      <td>666082916733198337</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 02:38:37 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a well-established sunblockerspaniel. Lost his other flip-flop. 6/10 not very waterproof https://t.co/3RU6x0vHB7</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666082916733198337/photo/1</td>
      <td>6</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2343</th>
      <td>666073100786774016</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:59:36 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Let's hope this flight isn't Malaysian (lol). What a dog! Almost completely camouflaged. 10/10 I trust this pilot https://t.co/Yk6GHE9tOY</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666073100786774016/photo/1</td>
      <td>10</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2344</th>
      <td>666071193221509120</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 01:52:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a northern speckled Rhododendron. Much sass. Gives 0 fucks. Good tongue. 9/10 would caress sensually https://t.co/ZoL8kq2XFx</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666071193221509120/photo/1</td>
      <td>9</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>666049248165822465</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 00:24:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a 1949 1st generation vulpix. Enjoys sweat tea and Fox News. Cannot be phased. 5/10 https://t.co/4B7cOc1EDq</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666049248165822465/photo/1</td>
      <td>5</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>666020888022790149</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-15 22:32:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>Here we have a Japanese Irish Setter. Lost eye in Vietnam (?). Big fan of relaxing on stair. 8/10 would pet https://t.co/BLDqew2Ijj</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666020888022790149/photo/1</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
<p>745 rows × 17 columns</p>
</div>



<a id='issues'></a>
## Identified Data Issues

#### Quality
##### 'twitter_archive' table
- Table contains some tweets that are just retweets, which are not supposed to be included
- Table contains some tweets that are just replies, which are not supposed to be included
- Table contains some tweets that are not original ratings with images, which are not supposed to be included
- 'tweet_id' isdatatype int instead of string
- 'timestamp' is not datatype datetime
- Created 'dog_stage' column non-null "None" values that should be null, as well as messy entries such as 'doggoNoneNoneNone', 'NoneflooferNoneNone', etc
- There are non-names (such as *a, an, the, this*, etc) in the 'name' column as well as non-null 'None' values.
- Denominator column contains incorrect values

##### 'image_predictions' table
- 'tweet_id' is an int instead of a string

##### 'tweet_json' table
- 'tweet_id' is an int instead of a string

#### Tidiness
- Dog types (puppo, floofer, etc) in 'twitter_archive' table should be just one 'dog_stage' column.
- All 3 tables should be combined into one table.

<a id='clean'></a>
## Data Cleaning


```python
# Creating copies of dataframes for cleaning
twitter_archive_clean = twitter_archive.copy()
image_predictions_clean = image_predictions.copy()
tweet_json_clean = tweet_json.copy()
```

### Tidiness

#### 1) Dog types (puppo, floofer, etc) in 'twitter_archive' table should be just one 'dog_stage' column.

##### Define
Combine the *doggo*, *floofer*, *pupper*, *puppo* columns to a *dog_stage* column.

##### Code


```python
# Showing list and position of column names
twitter_archive_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
    tweet_id                      2356 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    doggo                         2356 non-null object
    floofer                       2356 non-null object
    pupper                        2356 non-null object
    puppo                         2356 non-null object
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB
    


```python
# Combining the 4 dog stage columns into one desciptive column, as well as dropping the no longer needed columns
twitter_archive_clean['dog_stage'] = twitter_archive_clean.iloc[:,13:17].apply(lambda x: ''.join(x), axis=1)
twitter_archive_clean.drop(['doggo', 'floofer', 'pupper', 'puppo'], axis=1, inplace=True)
```

##### Test


```python
twitter_archive_clean.head()
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>dog_stage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Phineas. He's a mystical boy. Only ever appears in the hole of a donut. 13/10 https://t.co/MgUWQ76dJU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643555336193/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>NoneNoneNoneNone</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Tilly. She's just checking pup on you. Hopes you're doing ok. If not, she's available for pats, snugs, boops, the whole bit. 13/10 https://t.co/0Xxu71qeIV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421306343426/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>NoneNoneNoneNone</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Archie. He is a rare Norwegian Pouncing Corgo. Lives in the tall grass. You never know when one may strike. 12/10 https://t.co/wUnZnhtVJB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181378084864/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>NoneNoneNoneNone</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Darla. She commenced a snooze mid meal. 13/10 happens to the best of us https://t.co/tD36da7qLQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557279858688/photo/1</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>NoneNoneNoneNone</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Franklin. He would like you to stop calling him "cute." He is a very fierce shark and should be respected as such. 12/10 #BarkWeek https://t.co/AtUZn91f7f</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558926688256/photo/1,https://twitter.com/dog_rates/status/891327558926688256/photo/1</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>NoneNoneNoneNone</td>
    </tr>
  </tbody>
</table>
</div>



#### 2) All 3 tables should be combined into one table.

##### Define
First change *id* in the tweet_json table to *tweet_id*, then merge the three tables together, joining on *tweet_id*.

##### Code


```python
# Renaming the 'id' column
tweet_json_clean.rename(columns={'id': 'tweet_id'}, inplace=True)
tweet_json_clean.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2335 entries, 0 to 2334
    Data columns (total 3 columns):
    tweet_id          2335 non-null int64
    favorite_count    2335 non-null int64
    retweet_count     2335 non-null int64
    dtypes: int64(3)
    memory usage: 54.8 KB
    


```python
# Merging all 3 tables
twitter_archive_master = pd.merge(
    pd.merge(twitter_archive_clean, image_predictions_clean,
             on='tweet_id', how='left')
                                  , tweet_json_clean, on='tweet_id', how='left')
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2356 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      2356 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2356 non-null object
    source                        2356 non-null object
    text                          2356 non-null object
    retweeted_status_id           181 non-null float64
    retweeted_status_user_id      181 non-null float64
    retweeted_status_timestamp    181 non-null object
    expanded_urls                 2297 non-null object
    rating_numerator              2356 non-null int64
    rating_denominator            2356 non-null int64
    name                          2356 non-null object
    dog_stage                     2356 non-null object
    jpg_url                       2075 non-null object
    img_num                       2075 non-null float64
    p1                            2075 non-null object
    p1_conf                       2075 non-null float64
    p1_dog                        2075 non-null object
    p2                            2075 non-null object
    p2_conf                       2075 non-null float64
    p2_dog                        2075 non-null object
    p3                            2075 non-null object
    p3_conf                       2075 non-null float64
    p3_dog                        2075 non-null object
    favorite_count                2335 non-null float64
    retweet_count                 2335 non-null float64
    dtypes: float64(10), int64(3), object(14)
    memory usage: 515.4+ KB
    

### Quality

#### 1) Table contains some tweets that are just retweets, which are not supposed to be included

##### Define
Drop rows where the 'retweeted_status_id' column is non-null. Per Twitter's tweet objects developer page, non-null 'retweeted_status' values indicate a retweet.

##### Code


```python
# Dropping rows where 'retweeted_status_id' is non-null
twitter_archive_master = twitter_archive_master[twitter_archive_master.retweeted_status_id.isnull()]
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2175 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      2175 non-null int64
    in_reply_to_status_id         78 non-null float64
    in_reply_to_user_id           78 non-null float64
    timestamp                     2175 non-null object
    source                        2175 non-null object
    text                          2175 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 2117 non-null object
    rating_numerator              2175 non-null int64
    rating_denominator            2175 non-null int64
    name                          2175 non-null object
    dog_stage                     2175 non-null object
    jpg_url                       1994 non-null object
    img_num                       1994 non-null float64
    p1                            1994 non-null object
    p1_conf                       1994 non-null float64
    p1_dog                        1994 non-null object
    p2                            1994 non-null object
    p2_conf                       1994 non-null float64
    p2_dog                        1994 non-null object
    p3                            1994 non-null object
    p3_conf                       1994 non-null float64
    p3_dog                        1994 non-null object
    favorite_count                2169 non-null float64
    retweet_count                 2169 non-null float64
    dtypes: float64(10), int64(3), object(14)
    memory usage: 475.8+ KB
    

#### 2) Table contains some tweets that are just replies, which are not supposed to be included

##### Define
Drop rows where the 'in_reply_to_status_id' column is non-null. Per Twitter's tweet objects developer page, non-null 'in_reply_to_status_id' values indicate a retweet.

##### Code


```python
# Dropping rows where 'in_reply_to_status_id' is non-null
twitter_archive_master = twitter_archive_master[twitter_archive_master.in_reply_to_status_id.isnull()]
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 2097 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      2097 non-null int64
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     2097 non-null object
    source                        2097 non-null object
    text                          2097 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 2094 non-null object
    rating_numerator              2097 non-null int64
    rating_denominator            2097 non-null int64
    name                          2097 non-null object
    dog_stage                     2097 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                2091 non-null float64
    retweet_count                 2091 non-null float64
    dtypes: float64(10), int64(3), object(14)
    memory usage: 458.7+ KB
    

#### 3) Table contains some tweets that are not original ratings with images, which are not supposed to be included

##### Define
Drop rows where the 'jpg_url' column is null. Based on the column name, I'm working under the assumption that null value in this column means that there are no images, which means that this is a tweet that does not contain an original rating, per the given project details.

##### Code


```python
# Dropping rows where 'jpg_url' is null
twitter_archive_master = twitter_archive_master[twitter_archive_master.jpg_url.notnull()]
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      1971 non-null int64
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     1971 non-null object
    source                        1971 non-null object
    text                          1971 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 1971 non-null object
    rating_numerator              1971 non-null int64
    rating_denominator            1971 non-null int64
    name                          1971 non-null object
    dog_stage                     1971 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                1965 non-null float64
    retweet_count                 1965 non-null float64
    dtypes: float64(10), int64(3), object(14)
    memory usage: 431.2+ KB
    

#### 4) 'tweet_id' is datatype int instead of string

##### Define
Change 'tweet_id' datatype from 'int' to 'str'.

*(Ignoring 'retweeted_status_id', 'retweeted_status_user_id', 'in_reply_to_status_id', and 'in_reply_to_user_id' columns in this, as they are not necessary for any analysis later on. However, I am not dropping these columns, in case more WeRateDog tweets are added to this master dataset in the future or if future analysts decide to use these columns.)*

##### Code


```python
# Changing 'tweet_id' to datatype str
twitter_archive_master['tweet_id'] = twitter_archive_master.astype(str)
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      1971 non-null object
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     1971 non-null object
    source                        1971 non-null object
    text                          1971 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 1971 non-null object
    rating_numerator              1971 non-null int64
    rating_denominator            1971 non-null int64
    name                          1971 non-null object
    dog_stage                     1971 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                1965 non-null float64
    retweet_count                 1965 non-null float64
    dtypes: float64(10), int64(2), object(15)
    memory usage: 431.2+ KB
    

#### 5) 'timestamp' is not datatype datetime

##### Define
Change 'timestamp' datatype from 'str' to datetime.

*(Ignoring 'retweeted_status_timestamp' column, as it is not necessary for any analysis later on. However, I am not dropping this column, in case more WeRateDog tweets are added to this master dataset in the future or if future analysts decide to use this column.)*

##### Code


```python
# Changing 'timestamp' to datatype datetime
twitter_archive_master['timestamp'] = pd.to_datetime(twitter_archive_master['timestamp'])
```

##### Test


```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      1971 non-null object
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     1971 non-null datetime64[ns]
    source                        1971 non-null object
    text                          1971 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 1971 non-null object
    rating_numerator              1971 non-null int64
    rating_denominator            1971 non-null int64
    name                          1971 non-null object
    dog_stage                     1971 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                1965 non-null float64
    retweet_count                 1965 non-null float64
    dtypes: datetime64[ns](1), float64(10), int64(2), object(14)
    memory usage: 431.2+ KB
    


```python
twitter_archive_master.head()
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>...</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Phineas. He's a mystical boy. Only ever appears in the hole of a donut. 13/10 https://t.co/MgUWQ76dJU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643555336193/photo/1</td>
      <td>...</td>
      <td>0.097049</td>
      <td>False</td>
      <td>bagel</td>
      <td>0.085851</td>
      <td>False</td>
      <td>banana</td>
      <td>0.076110</td>
      <td>False</td>
      <td>37481.0</td>
      <td>8163.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Tilly. She's just checking pup on you. Hopes you're doing ok. If not, she's available for pats, snugs, boops, the whole bit. 13/10 https://t.co/0Xxu71qeIV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421306343426/photo/1</td>
      <td>...</td>
      <td>0.323581</td>
      <td>True</td>
      <td>Pekinese</td>
      <td>0.090647</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.068957</td>
      <td>True</td>
      <td>32217.0</td>
      <td>6043.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Archie. He is a rare Norwegian Pouncing Corgo. Lives in the tall grass. You never know when one may strike. 12/10 https://t.co/wUnZnhtVJB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181378084864/photo/1</td>
      <td>...</td>
      <td>0.716012</td>
      <td>True</td>
      <td>malamute</td>
      <td>0.078253</td>
      <td>True</td>
      <td>kelpie</td>
      <td>0.031379</td>
      <td>True</td>
      <td>24282.0</td>
      <td>3999.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Darla. She commenced a snooze mid meal. 13/10 happens to the best of us https://t.co/tD36da7qLQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557279858688/photo/1</td>
      <td>...</td>
      <td>0.170278</td>
      <td>False</td>
      <td>Labrador_retriever</td>
      <td>0.168086</td>
      <td>True</td>
      <td>spatula</td>
      <td>0.040836</td>
      <td>False</td>
      <td>40807.0</td>
      <td>8309.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Franklin. He would like you to stop calling him "cute." He is a very fierce shark and should be respected as such. 12/10 #BarkWeek https://t.co/AtUZn91f7f</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558926688256/photo/1,https://twitter.com/dog_rates/status/891327558926688256/photo/1</td>
      <td>...</td>
      <td>0.555712</td>
      <td>True</td>
      <td>English_springer</td>
      <td>0.225770</td>
      <td>True</td>
      <td>German_short-haired_pointer</td>
      <td>0.175219</td>
      <td>True</td>
      <td>39021.0</td>
      <td>9012.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 27 columns</p>
</div>



#### 6) Created 'dog_stage' column non-null "None" values that should be null, as well as messy entries such as 'doggoNoneNoneNone', 'NoneflooferNoneNone', etc

##### Define
Remove the word 'None' from the 'dog_stage' column, as well as turn 'NoneNoneNoneNone' values to null.

##### Code


```python
# Gaining an understanding of what values are in the 'dog_stage' column
twitter_archive_master.dog_stage.value_counts()
```




    NoneNoneNoneNone        1668
    NoneNonepupperNone      201 
    doggoNoneNoneNone       63  
    NoneNoneNonepuppo       22  
    doggoNonepupperNone     8   
    NoneflooferNoneNone     7   
    doggoNoneNonepuppo      1   
    doggoflooferNoneNone    1   
    Name: dog_stage, dtype: int64




```python
# Replacing messy values with clearer, correct dog stage values
twitter_archive_master['dog_stage'] = twitter_archive_master['dog_stage'].replace({'NoneNoneNoneNone': np.nan,
                                                                                   'NoneNonepupperNone': 'pupper',
                                                                                   'doggoNoneNoneNone': 'doggo',
                                                                                   'NoneNoneNonepuppo': 'puppo',
                                                                                   'doggoNonepupperNone': 'Mix (doggo, pupper)',
                                                                                   'NoneflooferNoneNone': 'floofer',
                                                                                   'doggoNoneNonepuppo': 'Mix (doggo, puppo)',
                                                                                   'doggoflooferNoneNone': 'Mix (doggo, floofer)'})
```

##### Test


```python
# Rechecking values in 'dog_stage' column
twitter_archive_master.dog_stage.value_counts()
```




    pupper                  201
    doggo                   63 
    puppo                   22 
    Mix (doggo, pupper)     8  
    floofer                 7  
    Mix (doggo, puppo)      1  
    Mix (doggo, floofer)    1  
    Name: dog_stage, dtype: int64




```python
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      1971 non-null object
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     1971 non-null datetime64[ns]
    source                        1971 non-null object
    text                          1971 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 1971 non-null object
    rating_numerator              1971 non-null int64
    rating_denominator            1971 non-null int64
    name                          1971 non-null object
    dog_stage                     303 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                1965 non-null float64
    retweet_count                 1965 non-null float64
    dtypes: datetime64[ns](1), float64(10), int64(2), object(14)
    memory usage: 431.2+ KB
    

#### 7) There are non-names (such as *a, an, the, this*, etc) in the 'name' column as well as non-null 'None' values.

##### Define
Re-extract the dog names from the text, using Regex rules. The tweets in which the dog names were not included will not be captured by the Regex, and thus return as null. This will get rid of the non-null 'None' values.

##### Code


```python
# Assuming names only contain alphanumeric characters
twitter_archive_master['name'] = twitter_archive_master.text.str.extract('(?:This is |Meet |named )([A-Z]\w*)', expand=True)
```

##### Test


```python
# Checking for non-names such as 'a', 'an', 'the', etc
twitter_archive_master.name.value_counts()
```




    Charlie      10
    Lucy         9 
    Cooper       9 
    Tucker       9 
    Penny        8 
    Oliver       8 
    Winston      7 
    Lola         7 
    Toby         7 
    Daisy        7 
    Bella        6 
    Koda         6 
    Bo           6 
    Sadie        6 
    Stanley      6 
    Jax          6 
    Scout        5 
    Rusty        5 
    Louis        5 
    Bailey       5 
    Oscar        5 
    Chester      5 
    Milo         5 
    Buddy        5 
    Jerry        4 
    Archie       4 
    Brody        4 
    Clark        4 
    Cassie       4 
    Larry        4 
                .. 
    Tedders      1 
    Jett         1 
    Tedrick      1 
    Hammond      1 
    Rueben       1 
    Suki         1 
    Kathmandu    1 
    Jennifur     1 
    Jiminus      1 
    Lambeau      1 
    Oreo         1 
    Andru        1 
    Mya          1 
    Tuco         1 
    Terrance     1 
    Clarkus      1 
    Rolf         1 
    Chef         1 
    Spencer      1 
    Ulysses      1 
    Clybe        1 
    Shawwn       1 
    Tobi         1 
    Griffin      1 
    Remy         1 
    Jessifer     1 
    Livvie       1 
    Covach       1 
    Skye         1 
    Carly        1 
    Name: name, Length: 883, dtype: int64




```python
# Checking for non-null 'None' values in the 'name' column
twitter_archive_master.query('name == "None"')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>...</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
<p>0 rows × 27 columns</p>
</div>




```python
# Checking that the 'name' column now contains null values
twitter_archive_master.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1971 entries, 0 to 2355
    Data columns (total 27 columns):
    tweet_id                      1971 non-null object
    in_reply_to_status_id         0 non-null float64
    in_reply_to_user_id           0 non-null float64
    timestamp                     1971 non-null datetime64[ns]
    source                        1971 non-null object
    text                          1971 non-null object
    retweeted_status_id           0 non-null float64
    retweeted_status_user_id      0 non-null float64
    retweeted_status_timestamp    0 non-null object
    expanded_urls                 1971 non-null object
    rating_numerator              1971 non-null int64
    rating_denominator            1971 non-null int64
    name                          1278 non-null object
    dog_stage                     303 non-null object
    jpg_url                       1971 non-null object
    img_num                       1971 non-null float64
    p1                            1971 non-null object
    p1_conf                       1971 non-null float64
    p1_dog                        1971 non-null object
    p2                            1971 non-null object
    p2_conf                       1971 non-null float64
    p2_dog                        1971 non-null object
    p3                            1971 non-null object
    p3_conf                       1971 non-null float64
    p3_dog                        1971 non-null object
    favorite_count                1965 non-null float64
    retweet_count                 1965 non-null float64
    dtypes: datetime64[ns](1), float64(10), int64(2), object(14)
    memory usage: 431.2+ KB
    

#### 8) Denominator column contains some incorrect values where the first of two ratios was taken instead of the second ratio (the true dog rating). Additionally, tweet_id #810984652412424192 was incorrectly captured as a dog rating tweet (was actually a GoFundMe link upon further inspection), which resulted in an incorrect rating_denominator.

##### Define
For dog rating tweets where there were two ratios, update the rating_numerator and rating_denominator with the correct values. Additionally, drop tweet_id #810984652412424192

##### Code


```python
# Creating separate dataframe to more closely inspect incorrect 'rating_denominator' values
twitter_archive_mask = twitter_archive_master.copy()
```


```python
# Filtering for only the relevant columns and values
twitter_archive_mask = twitter_archive_mask[['tweet_id', 'text', 'rating_numerator', 'rating_denominator']]
twitter_archive_mask.query('rating_denominator != 10')
```




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
      <th>tweet_id</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>433</th>
      <td>820690176645140481</td>
      <td>The floofs have been released I repeat the floofs have been released. 84/70 https://t.co/NIYC820tmd</td>
      <td>84</td>
      <td>70</td>
    </tr>
    <tr>
      <th>516</th>
      <td>810984652412424192</td>
      <td>Meet Sam. She smiles 24/7 &amp;amp; secretly aspires to be a reindeer. \nKeep Sam smiling by clicking and sharing this link:\nhttps://t.co/98tB8y7y7t https://t.co/LouL5vdvxx</td>
      <td>24</td>
      <td>7</td>
    </tr>
    <tr>
      <th>902</th>
      <td>758467244762497024</td>
      <td>Why does this never happen at my front door... 165/150 https://t.co/HmwrdfEfUE</td>
      <td>165</td>
      <td>150</td>
    </tr>
    <tr>
      <th>1068</th>
      <td>740373189193256964</td>
      <td>After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https://t.co/XAVDNDaVgQ</td>
      <td>9</td>
      <td>11</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>731156023742988288</td>
      <td>Say hello to this unbelievably well behaved squad of doggos. 204/170 would try to pet all at once https://t.co/yGQI3He3xv</td>
      <td>204</td>
      <td>170</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>722974582966214656</td>
      <td>Happy 4/20 from the squad! 13/10 for all https://t.co/eV1diwds8a</td>
      <td>4</td>
      <td>20</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>716439118184652801</td>
      <td>This is Bluebert. He just saw that both #FinalFur match ups are split 50/50. Amazed af. 11/10 https://t.co/Kky1DPG4iq</td>
      <td>50</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1228</th>
      <td>713900603437621249</td>
      <td>Happy Saturday here's 9 puppers on a bench. 99/90 good work everybody https://t.co/mpvaVxKmc1</td>
      <td>99</td>
      <td>90</td>
    </tr>
    <tr>
      <th>1254</th>
      <td>710658690886586372</td>
      <td>Here's a brigade of puppers. All look very prepared for whatever happens next. 80/80 https://t.co/0eb7R1Om12</td>
      <td>80</td>
      <td>80</td>
    </tr>
    <tr>
      <th>1274</th>
      <td>709198395643068416</td>
      <td>From left to right:\nCletus, Jerome, Alejandro, Burp, &amp;amp; Titson\nNone know where camera is. 45/50 would hug all at once https://t.co/sedre1ivTK</td>
      <td>45</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1351</th>
      <td>704054845121142784</td>
      <td>Here is a whole flock of puppers.  60/50 I'll take the lot https://t.co/9dpcw6MdWa</td>
      <td>60</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1433</th>
      <td>697463031882764288</td>
      <td>Happy Wednesday here's a bucket of pups. 44/40 would pet all at once https://t.co/HppvrYuamZ</td>
      <td>44</td>
      <td>40</td>
    </tr>
    <tr>
      <th>1635</th>
      <td>684222868335505415</td>
      <td>Someone help the girl is being mugged. Several are distracting her while two steal her shoes. Clever puppers 121/110 https://t.co/1zfnTJLt55</td>
      <td>121</td>
      <td>110</td>
    </tr>
    <tr>
      <th>1662</th>
      <td>682962037429899265</td>
      <td>This is Darrel. He just robbed a 7/11 and is in a high speed police chase. Was just spotted by the helicopter 10/10 https://t.co/7EsP8LmSp5</td>
      <td>7</td>
      <td>11</td>
    </tr>
    <tr>
      <th>1779</th>
      <td>677716515794329600</td>
      <td>IT'S PUPPERGEDDON. Total of 144/120 ...I think https://t.co/ZanVtAtvIq</td>
      <td>144</td>
      <td>120</td>
    </tr>
    <tr>
      <th>1843</th>
      <td>675853064436391936</td>
      <td>Here we have an entire platoon of puppers. Total score: 88/80 would pet all at once https://t.co/y93p6FLvVw</td>
      <td>88</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>1</td>
      <td>2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Updating the 5 identified double ratio instances from the filtered dataframe above
twitter_archive_master.at[1068, 'rating_numerator'] = 14
twitter_archive_master.at[1068, 'rating_denominator'] = 10
twitter_archive_master.at[1165, 'rating_numerator'] = 13
twitter_archive_master.at[1165, 'rating_denominator'] = 10
twitter_archive_master.at[1202, 'rating_numerator'] = 11
twitter_archive_master.at[1202, 'rating_denominator'] = 10
twitter_archive_master.at[1662, 'rating_numerator'] = 10
twitter_archive_master.at[1662, 'rating_denominator'] = 10
twitter_archive_master.at[2335, 'rating_numerator'] = 9
twitter_archive_master.at[2335, 'rating_denominator'] = 10
```


```python
# Dropping tweet_id #810984652412424192 in the master dataframe
twitter_archive_master = twitter_archive_master[twitter_archive_master['tweet_id'] != '810984652412424192']
```

##### Test


```python
# Verifying that the non-dog rating tweet was dropped
twitter_archive_master.query('tweet_id == 810984652412424192')
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>...</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
<p>0 rows × 27 columns</p>
</div>




```python
# Re-creating separate dataframe to more closely inspect incorrect 'rating_denominator' values
twitter_archive_mask = twitter_archive_master.copy()
```


```python
# Re-filtering for only the relevant columns and values
twitter_archive_mask = twitter_archive_mask[['tweet_id', 'text', 'rating_numerator', 'rating_denominator']]
twitter_archive_mask.loc[[1068,1165,1202,1662,2335]]
```




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
      <th>tweet_id</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1068</th>
      <td>740373189193256964</td>
      <td>After so many requests, this is Bretagne. She was the last surviving 9/11 search dog, and our second ever 14/10. RIP https://t.co/XAVDNDaVgQ</td>
      <td>14</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>722974582966214656</td>
      <td>Happy 4/20 from the squad! 13/10 for all https://t.co/eV1diwds8a</td>
      <td>13</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>716439118184652801</td>
      <td>This is Bluebert. He just saw that both #FinalFur match ups are split 50/50. Amazed af. 11/10 https://t.co/Kky1DPG4iq</td>
      <td>11</td>
      <td>10</td>
    </tr>
    <tr>
      <th>1662</th>
      <td>682962037429899265</td>
      <td>This is Darrel. He just robbed a 7/11 and is in a high speed police chase. Was just spotted by the helicopter 10/10 https://t.co/7EsP8LmSp5</td>
      <td>10</td>
      <td>10</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar. 9/10 https://t.co/d9NcXFKwLv</td>
      <td>9</td>
      <td>10</td>
    </tr>
  </tbody>
</table>
</div>



#### Saving cleaned master dataset to CSV file


```python
twitter_archive_master.to_csv('twitter_archive_master.csv', index=False);
```

<a id='analysis'></a>
## Data Analysis


```python
# Loading in cleaned dataset into a pandas dataframe
df = pd.read_csv('twitter_archive_master.csv')
df.head()
```




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
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>...</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
      <th>favorite_count</th>
      <th>retweet_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Phineas. He's a mystical boy. Only ever appears in the hole of a donut. 13/10 https://t.co/MgUWQ76dJU</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643555336193/photo/1</td>
      <td>...</td>
      <td>0.097049</td>
      <td>False</td>
      <td>bagel</td>
      <td>0.085851</td>
      <td>False</td>
      <td>banana</td>
      <td>0.076110</td>
      <td>False</td>
      <td>37481.0</td>
      <td>8163.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Tilly. She's just checking pup on you. Hopes you're doing ok. If not, she's available for pats, snugs, boops, the whole bit. 13/10 https://t.co/0Xxu71qeIV</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421306343426/photo/1</td>
      <td>...</td>
      <td>0.323581</td>
      <td>True</td>
      <td>Pekinese</td>
      <td>0.090647</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.068957</td>
      <td>True</td>
      <td>32217.0</td>
      <td>6043.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Archie. He is a rare Norwegian Pouncing Corgo. Lives in the tall grass. You never know when one may strike. 12/10 https://t.co/wUnZnhtVJB</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181378084864/photo/1</td>
      <td>...</td>
      <td>0.716012</td>
      <td>True</td>
      <td>malamute</td>
      <td>0.078253</td>
      <td>True</td>
      <td>kelpie</td>
      <td>0.031379</td>
      <td>True</td>
      <td>24282.0</td>
      <td>3999.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Darla. She commenced a snooze mid meal. 13/10 happens to the best of us https://t.co/tD36da7qLQ</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557279858688/photo/1</td>
      <td>...</td>
      <td>0.170278</td>
      <td>False</td>
      <td>Labrador_retriever</td>
      <td>0.168086</td>
      <td>True</td>
      <td>spatula</td>
      <td>0.040836</td>
      <td>False</td>
      <td>40807.0</td>
      <td>8309.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24</td>
      <td>&lt;a href="http://twitter.com/download/iphone" rel="nofollow"&gt;Twitter for iPhone&lt;/a&gt;</td>
      <td>This is Franklin. He would like you to stop calling him "cute." He is a very fierce shark and should be respected as such. 12/10 #BarkWeek https://t.co/AtUZn91f7f</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558926688256/photo/1,https://twitter.com/dog_rates/status/891327558926688256/photo/1</td>
      <td>...</td>
      <td>0.555712</td>
      <td>True</td>
      <td>English_springer</td>
      <td>0.225770</td>
      <td>True</td>
      <td>German_short-haired_pointer</td>
      <td>0.175219</td>
      <td>True</td>
      <td>39021.0</td>
      <td>9012.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 27 columns</p>
</div>



### 1) What are the top five most common dog names for WeRateDogs?


```python
df.name.value_counts(ascending=False)[:10]
```




    Charlie    10
    Lucy       9 
    Cooper     9 
    Tucker     9 
    Penny      8 
    Oliver     8 
    Lola       7 
    Winston    7 
    Daisy      7 
    Toby       7 
    Name: name, dtype: int64



**It appears that the top five most common dog names are 'Charlie', 'Tucker', 'Cooper', 'Lucy', and 'Penny'/'Oliver' (tied for 5th).**

### 2) What is the average retweet and favorite counts per original rating tweet?


```python
# First showing the average retweet counts
df.retweet_count.mean()
```




    2618.1354378818737




```python
# Showing the average favorite count
df.favorite_count.mean()
```




    8635.766293279023



**It appears that the average WeRateDogs retweet count is roughly 2638 retweets, and the average favorite count is 8694 favorites.**

Note: These numbers are only referring to the retweet count and favorite count for tweets that contain original ratings. Additionally, this is averaged out over the entire lifespan of the WeRateDogs account, including before it reached it currently high levels of popularity.

### 3) What dog stages are most common?


```python
# Displaying a count of the various dog stages
df.dog_stage.value_counts()
```




    pupper                  201
    doggo                   63 
    puppo                   22 
    Mix (doggo, pupper)     8  
    floofer                 7  
    Mix (doggo, puppo)      1  
    Mix (doggo, floofer)    1  
    Name: dog_stage, dtype: int64




```python
# Generating a plot of the above data
dog_stage_list = df.dog_stage.value_counts()
dog_stage_list.sort_values().plot(kind='bar', colormap="viridis", figsize=(12,6))
plt.title('WeRateDog Dog Stages', fontsize=15)
plt.xlabel('Dog Stages', fontsize=13)
plt.xticks(rotation = 50)
plt.ylabel('Number of dogs', fontsize=13);
```


![png](output_135_0.png)



```python
# Generating total number of dogs that had their dog stage included in the tweet
has_dog_stage = df.dog_stage.value_counts().sum()
has_dog_stage
```




    303




```python
# Generating total number of tweets
total_tweet_num = df.tweet_id.count()
total_tweet_num
```




    1970




```python
# Generating ratio of tweets that included a dog stage versus those that did not
print(has_dog_stage / total_tweet_num)
```

    0.15380710659898478
    

**It appears that by a fairly large margin, dogs in the pupper stage (small and young) are overwhelmingly shown with 201 instances, compared to the next closest dog stage of doggo (big and older) at 63.**

> **Note:** Only around 15.3% of WeRateDog original rating tweets included a dog stage.

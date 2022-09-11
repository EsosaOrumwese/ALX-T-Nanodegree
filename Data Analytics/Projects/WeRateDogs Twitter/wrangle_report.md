# Data Wrangling Project - WeRateDogs Twitter Archive

## Introduction
WeRateDogs is a Twitter account that rates people's dogs with a humorous comment about the dog. These ratings almost always have a denominator of 10. The numerators, though? Almost always greater than 10. 11/10, 12/10, 13/10, etc. Why? Because "they're good dogs Brent." WeRateDogs has over 4 million followers and has received international media coverage.

WeRateDogs downloaded their Twitter archive and sent it to Udacity via email exclusively for you to use in this project. This archive contains basic tweet data (tweet ID, timestamp, text, etc.) for all 5000+ of their tweets as they stood on August 1, 2017.

![WeRateDogs Twitter](https://video.udacity-data.com/topher/2017/October/59dd378f_dog-rates-social/dog-rates-social.jpg)

### Project Steps Overview
With the main focus of this project being on data wrangling, it will be divided into the following steps:
1. Gathering data
2. Assessing data
3. Cleaning data
4. Storing data
5. Analyzing and visualizing data
6. Reporting
      - My data wrangling efforts
      - My data analyses and visualizations.

### Aim
The goal is to create interesting and trustworthy analyses and visualizations. The Twitter archive is great, but it only contains very basic tweet information. Additional gathering, then assessing and cleaning is required for "Wow!"-worthy analyses and visualizations.

### The Data
In this project, I will work on the following three datasets.

#### **Enhanced Twitter Archive**
This data was provided by the Udacity team. The WeRateDogs Twitter archive contains basic tweet data for all 5000+ of their tweets, but not everything. One column the archive does contain though: each tweet's text, which was used to extract rating, dog name, and dog "stage" (i.e. doggo, floofer, pupper, and puppo) to make this Twitter archive "enhanced." Of the 5000+ tweets, they filtered for tweets with ratings only (there are 2356).
![image.png](https://video.udacity-data.com/topher/2017/October/59dd4791_screenshot-2017-10-10-18.19.36/screenshot-2017-10-10-18.19.36.png)

They extracted this data programmatically, but they didn't do a very good job. The ratings probably aren't all correct. Same goes for the dog names and probably dog stages (see below for more information on these) too. I'll need to assess and clean these columns if I want to use them for analysis and visualization.

The Dogtionary explains the various stages of dog: doggo, pupper, puppo and floof(er)
![image.png](https://video.udacity-data.com/topher/2017/October/59e04ceb_dogtionary-combined/dogtionary-combined.png)

#### **Additional Data via the Twitter API**
Back to the basic-ness of Twitter archives: ***retweet count*** and ***favorite count*** are two of the notable column omissions. Fortunately, this additional data can be gathered by anyone from Twitter's API. Since I have access to Twitter's API and the tweet IDs of the tweets in the Enhanced archive, I can gather the needed data for all 5000+ by querying Twitter's API.

#### **Image Predictions File**
Every image in the WeRateDogs Twitter archive was run through a neural network that can classify breeds of dogs. The results: a table full of image predictions (the top three only) alongside each tweet ID, image URL, and the image number that corresponded to the most confident prediction (numbered 1 to 4 since tweets can have up to four images).
![imgpredict](https://video.udacity-data.com/topher/2017/October/59dd4d2c_screenshot-2017-10-10-18.43.41/screenshot-2017-10-10-18.43.41.png)

So for the last row in that table:
* `tweet_id` is the last part of the tweet URL after "status/" → https://twitter.com/dog_rates/status/889531135344209921
* `p1` is the algorithm's #1 prediction for the image in the tweet → **golden retriever**
* `p1_conf` is how confident the algorithm is in its #1 prediction → **95%**
* `p1_dog` is whether or not the #1 prediction is a breed of dog → **TRUE**
* `p2` is the algorithm's second most likely prediction → **Labrador retriever**
* `p2_conf` is how confident the algorithm is in its #2 prediction → **1%**
* `p2_dog` is whether or not the #2 prediction is a breed of dog → **TRUE**
* etc.

## Gathering Data
Let's import necessary libraries for this section
```python:
import pandas as pd
import numpy as np
import tweepy
import requests
import json
import os
from bs4 import BeautifulSoup
```

### Data gotten by hand
The WeRateDogs Twitter archive was given to me by hand and was read into a pandas dataframe.

```python:
twitter_enhanced_df = pd.read_csv('twitter-archive-enhanced.csv')
twitter_enhanced_df.head()
```
![manual-download-gather.png](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/manual-download-gathering.png)

### Data downloaded from Web
We downloaded the tweet image predictions programmatically using the requests library.
```python:
url = "https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv"
response = requests.get(url)

with open("image_predictions.tsv",mode='wb') as file:
      file.write(response.content)
```
```python:
img_predictions_df = pd.read_csv('image_predictions.tsv', sep='\t')
img_predictions_df.head()
```
![download-web-gather.png](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/download-from-web-gathering.png)

### Data from Twitter's API
I used Tweepy to query Twitter's API for the retweet count and favorite count for each tweet. The plan was;
- Create API object
- Create empty list for containing dictionary with keys 'tweet_id' and 'json_string' or 'tweet_data'
- Create an empty list for storing errors
- Iterate through the tweet_id in the twitter_enhanced_df
- Use the try-except statement to catch errors
- Monitor each iterations time.
- Get status of tweet
- Convert json status to string so that it can be stored.
- Store the tweet_id and json_string in a dictionary
- Store dictionary in empty list
- Store errors in list.
- Work on the errors
- Store each json_string in a new line in `tweet_json.txt` file

```python:
# library for timing our code
import timeit
```

You can get your API keys, secrets and tokens by signing up for the [Twitter Developer Account](https://developer.twitter.com)

```python
api_key = "your api key"
api_key_secret = "your api key secret"
access_token = "your access token"
access_token_secret = "your access token secret"
```

Creating the API object that I'll use to gather the additional Twitter data.

```python:
auth = tweepy.OAuthHandler(api_key, api_key_secret)
auth.set_access_token(access_token, access_token_secret)

api = tweepy.API(auth=auth, wait_on_rate_limit=True)
```
Defining relevant functions:
```python:
def tweet_status(tweetID):
      '''
      To get the status of a particular tweetID. Returns a dictioary of tweet ID and
      the json data in string format.
      '''
      # get status of the tweet
      status = api.get_status(tweetID, tweet_mode='extended')
      # get the json data in string
      tweet_data = json.dumps(status._json)
      
      return {'tweet_id': tweetID, 'tweet_data':tweet_data}
```

Note that this code takes a long time to run, so we'll time it to monitor its progress. It took me 30mins to download the additional data

```python
tweets = [] # creating empty list for containing dictionary returned from tweet_status()
errorList = [] # list for containing the failed tweet IDs

i = 1 # counter
for id in twitter_enhanced_df.tweet_id:
      
      print("{}: {}".format(i, id))
      
      start = timeit.timeit() # get start time
      try:
            status = tweet_status(id) # get status for tweet ID
            tweets.append(status) # append return dictionary to tweet_json
      except Exception as e:
            # get the tweet ID and its particular error
            errorList.append({
                              'tweet_id': id,
                              'error': str(e)
                              })
      end = timeit.timeit() # get end time
      print("\truntime:",end - start)
      i += 1
```
![tweepy-data-gather.png](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/tweepy-data-gather.png)

Next, I wrote the tweet data into a file `tweet_json.txt`
```python:
# write only 'tweet_data' of each dictionary in `tweets` to a txt file
c = 0
for status in tweets:
      # creating file and storing data
      if c == 0:
            with open('tweet_json.txt', 'w') as file:
                  file.write(status['tweet_data'])
      else: # appending each json data as a new line
            with open('tweet_json.txt', 'a') as file:
                  file.write("\n"+status['tweet_data'])
      c = 1
```
Then I read the `tweet_json.txt` line by line into a pandas DataFrame with `tweet ID`, `retweet count` and `favorite count`
```python:
tweet_json = [] # creating empty list to store dictionary

with open('tweet_json.txt', 'r') as file:
      # iterating through each line in 'file'
      for line in file.readlines():
            # converting 'line' from string to dictionary type
            data = json.loads(line)
            # appending dictionary contain our values of interest to 'tweet_json'
            tweet_json.append({
                              'tweet_id': data['id'],
                              'retweet_count': data['retweet_count'],
                              'favorite_count': data['favorite_count']
                               })
```
```python:
tweet_json_df = pd.DataFrame(tweet_json)
tweet_json_df
```
![tweet_json.png](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/tweet_json.png)

### Data from Web Scraping
For data cleaning purposes, I needed to scrape a list containing the 120 common dog breeds of [Kaggle web page](https://www.kaggle.com/competitions/dog-breed-identification/data) using BeautifulSoup.

Downloading the web page
```python:
import urllib.request, urllib.error, urllib.parse

url = 'https://www.kaggle.com/competitions/dog-breed-identification/data'

response = urllib.request.urlopen(url)
webContent = response.read().decode('UTF-8')

with open('dog_breed_kaggle.html', 'w') as f:
      f.write(webContent)
```

Parsing html
```python:
with open('dog_breed_kaggle.html') as file:
      # making the soup
      # passing file into BeautifulSoup together with parser, lxml
      soup = BeautifulSoup(file, 'lxml')
      # finding our specific content using its tag and class
      dog_breed_parse = soup.find_all('script', class_="kaggle-component")[1]
```
Getting the required data from the parsed html
```python:
# converting to string and slicing the required text
dog_breed_parse = str(dog_breed_parse)[143:-111]

# converting string to json and using the right keys to get closer to our content
dog_breed_parse = json.loads(dog_breed_parse)['dataIntro']['content']

# splitting by ':' then by '#'
dog_breed_split = dog_breed_parse.split(':')[1].split('##')[0]

# selecting only required data
dog_breed_parse = dog_breed_split[6:-2]

# splitting by '\n' to form a list
dog_breed_list = dog_breed_parse.split()
dog_breed_kaggle = pd.Series(dog_breed_list)
```

Adding the extra dog breeds that are present ing `img_predictions_df`
```python:
dog_breed_kaggle = pd.concat([dog_breed_kaggle, pd.Series(['angora','dalmatian'])], ignore_index=True)
dog_breed_kaggle.sort_values(ignore_index=True, inplace=True)
```
```python:
dog_breed_kaggle
```
![dog-breed.img](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/dog-bred-kaggle.png)

## Assessing Data
After carrying out both visual and programmatic assessments on the datasets, the following quality and tidiness issues were discovered.

### Quality Issues

**`twitter_enhanced_df`**
1. Some rows have a 'None' value for `doggo`, `floofer`, `pupper` and `puppo` columns.
1. `name` column contains incorrect names like 'a', 'the', 'quite', etc.
2. Some rows have a 'None' value for `name`, 
1. Missing data in `name`, `doggo`, `floofer`, `pupper` and `puppo` columns.
3. 181 records are retweets which are basically a repeat of an initial Twitter ID.
3. `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_user_id` and `retweeted_status_timestamp` have NaN values.
4. `timestamp` is an object type.

**`img_predictions_df`**
1. For some Twitter IDs, `p1_dog`, `p2_dog` and `p3_dog` are all False indicating that there's no correct dog breed prediction for those IDs.
2. `p1`, `p2` and `p3` contain some dog breeds in title cases and also contains invalid dog breeds in **`img_predictions_clean`**

**`tweet_json_df`**
1. 29 tweet IDs from `twitter_enhanced_df` don't have a record here because those tweets were either deleted or the accounts went private.


### Tidiness Issues
1. Join **`tweet_json_df`** to **`twitter_enhanced_df`** on `tweet_id`
1. **`img_predictions_df`** has 3 dog breeds. Only one with the %confidence is needed.
1. Drop rows in **`img_predictions_df`** with `percent_conf` < 0.25 because can't be trusted
1. Join **`img_predictions_df`** to **`twitter_enhanced_df`** on `tweet_id`.
2. Drop unneccesary (not useful in our viz) columns in **`twitter_enhanced_df`**: [1,2,6,7,8] 
3. Dog stages are separated into 4 columns (`doggo`, `floofer`, `pupper`, `puppo`) instead of just one [**`twitter_enhanced_df`**]
4. Create one column `rating` that's equal to numerator/denominator

## Cleaning Data
### Issue 1: **Some rows have a 'None' value for `doggo`, `floofer`, `pupper` and `puppo` columns. [`twitter_enhanced_clean`]**
I handled this by ;
- Iterating through the columns of interest
- Replace every 'None' value with `np.nan`

### Issue 2: **`name` column contains incorrect names like 'a', 'the', 'quite', etc.  [`twitter_enhanced_clean`]**
The wrong names included,
```python:
twitter_enhanced_clean.name[twitter_enhanced_clean.name.str.islower()].unique()
```
![incorrectNames.png](/Data%20Analytics/Projects/WeRateDogs%20Twitter/report-img/incorrect%20names.png)
This was handled by;
- Identifying the incorrect names. They were all in lower cases except for 'None'
- Replace these names and 'None' with `np.nan` since the actual names couldn't be gotten.

### **Issue 3: Missing data in `name`, `doggo`, `floofer`, `pupper` and `puppo` columns. [`twitter_enhanced_clean`]**
After inspecting the tweets using the tweet_id, I noticed that these missing data couldn't be gotten from the available data. So, I let it be.

### **Issue 4: 181 records are retweets which are basically a repeat of an initial Twitter ID. [`twitter_enhanced_clean`]**
I handled this by;
- Getting the rows where ` retweeted_status_id` were not null.
- Dropping those rows.

### **Issue 5: timestamp is an object type. [`twitter_enhanced_clean`]**
- I converted the column to datetime type using `pd.to_datatime()`

### **Issue 6: For some Twitter IDs, `p1_dog`, `p2_dog` and `p3_dog` are all False indicating that there's no correct dog breed prediction for those IDs. [`img_predictions_clean`]**
I found that 324 records in `img_predictions_df` fell into this category. The actual pictures either show a different animal or the neural network couldn't detect the dog due to the dogs not properly represented in the photo. I handled this issue by;
- Getting the rows that were `False` for the columns of interest,
- Dropping those rows

### **Issue 7: `p1`, `p2` and `p3` contain some dog breeds in title cases and also contains invalid dog breeds. [`img_predictions_clean`]**
I handled this by;
- Making `p1`, `p2` and `p3` columns lowercase
- Iterating through each column
- If dog_breed isn't in dog_breed_kaggle; replace it and its pconf with NaN values and 0's respectively
- Else pass

### **Issue 8: Join `tweet_json_clean` to `twitter_enhanced_clean` on `tweet_id`**
- With `twitter_enhanced_clean` on the left, merge with 'inner join' to `twitter_enhanced_clean`
- Save as `master_df`

### **Issue 9: `img_predictions_clean` has 3 dog breeds. Only one with the percentage confidence is needed.**
1. Iterate through each row
2. For each row
      * Find the maximum between p1_conf, p2_conf and p3_conf
      * Assign the max value to new column percent_conf
      * Assign the corresponding dog breed to dog_breed

### **Issue 10: Drop rows in img_predictions_df with `percent_conf` < 0.25 because can't be trusted**
By getting only a subset where `percent_conf` > 0.25, I was able to get records that had fairly accurate predictions of the dog's breed.

### **Issue 11: Join `img_predictions_clean` to `master_df` on `tweet_id`.**
With the left dataframe as `master_df` and the right as `image_predictions_clean`, I was able to merge both dataframes using 'left join' on `tweet_id`.

### **Issue 12: Drop unneccesary (not useful in our viz) columns in `master_df`:**
Columns that wouldn't impact the later analysis and visualization were dropped.
```python:
drop_cols = ['in_reply_to_status_id', 'in_reply_to_user_id', 'retweeted_status_id', 
             'retweeted_status_user_id', 'retweeted_status_timestamp', 'p1',
             'p1_conf', 'p1_dog', 'p2', 'p2_conf', 'p2_dog', 'p3', 'p3_conf',
             'p3_dog', 'percent_conf', ]
master_df.drop(drop_cols, axis=1, inplace=True)
```

### **Issue 13: Dog stages are separated into 4 columns (`doggo`, `floofer`, `pupper`, `puppo`) instead of just one dog_stage**
I handled this with the following steps;
- Create a list of columns you want to leave, `id_vars`
- Using `pd.melt()` with identifiers (`id_vars`) and value_name as `dog_stage`,
- Drop created `variable` column.

I noticed that since some of the columns (doggo, floofer, pupper, puppo) contained `NaN` values, this created duplicated values during the melt and using `df.drop_duplicates()` will still leave one NaN value because it only considers the NaNs as duplicates without including rows that have a value. This was handled by;
- Dropping 3 out of the 4 NaN duplicates using `.drop_duplicates()`,
- Find the `tweet_id` for rows that have a value (doggo, floofer, pupper, puppo)
- Create a subset of the `master_df` for all the selected `tweet_id`'s
- From that subset, create another subset where `dog_stage` is NaN.
- From `master_df` drop all indexes in that subset.

```python:
# dropping initial duplicates
master_df.drop_duplicates(inplace=True, ignore_index=True)

# find tweet_id for rows that have a non-null value in `dog_stage`
dog_stage_nonull = master_df[master_df.dog_stage.notna()]
dog_stage_nonan_tweetId = dog_stage_nonull.tweet_id

# create a subset of the master_df for all selected tweet_ids
## this df includes rows where dog_stage has a value and is also Null
master_dog_stage = master_df.query("tweet_id in @dog_stage_nonan_tweetId")

# create another subset from `master_dog_stage` where dog_stage is NaN
## selecting out the rows with NaN values. These are the real duplicates
master_dog_stage_NaN = master_dog_stage[master_dog_stage.dog_stage.isna()]

# from master_df drop all indexes in master_dog_stage_NaN
master_df = master_df.drop(index=master_dog_stage_NaN.index).reset_index(drop=True)
```

### **Issue 14: Create one column ,`rating`, that's equal to numerator/denominator**
This was done by;
- Divide the `rating_numerator` by `rating_denominator` for each column
- Drop `rating_numerator` and `rating_denominator`.

After doing this, I noticed that one of the records had a rating == `inf` because it's `rating_denominator` was 0. After further investigation, I noticed there was a reply tweet and it's initial tweet had the correct rating. As a result, I dropped the row where `rating` == `inf`.

## Storing Data
The cleaned `master_df` was stored in a csv file,
```python:
master_df.to_csv('twitter_archive_master.csv', index=False)
```
and also in a database
```python:
from sqlalchemy import create_engine

# create SQL Alchemy Engine and empty weratedogs database
engine = create_engine('sqlite:///weratedogs.db')

# store master df in a table called twitter_archive_master in database
master_df.to_sql('twitter_archive_master', engine, index=False)
```

## Inferences and Conclusions
This was a reallt fun dataset to wrangle and although, there's still more cleaning that can be done, I believe the insights that I was able to derive are interesting. Let's revisits the questions that were posed earlier on.

### **Q1. What are the popular dog breeds that got rated by WeRateDogs?**
I found out that golden retrievers, Labrador retrievers and pembroke were the top 3 dog breeds that got rated by WeRateDogs with a count of 138, 88 and 85 respectively.

### **Q2. Who are the top 3 good boys/girls?**
For this question, I only dealt with records that had valid dog names. I found out that Atticus, Logan and Sam were the top 3 good dogs with ratings of 177.6, 7.5, 3.4. Even though there were outliers in this dataset, they didn't impact the ratings because those doggo deserved it. Atticus, Logan and Sam all got high ratings because of their costumes but Atticus seemed to be dressed better.

### **Q3. What's everybody's top 10 favorite dogs?**
Also working with records that have valid names and using the `favorite_count` as a measure, Stephan came out as the most favorite dog with 111.1K likes and 51.4K retweets followed by Jamesy with 108.5K likes and 30.1K retweets.

### **Q4. What is the average rating of dogs in respect to their dog stages?**
It was found that puppos were rated, on an average, higher than other dog stages with an average of 1.21. While puppers have the lowest average rating of 1.08.

### **Q5. Who are the best dogs of the different dog stages?**
It was noticed that Sophie the pupper had the highest rating of 2.7 while Grizzwald the floofer and Stuart the puppo had the least rating of 1.3 with Cassie the doggo in the middle with 1.4.
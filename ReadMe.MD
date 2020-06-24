# Summary of Twitter Project

## Introduction

#### For this project. I worked with the Center for Creative Leadership. We talked over requirements,
#### time tables, and the overall project. I am to grab the maximum number of tweets with the tweets.
#### The particular set of tweets will have "search_words = "leader OR managers OR bosses OR directors"".
#### This means that tweepy (the python twitter api package) will scan for tweets with these search words and store them
#### After a good amount of tweets has been stored with these search words, they will be cleaned.
#### After the data is ready, intent analyisis will occur. I ended up using BERT and the bbc nlp data set to help train the model
#### I categorized the dataset using BERTs categorization. Which places tweets into bins such as sports, politics, business...etc.

---

## A look at the code

#### To gather up the Data. I used python and the package tweepy. 

#### I used a bash cript to run my python script twice a day. 
#### I enjoy using cron tabs and found it easiest. 

<p>&nbsp;</p>

```bash
0   */12   *   *   *   mypythonscript
```
## Python Script to gather data

<p>&nbsp;</p>

```python
import os
import tweepy as tw
import pandas as pd
#import texthero as hero
from stop_words import get_stop_words
```
#### Credentials of the twitter api. 

<p>&nbsp;</p>

```python

CONSUMER_KEY = CONSUMER_KEY
CONSUMER_SECRET = CONSUMER_SECRET
ACCESS_TOKEN = ACCESS_TOKEN
ACCESS_TOKEN_SECRET = ACCESS_TOKEN_SECRET

```
<p>&nbsp;</p>

#### Here we list the search words we're looking for. The "q" argument of tweepy takes the "OR" 
#### statement as a comma would in a list.
#### Ontop of that, I set a date only till 2019-01-01. I initially set it to 2020-01-01 but wanted the wider range of tweets.
#### I also didn't want retweets since it's just the same as the original and wouldn't provide any more insight
#### I didn't want to double up on the same tweet. So filtered that out
#### With tweepy I could grab 30,000 per day, I did so in 15k chunks. I did this because sometimes it would have a bad call
#### and stop everything. Since it took a good amount of time, I thought it easier to minimize losses.

<p>&nbsp;</p>

```python

search_words = "leader OR managers OR bosses OR directors"
date_since = "2019-01-01"

new_search = search_words + " -filter:retweets"

tweets = tw.Cursor(api.search, 
                           q=new_search,
                           lang="en",
                           since=date_since).items(15000)
						   
```

#### I pulled 4 objects in particular. Username to account for duplicates. 
#### Location, in case anything can be gleamed from it. Number of followers per that account, and text of the actual tweet.
#### All the objects can be found here https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/tweet-object

<p>&nbsp;</p>
<p>&nbsp;</p>

```python

users_locs = [[tweet.user.screen_name, tweet.user.location,tweet.user.followers_count,tweet.text] for tweet in tweets]

tweet_text = pd.DataFrame(data=users_locs, 
                    columns=['user', "location","followers","text"])
tweet_text.to_csv(r"D:\Documents\Regis\practicum_s1\center_tweets\center_Tweets_15_5.csv") 
```
<p>&nbsp;</p>
<p>&nbsp;</p>

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/output_example.PNG)

<p>&nbsp;</p>

#### I added a replace command to comb through the column to eradicate any not wanted text. 
```python
tweet_text = tweet_text['text'].str.replace(',|\$€¦', '')
```
#### After the clean up and some touching up to make sure the output it great for the correct text editor we were using. This was the output and text file.

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/clean_text.PNG)

#### After the initial cleanup. I wanted to see what words were the most used.

<p>&nbsp;</p>

```python
from collections import Counter
import pandas as pd
import nltk

top_N = 25

df = pd.read_csv(r"D:\Documents\Regis\practicum_s1\center_tweets\center_Tweets_15_all_sheets.csv",
                 usecols=['text'])

df['text'] = df['text'].str.replace(",|\$€¦'.", '')

pd.Series(' '.join(df['text']).lower().split()).value_counts()[:10]


stopwords = nltk.corpus.stopwords.words('english')
# RegEx for stopwords
RE_stopwords = r'\b(?:{})\b'.format('|'.join(stopwords))
# replace '|'-->' ' and drop all stopwords
words = (df.text.str.lower().replace([r'\|', RE_stopwords], [' ', ''], regex=True).str.cat(sep=' ').split()
)

# generate DF out of Counter
rslt = pd.DataFrame(Counter(words).most_common(top_N),
                    columns=['Word', 'Frequency']).set_index('Word')
print(rslt)

# plot
rslt.plot.bar(rot=0, figsize=(25,15), width=0.8)
rslt.

```

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/top_25_words.PNG)

```python
                  Frequency
Word                       
leader                24641
'                      8993
.                      4080
managers               3733
@realdonaldtrump       3498
like                   2908
best                   2760
one                    2661
leader.                2593
people                 2230
bosses                 2048
-                      1998
directors              1905
great                  1903
would                  1629
new                    1535
good                   1467
know                   1466
trump                  1414
think                  1394
get                    1381
us                     1345
need                   1285
world                  1209
want                   1141

```

<p>&nbsp;</p>

#### Two things that stick out. They were talking about donald Trump to a large degree (negative or poisitive, can't tell from this). They were also talking about work.
#### references to directors, and bosses were in there quite a bit. "best" was also high up on the count, so most of the leader references might be positive.

















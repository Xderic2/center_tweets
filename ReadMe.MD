# Summary of Twitter Project

## Introduction

#### For this project. I worked with the Center for Creative Leadership. We talked over requirements,
#### time tables, and the overall project. I am to grab the maximum number of tweets with the tweets
#### The particular set of tweets will have "search_words = "leader OR managers OR bosses OR directors""
#### This means that tweepy (the python twitter api package) will scan for tweets with these search words and store them
#### After a good amount of tweets has been stored with these search words, they will be cleaned
#### After the data is ready, intent analyisis will occur. I ended up using BERT and the bbc nlp data set to help train the model
#### I categorized the dataset using BERTs categorization. Which places tweets into bins such as sports, politics, business...etc

---

## A look at the code

#### To gather up the Data. I used python and the package tweepy. 

#### I used a bash cript to run my python script twice a day
#### I enjoy using cron tabs and found it easiest

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
#### Credentials of the twitter api

<p>&nbsp;</p>

```python

CONSUMER_KEY = CONSUMER_KEY
CONSUMER_SECRET = CONSUMER_SECRET
ACCESS_TOKEN = ACCESS_TOKEN
ACCESS_TOKEN_SECRET = ACCESS_TOKEN_SECRET

```
<p>&nbsp;</p>

#### Here we list the search words we're looking for. The "q" argument of tweepy takes the "OR" 
#### statement as a comma would in a list
#### Ontop of that, I set a date only till 2019-01-01. I initially set it to 2020-01-01 but wanted the wider range of tweets
#### I also didn't want retweets since it's just the same as the original and wouldn't provide any more insight
#### I didn't want to double up on the same tweet. So filtered that out
#### With tweepy I could grab 30,000 per day, I did so in 15k chunks. I did this because sometimes it would have a bad call
#### and stop everything. Since it took a good amount of time, I thought it easier to minimize losses

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

#### I pulled 4 objects in particular. Username to account for duplicates
#### Location, in case anything can be gleamed from it. Number of followers per that account, and text of the actual tweet
#### All the objects can be found here https://developer.twitter.com/en/docs/tweets/data-dictionary/overview/tweet-object

<p>&nbsp;</p>
<p>&nbsp;</p>

```python

users_locs = [[tweet.user.screen_name, tweet.user.location,tweet.user.followers_count,tweet.text] for tweet in tweets]

tweet_text = pd.DataFrame(data=users_locs, 
                    columns=['user', "location","followers","text"])
tweet_text.to_csv(r"D:\Documents\Regis\practicum_s1\center_tweets\center_Tweets_15_5.csv") 
```
#### A sample of one of these pulls can be found under "center_tweets_output" as a csv in this Github or here through google sheets
#### https://docs.google.com/spreadsheets/d/1FKY0vHKDhSIfEyiJ4VNXn-jNItubhpTLkDtSyPocVIg/edit?usp=sharing

<p>&nbsp;</p>
<p>&nbsp;</p>

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/output_example.PNG)

<p>&nbsp;</p>

#### I added a replace command to comb through the column to eradicate any not wanted text
```python
tweet_text = tweet_text['text'].str.replace(',|\$€¦', '')
```
#### After the clean up and some touching up to make sure the output it great for the correct text editor we were using. This was the output and text file

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/clean_text.PNG)

#### After the initial cleanup. I wanted to see what words were the most used

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

#### Two things that stick out. They were talking about donald Trump to a large degree (negative or poisitive, can't tell from this). They were also talking about work
#### references to directors, and bosses were in there quite a bit. "best" was also high up on the count, so most of the leader references might be positive

<p>&nbsp;</p>

#### Next, I was curious how many words were actually used when describing their thoughts
#### I de

```python
count = df['text'].str.split().apply(len).value_counts()
count.index = count.index.astype(str) + ' words:'
count.sort_index(inplace=True)
count
```

#### I noticed that the max seemed suspicious to me. Twitter was capping it out and just adding dots "..." for sentences that kept on going. This was not addressed in the api (at least from what I could fine).

```python
1 words:	104
2 words:	250
3 words:	616
4 words:	983
5 words:	1075
6 words:	1136
7 words:	1120
8 words:	1234
9 words:	1481
10 words:	1584
11 words:	1531
12 words:	1605
13 words:	1739
14 words:	1968
15 words:	2223
16 words:	2795
17 words:	3648
18 words:	4589
19 words:	5744
20 words:	6206
21 words:	5919
22 words:	4825
23 words:	3488
24 words:	2187
25 words:	1091
26 words:	496
27 words:	231
28 words:	86
29 words:	27
30 words:	7
31 words:	5
32 words:	6
33 words:	1
```
#### I then ran The ludwig model. This did a lot of the backend stuff for me
#### below is the colab I used for this since the bbc-text file I used to train the model was importated via gtools in colab
#### I went with a model that could be trained with other text, because I could not physically label my newly formed twitter data
#### I also didn't want to do an unsupervised approach because I wanted control over what that labels were going to be and how the classifier would clasify

#### The main help is this article. Where the writer combined ludwig and BERT for intent analysis
#### https://www.searchenginejournal.com/automated-intent-classification-using-deep-learning-part-2/318691/

https://colab.research.google.com/drive/1Qd1Z83DSNj2UK_ScakFfg3WUe7d4Bp_n?usp=sharing

<p>&nbsp;</p>

![](https://github.com/Xderic2/center_tweets/blob/master/Images_reee/training_epoch.PNG)

#### After that. We can test our model on our actual tweets
#### So I loaded the model I just made, and a csv
#### The csv is a dataset of 200 tweets that I manually labeled. This just came from the set of 15k that I normally imported from my bash/python script daily.

```python
from ludwig.api import LudwigModel

test_df = pd.read_csv(r"D:\Documents\Regis\practicum_s1\center_tweets\center_Tweets_200_5.csv")

model = LudwigModel.load("results/experiment_run_0/model")

predictions = model.predict(test_df)

test_df.join(predictions)[["text", "category_predictions"]]
```

#### I got 72% accuracy on my dataset of 200

#### About half were politically based. Considering the timeline this seems reasonable. The other large chunk was business, which I'm unsure of considering what I thought twitter was used for.

### Issues 
#### - only 200 to test it on (doing something like mechanical turk would have solved this)
#### - The original dataset is not based on tweets. So was hard to get it correct. Could have been solved by the above.
#### I did categorize the other 110,000. But Can't know for certain the accuracy of the bigger group.


















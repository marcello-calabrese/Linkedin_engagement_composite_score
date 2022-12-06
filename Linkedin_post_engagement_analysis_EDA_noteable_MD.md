# Linkedin Posts Engagement Scoring and Analysis of YouHosty

## Introduction

Youhosty is an Italian realities in property management, with a high technological impact, for the provision of quality services, connected to short and medium-term rentals.
They provide a wide range of services, from the management of the property to the management of the guests, from the cleaning to the maintenance of the property, from the marketing to the management of the financial flows. They are a team of professionals who have been working in the sector for years and who have decided to join forces to offer a complete service to the owners of the properties. The other services they provide include:

- **Real Estate Consulting Services:** Thanks to the partnership with leading real estate agencies, renovation and furniture companies, Youhosty is able to provide a range of 360-degree services for your property.
- **Legal Advice Services:** Thanks also to the partnership with a leading international law firm considered among the most innovative and appreciated on the market - they are able to provide support for the most varied problems concerning properties. 
- **Financial Services:** Thanks to the partnership with a leading international financial institution, Youhosty is able to provide financial services through investment professionals mandated by a leading Private Bank, at dedicated conditions to manage the financial flows of the property.

To know more about Youhosty, visit their English website [here](https://www.youhosty.com/en-gb).

## Data Analysis Project Overview

In this notebook we refer to the Linkedin profile posts data of Youhosty to analyse the engagement of the posts. The data collected refer to the period from 01/01/2021 to 31/01/2022.

The posts topics are mixed, some of them focus on the promotion of properties for rent, others focus on sector insights and updates.


Main scope is to find if there are patterns between posts components (words, sentences, etc) and user engagement to create a scoring system to evaluate the engagement of the posts. The scoring system will be based on a tier system (gold, silver, bronze), where each tier will have a different score. The higher the tier, the higher the score. After the implementation of the scoring system we will create a world cloud to see the most used words in the posts to identify the most important topics.
The posts are in Italian language. I will provide a translation of the most important words and sentences.

**Key questions are**: will the flat rental promotion posts be part of the gold and silver tiers or is the insights posts being the winner of the gold and silver medal considering that LinkedIn is a professional social network where besides jobs related posts is mainly used to promote insights and Thought Leadership content?


## About the dataset

The dataset has been collected downloading the Linkedin posts data from Youhosty account. The dataset contains the following columns:

- **Post Date:** Date of the post
- **Update Post**: Text of the post
- **Impressions**: Number of impressions of the post
- **Clicks**: Number of clicks on the post
- **Clicks through rate**: Percentage of clicks on the post
- **Likes**: Number of likes on the post
- **Comments**: Number of comments on the post
- **Shares**: Number of shares on the post
- **Follows**: Number of follows on the post
- **Engagement**: Number of engagement on the post


## Action Plan:
- Cleaning the data (data type adjustments, missing values, special characters, emoji, hashtags)
- Descriptive analysis: likes, shares, comments and engagement count, correlation patterns (if any)
- Scoring system implementation
- Finalize the dataframe to implement composite scoring and wordcloud of most engaging



### 1.Cleaning the data(data type adjustments, missing values, special characters, emoji, hashtags)

In this section we will clean the data, removing the missing values, the special characters, the emoji and the hashtags. We will also adjust the data type of the columns.
But first, let's import the libraries we will need for the analysis.
The libraries we will use are:
- **Pandas:** to manipulate the data
- **Numpy:** to manipulate the data
- **Matplotlib:** to plot the data
- **Seaborn:** to plot the data
- **Neattext:** to clean the text
- **Wordcloud:** to plot the wordcloud


```python
#import main libraries for reading the excel file, cleaning the text

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
!pip install neattext
import neattext as nt
```

    Unable to revert mtime: /opt/conda/fonts


    Collecting neattext
      Downloading neattext-0.1.3-py3-none-any.whl (114 kB)
    [K     |████████████████████████████████| 114 kB 23.9 MB/s 
    [?25hInstalling collected packages: neattext
    Successfully installed neattext-0.1.3



```python
# Reading the file with posts into a dataframe

df = pd.read_excel('youhosty_updates_1_jan_2021_23_jan_2021.xls', sheet_name='Update engagement')
```


```python
df
```




```python
# drop the column with missing values ==> Follows

df.drop(columns='Follows', axis=1, inplace=True)
```


```python
# drop the rows with missing values in the column Update Post and reset the index

df.dropna(axis=0, inplace=True)
```


```python
df.reset_index(drop=True)
```



**Summary of posts Metrics**

- **Total Impressions:** 7852
- **Total Clicks:** 192
- **Click Through Rate Average:** 0.02
- **Total Likes**: 175
- **Total Comments**: 4
- **Total Shares**: 42
- **Average Engagement Rate**: 0.06


```python
# sort the dataframe by Created Date

df.sort_values(by=['Created date'], inplace=True)
```


```python
df.reset_index(drop=True, inplace=True)
```


```python
df
```




```python
# cleaning the text in the Update Post from special characters, emoji, hashtags ecc.. using the neattext library

import neattext.functions as nfx
```


```python
# apply the  cleaning text function to the column Update Post

def clean_column(column):
    # remove user handles
    df['Cleaned_posts'] = df['Update Post'].apply(nfx.remove_userhandles)
    # remove emojis
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_emojis)
    # remove hashtags
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_hashtags)
    # remove special characters
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_special_characters)
    # remove website
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_urls)
    # remove punctuations
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_punctuations)
    # remove short words
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_puncts)
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_stopwords)
    df['Cleaned_posts'] = df['Cleaned_posts'].apply(nfx.remove_shortwords)
    df.drop(columns='Update Post', axis=1, inplace=True)
    return df

clean_column(df['Update Post'])
```




```python
# move a the cleaned posts column before impression column

def move_column(df, col_name, before_col_name):
    col_index = df.columns.get_loc(col_name)
    before_col_index = df.columns.get_loc(before_col_name)
    col_index_before = col_index - 1
    df.insert(before_col_index, col_name, df.pop(col_name))
    return df

move_column(df, 'Cleaned_posts', 'Impressions')
```



### 2. Descriptive analysis: likes, shares, comments and engagement count, correlation patterns (if any)

Checking correlations between numerical variables to identify stronger relationships to identify the key metrics to select for the engagement scoring model


```python
df
```




```python
df
```




```python
# boxplots of key numerical metrics

df.boxplot(['Likes','Clicks','Shares', 'Comments', 'Click through rate (CTR)'], figsize=(15, 10))

```




    <AxesSubplot:>




    
![png](output_23_1.png)
    



```python
# sort by Engagement rate descending to start to graps which kind of posts topics are getting highest engagement rate

df.head(5).sort_values(by='Engagement rate', ascending=False)
```



#### **observations of the heatmapand relplots:** 
- ctr and clicks high corr so we can drop ctr 
- shares and likes show a correlation (not high)
- clicks also shows some correlation with likes
- cliks likes and shares should be the engagement rate to take into considerations
- negative correlation between impressions and engagement rate
- looking at the top 5 of cliks, likes, shares, etc, it seems that posts referring to sector topics, insights and research drive the most of the engagement
- **next steps:** featuring engineering: keep only shares, likes, clicks, impressions and drop ctr and engagement rate to build our own engagement rate ratio as a base to create the engagement scoring

**next steps:** featuring engineering: keep only shares, likes, clicks, impressions and drop ctr, comments and engagement rate.

We are excluding CTR to make our engagment scoring clean and don't skew the data. 

We exclude comments because they present too many ''zeros'' values not adding anything to the efficiency of our own engagement rate.


```python
# new dataframe with only shares, likes, clicks, impressions, comments, creation date
df_clean = df[['Created date', 'Cleaned_posts','Impressions', 'Clicks', 'Likes', 'Shares']]
df_clean
```



### Scoring System Implementation

**Creation of our own Engagement rate**

We are going to create our engagement rate after cleaning the dataset (excluding, comments, CTR Click Through Rate and LinkedIN engagement rate).


```python
df_clean['Own_Eng_rate'] = (df_clean['Clicks'] + df_clean['Likes'] + df_clean['Shares']) / df_clean['Impressions']
```

**Sorting the top 10 posts by our own engagement rate we created**


```python
df_clean.head(10).sort_values(by='Own_Eng_rate', ascending=False)

```



**Initial Observations**

Looking at the top 10 posts by Engagement Rate, we already seeing that posts related to sector insights or updates are at the top.

**The topic with the highest eng rate (8.6%)** for example (in italian: 'Google spinge ripresa turismo..') that in English means: "Google pushes the recovery of tourism in Italy
Also participate with your property in the resumption of short-term rent..." **refers to the google article about the recovery of the short term market and invite property owners to participate to the recovery** (as reminder these posts refer to date range of Jan 2021 to Jan 2022 therefore between the COVID pandemic and its last wave. Italy for who doesn't know was one of the country hard hit by COVID and the rental sector was extremely battered).

**Creation of a composite score for the posts to visualize the most engaging ones**

To create the composite score, we are going to import sklearn libraries Standard Scaler and MinMaxScaler. 


```python
# Import of the normalizer and scalers from sklearn

from sklearn.preprocessing import StandardScaler, MinMaxScaler
#initialize the scaler and the Min Max scaler
scale = StandardScaler()
min_max = MinMaxScaler()
```


```python
# we isolate the numerics columns to calculate the scaling and after generate the composite score.

numerics = ['int64', 'int64', 'int64', 'int64', 'float64']
df_clean.select_dtypes(numerics)
```




```python
numerical_columns = df_clean.select_dtypes(numerics).columns
```


```python
# create a copy to attach the composite score to the original data frame

df_clean_feat_2 = df_clean.copy()
```


```python
# scale the data and we check the first 5 rows

df_clean_feat_2[numerical_columns] = scale.fit_transform(df_clean_feat_2[numerical_columns])
df_clean_feat_2.head()
```




```python
# Create a composite score adding the columns

df_clean_feat_2['Composite_Score'] = df_clean_feat_2.sum(axis=1)

df_clean_feat_2.head(5)
```




```python
# Scale the Composite_Score table with a min max scaler of a scale of 0 to 5

df_clean_feat_2['Composite_Score'] = min_max.fit_transform(df_clean_feat_2[['Composite_Score']]) * 5
```


```python
# Sort the Composite Score from highest to lowest

df_clean_feat_2.sort_values(by='Composite_Score', ascending=False)
```




```python
# Now we create a posts engagement tier system with Gold, silver and bronze tiers using Pandas.qcut

df_clean_feat_2['Eng_tiers'] = pd.qcut(df_clean_feat_2['Composite_Score'], 3, labels=['Bronze', 'Silver', 'Gold'])
```


```python
# After creating the Composite score tiers Gold, Silver, Bronze, we sort them to show Gold on top and we visualize how many posts there are in the different tiers using DEX
df_clean_feat_2.head(25).sort_values(by='Composite_Score', ascending=False)
```




```python
# Now we add composite score and eng tiers score to the original dataframe

df_clean['Composite_Score'] = df_clean_feat_2['Composite_Score']
```


```python
df_clean['Eng_tiers'] = df_clean_feat_2['Eng_tiers']
```


```python
# To create the wordcloud we select the top 25 posts based on the composite score tiers

df_cl_top25 = df_clean.head(25).sort_values(by='Composite_Score', ascending=False)
```


```python
df_cl_top25
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
      <th>Created date</th>
      <th>Cleaned_posts_lowercase</th>
      <th>Impressions</th>
      <th>Clicks</th>
      <th>Likes</th>
      <th>Shares</th>
      <th>Own_Eng_rate</th>
      <th>Composite_Score</th>
      <th>Eng_tiers</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13</th>
      <td>2021-05-29</td>
      <td>flussi prenotazione sensibile ripresa forte ri...</td>
      <td>939</td>
      <td>13</td>
      <td>5</td>
      <td>0</td>
      <td>0.019169</td>
      <td>5.000000</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-02-02</td>
      <td>momento festeggiare nostri risultati traveller...</td>
      <td>591</td>
      <td>19</td>
      <td>9</td>
      <td>3</td>
      <td>0.052453</td>
      <td>3.020095</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2021-04-26</td>
      <td>prenota nostri appartamenti smart working rice...</td>
      <td>380</td>
      <td>7</td>
      <td>11</td>
      <td>1</td>
      <td>0.050000</td>
      <td>1.702128</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2021-03-26</td>
      <td>buone notizie giugno viaggiare turismo condizi...</td>
      <td>344</td>
      <td>8</td>
      <td>6</td>
      <td>1</td>
      <td>0.043605</td>
      <td>1.465721</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-02-05</td>
      <td>dopo stop mino tornera ancora attrattiva leggi...</td>
      <td>292</td>
      <td>8</td>
      <td>9</td>
      <td>2</td>
      <td>0.065068</td>
      <td>1.182033</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>0</th>
      <td>2021-01-29</td>
      <td>google spinge ripresa turismo italia partecipa...</td>
      <td>244</td>
      <td>13</td>
      <td>7</td>
      <td>1</td>
      <td>0.086066</td>
      <td>0.910165</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-02-20</td>
      <td>perderti ultimo articolo</td>
      <td>248</td>
      <td>6</td>
      <td>6</td>
      <td>2</td>
      <td>0.056452</td>
      <td>0.892435</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-02-13</td>
      <td>sanvalentino youhosty regati momenti speciali ...</td>
      <td>248</td>
      <td>5</td>
      <td>5</td>
      <td>0</td>
      <td>0.040323</td>
      <td>0.868794</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>19</th>
      <td>2021-09-16</td>
      <td>cliente stai beneficiando del grande ripresa  ...</td>
      <td>245</td>
      <td>6</td>
      <td>4</td>
      <td>1</td>
      <td>0.044898</td>
      <td>0.856974</td>
      <td>Gold</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2021-03-15</td>
      <td>super campagna youhostygli affitti brevi stann...</td>
      <td>232</td>
      <td>5</td>
      <td>10</td>
      <td>3</td>
      <td>0.077586</td>
      <td>0.821513</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>11</th>
      <td>2021-05-10</td>
      <td>prenota nostri appartamenti smart working rice...</td>
      <td>227</td>
      <td>1</td>
      <td>6</td>
      <td>0</td>
      <td>0.030837</td>
      <td>0.726950</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>21</th>
      <td>2021-11-08</td>
      <td>youhosty ricerca risorsa conosca lingua ingles...</td>
      <td>218</td>
      <td>9</td>
      <td>5</td>
      <td>1</td>
      <td>0.068807</td>
      <td>0.721040</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>18</th>
      <td>2021-07-28</td>
      <td>cliente youhosty beneficiando del grande ripre...</td>
      <td>217</td>
      <td>3</td>
      <td>4</td>
      <td>4</td>
      <td>0.050691</td>
      <td>0.691489</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>22</th>
      <td>2021-11-16</td>
      <td>stiamo assumendo conosci qualcuno potrebbe int...</td>
      <td>210</td>
      <td>11</td>
      <td>4</td>
      <td>0</td>
      <td>0.071429</td>
      <td>0.673759</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2021-03-04</td>
      <td>stress gestione immobiliare apertura casa vaca...</td>
      <td>198</td>
      <td>6</td>
      <td>7</td>
      <td>4</td>
      <td>0.085859</td>
      <td>0.614657</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>20</th>
      <td>2021-11-05</td>
      <td>mesi caldi affitti brevile vacanze estive fatt...</td>
      <td>199</td>
      <td>4</td>
      <td>7</td>
      <td>2</td>
      <td>0.065327</td>
      <td>0.596927</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2021-04-05</td>
      <td>festeggia pasqua vuoi negli appartamenti youho...</td>
      <td>205</td>
      <td>2</td>
      <td>3</td>
      <td>0</td>
      <td>0.024390</td>
      <td>0.585106</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>24</th>
      <td>2021-11-26</td>
      <td>prova comodit arrivare senza pensieri daleropo...</td>
      <td>193</td>
      <td>8</td>
      <td>7</td>
      <td>1</td>
      <td>0.082902</td>
      <td>0.579196</td>
      <td>Silver</td>
    </tr>
    <tr>
      <th>23</th>
      <td>2021-11-17</td>
      <td>grazie leggere recensioni bello riempie orgogl...</td>
      <td>182</td>
      <td>6</td>
      <td>3</td>
      <td>0</td>
      <td>0.049451</td>
      <td>0.472813</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2021-04-15</td>
      <td>prenota nostri appartamenti smart working rice...</td>
      <td>173</td>
      <td>1</td>
      <td>11</td>
      <td>1</td>
      <td>0.075145</td>
      <td>0.443262</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>14</th>
      <td>2021-06-01</td>
      <td>campagna vaccinale green pass innalzamento tem...</td>
      <td>162</td>
      <td>6</td>
      <td>5</td>
      <td>3</td>
      <td>0.086420</td>
      <td>0.384161</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2021-06-12</td>
      <td>forte ripresa perdere loccasione entrata extra...</td>
      <td>155</td>
      <td>6</td>
      <td>5</td>
      <td>2</td>
      <td>0.083871</td>
      <td>0.336879</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>15</th>
      <td>2021-06-03</td>
      <td>visita wwwyouhostycom scoprire servizi erogati...</td>
      <td>157</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>0.025478</td>
      <td>0.295508</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2021-05-22</td>
      <td>campagna vaccinale green pass innalzamento tem...</td>
      <td>143</td>
      <td>2</td>
      <td>4</td>
      <td>2</td>
      <td>0.055944</td>
      <td>0.236407</td>
      <td>Bronze</td>
    </tr>
    <tr>
      <th>17</th>
      <td>2021-07-07</td>
      <td>forte ripresa perdere occasione entrata extra ...</td>
      <td>121</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>0.049587</td>
      <td>0.094563</td>
      <td>Bronze</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Creation of a dataframe with gold posts only to create a Wordcloud and identify the top quartile topics

df_cl_gold = df_cl_top25[df_cl_top25['Eng_tiers'].isin(['Gold'])]
```

### Conclusions: create a wordcloud with posts that are in the gold tier with DEX**


```python
df_cl_gold
```



**Creation of the wordcloud with the wordcloud library**


```python
#install wordcloud library
!pip install wordcloud

#import wordcloud library
from wordcloud import WordCloud
#function to get the wordcloud
def create_wordcloud_from_column(df, col_name, title, figsize=(20, 20)):
    wordcloud = WordCloud(background_color="white", max_words=1000, max_font_size=40, scale=3,
                          random_state=1).generate(str(df[col_name]))
    fig = plt.figure(figsize=figsize)
    plt.imshow(wordcloud, interpolation="bilinear")
    plt.axis("off")
    plt.title(title)
    plt.show()
```

    Collecting wordcloud
      Downloading wordcloud-1.8.2.2-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (458 kB)
    [K     |████████████████████████████████| 458 kB 22.1 MB/s 
    [?25hRequirement already satisfied: matplotlib in /opt/conda/lib/python3.9/site-packages (from wordcloud) (3.4.3)
    Requirement already satisfied: pillow in /opt/conda/lib/python3.9/site-packages (from wordcloud) (8.3.2)
    Requirement already satisfied: numpy>=1.6.1 in /opt/conda/lib/python3.9/site-packages (from wordcloud) (1.23.3)
    Requirement already satisfied: kiwisolver>=1.0.1 in /opt/conda/lib/python3.9/site-packages (from matplotlib->wordcloud) (1.3.2)
    Requirement already satisfied: cycler>=0.10 in /opt/conda/lib/python3.9/site-packages (from matplotlib->wordcloud) (0.10.0)
    Requirement already satisfied: python-dateutil>=2.7 in /opt/conda/lib/python3.9/site-packages (from matplotlib->wordcloud) (2.8.2)
    Requirement already satisfied: pyparsing>=2.2.1 in /opt/conda/lib/python3.9/site-packages (from matplotlib->wordcloud) (3.0.9)
    Requirement already satisfied: six in /opt/conda/lib/python3.9/site-packages (from cycler>=0.10->matplotlib->wordcloud) (1.16.0)
    Installing collected packages: wordcloud
    Successfully installed wordcloud-1.8.2.2



```python
create_wordcloud_from_column(df_cl_gold, 'Cleaned_posts', 'Wordcloud of gold posts tier')
```


    
![png](output_54_0.png)
    


## Final Observations:

- Posts that refer to sector trends drive more engagement (**words: recovery, tourism, flows, results, reopening, increment, vaccins..)**
- Posts with a defined call to action or a strong judgement (read, make a gift) shows higher engagement
- Posts that promote offers and products are less engaging as shown they are in the ''bronze category''
- To improve the social channel engagement, the recommendation is to focus on posts that provide insights on sector trends and updates.

Based on this analysis and the results of the composite score, we can confirm the importance of using LinkedIn to share/promote sector insights, researches, thought leadership content that provide value to the followers.

Instagram and Facebook are more suitable social networks to promote products and services in specific sectors such as the short rental/real estate business.


```python

```

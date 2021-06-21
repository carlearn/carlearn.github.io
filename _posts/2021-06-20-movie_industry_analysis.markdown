---
layout: post
title:      "Movie Industry Analysis"
date:       2021-06-20 23:36:08 -0400
permalink:  movie_industry_analysis
---


Most big companies are creating original video contents recently. Microsoft wants to launch a new movie studio and join the game of making movies. Our team is charged with data analysis on the movie industry. The review period is 2010 - 2019, a decade pre-pandemic.

## Objective

* What types of films are doing best at the box office?
* When is the best time to release films?
* Who are the directors and actors to consider for the new film?
* What are the challenges the new studio will face?

## Methodology

**1. Review Period:** 2010 - 2019

**2. Datasource:**

First, we will use TMDb API to retrieve movie details data including: 'id', 'imdb_id', 'original_title', 'title', 'genres', 'release_date', 'budget', 'revenue', 'popularity', 'vote_average', 'vote_count', and 'runtime'. Next, we will join the IMDb datasets provided to get the crew (including directors and actors) details.

API: Please refer to the blog: [How to Find the Datasets for Movie Analysis](https://carlearn.github.io/how_to_find_the_right_datasets).

Join IMDb datasets:

```

from functools import reduce
dfs = [df_imdb_title_basics, df_imdb_crew, df_imdb_ratings, df_imdb_principals]
df_tconst = reduce(lambda left,right: pd.merge(left,right,on='tconst'), dfs)


df_imdb = pd.merge(df_tconst, df_imdb_name, on='nconst')
df_imdb_crew = df_imdb[(df_imdb.category == 'director') | (df_imdb.category == 'actor')]

```


**3. Clean the Data:**

* check whether there's any missing data and duplicates
* convert dtypes

```

df_tmdb.release_date = pd.to_datetime(df_tmdb.release_date)
df_tmdb.id = df_tmdb.id.astype(int)
df_tmdb.vote_count = df_tmdb.vote_count.astype(int)
df_tmdb.runtime = df_tmdb.runtime.astype(int)
df_tmdb.genres = df_tmdb.genres.apply(lambda x: eval(x))

```

* split the column of 'genres' to get respective movie category

```

def split_genres(data):
    '''split the genres column in the raw data to get the specific category of movie genres.'''
    genres_name = []
    for i in range(0, len(data)):
        genres_name.append(data[i]['name'])
    return genres_name
		
df_tmdb.genres = df_tmdb.genres.apply(split_genres)

genres_list = df_tmdb['genres'].apply(pd.Series).stack()
genres_list.index = genres_list.index.droplevel(-1)
genres_list.name = 'genres'
del df_tmdb['genres']
df_tmdb = df_tmdb.join(genres_list)

```

* convert money values into million: including columns 'revenue' and 'budget'

```

df_tmdb['budget($mil)'] = (df_tmdb.budget.astype(float)/1000000).round(2)
df_tmdb['revenue($mil)'] = (df_tmdb.revenue.astype(float)/1000000).round(2)

```

* create a new column 'profit'

```

df_tmdb['profit($mil)'] = df_tmdb['revenue($mil)'] - df_tmdb['budget($mil)']

```

**4. Exploratory Data Analysis:**

* use groupby function to get the relationship between genres and profits / popularity: take profits by genres as an example

```

s1=df_tmdb.groupby('genres')['profit($mil)'].mean()
s2=df_tmdb.groupby('genres')['profit($mil)'].sum()
df_1 = pd.concat([s1,s2],axis=1)

df_1.columns = ['avg_profit','total_profit']

```

* use groupby function to analyze the seasonality of respective movie genres

```

month_index = ['January', 'February', 'March', 'April', 'May', 'June',
               'July', 'August', 'September', 'October', 'November', 'December']
df_3e = df_tmdb.groupby(['genres','release_month'])['profit($mil)'].mean().unstack()
df_3e = df_3e.fillna(0).astype(float).round(2)
df_3e = df_3e[month_index]
df_3f = df_3e.loc[['Adventure','Animation','Science Fiction']]
df_3g = df_3f.transpose()

```

* use merge to join TMDb and IMDb data frames

```

df_4 = pd.merge(df_tmdb, df_imdb_crew.rename(columns={'tconst': 'imdb_id'}), on='imdb_id', how='left')

```

* use matplotlib and seaborn to visualize the data: please refer to the next section


**5. Summarize the recommendations, challenges and next steps**


## Summary (Data Visualization)

#### What types of films are doing best at the box office?

> Most Profitable Genres of Movies

![](https://raw.githubusercontent.com/carlearn/dsc-mod-1-project-v2-1-online-ds-sp-000/master/images/profit_by_genres.png)


> Most Popular Genres of Movies

![](https://raw.githubusercontent.com/carlearn/dsc-mod-1-project-v2-1-online-ds-sp-000/master/images/popularity_by_genres.png)


#### When is the best time to release films?

> Seasonality of Film Release

![](https://raw.githubusercontent.com/carlearn/dsc-mod-1-project-v2-1-online-ds-sp-000/master/images/seasonality_by_genres.png)


#### Who are the directors and actors to consider for the new film?

> Top 5 Directors

![](https://raw.githubusercontent.com/carlearn/dsc-mod-1-project-v2-1-online-ds-sp-000/master/images/top_5_directors.png)

> Top 5 Actors

![](https://raw.githubusercontent.com/carlearn/dsc-mod-1-project-v2-1-online-ds-sp-000/master/images/top_5_actors.png)


## Recommendation

**1. Genres**

The new studio that Microsoft will launch shall produce the movies in genres of Adventure, Animation and Science Fiction from the profitability and popularity perspective.

**2. Seasonality**

The best months to release these genres of movies should be: June, November/December (summer and holiday seasons)

**3. Best Crew**

The best crew to hire are determined by movie genres.


## Future Work

The analysis is based on the movie data on TMDb and IMDb for the period between 2010 and 2019 (i.e. pre-pandemic). However, the pandemic has changed the movie industry and especially the movie viewing habits of audience. A new studio will face the following challenges:

**1. Data source:**

As an alternative to the theater, people tend to view videos via social media and over-the-top media service (i.e. OTT service, such as Netflix, Hulu, Amazon Prime and etc). Therefore, our data sources should be expanded and cover more channels instead of theater box office numbers alone. 

**2. Movie genres:**

Pandemics may have changed people's appetite to the genres of movies. Parents may spend more time with their kids to watch Animation, Family and History movies, while young people staying at home may prefer Action, Thriller and Horror movies to get excitement. We may reconsider which genre of movies is making more profits or getting more popular. Accordingly, we need to reevaluate the crew (including directors and actors) to hire.

**3. Release time:**

As people tend to watch movies via OTT service, the release time may be more flexible. 

**4. Big IP / Series Movies:**

On the other hand, the profits and top crews are largely determined by big IP movies. As a new studio, we should take the impact of big IPs into consideration. 


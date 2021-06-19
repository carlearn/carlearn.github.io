---
layout: post
title:      "How to Find the Right Datasets"
date:       2021-06-19 12:22:35 +0000
permalink:  how_to_find_the_right_datasets
---


As the saying goes, the first step is always the hardest.

My recent project was to analyze the box office data to help Microsoft understand the movie industry and then open a new movie studio. I first loaded all the datasets, including the Box Office Mojo, IMDb, Rotten Tomatoes, TMDb, and the Numbers. Then I tried to combine all the datasets into a large data frame for further ETL. However, there's a problem with joining datasets from multiple sources, as formats are almost never standardized across the sources.

For example, the movie "Star Wars: The Force Awakens" has different title format in IMDb, TMDb and The Numbers datasets. 

| data_source | title |
| -------- | --------|
| IMDb      | Star Wars: Episode VII - The Force Awakens      |
| TMDb     | Star Wars: The Force Awakens    | 
| The Numbers     | Star Wars Ep. VII:  The Force Awakens    | 


**When coming across such problem as combing datasets from different sources, there are three solutions to find the standized unique key:**

1. Are there many edge cases like this? If there’s aren’t many edge cases, we can manually clean the titles;
1. If there are a bunch of edge cases, another option would be to grab partial titles (maybe just grab the first couple of words from the title) and join on the partial title and the year the film was released;
1. If the above two solutions do not work, we can try APIs to see if there's a unique key that you can join on.


I tried to use partial title and year the film was released, but still missed some movie titles with special symbols. Except some common apostrophes such as comma, colon and dash, some apostrophes in movie titles are not easily to find out. Most of the titles incurred with unmatched problems are those popular series movies with substantial revenues. For example, Fast & Furious.

Therefore, my final solution is to get more complete datasets from TMDb API where I can get movie title, release_year, budget, revenue, popularity and imdb_id. Accordingly, I can use the unique imdb_id to link the datasets provided to get more information of directors and actors.



### **How to get movie information from TMDb API?**


**1. Register for an API key on TMDb website**

[TMDb API Website](https://www.themoviedb.org/documentation/api)


**2. Import package: Json and Request**

****
```
import json
import requests
```
****

**3. Retrieve API key and load it into a variable**

****
```
def get_keys(path):
    '''takes in a path string, loads the json file, and returns the info inside'''
    with open(path) as f:
        return json.load(f)

keys = get_keys("API KEY FILE PATH")

api_key = keys['api_key']
```
****

**4. Get movie list from TMDb API**

****
```
movie_response = {}

for i in range (1,501):
    url = 'https://api.themoviedb.org/3/discover/movie?api_key={}\
    &language=en-US&sort_by=popularity.desc&page={}\
    &primary_release_date.gte=2010-01-01&primary_release_date.lte=2019-12-31'.format(api_key, i)
    movie_response[i] = (requests.get(url).json())
```
****

**5. Create a dataframe to store the movie list**

****
```
tmdb_data1 = pd.DataFrame()

for x in movie_response:
    df = pd.DataFrame.from_dict(movie_response[x]['results'])
    tmdb_data1 = pd.concat([df, tmdb_data1])

tmdb_data1
```
****

**6. Get movie details from TMDb API**

According to TMDb API website, we can get primary movie details including 26 items via submitting the request /movie/{movie_id} . Therefore, we need to iterate each movie in the above movie list to get the movie details. 

****
```
movie_details = {}

for i, element in enumerate(list(map(int, tmdb_data1.id))):
    url = 'https://api.themoviedb.org/3/movie/{}?api_key={}'.format(element, api_key)
    movie_details[i] = (requests.get(url).json())
```
****

**7. Create a new data frame to store all the movie details**

****
```
tmdb_data = pd.DataFrame()

for x in range(0, len(movie_details)):
    df = pd.DataFrame.from_dict(movie_details[x], orient='index')
    df = df.transpose()
    tmdb_data = pd.concat([df, tmdb_data])

tmdb_data
```
****

**8. Select the necessary columns and export the raw data into csv file**

****
`tmdb_data.to_csv('tmdb_data.csv', index=False)`
****


**9. Upon the tmdbdata is cleaned, we can combine IMDb datasets on identical movie ID to get the crew details.** 


 **Finding the right datasets is the first but very important step of data analysis. API is a very useful tool to help you get customized and most up-to-date information for further exploratory data analysis. **

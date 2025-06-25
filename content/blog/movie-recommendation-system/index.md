---
title: "Movie Recommendation System"
date: 2025-06-05
description:
  "A comprehensive movie recommendation system built using the MovieLens dataset
  with popularity-based, content-based, and collaborative filtering techniques."
tags:
  [
    "python",
    "machine-learning",
    "recommendation-system",
    "data-science",
    "streamlit",
  ]
toc: true
---

I recently developed a comprehensive movie recommendation system using the
MovieLens dataset. This project demonstrates various recommendation techniques
including popularity-based filtering, content-based filtering, and collaborative
filtering.

## About the Project

The system helps users discover new movies based on their preferences and
viewing history. It's built with Python and leverages several machine learning
techniques to provide personalized movie recommendations.

{{< video src="demo.mp4" >}}

## Dataset

For this project, I used the
[MovieLens small dataset](https://grouplens.org/datasets/movielens/) which
contains approximately 100,000 ratings from 600+ users on 9,000+ movies. This
smaller dataset is more manageable for version control purposes compared to the
full dataset which has over 33 million ratings.

The dataset includes:

- **movies.csv**: Information about movies (ID, title, genres)
- **ratings.csv**: User ratings for movies (userID, movieID, rating, timestamp)
- **tags.csv**: User-generated tags for movies
- **links.csv**: Links to movie pages on IMDb and TMDb

## Recommendation Approaches

### 1. Popularity-Based Recommendations

This is the simplest approach that recommends movies based on their popularity
(average ratings and number of ratings). These recommendations are the same for
all users regardless of personal preferences.

```python
def get_popular_recommendations(n=10):
    # Calculate weighted rating
    C = df_ratings['rating'].mean()
    m = df_movies_with_ratings['num_ratings'].quantile(0.9)

    qualified = df_movies_with_ratings.copy().loc[df_movies_with_ratings['num_ratings'] >= m]
    qualified['score'] = qualified.apply(weighted_rating, axis=1)

    # Sort movies based on score and return top n
    qualified = qualified.sort_values('score', ascending=False)
    return qualified[['movieId', 'title', 'genres', 'score']].head(n)
```

### 2. Content-Based Filtering

This approach recommends movies similar to ones the user has liked in the past
based on movie attributes. In my implementation, I used movie genres to find
similar movies.

```python
def get_content_based_recommendations(movie_title, n=10):
    # Get the index of the movie in our dataframe
    idx = indices[movie_title]

    # Get the pairwise similarity scores for all movies with that movie
    sim_scores = list(enumerate(cosine_sim[idx]))

    # Sort the movies based on the similarity scores
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)

    # Get the scores of the 10 most similar movies (excluding the input movie)
    sim_scores = sim_scores[1:n+1]

    # Get the movie indices
    movie_indices = [i[0] for i in sim_scores]

    # Return the top n most similar movies
    return df_movies.iloc[movie_indices][['movieId', 'title', 'genres']]
```

### 3. Collaborative Filtering

This approach makes recommendations based on the similarity between users and/or
items. I implemented Matrix Factorization using Singular Value Decomposition
(SVD).

```python
def get_collaborative_filtering_recommendations(user_id, n=10):
    # Get all movies the user hasn't seen
    user_ratings = df_ratings[df_ratings['userId'] == user_id]
    movies_user_has_seen = user_ratings['movieId'].tolist()
    movies_to_predict = df_movies[~df_movies['movieId'].isin(movies_user_has_seen)]

    # Predict ratings for all unseen movies
    predictions = []
    for _, movie in movies_to_predict.iterrows():
        pred = model.predict(user_id, movie['movieId']).est
        predictions.append((movie['movieId'], movie['title'], movie['genres'], pred))

    # Sort by predicted rating
    predictions.sort(key=lambda x: x[3], reverse=True)

    # Return top n recommendations
    return pd.DataFrame(predictions[:n], columns=['movieId', 'title', 'genres', 'predicted_rating'])
```

## Web Application

I built a beautiful Streamlit web application with a Gruvbox Medium theme to
showcase the recommendation system. The app features:

- Interactive movie recommendations
- Data visualizations
- Multiple recommendation approaches
- Filtering and sorting options
- User-friendly interface

<!-- ![Streamlit Web App](streamlit-app.jpg) -->

## Command-Line Interface

For quick recommendations, I also implemented a command-line interface:

```bash
# Get popularity-based recommendations
./recommend.py --popular --num 10

# Get content-based recommendations for a movie
./recommend.py --movie "The Matrix (1999)" --num 10

# Get collaborative filtering recommendations for a user
./recommend.py --user 1 --num 10
```

## Challenges and Learnings

Building this recommendation system taught me several important lessons:

1. **Handling Sparse Data**: Movie ratings are typically very sparse, with most
   users rating only a small fraction of available movies. Learning to handle
   this sparsity was crucial.

2. **Cold Start Problem**: Addressing the challenge of recommending movies to
   new users with little or no history.

3. **Balancing Accuracy and Diversity**: Ensuring recommendations aren't too
   similar while still being relevant.

4. **Scalability**: Making the system efficient enough to handle large datasets
   and provide real-time recommendations.

## Future Improvements

I plan to enhance the system with:

- Additional features for content-based filtering (actors, directors, etc.)
- Hybrid recommendation approaches
- User authentication for the web app
- More detailed evaluation metrics
- Cloud deployment of the Streamlit app

Check out the [GitHub repository](https://github.com/shubhpsd/movie-recosystem)
for the complete source code and more details about this project!

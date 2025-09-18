---
title: "Movie Recommendation Engine: Netflix but worse"
date: 2025-06-05
description:
  "Built a movie recommendation system that actually gets your taste. Three
  different algorithms, one goal: find your next binge-watch obsession."
tags:
  [
    "python",
    "machine-learning",
    "recommendation-system",
    "data-science",
    "streamlit",
    "movielens",
  ]
toc: true
cover:
  src: ./movie-cover.webp
  alt: "People in movie theatre."
---

## What I Built

Ever spend more time scrolling through Netflix than actually watching anything?
Yeah, me too. So I thought, "What if I could build something that actually knows
what I want to watch?"

Enter my movie recommendation engine - three different algorithms working
together to find your next favorite film.

{{< video src="demo.mp4" >}}

## Why I Built This

Look, I'm a data nerd who also happens to love movies. But finding good
recommendations? That's harder than debugging Python at 2 AM.

Most recommendation systems either give you what's popular (boring) or
completely miss the mark on your taste.

I wanted something that actually understands different ways people discover
movies

Sometimes you want what's trending, sometimes you want more of what you already
love, and sometimes you want to discover hidden gems based on people with
similar taste.

## The Dataset Deep Dive

I used the
[MovieLens small dataset](https://grouplens.org/datasets/movielens/) - about
100,000 ratings from 600+ users rating 9,000+ movies. Why the "small" version?
Because this is a concept project, and I wanted it to be responsive, will take
it forward in the future, with the full 33 million+ dataset.

What's in the box:

- **movies.csv**: All the movie info (titles, genres, IDs)
- **ratings.csv**: The good stuff - who rated what and how much they liked it
- **tags.csv**: User-generated tags (surprisingly useful!)
- **links.csv**: Links to IMDb and TMDb pages

## Three Ways to Find Your Next Favorite Movie

### 1. Popular Stuff (For When You Want to Join the Conversation)

Sometimes you just want to watch what everyone's talking about. This approach
finds movies with great ratings AND enough people actually watching them. No
point recommending a 5-star movie if only 3 people have seen it, right?

```python
def get_popular_recommendations(n=10):
    # Calculate weighted rating (IMDB's approach)
    C = df_ratings['rating'].mean()
    m = df_movies_with_ratings['num_ratings'].quantile(0.9)

    # Only consider movies with enough ratings
    qualified = df_movies_with_ratings.copy().loc[df_movies_with_ratings['num_ratings'] >= m]
    qualified['score'] = qualified.apply(weighted_rating, axis=1)

    # Sort and return the crowd favorites
    qualified = qualified.sort_values('score', ascending=False)
    return qualified[['movieId', 'title', 'genres', 'score']].head(n)
```

### 2. More Like This (For When You Know What You Like)

Found a movie you absolutely loved? This finds movies with similar vibes based
on genres, themes, and other attributes. If you're into sci-fi thrillers, it'll
find more sci-fi thrillers.

```python
def get_content_based_recommendations(movie_title, n=10):
    # Find the movie in our database
    idx = indices[movie_title]

    # Calculate similarity with all other movies
    sim_scores = list(enumerate(cosine_sim[idx]))

    # Sort by similarity (excluding the input movie itself)
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    sim_scores = sim_scores[1:n+1]

    # Get the most similar movies
    movie_indices = [i[0] for i in sim_scores]
    return df_movies.iloc[movie_indices][['movieId', 'title', 'genres']]
```

### 3. People Like You Also Enjoyed (The Netflix Special)

This is where the magic happens - finding people with similar taste and seeing
what they loved that you haven't watched yet. Uses matrix factorization (fancy
math) to predict what you'd rate unseen movies.

```python
def get_collaborative_filtering_recommendations(user_id, n=10):
    # Find movies this user hasn't seen
    user_ratings = df_ratings[df_ratings['userId'] == user_id]
    movies_user_has_seen = user_ratings['movieId'].tolist()
    movies_to_predict = df_movies[~df_movies['movieId'].isin(movies_user_has_seen)]

    # Predict ratings for unseen movies
    predictions = []
    for _, movie in movies_to_predict.iterrows():
        predicted_rating = model.predict(user_id, movie['movieId']).est
        predictions.append((movie['movieId'], movie['title'], movie['genres'], predicted_rating))

    # Sort by predicted rating and return top picks
    predictions.sort(key=lambda x: x[3], reverse=True)
    return pd.DataFrame(predictions[:n], columns=['movieId', 'title', 'genres', 'predicted_rating'])
```

## Building the Interface

### Streamlit Web App (Will change)

Built a web interface with my favorite Gruvbox Medium theme (because everything
looks better in retro colors). The app lets you:

- Try all three recommendation types
- Filter by genre, year, or rating
- See data visualizations of the recommendations
- Browse through results with a clean, responsive interface

It's actually easy and fun to use. No boring tables or academic-looking charts -
just smooth interactions and instant results.

### Command-Line Tool (For the Terminal Nerds)

Because sometimes you just want recommendations without leaving your terminal:

```bash
# Get what's trending
./recommend.py --popular --num 10

# Find movies like The Matrix
./recommend.py --movie "The Matrix (1999)" --num 10

# Get personalized picks for user 1
./recommend.py --user 1 --num 10
```

Perfect for when you're deep in a coding session and need a break
recommendation, or maybe something playing on the second monitor as you code.

## What I Learned (The Hard Way)

### The Sparse Data Reality Check

Movie ratings are basically Swiss cheese - full of holes. Most people rate maybe
50 movies out of thousands available. Teaching an algorithm to make sense of
this was like solving a puzzle with 90% of the pieces missing.

The solution? Smart data handling and accepting that sometimes "I don't know" is
a valid recommendation algorithm response.

### The Cold Start Struggle

New users with zero rating history? That's recommendation hell. Can't use
collaborative filtering, content-based needs preferences to work with.

My workaround: Start with popular movies, then quickly learn from any ratings
they give. The system gets smarter with every interaction.

### The Echo Chamber Problem

Pure collaborative filtering can create recommendation bubbles - you only get
suggestions similar to what you already like. Sometimes you want to discover
something completely different.

That's why having multiple approaches matters. Each algorithm brings different
strengths to the table.

### Performance vs. Accuracy Balancing Act

Real-time recommendations vs. perfect accuracy - pick one. Users won't wait 30
seconds for the "perfect" recommendation when a "good" one in 2 seconds works
fine.

Lots of optimization went into making this snappy while keeping recommendations
relevant.

## What's Next?

This project is in development. Here's what I'm planning to add:

**More Content Features**: Right now I'm only using genres, but movies have so
much more - directors, actors, release year, even movie poster colors could
influence recommendations.

**Hybrid Approach**: Combining all three methods intelligently instead of
treating them separately. Sometimes you want popular + similar, sometimes you
want collaborative + content-based.

**Real User Accounts**: The Streamlit app currently uses demo data, but I want
to add user authentication so people can build their own recommendation
profiles.

**TypeScript Web App**: Let's be honest, Streamlit is basic af. I want to
rebuild this as a proper web app with Next.js/TypeScript, clean UI components,
smooth animations, and a way better user experience. Think Netflix-level polish
but for recommendations.

**Better Evaluation**: More sophisticated metrics to actually measure how good
the recommendations are. User satisfaction surveys, click-through rates, that
kind of thing.

**Cloud Deployment**: Getting the app hosted properly so people can actually use
it without running Python locally.

The [GitHub repository](https://github.com/shubhpsd/movie-recosystem) has all
the code, datasets, and setup instructions if you want to dive deeper or build
your own version!

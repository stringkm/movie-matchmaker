# movie-matchmaker

## Things to do from here
* add Pearson to Top-k
* figure out why Cosine really didn't work well
* Exploring data: make interesting plots like distribution of # of ratings for movies and users

## Requirements
* assumes the [MovieLens dataset](https://grouplens.org/datasets/movielens/20m/) is downloaded and its contents are set up in the same directory as the code in a folder named `data/` (this can be renamed but to do so the `DATA_DIR` needs to be set appropriately in `utils.py`)
* `numpy`, `pandas`, `matplotlib`

## Contents

#### Demos
* `read_data.ipynb`: example showing how to use the API built in `utils.py` to interact with the dataset with `pandas` commands

#### Processed Data
This folder is used to hold processed data that took awhile to compute (iterating over 10,000s of rows) but isn't that big storage-wise
* `movies_unflattened.pkl` is the contents of `movies.csv` but with each genre as a column and a 1 (0) if the genre applies (does not apply) to the movie
* `movies_withstats.pkl` is the same as `movies_unflattened.pkl` with two added columns for the total number of reviews each movie received and the average of those reviews

# Report

### Introduction
People all over the world have long valued the opinions of their friends with similar interests over people with whom they have little in common, especially concerning movies. How often has a glowing review from a friend or critic convinced you to go see a movie you otherwise would have ignored, or a scathing review from someone you dislike failed to change your opinion? Now, with the power of the Internet, data can be recorded and used on a global scale, expanding one’s social network far beyond the boundaries of one’s sphere of influence. Imagine the predictive power when you consider the opinions of people with similar interests whom you will likely never meet? This is the motivation behind recommendation systems. They combine the statistical power of a large sample size with the personalized recommendations of a friend, showing the user new content they might not find otherwise and bringing money to companies when they watch or buy it. 

Numerous companies have taken full advantage of this, most notably Netflix and Amazon. But with large amounts of data comes the challenge of making the best use of it in an efficient way. In a famous endeavor of crowdsourcing, Netflix launched a competition which awarded a $1 million dollar Netflix Prize to the team that designed a recommendation algorithm that improved their accuracy by 10%. Now, competitions like this are common on data science-oriented platforms such as Kaggle and InnoCentive. Despite the trends in various machine learning and statistical algorithms, the collaborative filtering method has retained its place as a useful starting place for recommendation algorithms.

#### Collaborative Filtering
To predict a movie rating for a user, collaborative filtering creates a collaboration of items or users with similar properties, while filtering out items or users with dissonant properties. It does this by clustering the items or users into a neighborhood based on a user’s past rating or purchasing history. It then uses this constructed neighborhood to predict new ratings for movies a user has not yet seen. 

#### User-based vs. item-based
For a movie recommendation engine built on movie ratings, there are multiple approaches one could take to assess similarities between users and movies. Two straightforward approaches involved clustering users (user-based) and clustering the movies (item-based). In item-based collaborative filtering, movies are clustered together based on how similar they are to other movies. Movies are then recommended to the user if they are similar to movies the user has rated favorably in the past. In user-based collaborative filtering, users are clustered together based on their past ratings history, and the ratings for an unseen movie are estimated based on the ratings by users with similar ratings histories. For this project, we will focus on building a basic user-based recommendation algorithm.

### Dataset
MovieLens is a non-commercial website run by the research group GroupLens that provides users with movie recommendations. Users can search the site for a movie based on title, genre, cast members, and more. They can also rate movies and tag them with identifying words, such as the name of the director, a topic the movie addresses, or a feeling the movie’s atmosphere evokes, etc. This identifying information helps other users find the movie in either recommendations or their own searches. MovieLens has been around for over 20 years, creating a rich and detailed data set for building accurate recommendation algorithms.

For this project, we mainly focus on the latest small version of the MovieLens dataset https://grouplens.org/datasets/movielens/latest/. This version includes 100,004 ratings and 1,296 tags for 9,125 movies uploaded by 671 users between January 9, 1995 and October 16, 2015. Each user included in the data set had rated at least 20 movies, but was otherwise chosen at random. No identifying information on the user was recorded except for their movie ratings, tags, and timestamps for the two. All included movies had at least one rating.

In addition to the reduced dataset, we also perform some data exploration on the latest full stable benchmark version https://grouplens.org/datasets/movielens/20m/. This version includes 20,000,263 ratings and 465,564 tags for 27,278 movies uploaded by 138,493 users between January 9, 1995 and March 31, 2015. It additionally includes machine-learning-generated genome data with relevance scores for 1,128 tags applied to each movie (ie. 30,769,584 relevance scores).

Since the smaller dataset we use for most of our analysis was specifically designed for developers, it may change over time. As of the date of access (April 2018) the last update was October 2016. To ensure our results are reproducible, we include a copy of the smaller 3MB dataset in our Github repository. We do not include a copy of the full benchmark version, since this version will always be accessible through the permalink and on the GroupLens website.

### Methodology
Throughout this project we utilize user-based collaborative filtering. We use the rating data to compute the similarity between users. Then we utilize those similarities to compute a prediction for the user. This prediction can then be used to provide recommendations to the user.

To evaluate the effectiveness of our methods, we split the ratings data into training and test samples. Each user has 20% of their ratings randomly selected, placed into the test sample, and masked from the training set. Note that the movies for which ratings are included (excluded) in the test (from the training) sample are not the same across users. We then predict the ratings and compute the mean squared error (MSE) between the predictions and the test sample, using this as an evaluation of effectiveness and also to compare between different flavors of our method.

#### Predicting ratings
In collaborative filtering, it is convenient to model a user’s movie ratings as a 1 x m vector in a highly dimensional space ℝm (m=number of movies available to rate). Once each user is modeled as a vector, one can calculate the similarity between different users based on the element values of their associated vectors. For n users, all of the user’s ratings combined together form a n x m matrix.

To predict the rating a user would give a particular movie, we compute a weighted average of all the ratings by other users for that movie:
ru,i=|simil(u,u')|-1simil(u, u')ru',i
where the weights are the computed similarities between the users. In the scheme where each user’s ratings is a vector, this essentially boils down to matrix multiplication (while being careful only to consider users who had rated the particular movie in question). We consider two different measures of similarity between users: Pearson correlation and vector cosine similarity. We additionally investigate the benefits of including top-k filtering in our analysis.

#### Pearson Correlation Coefficient
When treating the users’ ratings as a vectors, it is easy to determine their similarity using the Pearson correlation coefficient. Just as the 2D version measures the correlation of two 1-D vectors, the Pearson correlation measures the covariance of two m-D vectors containing two separate user’s ratings. The Pearson correlation coefficient is given by:
similP(x,y)=iIxy(rx,i -rx) (ry,i -ry)iIxy(rx,i -rx)2 iIxy(ry,i -ry)2 
where x and y are different users, Ixy contains the movies that users x and y both rated, rx,i and ry,i  are the ratings given to movie i by user x and y, respectively, and rx, ry is the mean rating over all movies in set Ixy  given by user x, y.

Because the Pearson correlation involves calculating the variance around the mean for each user, this metric is robust against incorrectly weighting users who routinely rate movies higher than other users. For instance, if user1 has an average rating of 4 and user2 has an average rating of 2, but they both rate movie a 1 point higher than their average rating and movie b 2 points lower than their average rating, they will still have a very high Pearson similarity value even though their rating values are not the same. 

While the Pearson correlation is robust against constant offsets in mean ratings between users, easy to interpret and easy to implement using pandas’ corr function, it has drawbacks. If two users have rated very few movies in common but have similar ratings for that small set of movies, their Pearson similarity value can be unrealistically high, while two users who have rated hundreds of movies in common, but have several discrepant ratings will be deemed less similar and thus weighted less heavily when predicting ratings. Another complication with using the Pearson correlation to assess similarity is that the predicted ratings can lie outside the domain of the original ratings used to calculate them. For instance, a predicted rating calculated from ratings on a scale of 0 to 5 can have a negative value or a much higher positive one than 5 depending on the similarity values used to calculate it. To offset some of these shortcomings, other similarity metrics are available, such as the cosine similarity.

#### Cosine Similarity
In addition to Pearson correlation, an increasingly popular weighting method in collaborative filtering research is the cosine similarity. If each user’s ratings compose a vector in m-dimensional space (m being the number of movies), then the cosine similarity is the cosine of the angle between the two vectors
similC(x,y)=cos()=xy||x||||y||
Now, the main difference between Pearson and cosine similarities is how to handle items that one user has rated while the other has not. Pearson, by definition, only takes into account movies that are overlapped between users. However, Cosine similarity must have the vector defined in all m-dimensions. So in practice, all movies that would appear NaN in a ratings matrix are set to 0. Cosine similarity’s main advantage over Pearson weighting is that for our parameter space [0.0, 5.0] the allowed values for cosine range from 0 to 1, which means that there cannot be predictions that fall outside of our domain.

The caveat with this approach is that it tends to dampen the similarity between two users. Imagine user1 has 20 only ratings that perfectly match user2’s, but user2 has an additional 30 ratings for movies that user1 has not rated. Each added rating from user2 adds a dimension between their vectors, increasing the angle between them, thus dampening the similarity. As a result, users with more ratings will tend to have smaller cosine similarities to all other users.

#### Top-k filtering
To further reduce the impact of users that have little similarity to each other, often algorithms will identify the k most similar users to a user of interest and only use those to compute rating predictions. All of the math is the same; the difference is restricting the sum when predicting ratings to the k most similar users. We implement top-k filtering and attempt to tune k for each weighting scheme (both Pearson and Cosine similarity) by minimizing the MSE between the training and test samples. For simplicity, we use the already computed weights to rank the users and select the k most similar. Another popular method to select the k users is to perform a nearest neighbor search.

### Results
To evaluate the effectiveness of our algorithm, we compute the mean squared error (MSE) between predicted ratings and the isolated test sample. As highlighted below, we find small differences between weight methods and marginal improvement from implementing top-k filtering. 

#### Comparing weights
If one chose the mean rating (2.5) of the range of rating values (0.5-5) as the predicted rating for a uniformly distributed set of test ratings, one would expect a MSE of ~2.125. The Pearson correlation similarity yielded an MSE of ~0.9905, which beats the constant guessing by about a rating value of 1. Meanwhile, the cosine similarity yielded an MSE of ~0.9645, slightly better than with the Pearson correlation similarity. 

#### Improvements from top-k
We find that implementing top-k filtering yields a small improvement on the MSE for both methods of computing similarity between users. For Pearson correlation similarity, the minimum MSE of ~0.947 occurs for k=400, while for cosine similarity the minimum MSE of ~0.935 occurs for k=300. We note that as k gets smaller the MSE increases at a much greater rate for Pearson correlation similarity than it does for cosine similarity.

It is also important to consider the improvements from top-k filtering in light of the increased computational time. Due to the required use of a for-loop over each user (the k most similar users are different for each active user), computational overhead becomes a major drawback to using the top-k implementation. Computing a rating prediction matrix via top-k took anywhere from 2x to ~100x longer than simply using all users when computing predictions, depending on the exact value of k. Since the complexity of the top-k algorithm is O(nk) with significant overhead from the use of the for-loop, the slight improvement of ~0.05 in MSE is likely not worth the cost, particularly if one is interested in scaling the algorithm to a larger ratings dataset (e.g. the 20 million stable benchmark version of MovieLens). 

#### Deliverables
All of the source code and data products used in this assignment are available at https://github.com/stringkm/movie-matchmaker. ratemovies.py chooses random movies from the MovieLens movie list and asks the user to rate them. After the user has rated a number of input movies (25 by default, adjustable by the user), the script creates a ratings file that can be used as input for the Pearson and Cosine recommendation engines.

#### A Test on the Authors
A straightforward test to see how our recommendation algorithm is working is to generate movie recommendations for ourselves. Using ratemovies.py, both of the authors rated 25 movies each, then predicted ratings for unwatched movies using the Pearson and Cosine similarities. The code returns the movie titles with the top 5 and lowest 5 predicted ratings. 

Using the Pearson correlation as the metric for similarity and the input contained in katelyn_ratings.csv, the collaborative filtering returned Blame it on Rio, Wristcutters: A Love Story, Butch Cassidy and the Sundance Kid, Her Alibi, and Black Hawk Down as the top 5 movie rating predictions and Watchmen, Religulous, Accepted, the People vs. Larry Flynt, and Snow White and the Huntsman as the worst 5 movie rating predictions. Given that the ratings used as input were generally favorable towards romantic comedies, historical dramas, and animated children’s movies, the titles of the best rated movies were somewhat surprising; however, the genres of the movies seem to match the inputs fairly well. However, the lowest ranked movie happens to be a title Katelyn enjoyed. Perhaps if the input contained more movie ratings than 25, the predicted ratings would be more accurate.

This was also tested on the ratings from alex_ratings.csv using cosine similarity as the weighting scheme. Charlotte's Web, Persuasion, Jane Eyre, Apache, and The Kentuckian were the top 5 highest rated movies from the Cosine similarity recommendation algorithm, while Mohenjo Daro, Daniel Tosh: Completely Serious, Ice Age: The Great Egg-Scapade, and Life is Sacred were predicted to have the lowest 5 ratings. The input ratings tended towards childhood movies and dramas, which are featured in the top 5. In addition, the only comedy rated was given 1.0 stars, so it makes sense that the bottom 5 predictions tend towards comedy.

### Conclusions
Based on our experience making these recommendation systems and testing them using our own movie preferences, we have discovered that designing a recommendation system is a complicated undertaking. Even though large data sets with millions of movie ratings like MovieLens exist, the dearth of information shared between specific users makes the ratings vectors large and sparse. With more movies coming out each year, giving users on MovieLens more variety to watch and rate, this challenge will only compound.
 
Despite this, there are several changes we could make to our algorithm to improve the accuracy of the predicted ratings. Instead of using only the similarities between users, we could also use the tag information on the movies to determine the similarity between items. Combining both the user-based similarity and the item-based similarity into a hybrid recommendation system may improve predictions about how a user will rate a specific movie.

# Recipe Amore
---
## Recap

Goal 1: To create an interactive cookbook.  

Using a wealth of recipes, create a product that displays recipes containing user-entered ingredients, whether it's the food in the fridge about to bad or an adventurous combination.  

Goal 2: Create a collaborative recommendation system based on users ratings that can help other users find recipes they might enjoy!  

---
## Data Collection, Exploratory Analysis, and Cleaning

Data collection, exploratory analysis, and cleaning processes were performed much at the same time, informing each other. These processes are separated in this document but may refer to one another within their respective sections.  
---
## Data Collection  

Initial scraping was done using a python package made by github user hhursev, which used BeautifulSoup under the hood. This scraper only pulled in the initial 5 recipe variables: Recipe ID, Title, Total time, Ingredients, Instructions, and a variable called 'Links'. This scraper only scrapes one URL at a time so automation had to be manually created and the results stored in a Pandas DataFrame. Because allrecipes recipe URLs are differentiated by the Recipe ID, "http://www.allrecipes.com/recipe/{RecipeID}", and scraping was done over a range. Many integers lead to 404 URLs with no recipe (likely, they were deleted). Many such 404 URLs were scraped, leaving empty data in the DataFrame. Recipes are user-submitted which results in many missing cooking times because the user didn't add a cook time value.  

To get Ratings and Number of ratings, a scraper had to be designed. To skip the aforementioned 404 potential recipe URLs, steps were taken: The initial pandas DataFrame had all 404 rows removed and the Recipe ID column's values were turned to a list whose values were used, instead of a range, to choose which URLs to scrape. This scraper would incorrectly assign values of zero to both target variables for some recipes, as confirmed by checking several of such null-value-rating recipes' URLs. While worrying from a scraping perspective, this is proprietary data. Thousands of accurate data points were collected: enough to build a recommender system.  

---
## EDA and Data Cleaning  

Initial, uncleaned DataFrame had 25994 rows, 8572 of which had no instructions, 8570 of which were 404 recipes. An unexpected column was confirmed to be an unnamed duplication of the recipe number. As of the typing of this document, thousands of Rating and Number of rating values were collected, but they were not yet incorporated. Only 211 ratings were collected.  

<img src="https://imgur.com/cfQTzEB.png" style="float: left; margin: 15px; height: 50px">  

#### Bummer.  

Not every URL has a recipe attached. Many have been removed either by the user or by allrecipes themselves for any number of potential reasons. Allrecipes.com uses this eggy image for their 404 to let you know they don't have a recipe there. Before building the recommender system, it was important to have these cleared out so you can better guarantee every recipe you see will be a winner (or at least extant).  

<img src="https://imgur.com/hy1i2Rx.png" style="float: left; margin: 15px; height: 50px">  

#### Pizza!  

Checking for duplicate values in the Title column revealed only three titles that repeated more than three times. The least of these was "English Trifle" and a manual search on allrecipes.com revealed they are not duplicates.  

The most common duplicated value was for the aforementioned 'Bummer." 404 URLs. Again, 8570 of these were present.  

The final duplicated title was 'Johnsonville® Three Cheese Italian Style Chicken Sausage Skillet Pizza' with 2202 repetitions. (A search on their website using that term yields 21596 results!) Searching on their website indicates these are not the actually recipe titles. Exploration of the data shows this entry seems to take the place of 404 URLs at some point.  

The pizza and bummers gone, the filter and recommender system were ready to be built.  

## Recommender: Building a DataFrame.  

The goal, again: building a collaborative recommender which takes in user ratings and uses them to predict how much another user might like a new recipe. From this point, much can be done, such as actively recommending foods to a user, but all such recommendations are (should be) based on data-driven prediction.  

#### The Cold Start Problem  

Average ratings and number of ratings for each item were not enough. Without any individual user data, recommendations cannot be provided to other users. With over 15,000 recipes, a hundred users could use a different recipe each day for a year and see little overlap in attempted recipes! This "Cold Start" problem is common to collaborative recommenders and is a large part in why they are more a part of business (or even recommendation) solutions rather than the only tool used.  

The living URLs, stored as 'Recipe Numbers' in the cleaned DataFrame were extracted and applied as columns to the DataFrame of user ratings.  

Although it would do little to recommend a decent recipe to another user with seemingly similar tastes, 21 artificial users created with randomly-generated ratings were generated to build the recommender system around to which new user data can later be added.  

## Friends! <img src="https://imgur.com/W1OL0mh.png" style="float: left; margin: 15px; height: 50px">  

By measuring similarity, weights are able to be generated; The more similar two users' tastes, the more their scores will weigh on influencing recommendations for untried, unrated recipes.  

A similarity matrix was generated in which each user's similarity to each other user was calculated and recorded to inform the recommendation algorithm how much relative credence to give potential recommendations by a particular user to a particular user. As each user rates new items, these weights will change.  

Because this is randomly generated data, the similarities are few in either the positive or the negative.  

<img src="https://imgur.com/XtUdVk7.png" style="float: left; margin: 15px; height: 50px">  

RMSE for Isaiah was 2.79: The model is bad without more user data.

## Filter  

Initial steps were misguided, but provided a bit of code retained for later use. The initial design called for a dictionary of dictionaries. Eventually, this was abandoned for Pandas. This initial code is retained because it is well formatted for outputs of recipe queries, given some modification.  

The recipe variables are:  

    Recipe ID: The unique number that appends the recipe url on allrecipes.com (integer)  
    Title: The name of the recipe (string)  
    Total time: The number of minutes the recipe should take to prepare and cook. (integer)  
    Ingredients: The ingredients that go into the recipe. (list of strings)  
    Instructions: Step-by-step cooking instructions (list)  
    Rating: The average rating left by users  
    Number of ratings: The number of ratings left by users.  

#### To Do:  

Repair the filter. Currently, it either returns a dataframe of all the recipes with all searched ingredients indiscriminately or it stops after the first ingredient.  

Ideally there would be many filters which take into account ratings. Initially, it was my hope that the recipe rating could be incorporated to this purpose. Ideas include "Top rated by all users", searches that include some randomness (ignoring ratings) for variety, searches that include highly rated recipes with few ratings (god mode: your rating can decide the fate of the recipe!), and a user-user recommender based search where foods take into account the predictions of the model. All of the pieces are present!  

Create more realistic user data. Thousands of users with fewer recipes rated each, to start. Using the number of ratings from my scraped data, I could even weigh different recipes to receive more ratings.  

Perhaps, if possible, scraping user rating data from allrecipes to create further realistic user data to eliminate the cold start issue.  

Scraping ratings and number of ratings again with a goal of eliminating the false zero values.  

Differentiate between recipes that truly have no ratings (Number of ratings = 0) and recipes that the scraper can't find (Number of ratings = N/A).  

Create a function for adding new users. While it is possible to do right now, it would have to be done by recreating the entire DataFrame with the new user's data incorporated.  

Create a function for adding new ratings. While it is possible to do right now (and easy!), this kind of fundamental (to this project) functionality belongs in a function.  

Provide an output of a recipe that is better than a pandas DataFrame.  

Provide a link to the recipe on allrecipes.  

To separate the ingredients into a set list rather than arbitrary strings that are searched through.
This could potentially be expanded for versatility. "You said 'cheese', did you mean 'cheese curd'?" This example is bad, but it illustrates what I mean.  

Link recommendations to recipes by way of recipe number.  

Make this a flask application

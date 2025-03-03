# dva-final-project

# Overview

This project uses scraped recipes from https://www.allmenus.com to recommend dishes with similar ingredients to an input dish. All restaurants in NYC serving the queried dish, or any of the 5 most similar dishes are displayed on a map. My primary responsibility was cleaning the recipe data, and determining which recipes were most similar to each other, and providing an output that was simple to translate into our visualization. Please see https://github.com/DKHowell/dva-final-project/blob/main/find_similar_dishes.py.

# 1. [Nana] Scraping Menu Data

We scraped the menus of the top 500 most popular restaurants in NYC according to https://www.allmenus.com/ny/new-york/-/.
The scraper code can be run from `scrape_menu_dishes.py`.

The list of restaurants is stored under `menu_data/restaurant_list.tsv`. Each row consists of:
- Restuarant Name
- Relative URL of Restaurant
- Restaurant's cuisine
- Restaurant address

The dishes served at each restaurant is stored under `menu_data/restaurant_dish_list.tsv`. Each row consists of:
- Relative URL of Restaurant (can be used to link back to restaurant in `menu_data/restaurant_list.tsv`)
- Name of dish
- Dish description (if available)
- Dish price

# 2. [Nana] Getting Recipe Data

We used the [Edamam Recipe Search API](https://developer.edamam.com/edamam-recipe-api).
The free developer account comes with 10K API calls a limit, max 10 calls per minute.

Hitting the API requires you to sign up for an account, which will get you an application key and application id.

The `scrape_recipes.py` script gets unique list of dish names stored in `menu_data/restaurant_dish_list.tsv` and runs each dish against Edamam's Recipe Search API.
The JSON output of each search is stored under `recipe_search_data` directory, where the search results of searching each dish is stored in a separate file. 

In this sense, each restaurant dish should correspond 1:1 to a JSON file in the `recipe_search_data` directory.

Below are examples of dish names that need cleaning up as they currently return 0 recipe results:
- "08. virginia ham" -> remove the "08" 
- "03. vegetarian omelette peppers&comma; onions&comma; tomatoes and cheese" -> remove "03. and &comma and ;"
- "sardegna sandwich platte" -> might return more recipe results of "platter" was removed
- "awesome caramel pretzel rod" -> perhaps remove "awesome" for the dish name to get more recipe results
- "hail caesar salad" -> there could be more results if we removed "hail"
- "(boar's head meat) philly cheese steak" -> There would probably be results stored if we got rid of "(boar's head meat)" in the search query

# 3. [Swaraj Patankar] Match dishes from menu data to recipes


- Sign up for an account on Edaman and get your own application key and application id so we are not rate limited by just 1 account. Just remember to modify these values in `scrape_recipes.py` `__init__`. 
- **Clean up dish names**: See notes above. Add an additional preprocessing step to the `write_recipes_for_all_dishes()` function in `scrape_recipes.py`. Then run the preprocessed dish names through the `write_recipe(dish_name)` function.
- Append to a text file where each row contains `restaurant_url`, `original_dish_name`, `matched_recipe`. This will make it easier to go between restaurant dish names -> recipes.
-- Store all recipes in a Pandas DataFrame then convert it to a CSV

# 4. [David Howell] Creating Ingredient Vectors

We took the matched restaurant dish and recipe data and converted it into vector representations of ingredients that can be used to calculate recipe similarity. The code for ingredient vector creation can be found in `dishes/find_similar_dishes.py`
- For each list of ingredients (recipe), we cleaned the individual ingredient strings, stemmed each word, and then concatenated the cleaned ingredients into one long ingredient string. This was achieved through the function `clean_ingredients()`
- We then converted these ingredient strings into vector representations by using TF-IDF in the function `create_ingredient_vectors()`
- The vector representations form a sparse matrix of shape **number of ingredients X number of unique ingredients across all recipes**

# 5. [David Howell] Finding Most Similar Dishes

We took the sparse ingredient matrix and found the most similar dishes to each recipe by using two methods: cosine similarity and kmeans clustering. We then compared the results of the two methods to determine which recommended the most similar dishes. The code can be found in `dishes/find_similar_dishes.py`

After comparison, we found that the cosine similarity algorithm produced better recommendations. This code can be found in the `compare_algorithms()` function.

The final output to be used for the visualization is a pair of csv files:

The list of restaurants and the dishes they serve is `restaurants_df.csv`. Each row consists of:
- The URL of the restaurant
- The dish name

The list dishes and their similar recommended dishes is `recipes_df.csv`. Each row consists of:
- The dish name
- The indexes in `restaurants_df.csv` that feature the dish
- The names of the Top 5 recommended dishes
- The indexes in `restaurants_df.csv` that feature the top 5 recommended dishes
- The similarity scores for each of the recommended dishes on a 1-10 scale

For visualization, the indexes of the recommended dishes from `recipes_df.csv` are mapped back to the restaurants in `restaurants_df.csv` to locate them on a map.

# 6. [Ilkay] Service and visualization

Visualization can be launched by running a python server from viz folder (python -m http.server 8888) and typing http://localhost:8888/index.html in the browser.
Visualization will depend on 2 other files viz/d3_data.csv and viz/restaurant_location/updated_restaurant_list.tsv.

Snapshot of preliminary visualization with real data is shown below. Typing/selection in dish drop-down is propogated to similar dishes and restaurants lists and the restaurants are marked on the map.

![index](images/index.JPG)

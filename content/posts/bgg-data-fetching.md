+++
title = 'BoardGameGeek board games data fetching with Python'
date = 2023-09-27T15:27:08+02:00
slug = "boardgamegeek-python-data-fetching"
+++

![Python Logo](/python-logo.png)

This script will fetch board games data from BoardGameGeek API and store the data in a CSV file. The API response is in XML format and since there is no endpoint in order to fetch multiple board games data at once, this will work by making a request to the endpoint for a single board game based on board game(s) ID, while incrementing the ID after each request withing the given range of IDs.

Check out the repo on my [GitHub profile](https://github.com/ognard/bgg-data-fetcher) 

The information fetched and stored for each board game is the following:

`name`, `game_id`, `rating`, `weight`, `year_published`, `min_players`, `max_players`, `min_play_time`, `max_pay_time`, `min_age`, `owned_by`, `categories`, `mechanics`, `designers`, `artists` and `publishers`.

We start by importing the needed libraries for this script:

```
# Import libraries
from bs4 import BeautifulSoup
from csv import DictWriter
import pandas as pd
import requests
import time
```

We'll need to define the headers for the request and the pause between each request (in seconds). An information on the requests rate limit is not available in BGG API documentation and there is some unofficial information in their forums that it is limited to 2 requests per second. The pause between requests may need to be adjusted, if the script start to hit the limit rate.

```
# Define request url headers
headers = {
	"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.16; rv:85.0) Gecko/20100101 Firefox/85.0",
	"Accept-Language": "en-GB, en-US, q=0.9, en"
}

# Define sleep timer value between requests
SLEEP_BETWEEN_REQUEST = 0.5
```

The next thing is to define the range of board game IDs that need to be fetched from BGG and processed. At the time of creating this script, the upper range limit for which there is an existing board game data is approximately 402000 ids and this number is most likely to increase in the future.

```
# Define game ids range
game_id = 264882 # initial game id
last_game_id = 264983 # max game id (currently, it's around 402000)
```

The following is a function that will be called when the script is completed based on the range of IDs. Also, if there is an error when making a request, this function will be called in order to store all the data appended to the `games` list up to the point when the exeption happened.

```
# CSV file saving function
def save_to_csv(games):
    csv_header = [
        'name', 'game_id', 'rating', 'weight', 'year_published', 'min_players', 'max_players',
        'min_play_time', 'max_play_time', 'min_age', 'owned_by', 'categories',
        'mechanics', 'designers', 'artists', 'publishers'
    ]
    with open('BGGdata.csv', 'a', encoding='UTF8') as f:
        dictwriter_object = DictWriter(f, fieldnames=csv_header)
        if f.tell() == 0:
            dictwriter_object.writeheader()
        dictwriter_object.writerows(games)
```

The in the following part is the main logic of this script. It will execute the code while in the IDs range, which means it will make requests to the BGG API, get all of the data by using BeautifulSoup, make the necessary checks if the data is related to board games (there is data that is related to other categories. Refer to ![BGG API](https://boardgamegeek.com/wiki/page/BGG_XML_API2) for more information.), after that it will process and append the data to the `games` list and finally store to the CSV file.

```
# Create an empty 'games' list where each game will be appended
games = []

while game_id <= last_game_id:
    url = "https://boardgamegeek.com/xmlapi2/thing?id=" + str(game_id) + "&stats=1"
    
    try:
        response = requests.get(url, headers=headers)
    except Exception as err:
        # In case of exception, store to CSV the fetched items up to this point.
        save_to_csv(games)
        print(">>> ERROR:")
        print(err)
    
   
    soup = BeautifulSoup(response.text, features="html.parser")
    item = soup.find("item")
    
    # Check if the request returns an item. If not, break the while loop
    if item:
        # If the item is not a board game - skip
        if not item['type'] == 'boardgame':
            game_id += 1
            continue
        
        # Set values for each field in the item
        name = item.find("name")['value']
        year_published = item.find("yearpublished")['value']
        min_players = item.find("minplayers")['value']
        max_players = item.find("maxplayers")['value']
        min_play_time = item.find("minplaytime")['value']
        max_play_time = item.find("maxplaytime")['value']
        min_age = item.find("minage")['value']
        rating = item.find("average")['value']
        weight = item.find("averageweight")['value']
        owned = item.find("owned")['value']
        categories = []
        mechanics = []
        designers = []
        artists = []
        publishers = []
        
        links = item.find_all("link")
        
        for link in links:
            if link['type'] == "boardgamecategory":
                categories.append(link['value'])
            if link['type'] == "boardgamemechanic":
                mechanics.append(link['value'])
            if link['type'] == "boardgamedesigner":
                designers.append(link['value'])
            if link['type'] == "boardgameartist":
                artists.append(link['value'])
            if link['type'] == "boardgamepublisher":
                publishers.append(link['value'])
                
        game = {
            "name": name,
            "game_id": game_id,
            "rating": rating,
            "weight": weight,
            "year_published": year_published,
            "min_players": min_players,
            "max_players": max_players,
            "min_play_time": min_play_time,
            "max_play_time": max_play_time,
            "min_age": min_age,
            "owned_by": owned,
            "categories": ', '.join(categories),
            "mechanics": ', '.join(mechanics),
            "designers": ', '.join(designers),
            "artists": ', '.join(artists),
            "publishers": ', '.join(publishers),
        }
        
        # Append the game (item) to the 'games' list
        
        games.append(game)
                   
    else:
        # If there is no data for the request - skip to the next one
        print(f">>> Empty item. Skipped item with id ({game_id}).")
        game_id += 1
        continue
    
    # Increment game id and set sleep timer between requests
    game_id += 1
    time.sleep(SLEEP_BETWEEN_REQUEST)
    
save_to_csv(games)    
```

Below you can preview the first few rows of records in the CSV file as pandas DataFrame.

```
# Preview the CSV as pandas DataFrame
df = pd.read_csv('./BGGdata.csv')
print(df.head(5))
```
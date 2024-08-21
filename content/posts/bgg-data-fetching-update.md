+++
title = "I've updated the Python fetcher for BoardGameGeek data"
date = 2023-09-28T15:31:18+02:00
slug = "boardgamegeek-python-data-fetching-updated"
+++

![Python Logo](/python-logo.png)

This script will fetch items data from BoardGameGeek API and store the data in a CSV file. 

I updated the [preivous script](https://www.ognard.com/posts/boardgamegeek-python-data-fetching/). Since the API response is in XML format and since there is no endpoint to fetch all items at once, the previous script would loop through a provided IDs range, making calls one by one for each item. That's not optimal, it takes long time for larger range of IDs (currently the highest number of items (IDs) available on BGG goes as high as 400k+) and the results may not be reliable. Therefore, with some modifications in this script, more item IDs will be added as parameter value to a single request url, and with that, a single response will return multiple items (~800 was the highest number that a single response returned back. BGG may eventually change this later; you can easily tweak `batch_size` in order to adjust if needed).

Additionally, this script will fetch all of the items, and not only the data related to board games.

The information fetched and stored for each board game is the following:

`name`, `game_id`, `type`, `rating`, `weight`, `year_published`, `min_players`, `max_players`, `min_play_time`, `max_pay_time`, `min_age`, `owned_by`, `categories`, `mechanics`, `designers`, `artists` and `publishers`.

The updates in this script is as it follows; We start by importing the needed libraries for this script:

```
# Import libraries
from bs4 import BeautifulSoup
from csv import DictWriter
import pandas as pd
import requests
import time
```

The following is a function that will be called when the script is completed based on the range of IDs. Also, if there is an error when making a request, this function will be called in order to store all the data appended to the `games` list up to the point when the exeption happened.

```
# CSV file saving function
def save_to_csv(games):
    csv_header = [
        'name', 'game_id', 'type', 'rating', 'weight', 'year_published', 'min_players', 'max_players',
        'min_play_time', 'max_play_time', 'min_age', 'owned_by', 'categories',
        'mechanics', 'designers', 'artists', 'publishers'
    ]
    with open('bgg.csv', 'a', encoding='UTF8') as f:
        dictwriter_object = DictWriter(f, fieldnames=csv_header)
        if f.tell() == 0:
            dictwriter_object.writeheader()
        dictwriter_object.writerows(games)
```

We'll need to define the headers for the requests. Pause between requests can be set through `SLEEP_BETWEEN_REQUESTS` (I have seen some information that rate limit is 2 requests per second, but it may be outdated information since I had no trouble with pause being set to 0). Additionally, here are set the values for starting point ID (`start_id_range`), maximum range (`max_id_range`) and `batch_size` is how many games should the response return back. Base url is defined in this section, but the IDs are added in the next part of the script. 

```
# Define request url headers
headers = {
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.16; rv:85.0) Gecko/20100101 Firefox/85.0",
    "Accept-Language": "en-GB, en-US, q=0.9, en"
}

# Define sleep timer value between requests
SLEEP_BETWEEN_REQUEST = 0

# Define max id range
start_id_range = 0
max_id_range = 403000
batch_size = 800
base_url = "https://boardgamegeek.com/xmlapi2/thing?id="
```

The in the following part is the main logic of this script. At first, based on the batch size, it will make a string of IDs that are within the defined IDs range, but not more IDs than the defined number in `batch_size` and that will be appended to `id` parameter of the url. With that, each response will return data for the number of items that is same as the batch size. After that it will process and append the data to the `games` list for each response and finally append to the CSV file.

```
# Main loop that will iterate between the starting and maximum range in intervals of the batch size
for batch_start in range(start_id_range, max_id_range, batch_size):
    # Make sure that the batch size will not exceed the maximum ids range
    batch_end = min(batch_start + batch_size - 1, max_id_range)
    # Join and append to the url the IDs within batch size
    ids = ",".join(map(str, range(batch_start, batch_end + 1)))
    url = f"{base_url}?id={ids}&stats=1"
    
    # If by any chance there is an error, this will throw the exception and continue on the next batch
    try:
        response = requests.get(url, headers=headers)
    except Exception as err:
        print(err)
        continue
    
    
    if response.status_code == 200:
        soup = BeautifulSoup(response.text, features="html.parser")
        items = soup.find_all("item")
        games = []
        for item in items:
            if item:
                try:
                    # Find values in the XML
                    name = item.find("name")['value'] if item.find("name") is not None else 0
                    year_published = item.find("yearpublished")['value'] if item.find("yearpublished") is not None else 0
                    min_players = item.find("minplayers")['value'] if item.find("minplayers") is not None else 0
                    max_players = item.find("maxplayers")['value'] if item.find("maxplayers") is not None else 0
                    min_play_time = item.find("minplaytime")['value'] if item.find("minplaytime") is not None else 0
                    max_play_time = item.find("maxplaytime")['value'] if item.find("maxplaytime") is not None else 0
                    min_age = item.find("minage")['value'] if item.find("minage") is not None else 0
                    rating = item.find("average")['value'] if item.find("average") is not None else 0
                    weight = item.find("averageweight")['value'] if item.find("averageweight") is not None else 0
                    owned = item.find("owned")['value'] if item.find("owned") is not None else 0
                    
                    
                    link_type = {'categories': [], 'mechanics': [], 'designers': [], 'artists': [], 'publishers': []}
                    
                    links = item.find_all("link")
            
                    # Append value(s) for each link type
                    for link in links:                            
                        if link['type'] == "boardgamecategory":
                            link_type['categories'].append(link['value'])
                        if link['type'] == "boardgamemechanic":
                            link_type['mechanics'].append(link['value'])
                        if link['type'] == "boardgamedesigner":
                            link_type['designers'].append(link['value'])
                        if link['type'] == "boardgameartist":
                            link_type['artists'].append(link['value'])
                        if link['type'] == "boardgamepublisher":
                            link_type['publishers'].append(link['value'])
                    
                    # Append 0 if there is no value for any link type
                    for key, ltype in link_type.items():
                        if not ltype:
                            ltype.append("0")
                         
                    game = {
                        "name": name,
                        "game_id": item['id'],
                        "type": item['type'],
                        "rating": rating,
                        "weight": weight,
                        "year_published": year_published,
                        "min_players": min_players,
                        "max_players": max_players,
                        "min_play_time": min_play_time,
                        "max_play_time": max_play_time,
                        "min_age": min_age,
                        "owned_by": owned,
                        "categories": ', '.join(link_type['categories']),
                        "mechanics": ', '.join(link_type['mechanics']),
                        "designers": ', '.join(link_type['designers']),
                        "artists": ', '.join(link_type['artists']),
                        "publishers": ', '.join(link_type['publishers']),
                    }
                    
                    # Append current item to games list
                    games.append(game)
                except TypeError:
                    print(">>> NoneType error. Continued on the next item.")
                    continue
        save_to_csv(games)
               
        print(f">>> Request successful for batch {batch_start}-{batch_end}")
    else:
        print(f">>> FAILED batch {batch_start}-{batch_end}")
    
    # Pause between requests
    time.sleep(SLEEP_BETWEEN_REQUEST)
```

Below you can preview the first few rows of records in the CSV file as pandas DataFrame.

```
# Preview the CSV as pandas DataFrame
df = pd.read_csv('./bgg.csv')
print(df.head(5))
```
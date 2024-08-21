+++
title = 'SQL Data Exploration for BoardGameGeek datasets'
date = 2023-09-30T15:34:24+02:00
slug = "boardgamegeek-sql-data-exploration"
+++

![SQL Logo](/sql-logo.png)

Based on the BoardGameGeek data fetched with [the Python script](https://github.com/ognard/bgg-batch-fetcher) (and [this script](https://github.com/ognard/bgg-csv-splitter)) I've created before, I have made some SQL data exploration for the datasets that are available in the repository. You can check the SQL queries in the `queries.sql` file in [this repo](https://github.com/ognard/bgg-sql-exploration), or clone the repo to your machine and use the datasets for your own exploration needs.

For start I will find number of games published in each year, and which year have the highest number of published games.
```
SELECT year_published, COUNT(*) total_games
FROM bgg.items
WHERE type = 'boardgame'
	AND year_published <> '0'
	AND rating <> '0'
	AND year_published < 2024
	AND year_published > 2012
GROUP BY year_published
ORDER BY year_published;
```

The output:

|year_published|total_games|
|--------------|-----------|
|2013|2539|
|2014|2941|
|2015|3158|
|2016|3439|
|2017|3619|
...


Next exploration is about the average rating of games per year for the past 20 years:
```
SELECT year_published, ROUND(AVG(rating), 3) AS rating
FROM bgg.items
WHERE type = 'boardgame'
	AND year_published <> '0'
   	AND rating <> '0'
   	AND year_published < 2024
	AND year_published > 2012
GROUP BY year_published
ORDER BY year_published;
```
The output:

|year_published|rating|
|--------------|------|
|2013|6.284|
|2014|6.289|
|2015|6.369|
|2016|6.517|
|2017|6.645|
...


Followed by a discovery for the top 10 board game categories in the past 10 years:
```
SELECT categories, SUM(games) AS total_games
FROM (
    SELECT categories, COUNT(*) AS games
    FROM bgg.items AS items
    LEFT JOIN bgg.categories AS categories
    USING (game_id)
    WHERE year_published <> '0'
        AND categories <> '0'
        AND type = 'boardgame'
        AND year_published > 2012
        AND year_published < 2024
    GROUP BY categories
) AS category_counts
GROUP BY categories
ORDER BY total_games DESC
LIMIT 10;
```

The output:

|categories|total_games|
|----------|-----------|
|Card Game|14860|
|Party Game|5345|
|Wargame|5129|
|Fantasy|4928|
|Children's Game|4164|
...


And also, beside the top categories, here are the 10 most popular mechanics in the past 10 years:
```
SELECT mechanics, SUM(games) AS total_games
FROM (
    SELECT mechanics, COUNT(*) AS games
    FROM bgg.items AS items
    LEFT JOIN bgg.mechanics AS mechanics
    USING (game_id)
    WHERE year_published <> '0'
        AND mechanics <> '0'
        AND type = 'boardgame'
        AND year_published > 2012
        AND year_published < 2024
    GROUP BY mechanics
) AS mechanics_counts
GROUP BY mechanics
ORDER BY total_games DESC
LIMIT 10;
```

The output:

|mechanics|total_games|
|---------|-----------|
|Dice Rolling|10461|
|Hand Management|7833|
|Set Collection|5305|
|Variable Player Powers|4215|
|Cooperative Game|4195|
...


Next in the list are the top five categories in the past 10 years with total games per category / per year ratio
```
WITH top_categories AS (
    SELECT categories, COUNT(*) AS total_games
    FROM bgg.items AS items
    LEFT JOIN bgg.categories AS category
    USING (game_id)
    WHERE type = 'boardgame'
        AND year_published <> '0'
        AND rating <> '0'
        AND year_published > 2012
        AND year_published < 2024
    GROUP BY categories
    ORDER BY total_games DESC
    LIMIT 5
)
SELECT year_published, categories, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.categories AS category
USING (game_id)
WHERE type = 'boardgame'
	AND year_published <> '0'
	AND rating <> '0'
	AND year_published > 2012
	AND year_published < 2024
	AND categories IN (SELECT categories FROM top_categories)
GROUP BY year_published, categories
ORDER BY year_published
```

The output:

|year_published|categories|total_games|
|--------------|----------|-----------|
|2013|Card Game|817|
|2013|Dice|238|
|2013|Fantasy|234|
|2013|Party Game|269|
|2013|Wargame|325|
|2014|Card Game|1022|
...

In addition, I've done the similar exploration for the top five mechanics in the past 10 years with total games per category / per year ratio:
```
WITH top_mechanics AS (
    SELECT mechanics, COUNT(*) AS total_games
    FROM bgg.items AS items
    LEFT JOIN bgg.mechanics AS mechanic
    USING (game_id)
    WHERE type = 'boardgame'
        AND year_published <> '0'
        AND rating <> '0'
        AND year_published > 2012
        AND year_published < 2024
    GROUP BY mechanics
    ORDER BY total_games DESC
    LIMIT 5
)
SELECT year_published, mechanics, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.mechanics AS mechanic
USING (game_id)
WHERE type = 'boardgame'
	AND year_published <> '0'
	AND rating <> '0'
	AND year_published > 2012
	AND year_published < 2024
	AND mechanics IN (SELECT mechanics FROM top_mechanics)
GROUP BY year_published, mechanics
ORDER BY year_published
```

The output:

|year_published|mechanics|total_games|
|--------------|---------|-----------|
|2013|Cooperative Game|148|
|2013|Dice Rolling|635|
|2013|Hand Management|487|
|2013|Set Collection|327|
|2013|Variable Player Powers|260|
...

Next are the top 15 most active publishers in the past 10 years,
```
SELECT publishers, COUNT(*) AS total_games
FROM bgg.items as items
LEFT JOIN bgg.publishers
USING (game_id)
WHERE publishers <> '0'
	AND year_published > 2012
	AND year_published < 2024
	AND type = 'boardgame'
	AND publishers NOT IN (
        '(Self-Published)',
        '(Web published)',
        'Inc.',
        'LLC',
        'Ltd.',
        '(Unknown)',
        '(Looking for a publisher)',
        '(Public Domain)'
    )
GROUP BY publishers
ORDER BY total_games DESC
LIMIT 15;
```

The output:

|publishers|total_games|
|----------|-----------|
|Hasbro|506|
|Pegasus Spiele|468|
|Hobby World|397|
|Korea Boardgames Co.|384|
|Rebel Sp. z o.o.|369|
...

top 15 game designers in the past 10 years,
```
SELECT designers, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.designers
USING (game_id)
WHERE type = "boardgame"
	AND year_published > 2012
	AND year_published < 2024
	AND designers NOT IN (
		'0',
		'(Uncredited)',
		'Jr.'
	)
GROUP BY designers
ORDER BY total_games DESC
LIMIT 15;
```

The output:

|designers|total_games|
|---------|-----------|
|Reiner Knizia|203|
|Paul Rohrbaugh|130|
|Joseph Miranda|110|
|Prospero Hall|109|
|Charles Darrow|108|
...

and the top 15 game artists in the past 10 years:
```
SELECT artists, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.artists
USING (game_id)
WHERE type = "boardgame"
	AND year_published > 2012
	AND year_published < 2024
	AND artists NOT IN (
		'0',
		'(Uncredited)'
	)
GROUP BY artists
ORDER BY total_games DESC
LIMIT 15;
```

The output:

|artists|total_games|
|-------|-----------|
|Joe Youst|168|
|Mark Mahaffey|135|
|Ilya Kudriashov|123|
|Klemens Franz|109|
|Michael Menzel|106|
...

There are two more discoveries that I have made, one being the top 15 categories in the past 10 years that have the highest average rating (although, these are not ordered by the games published in these categories; some other categories have more games published)
```
SELECT categories, ROUND(AVG(rating), 3) AS average_rating, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.categories AS categories
USING (game_id)
WHERE type = 'boardgame'
	AND categories <> '0'
	AND year_published > 2012
	AND year_published < 2024
GROUP BY categories
ORDER BY average_rating DESC
LIMIT 15;
```

The output:

|categories|average_rating|total_games|
|----------|--------------|-----------|
|World War II|6.504|1448|
|Vietnam War|6.473|96|
|Civilization|6.437|466|
|Renaissance|6.382|311|
|Civil War|6.353|199|
...

And finally, the top 15 mechanics in the past 10 years that have the highest average rating:
```
SELECT mechanics, ROUND(AVG(rating), 3) AS average_rating, COUNT(*) AS total_games
FROM bgg.items AS items
LEFT JOIN bgg.mechanics AS mechanics
USING (game_id)
WHERE type = 'boardgame'
	AND mechanics <> '0'
	AND year_published > 2012
	AND year_published < 2024
GROUP BY mechanics
ORDER BY average_rating DESC
LIMIT 15;
```

The output:

|mechanics|average_rating|total_games|
|---------|--------------|-----------|
|Auction: English|8.348|2|
|Tags|7.6|32|
|Neighbor Scope|7.557|8|
|Auction Compensation|7.374|2|
|Ratio / Combat Results Table|7.374|137|
...
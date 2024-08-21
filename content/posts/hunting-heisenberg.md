+++
title = 'Hunting Heisenberg with Django Rest Framework'
date = 2023-09-04T15:18:54+02:00
slug = "hunting-heisenberg-django-rest-framework"
+++

![Heisenberg Logo](/heisenberg-logo.png)

## The idea

The idea was to create a simple platform for DEA agents, to manage information about characters from the Breaking Bad/Better Call Saul universe. To make the DEA agents' life easier, they need to have an API endpoint that allows filtering information about characters by their name, date of birth, occupation or the fact whether they are a suspect.

As the DEA is trying to put drug lords behind bars, they are tracking them and the people around their location. They store timestamps and particular locations as geographical coordinates in a related table. The endpoint that will expose the data needs to allow filtering of location entries, that were within a specific distance from a particular geographical point, as well as who they were assigned to, and the datetime range of when they were recorded. The ordering for this endpoint should allow taking into consideration distance from a specified geographical point, both ascending and descending.

To see how it was done, you can set up this project locally by following the documentation below and testing it on your own.

You can find the code in my GitHub repository [here](https://github.com/drangovski/breaking-bad-api).

## Setting up the project

As prerequisites, you will need to have **Docker** and **docker-compose** installed on your system.

At first, go to your project's folder and clone the Breaking Bad API repository:

`git clone git@github.com:drangovski/breaking-bad-api.git`

`cd breaking-bad-api`

Then you will need to create and `.env` file where you will put the values for the following variables:

```
POSTGRES_USER=heisenberg
POSTGRES_PASSWORD=iamthedanger
POSTGRES_DB=breakingbad
DEBUG=True
SECRET_KEY="<SECRET KEY>"
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=breakingbad
SQL_USER=heisenberg
SQL_PASSWORD=iamthedanger
SQL_HOST=db<br />
SQL_PORT=5432
```

> Note: If you want, you can use the **env\_generator.sh** file to create .env file for you. This will automatically generate the **SECRET\_KEY** as well. To run this file, first, give it permission with chmod +x env\_generator.sh and then run it with ./env\_generator.sh

Once you have this set, you can run:

`docker-compose build`

`docker-compose up`

This will start the Django application at [`localhost:8000`](http://localhost:8000). To access the API, the URL will be [`localhost:8000/api`](http://localhost:8000/api).

## Mock Locations

For the sake of the theme of these projects (and eventually, to make your life a bit easier :)), you can eventually use the following locations and their coordinates:

| Location | Longitude | Latitude |
| --- | --- | --- |
| Los Pollos Hermanos | 35.06534619552971 | \-106.64463423464572 |
| Walter White House | 35.12625330483283 | \-106.53566597939896 |
| Saul Goodman Office | 35.12958969793146 | \-106.53106126774908 |
| Mike Ehrmantraut House | 35.08486667169461 | \-106.64115047513016 |
| Jessie Pinkman House | 35.078341181544396 | \-106.62404891988452 |
| Hank & Marrie House | 35.13512843853582 | \-106.48159991250327 |



```
import requests
import json

url = 'http://localhost:8000/api/characters/'

headers = {'Content-Type' : "application/json"}

response = requests.get(url, headers=headers, verify=False)

if response.status_code == 200:
    data = response.json()
    print(json.dumps(data, indent=2))
else:
    print("Request failed with status code:", response.status_code)
```

## Characters

### Retrieve all characters

To retrieves all existing characters in the database.

`GET /api/characters/`


```
[
    {
        "id": 1,
        "name": "Walter White",
        "occupation": "Chemistry Professor",
        "date_of_birth": "1971",
        "suspect": false
    },
	{
        "id": 2,
        "name": "Tuco Salamanca",
        "occupation": "Grandpa Keeper",
        "date_of_birth": "1976",
        "suspect": true
    }
]
```

### Retrieve a single character

To retrieve a single character, pass the character's `ID` to the enpoint.

`GET /api/characters/{id}`

## Create a new character

You can use the POST method to to **/characters/** endpoint in order to create a new character.

`POST /api/characters/`

### Creation parameters

You will need to pass the following parameters in the query, in order to successfully create a character:

```
{
	"name": "string",
	"occupation": "string",
	"date_of_birth": "string",
	"suspect": boolean
}
```

| Parameter | Description |
| --- | --- |
| name | String value for the name of the character. |
| occupation | String value for the occupation of the character. |
| date\_of\_birth | String value for the date of brith. |
| suspect | Boolean parameter. True if suspect, False if not. |

## Character ordering

Ordering of the characters can be done by two fields as parameters: **name** and **date\_of\_birth**

`GET /api/characters/?ordering={name / date_of_birth}`

| Parameter | Description |
| --- | --- |
| name | Order the results by the name field. |
| date\_of\_birth | Order the results by the date\_of\_birth field. |

Additionally, you can add the parameter **ascending** with a value **1** or **0** to order the results in ascending or descending order.

`GET /api/characters/?ordering={name / date_of_birth}&ascending={1 / 0}`

| Parameter | Description |
| --- | --- |
| &ascending=1 | Order the results in ascending order by passing **1** as a value. |
| &ascending=0 | Order the results in descending order by passing **0** as a value. |

### Character filtering

To filter the characters, you can use the parameters in the table below. Case insensitive.

`GET /api/characters/?name={text}`

| Parameter | Description |
| --- | --- |
| /?name={text} | Filter the results by name. It can be any length and case insensitive. |
| /?occupaton={text} | Filter the results by occupation. It can be any length and case insensitive. |
| /?suspect={True / False} | Filter the results by suspect status. It can be True or False. |

### Character search

You can also use the search parameter in the query to search characters and retrieve results based on the fields listed below.

`GET /api/characters/?search={text}`

* name
    
* occupation
    
* date\_of\_birth
    

## Update a character

To update a character, you will need to pass the {id} of a character to the URL and make a PUT method request with the parameters in the table below.

`PUT /api/characters/{id}`

```
{
	"name": "Mike Ehrmantraut",
	"occupation": "Retired Officer",
	"date_of_birth": "1945",
	"suspect": false
}
```

| Parameter | Description |
| --- | --- |
| name | String value for the name of the character. |
| occupation | String value for the occupation of the character. |
| date\_of\_birth | String value for the date of birth. |
| suspect | Boolean parameter. True if suspect, False if not. |

## Delete a character

To delete a character, you will need to pass the {id} of a character to the URL and make DELETE method request.

`DELETE /api/characters/{id}`

## Locations

### Retrieve all locations

To retrieves all existing locations in the database.

`GET /api/locations/`


```

[
    {
        "id": 1,
        "name": "Los Pollos Hermanos",
        "longitude": 35.065442792232716,
        "latitude": -106.6444840309555,
        "created": "2023-02-09T22:04:32.441106Z",
        "character": {
            "id": 2,
            "name": "Tuco Salamanca",
            "details": "http://localhost:8000/api/characters/2"
        }
    },
]
```

### Retrieve a single location

To retrieve a single location, pass the locations `ID` to the endpoint.

`GET /api/locations/{id}`

### Create a new location

You can use the POST method to **/locations/** endpoint to create a new location.

`POST /api/locations/`

#### Creation parameters

You will need to pass the following parameters in the query, to successfully create a location:

```
{
	"name": "string",
	"longitude": float,
	"latitude": float,
	"character": integer
}
```

| Parameter | Description |
| --- | --- |
| name | The name of the location. |
| longitude | Longitude of the location. |
| latitude | Latitude of the location. |
| character | This is the id of a character. It is basically ForeignKey relation to the Character model. |

> Note: Upon creation of an entry, the Longitude and the Latitude will be converted to a PointField() type of field in the model and stored as a calculated geographical value under the field `coordinates`, in order for the location coordinates to be eligible for GeoDjango operations.

### Location ordering

Ordering of the locations can be done by providing the parameters for the longitude and latitude coordinates for a single point, and a radius (in meters). This will return all of the locations stored in the database, that are in the provided radius from the provided point (coordinates).

`GET /api/locations/?longitude={longitude}&latitude={latitude}&radius={radius}`

| Parameter | Description |
| --- | --- |
| longitude | The longitude parameter of the radius point. |
| latitude | The latitude parameter of the radius point. |
| radius | The radius parameter (in meters). |

Additionally, you can add the parameter **ascending** with values **1** or **0** to order the results in ascending or descending order.

`GET /api/locations/?longitude={longitude}&latitude={latitude}&radius={radius}&ascending={1 / 0}`

| Parameter | Description |
| --- | --- |
| &ascending=1 | Order the results in ascending order by passing **1** as a value. |
| &ascending=0 | Order the results in descending order by passing **0** as a value. |

### Locaton filtering

To filter the locations, you can use the parameters in the table below. Case insensitive.

`GET /api/locations/?character={text}`

| Parameter | Description |
| --- | --- |
| /?name={text} | Filter the results by location name. It can be any length and case insensitive. |
| /?character={text} | Filter the results by character. It can be any length and case insensitive. |
| /?created={timeframe} | Filter the results by when they were created. Options: today, yesterday, week, month & year. |


> Note: You can combine filtering parameters with ordering parameters. Just keep in mind that if you filter by any of these fields above and want to use the ordering parameters, you will always need to pass longitude, latitude and radius altogether. Additionally, if you need to use ascending parameter for ordering, this parameter can't be passed without longitude, latitude and radius as well.

### Update a location

To update a location, you will need to pass the {id} of locations to the URL and make a PUT method request with the parameters in the table below.

`PUT /api/locations/{id}`


```
{
    "id": 1,
    "name": "Los Pollos Hermanos",
    "longitude": 35.065442792232716,
    "latitude": -106.6444840309555,
    "created": "2023-02-09T22:04:32.441106Z",
    "character": {
        "id": 2,
        "name": "Tuco Salamanca",
        "occupation": "Grandpa Keeper",
        "date_of_birth": "1975",
        "suspect": true
    }
}
```

| Parameter | Description |
| --- | --- |
| name | String value for the name of the location. |
| longitude | Float value for the longitude of the location. |
| latitude | Float value for the latitude of the location. |

## Delete a location

To delete a location, you will need to pass the {id} of a location to the URL and make a DELETE method request.

`DELETE /api/locations/{id}`
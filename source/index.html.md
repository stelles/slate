---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - json

toc_footers:
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

<!-- includes:
  - errors -->

search: true
---

# Introduction

The Data Connector (DC) will be an HTTP Rest API server which the Stratifyd Web Client will push queries to the DC server and the DC server will retrieve the data based on the query and push it back to the Stratifyd server for analysis. Please look at this [server flow diagram](https://imgur.com/XLWPOsS) for a quick understanding of how the flow will act. 


# Data Connector
##Fields

> Example data connector looks like this.

```json
{
  "name" : "amazon",
  "display_name" : "Amazon",
  "params" : [
    {
      "regex": "amazon\\.[a-z]{2,3}.*\\/([A-Z0-9]{10})", 
      "multi": true, 
      "display_name": "Amazon Item URL", 
      "name": "item_url", 
      "req": true, 
      "type": "string", 
      "description": "Copy the Amazon item URL and paste it in the field. We can handle different languages."
    }
  ],
  "enabled" : true,
  "date_range" : true,
  "image_url" : "https://beta.stratifyd.com/images/dataconnectors/icon_amazon.svg",
  "roles" : "pm"
}
```

###Required
These fields are all required

`name` - URL safe variable name for data connector

`display_name` - Human readable name

`params` - array of [data connector parameters](#input-parameters) which control the input for the data connector.

`enabled` - boolean to enabled/disable the data connector

`date_range` - boolean to allow date range selection

`image_url` - an SVG image url which will be used as the image icon

###Optional
These fields are optional but allow for extra customization for the data connector

`description` - describes what the data connector does and what data it will pull down

`subdomain` - array of domains which will have access to the data connector (default all access)

`role` - array of predefined roles to categorize the data connector (defaults to category all)

* `cx` - Customer Experience
* `pm` - Product Management
* `research` - Research
* `hr` - Human Resources
* `marketing` - Marketing

##Input Parameters

> Examples

```json
    {
      "regex": "amazon\\.[a-z]{2,3}.*\\/([A-Z0-9]{10})", 
      "multi": true, 
      "display_name": "Amazon Item URL", 
      "name": "item_url", 
      "req": true, 
      "type": "string", 
      "description": "Copy the Amazon item URL and paste it in the field. We can handle different languages."
    }
```

```json
    {
      "display_name": "Comments Only",
      "name": "comments_only",
      "default": true,
      "req": false,
      "type": "bool",
      "description": "Ignore a user's self posts."
    }
```

```json
    {
      "display_name": "Ratings",
      "description": "Specify which ratings you would like to search or none to search them all.",
      "default": {
          "1": True,
          "2": True,
          "3": True,
          "4": True,
          "5": True,
      },
      "req": False,
      "type": "multi_bool",
      "name": "ratings"
    }
```

```json
    {
      "type": 'integer',
      "req": True,
      "description": "Sets max number of comments to try to crawl and analyze. (Max 50,000)",
      "display_name": "Comment Limit",
      "name": "max_comments",
      "default": 10000,
      "max": 50000,
      "min": 0,
    }
```

```json
    {
      "type" : "list",
      "req" : true,
      "display_name" : "Text Analysis",
      "name" : "chat_type",
      "description" : "Please select if you want to analyze the agent's chat, the visitor chat or both together.",
      "options": [
          {
            "label" : "Agent",
            "value" : "agent"
          },
          {
            "label" : Visitor",
            "value" : "visitor"
          }
      ],
      "default" : "Agent and Visitor"
    }
```

These fields make up generic input parameters for the data connectors. The starred fields for each type are optional unless specfied.

### Required
`display_name` - Human readable string with the parameters name

`name` - Programatic variable used for passing the parameter between servers.

`type` - declares what the input [parameter type](#parameter-types) 

`req` - boolean declaring if the paramter is required

<aside class="warning">The parameter names `start_date` and `end_date` are **NOT** allowed.</aside>


### Parameter    Types
Each of these types is a different type of input

#### String

`string` - user passes in a string and will be returned as a string or array of strings if `multi : true`.

* `regex` - string of regular expression for verification on the input
* `multi` - boolean modifier which allows for many strings to be passed in as well as changes return from string to array of strings.

####Boolean

`bool` - a boolean flag for the user to select

* `default` - boolean value that the input is defaults to

####Multi Boolean

`multi_bool` - a key, value of input names and boolean flags respectively

* `default` - *required* JSON object with key,value pairs where the key is the flag and the value is a boolean

####Integer

`int` - integer for the user to enter

* `min` - integer that declares the minimum value the user can select
* `max` - integer that declares the maximum value the user can select
* `default` - default integer

####List

`list` - gives the user a list of pre-defined inputs to select

* `options` - **required** Array of JSON objects which each contain the keys `label` and `value` where the label is the human readable string and the value is the programtatic value which will be returned
* `multi` - allows the user to select multiple entries from the options list
* `default` - string of the default `value` key inside the options array


---------------------------------------------------------
#Server Calls

```python
from requests import get

req = get("http://localhost/external")
```

> The above will return all data connectors available on this server.

```json
  [
    {
      "name" : "amazon",
      "display_name" : "Amazon",
      "params" : [
        {
          "regex": "amazon\\.[a-z]{2,3}.*\\/([A-Z0-9]{10})", 
          "multi": true, 
          "display_name": "Amazon Item URL", 
          "name": "item_url", 
          "req": true, 
          "type": "string", 
          "description": "Copy the Amazon item URL and paste it in the field. We can handle different languages."
        }
      ],
      "enabled" : true,
      "description" : "Copy the Amazon Product url(s) and Stratifyd will analyze the product reviews."
      "date_range" : true,
      "image_url" : "https://beta.stratifyd.com/images/dataconnectors/icon_amazon.svg",
      "roles" : "pm"
    }
  ]
```


The DC HTTP Server only must handle 2 calls, a GET request for Data connector parameters and POST request that begin the data gathering process which then POST's the data back to the Stratifyd server.

## Load Data Connectors
This will be the **only** GET reqeuest needed for the DC server.  The response will be a JSON array of [data connectors](#data-connector). The endpoint will be `/external`. There will be no URL parameters or special headers.

## Start Data Stream

The DC server process once the query data request is made can be found [here](https://imgur.com/PJ9clb8).  Please review the decision tree which the DC server should follow.

```python
from requests import post

body = {
    "start_date" : null,
    "end_date" : null,
    "item_url" : "https://www.amazon.com/gp/product/B0013JOK8K"
}

### These url params are important for returning data please save them
url_params = {
    'token':  $ACCESS_TOKEN,
    'domain': SUBDOMAIN,
    "id": $STREAM_ID,
}

req = post("http://localhost/external/amazon", data=body)
```

> Above request will return [indexes](#stratifyd-indexes) for the analysis. Example response below

```json
  {
    "text_index" : ["review_text"],
    "date_index" : {
      "unknown" : "review_date",
    }
  }
```

###Stratifyed Indexes
These indexes are used for our analysis engine to process specfic fields. The basic format is `index_type : field_name`

> Example Indexes

```json
  {
    "text_index" : [
      "comment", 
      "title"
    ]
  }
```

```json
  {
    "date_index" :{
      'unknown' : 'review_date'
    }
  }
```

```json
  {
    "geo_index" : {
      "global" : "customer_country",
      "subglobal" : "customer_state",
      "local" : "customer_city"
    }
  }
```

####Text
`text_index` - array of indexes used for textual analysis


####Date
`date_index` - nested indexes with options used for temporal analysis

* `unknown` - field with a string of text that needs parsing into a date
* `timestamp` - either second or milisecond unix timestamp 
* `day` - integer day of the month
* `month` - integer month of the year
* `year` - year 



####Geo
`geo_index` - nested indexes that contain fields with varying geographical information

* `unknown` - geographical string we attempt to parse
* `global`  - country  
* `subglobal` - state/providence 
* `local` - city 
* `sublocal` - street name 
* `postcode` - postal code 
* `latitude` - latitude column 
* `logintude` - longingtude column 
* `latitude_longitude` - column with lat/long in that order 
* `longitude_latitude` - column with long/lat in that order 
* `ip_address` - IP address column  
* `phone_number` - phone # 


## Return Data
To return data, using the url parameters from the [initial data request](#start-date-stream).

Using the request schema, host and port from the port request along with the url parameters we can construct the return POST requests. The url will look like this `https://alphaapi.stratifd.com/streams/5ace79e3b2308449f3608159/documents`.

The following headers will looke like this

Using the following headers

* `Content-Type : 'application/json'`
* `X-Taste-Tmp : ACCESS_TOKEN`
* `X-Taste-App : SUBDOMAIN`

```python
from requests import post
headers = {
    'Content-Type' : 'application/json',
    'X-Taste-Tmp' : ACCESS_TOKEN,
    'X-Taste-App' : SUBDOMAIN
}
url = "https://alphaapi.stratifyd.com/streams/%s/documents" % $STREAM_ID
body = [doc1, doc2, doc3] 
req = post(url, headers=headers, data=body)
```



<aside class="warning">
If status code of post request isn't <code>200</code> please end the stream.
</aside>


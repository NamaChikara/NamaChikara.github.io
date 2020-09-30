---
layout: post
title: Spotify Playlist ETL 
date: 2020-08-15
description: Playlist Recommendation - Part 1
---



### High-Level Sketch

There will be a separate class for each step of the ETL. This way the scope of each class is well defined and complexity can be managed. The Extract class will use a Spotify API token to obtain the full playlist data. Next, the Transform class will wrangle the data into a data frame of track details. This data frame will be saved in an AWS S3 bucket via the Load class.

```
class Extract:

  def get_api_token():
    # obtain API token for Spotify API

  def get_playlist_data():
    # use API token to obtain arbitrary playlist data from Spotify

class Transform:

  def get_tracks_from_playlist():
    # extract track details from nested JSON of playlist data

class Load:

  def load():
    # load dataframe of playlist tracks to AWS S3 bucket


```

### Extract

#### Authorize app to fetch data from Spotify

There are four authorization flows in Spotify's documentation, three of which are for accessing specific users' data. The [client credentials flow](https://developer.spotify.com/documentation/general/guides/authorization-guide/#client-credentials-flow) is only for accessing public Spotify data. This is the flow we're going to use because it is the simplest and we don't need specific users' data.  Two things are required: Spotify API credentials and a method for sending a POST request to the `/api/token` endpoint of the Accounts service.

##### Obtain Spotify API credentials

A Spotify account is required for this step; Spotify has a free tier account that doesn't require any payment information. After you've created an account follow the steps to [Register Your App](https://developer.spotify.com/documentation/general/guides/app-settings/#register-your-app) and copy down the Client ID and Client Secret. There is no need to whitelist a redirect URI.

##### Implement a method for requesting an API token

#### Fetch arbitrary playlist data from Spotify

### Transform

#### Wrangle track data from the full playlist object

https://developer.spotify.com/documentation/web-api/reference/object-model/#playlist-object-full

### Load

## Next Steps

We can now extract arbitrary playlist data from Spotify and upload its tracks to an S3 bucket. Our ETL classes are neatly organized, but there is more work to do before we can "productionize" it. In the next post, I'll be going over the process of containerizing the ETL with Docker, including attaching AWS and Spotify credentials at run time.

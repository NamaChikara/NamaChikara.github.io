---
layout: post
title: Spotify Playlist ETL
date: 2020-08-15
description: Playlist Recommendation - Part 1
---



### High-Level Sketch

There will be a separate class for each step of the ETL. This way the scope of each class is well defined and complexity can be managed. The Extract class will use a Spotify API token to obtain the full playlist data. Next, the Transform class will wrangle the data into a data frame of track details. Finally, this data frame will be saved in an AWS S3 bucket via the Load class.

```
class Extract:

  def get_api_token():
    # obtain token for Spotify API

  def get_playlist_data():
    # pull arbitrary playlist data from Spotify

class Transform:

  def get_tracks_from_playlist():
    # extract track details from playlist data

class Load:

  def load():
    # load dataframe of playlist tracks to AWS S3 bucket
```
In the course of this project, I also implemented methods for pulling playlist and artist metadata from the Spotify API. These additional data sets round out the playlist track data with features such as the number of playlist followers and artist genre. My full class definitions can be found hereNEEDTOLINK.

### Extract

There are four authorization flows in Spotify's documentation, three of which are for accessing specific users' data. The [client credentials flow](https://developer.spotify.com/documentation/general/guides/authorization-guide/#client-credentials-flow) is only for accessing public Spotify data. This is the flow we're going to use because it is the simplest and we don't need specific users' data.  Two things are required: Spotify API credentials and a method for sending a POST request to the `/api/token` endpoint of the Accounts service.

#### Obtain Spotify API credentials

A Spotify account is required for this step; Spotify has a free tier account that doesn't require any payment information. After you've created an account follow the steps to [Register Your App](https://developer.spotify.com/documentation/general/guides/app-settings/#register-your-app) and copy down the Client ID and Client Secret.

We want to keep these values out of our code base, but we also want the Extract class to have access to them. An easy way to do this is to create a file `config.py` with a dictionary containing the Client ID and Secret:
```
# config.py

spotify_creds = {
  "client_id": <your_client_id_here>,
  "client_secret": <your_client_secret_here>
}
```
We can then import these values in our ETL script:
```
from config import spotify_creds

class Extract:

  def __init__(self):
    self.client_id = spotify_creds['client_id']
    self.client_secret = spotify_creds['client_secret']
```

#### Method for requesting an API token

The client credentials flow is straightforward to implement using the `requests` library:

```
class Extract:
  ...
  def get_access_token():

    response = requests.post(
      url='https://accounts.spotify.com/api/token',
      data={'grant_type': 'client_credentials'},
      auth=(self.client_id, self.client_secret)
    )

    if response.status_code == 200:
      access_token = response.json()['access_token']
    else:
      access_token = None
      print(f"{response.status_code} Error in API call")

    return access_token
```
Spotify API access tokens are valid for 1 hour. Rather than request a new token each time we need one, we can implement a token cache. A class variable `self.access_token_time` will be used to keep track of when the latest token was retrieved; `self.access_token` will store the most recent token value. If the current time is 3400 seconds past the last access time, we grab a new token:

```
from time import time

class Extract:

  def __init__(self):
    self.access_token = None
    self.access_token_time = None

  def get_access_token():

    now = int(time()) # seconds since epoch

    if self.access_token_time is None or now > self.access_token_time + 3400
      self.access_token_time = now
      # submit POST request for new token
```

#### Fetch playlist data from Spotify

```
class Extract:
  ...
  def get_spotify_data(self, url, params=None):

    response  = requests.get(
      url=url,
      headers={'Authorization': 'Bearer ' + self.get_access_token()},
      params=params
    )
    return response.json()

  def get_playlist_tracks(self, playlist_id):

    url = 'https://api.spotify.com/v1/playlists/{playlist_id}/tracks'.format(playlist_id=playlist_id)

    json_response = self.get_spotify_data(url)
    df = pd.json_normalize(json_response['items'])
    next_url = json_response['next']

    while next_url is not None:
      json_response = self.get_spotify_data(next_url)
      df = df.append(pd.json_normalize(json_response['items']))
      next_url = json_response['next']

    df.insert(0, column='track_no', value=np.arange(len(df)))

    return df
```

#### Putting it all together + documentation

https://stackoverflow.com/questions/838545/div-vertical-scrollbar-show

```
# etl.py

from config import spotify_creds
from time import time

class Extract:
  """Extract various Spotify data via the Spotify API.  API credentials required.

   Attributes
   ----------
   client_id : str
       Spotify client id, obtained by registering an App with Spotify
   client_secret: str
       Spotify client secret, obtained by registering an App with Spotify
   access_token_time : int
       Seconds from epoch to when a Spotify access token was created
   access_token_string : str
       Access token for Spotify API

   Notes
   -----
   * A config module containing a dict named `default` with keys 'client_id' and 'client_secret' containing
   Spotify API credentials is required for this class
   * Spotify Playlist data is currently the only supported API return type.  Extending to other other returns
   will be added as needed
   """

  def __init__(self):
    """Retrieve Spotify API Client ID and Client Secret from config.py module
    """

    self.client_id = spotify_creds['client_id']
    self.client_secret = spotify_creds['client_secret']
    self.access_token = None
    self.access_token_time = None

  def get_access_token():
    """Retrieves a new Spotify API access token every 3400 seconds

    Spotify API access tokens are valid for 1 hr (3600 seconds).  An access token is stored in
    `self.access_token_string` until the difference between the current time and
    `self.access_token_time` exceeds 3400 seconds.

    Returns
    -------
    str
        API access token
    """

    now = int(time()) # seconds since epoch

    # Check if the token needs to be refreshed.
    if self.access_token_time is None or now > self.access_token_time + 3400
      self.access_token_time = now
      response = requests.post(
        url='https://accounts.spotify.com/api/token',
        data={'grant_type': 'client_credentials'},
        auth=(self.client_id, self.client_secret)
      )
      if response.status_code == 200:
        self.access_token = response.json()['access_token']
      else:
        self.access_token = None
        print(f"{response.status_code} Error in API call")

    return self.access_token
```

#### Fetch arbitrary playlist data from Spotify

### Transform

#### Wrangle track data from the full playlist object

https://developer.spotify.com/documentation/web-api/reference/object-model/#playlist-object-full

### Load

## Next Steps

We can now extract arbitrary playlist data from Spotify and upload its tracks to an S3 bucket. Our ETL classes are neatly organized, but there is more work to do before we can "productionize" it. In the next post, I'll be going over the process of containerizing the ETL with Docker, including attaching AWS and Spotify credentials at run time.

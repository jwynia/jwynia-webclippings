# j soma - Building Spotify playlists based on vibes using LangChain and GPT
[j soma - Building Spotify playlists based on vibes using LangChain and GPT](https://jonathansoma.com/words/spotify-langchain-chatgpt.html) 

     
    j soma - Building Spotify playlists based on vibes using LangChain and GPT                

Building Spotify playlists based on vibes using LangChain and GPT
=================================================================

How to run arbitrary libraries with LangChain to integrate Spotify with GPT, with a nice introduction to APIChain, PALChain and SequentialChain.

Author

Jonathan Soma

Published

March 27, 2023

Contents
--------

*   [The universal need for on-demand, vibes-based Spotify playlists](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#the-universal-need-for-on-demand-vibes-based-spotify-playlists)
*   [The Playlist](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#the-playlist)
*   [Preparation and setup](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#preparation-and-setup)
    *   [Getting my API keys](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#getting-my-api-keys)
    *   [Accessing OpenAI/GPT](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#accessing-openaigpt)
    *   [Accessing Spotify](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#accessing-spotify)
        *   [Step one: Find the artist URI](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#step-one-find-the-artist-uri)
        *   [Step two: Find the tracks URIs](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#step-two-find-the-tracks-uris)
        *   [Step three: Find the audio features](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#step-three-find-the-audio-features)
*   [Existing chains](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#existing-chains)
    *   [How the APIChain works](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#how-the-apichain-works)
        *   [Why this doesn’t work for our Spotify use case](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#why-this-doesnt-work-for-our-spotify-use-case)
    *   [How the PALChain works](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#how-the-palchain-works)
        *   [Why this doesn’t work for our Spotify use case](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#why-this-doesnt-work-for-our-spotify-use-case-1)
*   [Our process](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#our-process)
    *   [The Spotipy code prompt](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#the-spotipy-code-prompt)
    *   [PALChain for data access](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#palchain-for-data-access)
    *   [LLMChain for cleanup](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#llmchain-for-cleanup)
*   [Connecting the chains](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#connecting-the-chains)
*   [Reflections](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#reflections)

Hi, I’m Soma! You can find me on email at [jonathan.soma@gmail.com](mailto:jonathan.soma@gmail.com), on Twitter at [@dangerscarf](https://twitter.com/dangerscarf), or maybe even on [this newsletter I’ve never sent](https://tinyletter.com/jsoma).

The universal need for on-demand, vibes-based Spotify playlists
===============================================================

The top track or two for practically any band on Spotify is a _slow one_. The Cars have Drive, Green Day has Boulevard of Broken Dreams, Blink 182 has I Miss You. They put me to sleep! Instead of screaming at Alexa to “play The Cars” I want to say “play The Cars _but none of those boring slow songs_.”

Spotify actually knows the diference between a slow song and an upbeat one, although somewhat secretly – they don’t reveal it in the app, but their API includes data for songs like energy level and danceability. **Can we combine this information with [LangChain](https://langchain.readthedocs.io/) and GPT to talk to Spotify through natural language?**

Trying to get this to work, you hit a wall pretty quickly: while LangChain supports APIs, the Spotify API is an awful complex OAuth2 beast that doesn’t fit the existing examples. The [Spotipy library](https://spotipy.readthedocs.io/) is a lot easier to use, but as of this moment it isn’t super-simple to run arbitrary code through LangChain.

_But let’s make it happen anyway!_ In this walkthrough, we’ll look at:

1.  Accessing Spotify data through the Spotipy Python library
2.  How LangChain’s `APIChain` (API access) and `PALChain` (Python execution) chains are built
3.  Combining aspects both to allow LangChain/GPT to use arbitrary Python packages
4.  Putting it all together to let you, GPT and Spotify and have a little chat about your musical tastes

If you’re a LangChain pro and are just looking for code to run, you can can skip all these words [and just visit the GitHub repo](https://github.com/jsoma/spotify-langchain-gpt)

If you are a less technical person, just relax while you’re reading the next few sections! They’re the “why” for how we end up tackling our problem.

The Playlist
============

To get the disappointment and/or excitement out of the way early on, here’s the playlist we wind up with at the end:

The prompt was “Give me a list of Cars, Blink 182 and Sum 41 songs that are upbeat, loud and fun. Make sure the songs are popular enough for me to have heard of them.”

Speaking of The Rock Show, but there’s nothing better than listening to it while rolling up to the [NJ Mineral, Fossil, Gem & Jewelry Show](https://nj.show/). It’s coming up!!

Preparation and setup
=====================

Getting my API keys[](#getting-my-api-keys)
-------------------------------------------

Both GPT and Spotify require me to prove my identity using **API keys**. If you had my keys you’d be able to impersonate me, talk to my chatbots, and make a bunch of awful playlists – we don’t want _any_ of those happening. Instead of putting the API keys in my notebook, I’m using [dotenv-python](https://youtu.be/YdgIWTYQ69A) to keep them nice and secret. I recommend it!

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb1-1)%load_ext dotenv
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb1-2)%dotenv`
```

Accessing OpenAI/GPT[](#accessing-openaigpt)
--------------------------------------------

To access GPT-3.5-turbo, we’re use to use a LangChain [chain](https://langchain.readthedocs.io/en/latest/modules/chains/getting_started.html).

It’s a little more complicated than [when we were talking to fairy tales](https://jonathansoma.com/words/multi-language-qa-gpt.html), but the former method of using a plain `OpenAI` object is being deprecated in favor of `ChatOpenAI`.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb2-1)from langchain.chat_models import ChatOpenAI
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb2-2)from langchain.prompts import PromptTemplate
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb2-3)from langchain import LLMChain
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb2-5)llm = ChatOpenAI(model_name='gpt-3.5-turbo')`
```

Let’s test it out with a sample `PromptTemplate` about energetic songs.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb3-1)prompt = PromptTemplate(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb3-2)    input_variables=["question"],
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb3-3)    template="{question}",
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb3-4))
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb3-6)chain = LLMChain(llm=llm, prompt=prompt)`
```

Now that it’s assembled, let’s use it.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb4-1)response = chain.run("The Promise Ring")
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb4-2)print(response)`
```

```
Some of the more energetic songs by The Promise Ring include:

1. "Is This Thing On?"
2. "Emergency! Emergency!"
3. "Red & Blue Jeans"
4. "Happiness is All the Rage"
5. "Make Me a Chevy"
6. "Jersey Shore"
7. "B Is for Bethlehem"
8. "Why Did We Ever Meet?"
9. "Stop Playing Guitar"
10. "Pink Chimneys"
```

Yes, that’s an answer, but **it isn’t good enough**. Asking GPT for energetic songs works, I want to at least _pretend_ that we’re basing this on science! You never know if record labels from the late 90’s are funnelling money to OpenAI to bias the results.

Luckily, Spotify has that information: their API includes [access a track’s audio features](https://developer.spotify.com/documentation/web-api/reference/#/operations/get-several-audio-features) including loudness, danceability and energy! With that in mind, our goal is now to **create a way for GPT and Spotify to interface so that we can leverage that information when building our playlist.**

Accessing Spotify[](#accessing-spotify)
---------------------------------------

We’re going to use [the Spotipy Python library](https://spotipy.readthedocs.io/en/2.22.1/) to access Spotify. It handles all of the OAuth login, the refreshing of tokens (they’re only good for 5 minutes!), and everything of that ilk.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-1)import os
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-2)import spotipy
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-3)from spotipy.oauth2 import SpotifyClientCredentials
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-5)auth = SpotifyClientCredentials(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-6)    client_id=os.environ['SPOTIPY_CLIENT_ID'],
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-7)    client_secret=os.environ['SPOTIPY_CLIENT_SECRET']
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-8))
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb6-9)sp = spotipy.Spotify(auth_manager=auth)`
```

If we want songs by the band Wet Leg, we can’t just say “give me Wet Legs top tracks.” Instead, we need to find Wet Leg’s `URI` \- uniform resource indicator, Spotify’s cataloguing ID – and then use _that_ to get the top tracks. It’s important to note that **it’s almost always a multi-step process to get anything useful from Spotify**.

For example, let’s say we want to filter for energetic songs, which means we want a track’s audio features. If you wanted the audio features for a track but only know the band name, the process looks like what is outlined below:

### Step one: Find the artist URI[](#step-one-find-the-artist-uri)

We’ll use Spotify’s search to try to find the artist Wet Leg, and then assume the first result is the right one.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb7-1)response = sp.search('Wet Leg', type='artist')
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb7-2)uri = response['artists']['items'][0]['uri']
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb7-3)uri`
```

```
'spotify:artist:2TwOrUcYnAlIiKmVQkkoSZ'
```

### Step two: Find the tracks URIs[](#step-two-find-the-tracks-uris)

We’ll then use the artist’s URI to find some top tracks from that artist (there’s actually an endpoint for top tracks!).

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb9-1)response = sp.artist_top_tracks(uri)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb9-2)top_five = response['tracks'][:5]
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb9-4)for track in top_five:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb9-5)    print(track['popularity'], track['name'], track['uri'])`
```

```
70 Wet Dream spotify:track:260Ub1Yuj4CobdISTOBvM9
66 Chaise Longue spotify:track:0nys6GusuHnjSYLW0PYYb7
63 Being In Love spotify:track:4VBE0mwU8Nmm8hiqfCe4Ve
62 Angelica spotify:track:3EwTIu5qka2l5ZekB0b6QC
60 Ur Mum spotify:track:4ug5wsIcbAPBun8TCKn2t6
```

### Step three: Find the audio features[](#step-three-find-the-audio-features)

Instead of coming with the track results, the danceability and all of those scores are in a completely different endpoint! So we’ll now use the track URIs to access the audio features.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb11-1)import pandas as pd
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb11-3)uris = [track['uri'] for track in top_five]
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb11-4)audio_features = sp.audio_features(uris)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb11-5)pd.DataFrame(audio_features)`
```

|  | danceability | energy | key | loudness | mode | speechiness | acousticness | instrumentalness | liveness | valence | tempo | type | id | uri | track_href | analysis_url | duration_ms | time_signature |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0 | 0.721 | 0.701 | 2 | -5.941 | 1 | 0.0306 | 0.000927 | 0.026000 | 0.234 | 0.892 | 130.091 | audio_features | 260Ub1Yuj4CobdISTOBvM9 | spotify:track:260Ub1Yuj4CobdISTOBvM9 | https://api.spotify.com/v1/tracks/260Ub1Yuj4Co... | https://api.spotify.com/v1/audio-analysis/260U... | 140080 | 3 |
| 1 | 0.684 | 0.749 | 7 | -6.565 | 1 | 0.0600 | 0.001350 | 0.111000 | 0.141 | 0.935 | 160.021 | audio_features | 0nys6GusuHnjSYLW0PYYb7 | spotify:track:0nys6GusuHnjSYLW0PYYb7 | https://api.spotify.com/v1/tracks/0nys6GusuHnj... | https://api.spotify.com/v1/audio-analysis/0nys... | 196905 | 4 |
| 2 | 0.716 | 0.687 | 9 | -4.940 | 0 | 0.0342 | 0.009220 | 0.110000 | 0.123 | 0.342 | 126.030 | audio_features | 4VBE0mwU8Nmm8hiqfCe4Ve | spotify:track:4VBE0mwU8Nmm8hiqfCe4Ve | https://api.spotify.com/v1/tracks/4VBE0mwU8Nmm... | https://api.spotify.com/v1/audio-analysis/4VBE... | 122467 | 4 |
| 3 | 0.491 | 0.870 | 0 | -5.138 | 1 | 0.0393 | 0.000141 | 0.000729 | 0.368 | 0.314 | 131.989 | audio_features | 3EwTIu5qka2l5ZekB0b6QC | spotify:track:3EwTIu5qka2l5ZekB0b6QC | https://api.spotify.com/v1/tracks/3EwTIu5qka2l... | https://api.spotify.com/v1/audio-analysis/3EwT... | 232320 | 4 |
| 4 | 0.685 | 0.720 | 4 | -5.553 | 1 | 0.0280 | 0.007020 | 0.275000 | 0.425 | 0.554 | 133.016 | audio_features | 4ug5wsIcbAPBun8TCKn2t6 | spotify:track:4ug5wsIcbAPBun8TCKn2t6 | https://api.spotify.com/v1/tracks/4ug5wsIcbAPB... | https://api.spotify.com/v1/audio-analysis/4ug5... | 201253 | 4 |

So what I’m saying is: we can’t just hit one endpoint and run away. **This is a lot of work!**

Existing chains
===============

When attempting to talk to the Spotify API through Spotipy, there are two obvious answers from the LangChain documentation that might come to mind:

*   `APIChain`, which is used for talking to APIs
*   `PALChain`, which is used for running Python code

Now we’ll look at the shortcomings of each and why we need to create our own custom chain.

How the APIChain works[](#how-the-apichain-works)
-------------------------------------------------

An APIChain can be used to access an API! This is a slightly adapted version of the [APIChain example from the docs](https://langchain.readthedocs.io/en/latest/modules/chains/examples/api.html).

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb12-1)from langchain.chains import APIChain
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb12-2)from langchain.chains.api import open_meteo_docs
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb12-4)chain_new = APIChain.from_llm_and_api_docs(llm, open_meteo_docs.OPEN_METEO_DOCS, verbose=True)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb12-5)chain_new.run('What is the weather like right now in Munich, Germany in degrees Farenheit? Do not include a forecast.')`
```

```


> Entering new APIChain chain...
https://api.open-meteo.com/v1/forecast?latitude=48.137154&longitude=11.576124&current_weather=true&temperature_unit=fahrenheit
{"latitude":48.14,"longitude":11.58,"generationtime_ms":0.1989603042602539,"utc_offset_seconds":0,"timezone":"GMT","timezone_abbreviation":"GMT","elevation":526.0,"current_weather":{"temperature":50.0,"windspeed":16.1,"winddirection":254.0,"weathercode":3,"time":"2023-03-26T16:00"}}

> Finished chain.
```

```
'The weather in Munich, Germany right now is 50 degrees Fahrenheit.'
```

An important thing to take note of here is `open_meteo_docs.OPEN_METEO_DOCS`: along with our prompt and an llm, we’re also sending the documentation for the Open-Meteo API. It looks like this:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb15-1)print(open_meteo_docs.OPEN_METEO_DOCS[:1000])`
```

```
BASE URL: https://api.open-meteo.com/

API Documentation
The API endpoint /v1/forecast accepts a geographical coordinate, a list of weather variables and responds with a JSON hourly weather forecast for 7 days. Time always starts at 0:00 today and contains 168 hours. All URL parameters are listed below:

Parameter   Format  Required    Default Description
latitude, longitude Floating point  Yes     Geographical WGS84 coordinate of the location
hourly  String array    No      A list of weather variables which should be returned. Values can be comma separated, or multiple &hourly= parameter in the URL can be used.
daily   String array    No      A list of daily weather variable aggregations which should be returned. Values can be comma separated, or multiple &daily= parameter in the URL can be used. If daily weather variables are specified, parameter timezone is required.
current_weather Bool    No  false   Include current weather conditions in the JSON output.
temperature_unit    String  No  celsius If fahrenheit is set, al
```

But what is the chain doing with the Open-Meteo docs? If we [dig around in the source code](https://github.com/hwchase17/langchain/blob/master/langchain/chains/api/base.py) we can find a few lines of code that get into the details:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb17-1)get_request_chain = LLMChain(llm=llm, prompt=api_url_prompt)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb17-2)requests_wrapper = RequestsWrapper(headers=headers)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb17-3)get_answer_chain = LLMChain(llm=llm, prompt=api_response_prompt)`
```

These are used in a three-step process:

1.  Get the API URL
2.  Use the API URL to get the data
3.  Process the data into an answer to the question

**The first step** builds an `LLMChain` to talk to GPT. LangChain then provides the API documentation to GPT, and asks it to determine the API endpoint to visit.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-1)"""You are given the below API Documentation:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-3) {api_docs}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-5)Using this documentation, generate the full API url to call for answering the user question.
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-6)You should build the API url in order to get a response that is as short as possible, while still getting the necessary information to answer the question. Pay attention to deliberately exclude any unnecessary pieces of data in the API call.
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-8)Question:{question}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb18-9)API url:"""`
```

**The second step** builds a `RequestsWrapper` to access the API URL and returns the response. But it isn’t a human-readable response to our question yet, it’s almost always going to be a bunch of JSON.

**The final step** uses another `LLMChain` to talk to GPT again: LangChain sends the API response to GPT and asks for a human-readable summary to answer the question.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb19-1)"""Here is the response from the API:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb19-3){api_response}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb19-5)Summarize this response to answer the original question.
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb19-6)Summary:"""`
```

### Why this doesn’t work for our Spotify use case[](#why-this-doesnt-work-for-our-spotify-use-case)

Even though we want to talk to an API, **we want to talk to an API through the Spotipy Python library, not a series of URLs**. Since the `APIChain` is based around making an actual request to somewhere on the internet, this isn’t going to work for us.

If the Spotify API were a nice simple REST API we could just feed `APIChain` the documentation, but that isn’t the case.

How the PALChain works[](#how-the-palchain-works)
-------------------------------------------------

A PALChain can be used to create and run arbitrary Python code! This is [the PALChain example from the docs](https://langchain.readthedocs.io/en/latest/modules/chains/examples/pal.html).

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb20-1)from langchain.chains import PALChain
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb20-3)pal_chain = PALChain.from_math_prompt(llm, verbose=True)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb20-4)question = "Jan has three times the number of pets as Marcia. Marcia has two more pets than Cindy. If Cindy has four pets, how many total pets do the three have?"
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb20-5)pal_chain.run(question)`
```

```


> Entering new PALChain chain...
def solution():
    """Jan has three times the number of pets as Marcia. Marcia has two more pets than Cindy. If Cindy has four pets, how many total pets do the three have?"""
    cindy_pets = 4
    marcia_pets = cindy_pets + 2
    jan_pets = marcia_pets * 3
    total_pets = cindy_pets + marcia_pets + jan_pets
    result = total_pets
    return result

> Finished chain.
```

```
'28'
```

If we look at [the code for the math chain’s prompt](https://github.com/hwchase17/langchain/blob/master/langchain/chains/pal/math_prompt.py) it’s _very_ long. Here’s a portion of it:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-1)from langchain.prompts.prompt import PromptTemplate
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-3)template = (
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-4)    '''
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-5)Q: Olivia has $23. She bought five bagels for $3 each. How much money does she have left?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-7)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-10)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-11) """Olivia has $23. She bought five bagels for $3 each. How much money does she have left?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-12) money_initial = 23
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-13) bagels = 5
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-14) bagel_cost = 3
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-15) money_spent = bagels * bagel_cost
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-16) money_left = money_initial - money_spent
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-17) result = money_left
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-18) return result
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-20)Q: There are 15 trees in the grove. Grove workers will plant trees in the grove today. After they are done, there will be 21 trees. How many trees did the grove workers plant today?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-22)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-25)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-26) """There are 15 trees in the grove. Grove workers will plant trees in the grove today. After they are done, there will be 21 trees. How many trees did the grove workers plant today?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-27) trees_initial = 15
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-28) trees_after = 21
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-29) trees_added = trees_after - trees_initial
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-30) result = trees_added
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-31) return result
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-37)Q: {question}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-39)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-40)'''.strip()
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-41)    + "\n\n\n"
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-42))
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb23-43)MATH_PROMPT = PromptTemplate(input_variables=["question"], template=template)`
```

That prompt only gives you Python code, though, not the actual result! To see what happens with the result we need to [check the PALChain code itself](https://github.com/hwchase17/langchain/blob/master/langchain/chains/pal/base.py#L57-L68), lightly edited for clarity:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-1)def _call(self, inputs: Dict[str, str]) -> Dict[str, str]:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-2)    llm_chain = LLMChain(llm=self.llm, prompt=self.prompt)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-3)    code = llm_chain.predict(stop=[self.stop], **inputs)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-4)    repl = PythonREPL(_globals=self.python_globals, _locals=self.python_locals)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-5)    res = repl.run(code + f"\n{self.get_answer_expr}")
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-6)    output = {self.output_key: res.strip()}
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb24-7)    return output`
```

The code looks a little wild, but the process is pretty simple:

1.  Use the LLM and the prompt to generate some code
2.  Create a Python REPL to run the generated code, sending along some global and local variables
3.  Run the code contained in `self.get_answer_expr` to get the result of the solution.

In this case, the `get_answer_expr` is [`print(solution())`](https://github.com/hwchase17/langchain/blob/master/langchain/chains/pal/base.py#L77). Our prompt insists the answer be put in a function called `solution`, so this is how we call the function and obtain the result.

### Why this doesn’t work for our Spotify use case[](#why-this-doesnt-work-for-our-spotify-use-case-1)

With a little work, we actually _can_ make it work! It just needs a little extra effort and inspiration from `APIChain`.

Our process
===========

Our process is going to take two steps:

1.  Use a `PALChain` to write and run the Spotipy code
2.  Use an `LLMChain`to clean it up and provide an answer

The separte steps of “data acquisition first, then clean up for the presentation” is inspired by our friend `APIChain`. In theory we might even be able to split this into three steps – develop the code, run the code, analyze the results – but let’s keep it to two for now.

The Spotipy code prompt[](#the-spotipy-code-prompt)
---------------------------------------------------

How do we get GPT to write Spotipy code for us? It’s similar to the API example – giving the documentation to GPT along with our question – but in this case we wouldn’t use Spotify’s API documentation, we’d use documentation for the Spotipy library.

While we could try feeding GPT [the entire documentation page for Spotipy](https://spotipy.readthedocs.io/en/2.22.1/), it’s too long to do it all at once. We _could_ chunk it and feed it into a reference database that is selectively queried for [relevant content](https://jonathansoma.com/words/multi-language-qa-gpt.html)… but that’s just too much work. We want something simple.

Instead, we’re going to go the lazy route: **the Spotify library has been around for _ages_ and GPT already knows how it works, so we’ll just rely on its in-built knowledge.** We just need to provide a few examples of how we like to work with the library and what we need returned, and GPT will follow our lead.

Here’s our prompt for generating Spotipy code to access the Spotify API:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-1)from langchain.prompts.prompt import PromptTemplate
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-4)SPOTIPY_PROMPT_TEMPLATE = (
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-5)    '''
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-6)API LIMITATIONS TO NOTE
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-7)* When requesting track information, the limit is 50 at a time
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-8)* When requesting audio features, the limit is 100 at a time
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-9)* When selecting multiple artists, the limit is 50 at a time
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-10)* When asking for recommendations, the limit is 100 at a time
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-11)=====
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-13)Q: What albums has the band Green Day made?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-15)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-18)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-19) """What albums has the band Green Day made?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-20) search_results = sp.search(q='Green Day', type='artist')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-21) uri = search_results['artists']['items'][0]['uri']
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-22) albums = sp.artist_albums(green_day_uri, album_type='album')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-23) return albums
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-28)Q: Who are some musicians similar to Fiona Apple?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-30)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-33)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-34) """Who are some musicians similar to Fiona Apple?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-35) search_results = sp.search(q='Fiona Apple', type='artist')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-36) uri = search_results['artists']['items'][0].get('uri')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-37) artists = sp.artist_related_artists(uri)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-38) return artists
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-42)Q: Tell me what songs by The Promise Ring sound like
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-44)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-47)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-48) """Tell me what songs by The Promise Ring sound like?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-49) search_results = sp.search(q='The Promise Ring', type='artist')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-50) uri = search_results['artists']['items'][0].get('uri')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-51) tracks = sp.artist_top_tracks(uri)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-52) track_uris = [track.get('uri') for track in tracks['tracks']]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-53) audio_details = sp.audio_features(track_uris)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-54) return audio_details
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-58)Q: Get me the URI for the album The Colour And The Shape
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-60)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-63)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-64) """Get me the URI for the album The Colour And The Shape"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-65) search_results = sp.search(q='The Colour And The Shape', type='album')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-66) uri = search_results['albums']['items'][0].get('uri')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-67) return uri
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-71)Q: What are the first three songs on Diet Cig's Over Easy?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-73)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-76)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-77) """What are the first three songs on Diet Cig's Over Easy?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-78) # Get the URI for the album
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-79) search_results = sp.search(q='Diet Cig Over Easy', type='album')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-80) album = search_results['albums']['items'][0]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-81) album_uri = album['uri']
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-82) # Get the album tracks
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-83) album_tracks = sp.album_tracks(album_uri)['items']
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-84) # Sort the tracks by duration
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-85) first_three = album_tracks[:3]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-86) tracks = []
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-87) # Only include relevant fields
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-88) for i, track in enumerate(first_three):
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-89) # track['album'] does NOT work with sp.album_tracks
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-90) # you need to use album['name'] instead
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-91) tracks.append({{
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-92) 'position': i+1,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-93) 'song_name': track.get('name'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-94) 'song_uri': track['artists'][0].get('uri'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-95) 'artist_uri': track['artists'][0].get('uri'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-96) 'album_uri': album.get('uri'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-97) 'album_name': album.get('name')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-98)  }})
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-99) return tracks
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-102)Q: What are the thirty most danceable songs by Metallica?
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-104)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-107)def solution():
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-108) """What are most danceable songs by Metallica?"""
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-109) search_results = sp.search(q='Metallica', type='artist')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-110) uri = search_results['artists']['items'][0]['uri']
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-111) albums = sp.artist_albums(uri, album_type='album')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-112) album_uris = [album['uri'] for album in albums['items']]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-113) tracks = []
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-114) for album_uri in album_uris:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-115) album_tracks = sp.album_tracks(album_uri)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-116) tracks.extend(album_tracks['items'])
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-117) track_uris = [track['uri'] for track in tracks]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-118) danceable_tracks = []
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-119) # You can only have 100 at a time
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-120) for i in range(0, len(track_uris), 100):
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-121) subset_track_uris = track_uris[i:i+100]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-122) audio_details = sp.audio_features(subset_track_uris)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-123) for j, details in enumerate(audio_details):
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-124) if details['danceability'] > 0.7:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-125) track = tracks[i+j]
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-126) danceable_tracks.append({{
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-127) 'song': track.get('name')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-128) 'album': track.get('album').get('name')
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-129) 'danceability': details.get('danceability'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-130) 'tempo': details.get('tempo'),
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-131)  }})
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-132) # Be sure to add the audio details to the track
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-133) danceable_tracks.append(track)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-134) return danceable_tracks
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-138)Q: {question}. Return a list or dictionary, only including the fields necessary to answer the question, including relevant scores and the uris to the albums/songs/artists mentioned. Only return the data – if the prompt asks for a format such as markdown or a simple string, ignore it: you are only meant to provide the information, not the formatting. A later step in the process will convert the data into the new format (table, sentence, etc).
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-140)# solution in Python:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-141)'''.strip()
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-142)    + "\n\n\n"
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-143))
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb25-145)SPOTIPY_PROMPT = PromptTemplate(input_variables=["question"], template=SPOTIPY_PROMPT_TEMPLATE)`
```

PALChain for data access[](#palchain-for-data-access)
-----------------------------------------------------

The PALChain example from the docs [makes it look so simple](https://python.langchain.com/en/latest/modules/chains/examples/pal.html):

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb26-1)pal_chain = PALChain.from_math_prompt(llm, verbose=True)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb26-2)question = "Jan has three times the number of pets as Marcia. Marcia has two more pets than Cindy. If Cindy has four pets, how many total pets do the three have?"
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb26-3)pal_chain.run(question)`
```

But a lot of work is happening behind the scenes! In our case, we’re going to be building a PALChain from scratch instead of relying on a constructor.

There are a couple important additions we make as we initialize the PALChain. First, we need to provide our initialized and authenticated Spotipy `sp` instance so the PythonREPL can access Spotipy. We’ll do this using `python_globals=`.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb27-1)python_globals={
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb27-2)    'sp': sp
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb27-3)},`
```

The next step is wrangling our data. Results from chains come as strings, but the Spotify API returns JSON (or more specifically, a Python dictionary). To nicely convert our dictionary into a string we’ll be using `json.dumps`. The `json` module isn’t included by default, so this requires importing hte json library before we do the conversion.

Both of these steps are squished into the `get_answer_expr` parameter. It’s a bit garish but it works!

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb28-1)get_answer_expr="import json; print(json.dumps(solution()))",`
```

Finally, we’re also adding `return_intermediate_steps=True` to make sure it returns the result of the code running _and_ the code it ran.

This is what it looks like all put together:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-1)from langchain.chains import PALChain
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-3)spotify_chain = PALChain(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-4)    llm=llm,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-5)    prompt=SPOTIPY_PROMPT,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-6)    python_globals={
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-7)        'sp': sp
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-8)    },
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-9)    stop='\n\n\n',
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-10)    verbose=True,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-11)    return_intermediate_steps=True,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-12)    get_answer_expr="import json; print(json.dumps(solution()))",
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb29-13))`
```

It’s complicated enough, but does it work?

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb30-1)spotify_response = spotify_chain({'question': "What are the most popular Bouncing Souls songs?"})
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb30-2)spotify_response['result']`
```

```


> Entering new PALChain chain...
def solution():
    """What are the most popular Bouncing Souls songs?"""
    search_results = sp.search(q='Bouncing Souls', type='artist')
    uri = search_results['artists']['items'][0].get('uri')
    top_tracks = sp.artist_top_tracks(uri)
    top_track_uris = [track.get('uri') for track in top_tracks['tracks']]
    audio_details = sp.audio_features(top_track_uris)
    popular_songs = []
    for i, track in enumerate(top_tracks['tracks']):
        details = audio_details[i]
        popular_songs.append({
            'song_name': track.get('name'),
            'song_uri': track.get('uri'),
            'artist_name': track.get('artists')[0].get('name'),
            'artist_uri': track.get('artists')[0].get('uri'),
            'album_name': track.get('album').get('name'),
            'album_uri': track.get('album').get('uri'),
            'popularity': track.get('popularity'),
            'danceability': details.get('danceability'),
            'energy': details.get('energy'),
            'key': details.get('key'),
            'loudness': details.get('loudness'),
            'mode': details.get('mode'),
            'speechiness': details.get('speechiness'),
            'acousticness': details.get('acousticness'),
            'instrumentalness': details.get('instrumentalness'),
            'liveness': details.get('liveness'),
            'valence': details.get('valence'),
            'tempo': details.get('tempo'),
        })
    return popular_songs[:10] # Return top 10 songs

> Finished chain.
```

```
'[{"song_name": "True Believers", "song_uri": "spotify:track:4fRmFVMd0c1SGfzazBJIM8", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "How I Spent My Summer Vacation", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "popularity": 55, "danceability": 0.237, "energy": 0.981, "key": 0, "loudness": -4.32, "mode": 1, "speechiness": 0.0989, "acousticness": 0.000296, "instrumentalness": 3.81e-05, "liveness": 0.202, "valence": 0.475, "tempo": 98.181}, {"song_name": "Lean On Sheena", "song_uri": "spotify:track:7IR7GUO0dUyUsBp7BfQ3vJ", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Gold Record", "album_uri": "spotify:album:3MF7PvmrMjEXGvA8fP3L6l", "popularity": 51, "danceability": 0.491, "energy": 0.866, "key": 11, "loudness": -4.431, "mode": 1, "speechiness": 0.0583, "acousticness": 0.16, "instrumentalness": 0.000211, "liveness": 0.13, "valence": 0.694, "tempo": 175.969}, {"song_name": "Hopeless Romantic", "song_uri": "spotify:track:180mXjN61yhrKhbY2yQc0E", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Hopeless Romantic", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "popularity": 49, "danceability": 0.243, "energy": 0.981, "key": 4, "loudness": -5.251, "mode": 1, "speechiness": 0.074, "acousticness": 0.000164, "instrumentalness": 1.11e-05, "liveness": 0.207, "valence": 0.216, "tempo": 105.022}, {"song_name": "Manthem", "song_uri": "spotify:track:5pSjxUAwOol5e0TWp1ecHC", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "How I Spent My Summer Vacation", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "popularity": 46, "danceability": 0.524, "energy": 0.986, "key": 2, "loudness": -2.865, "mode": 1, "speechiness": 0.0634, "acousticness": 9.44e-05, "instrumentalness": 0.000214, "liveness": 0.0772, "valence": 0.724, "tempo": 94.348}, {"song_name": "Sing Along Forever", "song_uri": "spotify:track:5feYKXxg4HL2APTQGCfAav", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Anchors Aweigh", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "popularity": 46, "danceability": 0.592, "energy": 0.964, "key": 0, "loudness": -3.672, "mode": 1, "speechiness": 0.0817, "acousticness": 0.00912, "instrumentalness": 0, "liveness": 0.27, "valence": 0.585, "tempo": 101.252}, {"song_name": "Say Anything", "song_uri": "spotify:track:06peZfvxR5721oGqHwogha", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Bouncing Souls", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "popularity": 45, "danceability": 0.448, "energy": 0.995, "key": 6, "loudness": -3.111, "mode": 1, "speechiness": 0.0539, "acousticness": 0.00275, "instrumentalness": 0, "liveness": 0.297, "valence": 0.643, "tempo": 101.405}, {"song_name": "Ole", "song_uri": "spotify:track:2McQQA5nCLVL0XvzcxWhFC", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Hopeless Romantic", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "popularity": 43, "danceability": 0.33, "energy": 0.833, "key": 7, "loudness": -6.507, "mode": 1, "speechiness": 0.0768, "acousticness": 0.0491, "instrumentalness": 0, "liveness": 0.687, "valence": 0.553, "tempo": 128.329}, {"song_name": "Ten Stories High", "song_uri": "spotify:track:1t9Y1HGwikUCCo5xCupAnT", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Ten Stories High", "album_uri": "spotify:album:0wdbr46ndnwB1cgZoNzT48", "popularity": 30, "danceability": 0.361, "energy": 0.984, "key": 5, "loudness": -1.913, "mode": 1, "speechiness": 0.0883, "acousticness": 0.000322, "instrumentalness": 0.000873, "liveness": 0.329, "valence": 0.491, "tempo": 198.064}, {"song_name": "Kids and Heroes", "song_uri": "spotify:track:7ru4QA7k7ViuLS9oDtdRBI", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Anchors Aweigh", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "popularity": 43, "danceability": 0.405, "energy": 0.963, "key": 0, "loudness": -5.216, "mode": 1, "speechiness": 0.0697, "acousticness": 0.0127, "instrumentalness": 0.000256, "liveness": 0.289, "valence": 0.198, "tempo": 101.759}, {"song_name": "Kate Is Great", "song_uri": "spotify:track:1VT2wLreLu0l7E4T0JDedh", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Bouncing Souls", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "popularity": 42, "danceability": 0.358, "energy": 0.93, "key": 2, "loudness": -4.726, "mode": 1, "speechiness": 0.164, "acousticness": 0.0822, "instrumentalness": 0, "liveness": 0.0699, "valence": 0.809, "tempo": 175.011}]'
```

Let’s look at the three separate keys the `PALChain` response gives us.

First, the **question**:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb33-1)spotify_response['question']`
```

```
'What are the most popular Bouncing Souls songs?'
```

Second, the **intermediate steps** (the code that it ran):

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb35-1)print(spotify_response['intermediate_steps'])`
```

```
def solution():
    """What are the most popular Bouncing Souls songs?"""
    search_results = sp.search(q='Bouncing Souls', type='artist')
    uri = search_results['artists']['items'][0].get('uri')
    top_tracks = sp.artist_top_tracks(uri)
    top_track_uris = [track.get('uri') for track in top_tracks['tracks']]
    audio_details = sp.audio_features(top_track_uris)
    popular_songs = []
    for i, track in enumerate(top_tracks['tracks']):
        details = audio_details[i]
        popular_songs.append({
            'song_name': track.get('name'),
            'song_uri': track.get('uri'),
            'artist_name': track.get('artists')[0].get('name'),
            'artist_uri': track.get('artists')[0].get('uri'),
            'album_name': track.get('album').get('name'),
            'album_uri': track.get('album').get('uri'),
            'popularity': track.get('popularity'),
            'danceability': details.get('danceability'),
            'energy': details.get('energy'),
            'key': details.get('key'),
            'loudness': details.get('loudness'),
            'mode': details.get('mode'),
            'speechiness': details.get('speechiness'),
            'acousticness': details.get('acousticness'),
            'instrumentalness': details.get('instrumentalness'),
            'liveness': details.get('liveness'),
            'valence': details.get('valence'),
            'tempo': details.get('tempo'),
        })
    return popular_songs[:10] # Return top 10 songs
```

Finally, the **actual response**. In the `PALChain` examples it’s mostly the result of a quick calculation, but this time it’s a whole big mess of JSON:

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb37-1)spotify_response['result']`
```

```
'[{"song_name": "True Believers", "song_uri": "spotify:track:4fRmFVMd0c1SGfzazBJIM8", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "How I Spent My Summer Vacation", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "popularity": 55, "danceability": 0.237, "energy": 0.981, "key": 0, "loudness": -4.32, "mode": 1, "speechiness": 0.0989, "acousticness": 0.000296, "instrumentalness": 3.81e-05, "liveness": 0.202, "valence": 0.475, "tempo": 98.181}, {"song_name": "Lean On Sheena", "song_uri": "spotify:track:7IR7GUO0dUyUsBp7BfQ3vJ", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Gold Record", "album_uri": "spotify:album:3MF7PvmrMjEXGvA8fP3L6l", "popularity": 51, "danceability": 0.491, "energy": 0.866, "key": 11, "loudness": -4.431, "mode": 1, "speechiness": 0.0583, "acousticness": 0.16, "instrumentalness": 0.000211, "liveness": 0.13, "valence": 0.694, "tempo": 175.969}, {"song_name": "Hopeless Romantic", "song_uri": "spotify:track:180mXjN61yhrKhbY2yQc0E", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Hopeless Romantic", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "popularity": 49, "danceability": 0.243, "energy": 0.981, "key": 4, "loudness": -5.251, "mode": 1, "speechiness": 0.074, "acousticness": 0.000164, "instrumentalness": 1.11e-05, "liveness": 0.207, "valence": 0.216, "tempo": 105.022}, {"song_name": "Manthem", "song_uri": "spotify:track:5pSjxUAwOol5e0TWp1ecHC", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "How I Spent My Summer Vacation", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "popularity": 46, "danceability": 0.524, "energy": 0.986, "key": 2, "loudness": -2.865, "mode": 1, "speechiness": 0.0634, "acousticness": 9.44e-05, "instrumentalness": 0.000214, "liveness": 0.0772, "valence": 0.724, "tempo": 94.348}, {"song_name": "Sing Along Forever", "song_uri": "spotify:track:5feYKXxg4HL2APTQGCfAav", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Anchors Aweigh", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "popularity": 46, "danceability": 0.592, "energy": 0.964, "key": 0, "loudness": -3.672, "mode": 1, "speechiness": 0.0817, "acousticness": 0.00912, "instrumentalness": 0, "liveness": 0.27, "valence": 0.585, "tempo": 101.252}, {"song_name": "Say Anything", "song_uri": "spotify:track:06peZfvxR5721oGqHwogha", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Bouncing Souls", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "popularity": 45, "danceability": 0.448, "energy": 0.995, "key": 6, "loudness": -3.111, "mode": 1, "speechiness": 0.0539, "acousticness": 0.00275, "instrumentalness": 0, "liveness": 0.297, "valence": 0.643, "tempo": 101.405}, {"song_name": "Ole", "song_uri": "spotify:track:2McQQA5nCLVL0XvzcxWhFC", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Hopeless Romantic", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "popularity": 43, "danceability": 0.33, "energy": 0.833, "key": 7, "loudness": -6.507, "mode": 1, "speechiness": 0.0768, "acousticness": 0.0491, "instrumentalness": 0, "liveness": 0.687, "valence": 0.553, "tempo": 128.329}, {"song_name": "Ten Stories High", "song_uri": "spotify:track:1t9Y1HGwikUCCo5xCupAnT", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Ten Stories High", "album_uri": "spotify:album:0wdbr46ndnwB1cgZoNzT48", "popularity": 30, "danceability": 0.361, "energy": 0.984, "key": 5, "loudness": -1.913, "mode": 1, "speechiness": 0.0883, "acousticness": 0.000322, "instrumentalness": 0.000873, "liveness": 0.329, "valence": 0.491, "tempo": 198.064}, {"song_name": "Kids and Heroes", "song_uri": "spotify:track:7ru4QA7k7ViuLS9oDtdRBI", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "Anchors Aweigh", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "popularity": 43, "danceability": 0.405, "energy": 0.963, "key": 0, "loudness": -5.216, "mode": 1, "speechiness": 0.0697, "acousticness": 0.0127, "instrumentalness": 0.000256, "liveness": 0.289, "valence": 0.198, "tempo": 101.759}, {"song_name": "Kate Is Great", "song_uri": "spotify:track:1VT2wLreLu0l7E4T0JDedh", "artist_name": "The Bouncing Souls", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_name": "The Bouncing Souls", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "popularity": 42, "danceability": 0.358, "energy": 0.93, "key": 2, "loudness": -4.726, "mode": 1, "speechiness": 0.164, "acousticness": 0.0822, "instrumentalness": 0, "liveness": 0.0699, "valence": 0.809, "tempo": 175.011}]'
```

**Looks great!** These don’t answer our question, they only provides the data, so we’ll need one more step.

LLMChain for cleanup[](#llmchain-for-cleanup)
---------------------------------------------

This is similar to what happens in the `APIChain`: we have an API response, but we want something a little more human. We’ll use an `LLMChain` to send the JSON to GPT along with our question, then get back a readable response.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-1)RESPONSE_CLEANUP_PROMPT_TEMPLATE = (""" 
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-2)Using this code:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-4)```python
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-5){intermediate_steps}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-6)```
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-8)We got the following output from the Spotify API:
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-10)```json
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-11){result}
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-12)```
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-14)Using the output above as your data source, answer the question {question}. Don't describe the code or process, just answer the question.
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-15)Answer:"""
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-16))
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-18)RESPONSE_CLEANUP_PROMPT = PromptTemplate(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-19)    input_variables=["question", "intermediate_steps", "result"],
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-20)    template=RESPONSE_CLEANUP_PROMPT_TEMPLATE,
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb39-21))`
```

In the prompt above, we’re providing three things to the prompt:

*   The original **question** we want an answer to
*   The **intermediate steps**, which is the actual Python code the `PALChain` created
*   The **result**, the _output_ of the Python code from the `PALChain` (aka the JSON)

We can now use this prompt with an `LLMChain` to turn the JSON into an actual answer.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-1)explainer_chain = LLMChain(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-2)    llm=llm,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-3)    prompt=RESPONSE_CLEANUP_PROMPT,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-4)    verbose=True,
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-5)    output_key='answer'
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb40-6))`
```

Now that we’ve built the structure of the explainer, let’s feed it the previous Spotify response and see what happens.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb41-1)explainer_response = explainer_chain(spotify_response)`
```

```


> Entering new LLMChain chain...
Prompt after formatting:
 
Using this code:

```python
def solution():
    """What are the most popular Bouncing Souls songs?"""
    search_results = sp.search(q='Bouncing Souls', type='artist')
    uri = search_results['artists']['items'][0]['uri']
    top_tracks = sp.artist_top_tracks(uri)
    tracks = []
    for i, track in enumerate(top_tracks['tracks']):
        # Only include relevant fields
        tracks.append({
            'position': i+1,
            'song_name': track.get('name'),
            'song_uri': track.get('uri'),
            'artist_uri': uri,
            'album_uri': track.get('album').get('uri'),
            'album_name': track.get('album').get('name'),
            'popularity': track.get('popularity')
        })
    return tracks
```

We got the following output from the Spotify API:

```json
[{"position": 1, "song_name": "True Believers", "song_uri": "spotify:track:4fRmFVMd0c1SGfzazBJIM8", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "album_name": "How I Spent My Summer Vacation", "popularity": 54}, {"position": 2, "song_name": "Lean On Sheena", "song_uri": "spotify:track:7IR7GUO0dUyUsBp7BfQ3vJ", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:3MF7PvmrMjEXGvA8fP3L6l", "album_name": "The Gold Record", "popularity": 51}, {"position": 3, "song_name": "Hopeless Romantic", "song_uri": "spotify:track:180mXjN61yhrKhbY2yQc0E", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "album_name": "Hopeless Romantic", "popularity": 49}, {"position": 4, "song_name": "Manthem", "song_uri": "spotify:track:5pSjxUAwOol5e0TWp1ecHC", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:64zbLX1ze8N3kcAMX0qq7G", "album_name": "How I Spent My Summer Vacation", "popularity": 46}, {"position": 5, "song_name": "Sing Along Forever", "song_uri": "spotify:track:5feYKXxg4HL2APTQGCfAav", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "album_name": "Anchors Aweigh", "popularity": 46}, {"position": 6, "song_name": "Say Anything", "song_uri": "spotify:track:06peZfvxR5721oGqHwogha", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "album_name": "The Bouncing Souls", "popularity": 45}, {"position": 7, "song_name": "Ole", "song_uri": "spotify:track:2McQQA5nCLVL0XvzcxWhFC", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:56CbFyDsG65LI1Eoh7hsOT", "album_name": "Hopeless Romantic", "popularity": 43}, {"position": 8, "song_name": "Kids and Heroes", "song_uri": "spotify:track:7ru4QA7k7ViuLS9oDtdRBI", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:1xgfRXjCoynPLqtdNu50pR", "album_name": "Anchors Aweigh", "popularity": 42}, {"position": 9, "song_name": "Kate Is Great", "song_uri": "spotify:track:1VT2wLreLu0l7E4T0JDedh", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:7LgICzKkhaLV9Gttn8xM7a", "album_name": "The Bouncing Souls", "popularity": 41}, {"position": 10, "song_name": "Ten Stories High", "song_uri": "spotify:track:0Wz9RJySVFtUTFQk8sjRBv", "artist_uri": "spotify:artist:3mvTAjG7rcyk7DQzLwauzV", "album_uri": "spotify:album:5xEwUAv3WJiDtHScEPliQl", "album_name": "Ten Stories High", "popularity": 40}]
```

Using the output above as your data source, answer the question What are the most popular Bouncing Souls songs?. Don't describe the code or process, just answer the question.
Answer:

> Finished chain.
```

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb43-1)print(explainer_response['answer'])`
```

```
The most popular Bouncing Souls songs, based on the provided data, are:

1. True Believers
2. Lean On Sheena
3. Hopeless Romantic
4. Manthem
5. Sing Along Forever
6. Say Anything
7. Ole
8. Kids and Heroes
9. Kate Is Great
10. Ten Stories High
```

I completely disagree with everyone’s taste in music, but it’s a perfect response!

Connecting the chains
=====================

Now that we have a working `PALChain` to access the API and an `LLMChain` to process the resulting JSON, let’s connect them together so this no longer takes two steps.

We’re going to use a `SequentialChain` instead of a `SimpleSequentialChain` because the explainer chain needs the question, code _and_ output from the API chain. The SSC only supports one thing being passed along, which would restrict its ability to get all of the necessary information.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-1)from langchain.chains import SequentialChain
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-3)overall_chain = SequentialChain(
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-4)    chains=[spotify_chain, explainer_chain],
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-5)    input_variables=['question'],
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-6)    verbose=True
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb45-7))`
```

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb46-1)overall_response = overall_chain.run("Give me a list of Cars, Blink 182 and Sum 41 songs that are upbeat, loud and fun. Make sure the songs are popular enough for me to have heard of them.")`
```

```


> Entering new SequentialChain chain...


> Entering new PALChain chain...
def solution():
    """Give me a list of Cars, Blink 182 and Sum 41 songs that are upbeat, loud and fun."""
    # Get the URIs for the artists
    artists = ['The Cars', 'Blink-182', 'Sum 41']
    artist_uris = []
    for artist in artists:
        search_results = sp.search(q=artist, type='artist', limit=1)
        if search_results['artists']['total'] > 0:
            artist_uri = search_results['artists']['items'][0]['uri']
            artist_uris.append(artist_uri)
    # Get the top tracks for each artist
    tracks = []
    for artist_uri in artist_uris:
        artist_tracks = sp.artist_top_tracks(artist_uri)
        tracks.extend(artist_tracks['tracks'])
    track_uris = [track['uri'] for track in tracks]
    # Only keep tracks that are upbeat, loud and fun
    upbeat_tracks = []
    for i in range(0, len(track_uris), 100):
        subset_track_uris = track_uris[i:i+100]
        audio_details = sp.audio_features(subset_track_uris)
        for j, details in enumerate(audio_details):
            if details['valence'] > 0.7 and details['energy'] > 0.7 and details['loudness'] > -7:
                track = tracks[i+j]
                upbeat_tracks.append({
                    'song': track.get('name'),
                    'artist': track.get('artists')[0].get('name'),
                    'album': track.get('album').get('name'),
                    'valence': details.get('valence'),
                    'energy': details.get('energy'),
                    'loudness': details.get('loudness'),
                    'song_uri': track.get('uri'),
                    'artist_uri': track.get('artists')[0].get('uri'),
                    'album_uri': track.get('album').get('uri')
                })
    return upbeat_tracks[:30]

> Finished chain.


> Entering new LLMChain chain...
Prompt after formatting:
 
Using this code:

```python
def solution():
    """Give me a list of Cars, Blink 182 and Sum 41 songs that are upbeat, loud and fun."""
    # Get the URIs for the artists
    artists = ['The Cars', 'Blink-182', 'Sum 41']
    artist_uris = []
    for artist in artists:
        search_results = sp.search(q=artist, type='artist', limit=1)
        if search_results['artists']['total'] > 0:
            artist_uri = search_results['artists']['items'][0]['uri']
            artist_uris.append(artist_uri)
    # Get the top tracks for each artist
    tracks = []
    for artist_uri in artist_uris:
        artist_tracks = sp.artist_top_tracks(artist_uri)
        tracks.extend(artist_tracks['tracks'])
    track_uris = [track['uri'] for track in tracks]
    # Only keep tracks that are upbeat, loud and fun
    upbeat_tracks = []
    for i in range(0, len(track_uris), 100):
        subset_track_uris = track_uris[i:i+100]
        audio_details = sp.audio_features(subset_track_uris)
        for j, details in enumerate(audio_details):
            if details['valence'] > 0.7 and details['energy'] > 0.7 and details['loudness'] > -7:
                track = tracks[i+j]
                upbeat_tracks.append({
                    'song': track.get('name'),
                    'artist': track.get('artists')[0].get('name'),
                    'album': track.get('album').get('name'),
                    'valence': details.get('valence'),
                    'energy': details.get('energy'),
                    'loudness': details.get('loudness'),
                    'song_uri': track.get('uri'),
                    'artist_uri': track.get('artists')[0].get('uri'),
                    'album_uri': track.get('album').get('uri')
                })
    return upbeat_tracks[:30]
```

We got the following output from the Spotify API:

```json
[{"song": "You Might Think", "artist": "The Cars", "album": "Heartbeat City (Expanded Edition)", "valence": 0.971, "energy": 0.855, "loudness": -6.031, "song_uri": "spotify:track:35wVRTJlUu2kDkqXFegOKt", "artist_uri": "spotify:artist:6DCIj8jNaNpBz8e5oKFPtp", "album_uri": "spotify:album:7LPfdVDw4uXf9Bw5LQDESf"}, {"song": "Magic - 2017 Remaster", "artist": "The Cars", "album": "Heartbeat City (Expanded Edition)", "valence": 0.964, "energy": 0.794, "loudness": -6.381, "song_uri": "spotify:track:0SAbkr0dS7WK3yJSjaZSZl", "artist_uri": "spotify:artist:6DCIj8jNaNpBz8e5oKFPtp", "album_uri": "spotify:album:7LPfdVDw4uXf9Bw5LQDESf"}, {"song": "Shake It Up - 2017 Remaster", "artist": "The Cars", "album": "Shake It Up (Expanded Edition)", "valence": 0.854, "energy": 0.827, "loudness": -5.738, "song_uri": "spotify:track:3ZBzJbqwV2gQUAe4ofghOo", "artist_uri": "spotify:artist:6DCIj8jNaNpBz8e5oKFPtp", "album_uri": "spotify:album:7KpSpJbHn3SYZIMHKkdO6V"}, {"song": "First Date", "artist": "blink-182", "album": "Take Off Your Pants And Jacket", "valence": 0.882, "energy": 0.928, "loudness": -4.344, "song_uri": "spotify:track:1fJFuvU2ldmeAm5nFIHcPP", "artist_uri": "spotify:artist:6FBDaR13swtiWwGhX1WQsP", "album_uri": "spotify:album:3nHpBmW5wJXGeC3ojBkpey"}, {"song": "The Rock Show", "artist": "blink-182", "album": "Take Off Your Pants And Jacket", "valence": 0.83, "energy": 0.959, "loudness": -4.563, "song_uri": "spotify:track:2ydUT1pFhuLDnouelIv4WH", "artist_uri": "spotify:artist:6FBDaR13swtiWwGhX1WQsP", "album_uri": "spotify:album:3nHpBmW5wJXGeC3ojBkpey"}, {"song": "EDGING", "artist": "blink-182", "album": "EDGING", "valence": 0.706, "energy": 0.905, "loudness": -3.117, "song_uri": "spotify:track:2wVWGFVkL5I3JGsoWBx2AZ", "artist_uri": "spotify:artist:6FBDaR13swtiWwGhX1WQsP", "album_uri": "spotify:album:0EspGdWdoWAxa5mBdQ5z55"}, {"song": "In Too Deep", "artist": "Sum 41", "album": "All Killer, No Filler", "valence": 0.766, "energy": 0.844, "loudness": -5.875, "song_uri": "spotify:track:1HNE2PX70ztbEl6MLxrpNL", "artist_uri": "spotify:artist:0qT79UgT5tY4yudH9VfsdT", "album_uri": "spotify:album:2UCWsnmZEVg9HhnMeKTsim"}, {"song": "Over My Head (Better Off Dead)", "artist": "Sum 41", "album": "Does This Look Infected?", "valence": 0.773, "energy": 0.919, "loudness": -4.462, "song_uri": "spotify:track:3SO0vfryYv381w1ImgWONG", "artist_uri": "spotify:artist:0qT79UgT5tY4yudH9VfsdT", "album_uri": "spotify:album:2kLmv0O8blKeM5HKxLtQrC"}, {"song": "Underclass Hero", "artist": "Sum 41", "album": "Underclass Hero", "valence": 0.765, "energy": 0.988, "loudness": -2.979, "song_uri": "spotify:track:6dXizHF3KbmdvOgvMAhnQC", "artist_uri": "spotify:artist:0qT79UgT5tY4yudH9VfsdT", "album_uri": "spotify:album:4fc73QNw5EjIorFfZ6n6YG"}]
```

Using the output above as your data source, answer the question Give me a list of Cars, Blink 182 and Sum 41 songs that are upbeat, loud and fun. Make sure the songs are popular enough for me to have heard of them.. Don't describe the code or process, just answer the question.
Answer:

> Finished chain.

> Finished chain.
```

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb48-1)print(overall_response)`
```

```
- You Might Think by The Cars
- Magic by The Cars 
- Shake It Up by The Cars
- First Date by blink-182 
- The Rock Show by blink-182 
- In Too Deep by Sum 41 
- Over My Head (Better Off Dead) by Sum 41 
- Underclass Hero by Sum 41
```

**Tada! There we go! That’s it!** That’s… it? Guess the magic is in the process.

Let’s see how it approached each aspect of my prompt:

> Give me a list of Cars, Blink 182 and Sum 41 songs

Made a list of the bands, looped through each band to search for them.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb50-1)artists = ['The Cars', 'Blink-182', 'Sum 41']
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb50-2)artist_uris = []
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb50-3)for artist in artists:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb50-4)    search_results = sp.search(q=artist, type='artist', limit=1)`
```

> Make sure the songs are popular enough for me to have heard of them.

GPT only pulled the top tracks from each artist, which implies popularity.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb51-1)for artist_uri in artist_uris:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb51-2)    artist_tracks = sp.artist_top_tracks(artist_uri)
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb51-3)    tracks.extend(artist_tracks['tracks'])`
```

> that are upbeat, loud and fun.

Filtering is done based on scores of valence, energy and loudness. Valence is “musical positiveness,” so that plus energy and loudness seems like a reasonable decision.

```
`[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb52-1)audio_details = sp.audio_features(subset_track_uris)
[](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb52-2)for j, details in enumerate(audio_details):
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb52-3)    if details['valence'] > 0.7 and details['energy'] > 0.7 and details['loudness'] > -7:
 [](https://jonathansoma.com/words/spotify-langchain-chatgpt.html#cb52-4)        track = tracks[i+j]`
```

One of the redeeming parts of running this code again and again was seeing how GPT translated the qualitative characteristics like “loud” or “fun” into actual quantitative numbers. Almost every time was a little bit different!

Reflections
===========

Mission: accomplished?

This was not as easy as I thought, _and_ the end result isn’t as nice as I’d like! But the path was really worth it in terms of exploration and understanding. Let’s talk about a few difficulties I ran into:

**I resisted reading the LangChain source early on,** and trying to make do with only the examples really slowed me down. Just open the code!!!! It’s plenty readable and concepts that might have been fuzzy clear up when you really dig in and see how the pieces fit together.

**Reminding GPT of gotchas with the Spotify API was difficult.** For example, tracks don’t have `track['album']['name']` if they’re from the `sp.album_tracks` endpoint while they do from `sp.artist_top_tracks`, different endpoints have different limits, etc etc. Carefully crafting examples while trying to not have ten thousand of them was not fun.

**Convincing the PALChain to not do too much work was almost impossible.** Since a Python-executing `PALChain` can do pretty much _anything_, it would often end up giving me Markdown tables or other formatting situations that I was hoping to instead get from the later `LLMChain` output. The fix as a combination of fine-tuning the prompt – _“you are ONLY providing DATA”_ – and adjusting my queries into a format closer to “find this data, then display it this way.”

All in all, maybe this would probably be better broken up into pieces via ReAct tools? I just didn’t think of that until I was like halfway through, but we at least survived.

It didn’t make a playlist!!! It just gave you a list of songs!!!

As much as I’m proud of my little LangChain child here, I was not about to give this bad boy access to my Spotify playlists. That also would have simplified the process - instead of going through the second `LLMChain`, we could have simply stopped at the `PALChain` part!

It’s also slightly more complicated auth flow with Spotify, which I didn’t want to step through. Maybe next time!

Feel free to check [the GitHub repo](https://github.com/jsoma/spotify-langchain-gpt) if you’d like a simple notebook that has the code pre-arranged for you.
---
layout: page
title: Infinite Playlists
description: Sequence-To-Sequence playlist generator based on Spotify featured playlists.
long_description: <h3>The process of discovering new playlists and/or artists on Spotify is far from perfect. There are a limited number of suggestions on their "Discover" and "Browse" pages, and there are only a handful of recommended songs attached to each playlist. This project aims to make discovery easier.</h3>The steps taken to create recommendations are as follows:<ol><li>Develop a Python script for extracting playlist data from Spotify, transforming that data into a tabular format, and uploading the result to an AWS S3 bucket.</li><li>Write a Dockerfile to containerize the ETL script. An entrypoint should be used to pass Spotify playlist ids to the script.</li><li>Leverage Kubernetes to run several ETL containers in parallel, using a Redis-based work queue to process ~200,000 Spotify playlists.</li><li>Develop a seq2seq model in Pytorch or Keras that accepts a new playlist ID as input and recommends one of the ~200,000 playlists.</li><li>Host the model in a simple Flask app.</li></ol>So far, steps 1-2 are complete; 3-4 are in progress.<h3>For more details, <a href="https://github.com/ZackBarry/infinitePlaylists" style = "color:blue">see the repository.</a></h3>
img: /assets/img/1.jpg
---

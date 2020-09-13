---
layout: post
title: Playlist Recommendation Project
date: 2020-07-20
description: Playlist Recommendation - Part 0
---

The process of discovering new playlists and/or artists on Spotify is far from perfect. There are a limited number of suggestions on their "Discover" and "Browse" pages, and there are only a handful of recommended songs attached to each playlist. As a data practitioner who loves to dig through crates of music to find new albums to obsess over, I wanted to see if I could improve this recommendation engine.

### Inspiration

This project is inspired by two papers. First, [Representation, Exploration and Recommendation of Music Playlists](https://arxiv.org/pdf/1907.01098.pdf) by P. Papreja, H. Venkateswara, and S. Panchanathan of Arizona State University. In this paper, the researchers take inspiration from neural machine translation to apply Sequence-to-Sequence models to the problem of playlist recommendation. They provide an example approach to corpus creation that seemed reasonable and replicable.

I wanted to extend the above work by taking a different approach to recommendation. To that end I found [Graph-based Recommendation Systems: Comparison Analysis between Traditional Clustering Techniques and Neural Embedding](http://snap.stanford.edu/class/cs224w-2017/projects/cs224w-58-final.pdf) by S. Jang, S. Kim, and J. Ha of Stanford University to be useful. This paper compares collaborative filtering, random walks, and neural embeddings as recommendation methods for products. Since market baskets are related through common items just as playlists are related by common songs, their methods can be adapted to playlists.

### Project Plan

Apart from discovering new music, I am excited to solve each part of an end-to-end machine learning project. This project will be broken in to 5 stages:

1. POC ETL - Determine how to extract a single playlists' details from Spotify's API to a cloud storage provider for future analysis. (*Python*, *Spotify API*, *AWS S3*)

2. Scale ETL - Build a database of > 200,000 playlists for modeling. (*Docker*, *k8s*, *AWS ECR*)

3. EDA - Determine how sparse the (playlist, song) matrix is, and the occurrence of difference genres. Explore potential models based on the results. (*Python*, *Spark*)

4. Build Model - Depending on the result of step 3, construct a recommendation model. (*Python*, *clustering*?, *word2vec*?, ...)

5. Deploy Model - Construct a dashboard where a user can input a playlist ID and receive a list of recommended playlists. (*Python*, *Flask*, *Spotify API*)

I'll be tracking my code on GitHub: https://github.com/ZackBarry/infinitePlaylists.

### Dependencies

A major goal for this project is self-improvement. The data science stack that I work with at my day job is comprised of R, Spark, Shiny, Airflow, and Google Cloud Platform. I have some experience working with Python, but very little exposure to AWS and Docker and none to Kubernetes. As such, Steps 2 and 5 contain many unknowns. However, I've succeeded at picking up new languages and tools in the past and I'm confident I can tackle any problems as they arise.

### Bonus: Music Recommendations

If you've made it this far, I'd love to share some of my favorite albums with you. Check out [this playlist](https://open.spotify.com/playlist/6lFw2lTLiXnEYTkdUl7ANj?si=kdGKJF7rRHuicRALjyy04w) on Spotify, and let me know what you think!

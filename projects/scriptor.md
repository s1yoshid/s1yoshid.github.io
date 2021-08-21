---
layout: page
title: Scriptor
description: >
  How you install Hydejack depends on whether you start a new site,
  or change the theme of an existing site.
hide_description: true
sitemap: false
permalink: /scriptor
---

**Scriptor** allows students to search for keywords and instantly be presented with the exact material they want. It returns precise timestamps in podcasts that match the information users seek, efficiently parsing through UCSD's vast database so students don't have to. It also includes a personalized interface, allowing users to favorite, save their history, and more.

<iframe width="560" height="315" src="https://www.youtube.com/embed/wJXGAqXLZew" frameborder="0"> </iframe>

This project was created for my CSE 110 Software Engineering course at UCSD in which it ended up being voted as the best project of the quarter. While I'd like to show the working website, it costs money to keep the website up and running so it has no live demo. If you'd like to see how it works and how it works, feel free to watch the video above! This project was conveived and created over a span of 10 weeks, thus still has a number of issues. One of the issues we faced during development was the inaccessibility of locked podcasts on the UCSD podcasts page. Another issues we ran into was the limitations of the Speech-to-Text API in which transcripts of lectures included a number of incorrectly identified words. However, it was well within the error margin as Speech-to-Text technology is yet to be perfected.

## Tools Used  
 * Google Cloud Speech-to-Text API
 * Elasticsearch
 * Python
 * Docker
 * Node
 * NPM
 * Git
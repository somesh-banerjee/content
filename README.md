# Basic Usage of Hugo

## Create new site
```
hugo new site <site_name>
```

## Create new post
```
cd <site_name>
hugo new posts/my-first-post.md
```

## Basic Header structure of posts
```
+++
title = "Knife Machine"
date = "2021-08-29T13:26:11+05:30"
author = "Somesh Banerjee"
authorTwitter = "banerjee_somesh"
cover = ""
tags = ["InfoSec", "HackTheBox"]
keywords = ["Linux", "knife", "RCE"]
description = ""
showFullContent = false
readingTime = true
draft: false
+++
```

## Running hugo server in local host
```
hugo server
```
Options:\
`-d`: to deploy drafts also

## Generating static pages based on some theme
```
hugo -t <theme-name>
```

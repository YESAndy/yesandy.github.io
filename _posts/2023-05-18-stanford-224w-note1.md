---
title: CS224W--Graph ML introduction and feature engineering
date: 2023-05-18 13:13:00 +/-0080
categories: [note]
tags: [course]     # TAG names should always be lowercase
layout: post
---

## Content
1. Why Graph in ML?
2. Classic Graph ML tasks
3. Choice of representations
4. Traditional feature engineering

<h2 data-toc-skip>1. Why Graph in ML?</h2>

### 1.1 Graph structure naturally exists in many data
![Desktop View](/assets/img/post/2023-05-18-data-as-graphs-eg.png){: width="972" height="589" }
_Data examples that can be represented as graph (from lecture 1.1 slide p6)_

It is beneficial to predict by explicitly modeling relationships.

### 1.2 Instead of feature engineering, 

However, it is also hard to learn from such structure, because:

1. topology-structured data like images and text, networks can have arbitary size and complex topological structure.
2. no fixed node ordering
3. often dynamic and have **multimodal features**

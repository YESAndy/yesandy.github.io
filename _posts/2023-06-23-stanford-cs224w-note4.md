---
title: CS224W--Graph as matrix - PageRank
author: andy
date: 2023-06-23 12:51:00 +/-0080
categories: [note]
tags: [cs224w]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. PageRank (lecture 4.1 & 4.2)
2. Random walk and restarts  (lecture 4.3)
3. Matrix factorization and node embeddings (lecture 4.4)

### 1. PageRank

web page contains in-links and out-links

The importance of page can be defined as the number of in-links, AKA links as votes

**The flow model** 

![Desktop View](/assets/img/post/2023-06-23-linkasvote.png){: width="480" height="480" }
_Link as vote_

Definitions:
 - Rank $r_j = \sum_{i \rightarrow j} \frac{r_i}{d_i}$, where $d_i$ is out-defree of node i.
 - Stochastic adjacency matrix $M_{ij} = \frac{1}{d_j}$, is a <mark>column</mark> stochastic matrix (columns sum to 1).
 - Rank vector $r$, where $\sum_i r_i = 1$.
 - <mark>$r = M r$</mark>.

**Connect to random walk**

Definitions:
 - $p(t)$ probability distribution over pages
 - $p(t+1) = M \dot p(t)$, $p(t)$ is a <mark>stationary distribution</mark> of a random walk.

---
title: CS224W--Node Embedding
author: andy
date: 2023-06-20 14:40:00 +/-0080
categories: [note]
tags: [cs224w]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. Node embeddings (lecture 3.1)
2. Random walk for node embeddings  (lecture 3.2)
3. Embed entire graph (lecture 3.3)

### 1. Node embedding


### 2. Random walk
**Random walk**: given a graph and a starting point, select a neighbor of it at random and move to the selected neighbor, and repeat it again and again.

Random walk embeddings: 
1) estimate the probability of visiting node v on a random walk starting from node u using some random walk strategy R,
2) optimize embeddings to encode these random walk statistics.

random walk optimization:
$Loss = \sum_{u \in V} \sum_{v \in N_R(u)} - log(P(v|z_u))$,
where $P(v|z_u) = \frac{exp(z_u^T z_v)}{\sum_{n \in V} exp(z_u^T z_n)}$, use softmax because we want node v to be most similar to node u out of all nodes n.

negative sampling
To reduce computational complexity, only sample a subset of all the nodes:
$$log(P(v|z_u)) = log(\frac{exp(z_u^T z_v)}{\sum_{n \in V} exp(z_u^T z_n)}) \aprox log(\sigma(z_u^T z_v)) - \sum_{i=1}^k log(\sigma(z_u^T z_{n_i}))$$, where $n_i \in P_V$, $P_V$ is random distribution other than uniform distribution.

**node2vec**
two strategies to define a neighborhood:
global: depth first search; local: breadth first search.


### Embed entire graph



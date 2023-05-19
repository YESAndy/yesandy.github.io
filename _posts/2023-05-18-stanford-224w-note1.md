---
title: CS224W--Graph ML introduction and feature engineering
date: 2023-05-18 13:13:00 +/-0080
categories: [note]
tags: [course]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---

## Content
1. Why Graph in ML?
2. Classic Graph ML tasks
3. Choice of representations
4. Traditional feature engineering

### 1. Why Graph in ML?

#### 1.1 Graph structure naturally exists in many data
![Desktop View](/assets/img/post/2023-05-18-data-as-graphs-eg.png){: width="972" height="589" }
_Data examples that can be represented as graph (from lecture 1.1 slide p6)_

It is beneficial to predict by explicitly modeling relationships.

#### 1.2 Use Deep Neural Network instead of feature engineering
With the advances in DL, we can input graph structure data into a end-to-end model that automatically learn and extracts useful features for graph ML tasks.

Therefore, a significant focus of graph learning is **representation learning**.

#### 1.3 Hard to learn from graph

![Desktop View](/assets/img/post/2023-05-18-data-structure-compare.png){: width="640" height="320" }
_Different data structures (from lecture 1.1 slide)_

However, it is also hard to learn from such structure, because:

1. topology-structured data like images and text, networks can have arbitary size and complex topological structure.
2. no fixed node ordering
3. often dynamic and have **multimodal features**

### 2. Classic graph ML tasks

![Desktop View](/assets/img/post/2023-05-18-task-types.png){: width="640" height="480" }
_Different tasks(from lecture 1.1 slide)_

#### 2.1 graph level
**graph classification**: categorize different graphs

example: molecule property prediction

**graph generalization**
example: drug discovery

**graph evolution**
example: physical simulation

#### 2.2 node level
**node classification**: predict a proterty of a node
categorize users/items


#### 2.3 subgraph level
**clustering**: detect if nodes form a community
social circle detection


#### 2.4 edge level
**link prediction**: predict whether there are missing links between two nodes
example: knowledge graph completion







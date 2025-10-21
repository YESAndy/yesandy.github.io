---
title: OpenFoam note1
author: andy
date: 2025-10-21 13:13:00 +/-0080
categories: [note]
tags: [openfoam]     # TAG names should always be lowercase
math: true
toc: true
layout: post
---


## Some concepts
### Patch
A patch is a named collection of boundary faces (the outer skin of your mesh). Each patch appears in constant/polyMesh/boundary and carries a type (e.g. patch, wall, empty, wedge, etc.) and is where you apply boundary conditions in 0/U, 0/p, etc.

### Boundary face
Boundary face = a face that has one owner cell (the other side is “outside” the mesh/domain).

### Internal face
Internal face = a face shared by two cells (it has an owner and a neighbour). Internal faces are not part of any patch.

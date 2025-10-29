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


## system
### blockMeshDict
__blockMeshDict__ defines the configuration for generating mesh.

usually include these items:
 - FoamFile: metadata
 - scale
 - vertices
 - blocks: hex, resolution, ...
 - boundary: boundary conditions for all geometric components

### controlDict
__controlDict__ defines the control of time and reading and writing of the solution data. A typical template is like this:

```C#
/*--------------------------------*- C++ -*----------------------------------*\
| =========                 |                                                 |
| \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox           |
|  \\    /   O peration     | Version:  v2412                                 |
|   \\  /    A nd           | Website:  www.openfoam.com                      |
|    \\/     M anipulation  |                                                 |
\*---------------------------------------------------------------------------*/
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      controlDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

application     icoFoam;

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         0.5;

deltaT          0.005;

writeControl    timeStep;

writeInterval   20;

purgeWrite      0;

writeFormat     ascii;

writePrecision  6;

writeCompression off;

timeFormat      general;

timePrecision   6;

runTimeModifiable true;


// ************************************************************************* //
```

### fvSchemes
The user specifies the choice of finite volume discretisation schemes in the fvSchemes dictionary.

### fvSolution
The specification of the linear equation solvers and tolerances and other algorithm controls is made in the fvSolution dictionary.

## constant
### PhysicalProperties
__PhysicalProperteis__ descirbes the physical properties such as Reynolds number

## 0 (initial values)

## viewer

## Mesh generation
In Openfoam, mesh generation workflow is like this:
```
blockMesh or external mesher
           │
           ▼
   ┌────────────────┐
   │ Background mesh│
   └────────────────┘
           │
           ▼
   ┌────────────────┐
   │  snappyHexMesh │◄───────────── Geometry (STL file)
   └────────────────┘
           │
           ▼
   ┌────────────────┐
   │  OpenFOAM mesh │
   └────────────────┘
```
### SnapyHexMesh
refer to this (doc)[https://www.wolfdynamics.com/wiki/meshing_OF_SHM.pdf].

## Some concepts
### Patch
A patch is a named collection of boundary faces (the outer skin of your mesh). Each patch appears in constant/polyMesh/boundary and carries a type (e.g. patch, wall, empty, wedge, etc.) and is where you apply boundary conditions in 0/U, 0/p, etc.

### Boundary face
Boundary face = a face that has one owner cell (the other side is “outside” the mesh/domain).

### Internal face
Internal face = a face shared by two cells (it has an owner and a neighbour). Internal faces are not part of any patch.

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

### SnapyHexMesh
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
refer to this (doc)[https://www.wolfdynamics.com/wiki/meshing_OF_SHM.pdf].

A snappyHexDict usually contains:
 - geometry: store info about stl
 - castellatedMeshControls: config about mesh generation
 - snapControls
 - MeshQualityControls


Here is a template snappyHexDict file:

```
FoamFile
{
    version     2.0;
    format      ascii;
    class       dictionary;
    object      snappyHexMeshDict;
}

castellatedMesh  true;
snap             true;
addLayers        false;

mergeTolerance   1e-6;
mergePatchFaces  yes;
keepPatches      yes;

geometry
{
    house
    {
        type triSurfaceMesh;
        file "house.stl";
    }
}
castellatedMeshControls
{
    maxLocalCells        1000000;
    maxGlobalCells       2000000;
    minRefinementCells   0;
    nCellsBetweenLevels  2;
    resolveFeatureAngle  30;
    allowFreeStandingZoneFaces true;
    maxLoadUnbalance     0.10;
    features ();

    refinementSurfaces
    {
        house
        {
            level (1 1);
            patchInfo { type wall; }
            regions
            {
                ceiling     { level (1 1); patchInfo { type wall; } }
                floor       { level (1 1); patchInfo { type wall; } }
                walls_side  { level (1 1); patchInfo { type wall; } }
            }
        }
    }
    refinementRegions {}

    locationInMesh (40.119041442871094 2.6096878051757812 -74.18577246665954);
}

snapControls
{
    nSmoothPatch 5;
    tolerance 2.0;
    nSolveIter 30;
    nRelaxIter 5;
}

addLayersControls
{
    relativeSizes true;
    layers ();
}

meshQualityControls
{
    maxNonOrtho          65;
    maxBoundarySkewness  20;
    maxInternalSkewness  4;
    maxConcave           80;
    minVol               1e-13;
    minTetQuality        1e-9;
    minArea              -1;
    minTwist             0.02;
    minDeterminant       0.001;
    minFaceWeight        0.02;
    minVolRatio          0.01;
    minTriangleTwist     -1;
    nSmoothScale         4;
    errorReduction       0.75;
    relaxed { maxNonOrtho 75; }
}
```


## constant
### PhysicalProperties
__PhysicalProperteis__ descirbes the physical properties such as Reynolds number

## 0 (initial values)
store intial values for pressure, temperature, velocity,...

## viewer
paraFoam




## Some concepts
### Patch
A patch is a named collection of boundary faces (the outer skin of your mesh). Each patch appears in constant/polyMesh/boundary and carries a type (e.g. patch, wall, empty, wedge, etc.) and is where you apply boundary conditions in 0/U, 0/p, etc.

### Boundary face
Boundary face = a face that has one owner cell (the other side is “outside” the mesh/domain).

### Internal face
Internal face = a face shared by two cells (it has an owner and a neighbour). Internal faces are not part of any patch.

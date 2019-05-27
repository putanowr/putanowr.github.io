.. include:: ../../replaces.txt
.. _mesh-sec:

******
Meshes
******

Simple meshes from mesh factory
===============================

Visualizing meshes
==================

Reading meshes from files
=========================

Mesh generation
===============

Dimension of the generated mesh
-------------------------------

For a start we should clearly distinguish between dimension of gemetric space
into which meshis embedded and the topological dimension of the mesh. The
geometric dimension, that is number of coordinates a point is always 3. The
topological dimension can vary between 1 and 3. As meshes can consist of
elements of different topological dimension the topological dimension of a mesh
is the highest dimension of its elements. If we denote by :math:`dim_{\text{Geom}}`
intrinsic dimension of a geometric object and by :math:`dim_{\text{Mesh}}` topological
dimension of the mesh of the object then we always have:

.. math::
    dim_{\text{Mesh}} \leqslant dim_{\text{Geom}}


Predefined geometric models
===========================

Basic mesh data
===============

Handling mesh connectivity
==========================

Building mesh hierarchies
=========================

Submeshes
=========

Geometric transofrmation of meshes
==================================


Selecting and tagging mesh enetities
====================================

Selecting nodes
---------------

Selecting edges
---------------

Selecting cells
---------------

Selecting entities per dimension 
--------------------------------

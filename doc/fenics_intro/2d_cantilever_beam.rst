2D Cantilever Beam Simulation
=============================

Introduction
------------

The simplest mechanical deformation simulation we can run is the 2d cantilever beam. This beam consists
of a rectangle of constant Young's modulus, fixed on the left with a downwards load on the right. Let's
have this beam be :math:`16\text{mm}` long in the x direction and :math:`3\text{mm}` thick in the y
direction::

 from __future__ import print_function
 import region_selector_2d as rs
 from dolfin import *

 length = 16.0 # [mm]
 thickness = 3.0 # [mm]
 resolution = 2 # [Nodes/mm]
 
 resX = int(resolution * length) # Num nodes in x axis
 resY = int(resolution * thickness) # Num nodes in y axis
 
 mesh = RectangleMesh(Point(0.0,0.0), Point(length, thickness), resX, resY)
 
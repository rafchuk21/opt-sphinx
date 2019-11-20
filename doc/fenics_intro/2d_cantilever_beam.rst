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

Now let's label the mesh. The end of the beam at :math:`x=0` should be fixed, and we will apply our load on the top of the beam, but only on the last millimeter farthest from the fixed end.

::

	fixedRegion = rs.GetLinearBoundary.from_points(Point(0.0,0.0), Point(0.0,thickness))
	loadRegion = rs.GetLinearBoundary.from_points(Point(length-1.0, thickness), Point(length, thickness))
	
	boundaries = MeshFunction('size_t', mesh, mesh.topology().dim()-1)
	boundaries.set_all(0)
	fixedRegion.mark(boundaries, 1)
	loadRegion.mark(boundaries, 2)
	
	ds = Measure('ds', domain=mesh, subdomain_data=boundaries)

When we write the boundaries information to a file using this code::

	folder_name = './2d_cantilever_results'
	
	boundaryFile = File('%s/boundaries.pvd' % folder_name)
	boundaryfile << boundaries

We can open the generated .pvd file in ParaView to see our mesh.

.. image:: 2d_cantilever_beam_mesh_01.png

We can see that most of the mesh has a 0 value set to it, with the left side set to a value of 1 and the right part of the top edge of the rectangle set to a value of 2. This matches the values that we assigned to the mesh above.

We can now write our model for the deformation of the beam. There's some math behind it, but we end up with

.. math::

   \text{Find } \boldsymbol{u} \in V \text{ s.t. } \int \langle \boldsymbol{\sigma}(\boldsymbol{u}), \boldsymbol{\varepsilon}(\boldsymbol{v}) \rangle = \int \boldsymbol{f}\cdot \boldsymbol{v} \quad\forall \boldsymbol{v} \in V


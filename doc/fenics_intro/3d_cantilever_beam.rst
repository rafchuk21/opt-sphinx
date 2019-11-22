3D Cantilever Beam Simulation
=============================

We can make a 3D cantilever beam simulation almost identically to how we made the 2D cantilever beam. We start by defining a mesh for a beam::

    from __future__ import print_function
    import numpy as np
    import region_selector_3d as rs
    from dolfin import *
    
    length = 16.0
    width = 5.0
    thickness = 3.0
    resolution = 2
    
    resX = int(resolution * length)
    resY = int(resolution * width)
    resZ = int(resolution * thickness)
    
    mesh = BoxMesh(Point(0.0,0.0,0.0), Point(length, width, thickness), resX, resY, resZ)

We will now label the fixed and load regions using ``region_selector_3d``. We will fix the face at :math:`x=0` and apply the load on the last 1mm on the top face of the beam furthest from the fixed end::

    fixedRegion = rs.GetPlanarBoundary.from_coord('x', 0.0)
    loadRegion = rs.GetPlanarBoundary.from_points(Point(length, 0.0, thickness), Point(length, width, thickness),\
            Point(length - 1.0, width, thickness), Point(length - 1.0, 0.0, thickness))
    
    boundaries = MeshFunction('size_t', mesh, mesh.topology.dim()-1)
    boundaries.set_all(0)
    fixedRegion.mark(boundaries,1)
    loadRegion.mark(boundaries,2)
    
    folder_name = './3d_cantilever_results'
    boundaryfile = File('%s/boundaries.pvd' % folder_name
    boundaryfile << boundaries
    
    ds = Measure('ds', domain = mesh, subdomain_data = boundaries)

ParaView lets us see our defined domains.

.. image:: 3d_cantilever_beam_mesh_01.png

We can see that the entire near face is marked 1, and the final milimeter of the top face is labeled 2, matching our description.
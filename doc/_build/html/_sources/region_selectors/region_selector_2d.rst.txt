2D Region Selector
==================

This file contains definitions for different 2D regions we may want to select.

------------------
Rectangular Region
------------------
This class selects a rectangular region whose sides are parallel to the :math:`x` and 
:math:`y` axes. It has two classmethods to allow it to be defined with either a pair of 
points or four floats. In general, it is recommended that the classes be created through 
the classmethods.

::

    class GetRectangularRegion(SubDomain):

The internal object init function takes four floats representing the two opposing corners 
of the rectangle. It sets ``x1`` and ``y1`` to the min of the two given values for x and y 
and ``x2`` and ``y2`` to the max so that the ``between()`` function can be used simply.

::

    def __init__(self, x1:float, y1:float, x2:float, y2:float):
        super().__init__()
        self.x1 = np.minimum(x1, x2)
        self.y1 = np.minimum(y1, y2)
        self.x2 = np.maximum(x1, x2)
        self.y2 = np.maximum(y1, y2)

This classmethod allows for region declaration using a pair of points. Their coordinates 
are used to call the main init function.

::

    @classmethod
    def from_points(cls, p1:Point, p2:Point) -> 'GetRectangularRegion':
        return cls(p1.x(), p1.y(), p2.x(), p2.y())

This classmethod gives an alias classmethod for the object init function.

::

    @classmethod
    def from_floats(cls, x1:float, y1:float, x2:float, y2:float) -> 'GetRectangularRegion':
        return cls(x1, y1, x2, y2)

The ``inside()`` method for this class simply returns whether a given :math:`\vec{x}` is between ``x1`` and ``x2`` and between ``y1`` and ``y2``.

::

    def inside(self, x, on_boundary):
        return(between(x[0], (self.x1, self.x2)) and \
                between(x[1], (self.y1, self.y2))

------------------
Circular Region
------------------

This class selects circular regions. Its internal init method takes three floats describing 
the center of the circle and its radius. It also has two classmethods which can be used to 
instantiate the class, one which takes three floats and one which takes a point and a float.

::

    class GetCircularRegion(SubDomain):
        def __init__(self, cx:float, cy:float, r:float):
            self.cx = cx
            self.cy = cy
            self.r = r
       
        @classmethod
        def from_points(cls, c:Point, r:float) -> 'GetCircularRegion':
            return cls(c.x(), c.y(), r)
       
        @classmethod
        def from_floats(cls, cx:float, cy:float, r:float) -> 'GetCircularRegion':
            return cls(cx, cy, r)
    
        def inside(self, x, on_boundary):
            return (x[0] - self.cx)**2.0 + (x[1] - self.cy)**2.0 <= r**2.0

------------------
Linear Boundary
------------------

------------------
Complete Code
------------------
The complete code follows and can also be downloaded :download:`here </../code/region_selector_2d.py>`.
::

    from dolfin import *
    import numpy as np

    # Select rectangular region
    class GetRectangularRegion(SubDomain):

        # x1: x coordinate of one corner
        # y1: y coordinate of one corner
        # x2: x coordinate of opposite corner
        # y2: y coordinate of opposite corner
        def __init__(self, x1:float, y1:float, x2:float, y2:float):
            super().__init__()
            self.x1 = np.minimum(x1, x2)
            self.y1 = np.minimum(y1, y2)
            self.x2 = np.maximum(x1, x2)
            self.y2 = np.maximum(y1, y2)

        # p1: Point of one corner of rectangle
        # p2: Point of opposite corner of rectangle
        @classmethod
        def from_points(cls, p1:Point, p2:Point) -> 'GetRectangularRegion':
            return cls(p1.x(), p1.y(), p2.x(), p2.y())

        # x1: x coordinate of one corner
        # y1: y coordinate of one corner
        # x2: x coordinate of opposite corner
        # y2: y coordiante of opposite corner
        @classmethod
        def from_floats(cls, x1:float, y1:float, x2:float, y2:float) -> 'GetRectangularRegion':
            return cls(x1, y1, x2, y2)

        def inside(self,x,on_boundary):
            return between(x[0], (self.x1,self.x2)) and between(x[1], (self.y1,self.y2))

    # Select circular region
    class GetCircularRegion(SubDomain):
        # cx: float x coord of circle center
        # cy: float y coord of circle center
        # r: float radius of circle
        def __init__(self, cx:float, cy:float, r:float):
            super().__init__()
            self.cx = cx
            self.cy = cy
            self.r = r
        
        # c: Point center of circle
        # r: float radius of circle
        @classmethod
        def from_points(cls, c:Point, r:float) -> 'GetCircularRegion':
            return cls(c.x(), c.y(), r)

        # cx: float x coord of circle center
        # cy: float y coord of circle center
        # r: float radius of circle
        @classmethod
        def from_floats(cls, cx:float, cy:float, r:float) -> 'GetCircularRegion':
            return cls(cx, cy, r)

        def inside(self,x,on_boundary):
            return (x[0] - self.cx)**2.0 + (x[1] - self.cy)**2.0 <= self.r**2.0

    # Select linear horizonta/vertical boundary region
    class GetLinearBoundary(SubDomain):
        # coord: Constant coordinate of the boundary line
        # range1: Min value along line to select
        # range2: Max value along line to select
        # horizontal: True/False whether the line is horizontal or vertical
        def __init__(self, coord:float, range1:float, range2:float, horizontal:bool):
            super().__init__()
            self.coord = coord
            self.range1 = np.minimum(range1,range2)
            self.range2 = np.maximum(range1,range2)
            self.ishorizontal = horizontal

        # p1: Point on one end of line
        # p2: Point on other end of line
        @classmethod
        def from_points(cls, p1:Point, p2:Point) -> 'GetLinearBoundary':
            x1 = p1.x()
            y1 = p1.y()
            x2 = p2.x()
            y2 = p2.y()

            # If x doesn't change, line is vertical
            if near(x1, x2):
                return cls(x1, y1, y2, False)
            # If y doesn't change, line is horizontal
            elif near(y1, y2):
                return cls(y1, x1, x2, True)
            else:
                raise ValueError("Linear boundaries must be horizontal or vertical")
            
        # coord: Constant coordinate of the boundary line
        # range1: Min value along line to select
        # range2: Max value along line to select
        # horizontal: True/False whether the line is horizontal or vertical
        @classmethod
        def from_floats(cls, coord:float, range1:float, range2:float, horizontal:bool) -> 'GetLinearBoundary':
            return cls(coord, range1, range2, horizontal)

        def inside(self,x,on_boundary):
            if self.ishorizontal:
                return near(x[1], self.coord) and between(x[0], (self.range1, self.range2)) and on_boundary
            else:
                return near(x[0], self.coord) and between(x[1], (self.range1, self.range2)) and on_boundary

    # Selects a single point
    class SelectPoint(SubDomain):
        # p: Point to select
        def __init__(self, p:Point):
            super().__init__()
            self.p = p

        def inside(self, x, on_boundary):
            return near(x[0], self.p.x()) and near(x[1], self.p.y())
<p align="center">
  <a href="https://github.com/nschloe/dmsh"><img alt="dmsh" src="figs/logo-with-text.svg" width="50%"></a>
  <p align="center">The worst mesh generator you'll ever use.</p>
</p>

[![PyPi Version](https://img.shields.io/pypi/v/dmsh.svg?style=flat-square)](https://pypi.org/project/dmsh/)
[![Packaging status](https://repology.org/badge/tiny-repos/python:dmsh.svg)](https://repology.org/project/python:dmsh/versions)
[![PyPI pyversions](https://img.shields.io/pypi/pyversions/dmsh.svg?style=flat-square)](https://pypi.org/project/dmsh/)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4728039.svg?style=flat-square)](https://doi.org/10.5281/zenodo.4728039)
[![GitHub stars](https://img.shields.io/github/stars/nschloe/dmsh.svg?style=flat-square&logo=github&label=Stars&logoColor=white)](https://github.com/nschloe/dmsh)
[![PyPi downloads](https://img.shields.io/pypi/dm/dmsh.svg?style=flat-square)](https://pypistats.org/packages/dmsh)

[![Discord](https://img.shields.io/static/v1?logo=discord&label=chat&message=on%20discord&color=7289da&style=flat-square)](https://discord.gg/PBCCvwHqpv)

[![gh-actions](https://img.shields.io/github/workflow/status/nschloe/dmsh/ci?style=flat-square)](https://github.com/nschloe/dmsh/actions?query=workflow%3Aci)
[![codecov](https://img.shields.io/codecov/c/github/nschloe/dmsh.svg?style=flat-square)](https://app.codecov.io/gh/nschloe/dmsh)
[![LGTM](https://img.shields.io/lgtm/grade/python/github/nschloe/dmsh.svg?style=flat-square)](https://lgtm.com/projects/g/nschloe/dmsh)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square)](https://github.com/psf/black)

Inspired by [distmesh](http://persson.berkeley.edu/distmesh/), dmsh can be slow,
requires a lot of memory, and isn't terribly robust either.

On the plus side,

 * it's got a user-friendly interface,
 * is pure Python (and hence easily installable on any system), and
 * it produces pretty high-quality meshes.

Combined with [optimesh](https://github.com/nschloe/optimesh), dmsh produces the
highest-quality 2D meshes in the west.

### Examples

#### Primitives

<img alt="circle" src="figs/circle.png" width="100%"> | <img alt="circle" src="figs/rectangle.png" width="100%"> | <img alt="circle" src="figs/polygon.png" width="100%">
|:---:|:---:|:---:|

```python
import dmsh
import meshio
import optimesh

geo = dmsh.Circle([0.0, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)

# optionally optimize the mesh
X, cells = optimesh.optimize_points_cells(X, cells, "CVT (full)", 1.0e-10, 100)

# visualize the mesh
dmsh.helpers.show(X, cells, geo)

# and write it to a file
meshio.Mesh(X, {"triangle": cells}).write("circle.vtk")
```

```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0)
X, cells = dmsh.generate(geo, 0.1)
```

```python
import dmsh

geo = dmsh.Polygon(
    [
        [0.0, 0.0],
        [1.1, 0.0],
        [1.2, 0.5],
        [0.7, 0.6],
        [2.0, 1.0],
        [1.0, 2.0],
        [0.5, 1.5],
    ]
)
X, cells = dmsh.generate(geo, 0.1)
```

#### Combinations

##### Difference

<img src="figs/moon.png" width="100%"> | <img src="figs/pacman.png" width="100%"> | <img src="figs/rectangle-hole-refinement.png" width="100%">
:-------------------:|:------------------:|:----:|

```python
import dmsh

geo = dmsh.Circle([-0.5, 0.0], 1.0) - dmsh.Circle([+0.5, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)
```
```python
import dmsh

geo = dmsh.Circle([0.0, 0.0], 1.0) - dmsh.Polygon([[0.0, 0.0], [1.5, 0.4], [1.5, -0.4]])
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```

The following example uses a nonconstant edge length; it depends on the distance to the
circle `c`.
```python
import dmsh
import numpy as np

r = dmsh.Rectangle(-1.0, +1.0, -1.0, +1.0)
c = dmsh.Circle([0.0, 0.0], 0.3)
geo = r - c

X, cells = dmsh.generate(geo, lambda pts: np.abs(c.dist(pts)) / 5 + 0.05, tol=1.0e-10)
```

##### Union

<img src="figs/union-circles.png" width="100%"> | <img src="figs/union-rectangles.png" width="100%"> | <img src="figs/union-three-circles.png" width="100%">
:-------------------:|:------------------:|:----:|

```python
import dmsh

geo = dmsh.Circle([-0.5, 0.0], 1.0) + dmsh.Circle([+0.5, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.15)
```
```python
import dmsh

geo = dmsh.Rectangle(-1.0, +0.5, -1.0, +0.5) + dmsh.Rectangle(-0.5, +1.0, -0.5, +1.0)
X, cells = dmsh.generate(geo, 0.15)
```
```python
import dmsh
import numpy as np

angles = np.pi * np.array([3.0 / 6.0, 7.0 / 6.0, 11.0 / 6.0])
geo = dmsh.Union(
    [
        dmsh.Circle([np.cos(angles[0]), np.sin(angles[0])], 1.0),
        dmsh.Circle([np.cos(angles[1]), np.sin(angles[1])], 1.0),
        dmsh.Circle([np.cos(angles[2]), np.sin(angles[2])], 1.0),
    ]
)
X, cells = dmsh.generate(geo, 0.15)
```

#### Intersection

<img src="figs/intersection-circles.png" width="100%"> | <img src="figs/intersection-three-circles.png" width="100%"> | <img src="figs/intersection-circle-halfspace.png" width="100%">
:-------------------:|:------------------:|:----:|

```python
import dmsh

geo = dmsh.Circle([0.0, -0.5], 1.0) & dmsh.Circle([0.0, +0.5], 1.0)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```

```python
import dmsh
import numpy as np

angles = np.pi * np.array([3.0 / 6.0, 7.0 / 6.0, 11.0 / 6.0])
geo = dmsh.Intersection(
    [
        dmsh.Circle([np.cos(angles[0]), np.sin(angles[0])], 1.5),
        dmsh.Circle([np.cos(angles[1]), np.sin(angles[1])], 1.5),
        dmsh.Circle([np.cos(angles[2]), np.sin(angles[2])], 1.5),
    ]
)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```

The following uses the `HalfSpace` primtive for cutting off a circle.
```python
import dmsh

geo = dmsh.HalfSpace([1.0, 1.0]) & dmsh.Circle([0.0, 0.0], 1.0)
X, cells = dmsh.generate(geo, 0.1)
```

### Rotation, translation, scaling

<img src="figs/rotation.png" width="100%"> | <img src="figs/scaling.png" width="100%">
|:----:|:----:|

```python
import dmsh
import numpy as np

geo = dmsh.Rotation(dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0), 0.1 * np.pi)
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-10)
```
```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0) + [1.0, 1.0]
X, cells = dmsh.generate(geo, 0.1)
```
```python
import dmsh

geo = dmsh.Rectangle(-1.0, +2.0, -1.0, +1.0) * 2.0
X, cells = dmsh.generate(geo, 0.1, tol=1.0e-5)
```

### Local refinement

<img alt="local-refinement" src="figs/local-refinement.png" width="30%">

All objects can be used to refine the mesh according to the distance to the object;
e.g. a `Path`:
```python
import dmsh

geo = dmsh.Rectangle(0.0, 1.0, 0.0, 1.0)

p1 = dmsh.Path([[0.4, 0.6], [0.6, 0.4]])


def edge_size(x):
    return 0.03 + 0.1 * p1.dist(x)


X, cells = dmsh.generate(geo, edge_size, tol=1.0e-10)
```


### Custom shapes
It is also possible to define your own geometry. Simply create a class derived from
`dmsh.Geometry` that contains a `dist` method and a method to project points onto the
boundary.

```python
import dmsh
import numpy as np


class MyDisk(dmsh.Geometry):
    def __init__(self):
        super().__init__()
        self.r = 1.0
        self.x0 = [0.0, 0.0]
        self.bounding_box = [-1.0, 1.0, -1.0, 1.0]
        self.feature_points = np.array([[], []]).T

    def dist(self, x):
        assert x.shape[0] == 2
        y = (x.T - self.x0).T
        return np.sqrt(np.einsum("i...,i...->...", y, y)) - self.r

    def boundary_step(self, x):
        # project onto the circle
        y = (x.T - self.x0).T
        r = np.sqrt(np.einsum("ij,ij->j", y, y))
        return ((y / r * self.r).T + self.x0).T


geo = MyDisk()
X, cells = dmsh.generate(geo, 0.1)
```

### Debugging

![level-set-poly](figs/levelset-polygon.png) | ![level-set-rect-hole](figs/levelset-rect-hole.png)
|:----:|:----:|

dmsh is rather fragile, but sometimes the break-downs are due to an incorrectly defined
geometry. Use
```
geo.show()
```
to inspect the level set function of your domain. (It must be negative inside the
domain and positive outside. The 0-level set forms the domain boundary.)


### Installation

dmsh is [available from the Python Package
Index](https://pypi.org/project/dmsh/), so simply type
```
pip install dmsh
```
to install.

### Testing
To run the dmsh unit tests, check out this repository and type
```
tox
```

### License
This software is published under the [MIT
license](https://en.wikipedia.org/wiki/MIT_License).

# Grasshopper IronPython — API Reference

## Rhino.Geometry (rg) Common Operations

### Points

```python
import Rhino.Geometry as rg

pt = rg.Point3d(x, y, z)
pt2 = rg.Point3d(pt)  # copy
dist = pt.DistanceTo(pt2)

# Midpoint
mid = rg.Point3d(
    (pt.X + pt2.X) / 2,
    (pt.Y + pt2.Y) / 2,
    (pt.Z + pt2.Z) / 2
)

# Move point
moved = rg.Point3d(pt.X + dx, pt.Y + dy, pt.Z + dz)
```

### Vectors

```python
vec = rg.Vector3d(x, y, z)
vec = rg.Vector3d(pt2 - pt)  # from point to point
vec.Unitize()  # normalize in-place, returns bool
length = vec.Length
dot = vec * vec2  # dot product
cross = rg.Vector3d.CrossProduct(vec, vec2)

# Standard axes
rg.Vector3d.XAxis  # (1,0,0)
rg.Vector3d.YAxis  # (0,1,0)
rg.Vector3d.ZAxis  # (0,0,1)
```

### Lines and Curves

```python
line = rg.Line(pt_start, pt_end)
line.Length
line.PointAt(0.5)  # midpoint (t from 0 to 1)
line.ClosestPoint(pt, limitToFiniteSegment=True)
line.From  # start point
line.To    # end point
line.Direction  # vector from→to (not unitized)

# LineCurve (curve wrapper for Line)
lc = rg.LineCurve(pt_start, pt_end)
lc.PointAtStart
lc.PointAtEnd

# Line-line intersection
success, t1, t2 = rg.Intersect.Intersection.LineLine(line1, line2)
```

### Polylines

```python
pts = [rg.Point3d(0,0,0), rg.Point3d(1,0,0), rg.Point3d(1,1,0)]
polyline = rg.Polyline(pts)
polyline.Count        # number of points
polyline[0]           # first point
polyline.Length       # total length
polyline.IsClosed

# From PolylineCurve
plc = rg.PolylineCurve(pts)
pl = rg.Polyline()
success = plc.TryGetPolyline(pl)
```

### Planes

```python
plane = rg.Plane(origin_pt, normal_vec)
plane = rg.Plane(origin_pt, x_axis_vec, y_axis_vec)
plane = rg.Plane.WorldXY

plane.Origin     # rg.Point3d
plane.Normal     # rg.Vector3d
plane.XAxis
plane.YAxis
```

### Rectangles

```python
rect = rg.Rectangle3d(plane, width, height)
rect = rg.Rectangle3d(plane, corner1_pt, corner2_pt)
rect.Center
rect.Plane
rect.Width
rect.Height
rect.BoundingBox
crv = rect.ToNurbsCurve()  # for extrusion etc.
```

### BoundingBox

```python
bbox = rg.BoundingBox(min_pt, max_pt)
bbox = geom.GetBoundingBox(True)  # accurate=True
bbox.Min  # rg.Point3d
bbox.Max  # rg.Point3d
bbox.Center
bbox.IsValid
bbox = rg.BoundingBox.Union(bbox1, bbox2)
```

### Transformations

```python
xform = rg.Transform.Translation(vec)
xform = rg.Transform.Rotation(angle_rad, axis_vec, center_pt)
xform = rg.Transform.Scale(center_pt, factor)
xform = rg.Transform.PlaneToPlane(source_plane, target_plane)

# Apply to geometry (returns new object for immutable types)
new_pt = rg.Point3d(pt)
new_pt.Transform(xform)

crv_copy = crv.DuplicateCurve()
crv_copy.Transform(xform)
```

### Brep Operations

```python
extr = rg.Extrusion.Create(profile_curve, height, cap=True)
brep = extr.ToBrep()

# Boolean operations
result = rg.Brep.CreateBooleanUnion(breps_list, tolerance)
result = rg.Brep.CreateBooleanDifference(brep_a, brep_b, tolerance)
result = rg.Brep.CreateBooleanIntersection(brep_a, brep_b, tolerance)
```

### Intersections

```python
import Rhino.Geometry.Intersect as ix

# Curve-Curve
events = ix.Intersection.CurveCurve(crv1, crv2, tolerance, overlap_tol)
for ev in events:
    pt = ev.PointA
    t1 = ev.ParameterA
    t2 = ev.ParameterB

# Curve-Brep
success, pts, crv_overlaps = ix.Intersection.CurveBrep(crv, brep, tolerance)

# Line-Line
success, t1, t2 = ix.Intersection.LineLine(line1, line2)
```

## Grasshopper-Specific

### DataTree with typed .NET generics

Two import styles (both valid, first matches CodeStyleGuide):

```python
# Style 1 — via gh alias (CodeStyleGuide)
import Grasshopper as gh
import System
tree = gh.DataTree[System.Object]()
path = gh.Kernel.Data.GH_Path(0)

# Style 2 — direct import (common in project)
from Grasshopper import DataTree
from Grasshopper.Kernel.Data import GH_Path
tree = DataTree[object]()
path = GH_Path(0)
```

```python
# Create typed trees
tree = gh.DataTree[System.Object]()
tree = gh.DataTree[str]()
tree = gh.DataTree[float]()
tree = gh.DataTree[rg.Point3d]()

# Add items
path = gh.Kernel.Data.GH_Path(0)        # single index: {0}
path = gh.Kernel.Data.GH_Path(0, 1)     # nested: {0;1}
tree.Add(item, path)
tree.AddRange(list_items, path)  # preferred for adding lists

# Ensure branch exists (even if empty)
tree.EnsurePath(gh.Kernel.Data.GH_Path(5))

# Read
for branch in tree.Branches:
    for item in branch:
        pass

# Path info
tree.PathCount   # number of branches
tree.Paths       # list of GH_Path
branch = tree.Branch(path)  # items in branch
```

### ghenv Component API

```python
# Component message (version label)
ghenv.Component.Message = "MyComp v0.01"

# Component GUID (unique per instance)
guid = str(ghenv.Component.InstanceGuid)

# Runtime messages
import Grasshopper.Kernel as ghk
ghenv.Component.AddRuntimeMessage(
    ghk.GH_RuntimeMessageLevel.Warning, "msg"
)
ghenv.Component.AddRuntimeMessage(
    ghk.GH_RuntimeMessageLevel.Error, "msg"
)

# Input/Output descriptions (set programmatically)
ghenv.Component.Params.Input[0].Description = "Input param description"
ghenv.Component.Params.Output[0].Description = "Output param description"

# Force recompute
ghenv.Component.ExpireSolution(True)
```

### sc.sticky (persistent state between runs)

```python
import scriptcontext as sc

KEY = "my_key_" + str(ghenv.Component.InstanceGuid)

# Store
sc.sticky[KEY] = data

# Read safely
data = sc.sticky.get(KEY, default_value)

# Check existence
if KEY in sc.sticky:
    pass

# Delete
if KEY in sc.sticky:
    del sc.sticky[KEY]
```

## .NET Interop

### System types

```python
import System

# Check for .NET string
isinstance(val, (str, System.String))

# .NET list
net_list = System.Collections.Generic.List[System.Object]()
net_list.Add(item)

# Guid
guid = System.Guid.NewGuid()
```

### File I/O with System.IO

```python
import System.IO as SIO

temp_dir = SIO.Path.GetTempPath()
full_path = SIO.Path.Combine(dir_path, filename)
SIO.Path.GetExtension(path)
SIO.Path.GetFileNameWithoutExtension(path)
```

### JSON handling

```python
import json

# Read from file
with open(path, 'rb') as f:
    data = json.loads(f.read().decode('utf-8'))

# Write to file
s = json.dumps(data, ensure_ascii=False, indent=2)
with open(path, 'wb') as f:
    f.write(s.encode('utf-8'))

# Loose parsing (handle Python-style booleans)
import re
def parse_json_loose(s):
    try:
        return json.loads(s)
    except:
        s2 = s.replace("'", '"')
        s2 = re.sub(r"\bNone\b", "null", s2)
        s2 = re.sub(r"\bTrue\b", "true", s2)
        s2 = re.sub(r"\bFalse\b", "false", s2)
        return json.loads(s2)
```

## Geometry Type Checking

```python
def extract_endpoints(geom):
    """Extract start/end points from any line-like geometry"""
    if isinstance(geom, rg.Line):
        return (geom.From, geom.To)
    elif isinstance(geom, rg.LineCurve):
        return (geom.PointAtStart, geom.PointAtEnd)
    elif isinstance(geom, rg.PolylineCurve):
        return (geom.PointAtStart, geom.PointAtEnd)
    elif isinstance(geom, rg.Curve):
        if not geom.IsClosed:
            return (geom.PointAtStart, geom.PointAtEnd)
    return None
```

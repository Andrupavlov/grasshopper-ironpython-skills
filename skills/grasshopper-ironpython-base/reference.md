# Rhino.Geometry API Reference & .NET Interop

## Imports

```python
import Rhino.Geometry as rg
import Rhino
import System
import System.Drawing as sd
import System.Collections.Generic as scg
```

## Point3d, Vector3d, Plane

### Point3d

```python
pt = rg.Point3d(1.0, 2.0, 3.0)
pt2 = rg.Point3d(pt)                   # copy
origin = rg.Point3d.Origin              # (0, 0, 0)
unset = rg.Point3d.Unset               # invalid sentinel

x, y, z = pt.X, pt.Y, pt.Z
dist = pt.DistanceTo(pt2)
pt.Transform(xform)                     # in-place transform

mid = (pt + pt2) / 2.0                  # midpoint via operator overloads
vec = pt2 - pt                          # Point3d - Point3d → Vector3d
moved = pt + vec                        # Point3d + Vector3d → Point3d
```

### Vector3d

```python
v = rg.Vector3d(1.0, 0.0, 0.0)
v2 = rg.Vector3d(pt2 - pt)             # from two points

length = v.Length
v.Unitize()                             # normalize in-place (returns bool)
rev = rg.Vector3d(-v.X, -v.Y, -v.Z)    # reverse
rev = -v                                # reverse via operator

dot = v * v2                            # dot product (operator *)
cross = rg.Vector3d.CrossProduct(v, v2)
angle = rg.Vector3d.VectorAngle(v, v2)  # radians

rg.Vector3d.XAxis   # (1, 0, 0)
rg.Vector3d.YAxis   # (0, 1, 0)
rg.Vector3d.ZAxis   # (0, 0, 1)
```

### Plane

```python
pl = rg.Plane.WorldXY
pl = rg.Plane(origin_pt, normal_vec)
pl = rg.Plane(origin_pt, x_vec, y_vec)
pl = rg.Plane(pt_a, pt_b, pt_c)        # from 3 points

o = pl.Origin
n = pl.Normal
xdir, ydir = pl.XAxis, pl.YAxis

closest = pl.ClosestPoint(pt)           # returns Point3d
rc, u, v = pl.ClosestParameter(pt)      # rc=bool, u/v=float (UV coords)
dist = pl.DistanceTo(pt)                # signed distance
```

## Lines, Polylines, Arcs, Circles

### Line

```python
ln = rg.Line(pt_a, pt_b)
ln = rg.Line(pt_a, direction_vec, length)

start, end = ln.From, ln.To
mid = ln.PointAt(0.5)                   # parameter 0..1
length = ln.Length
ln_crv = ln.ToNurbsCurve()              # Line → NurbsCurve
closest = ln.ClosestPoint(pt, True)     # limitToFiniteSegment=True
```

### Polyline

```python
pts = [rg.Point3d(i, 0, 0) for i in range(5)]
pline = rg.Polyline(pts)
pline_crv = pline.ToNurbsCurve()        # Polyline → NurbsCurve
pline_crv = pline.ToPolylineCurve()     # Polyline → PolylineCurve

length = pline.Length
count = pline.Count                     # number of vertices
pt = pline[2]                           # access vertex by index
pline.Add(rg.Point3d(5, 0, 0))         # append vertex
is_closed = pline.IsClosed
```

### Arc & Circle

```python
arc = rg.Arc(center_pt, radius, angle_radians)
arc = rg.Arc(pt_a, pt_b, pt_c)         # 3-point arc
arc_crv = arc.ToNurbsCurve()

circle = rg.Circle(plane, radius)
circle = rg.Circle(pt_a, pt_b, pt_c)   # 3-point circle
circle_crv = circle.ToNurbsCurve()
circumference = circle.Circumference
center = circle.Center
```

## Curves (NurbsCurve, PolyCurve, general Curve)

All concrete curve types inherit from `rg.Curve` — use `rg.Curve` methods on any curve.

### Evaluation

```python
pt = crv.PointAtStart
pt = crv.PointAtEnd
pt = crv.PointAt(t)                     # parameter t
pt = crv.PointAtNormalizedLength(0.5)   # 0..1 along length

tangent = crv.TangentAt(t)
rc, plane = crv.FrameAt(t)             # rc=bool, plane with origin at t

length = crv.GetLength()
domain = crv.Domain                     # Interval(t0, t1)
t_mid = crv.Domain.Mid
```

### Closest point & parameters

```python
rc, t = crv.ClosestPoint(pt)           # rc=bool, t=parameter
closest_pt = crv.PointAt(t)
dist = pt.DistanceTo(closest_pt)

rc, t = crv.NormalizedLengthParameter(0.5)  # length fraction → param
```

### Splitting, trimming, joining

```python
segments = crv.Split(t)                 # returns array of Curve at param t

trimmed = crv.Trim(t0, t1)             # sub-curve between params

joined = rg.Curve.JoinCurves(curve_list)           # returns array
joined = rg.Curve.JoinCurves(curve_list, tolerance) # with tolerance
```

### Offsets & fillets

```python
offsets = crv.Offset(
    plane,          # offset plane
    distance,       # positive = left, negative = right
    tolerance,
    rg.CurveOffsetCornerStyle.Sharp
)

fillets = rg.Curve.CreateFilletCurves(
    crv_a, crv_a_pt,    # curve A and point near fillet
    crv_b, crv_b_pt,    # curve B and point near fillet
    radius,
    True,               # join
    True,               # trim
    True,               # arcExtension
    tolerance,          # tolerance
    tolerance           # angleTolerance
)
```

### Booleans & intersections

```python
events = rg.Intersect.Intersection.CurveCurve(
    crv_a, crv_b, tolerance, overlap_tolerance
)
for ev in events:
    if ev.IsPoint:
        pt = ev.PointA
        t_a, t_b = ev.ParameterA, ev.ParameterB
    elif ev.IsOverlap:
        overlap_crv = ev.OverlapA
```

### NurbsCurve construction

```python
nc = rg.NurbsCurve.CreateControlPointCurve(pts, degree)

nc = rg.NurbsCurve(degree, point_count)
for i, pt in enumerate(pts):
    nc.Points.SetPoint(i, pt)
```

### Curve type checks

```python
if isinstance(crv, rg.LineCurve):
    line = crv.Line
elif isinstance(crv, rg.ArcCurve):
    arc = crv.Arc
elif isinstance(crv, rg.PolylineCurve):
    pline = crv.TryGetPolyline()       # returns (bool, Polyline)
```

## Surfaces & Breps

### Surface evaluation

```python
u_dom = srf.Domain(0)                  # U domain (Interval)
v_dom = srf.Domain(1)                  # V domain

pt = srf.PointAt(u, v)
normal = srf.NormalAt(u, v)
rc, u, v = srf.ClosestPoint(pt)

rc, frame = srf.FrameAt(u, v)         # rc=bool, frame=Plane
```

### Surface creation

```python
srf = rg.NurbsSurface.CreateFromCorners(pt_a, pt_b, pt_c, pt_d)

loft = rg.Brep.CreateFromLoft(
    curves,                # IEnumerable<Curve>
    rg.Point3d.Unset,     # start point (or Unset)
    rg.Point3d.Unset,     # end point (or Unset)
    rg.LoftType.Normal,
    False                  # closed loft
)

extrusion = rg.Extrusion.Create(
    profile_crv,           # planar closed curve
    height,
    cap                    # bool — cap ends
)
```

### Brep operations

```python
brep = crv.Extrude(direction_vec).ToBrep()
brep.Faces.ShrinkFaces()

area = rg.AreaMassProperties.Compute(brep)
if area:
    a = area.Area
    centroid = area.Centroid

vol = rg.VolumeMassProperties.Compute(brep)
if vol:
    v = vol.Volume
    centroid = vol.Centroid

bbox = brep.GetBoundingBox(True)       # True = accurate
min_pt, max_pt = bbox.Min, bbox.Max
```

### Brep booleans

```python
results = rg.Brep.CreateBooleanUnion(brep_list, tolerance)
results = rg.Brep.CreateBooleanDifference(brep_a, brep_b, tolerance)
results = rg.Brep.CreateBooleanIntersection(brep_a, brep_b, tolerance)
```

### Brep topology traversal

```python
for face in brep.Faces:
    srf = face.UnderlyingSurface()
    area = rg.AreaMassProperties.Compute(face)

for edge in brep.Edges:
    crv = edge.EdgeCurve
    length = crv.GetLength()

for vertex in brep.Vertices:
    pt = vertex.Location
```

## Meshes

### Creation

```python
mesh = rg.Mesh()
mesh.Vertices.Add(0, 0, 0)
mesh.Vertices.Add(1, 0, 0)
mesh.Vertices.Add(1, 1, 0)
mesh.Vertices.Add(0, 1, 0)
mesh.Faces.AddFace(0, 1, 2, 3)         # quad face (4 indices)
mesh.Faces.AddFace(0, 1, 2)            # tri face (3 indices)
mesh.Normals.ComputeNormals()
mesh.Compact()
```

### Brep → Mesh

```python
mp = rg.MeshingParameters.Default      # or .Coarse, .Fine, .QualityRenderMesh
meshes = rg.Mesh.CreateFromBrep(brep, mp)
if meshes:
    joined = rg.Mesh()
    for m in meshes:
        joined.Append(m)
```

### Mesh queries

```python
count_v = mesh.Vertices.Count
count_f = mesh.Faces.Count
pt = mesh.Vertices[i]                  # Point3f
face = mesh.Faces[i]                   # MeshFace → .A, .B, .C, .D

bbox = mesh.GetBoundingBox(True)
area = rg.AreaMassProperties.Compute(mesh)
mesh_pt = mesh.ClosestMeshPoint(pt, 0.0)  # returns MeshPoint
```

## Transformations

All geometry with `.Transform(xform)` method. Transformations are `rg.Transform` matrices.

```python
move = rg.Transform.Translation(rg.Vector3d(dx, dy, dz))
move = rg.Transform.Translation(dx, dy, dz)

scale_uniform = rg.Transform.Scale(center_pt, factor)
scale_xyz = rg.Transform.Scale(plane, sx, sy, sz)

rot = rg.Transform.Rotation(angle_radians, axis_vec, center_pt)

mirror = rg.Transform.Mirror(plane)

p2p = rg.Transform.PlaneToPlane(source_plane, target_plane)

proj = rg.Transform.PlanarProjection(plane)
```

### Applying transforms

```python
geom_copy = geom.Duplicate()
geom_copy.Transform(xform)

combined = rg.Transform.Multiply(xform_b, xform_a)  # b applied after a
```

## Intersections (Rhino.Geometry.Intersect)

```python
from Rhino.Geometry.Intersect import Intersection

rc, t = Intersection.LinePlane(line, plane)

rc, a, b = Intersection.LineLine(line_a, line_b)   # a=param on line_a, b=param on line_b
# closest point on line_a: line_a.PointAt(a)

rc, line = Intersection.PlanePlane(plane_a, plane_b)

rc, pt = Intersection.PlanePlanePlane(pl_a, pl_b, pl_c)   # intersection Point3d

events = Intersection.CurveSurface(crv, srf, tol, overlap_tol)   # returns CurveIntersections

rc, crvs, pts = Intersection.BrepBrep(brep_a, brep_b, tolerance)   # crvs=Curve[], pts=Point3d[]

events = Intersection.CurveCurve(crv_a, crv_b, tol, overlap_tol)

pts = Intersection.MeshMeshFast(mesh_a, mesh_b)   # returns Point3d[] of intersection points
```

## BoundingBox

```python
bbox = geom.GetBoundingBox(True)       # accurate
bbox = geom.GetBoundingBox(plane)      # oriented to plane

min_pt = bbox.Min                      # Point3d
max_pt = bbox.Max                      # Point3d
center = bbox.Center
is_valid = bbox.IsValid

contains = bbox.Contains(pt)           # bool
bbox = rg.BoundingBox.Union(bbox, other_bbox)   # expand to include other (static; BoundingBox is a struct)
```

## Interval (parameter domains)

```python
iv = rg.Interval(0.0, 1.0)
iv = crv.Domain

t0, t1 = iv.T0, iv.T1
mid = iv.Mid
length = iv.Length
iv.Reverse()

t_norm = iv.NormalizedParameterAt(t)   # map t → 0..1
t_real = iv.ParameterAt(0.5)           # map 0..1 → actual param
```

## .NET Interop Patterns

### System.Drawing.Color

```python
import System.Drawing as sd

color = sd.Color.FromArgb(255, 128, 0)       # RGB
color = sd.Color.FromArgb(200, 255, 128, 0)  # ARGB with alpha
color = sd.Color.Red                          # named color

r, g, b, a = color.R, color.G, color.B, color.A
```

### .NET collections ↔ Python

```python
import System
import System.Collections.Generic as scg

net_list = scg.List[rg.Point3d]()
net_list.Add(pt)
net_list.AddRange(python_list)         # bulk add

python_list = list(net_list)           # .NET list → Python list

net_array = System.Array[rg.Point3d](python_list)
python_list = list(net_array)          # .NET array → Python list
```

### Type casting in GH context

```python
import Rhino

guid = System.Guid(guid_string)
obj_ref = Rhino.DocObjects.ObjRef(guid)
rh_obj = Rhino.RhinoDoc.ActiveDoc.Objects.FindId(guid)
geom = rh_obj.Geometry
```

### System.Enum flags

```python
import System
style = System.Enum.Parse(
    rg.CurveOffsetCornerStyle,
    "Sharp"
)
```

## Tolerance & Document Settings

```python
import Rhino

tol = Rhino.RhinoDoc.ActiveDoc.ModelAbsoluteTolerance   # typically 0.001 or 0.01
angle_tol = Rhino.RhinoDoc.ActiveDoc.ModelAngleToleranceRadians
units = Rhino.RhinoDoc.ActiveDoc.ModelUnitSystem         # e.g. UnitSystem.Millimeters
```

## Common Pitfalls

| Pitfall | Correct approach |
|---------|-----------------|
| `rg.Point3d` is a value type — assignment copies | Use `rg.Point3d(pt)` when you need an explicit copy |
| `Curve.ClosestPoint` returns `(bool, float)` | Always check `rc` before using the parameter `t` |
| `Brep.CreateBoolean*` may return `None` | Always check result for `None` before iterating |
| `Mesh.Vertices[i]` returns `Point3f`, not `Point3d` | Cast: `rg.Point3d(mesh.Vertices[i])` |
| `Transform` modifies geometry in-place | Call `.Duplicate()` first if you need the original |
| `NurbsCurve.CreateControlPointCurve` degree > len(pts)-1 | Ensure `degree <= len(pts) - 1` |
| `Extrusion.Create` requires planar closed profile | Validate with `crv.IsClosed` and `crv.IsPlanar()` |

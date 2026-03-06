# Grasshopper IronPython — Component Templates

## Template 1: Simple Data Processor

Minimal component that takes input, processes it, outputs result.

```python
"""
GH Python (IronPython)
Description: Process input data and produce output.

Inputs:
    data_in  : List[str] — input data items
    enabled  : bool      — enable processing (default: True)

Outputs:
    result   : List[str] — processed results
    count    : int       — number of items processed
    ok       : bool      — success flag
"""

ghenv.Component.Message = "DataProcessor v0.01"

ok = False
result = []
count = 0

if 'enabled' not in globals() or enabled is None:
    enabled = True
if 'data_in' not in globals() or data_in is None:
    data_in = []

try:
    if not data_in:
        result = []
        count = 0
    else:
        for item in data_in:
            processed = str(item).strip()
            if processed:
                result.append(processed)
        count = len(result)
    ok = True
except Exception as e:
    ok = False
    print("Error: {}".format(str(e)))
```

## Template 2: Geometry Processor with DataTree Output

Component that works with geometry and produces DataTree output.

```python
"""
GH Python (IronPython)
Description: Group points by proximity and output as DataTree.

Inputs:
    points    : List[Point3d] — input points
    tolerance : float         — grouping tolerance (default: 1.0)

Outputs:
    groups     : DataTree[Point3d] — grouped points
    centroids  : List[Point3d]     — centroid of each group
    debug      : List[str]         — debug messages
"""

import Rhino.Geometry as rg
import Grasshopper as gh
import System

ghenv.Component.Message = "PointGrouper v0.01"

DEFAULT_TOLERANCE = 1.0

groups = gh.DataTree[rg.Point3d]()
centroids = []
debug = []

if 'tolerance' not in globals() or tolerance is None:
    tolerance = DEFAULT_TOLERANCE

def find_group(point, group_centers, tol):
    """Find which group a point belongs to, or -1 if none"""
    for i, center in enumerate(group_centers):
        if point.DistanceTo(center) <= tol:
            return i
    return -1

def compute_centroid(pts):
    """Compute centroid of point list"""
    if not pts:
        return rg.Point3d.Origin
    sx, sy, sz = 0.0, 0.0, 0.0
    for p in pts:
        sx += p.X
        sy += p.Y
        sz += p.Z
    n = float(len(pts))
    return rg.Point3d(sx / n, sy / n, sz / n)

try:
    if not points:
        debug.append("No input points")
    else:
        group_lists = []
        group_centers = []

        for pt in points:
            idx = find_group(pt, group_centers, tolerance)
            if idx == -1:
                group_lists.append([pt])
                group_centers.append(pt)
            else:
                group_lists[idx].append(pt)
                group_centers[idx] = compute_centroid(group_lists[idx])

        for i, grp in enumerate(group_lists):
            path = gh.Kernel.Data.GH_Path(i)
            groups.AddRange(grp, path)
            centroids.append(group_centers[i])

        debug.append("Found {} groups from {} points".format(
            len(group_lists), len(points)))

except Exception as e:
    debug.append("Error: {}".format(str(e)))
```

## Template 3: Stateful Component with sc.sticky

Component that persists data between Grasshopper recalculations.

```python
"""
GH Python (IronPython)
Description: Accumulate values across multiple runs using sc.sticky.

Inputs:
    value : str  — value to store
    key   : str  — storage key
    reset : bool — clear stored data

Outputs:
    stored : List[str] — all stored values
    count  : int       — number of stored values
"""

import scriptcontext as sc

ghenv.Component.Message = "Accumulator v0.01"

comp_guid = str(ghenv.Component.InstanceGuid)
STICKY_PREFIX = "accumulator_"

stored = []
count = 0

if 'key' not in globals() or key is None:
    key = "default"
if 'value' not in globals():
    value = None

def get_sticky_key(user_key):
    """Build unique sticky key from user key and component GUID"""
    return "{0}{1}_{2}".format(STICKY_PREFIX, user_key or "default", comp_guid)

try:
    k = get_sticky_key(key)
    reset_flag = bool(reset) if 'reset' in globals() and reset else False

    if reset_flag:
        sc.sticky[k] = []

    current = sc.sticky.get(k, [])

    if value is not None and str(value).strip():
        current.append(str(value).strip())
        sc.sticky[k] = current

    stored = list(current)
    count = len(stored)

except Exception as e:
    print("Error: {}".format(str(e)))
```

## Template 4: Class-Based Component

Complex component using classes for organization (follows project style).

```python
"""
GH Python (IronPython)
Description: Process connections between panels across floors.

Inputs:
    connections : DataTree[str] — connection data per floor
    panels      : List[str]    — panel identifiers

Outputs:
    result      : DataTree[str] — processed connections
    debug       : List[str]     — debug info
"""

import Rhino.Geometry as rg
import Grasshopper as gh
import System

ghenv.Component.Message = "ConnectionProcessor v0.01"

TOLERANCE = 0.01


class ConnectionProcessor(object):
    """Process and validate panel connections"""

    def __init__(self, panels):
        self.panels = panels or []
        self.debug = []

    def validate_panel(self, panel_id):
        """Check if panel ID exists in panel list"""
        return panel_id in self.panels

    def process_branch(self, branch_items):
        """Process single branch of connection data"""
        results = []
        for item in branch_items:
            s = str(item).strip()
            if not s:
                continue
            if self.validate_panel(s):
                results.append(s)
            else:
                self.debug.append("Unknown panel: {}".format(s))
        return results


result = gh.DataTree[System.String]()
debug = []

try:
    processor = ConnectionProcessor(panels)

    if connections is not None and hasattr(connections, 'Branches'):
        for i, branch in enumerate(connections.Branches):
            path = gh.Kernel.Data.GH_Path(i)
            processed = processor.process_branch(branch)
            result.AddRange(processed, path)

    debug = processor.debug
    n_branches = (getattr(connections, "BranchCount", None)
                  if connections is not None else 0) or 0
    debug.append("Processed {} branches".format(n_branches))

except Exception as e:
    debug.append("Error: {}".format(str(e)))
```

## Template 5: JSON File I/O Component

Component that reads/writes JSON files (common in this project).

```python
"""
GH Python (IronPython)
Description: Read JSON file and extract specific keys.

Inputs:
    file_path : str       — path to JSON file
    keys      : List[str] — keys to extract (empty = all)

Outputs:
    values : List[str] — extracted values as JSON strings
    ok     : bool      — success flag
    error  : str       — error message if failed
"""

import json
import os

ghenv.Component.Message = "JsonReader v0.01"

ok = False
values = []
error = ""

def read_json_file(path):
    """Read and parse JSON from file path"""
    if not os.path.isabs(path):
        raise Exception("Path must be absolute: {}".format(path))
    if not os.path.exists(path):
        raise Exception("File not found: {}".format(path))
    with open(path, 'rb') as f:
        raw = f.read()
    return json.loads(raw.decode('utf-8'))

try:
    if not file_path or not str(file_path).strip():
        error = "No file path provided"
    else:
        data = read_json_file(str(file_path).strip())

        if keys and len(keys) > 0:
            for k in keys:
                k_str = str(k).strip()
                if k_str in data:
                    val = data[k_str]
                    values.append(json.dumps(val, ensure_ascii=False))
                else:
                    values.append("")
        else:
            values.append(json.dumps(data, ensure_ascii=False, indent=2))

        ok = True

except Exception as e:
    ok = False
    error = str(e)
```

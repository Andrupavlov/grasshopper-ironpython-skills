# grasshopper-ironpython

A Claude Code skill for writing IronPython 2.7 scripts in Rhino Grasshopper.

## What this skill covers

- **IronPython 2.7 constraints** — forbidden syntax, safe alternatives, quirks
- **Component file structure** — docstring format, import order, entry point pattern
- **Rhino.Geometry API** — Point3d, Vector3d, Line, Plane, BoundingBox, Brep, transforms, intersections
- **Grasshopper DataTree** — creation, iteration, typed generics, path operations
- **Input validation** — safe globals() checks, NameError-safe patterns
- **sc.sticky** — persistent state between recalculations
- **Error reporting** — runtime messages, component status, ExpireSolution
- **Style rules** — naming conventions, anti-patterns

## Files

| File | Contents |
|------|----------|
| `SKILL.md` | Main skill definition — constraints, patterns, style rules |
| `reference.md` | Rhino.Geometry and Grasshopper API quick reference |
| `examples.md` | 5 ready-to-use component templates |

## How to use

### Option A — copy into your project

```
your-project/
└── .agents/
    └── skills/
        └── grasshopper-ironpython/
            ├── SKILL.md
            ├── examples.md
            └── reference.md
```

### Option B — reference in CLAUDE.md

```markdown
# CLAUDE.md
See skill: grasshopper-ironpython
Skill path: .agents/skills/grasshopper-ironpython/SKILL.md
```

### Option C — install via skills CLI

```bash
npx skills install grasshopper-ironpython
```

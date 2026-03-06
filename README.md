# grasshopper-ironpython-skills

A collection of skills for writing IronPython 2.7 scripts in Rhino Grasshopper.

Each skill is a self-contained Markdown file with a YAML frontmatter header (`name`, `description`) followed by reference material and patterns. Skills can be used with any AI agent or tool that supports skill/context injection.

## Skills

| Skill | Path | Description |
|-------|------|-------------|
| `grasshopper-ironpython-base` | `skills/grasshopper-ironpython-base/` | Core constraints, component structure, DataTree, API reference |

More skills coming.

## Structure

```
grasshopper-ironpython-skills/
└── skills/
    └── grasshopper-ironpython-base/
        ├── SKILL.md       ← main skill definition
        ├── examples.md    ← 5 component templates
        └── reference.md   ← Rhino.Geometry + GH API reference
```

Each skill folder contains:
- `SKILL.md` — frontmatter + instructions/reference for the AI
- supporting files referenced from `SKILL.md` (examples, API docs, etc.)

## What `grasshopper-ironpython-base` covers

- **IronPython 2.7 constraints** — forbidden syntax, safe alternatives
- **Component file structure** — docstring format, import order, entry point pattern
- **Rhino.Geometry API** — Point3d, Vector3d, Line, Plane, BoundingBox, Brep, transforms, intersections
- **Grasshopper DataTree** — creation, iteration, typed generics, path operations
- **Input validation** — safe `globals()` checks, NameError-safe patterns
- **sc.sticky** — persistent state between recalculations
- **Error reporting** — runtime messages, component status, `ExpireSolution`
- **Style rules** — naming conventions, anti-patterns

## How to use

Copy the skill folder into your project alongside your agent configuration:

```
your-project/
└── skills/
    └── grasshopper-ironpython-base/
        ├── SKILL.md
        ├── examples.md
        └── reference.md
```

Or clone/submodule this repo and point your agent config at the relevant `SKILL.md`.

## License

MIT

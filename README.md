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

### Option 1 — Copy manually

Download or copy the skill folder into your project:

```
your-project/
└── skills/
    └── grasshopper-ironpython-base/
        ├── SKILL.md
        ├── examples.md
        └── reference.md
```

### Option 2 — Git clone

Clone the whole collection and reference the skill you need:

```bash
git clone https://github.com/Andrupavlov/grasshopper-ironpython-skills.git
```

### Option 3 — Git submodule

Add as a submodule to keep skills in sync with upstream:

```bash
git submodule add https://github.com/Andrupavlov/grasshopper-ironpython-skills.git skills
git submodule update --init
```

To update later:

```bash
git submodule update --remote
```

### Option 4 — Sparse checkout (single skill only)

Pull only one skill folder without the full repo:

```bash
git clone --no-checkout --depth=1 https://github.com/Andrupavlov/grasshopper-ironpython-skills.git
cd grasshopper-ironpython-skills
git sparse-checkout init --cone
git sparse-checkout set skills/grasshopper-ironpython-base
git checkout
```

## License

MIT

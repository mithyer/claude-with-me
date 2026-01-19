# Claude With Me

## For Claude: Read This First

When you read this file, follow these steps in order:

### Step 1: Load Conventions
Read `ios-conventions.md` in this same directory and apply all conventions to the current session.

### Step 2: Load Project Context
Read `../project-context/README.md` to understand available projects and their context.

### Step 3: Ask User
Ask the user: **"Which project will we be working on?"** and list the available projects from project-context.

### Step 4: Load Project Details
Once the user specifies a project, read that project's README from `../project-context/{ProjectName}/README.md` to understand its architecture and current state.

---

## What You'll Know After Loading

After completing the steps above, you should understand:
- Message prefixes (`[fix]`, `[imp]`, `[add]`, `[review]`, etc.)
- Scope limiters (`<dir:>`, `<file:>`, `<class:>`, `<func:>`)
- Swift/iOS coding standards and best practices
- The specific project's architecture and context

---

## Quick Reference

| Prefix | Action |
|--------|--------|
| `[fix]` | Bug fix |
| `[imp]` | Code improvement |
| `[add]` | New feature |
| `[rm]` | Remove feature/code |
| `[review]` | Code review (read-only) |
| `[search]` | Search codebase |
| `[add-note]` | Add documentation |
| `[file-add]<File>` | Create new file |
| `[file-rm]<File>` | Delete file |
| `[file-rn]<A,B>` | Rename file |
| `[:read]` suffix | Dry run, no changes |

Scope: `<dir:path>`, `<file:Name.swift:10-20>`, `<class:ClassName>`, `<func:funcName>`

---

## Related Directories

```
../
├── claude-with-me/          # This repo - conventions & standards
│   ├── README.md            # This file (entry point)
│   └── ios-conventions.md   # Detailed conventions
│
└── project-context/         # Project-specific context & notes
    ├── README.md            # Overview of all projects
    └── {ProjectName}/       # Per-project context
        └── README.md        # Project architecture & state
```

---

*Version: 1.2 | Updated: 2026-01-19*

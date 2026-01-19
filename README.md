# Claude With Me

Cross-project collaboration standards for iOS development with Claude Code.

## Purpose

This repository contains shared conventions, standards, and guidelines for collaborating with Claude on iOS projects. By maintaining these standards in a separate repository, they can be easily applied across multiple iOS projects.

## Contents

### `ios-conventions.md`
The main conventions document covering:
- Project structure standards
- Swift coding style
- Architecture patterns
- Testing guidelines
- Git workflow
- Communication patterns with Claude
- iOS best practices

### `.claude/commands/`
Custom slash commands for Claude Code:
- `/load-conventions` - Load iOS conventions into current session
- `/review-swift` - Comprehensive Swift code review

## Usage

### In New Projects

1. When starting work on an iOS project, use the load conventions command:
   ```
   /load-conventions
   ```

2. Claude will read and apply these standards to all interactions

### Updating Conventions

1. Edit `ios-conventions.md` as your team's practices evolve
2. Commit and push changes
3. Pull updates in other projects

### Cross-Project Consistency

By referencing this repository across multiple iOS projects, you ensure:
- Consistent code style
- Common architecture patterns
- Unified communication methods
- Shared best practices

## Structure

```
claude-with-me/
├── README.md                 # This file
├── ios-conventions.md       # Main conventions document
└── .claude/
    └── commands/            # Custom slash commands
        ├── load-conventions.md
        └── review-swift.md
```

## Version

Current Version: 1.0
Last Updated: 2026-01-19

## License

See LICENSE file for details.

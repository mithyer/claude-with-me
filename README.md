# Claude With Me

## For Claude: Read This First

When you read this file, follow these steps in order:

### Step 1: Load Project Context
Read `../project-context/README.md` to understand available projects and their context.

### Step 2: Ask User
Ask the user: **"Which project will we be working on?"** and list the available projects from project-context.

### Step 3: Load Project Details
Once the user specifies a project, read that project's README from `../project-context/{ProjectName}/README.md` to understand its architecture and current state.

### Step 4: Load Platform Conventions (if needed)
Based on project type, load the appropriate conventions file:
- iOS/Swift: `ios-conventions.md`
- (Future: `backend-conventions.md`, `web-conventions.md`, etc.)

---

## Code Modification Modes

**Claude will estimate code changes before starting:**

| Estimated Lines | Action |
|-----------------|--------|
| **≤200 lines** | "代码量约 X 行，直接执行" - proceed without asking |
| **>200 lines** | Offer mode selection below |

### Mode 1: Scoped Mode (LIMIT)
- Each change limited to **≤200 lines** of code added/removed
- If exceeds 200 lines, use **Skeleton First** approach (see below)
- Create task tracking files in `../project-context/{Project}/`
- Link code comments to task files

### Mode 2: Freeform Mode (FREE)
- No line limits
- Make changes as needed to complete the task
- Suitable for well-defined changes where full context is clear

---

### Skeleton First Approach

When code exceeds 200 lines, Claude builds **incrementally**:

**Pass 1 - Structure (搭架子):**
```swift
class AuthService {
    private let networkClient: NetworkClient

    init(networkClient: NetworkClient) {
        // TODO: Store dependency and configure
    }

    func login(email: String, password: String) async throws -> User {
        // TODO: Validate input format
        // TODO: Call network API
        // TODO: Parse and return response
        fatalError("Not implemented")
    }

    func logout() async {
        // TODO: Clear stored tokens
        // TODO: Notify observers
    }
}
```

**Pass 2+ - Implementation (填内容):**
- Claude fills TODO sections incrementally
- Each pass stays within **≤200 lines**
- User can review structure before implementation

**Benefits:**
- ✅ Always compilable (fatalError placeholders)
- ✅ Architecture review before implementation
- ✅ Easy to track progress via TODOs
- ✅ Lower risk of rework

---

## Command Prefixes

Use these prefixes at the start of your messages to indicate intent:

| Prefix | Action | Modifies Code |
|--------|--------|---------------|
| `[fix]` | Bug fix | Yes |
| `[imp]` | Code improvement | Yes |
| `[add]` | New feature | Yes |
| `[rm]` | Remove feature/code | Yes |
| `[review]` | Code review | No |
| `[search]` | Search codebase | No |
| `[read]` | Preview changes (dry run) | No |
| `[think]` | Deep thinking & exploration | No |
| `[doit]` | Execute previous read/think result | Yes |
| `[check]` | Review resolved conflicts (:keep) | No |
| `[again]` | Resume/retry interrupted operation | Yes |
| `[git]` | Git operations (commit, push, etc.) | No |
| `[lite]` | Low-token mode (I provide, you execute) | Partial |
| `[add-note]` | Add documentation | Yes |
| `[file-add]<File>` | Create new file | Yes |
| `[file-rm]<File>` | Delete file | Yes |
| `[file-rn]<A,B>` | Rename file | Yes |
| `[file-mv]<A,B>` | Move file (alias for rn) | Yes |
| `[list-cmd]` | List all commands | No |

### Scope Limiting

Add scope specifiers after the prefix tag:

```
[prefix]<dir:path file:FileA:10-20 class:ClassA func:functionName> description
```

**Parameters:**
- `dir:` - Limit to specific directory path
- `file:` - Limit to specific files (with optional line numbers: `file:Name.swift:10-20`)
- `class:` - Limit to specific classes
- `func:` - Limit to specific functions/methods

**Examples:**
- `[fix]<file:ConnectionManager.swift> Memory leak in connection handling`
- `[imp]<class:UserViewModel,ProfileViewModel> Better error handling`
- `[add]<dir:Features/Auth> Biometric authentication support`
- `[review]<file:Service.swift:100-150 func:fetchUser> Check this area`

### Command Modifiers

Modifiers can be added to any command with `:modifier` syntax. **Multiple modifiers can be combined.**

| Modifier | Effect |
|----------|--------|
| `:keep` | Insert conflict markers, user resolves |
| `:log` | Log changes to project-context |

**Combined example:** `[fix:keep:log]<file:Manager.swift> Memory leak`

---

#### `:keep` - Conflict Marker Mode

```
[fix:keep] Memory leak in ConnectionManager
[imp:keep]<func:connect> Refactor to async/await
```

Claude will insert **git-style conflict markers** instead of directly replacing code:

```swift
<<<<<<< BEFORE
func connect() {
    startConnection()
}
=======
func connect() async throws {
    try await startConnection()
}
>>>>>>> CLAUDE: Refactored to async/await
```

**Workflow:**
1. Claude inserts conflict markers for each change
2. User resolves conflicts in IDE (accept/reject/modify)
3. User says `[check]` or "检查"
4. Claude reviews the resolved code
5. Repeat until both satisfied

**Best for:** Critical code, learning, uncertain changes.

---

#### `:log` - Change Logging Mode

```
[fix:log] Memory leak in ConnectionManager
[add:log] New authentication feature
```

Claude will log all changes to `../project-context/{Project}/CHANGES/`:

```
../project-context/{Project}/CHANGES/
└── 2026-01-20-fix-memory-leak.md
```

**Log file format:**
```markdown
# Fix: Memory Leak in ConnectionManager

## Date
2026-01-20

## Files Modified
- `Common/Manager/ConnectionManager.swift:45-60`

## Reasoning
The closure was capturing `self` strongly, causing a retain cycle...

## Changes
### ConnectionManager.swift
- Line 48: Changed `{ self.handle() }` to `{ [weak self] in self?.handle() }`
- Line 55: Added `deinit` to verify deallocation

## Notes
Consider auditing other managers for similar issues.
```

**Best for:** Tracking decisions, team communication, future reference.

---

#### Combined Modifiers

```
[fix:keep:log] Memory leak - insert markers AND log changes
[imp:read:log] Analyze improvements and log plan (no changes)
```

Order doesn't matter: `[fix:keep:log]` = `[fix:log:keep]`

---

## Command Details

### `[fix]` - Bug Fix
```
[fix] The app crashes when tapping login button
[fix]<file:NetworkManager.swift:45-80> Fix error handling
```
Claude will: Focus on root cause, add defensive checks, consider edge cases.

### `[imp]` - Code Improvement
```
[imp] This method is too long and complex
[imp]<func:connectDevice> Better error handling
```
Claude will: Apply best practices, optimize performance, improve readability.

### `[add]` - New Feature
```
[add] Add biometric authentication
[add]<dir:Features/Settings> Dark mode support
```
Claude will: Design architecture first, follow existing patterns, implement with proper error handling.

### `[rm]` - Remove Feature/Code
```
[rm] Remove deprecated authentication method
[rm]<file:LegacyAPI.swift,OldManager.swift> Remove old code
```
Claude will: Check dependencies, remove carefully, clean up related resources.

### `[review]` - Code Review (Read-Only)
```
[review]<file:ConnectionManager.swift> Check for memory leaks
[review]<class:NetworkClient> Review error handling
```
Claude will: Analyze issues, check thread safety, suggest improvements without implementing.

### `[search]` - Search Codebase (Read-Only)
```
[search] Where is Bluetooth connection handled?
[search] All usages of deprecated API
```
Claude will: Search and list all relevant files/locations with context.

### `[read]` - Preview Changes (Read-Only)
```
[read:fix] Memory leak in ConnectionManager
[read:add] New authentication feature
[read:imp]<class:NetworkManager> Refactor error handling
```
Claude will:
- ✅ Analyze code and show **exactly what would be changed**
- ✅ Provide specific code examples of the modifications
- ✅ List affected files and line numbers
- ❌ **NEVER modify any files** - preview only

#### Targeted Preview with `[read:cmd]`

| Command | Preview Focus |
|---------|---------------|
| `[read:fix]` | What code changes would fix this bug |
| `[read:imp]` | What refactoring would be applied |
| `[read:add]` | What new code would be added |
| `[read:rm]` | What would be deleted |

### `[think]` - Deep Thinking & Exploration (Read-Only)
```
[think] How should we architect the offline sync feature?
[think]<class:ConnectionManager> What could cause the intermittent disconnections?
```
Claude will:
- **Go deeper**: Analyze root causes, not just symptoms
- **Go broader**: Consider multiple approaches, alternatives, trade-offs
- **Be divergent**: Explore edge cases, future implications, architectural impacts
- **Question assumptions**: Challenge existing design decisions if needed
- **Connect dots**: Link to related systems, patterns, or potential side effects
- ❌ **NEVER modify any files** - thinking only

#### Targeted Thinking with `[think:cmd]`

| Command | Focus Area |
|---------|------------|
| `[think:fix]` | Root cause analysis, why bug occurs, potential fixes, side effects |
| `[think:imp]` | Improvement strategies, refactoring approaches, performance gains |
| `[think:add]` | Feature design, architecture options, implementation trade-offs |
| `[think:rm]` | Removal impact, dependencies, migration strategy |

**Examples:**
```
[think:fix] App crashes on login - what could be the root causes?
[think:imp]<class:NetworkManager> How can we improve error handling?
[think:add] Offline sync feature - what are the architectural options?
[think:rm] Legacy auth system - what would be affected by removal?
```

---

#### `[read]` vs `[think]` Comparison

Both are **read-only** but serve different purposes:

| | `[read]` | `[think]` |
|--|----------|-----------|
| **Purpose** | Preview specific changes | Explore & analyze deeply |
| **Output** | "I would change X to Y" | "Here are possibilities/risks/trade-offs" |
| **Focus** | Concrete, convergent | Abstract, divergent |
| **Depth** | Normal | Deeper & broader |
| **When to use** | Know what you want, preview first | Uncertain, need exploration |

**Example:**
- `[read:fix] Memory leak` → "Change line 48: `self` → `[weak self]`"
- `[think:fix] Memory leak` → "3 possible causes... Approach A vs B... Similar issues elsewhere?... Design reconsideration..."

### `[doit]` - Execute Previous Analysis

Execute the result from a previous `[read]` or `[think]` command.

```
[doit]           # Execute the analyzed solution
[doit] A         # Execute option A (if multiple options)
[doit:keep]      # Execute with conflict markers
[doit:log]       # Execute and log changes
[doit:keep:log]  # Both
```

**Workflow:**
```
User: [think:fix] Memory leak in ConnectionManager
Claude: Found 3 possible approaches:
        A) Add [weak self] to closure
        B) Refactor to use delegation pattern
        C) Use Combine instead of closure

User: [doit] A
Claude: (executes option A)
```

**If single solution:** Claude executes immediately.
**If multiple options:** Claude asks user to choose (A/B/C or describe).

**Modifiers work with `[doit]`:**
- `[doit:keep]` - Execute with conflict markers
- `[doit:log]` - Execute and log to CHANGES/

### `[check]` - Review Resolved Conflicts

After resolving conflicts from `:keep` mode, use `[check]` for Claude to review.

```
[check]          # Review my conflict resolutions
[check] Done     # Finished resolving, please verify
```

**Workflow:**
1. `[fix:keep] Memory leak` → Claude inserts conflict markers
2. User resolves conflicts in IDE
3. `[check]` → Claude reviews the resolutions
4. Repeat until both satisfied

### `[again]` - Resume or Retry Interrupted Operation

Resume or retry a previously interrupted command.

```
[again]            # Smart: check state, decide continue or retry
[again:continue]   # Continue from where interrupted (skip completed)
[again:retry]      # Full retry (ignore previous progress)
```

**Smart mode behavior (`[again]`):**
1. Recall the last interrupted command
2. Check current state (git diff, file changes)
3. Detect user's manual modifications
4. Report: "Detected you changed X, now continuing..."
5. Execute appropriately

**Examples:**
```
User: [fix] Memory leak in ConnectionManager
Claude: (starts fixing, step 1 done, step 2 in progress...)
User: (interrupts, makes manual changes)
User: [again]
Claude: "Last command: [fix] Memory leak
        Detected: You modified line 52-55
        Continuing with remaining steps..."
```

```
User: [again:retry]    # Start over completely
User: [again:continue] # Skip completed parts
```

### `[git]` - Git Operations

Perform git-related operations.

```
[git] status                    # Show git status
[git] commit                    # Help write commit message and commit
[git] push                      # Push to remote
[git] pr                        # Create pull request
[git] diff                      # Show changes
[git] log                       # Show recent commits
```

**Common workflows:**

```
[git] commit           # Claude checks status, diff, writes commit message
[git] commit --amend   # Amend last commit (with safety checks)
[git] pr               # Create PR with summary from commits
[git] sync             # Pull and rebase from main
```

**Safety:** Claude will warn before destructive operations (force push, hard reset, etc.)

### `[lite]` - Low-Token Mode

Reduce token consumption by shifting execution to user.

```
[lite] Add unit tests for AuthService
[lite:add] New login feature
```

**How it works:**

| Task | Claude Does | User Does |
|------|-------------|-----------|
| Analysis/Design | ✅ Think & plan | Review |
| Code changes | ✅ Provide code/diff | Paste into IDE |
| Run tests/build | ❌ Skip | Run & report result |
| Search codebase | ❌ Skip | Search & tell location |
| Git operations | ❌ Skip | Execute commands |

**Workflow example:**
```
User: [lite:add] Add validation to login form

Claude: Here's the implementation:

1. Add to LoginViewModel.swift (line 45):
   ```swift
   func validate() -> Bool {
       guard !email.isEmpty else { return false }
       guard password.count >= 8 else { return false }
       return true
   }
   ```

2. Update LoginView.swift (line 78):
   ```swift
   Button("Login") {
       guard viewModel.validate() else { return }
       // ...
   }
   ```

Please paste these changes, run tests, and tell me the result.
```

**When to use:**
- Long coding sessions (preserve context)
- Large file changes (avoid Write tool overhead)
- Test/build cycles (you see output faster anyway)

**Can combine:** `[lite:fix]`, `[lite:add]`, `[lite:imp]`

### `[add-note]` - Add Documentation
```
[add-note]<class:ConnectionManager> Add class documentation
[add-note]<func:connect> Explain the connection flow
```
Claude will: Add clear comments explaining WHY, not just WHAT.

### `[file-add]<File>` - Create New File
```
[file-add]<AuthService.swift> Create authentication service
[file-add]<UserViewModel.swift> Add view model
```
Claude will: Create file with appropriate boilerplate and structure.

### `[file-rm]<File>` - Delete File
```
[file-rm]<LegacyManager.swift> Remove deprecated manager
```
Claude will: Check for usages first, warn if still referenced.

### `[file-rn]<Old,New>` - Rename File
```
[file-rn]<UserManager.swift,UserService.swift> Better naming
```
Claude will: Update all imports and references in other files.

---

## Mode 1: Task Tracking System

### Directory Structure
```
../project-context/{ProjectName}/
├── README.md              # Project overview
├── TODO/                  # New features & improvements
│   ├── {task-name}.md     # Active task
│   └── _archive/          # Completed tasks (100%)
└── FIXME/                 # Bug fixes
    ├── {bug-name}.md      # Active fix
    └── _archive/          # Completed fixes (100%)
```

### Task File Format
```markdown
# {Task Title}

## Status
- [ ] In Progress | [x] Paused | [x] Completed
- Progress: 45%

## Description
Brief description of what needs to be done.

## Code References
- `path/to/File.swift:123` - // TODO: {task-name} - description

## Subtasks
- [x] Step 1 completed
- [ ] Step 2 in progress
- [ ] Step 3 pending

## Notes
Any relevant notes or decisions.
```

### Code Comment Linking
```swift
// TODO: {task-name} - Brief description
// Links to: ../project-context/{Project}/TODO/{task-name}.md

// FIXME: {bug-name} - Brief description
// Links to: ../project-context/{Project}/FIXME/{bug-name}.md
```

### Task Lifecycle
1. **Create** - New task file when starting work
2. **Update** - Track progress (0-100%)
3. **Pause** - Mark as paused, can resume later
4. **Complete** - 100% done, move to `_archive/`

---

## Best Practices for Requests

**DO:**
- Provide context: "In the `UserProfileController`, I need to..."
- Specify file path if known
- Share error messages or logs in full

**DON'T:**
- Say "fix the code" without specifying what's wrong
- Assume Claude knows current state without reading files

---

## Related Directories

```
../
├── claude-with-me/          # This repo - conventions & standards
│   ├── README.md            # This file (entry point)
│   └── ios-conventions.md   # iOS-specific conventions
│
└── project-context/         # Project-specific context & notes
    ├── README.md            # Overview of all projects
    └── {ProjectName}/
        ├── README.md        # Project architecture
        ├── TODO/            # Feature tasks (Mode 1: LIMIT)
        ├── FIXME/           # Bug fix tasks (Mode 1: LIMIT)
        └── CHANGES/         # Change logs (:log modifier)
```

---

*Version: 2.0 | Updated: 2026-01-20*

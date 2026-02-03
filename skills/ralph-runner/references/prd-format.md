# PRD Format Specification

## Basic Structure

```json
{
  "project": "MyApp",
  "branchName": "ralph/feature-name",
  "description": "Feature description",
  "userStories": []
}
```

## Fields

| Field | Required | Description |
|-------|----------|-------------|
| `project` | No | Project name for display |
| `branchName` | Yes | Git branch Ralph creates/uses |
| `description` | No | Overall feature description |
| `userStories` | Yes | Array of user stories |

## User Story Format

```json
{
  "id": "US-001",
  "title": "Add login form",
  "description": "As a user, I want to log in with email and password.",
  "acceptanceCriteria": [
    "Email and password fields present",
    "Validation errors shown",
    "Typecheck passes"
  ],
  "priority": 1,
  "passes": false,
  "notes": ""
}
```

### User Story Fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Unique identifier (e.g., US-001) |
| `title` | Yes | Short title |
| `description` | No | Full description |
| `acceptanceCriteria` | No | Array of criteria to pass |
| `priority` | Yes | Lower = higher priority (1 runs first) |
| `passes` | Yes | Set to `true` when complete |
| `notes` | No | Notes from implementation |

## Complete Example

```json
{
  "project": "TaskApp",
  "branchName": "ralph/task-priority",
  "description": "Add priority levels to tasks",
  "userStories": [
    {
      "id": "US-001",
      "title": "Add priority field to database",
      "description": "As a developer, I need to store task priority.",
      "acceptanceCriteria": [
        "Add priority column: 'high' | 'medium' | 'low'",
        "Default to 'medium'",
        "Migration runs successfully",
        "Typecheck passes"
      ],
      "priority": 1,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-002",
      "title": "Display priority on task cards",
      "description": "As a user, I want to see task priority at a glance.",
      "acceptanceCriteria": [
        "Colored badge: red=high, yellow=medium, gray=low",
        "Visible without hovering",
        "Typecheck passes"
      ],
      "priority": 2,
      "passes": false,
      "notes": ""
    },
    {
      "id": "US-003",
      "title": "Add priority selector to edit",
      "description": "As a user, I want to change a task's priority.",
      "acceptanceCriteria": [
        "Dropdown in task edit modal",
        "Shows current priority as selected",
        "Saves on selection change",
        "Typecheck passes"
      ],
      "priority": 3,
      "passes": false,
      "notes": ""
    }
  ]
}
```

## Story Sizing Guidelines

Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"

## Tips

1. **One concern per story** - Single responsibility
2. **Include typecheck** - Always add "Typecheck passes"
3. **Order by priority** - Lower numbers run first
4. **Browser verification** - Add "Verify in browser" for UI stories

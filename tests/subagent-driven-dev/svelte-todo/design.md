# Svelte Todo List - Design

## Overview

Todo list app, Svelte. Create/complete/delete todos. localStorage persist.

## Features

- Add todos
- Toggle complete/incomplete
- Delete todos
- Filter: All / Active / Completed
- Clear completed
- localStorage persist
- Remaining count

## User Interface

```
┌─────────────────────────────────────────┐
│  Svelte Todos                           │
├─────────────────────────────────────────┤
│  [________________________] [Add]       │
├─────────────────────────────────────────┤
│  [ ] Buy groceries                  [x] │
│  [✓] Walk the dog                   [x] │
│  [ ] Write code                     [x] │
├─────────────────────────────────────────┤
│  2 items left                           │
│  [All] [Active] [Completed]  [Clear ✓]  │
└─────────────────────────────────────────┘
```

## Components

```
src/
  App.svelte           # Main app, state management
  lib/
    TodoInput.svelte   # Text input + Add button
    TodoList.svelte    # List container
    TodoItem.svelte    # Single todo with checkbox, text, delete
    FilterBar.svelte   # Filter buttons + clear completed
    store.ts           # Svelte store for todos
    storage.ts         # localStorage persistence
```

## Data Model

```typescript
interface Todo {
  id: string;        // UUID
  text: string;      // Todo text
  completed: boolean;
}

type Filter = 'all' | 'active' | 'completed';
```

## Acceptance Criteria

1. Add todo: type + Enter or click Add
2. Toggle completion: click checkbox
3. Delete todo: click X
4. Filter buttons show correct subset
5. "X items left" = incomplete count
6. "Clear completed" removes all completed
7. Persist across refresh (localStorage)
8. Empty state shows helpful message
9. All tests pass
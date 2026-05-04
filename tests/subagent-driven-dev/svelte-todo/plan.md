# Svelte Todo List - Implementation Plan

Execute plan via `superpowers:subagent-driven-development` skill.

## Context

Todo list app w/ Svelte. See `design.md` for full spec.

## Tasks

### Task 1: Project Setup

Create Svelte project w/ Vite.

**Do:**
- Run `npm create vite@latest . -- --template svelte-ts`
- Install deps w/ `npm install`
- Verify dev server work
- Clean default Vite template from App.svelte

**Verify:**
- `npm run dev` start server
- App show minimal "Svelte Todos" heading
- `npm run build` succeed

---

### Task 2: Todo Store

Svelte store for todo state.

**Do:**
- Create `src/lib/store.ts`
- Define `Todo` interface w/ id, text, completed
- Writable store, initial empty array
- Export fns: `addTodo(text)`, `toggleTodo(id)`, `deleteTodo(id)`, `clearCompleted()`
- Create `src/lib/store.test.ts` w/ tests per fn

**Verify:**
- Tests pass: `npm run test` (install vitest if needed)

---

### Task 3: localStorage Persistence

Persistence layer for todos.

**Do:**
- Create `src/lib/storage.ts`
- Impl `loadTodos(): Todo[]` and `saveTodos(todos: Todo[])`
- JSON parse err → return empty array
- Wire to store: load on init, save on change
- Tests for load/save/err

**Verify:**
- Tests pass
- Manual: add todo, refresh, todo persist

---

### Task 4: TodoInput Component

Input component for add todos.

**Do:**
- Create `src/lib/TodoInput.svelte`
- Text input bound to local state
- Add button → `addTodo()` + clear input
- Enter key also submit
- Disable Add when input empty
- Component tests

**Verify:**
- Tests pass
- Renders input + button

---

### Task 5: TodoItem Component

Single todo item component.

**Do:**
- Create `src/lib/TodoItem.svelte`
- Props: `todo: Todo`
- Checkbox toggle completion (`toggleTodo`)
- Text strikethrough when completed
- Delete button (X) → `deleteTodo`
- Component tests

**Verify:**
- Tests pass
- Renders checkbox, text, delete button

---

### Task 6: TodoList Component

List container component.

**Do:**
- Create `src/lib/TodoList.svelte`
- Props: `todos: Todo[]`
- Render TodoItem per todo
- Show "No todos yet" when empty
- Component tests

**Verify:**
- Tests pass
- Renders list of TodoItems

---

### Task 7: FilterBar Component

Filter + status bar component.

**Do:**
- Create `src/lib/FilterBar.svelte`
- Props: `todos: Todo[]`, `filter: Filter`, `onFilterChange: (f: Filter) => void`
- Count: "X items left" (incomplete count)
- Three filter buttons: All, Active, Completed
- Active filter visually highlighted
- "Clear completed" button (hide when no completed)
- Component tests

**Verify:**
- Tests pass
- Renders count, filters, clear button

---

### Task 8: App Integration

Wire components in App.svelte.

**Do:**
- Import all components + store
- Filter state (default: 'all')
- Compute filtered todos from filter state
- Render: heading, TodoInput, TodoList, FilterBar
- Pass props to each

**Verify:**
- App renders all components
- Add work
- Toggle work
- Delete work

---

### Task 9: Filter Functionality

Filtering end-to-end.

**Do:**
- Verify filter buttons change displayed todos
- 'all' → all todos
- 'active' → incomplete only
- 'completed' → completed only
- Clear completed removes completed + reset filter if needed
- Integration tests

**Verify:**
- Filter tests pass
- Manual check all filter states

---

### Task 10: Styling and Polish

CSS for usability.

**Do:**
- Style app to match design mockup
- Completed todos: strikethrough + muted color
- Active filter button highlighted
- Input focus styles
- Delete button on hover (or always on mobile)
- Responsive layout

**Verify:**
- App visually usable
- Styles don't break functionality

---

### Task 11: End-to-End Tests

Playwright tests for full user flows.

**Do:**
- Install Playwright: `npm init playwright@latest`
- Create `tests/todo.spec.ts`
- Test flows:
  - Add todo
  - Complete todo
  - Delete todo
  - Filter todos
  - Clear completed
  - Persistence (add, reload, verify)

**Verify:**
- `npx playwright test` pass

---

### Task 12: README

Doc project.

**Do:**
- Create `README.md` w/:
  - Project desc
  - Setup: `npm install`
  - Dev: `npm run dev`
  - Test: `npm test` + `npx playwright test`
  - Build: `npm run build`

**Verify:**
- README accurate
- Instructions work
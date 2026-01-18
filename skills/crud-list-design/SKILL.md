---
name: crud-list-design
description: "Use when designing frontend CRUD list pages - standardizes filter configuration, data flow, state management, and form interactions."
---

# CRUD List Design

## Overview

Framework-agnostic design guide for frontend CRUD list pages. Automatically detects project context and interactively collects requirements to generate standardized design documentation covering filter configuration, data interaction patterns, state management, and form data flows.

**Can be used independently** or **called by other design skills** (like `design-evolution`) as a reference standard for frontend list functionality.

**Output:** Structured Markdown design document saved to `docs/designs/crud-lists/YYYY-MM-DD-<resource-name>-list-design.md`

---

## Trigger Conditions

Use this skill when:
- User says "make a list page", "design a CRUD page", "create a table with filters"
- In `design-evolution` Stage 2 when frontend list functionality is involved
- User mentions "filter", "pagination", "search" in the context of list functionality

---

## Execution Process

### Stage 0: Auto-Detection (Silent)

**Before asking any questions, automatically scan the project:**

1. **Framework Detection:**
   - Check `package.json` for: `vue`, `react`, `next`, `nuxt`, `angular`
   - Scan `tsconfig.json` or `vite.config.*` for framework hints

2. **UI Library Detection:**
   - Scan `package.json` dependencies for: `element-plus`, `ant-design-vue`, `antd`, `@mui/material`, `naive-ui`, `arco-design`

3. **Existing List Page Discovery:**
   - Search for files matching: `*List.vue`, `*List.tsx`, `*-list.vue`, `*-list.tsx`
   - Look in common paths: `src/views/`, `src/pages/`, `pages/`, `app/`

4. **API Pattern Detection:**
   - Find existing API calls to identify request library (axios, fetch, ky, etc.)
   - Identify error handling patterns

**Present findings:**
```
"Based on your project, I detected:
- Framework: Vue 3
- UI Library: Element Plus
- Found 3 existing list pages: src/views/UserList.vue, src/views/OrderList.vue...
- API Library: axios with custom request wrapper

Is this correct? [Yes / Let me clarify]"
```

### Stage 1: Mode Selection & Context Confirmation

**Ask the user:**

1. "What is the resource name for this list page?" (e.g., "User Management", "Order List")

2. "Is this a **standard** list or does it need **advanced** features?"
   - **Standard Mode:** Basic filters, standard pagination, simple CRUD actions
   - **Advanced Mode:** Tree filters, remote search, inline edit, virtual scroll, etc.

**Standard Mode Path:**
- Present a pre-filled template based on detected patterns
- User confirms or adjusts specific fields
- Skip to Stage 5 (State Management) after quick config

**Advanced Mode Path:**
- Continue with detailed Stage 2-4 flow

---

### Stage 2: Filter Configuration Collection

#### Basic Fields

For each filter field, collect:

| Property | Options |
|----------|---------|
| Field name | Open text |
| Label | Open text |
| Type | `single-select` / `multi-select` / `date-range` / `text-input` / `number-range` / `cascading` / `tag-input` |
| Data source | `static` / `db-distinct` / `api` |
| Default value | Open text or "none" |

**Batch Collection Shortcut:**
```
"List all filter fields you need (comma-separated):
e.g., status, category, createdAt, keyword"

Then ask type/source for each one.
```

#### Advanced Field Types

**Tree-structured select:**
- Lazy load children?
- Multi-select? If yes: leaf-only or any node?
- Default expand level?

**Paginated select (large option sets):**
- Page size?
- Support remote search?

**Remote search:**
- Debounce delay (default: 300ms)
- Minimum characters (default: 2)
- Cache results?

**Cascading select (e.g., province → city → district):**
- How many levels?
- Load all at once or lazy?
- Allow selecting non-leaf?

**Number range:**
- Min/Max constraints?
- Step value?
- Unit label?

**Date range with presets:**
- Include presets? (Today, Last 7 days, Last 30 days, Custom)
- Date-only or datetime?
- Default range?

**Tag input (free-form multiple values):**
- Max tags?
- Validate format? (e.g., email, ID)

#### Output: Filter Configuration Table

| Field | Type | Data Source | API/Endpoint | Advanced Config | Default |
|-------|------|-------------|--------------|-----------------|---------|
| status | single-select | static | - | - | "all" |
| category | multi-select | db-distinct | /api/categories/distinct | tree, lazy, leaf-only | - |
| region | cascading | api | /api/regions | 3-level, lazy | - |
| price | number-range | - | - | min:0, step:100, unit:¥ | - |
| createdAt | date-range | - | - | presets, date-only | last 7 days |

---

### Stage 3: List Design

#### Column Definition

| Property | Description |
|----------|-------------|
| Field name | Data path (e.g., `user.name`) |
| Type | `text` / `number` / `date` / `badge` / `image` / `link` / `custom` |
| Sortable | Yes / No |
| Format | Date format, value mapping, number precision |
| Width | Fixed / Auto / Min-width |

**Ask:**
1. "What columns should be displayed?" (batch input)
2. "Which columns need sorting?"
3. "Any columns need special formatting?"

#### Table Features

**Inline Editing:**
- Support inline edit? (Yes / No)
- Which columns are editable?
- Save trigger: on-blur / explicit-save-button / auto-save

**Virtual Scroll:**
- Expected max rows per page? (If >500, recommend virtual scroll)
- Fixed row height or variable?

**Column Customization:**
- Can users show/hide columns?
- Can users reorder columns?
- Persist user preferences? (localStorage / server)

**Row Selection:**
- Support row selection? (None / Single / Multiple)
- Selection persistence across pages?

#### Pagination

- Style: `page/pageSize` or `offset/limit`
- Default page size
- Page size options (e.g., [10, 20, 50, 100])

#### Query Trigger Rules

| Event | Behavior |
|-------|----------|
| Initial load | Query with defaults |
| Filter change | Reset to page 1, query |
| Pagination change | Preserve filters, query |
| Sort change | Reset to page 1, query |

#### Action Column

- Available actions: Edit / Delete / View / Custom...
- Permission mapping for each action
- Confirmation required? (especially for delete)

---

### Stage 4: Form Data Flow Design

**Form Structure:**
- Same form for create/edit with mode param?
- Or separate forms?

**Form Initialization:**
- Create: default values (may include API-fetched defaults)
- Edit: populate from list item or fetch detail API?

**Submission Flow:**
```
Validate (client) → Submit API → Handle Response
                                  ├─ Success: close + refresh + toast
                                  └─ Error: field-level or form-level display
```

**Post-Submit Behavior:**
- Close modal and refresh list?
- Redirect to detail page?
- Stay on form (for "save and continue")?

**Error Display:**
- Field-level errors bound to inputs
- Form-level errors at top
- Network errors with retry option

---

### Stage 5: State Management Design

**State Structure:**
```typescript
interface ListState {
  data: Item[]
  total: number
  filters: FilterValues
  pagination: { page: number; pageSize: number }
  sorting: { field: string; order: 'asc' | 'desc' } | null
  loading: boolean
  error: Error | null
  // Optional advanced state
  selectedRows?: Item[]
  columnVisibility?: Record<string, boolean>
}
```

**Framework Recommendation:**
- React: `useState` + `useReducer`, or Zustand/Jotai
- Vue 3: `reactive()` + `ref()`, or Pinia

**Error Handling Strategy:**
- Preserve previous data on error?
- Show empty state on error?
- Auto-retry configuration?

---

### Stage 6: Edge Cases & UX

**Empty State:**
- Show empty illustration?
- Provide "Create first item" button?
- Different message for "no results" vs "no data"?

**Loading State:**
- Skeleton or spinner?
- Disable filters during load?

**Permission Handling:**
- Hide unauthorized actions?
- Or show disabled with tooltip?

**Batch Operations:**
- Support batch delete/export/status-change?
- Require secondary confirmation?
- Show progress for large batches?

---

## Design Validation Checklist

Before finalizing the design, verify:

- [ ] All filter fields have clear data sources defined
- [ ] API request shape documented: `{ filters, pagination, sorting }`
- [ ] API response shape documented: `{ data, total, page?, pageSize? }`
- [ ] Permission points cover all sensitive actions
- [ ] Error states have user-facing messages
- [ ] Empty state provides actionable guidance
- [ ] Form validation rules are specified
- [ ] Edge cases (empty, error, loading) are addressed

---

## Document Output Structure

### Generated Document Sections

1. **Overview** - Resource name, routes, reference pages
2. **Filter Configuration Table** - All fields with types and sources
3. **API Contract** - Request/Response shapes needed
4. **Table Column Definition** - Fields, types, formatting
5. **Table Features** - Inline edit, virtual scroll, column config
6. **Form Data Flow** - Create/Edit modes, validation, submission
7. **State Management** - State structure, update patterns
8. **Permission Control Points** - Actions and their required permissions
9. **Edge Cases** - Empty, error, loading states
10. **Implementation Snippets** (Optional) - Framework-specific code

### Save Location

```
docs/designs/crud-lists/YYYY-MM-DD-<resource-name>-list-design.md
```

**Versioning:**
- If exists: ask "Overwrite or create new version?"
- New version: append `-v2.md`, `-v3.md`

---

## Optional: Implementation Snippets

If user agrees, generate framework-specific code:

**Vue 3 + Element Plus:**
```typescript
// composables/useUserList.ts
export function useUserList() {
  const filters = reactive<FilterValues>({...})
  const pagination = reactive({ page: 1, pageSize: 20 })
  const { data, loading, error, execute } = useAsyncState(...)
  // ...
}
```

**React + Ant Design:**
```typescript
// hooks/useUserList.ts
export function useUserList() {
  const [filters, setFilters] = useState<FilterValues>({...})
  const { data, isLoading, error } = useQuery(...)
  // ...
}
```

Ask: "Would you like me to generate skeleton code for your framework?"

---

## Key Principles

**Smart Defaults:**
- Auto-detect before asking
- Present findings for confirmation
- Use existing project patterns as templates

**One Question at a Time:**
- Never overwhelm with multiple questions
- Prefer multiple choice over open text
- Batch similar inputs when possible

**Framework Agnostic Design:**
- TypeScript interfaces for data structures
- Pseudocode for logic flows
- Framework-specific code only when requested

**YAGNI:**
- Don't add unrequested features
- Advanced features only when explicitly needed
- Keep design focused on actual requirements

---

## Integration with Other Skills

**Standalone usage:**
- Generate complete design document
- Ask: "Ready to move to implementation?"

**Called by design-evolution:**
- Output becomes part of overall design
- Continue to module breakdown

**After completion:**
- Offer: `writing-plans` for implementation plan
- Offer: `using-git-worktrees` for isolated workspace

---

## Example Interaction (Standard Mode)

```
User: "Design a user management list page"
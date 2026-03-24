---
name: componentize
description: Break up large React files into well-organized components without overdoing it. Use when a React file has grown too large, contains multiple components, or when user asks to split/extract/organize/componentize.
argument-hint: <file to componentize>
---

Break the React file at `$ARGUMENTS` into smaller, well-placed component files. Be pragmatic — only extract what genuinely improves readability. Don't over-split.

## Steps

1. **Read the file** and understand every component and helper defined in it.
2. **Decide what to extract** using the rules below.
3. **Determine where each extracted piece should live** using the placement rules.
4. **Move the code**: create new files, update imports in the original file, and remove the extracted code from it.
5. **Verify** by running the project's typecheck/lint if available.

## Decision heuristic

**Extract when the component has a natural name, a clear boundary, and the parent gets simpler — not just shorter.**

For each candidate, weigh these factors:

1. **Reuse** — Used more than once? Extract. Single-use? Higher bar.
2. **Size** — A 60-line JSX section hiding the page's structure is worth it. A 10-line helper isn't.
3. **Prop-drilling cost** — If extracting means passing 6+ props that were just local variables, the cure is worse than the disease.
4. **Conceptual boundary** — Can you name it naturally? (`WebhookSettings`, `BillingCard`) If not, it probably shouldn't be a component.
5. **State coupling** — If the code reads/writes 3+ pieces of parent state, it's tightly coupled. Extracting just shuffles complexity.
6. **Readability of what remains** — After extraction, is the parent easier to scan? If it becomes a list of opaque `<SectionA /> <SectionB /> <SectionC />` with no visible structure, you've traded one problem for another.

## What to extract

Extract a piece of code into its own file when **any** of these are true:

- It's a named component used only as a child in the main component's JSX (e.g. `Toggle`, `StatCard`, `SettingsSection`).
- It's a substantial helper component (roughly >20 lines of JSX) even if used once.
- It's a custom hook or complex logic block that obscures the main component's flow.

## What NOT to extract

Leave code inline when **any** of these are true:

- It's a tiny helper (<15 lines) used only once and is trivially understandable in context (e.g. a small `Toggle` that's 10 lines).
- Extracting it would require passing 5+ props that are currently just local variables — the prop-drilling cost outweighs the readability gain.
- It's tightly coupled state logic that only makes sense next to the JSX consuming it.
- It's a one-liner render helper or formatting function.

## Where to place extracted components

Follow these conventions based on how the component is used:

### Route-specific components
If the extracted component is only used by one route/page, place it next to that route file:
```
src/routes/dashboard/settings/
  index.tsx              ← main page
  -components/           ← use dash-prefix per TanStack Router convention
    Toggle.tsx
    WebhookSettings.tsx
```

If the route directory doesn't exist yet (i.e. the route is a flat file like `settings.tsx`), first check if converting to a directory route makes sense. If the file is `index.tsx` inside a directory, create `-components/` as a sibling.

### Shared components
If the component is clearly reusable across multiple routes (generic UI primitives like Toggle, Modal, Card), place it in the project's shared components directory (e.g. `src/components/`). Check if a similar component already exists there first — don't duplicate.

### Hooks
Extract hooks to a `hooks/` directory at the appropriate scope (route-local or shared).

## How to split

For each extraction:

1. Create the new file with the component/hook.
2. Move any types, constants, or helpers that are **only** used by the extracted code.
3. Keep shared types/constants in the original file or a shared location.
4. Add proper imports/exports.
5. In the original file, replace the removed code with an import.

## Naming

- File name matches the default export: `Toggle.tsx` exports `Toggle`.
- Use PascalCase for component files, camelCase for hook files.
- Don't add `index.ts` barrel files unless one already exists in the pattern.

## Output

After splitting, provide a summary:

| Extracted | From | To | Reason |
|-----------|------|----|--------|
| `Toggle` | `settings/index.tsx` | `settings/-components/Toggle.tsx` | Generic UI, used 6 times |
| `WebhookSettings` | `settings/index.tsx` | `settings/-components/WebhookSettings.tsx` | 80-line section, self-contained |
| ... | ... | ... | ... |

And note anything you intentionally left inline and why.

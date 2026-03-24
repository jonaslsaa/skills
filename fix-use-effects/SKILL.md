---
name: fix-use-effects
description: Find useEffect calls in React code and refactor unnecessary ones. Use when reviewing React components for code quality, removing unnecessary Effects, or when user asks to clean up effects/hooks.
argument-hint: [file or directory to scan]
---

Find `useEffect` calls in the codebase (or in `$ARGUMENTS` if provided) and evaluate each one against React best practices. Remove or refactor unnecessary Effects. Don't touch useEffects that are *correctly* synchronizing with external systems but do flag them if you have an opinion.

## Steps

1. Search for all `useEffect` usages in the target scope using Grep.
2. For each `useEffect`, read the surrounding component code to understand what the Effect does.
3. Classify each Effect into one of the categories below.
4. For Effects that should be removed or refactored, make the change. For correct Effects, leave them alone.
5. After all changes, run the project's lint/typecheck if available to verify nothing broke.

## Classification & fix rules

### REMOVE — Compute during render
If the Effect updates state that could be derived from props or other state, delete the Effect and compute the value directly during render.

```tsx
// BAD
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(firstName + ' ' + lastName);
}, [firstName, lastName]);

// GOOD
const fullName = firstName + ' ' + lastName;
```

If the computation is expensive, wrap it in `useMemo` instead of `useEffect` + `setState`.

### REMOVE — Move to event handler
If the Effect runs logic that is a direct response to a user action (click, submit, change), move that logic into the event handler. You know exactly what happened in an event handler; by the time an Effect runs, you don't.

```tsx
// BAD
useEffect(() => {
  if (submitted) {
    post('/api/buy', { product });
  }
}, [submitted, product]);

// GOOD — in the onClick/onSubmit handler
function handleSubmit() {
  post('/api/buy', { product });
}
```

### REMOVE — Resetting state on prop change
If the Effect resets state when a prop changes, use a `key` on the component instead.

```tsx
// BAD
useEffect(() => {
  setComment('');
}, [userId]);

// GOOD — parent passes key={userId} so the component remounts
<ProfileEditor key={userId} />
```

### REMOVE — Setting state during render for prop changes
If the Effect sets a piece of state in response to a prop change (not a full reset), set it during rendering instead.

```tsx
// BAD
useEffect(() => {
  setSelection(null);
}, [items]);

// GOOD
const [prevItems, setPrevItems] = useState(items);
if (items !== prevItems) {
  setPrevItems(items);
  setSelection(null);
}
```

(This pattern is rare — usually computing during render or using a key is better.)

### REMOVE — Chains of Effects syncing state
If multiple Effects set state that trigger other Effects in a chain, consolidate into a single event handler or a single computation.

### KEEP — External system synchronization
Effects that synchronize with non-React systems are correct: DOM manipulation, third-party widgets, network subscriptions, browser APIs (IntersectionObserver, ResizeObserver, etc.), WebSocket connections, timers that sync with display state.

### KEEP but IMPROVE — Data fetching
Effects that fetch data are acceptable but should:
- Include a cleanup function to handle race conditions (AbortController or ignore flag).
- Consider whether a framework/library data-fetching solution (TanStack Query, SWR, loader functions) would be better. Suggest the improvement but don't force the migration.

### CONVERT — External mutable sources
If the Effect subscribes to an external mutable store (browser storage, global variable, third-party state), suggest converting to `useSyncExternalStore`.

### EXTRACT — Reusable or component-local Effects
If an Effect is reused or likely to be reused, or if it represents a cohesive piece of synchronization logic, extract it into a custom hook.

## Decision flowchart (use this for each Effect)

1. Can I compute this from props/state? → Compute during render (or `useMemo` if expensive). **Remove Effect.**
2. Does this run because the user did something? → Move to event handler. **Remove Effect.**
3. Am I resetting state when identity changes? → Use `key`. **Remove Effect.**
4. Am I syncing with an external system? → **Keep Effect.** Check cleanup.
5. Am I subscribing to an external mutable source? → Consider `useSyncExternalStore`.
6. Am I fetching data? → **Keep Effect** but ensure cleanup/cancellation. Suggest TanStack Query if appropriate.

## Output

After making changes, provide a summary table:

| File | Effect | Action | Reason |
|------|--------|--------|--------|
| ... | ... | Removed / Kept / Refactored | ... |

## Reference
See full documentation: https://react.dev/learn/you-might-not-need-an-effect

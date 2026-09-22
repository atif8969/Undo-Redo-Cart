# Challenge 4 — Undo / Redo Cart

A shopping cart (plain HTML/CSS/JS, no frameworks) with add, remove,
increase/decrease quantity — all fully undoable and redoable, plus a live
operation history log.

## The idea

Every action becomes a **command object**:
```js
{ apply()  { /* do it */ },
  revert() { /* exactly undo it */ } }
```
Two stacks manage them:

| Stack | Holds |
|---|---|
| `undoStack` | applied commands, most recent on top |
| `redoStack` | undone commands, most recent on top |

- **New action** → `apply()` it, push onto `undoStack`, **clear `redoStack`** (like Word/Git — a new action kills the old "redo future").
- **Undo** → pop `undoStack`, `.revert()`, push onto `redoStack`.
- **Redo** → pop `redoStack`, `.apply()`, push onto `undoStack`.

You only ever undo the *most recently applied* action, so nothing gets
reverted out of order.

## Files

- `challenge4-undo-redo-cart.html` — everything in one file
- Swap the `PRODUCTS` array near the top of the `<script>` for the real
  `data.js` contents before submitting (keep `id`, `name`, `price`).

## Code walkthrough

**State** — three variables drive the whole app:
```js
let cart = [];       // { id, name, price, qty }[]
let undoStack = [];
let redoStack = [];
```

**The four commands** — each returns a `{label, apply, revert}` object:
- `cmdAddProduct(product)` → apply: push item with qty 1. revert: remove by id.
- `cmdRemoveProduct(id)` → saves the item's **index + a copy** before removing, so revert re-inserts it in the exact same spot with `cart.splice(index, 0, snapshot)`.
- `cmdIncreaseQty(id)` / `cmdDecreaseQty(id)` → apply/revert just do `qty += 1` / `qty -= 1` in opposite directions.

**`doAction(command)`** — the only function allowed to change `cart`. Runs
`apply()`, pushes to `undoStack`, wipes `redoStack`, re-renders. Every
button click goes through this.

**`undo()` / `redo()`** — pop from one stack, apply/revert, push to the
other. Guard clause returns early if the stack is empty (this is also what
disables the buttons).

**`render()`** — called after every change. Rebuilds the catalog, cart
rows, totals (via `.reduce()`), button `disabled` states (`= stack.length === 0`),
and the history list from scratch, so the screen can never drift out of
sync with the real state.

**History log ordering** — the one non-obvious line:
```js
const chronological = [
  ...undoStack.map(c => ({ label: c.label, done: true })),
  ...[...redoStack].reverse().map(c => ({ label: c.label, done: false })),
];
```
`undoStack` is already oldest→newest. `redoStack` ends up newest→oldest (the
most recently undone action is pushed last), so it's reversed before being
appended — giving one correctly ordered timeline with a ✓/↺ marker for
active vs. undone.

## Likely cross-questions

- Why clear `redoStack` on a new action? → old redo path is no longer valid once you've branched.
- Why does `cmdRemoveProduct` need both the index *and* a copy? → without the index, undo would always re-insert at the end, not the original position.
- Why rebuild the whole UI in `render()` instead of patching one element? → simplicity at this scale; a bigger app would diff instead.
- Complexity? → each op is O(1)–O(n) (n = cart size, from `.find`/`.findIndex`) — fine for a cart.

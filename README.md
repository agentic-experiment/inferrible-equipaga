# inferrible·equipaga

Infer your packing list from a few trip details. Pick your trip, get a bag.
One `index.html`, zero dependencies, zero build step.

Live on GitHub Pages: https://agentic-experiment.github.io/inferrible-equipaga/
Add `?selftest` to the URL to run the embedded self-check in the console.

## Step-by-step reasoning

### 1. What should this app be?
The name was the brief. *equipaga* ≈ Spanish *equipaje* (luggage), *inferrible*
≈ inferable. So: an app that **infers what goes in your bag** from a few inputs
— trip length, climate, transport, activities, international or not.

### 2. How does the "inference" work?
Not machine learning — a rule engine. Each rule is a pure function
`when(trip) -> boolean` plus a human `why(trip)` explanation. Filtering the
rule table over the trip state *is* the inference:

```js
function infer(trip){
  return RULES.filter(r => r.when(trip))
              .map(r => ({ item:r.item, why:r.why(trip) }));
}
```

Rules are data, not branching code: adding a heuristic is a one-line object,
never an `if/else` chain. Compound conditions (e.g. gloves for both cold and
snow) are just a boolean expression in `when`.

### 3. Why only one file?
The task allowed HTML + CSS + vanilla JS. A trip form, a rule table and a
checklist fit comfortably in a single file, so that's what it is. No modules,
no bundler, no framework — the browser is the runtime.

### 4. State management
Two kinds of state:

- **Trip state** — read directly from the form on every `input` event. The
  list re-renders live as you change the trip; no submit button needed.
- **Packed state** — which items you've checked off, persisted in
  `localStorage` under `inferrible.checked`. So the bag survives a page
  reload. Trip *inputs* are deliberately not persisted — restating your trip
  is the point of using it again.

### 5. Progress feedback
The checklist drives a progress bar + "N of M packed" counter. Cheap to
compute, and it gives the app a purpose beyond "generate a list": actually
packing to the bar being full.

### 6. Design decisions
- Every item shows *why* it was inferred — the reasoning is the feature. A
  list of 30 items without reasons is noise; with reasons it's a conversation.
- Items collapse into categories by trigger (always, duration, climate,
  activity, transport) in a fixed order so the list reads top-to-bottom like a
  real packing flow.
- Dark, system-font UI with no image assets and no CSS framework — the whole
  stylesheet is hand-written.
- Responsive: form goes single-column under 640px.

### 7. Verification
The inference logic is pure, so it gets one runnable check: `selftest()`
(also triggered by `?selftest` in the URL). It asserts that a sample trip
yields expected items, infers no irrelevant ones (no snow boots on a beach
trip), and that the baseline count holds. Tested by extracting the logic and
running it under Node; zero test frameworks.

### 8. What's deliberately not here
- Backend, accounts, saved trips — the app has no server reason to exist.
- Per-user sync, sharing, print/PDF export — useful someday, not for v1.
- A "database" of real destination weather — climate is a manual choice.
  Crude but honest: the user knows their trip better than a weather API guess.
- Anything that would need a second file or a dependency.

The entire app is ~400 lines of HTML/CSS/JS. That's the whole product.

# DSA Visual Learning App Design

## Purpose

This repository will contain a pure frontend application for learning data structures and algorithms through synchronized execution visualizations. The learner will see a JavaScript source line highlighted while the corresponding data structure changes on screen.

The product is a visualization-first learning tool rather than a conventional article-based knowledge base, online code editor, or general-purpose debugger.

## Audience and Language

The initial audience is the repository owner, learning DSA with JavaScript.

All interface labels, lesson explanations, validation messages, and error messages will be written in English. JavaScript source code, identifiers, and technical terms will also remain in English. The first version will not provide localization or language switching.

## Technology

The application will use:

- React
- Vite
- JavaScript, without TypeScript
- A fully client-side architecture, without a backend

The first version will not introduce a router or a global state-management library. React state and focused reusable hooks are sufficient for a single lesson and its playback state.

## First-Version Scope

The first version will implement one complete reference lesson:

```text
Algorithms
└── Sorting
    └── Bubble Sort
```

This lesson is intended to validate the course model, player, code highlighting, and data animation before more lessons are added.

The first version includes:

- A three-panel desktop learning interface
- Read-only JavaScript source code
- Synchronized code-line highlighting, English explanation, and array state
- Previous, next, play, pause, and reset controls
- A deterministic Bubble Sort execution timeline
- Responsive desktop, tablet, and mobile layouts
- Reduced-motion behavior
- Automated validation and tests for the lesson data and playback flow

The first version explicitly excludes:

- Editing or executing arbitrary JavaScript
- Changing the input array
- Playback-speed controls
- Saving learning progress
- Authentication, a backend, or a database
- Routing between lessons
- Lessons other than Bubble Sort

## User Interface

### Desktop

The desktop interface has three columns:

1. A lesson navigation panel showing the current location and lesson sections
2. A read-only JavaScript code panel showing the active source line
3. A visualization panel showing the current array state, explanation, and playback controls

The lesson sections are:

1. Core Concept
2. Execution
3. Complexity
4. Summary

Execution is the only interactive section in the first reference lesson. Core Concept, Complexity, and Summary provide concise static supporting content within the same page; they do not introduce separate routes or exercises.

### Tablet and Mobile

At tablet widths, the navigation and code areas remain above the visualization, which moves to a full-width row. At mobile widths, the lesson navigation is hidden because the first version contains only one lesson, and the code and visualization panels are stacked vertically.

Controls must remain keyboard accessible and visibly labeled. Motion must respect the user's `prefers-reduced-motion` setting; state changes remain understandable when movement is reduced or disabled.

## Architecture

The design uses a declarative execution timeline. A lesson module defines its source lines and a sequence of teaching steps. The application renders those steps; it does not introspect or debug executing JavaScript.

The high-level data flow is:

```text
Bubble Sort lesson definition
            │
            ▼
      playback controller
            │
      current step only
      ┌─────┼──────────┐
      ▼     ▼          ▼
 code panel  array visualizer  explanation panel
```

The current step has one owner. The code panel, visualizer, and explanation panel must not maintain independent playback positions.

### Main Modules

`AppShell` owns the application layout and composes the navigation and active lesson.

`LessonNavigation` displays the course hierarchy and active section. It does not own algorithm or playback state.

`AlgorithmLesson` connects a lesson definition to the playback controller and passes the current step to its child panels.

`usePlayback` owns the step index and exposes previous, next, play, pause, and reset operations. It understands timeline boundaries but has no Bubble Sort-specific behavior.

`CodePanel` renders source lines and highlights the line referenced by the current step.

`ArrayVisualizer` renders stable array items and animates comparison, swap, and sorted states. It understands generic array visualization actions, not Bubble Sort control flow.

`ExplanationPanel` renders the operation label and learner-facing description for the current step.

`PlayerControls` invokes playback operations and reflects whether playback is running or at a timeline boundary.

## Lesson Data Model

A lesson definition contains metadata, read-only code lines, and ordered execution steps. The conceptual shape is:

```js
{
  id: "bubble-sort",
  title: "Bubble Sort",
  category: "Sorting",
  codeLines: [
    "function bubbleSort(arr) {",
    "  // ...",
    "}"
  ],
  steps: [
    {
      line: 2,
      action: "compare",
      items: [
        { id: "item-7", value: 7 },
        { id: "item-3", value: 3 }
      ],
      activeIds: ["item-7", "item-3"],
      sortedIds: [],
      description: "Compare 7 and 3."
    }
  ]
}
```

Array items have stable IDs independent of their positions. React can therefore preserve item identity across swaps, allowing an item to visibly move instead of being replaced by an unrelated DOM element.

Each step contains the complete visual state needed to render it. This makes backward navigation deterministic and avoids reconstructing earlier states by reversing mutations.

The supported first-version actions are:

- `initialize`
- `compare`
- `swap`
- `mark-sorted`
- `complete`

## Playback Behavior

The lesson opens on a non-playing preparation step.

- Previous moves back one step and pauses playback.
- Next moves forward one step and pauses playback.
- Play advances at a fixed interval.
- Pause keeps the current step visible.
- Reset pauses and returns to the preparation step.
- Reaching the final step automatically stops playback.
- Starting playback from the final step resets to the beginning before advancing.
- Controls that cannot act at a boundary are disabled.

All panels update from the same current step. A transition is considered complete only when the code highlight, operation label, explanation, item order, active items, and sorted items represent that step.

## Validation and Error Handling

Lesson definitions will be validated during development. Validation must reject at least:

- Duplicate lesson IDs
- An empty `codeLines` or `steps` collection
- A step that references a nonexistent code line
- Duplicate item IDs within a step
- `activeIds` or `sortedIds` that reference missing items
- An unsupported action

Invalid lesson data must produce a clear development error identifying the lesson and step. At runtime, the lesson area must show an English failure message instead of rendering inconsistent or partially synchronized panels.

The playback controller clamps navigation to valid timeline boundaries. Repeated play requests must not create multiple timers, and timers must be cleaned up when playback pauses or the lesson unmounts.

## Testing Strategy

Automated verification will cover three layers:

1. Lesson validation tests confirm that valid Bubble Sort data passes and representative malformed definitions fail with useful messages.
2. Playback-controller tests cover initial state, previous, next, play, pause, reset, final-step behavior, repeated play requests, and timer cleanup.
3. Component and browser-level tests confirm that one current step synchronizes the highlighted code line, explanation, and array state, and that a complete Bubble Sort playback reaches the sorted result.

Manual visual checks will cover desktop, tablet, and mobile layouts as well as reduced-motion behavior. Keyboard navigation and visible focus states will also be checked.

## Planned Repository Shape

The implementation plan may refine filenames, but the intended boundaries are:

```text
src/
├── app/
│   └── App.jsx
├── components/
│   ├── lesson/
│   ├── player/
│   └── visualizers/
├── content/
│   ├── data-structures/
│   └── algorithms/
│       └── sorting/
│           └── bubble-sort.js
├── hooks/
│   └── usePlayback.js
├── validation/
│   └── validateLesson.js
└── styles/
```

The `data-structures` and `algorithms` directories remain separate at the content level, preserving the original desired knowledge organization as the course expands.

## Expansion Path

After the Bubble Sort reference lesson proves the architecture, additional algorithms can supply new lesson definitions while reusing the playback controller and common panels. Array-based algorithms should reuse `ArrayVisualizer`; future linked-list, tree, and graph lessons can introduce visualization components for their own data shapes without changing the generic playback contract.

Likely subsequent lessons are Linear Search, Binary Search, Stack, Queue, and Linked List. Their inclusion is a later design and implementation decision, not part of this first version.

## Alternatives Considered

### Generator-Driven Execution

A real Bubble Sort implementation could yield compare and swap events while it runs. This would more closely resemble algorithm execution, but exact code-line mapping, backward navigation, and curated explanations would become more complex. It is unnecessary for a read-only teaching experience.

### Bespoke Component Per Algorithm

Each lesson could own its player and animation logic. This would make the first demo quick to build but would duplicate playback behavior and make future lessons inconsistent.

### Editable Code Runner

Running arbitrary learner code and deriving animations would require execution isolation, instrumentation, error handling, and a mapping between arbitrary code and visual semantics. That is a different product and is explicitly outside this design.

## Success Criteria

The first version succeeds when a learner can open the Bubble Sort lesson, manually step or automatically play through it, and understand how each highlighted JavaScript line changes the array. The final state must be correctly sorted, all three panels must remain synchronized, playback must be reversible through previous steps, and the experience must remain usable across the defined responsive layouts.

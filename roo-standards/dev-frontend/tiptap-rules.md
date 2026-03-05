# Tiptap React / TypeScript Rules

## Core Principles

- Use Functional Components and Hooks (useEditor).
- Maintain strict TypeScript types for all custom extensions.
- Prefer 'headless' logic; keep UI/CSS separate from extension logic.

## Editor Setup

- Always set `immediatelyRender: false` in `useEditor` to support SSR.
- Extensions should be memoized or defined outside the component to prevent unnecessary re-initializations.
- Use `EditorContent` for the main view and `useCurrentEditor` for sub-components (toolbars).

## Performance & State

- Do NOT sync editor state to a parent React state on every 'onUpdate' unless strictly necessary (e.g., character counts). Use debouncing for saves.
- Use `editor.commands` for programmatic changes; do not manipulate the DOM directly.

## Custom Extensions (Nodes/Marks)

- Use `Node.create` or `Mark.create` from `@tiptap/core`.
- Always implement `parseHTML` and `renderHTML` to ensure copy-paste compatibility.
- For interactive nodes, use `ReactNodeViewRenderer` to build the UI with React components.

## Media Handling

- Intercept image drops/pastes.
- Rule: No Base64 strings. Convert to URLs via an upload service before calling `setImage`.

## Transactions

- Use `editor.chain().focus()....run()` for combined actions.
- Access underlying ProseMirror state via `editor.state` and transactions via `editor.view.dispatch`.

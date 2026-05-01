---
name: improve-react-code
description: Find composition and state-lifting refactor opportunities in a React codebase. Audits for boolean-prop bloat, monolith components with variant explosion, sibling clusters that should share lego-brick subcomponents, render-prop slots that can't reach outside the box, and state synced between siblings via useEffect. Use when the user wants to improve react code, audit react components, find react refactor opportunities, run a react composition review, or asks "improve react", "audit react", "react architecture review".
---

# Improve React Code

Find refactor opportunities in the React side of a codebase, focused on composition and lifted state.

React codebases drift toward monolithic components: every new variant adds another boolean prop, another render-prop slot, another optional field in a config array. Eventually the file is hard to peek inside — for humans and AI alike. The way out is composition — split variants into distinct components built from a small set of named subcomponents and a lifted state provider.

A 15-boolean monolith is worth fixing on its own, but the biggest wins come from clusters of similar siblings (e.g. `CreateUserForm` and `UpdateUserForm`) that could share the same extracted pieces. Lifting state and replacing prop bloat with JSX is where most friction relaxes.

## Glossary

- **Brick** — a small, named subcomponent reused across variants (`UserFormHeader`, `ModalFooter`, `Wizard.Step`).
- **Lifted provider** — the context `Provider` extracted into its own component that takes `children`, so siblings outside the visual box can read its state and actions.
- **Sibling cluster** — 2+ components in the same feature that share most of their structure but differ in variant logic (e.g. `CreateUserForm` and `UpdateUserForm`, or `ChannelDialog` and `ThreadDialog`).
- **State slot** — `state` / `actions` / `meta` exposed via context. Children read from it; the implementation (useState, store, custom hook) lives in the component that renders the provider.
- **Meta** — the slot for non-state values (e.g. an `inputRef`) that several siblings need access to. Avoids `useImperativeHandle` and ref drilling.

## Core principles

Audit findings should be framed in terms of these principles. Reference them by short name in the **Problem** field of each candidate.

1. **Boolean props that gate which subtree renders → split into distinct components.** `<UserForm isUpdate>` becomes `<CreateUserForm>` and `<UpdateUserForm>`, sharing a `UserFormProvider`, `UserFormFields`, `UserFormSubmit`.
2. **Compound components.** Build features as a Provider + named subcomponents (Header, Body, Footer, Submit, etc.) plus distinct action subcomponents.
3. **JSX over arrays-of-config for variant UI.** Replace `[{label, isSortable, divider, isMenu, items}]` with `<Table.Column>`, `<Table.Divider>`, `<Table.MenuColumn>`.
4. **"Common*" wrappers are JSX inside JSX**; they never take their own boolean variants.
5. **Don't render absent UI by toggling a boolean — just don't render it.** A dialog that has no dropzone simply omits `<Dialog.Dropzone>`. No `disableDropzone` prop.
6. **Avoid `renderX` slot props as the primary surface.** They don't survive when the slot needs to render outside the visual box (e.g. a Modal whose Forward button lives below the modal). Children-as-function — for sharing internal state with the caller — is a different, valid pattern.
7. **Provider exposes `{ state, actions, meta }`.** Implementation lives in the component that renders the provider. Children stay agnostic — they work whether state is ephemeral, persisted, or synced across devices. The `actions` and `meta` slots are open-ended bags — start small (`actions={{ update: setState }}`) and grow them as new operations show up (`{ update, submit }`). When state comes from a custom hook (e.g. `useSyncedDraft`, `useFormStore`), the hook's return shape must conform to the provider's interface.
8. **Lift state by extracting the provider.** `<WizardProvider>{children}</WizardProvider>` lets Next/Back buttons rendered as siblings of the wizard read context directly — no `onStepChange` callback or shared ref. In practice, this usually means moving the provider into its own file (`WizardProvider.tsx`) so it can wrap the visual `Wizard.tsx` *and* whatever sits next to it.
9. **Use `meta` for refs and non-state values** instead of `useImperativeHandle` or ref drilling.
10. **Don't sync sibling state via `useEffect`.** A `useEffect(() => onChange(state), [state])` whose only job is lifting state up is the `onFormStateDidChange` smell. Lift the provider instead.
11. **Don't fear context re-renders.** React Compiler memoizes consumers by the slice they actually read.

Plus, prop-API hygiene:

- **Don't use `React.FC`.** Type props with an interface on the function parameter.
- **Prop API decision flow:** lists → `items` + `renderItem`; variants without internal data → `children` (compound); variants needing internal data → children-as-function; simple ReactNode swap → slot prop; 3+ props describing the same element → single slot prop; simple variants → value prop (`variant="ghost"`).
- **File placement:** feature folder first; shared `components/` only when 2+ features use it.
- **Composition over rigid props.** Prefer `<Button>{icon}{label}</Button>` over `<Button icon label />`.

## Threshold — when NOT to flag

Refuse to flag a component for compound-component refactoring unless **at least one** is true:

- (a) ≥2 distinct callers shape it differently.
- (b) The same condition is checked in ≥2 JSX branches inside one component.
- (c) UI rendered outside the visual box needs the component's state/actions.
- (d) Variants need different state-management implementations (ephemeral vs synced, etc.).
- (e) **Strongest signal** — 2+ structurally similar siblings exist that would share extracted bricks.

Below that threshold, single booleans, single render-prop slots, and children-as-function are fine. Don't refactor for the sake of it.

## Process

### 1. Explore

Use the Agent tool with `subagent_type=Explore` to walk the React side of the codebase. Don't follow rigid heuristics — explore organically, but also:

- Scan for **name-stem clusters** (`*Form`, `*Modal`, `*Dialog`, `*Card`, `*List`, `*Wizard`, `*Sheet`, `*Drawer`, `*Panel`, `*Composer`).
- For each cluster, read the candidate components and judge structural similarity by eye. Two components share a "substructure" when they render a recognizable named region (header, body, footer, action row, input area, etc.) the same way. A cluster qualifies when ≥2 components share ≥3 substructures.
- Note components with high prop count (many booleans), `useEffect` whose only job is calling a parent setter, `useImperativeHandle` outside design-system primitives, and array-of-config UI with heterogeneous item shapes.

Apply the **threshold** before promoting any finding into the report.

### 2. Present numbered candidates

Return a numbered list. Each candidate:

- **Files** — paths involved.
- **Problem** — which principle is being violated, in plain English. Reference the principle by short name.
- **Solution** — sketch the proposed compound API in flat-export style (`<UserFormProvider>`, `<UserFormFields>`, `<UserFormSubmit>`) — unless the codebase already uses dot-namespaced compound, in which case match that.
- **Reuse map** — which other components would consume the extracted bricks (this is what makes a finding *Critical*).
- **Severity** — Critical / Medium / Bikeshedding (see below).
- **Benefits** — testability, fewer call-site shapes, AI navigability.

Do NOT propose detailed interfaces yet. Ask: *"Which of these would you like to explore?"*

### 3. Grilling loop

Once the user picks a candidate, walk the design tree: what becomes a brick, what stays a one-off, where the provider sits, who lifts state, what `meta` carries, which siblings need access. Output a recommended interface (set of compound parts + provider shape) that the user can hand to Claude to implement.

## Detection signals, tiered

**Critical** — lead the report:

- Sibling clusters with shared bricks (threshold clause e).
- ≥3 boolean props gating JSX subtrees, used in ≥2 distinct shapes.
- State sync between siblings via `useEffect` (`onFormStateDidChange` smell).
- UI outside the visual box reaching back in via callbacks or shared refs.

**Medium**:

- Same condition checked in ≥2 JSX branches inside one component.
- `renderX` slot prop where the slot needs internal state and could live in context.
- Array-of-config UI with heterogeneous item shapes (`isMenu`, `divider`, `items`).
- `useImperativeHandle` for things that could live in context `meta`.
- File placement (feature-folder vs shared `components/`).

**Bikeshedding** — collapsed at the bottom of the report:

- `React.FC` usage.
- Lone components with 1–2 boolean props and a single call site.
- Render-prop slots that work fine and don't escape the box.
- 3+ props describing the same element with only 1 caller.

## Agnostic examples

- **UserForm with `isCreate` / `isUpdate` booleans** → split into `CreateUserForm` and `UpdateUserForm`, sharing `UserFormProvider`, `UserFormFields`, `UserFormSubmit`.
- **Modal with `renderFooter` whose footer needs modal state** → refactor to `<ModalProvider>` + `<Modal.Footer>` siblings reading shared context.
- **DataTable taking `[{label, isSortable, divider, isMenu, items}]`** → flat JSX subcomponents `<Table.Column>`, `<Table.Divider>`, `<Table.MenuColumn>`.
- **Wizard syncing state up via `useEffect(() => onStepChange(currentStep), [currentStep])`** → lift `<WizardProvider>` so the parent's Next/Back buttons read context directly.
- **Dialog with `disableDropzone`, `hideHeader`, `hideTitle` flags** → distinct dialog components that simply omit the parts they don't need.

## Style for proposed solutions

Default to **flat separate exports**:

```tsx
<UserFormProvider>
  <UserFormFields />
  <UserFormSubmit />
</UserFormProvider>
```

If the codebase already uses dot-namespaced compound (`<UserForm.Fields>`), match that — respect existing convention. The underlying composition principles are identical either way.

## Skips (default)

- `*.test.tsx`, `*.spec.tsx`, `*.stories.tsx`.
- Generated/codegen output (`*.gen.*`, GraphQL types).
- `node_modules`, `dist`, `build`, `.next`, `out`.
- Design-system primitives by location (`components/ui/`, `design-system/`, `primitives/`) and tiny single-export leaf files.
- Files under 50 lines.

User-targeted paths (e.g. `improve-react-code path/to/file.tsx`) bypass every skip — explicit beats implicit.

## Notes

- **Agnostic to state-management library.** The provider's interface is `{ state, actions, meta }`; the implementation can be `useState`, Zustand, Redux, a sync hook, or anything else.
- **React Compiler** removes most context-perf objections — a consumer is memoized by the slice it reads. If the project doesn't use the compiler yet, suggest it as a follow-up.
- **Respect existing codebase conventions** — naming style, file structure, dot-namespacing — even when proposing refactors.
- **Does not read CONTEXT.md or ADRs.** This skill audits code patterns, not domain modeling.

---
name: technical-implementation-plan
description: Create implementation-ready technical plans for product features and substantial code changes. Use when a plan must connect product requirements to end-to-end system flow, code-level interfaces, tests, and independently verifiable vertical-slice deliverables; do not use for task publishing or implementation itself.
---

# Technical Implementation Plan

Produce a plan that is concrete enough for an engineer or coding agent to implement without rediscovering the design. Planning is read-only unless the user separately asks to save or publish the plan. Do not implement the feature or create tasks merely because this skill is active.

## Ground the plan

Use the conversation, referenced requirements, and repository evidence. Inspect the relevant code paths, tests, domain documentation, and architecture decisions before naming files or interfaces. Reuse the project's terminology and established patterns.

Distinguish confirmed requirements from assumptions. Ask only about an unknown that would materially change the product behavior or architecture; otherwise make the smallest reasonable assumption and record it. Do not invent file paths, functions, dependencies, or existing behavior.

Look for prerequisite prefactoring that makes the change easy. Include it only when there is a concrete need, and place it before the feature slices. Keep it behavior-preserving and independently verifiable.

## Design deliverables as tracer bullets

Every numbered deliverable is a thin but complete vertical slice, not a horizontal layer of work.

<vertical-slice-rules>

- Each slice delivers a narrow, complete path through every relevant integration layer, such as schema, domain logic, API, UI, observability, and tests.
- A completed slice is demoable or verifiable on its own and has a clear externally observable outcome.
- Include only layers relevant to that slice; state why an apparently relevant layer is not needed.
- Split slices when they contain multiple independently valuable behaviors. Merge slices that cannot be verified independently.
- Order slices by dependency and identify blockers explicitly.
- Put any necessary prefactoring first.

</vertical-slice-rules>

Do not organize deliverables as “database,” “backend,” “frontend,” and “tests.” A slice may touch all of them.

## Harness-aware plan output

Keep the plan content portable Markdown, then apply only the rendering convention explicitly supported by the active harness. Determine the harness from system-provided environment, mode, or tool context; never infer Codex merely because the user typed `/plan`.

Use this precedence:

1. Follow explicit plan-output instructions supplied by the active harness.
2. In positively identified Codex Plan Mode, place the entire completed plan inside exactly one `<proposed_plan>` block. Put the opening and closing tags on their own lines and emit no second plan block.
3. In Claude Code Plan Mode, return the plan as normal Markdown and let Claude Code's native approval interface handle it. Do not emit Codex tags.
4. In Cursor Plan, Ask, or planning-oriented Custom Mode, return normal Markdown for Cursor's native interface. Do not emit Codex tags.
5. In an unknown or unidentified harness, return portable Markdown without proprietary tags or control syntax.

In every harness, finish repository exploration and resolve all material product and technical decisions before presenting the final plan. Ask for clarification when a material decision cannot be established from requirements or repository evidence. Keep every required section, signature, test, deferred decision, and traceability statement inside the native plan artifact when the harness provides one.

Do not create or update a repository plan file unless the user explicitly requests a persisted file and the active mode permits file mutation. Never emit another harness's proprietary control syntax.

## Required output

Use the following section order and headings.

### 1. Summary

State the user-visible outcome, the proposed technical approach, the main systems affected, and the important boundaries or non-goals. Keep this brief.

### 2. Product Requirements

List the product behavior the implementation must satisfy. Give each requirement a stable identifier such as `PR-1` so deliverables and tests can trace back to it. Include acceptance conditions, important error or edge behavior, and explicit non-goals. Label assumptions as assumptions rather than requirements.

### 3. System Flow of Complete Feature End to End

Describe the full runtime path from the initiating actor or event to the final observable result. Include relevant validation, state transitions, persistence, external calls, async boundaries, retries or failure handling, and user-visible feedback. Use a numbered sequence; add a compact diagram only when it materially clarifies branching or component relationships.

### 4. Deliverable 1: `<deliverable name>`

Continue with `Deliverable 2`, `Deliverable 3`, and so on. Order deliverables by dependency. For each deliverable, use this internal structure:

#### 4.1 Deliverable Overview

Explain the narrow end-to-end behavior delivered, why it is independently valuable, requirements covered, relevant layers crossed, dependencies or blockers, and how completion can be demonstrated or verified. State the observable acceptance criteria.

#### 4.2 Technical Implementation Structure

Describe the implementation file by file. Files may be grouped under subsystem headings, but every added, modified, or deleted file must have its own file-path subheading. Do not present a combined file inventory followed by detached types, functions, or implementation notes.

Use the applicable form of this template for every file:

##### `<exact/path/to/file.ext>`

**Change:** Add | Modify | Delete

**Purpose:** Explain this file's responsibility in the deliverable and why it belongs in this domain or package.

For a new code file, continue with:

**Implementation:**

**Types:**

```text
type Example struct {
    ID string
}
```

- `Example` — Explain what the type represents, its invariants, and which boundaries consume it.

**Functions:**

```text
func NewExample(dependency Dependency) *Example

func (example *Example) Execute(
    ctx context.Context,
    input Input,
) (Output, error)
```

- `NewExample` — Describe construction, validation, and retained dependencies.
- `Execute` — Describe control flow, state changes, collaborators, returned values, and error behavior.

Use the project's language in signature fences instead of `text`. Include argument names and types, return types, visibility, and meaningful error/result shapes. Mark names as proposed when they do not already exist.

For an existing code file, continue with:

**Existing behavior:** Summarize what the file currently owns and the behavior that remains unchanged.

**Changes:** Describe how the implementation changes or extends that behavior, including control flow, state changes, data mapping, integration boundaries, and error handling.

**Declaration changes:** When an existing function signature, method signature, interface, struct, enum, or other public type changes, show a minimal declaration-only diff based on the inspected code:

```diff
 type Dependencies struct {
     Authentication http.Handler
+    Reviews        http.Handler
 }

-func NewWorker(store Store) *Worker
+func NewWorker(store Store, synchronizer Synchronizer, config WorkerConfig) *Worker
```

Include enough unchanged context to identify the declaration. Do not show speculative implementation-body diffs. If behavior changes without a signature or type change, write `**Interface changes:** None. Existing signatures remain unchanged.` instead of manufacturing a diff.

**Integration and compatibility:** Identify callers that must change, interfaces implemented or consumed, schema/API/configuration/UI/observability contracts affected, reused code, compatibility constraints, and product requirements covered.

For a unit or component test file, use:

##### `<exact/path/to/file_test.ext>`

**Change:** Add | Modify

**Purpose:** Identify the production file or public behavior this test file verifies.

**Tests:**

###### `<TestName>`

```text
func TestName(t *testing.T)
```

- **Tests:** Summarize the production behavior under test.
- **Setup/Input:** List meaningful fixtures, dependencies, and inputs.
- **Execution:** Name the public function, method, handler, component, or workflow exercised.
- **Assertions:** State the observable results, persisted state, emitted requests, or errors.
- **Requirements:** List the covered product requirement identifiers.
- **Regression protected:** Name the correctness, security, or reliability risk, or write `Not a regression test`.

Use the test framework's actual declaration syntax. Repeat the named test subsection for each meaningful test or table-driven test group. Prefer tests through public interfaces and observable outcomes. Combine closely related scenarios when that keeps behavior clear, and add edge coverage only for meaningful risks.

For a non-code file, use:

##### `<exact/path/to/non-code-file>`

**Change:** Add | Modify | Delete

**Purpose:** Explain the role of the migration, query file, configuration, contract, fixture, or generated artifact.

**Changes:** Describe the exact schema, query, configuration, contract, fixture, or generation changes. Include constraints, defaults, indexes, rollback behavior, consumers, and product requirements when applicable.

**Functions and types:** Not applicable.

Give every generated file its own subheading, identify the source schema, query, or generator that produces it, and do not reproduce generated contents.

Enforce these rules across the whole deliverable:

- Mention each file only once and consolidate all of its work beneath its subheading.
- Place every proposed or changed type and function under the file that owns it.
- For modified files, distinguish preserved behavior from new behavior.
- Inspect the existing declaration before showing a diff; never fabricate the old side.
- Use a declaration diff only when a signature or type declaration changes.
- Give test files the same per-file treatment as production files.
- Do not use a detached `Add:`, `Modify:`, or generic file list as a substitute for the per-file breakdown.

#### 4.3 E2E Tests

Describe any end-to-end or integration tests needed for the complete slice. For each, give its name, file path, scenario, setup, execution path, and observable assertions. If no E2E test is warranted, write `Not required` and explain which lower-level or existing coverage verifies the slice.

#### 4.4 Deferred Decisions

List deliberately postponed choices, why they can wait, the trigger for revisiting them, and what future work they could affect. Write `None` when there are no deferred decisions. Do not hide unresolved implementation blockers here; surface those before presenting the plan as ready.

For later deliverables, use the deliverable section number in the internal headings, for example `5.1` through `5.4` for Deliverable 2.

## Quality check

Before returning the plan, verify that:

- every product requirement maps to at least one deliverable and a verification method;
- every deliverable is a complete, independently verifiable vertical slice;
- signatures, test names, and file paths are supported by repository evidence or clearly labeled as proposed;
- dependencies and prefactoring appear before the work they unblock;
- the end-to-end flow agrees with the deliverable sequence;
- deferred decisions are genuinely deferrable and no critical design choice is silently unresolved.

End with a compact requirement-to-deliverable traceability list when the mapping is not already obvious.

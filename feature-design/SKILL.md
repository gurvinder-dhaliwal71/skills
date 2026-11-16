---
name: feature-design
description: Technical design for a feature, written before any build: flows, architecture with call-stack diff trees, interfaces, schema, risks and a stacked PR plan. Use when asked to design a feature or write a technical design before building it.
---

# Feature Design

Write one design document that an owner can approve before any code exists. The design is the only file this skill writes. Source, tests, git history, PRs and other docs stay as they are.

Write plain, short sentences, one idea each (ASD-STE100 style). Use the same word for the same thing throughout. Prefer a table or a code block to a paragraph.

## 1. Read the request and the rules

- Read the ticket or request in full, with its comments and any parent ticket.
- Read the repository's rules: `AGENTS.md`, `CLAUDE.md`, contributing guides, and the architecture doc they point to.
- Fetch the default branch (`git fetch origin`) and note its SHA. This is the **base** for every call stack. Note open PRs that touch the same code.

Done when you can state the problem in two sentences, you know the repository's rules on ownership, tests and PR size, and you hold the base SHA.

## 2. Trace the real code

- Find every entry point the feature touches: routes, pages, webhooks, jobs, CLI commands.
- Follow each one down to the functions that will change. Read each function at the base SHA (`git show <base>:<path>`).
- Read the current schema and migrations for every table the feature reads or writes.
- Mark anything you cannot follow to source as `(unverified)`. A gap is better than a guessed caller.

Done when every function, table and route the design will name has been read at the base, or is marked `(unverified)`.

## 3. Identify the flows

A **flow** is one user journey: a person or a system starts it, and it ends in an outcome they see. Name each flow by what the person does ("Request deletion", "Open a disabled workspace").

- List one flow per journey, including the failure and race journeys that change behaviour.
- Pick a small cast of example people and objects. Reuse them in the flows and in the tests.

Done when every behaviour change from step 2 belongs to exactly one flow.

## 4. Fill the template

Copy [`DESIGN_TEMPLATE.md`](DESIGN_TEMPLATE.md) and fill every section in order. Each section has a one-line instruction; replace it with content. Keep the section numbers and headings. A section with nothing to say gets "No change." or "None."

For each flow, draw its call stack as described in **Call stack trees** below.

Done when every section is filled, every flow has its four architecture parts (High-Level, Call Stack, Interface Design, Data Schema), and every Open Point has a recommended default.

## 5. Write the design

- Write to the repository's design folder. Use the path the owner gives. Else, when `docs/design/` exists, use `docs/design/<ticket>-<slug>.md`. Else ask.
- Leave the file uncommitted unless the owner asks.

Done when the file exists at the agreed path and no other file changed.

## 6. Hand off for approval

Reply with the path, a three-line summary, and the Open Points. Stop there. The build starts only after the owner approves the design.

## Call stack trees

Each flow's Call Stack part is one diff tree. The format follows the "Draw the diff trees" step of [`../callstack-review/SKILL.md`](../callstack-review/SKILL.md). The essentials, so this works without that file:

- **Base** is the default branch (for example `origin/main`). **Head** is the planned change.
- Shorten paths with the aliases in `.callstack/PATH_ALIAS.md` when it exists. Above each block, print one legend line with the aliases it uses: ``Paths: `api/` = `apps/api/src/`, `core/` = `packages/core/src/` ``. Print no legend when no path uses an alias.
- Write a bold title, one or two sentences on what changes in the flow, then one fenced `diff` block.
- Each line is `<marker><connectors><function>()  <path>  <one-line explanation>`. The marker is `+` (added), `-` (removed) or a space (context), followed by one more space.
- Connectors are `├─`, `└─` and `│`, nested by call depth. Align paths and explanations into columns.
- A changed function is a `-` line with its base form, then a `+` line with its planned form. A planned new function is a `+` line with its planned path.
- Trace every context and `-` line from real source at the base. Mark what you cannot follow `(unverified)`.
- Show the changed lines, their ancestors up to the entry point, and one context line above and below. Collapse unrelated siblings.

````markdown
**Checkout**

Orders to unsupported regions are rejected before payment.

```diff
  POST /orders                  routes/orders.ts       Place an order from the cart
  └─ placeOrder()               orders/place.ts        Validate, price and charge the cart
+    ├─ checkShippingRegion()   shipping/regions.ts    Reject addresses outside supported regions
     └─ chargePayment()         payments/charge.ts     Charge the customer's card
```
````

These trees go into the design only.

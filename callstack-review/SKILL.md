---
name: callstack-review
description: Show the call stacks a pull request changes as diff-coloured trees, traced from source. Pass a PR number (or none for the current branch's PR); add --pr to write the result into the PR description.
disable-model-invocation: true
---

# Call Stack Review

Show a reviewer which call stacks a pull request changes, one diff tree per flow, with every line traced from source. Requires `gh` authenticated against the repository's GitHub remote.

The skill writes only two things: `.callstack/PATH_ALIAS.md` when it is missing, left uncommitted, and the PR description in `--pr` mode. Source, git history, the working tree's other files and the current branch stay as they are.

## 1. Read the PR

- Resolve the PR: the number given, or `gh pr view --json number` for the current branch.
- Read `gh pr view <n> --json number,title,body,baseRefName,baseRefOid,headRefOid` and `gh pr diff <n>`.
- From the remote that points at the PR's base repository, fetch the base branch and the PR head: `git fetch <remote> <baseRefName> pull/<n>/head`. This works for PRs from forks.
- The **base** is `git merge-base <baseRefOid> <headRefOid>`; the **head** is `<headRefOid>`. Read files with `git show <sha>:<path>`.

Done when you have the changed source files (tests and docs set aside), both SHAs, and a first guess at which runtime flows the changes sit on.

## 2. Load path aliases

Aliases shorten long, shared path prefixes in the trees. They live in the repository's `.callstack/PATH_ALIAS.md`.

- Read the file from the head, else the base, else the working tree.
- When it exists nowhere, create it in the working tree. List `git ls-files` at the head and add an alias for each **package root**: the folder a reader names the code by (`core`, `billing`, `parser`), extended through the wrapper folders under it that every file shares (`src/`, `lib/`, `internal/`, `src/main/java/com/acme/billing/`). Name the alias after the package (`packages/core/src/` → `core/`, `pkg/core/` → `core/`). Skip a root whose full path is already short. When two packages share a name, add the parent folder (`apps-api/`, `packages-api/`). Every alias and path ends in `/`. A repo with no package roots worth aliasing gets the file with an empty table.
- Write the file from [`PATH_ALIAS_TEMPLATE.md`](PATH_ALIAS_TEMPLATE.md): keep the heading, sentence and header rows verbatim and replace the placeholder row with one row per alias, both values in backticks. Keep rows that share a parent folder together (every `apps/…` row, then every `packages/…` row), and pad every cell so the column pipes line up. An empty table keeps the header and separator rows only.
- Leave the file uncommitted and unstaged.

Done when you hold the alias table, possibly empty, and know whether you created the file.

## 3. Trace each flow

A **flow** is one entry point a user or system triggers (an HTTP route, a webhook, a CLI command, a background job, a scheduled task) followed down to the changed functions. For each changed function, walk callers up to the flow's entry point and callees down one level where the change alters them.

- Read every displayed symbol at both SHAs. A function's old form comes from the base, its new form from the head.
- Distinguish nested calls from sequential siblings, and follow injected functions to the wiring that supplies them.
- Mark anything you cannot follow to source as `(unverified)`. A gap is better than a guessed caller.

Done when every runtime behaviour change in the diff sits in exactly one flow (plumbing such as row mapping may fold into its caller), and every line you will show has been read at the SHA it claims.

## 4. Draw the diff trees

Above the first flow, print one legend line with the aliases this review uses, in table order:

```markdown
Paths: `api/` = `apps/api/src/`, `core/` = `packages/core/src/`
```

Print no legend when no path uses an alias.

For each flow, write a bold title, one or two sentences on what the PR changes in it, then one fenced `diff` block. Each line is:

```text
<marker><tree connectors><function>()  <path>  <one-line explanation>
```

- **Marker** is the first character: `+` added, `-` removed, a space for unchanged context. Follow it with one more space so the tree aligns (`+ `, `- `, two spaces for context).
- **Connectors** are `├─`, `└─` and `│`, nested by call depth.
- **Path** is the file's path from the repository root, with the longest matching alias prefix replaced by its alias. Align paths and explanations into columns.
- **Explanation** says what the function does in this flow, in a few words.
- A changed function is a `-` line with its old form, path and explanation, then a `+` line with the new.
- Show only the changed lines, their ancestors up to the entry point, and one unchanged context line above and below. Collapse unrelated siblings.

Separate flows get separate blocks. Example:

````markdown
**Checkout**

Shipping is now quoted by the carrier instead of a flat-rate table, and orders to unsupported regions are rejected before payment.

```diff
  POST /orders                  routes/orders.ts       Place an order from the cart
  └─ placeOrder()               orders/place.ts        Validate, price and charge the cart
     ├─ reserveStock()          inventory/stock.ts     Hold the items for 15 minutes
+    ├─ checkShippingRegion()   shipping/regions.ts    Reject addresses outside supported regions
-    ├─ shippingCost()          shipping/flat-rate.ts  Look up cost in the flat-rate table
+    ├─ shippingCost()          shipping/rates.ts      Quote cost from the carrier
+    │  └─ carrier.quote()      carriers/client.ts     Call the carrier's rate API
     └─ chargePayment()         payments/charge.ts     Charge the customer's card
```
````

Colouring depends on the renderer: a terminal and GitHub show `+` lines green and `-` lines red; some apps show plain `+`/`-`.

Done when each flow from step 3 has its block, every `+`/`-` line maps to a hunk in `gh pr diff`, and every aliased path appears in the legend.

## 5. Deliver

By default, reply in the chat with the legend and flow blocks.

Only when the owner asks explicitly (for example `/callstack-review 140 --pr`), write the same content into the PR description under a `## Call stack` heading:

- Replace an existing `## Call stack` section (up to the next `## ` heading or the end) or, if none exists, append it.
- Keep every other byte of the body identical. Give this run its own body files, so parallel runs never share a path: `orig=$(mktemp "${TMPDIR:-/tmp}/pr<n>-orig.XXXXXX")` holds the body as read, `new=$(mktemp "${TMPDIR:-/tmp}/pr<n>-new.XXXXXX")` holds the edited body.
- Before editing, confirm `$new` still contains the first non-blank, non-heading line of `$orig` (this PR's own summary); on a mismatch, rebuild `$new` from a fresh read. Then run `gh pr edit <n> --body-file "$new"`, re-read the body, and diff it against `$orig` to confirm only that section changed.

If step 2 created `.callstack/PATH_ALIAS.md`, end the reply with: "Created `.callstack/PATH_ALIAS.md`. Review the aliases and commit it."

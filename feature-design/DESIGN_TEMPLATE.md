# <Ticket> · <Feature name>

Technical design, <date>. For review. No code changes yet.

Ticket: <link>. Base: `<default branch>` (`<sha>`)<, plus open PRs this builds on>.

## 1. Problem Statement

<What is wrong or missing today, and who it affects. Two to four sentences.>

## 2. Solution

<The outcome: what is true after this ships, as short bullets grouped by who sees it (users, other actors, the system). State each invariant and the module that owns it.>

## 3. Flows

<Name the example people and objects used in the flows and the tests.>

### 3.1 <Flow name>

<One flow per user journey: numbered steps of what the person clicks and what they see. Quote screens, copy and error messages. Give results per response as a table.>

## 4. Architecture

<One line: the path alias legend, Base = `<default branch>`, Head = this design.>

### 4.1 <Flow name> Architecture

#### 4.1a High-Level

<Name the actors in one line (for example Web = website, PG = PostgreSQL). Then a step table. Change is NEW, CHANGED or blank. Write a branch as sub-steps (3a, 3b) that start with the condition. No Mermaid and no ASCII sequence diagrams.>

| # | From → To | What happens | Change |
|---|-----------|--------------|--------|
| 1 | <Actor> → <Actor> | <Action> | |
| 1a | <Actor> → <Actor> | If <condition> → <result> | NEW |

<Then short bullets: why this shape, races as a table, and which module owns each rule.>

#### 4.1b Call Stack

<A call-stack diff tree for this flow, base = default branch, head = the planned change, in the format of the skill's "Call stack trees" section: legend, bold title, one or two sentences, one `diff` block.>

#### 4.1c Interface Design

<Code only, no prose: signatures, types, result unions, doc comments, and routes with status codes as code comments.>

```typescript
```

#### 4.1d Data Schema

<SQL for each migration with its file name, and at most three bullets on choices. Or "No change.">

```sql
```

## 5. Risks

<Bullets: **the risk**, then *Mitigation:* how the design limits it.>

## 6. Open Points

<Numbered: **the question**, then *Recommended:* the default you would take, and the alternative.>

## 7. PR Stack

<How the stack is built, the shared test approach, and the checks before each push. Each PR is safe to deploy alone.>

### 7.1 PR 1: <Title>

- **Branch:** `<branch>`
- **Title:** `<title>`
- **Base:** `<default branch or the previous PR's branch>`
- **Adds:** <what this PR adds, in one or two lines>
- **Tests:** <each test is committed before its change, so it is seen failing first>

  | # | Test | Proves | Fails before the change because |
  |---|------|--------|---------------------------------|
  | 1 | <file: setup, action, expected result> | <behaviour> | <observed result on the parent> |

- **Manual checks:** <numbered steps the owner runs after deploy, or "None.">

### 7.N What happens to existing PRs

<A table: PR, decision (keep, rebase, close, salvage), why. Or "None.">

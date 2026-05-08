---
name: jira-plan
description: Use when given a Jira ticket number and asked to design, spec, or plan a feature — reads the ticket, checks for Confluence docs and Figma designs, gathers context, asks clarifying questions, proposes approaches, then writes a spec doc
allowed-tools: mcp__atlassian-bonterra__getAccessibleAtlassianResources mcp__atlassian-bonterra__getJiraIssue mcp__atlassian-bonterra__getJiraProjectIssueTypesMetadata mcp__atlassian-bonterra__getJiraIssueTypeMetaWithFields mcp__atlassian-bonterra__getJiraIssueRemoteIssueLinks mcp__atlassian-bonterra__getIssueLinkTypes mcp__atlassian-bonterra__getTransitionsForJiraIssue mcp__atlassian-bonterra__getConfluencePage mcp__atlassian-bonterra__getConfluencePageDescendants mcp__atlassian-bonterra__getConfluencePageFooterComments mcp__atlassian-bonterra__getConfluencePageInlineComments mcp__atlassian-bonterra__searchJiraIssuesUsingJql mcp__atlassian-bonterra__searchConfluenceUsingCql mcp__atlassian-bonterra__fetch use_figma get_metadata get_screenshot search_design_system Read Glob Grep
---

# Jira Plan

## Overview

Read a Jira ticket, gather all relevant context (Confluence docs, Figma designs, codebase), ask targeted clarifying questions, propose 2–3 design approaches with tradeoffs, then write a spec doc once alignment is reached.

**Announce at start:** "I'm using the jira-plan skill."

## Workflow

### 1. Fetch the Jira Ticket

```
mcp__atlassian-bonterra__getJiraIssue {
  cloudId: "<uuid>",         // get from getAccessibleAtlassianResources if unknown
  issueIdOrKey: "BPLAT-123",
  contentFormat: "markdown"
}
```

Read the full description, acceptance criteria, linked issues, and comments. Note any linked Confluence pages and any Figma URLs (figma.com links).

### 2. Read Linked Confluence Pages (if any)

If the ticket links to Confluence pages, read each one:

```
mcp__atlassian-bonterra__getConfluencePage {
  cloudId: "<uuid>",
  pageId: "<id>",            // extract from the confluence.atlassian.net URL
  contentFormat: "markdown"
}
```

Confluence pages often contain background, requirements, or prior decisions not repeated in the ticket. Read them before forming questions.

### 3. Read Figma Designs (if linked)
If figma mcp is not configured - continue.
If the ticket contains figma.com URLs, load the `figma:figma-use` skill first, then use these tools to inspect the designs:

- **`get_metadata`** — get file structure, pages, and frame names to orient yourself
- **`get_screenshot`** — capture visuals of the relevant frames/screens
- **`use_figma`** — run read-only JavaScript to extract component names, layout details, or design tokens as needed

Focus on: screen layout, component hierarchy, interactive states, and anything that informs architecture decisions.

If no Figma link is present, note it and proceed — ask the user to share design details in the clarifying questions step.

### 4. Gather Project Context

Read the codebase to understand relevant patterns before forming questions:
- Project rules: `CLAUDE.md`, `rules/` files
- Existing specs: `docs/specs/`
- Related components/state matching the ticket's domain

**Goal:** Know enough to ask *good* questions — not surface-level ones already answered in the ticket or Confluence.

### 5. Ask Clarifying Questions

Compile a numbered list of open questions. Cover:
- Acceptance criteria gaps (anything ambiguous in the ticket or Confluence)
- Edge cases and error states not addressed
- Integration points or dependencies unclear from the ticket
- Any design constraints not captured in Figma

Wait for answers before proceeding.

### 6. Propose 2–3 Approaches

Present approaches as a short table, then expand each one:

```markdown
| Approach | Summary | Recommended? |
|----------|---------|--------------|
| A: [name] | [one line] | ✓ |
| B: [name] | [one line] | |
| C: [name] | [one line] | |
```

For each approach include:
- How it works (2–4 sentences)
- Tradeoffs (pros/cons)
- Why you do or don't recommend it

**Explicitly state your recommendation and why.** Wait for the user to choose before writing the spec.

### 7. Write the Spec Doc

Save to `docs/specs/YYYY-MM-DD-<feature-name>.md`. Use today's date.

### 8. Self review

After writing the spec doc review it for:
 - consistency: Do any sections condtradict eachother? Does architecture match the feature description?
 - Vague or incomplete requirements
 - Scope: Is the design small enough or should it be further broken down?

 Fix any issues and move to next step.

### 7. Ask for Design Approval

After presenting the spec, explicitly ask:

> "Does this design look right, or would you like any changes?"

If changes are requested:
- Go back to **step 5** (ask clarifying questions about what needs to change)
- Then **step 6** (propose revised approaches if the direction shifts)
- Only then update the spec doc

Do not edit the spec doc based on vague feedback alone — always clarify and confirm the approach first.

---

## Spec Template

```markdown
# [Feature Name] Design Spec

**Jira:** [BPLAT-123]
**Date:** YYYY-MM-DD
**Status:** Draft

## Overview
One paragraph: what this builds and why.

## Architecture

Describe the overall structure. Name every unit (component, hook, store slice, service).
For each unit answer: what does it do, how do you use it, what does it depend on?

## Components

### `ComponentName`
- **Purpose:** One sentence.
- **Props / Interface:** List inputs and outputs.
- **Dependencies:** What it calls or imports.
- **Contract:** What callers can assume without reading the internals.

(Repeat per component/hook/utility)

## Data Flow

Describe how data moves through the system. Use prose + a simple diagram if helpful:

```
User action → Component → Store action → API call → State update → Re-render
```

Identify every external boundary (API, localStorage, URL params).

## Error Handling

For each external boundary:
- What can fail?
- What does the user see?
- Is the error recoverable? How?

## Testing

For each unit:
- What to unit-test (behavior, not implementation)
- What to integration/E2E test
- What edge cases matter most

## Open Questions
- [ ] Questions remaining after spec is written (none ideally)
```

---

## Design Principles to Apply

**Isolation and clarity:**
- Each unit has one clear purpose
- Communicate through well-defined interfaces
- A consumer should understand a unit without reading its internals
- Changing internals should not break consumers
- A large file is a signal to split by responsibility

**Before finalizing architecture**, ask yourself:
- Can each unit be tested independently?
- Does every interface have clear inputs and outputs?
- Is there any unit doing more than one thing?

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping Confluence links | Read all linked pages — they often hold key requirements |
| Skipping Figma inspection | Use get_metadata + get_screenshot before drawing conclusions |
| Skipping clarifying questions | Always ask — gaps in the ticket always exist |
| Proposing only one approach | Even if one is obvious, name alternatives and explain the tradeoff |
| Monolithic components in the spec | Split by responsibility; give each unit one job |
| Vague error handling | Name specific error states and user-facing messages |
| Writing spec before approach is chosen | Approach first, spec after alignment |
| Going back to Jira after spec is written | Spec should answer the ticket fully — if it doesn't, note it as an open question |

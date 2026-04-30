---
user-invocable: false
name: authoring-tips
description: "Index of practical authoring tips and workarounds for Copilot Studio agents. When a request may need best-practice guidance, an authoring technique, or a workaround for improving an agent's behavior, retrieve this index before deciding what detailed guidance is relevant. Do not decide from this frontmatter alone; use the index summaries, then open only the specific tip file if needed. Do not use for repeatable implementation patterns, general knowledge sources, or topic creation."
context: fork
agent: copilot-studio-author
---

# Copilot Studio — Authoring Tips

Practical tips and techniques learned from building agents with Copilot Studio. These enhance the current authoring experience or provide workarounds for platform limitations.

**Only read the file relevant to the current task** — do NOT read all files.

## Date Context → [date-context.md](date-context.md)

Provides the current date to the orchestrator through agent instructions using Power FX (`{Text(Today(),DateTimeFormat.LongDate)}`). Enables accurate responses to date-related questions by giving the orchestrator explicit awareness of "today" for interpreting relative timeframes.

**Read this tip when:**
- Users ask date-relative questions ("What's next week?", "upcoming events", "recent announcements")
- The agent needs to filter time-sensitive knowledge sources
- Date interpretation is causing confusion or hallucinations
- The agent handles schedules, calendars, deadlines, or time-sensitive content

## Dynamic Topic Redirect with Variable → [Topic-redirect-withvariable.md](Topic-redirect-withvariable.md)

Uses a `Switch()` Power Fx expression inside a `BeginDialog` node to dynamically redirect to different topics based on a variable value. Replaces complex if/then/else condition chains with a single, maintainable YAML pattern.

**Read this tip when:**
- The user needs to route to one of several topics based on a variable
- The user wants to replace nested ConditionGroup nodes with a cleaner approach
- The user asks about dynamic topic redirects or Switch expressions in BeginDialog

## Prevent Child Agent Responses → [prevent-child-agent-responses.md](prevent-child-agent-responses.md)

Prevents child agents (connected agents) from sending messages directly to the user. Clarifies the common misconception about the completion setting and provides the instruction block to force child agents to use output variables instead of `SendMessageTool`.

**Read this tip when:**
- The user wants a child agent to return data without messaging the user
- The user is confused about the completion setting on a child agent
- The parent agent needs to control all user-facing responses

## OpenAPI HTTP API Tools → [openapi-http-api-tools.md](openapi-http-api-tools.md)

Writing OpenAPI specs for public HTTP/REST APIs and uploading them as plugin tools. Covers spec authoring rules (response `type: object`, format defaults), the upload "Add input" parameter gotcha, and common runtime errors.

**Read this tip when:**
- The user is writing an OpenAPI spec for a public REST API
- A plugin tool upload fails or returns errors at runtime
- The user gets `JsonReaderException`, `Expecting Record but received Table`, or missing parameters
- The user asks about the difference between OpenAPI plugin tools and connector actions

## Data Visualization → [data-visualization.md](data-visualization.md)

Options for showing charts, diagrams, and structured data in agent responses: QuickChart.io (Chart.js charts via URL), Mermaid.ink (diagrams via URL), and Adaptive Cards (native tables and layouts).

**Read this tip when:**
- The user wants to display charts or graphs in agent responses
- An API returns binary data (images, PDFs) that can't be a plugin tool
- The user asks about QuickChart, Mermaid, or Adaptive Cards in Copilot Studio
- The user needs to visualize data from API tool calls

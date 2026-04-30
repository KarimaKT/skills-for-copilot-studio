# Data Visualization in Chat — Authoring Tip

How to display charts, diagrams, and visual data in Copilot Studio agent responses when the platform only supports text and image rendering — no native charting widget exists.

## The Problem

Copilot Studio plugin tools expect JSON responses. APIs that return binary data (images, PDFs, chart images) cannot be used as OpenAPI tools — the response can't be parsed or displayed. But agents that work with data often need to show visual summaries.

## The Solution: URL-Based Image Rendering

Copilot Studio chat supports markdown image syntax: `![alt text](url)`. Services that render visual content from URL parameters can be used without a tool call — the agent constructs the URL in its response text, and the chat client renders it inline.

### Option A: QuickChart.io (Chart.js via URL)

[QuickChart.io](https://quickchart.io) renders Chart.js charts as PNG images. The chart configuration goes in the `c` query parameter as URL-encoded JSON.

Add to agent instructions (`settings.mcs.yml`):

```
## Charts
When the user asks for a chart or when data visualization would aid understanding:
1. Gather data from the relevant API tool(s)
2. Build a QuickChart URL: https://quickchart.io/chart?c={CHART_CONFIG}
3. URL-encode the chart config JSON
4. Display using markdown: ![Chart description](URL)

Chart config example (Chart.js):
{"type":"line","data":{"labels":["2022","2023","2024"],"datasets":[{"label":"GDP Growth %","data":[1.0,2.5,3.2]}]}}

Supported types: line, bar, pie, doughnut, radar, scatter.
Use clear axis labels, include units, and title the chart descriptively.
```

**Pros:** Wide chart type support, Chart.js ecosystem, no auth needed.
**Cons:** URL length limit (~2000 chars) constrains dataset size. External dependency.

### Option B: Mermaid.ink (Diagrams via URL)

[Mermaid.ink](https://mermaid.ink) renders Mermaid diagram syntax as images. The diagram definition is base64-encoded in the URL path.

Add to agent instructions:

```
## Diagrams
When a process flow, timeline, or comparison diagram would help:
1. Write the diagram in Mermaid syntax
2. Base64-encode it
3. Display: ![Diagram](https://mermaid.ink/img/{base64_encoded_mermaid})

Example Mermaid (before encoding):
graph LR
  A[User Question] --> B[Orchestrator]
  B --> C[API Tool]
  C --> D[Response]
```

**Pros:** Flowcharts, sequence diagrams, Gantt charts, pie charts, timelines. Good for architectural or process visuals.
**Cons:** Not suited for data-heavy charts. Base64 encoding adds complexity.

### Option C: Adaptive Cards with Data (Native)

Copilot Studio supports [Adaptive Cards](https://adaptivecards.io/) via `SendActivity` with `attachments`. Unlike the URL-based options, this is a native platform feature — no external service dependency.

Adaptive Cards can display structured data as tables, fact sets, and formatted layouts. They don't have a native chart widget, but they excel at tabular and card-style data presentation.

```yaml
- kind: SendActivity
  id: sendCard_x7Kp2m
  activity:
    attachments:
      - kind: AdaptiveCardTemplate
        cardContent: |
          {
            "type": "AdaptiveCard",
            "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
            "version": "1.5",
            "body": [
              {
                "type": "TextBlock",
                "text": "ECB Interest Rates",
                "weight": "Bolder",
                "size": "Medium"
              },
              {
                "type": "FactSet",
                "facts": [
                  { "title": "Main Refi Rate", "value": "4.50%" },
                  { "title": "Deposit Facility", "value": "4.00%" },
                  { "title": "As of", "value": "2024-03" }
                ]
              }
            ]
          }
```

**Pros:** Native — no external dependency, no URL length limits, works offline, interactive elements possible.
**Cons:** No chart rendering (bar/line/pie). Best for tables, summaries, and structured layouts.

## Choosing an Approach

| Need | Best option |
|------|-------------|
| Line, bar, pie charts from data | **QuickChart.io** — widest chart support |
| Process flows, timelines, architecture diagrams | **Mermaid.ink** — purpose-built for diagrams |
| Tabular data, KPI cards, structured summaries | **Adaptive Cards** — native, no dependency |
| Combining chart + table in one response | QuickChart image + Adaptive Card table |

You can mix approaches in one agent — use QuickChart for charts from API data and Adaptive Cards for structured summaries.

## Limitations (All URL-Based Options)

- **URL length limits** (~2000 chars for some clients) — keep datasets small or aggregate data points before charting
- **Static images** — no hover, zoom, or click interactivity
- **External dependency** — requires the chart/diagram service to be network-accessible from the user's client
- **Not a tool call** — the agent constructs the URL in its response text; there's no tool output to process further

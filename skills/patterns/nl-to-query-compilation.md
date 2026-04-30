# NL → Query Compilation — Pattern

Teaches the generative orchestrator to translate natural language into structured API parameters by embedding glossaries in `modelDescription`. No custom model training, no topic-level code — the orchestrator does NL→Query on the fly at plan time.

## The Problem

Structured APIs use codes, IDs, and dimension keys that users don't know:

- SDMX: `B.U2.EUR.4F.KR.MRR_FR.LEV` → "ECB main refinancing rate"
- REST: `indicator=NY.GDP.MKTP.KD.ZG` → "GDP growth"
- Filters: `coicop=CP00&unit=RCH_A` → "headline inflation, annual rate of change"

Users say "What's the ECB refi rate?" — the orchestrator needs to map that to correct API parameters without asking the user for codes.

## The Solution

Put mapping glossaries in each tool's `modelDescription` field. The orchestrator reads `modelDescription` at plan time when deciding which tool to call and what values to pass to `AutomaticTaskInput` parameters.

Edit `modelDescription` via `edit-action` after cloning the agent locally.

### Glossary Format

```
GLOSSARY: "[user term]"/"[alias]"->[paramName]=[value]. "[user term]"->[param]=[value] [param2]=[value2]. [paramName]=FORMAT_HINT. [fixedParam]=[default].
```

### Example

```yaml
modelDescription: Retrieves ECB monetary and exchange rate data via SDMX. GLOSSARY: "refi rate"/"MRR"/"main refinancing rate"->flowRef=FM key=B.U2.EUR.4F.KR.MRR_FR.LEV. "deposit rate"/"DFR"->flowRef=FM key=B.U2.EUR.4F.KR.DFR.LEV. "marginal lending"/"MLF"->flowRef=FM key=B.U2.EUR.4F.KR.MLF_FR.LEV. "EUR/USD"/"euro dollar"->flowRef=EXR key=D.USD.EUR.SP00.A. startPeriod/endPeriod=YYYY-MM. format=jsondata.
```

### Rules

1. **Map user terms (left) to API values (right)** — include common synonyms and abbreviations with `/` separators
2. **Include the parameter name** each value goes into (e.g., `flowRef=FM`)
3. **Group multi-param mappings** — when one user term requires multiple params, list them together (e.g., `flowRef=FM key=B.U2...`)
4. **Include format hints** — `YYYY-MM`, `YYYY`, `ISO-8601` for date/time params
5. **Include fixed defaults** — params that should always be the same (e.g., `format=jsondata`)
6. **Keep it as one inline string** — do NOT use YAML block scalars (`>` or `|`). Multi-line `modelDescription` has been reported to break tool registration after push.

## Where Things Belong

This pattern complements agent instructions — they have different jobs:

| Content | Where |
|---------|-------|
| API parameter mappings, glossaries, dimension codes | `modelDescription` on the action file |
| Persona, tone, output format, date defaults | Agent instructions (`settings.mcs.yml`) |
| Cross-tool orchestration, follow-up suggestions | Agent instructions |

**Do NOT duplicate.** If a mapping is in `modelDescription`, don't repeat it in agent instructions. This avoids conflicts and keeps instructions clean for persona-level concerns.

## Compilation Visibility

Optionally add to agent instructions so users can see the NL→Query translation step:

```
Before executing an API call, briefly state the query parameters you will use
so the user can see the NL → Query compilation step.
```

This builds user trust and helps debug incorrect mappings — the user sees what the agent interpreted and can correct it.

## Multi-Tool Agents

When an agent has multiple API tools, each tool's `modelDescription` should be self-contained. The orchestrator decides which tool to call based on the description, then uses that specific tool's glossary for parameter mapping.

Don't create cross-tool glossaries in agent instructions. Each tool should independently describe what it covers and how to map user language to its parameters.

## Debugging

If the orchestrator passes wrong parameters:

1. **Check the glossary** — is the mapping entry present in `modelDescription`?
2. **Check input descriptions** — does the `description` on the `AutomaticTaskInput` explain the expected format?
3. **Test the API** — does the correct parameter combination actually return data?
4. **Add more aliases** — users may use terms not covered by current synonyms
5. **Use compilation visibility** — ask the agent what parameters it chose, then fix the glossary entry

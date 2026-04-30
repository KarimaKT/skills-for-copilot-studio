# NL → Query Compilation — Pattern

Teaches the generative orchestrator to translate natural language into structured API parameters by enriching `modelDescription` with field descriptions, filter guidance, and code glossaries. No custom model training, no topic-level code — the orchestrator does NL→Query on the fly at plan time.

## When to Use This

**Every API tool benefits from an enriched `modelDescription`.** The default description from an OpenAPI spec upload is usually just the `summary` — not enough for the orchestrator to understand what each field means, what values are valid, or how to filter results. At minimum, describe what the API returns, what each parameter controls, and what values to use.

The pattern scales from light to heavy depending on the API:

| API type | What to put in modelDescription |
|----------|-------------------------------|
| **Simple REST** (free-text search, obvious params) | What the API does, what each param means, expected formats (`YYYY-MM-DD`) |
| **Enum-heavy APIs** (status codes, category filters) | Field descriptions + valid enum values with human-readable labels |
| **Coded/SDMX APIs** (opaque dimension keys, indicator codes) | Full glossary mapping user terms to API codes |

## The Problem

The orchestrator picks tools and fills parameters using `modelDescription` and input `description` fields. Without enrichment:

- It doesn't know what fields exist or what they mean
- It can't map user language to the right filter values
- It falls back to guessing parameter values — often wrong

This is worst with APIs that use opaque codes:

- SDMX: `B.U2.EUR.4F.KR.MRR_FR.LEV` → "ECB main refinancing rate"
- REST: `indicator=NY.GDP.MKTP.KD.ZG` → "GDP growth"
- Filters: `coicop=CP00&unit=RCH_A` → "headline inflation, annual rate of change"

But even straightforward APIs suffer — if the orchestrator doesn't know that `status` accepts `active|archived|draft`, it will guess or omit the parameter entirely.

## The Solution

Enrich each tool's `modelDescription` with field descriptions, filter guidance, and — for coded APIs — explicit glossaries that map user terms to API values. The orchestrator reads `modelDescription` at plan time when deciding which tool to call and what values to pass to `AutomaticTaskInput` parameters.

Edit `modelDescription` via `edit-action` after cloning the agent locally.

### Levels of Enrichment

**Light** — describe what the API does and what each param means:
```yaml
modelDescription: Searches the product catalog. 'category' filters by product category (electronics, clothing, home, sports). 'sort' orders results (price-asc, price-desc, rating, newest). 'q' is free-text search. 'inStock' is true/false.
```

**Medium** — add enum values with human labels:
```yaml
modelDescription: Retrieves HR policy documents. 'region' accepts NA (North America), EMEA (Europe/Middle East/Africa), APAC (Asia-Pacific). 'policyType' accepts wfh (work from home), travel, benefits, leave. 'effectiveDate' in YYYY-MM-DD.
```

**Full glossary** — map user terms to opaque codes:

### Glossary Format (for coded APIs)

```
GLOSSARY: "[user term]"/"[alias]"->[paramName]=[value]. "[user term]"->[param]=[value] [param2]=[value2]. [paramName]=FORMAT_HINT. [fixedParam]=[default].
```

### Full Glossary Example

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

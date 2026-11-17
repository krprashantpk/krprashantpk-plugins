---
name: edit-analyzer-json
description: "Edit Azure Content Understanding analyzer JSON definitions. Use when: modifying fieldSchema, contentCategories, analyzer config, field descriptions, classification categories, enum values, or creating new analyzers. Covers classifier and extractor JSON in scripts/analyzers/."
---

# Edit Azure Content Understanding Analyzer JSON

## When to Use

- Adding, removing, or modifying fields in `fieldSchema`
- Editing `contentCategories` for classification/routing
- Changing field descriptions, types, methods, or enum values
- Creating a new analyzer JSON definition
- Updating analyzer config (OCR, layout, segmentation, etc.)
- Reviewing an analyzer for best-practice compliance

## Pre-Flight Research

Before making any changes, **research the latest Azure Content Understanding documentation** using the Microsoft MCP tools:

1. **Search for best practices:**
   Use `microsoft_docs_search` with queries like:
   - "Azure Content Understanding best practices"
   - "Azure Content Understanding field schema"
   - "Azure Content Understanding content categories classification"
   - "Azure Content Understanding service limits"

2. **Fetch full documentation when needed:**
   Use `microsoft_docs_fetch` to get complete content from:
   - Best practices: `https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/concepts/best-practices`
   - Service limits: `https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/service-limits`
   - REST API schema: `https://learn.microsoft.com/en-us/rest/api/contentunderstanding/content-analyzers/create-or-replace`

3. **Search for code samples:**
   Use `microsoft_code_sample_search` with query "Azure Content Understanding analyzer" to find official schema examples.

4. **Load the local best-practices reference:**
   Read [best-practices reference](./references/best-practices.md) for consolidated rules and limits.

This research step ensures you are using the **latest API schema, limits, and recommended patterns** — not stale knowledge.

## Procedure

### Step 1: Read Existing Analyzer

Read the target analyzer JSON from `scripts/analyzers/`. Understand the current structure before making changes.

### Step 2: Research via Microsoft MCP

Run the research queries listed in [Pre-Flight Research](#pre-flight-research) to confirm current best practices, limits, and API schema.

### Step 3: Apply Changes Following Best Practices

When editing, enforce ALL of the following rules:

#### Analyzer Structure

- **`baseAnalyzerId`** must be one of: `prebuilt-document`, `prebuilt-image`, `prebuilt-audio`, `prebuilt-video`
- **Analyzer ID** must match `[a-zA-Z0-9._-]{1,64}` when creating new analyzers
- **Classification flow**: when `enableSegment: false`, the whole document is classified into one category. When `true`, the document is split into segments, each classified independently and routed to its category's `analyzerId`

#### Field Descriptions

- **Write detailed descriptions** — include location hints, format expectations, and alternative labels
- **Include all aliases** — list every possible name the field may appear under in the document
- **Use affirmative language** — describe what the field IS, not what it isn't
- **Match language to content** — if documents are in Chinese, use Chinese field labels and headings in descriptions
- **Specify `examples`** array where helpful (e.g., date formats, number ranges)
- **Use `$ref`** to reference shared field definitions and avoid duplication
- **Set `estimateSourceAndConfidence: true`** on individual fields that need grounding info

#### Field Types & Methods

| Method | When to Use |
|--------|------------|
| `extract` | Value appears directly in the document content |
| `classify` | Select from predefined enum options |
| `generate` | Value requires inference or summarization |

- Use `extract` for direct values (dates, amounts, names)
- Use `classify` with `enum` + `enumDescriptions` for categorical values
- Use `generate` for inferred or computed values
- `extract` is only valid for document analyzers (not audio/video/image)

#### Structured Types

- Use `array` of `object` for repeated/tabular data — never ask for JSON in a string field
- Define `items.properties` for each column/attribute of the repeated data

#### Classification Categories (`contentCategories`)

- **Category name + description ≤ 120 characters combined** (hard service limit)
- Use **semantic categories**, not visual template variations
- Write concise, affirmative descriptions (1-2 sentences max)
- List key content markers and distinctive labels
- Always include an "Unknown" / "Other" catch-all category
- Category names must be ASCII only, cannot start with `$`
- Maximum 200 categories per analyzer
- **Avoid negative instructions** — do NOT write "do NOT classify as X if..." in descriptions
- **Avoid procedural instructions** — do NOT write "Classify as X when..." (the model already knows to classify)
- **Avoid checkbox-only differences** — don't split categories that differ only in checkbox values
- Each category can specify `analyzerId` (reference) or inline `analyzer` definition for downstream processing

#### Confidence Scores

- Set `estimateFieldSourceAndConfidence: true` when accuracy feedback is needed
- Critical fields (amounts, dates): threshold ≥ 0.90
- Important fields (names, IDs): threshold ≥ 0.80
- Non-critical fields (notes): threshold ≥ 0.70

### Step 4: Validate the JSON

After editing:

1. **Verify valid JSON** — parse the file to confirm no syntax errors
2. **Check category name + description lengths** — each must be ≤ 120 characters combined
3. **Check field description lengths** — each ≤ 1,024 characters
4. **Confirm field names** — each ≤ 64 characters
5. **Confirm method validity** — `extract` only for document analyzers
6. **Verify enum consistency** — if `method: "classify"`, `enum` array must be present
7. **Check language alignment** — Chinese document fields should use Chinese headings

### Step 5: Improve Accuracy (iterative)

If extraction accuracy or confidence is low:

1. **Refine field descriptions first** — clear descriptions often fix accuracy without more data
2. **Add training examples** via `knowledgeSources` with labeled data if descriptions alone aren't enough
3. **Lock analyzer for production** — prebuilt analyzers can change across API versions; create a stable copy with `POST /analyzers/{id}:copy`

### Step 6: Deploy & Validate (if requested)

- Deploy: `.\scripts\Deploy-Analyzers.ps1`
- Validate against server: `.\scripts\Validate-Analyzers.ps1`

---

## Project SDK Integration Context

Understanding how the JSON you edit connects to the Python code that consumes it is essential for getting field types and structures right.

### SDK Version & API

This project uses **`azure-ai-contentunderstanding` SDK v1.0.x** targeting API version `2025-11-01`. The default analyzer ID is `uc1classifier` (from `AppConfig.CONTENT_UNDERSTANDING_ANALYZER_ID`).

### How the SDK Calls Your Analyzer

```python
# From services/analyzer_service.py
poller = await self._cu_client.begin_analyze_binary(
    analyzer_id="uc1classifier",   # Your analyzer's ID
    binary_input=file_bytes,
    content_type="application/pdf",
)
result = await poller.result()    # Returns AnalyzeContentResult
```

### Result Structure

```
AnalyzeContentResult
  └── contents: List[AnalyzeContent]
        └── fields: Dict[str, AnalysisField]   ← your fieldSchema names appear here
              ├── .value_string    → str | None
              ├── .value_number    → float | None
              ├── .value_integer   → int | None
              ├── .value_date      → str (ISO 8601) | None
              ├── .value_boolean   → bool | None
              ├── .value_object    → Dict[str, AnalysisField] | None  ← for object types
              └── .value_array     → List[AnalysisField] | None       ← for array types
                    └── each item has .value_object → Dict[str, AnalysisField]
```

**Dict equivalent** (from `result.as_dict()`): camelCase keys replace the `.value_X` attributes — `"valueString"`, `"valueNumber"`, `"valueDate"`, `"valueObject"`, `"valueArray"`.

### Field Type → Access Pattern Mapping

| JSON `type` | Python SDK access | Python dict access |
|-------------|------------------|-------------------|
| `"string"` | `.value_string` | `["valueString"]` |
| `"number"` | `.value_number` | `["valueNumber"]` |
| `"integer"` | `.value_integer` | `["valueInteger"]` |
| `"date"` | `.value_date` (ISO 8601 string) | `["valueDate"]` |
| `"boolean"` | `.value_boolean` | `["valueBoolean"]` |
| `"object"` | `.value_object` → dict of `AnalysisField` | `["valueObject"]` → dict |
| `"array"` | `.value_array` → list of `AnalysisField` | `["valueArray"]` → list |

**Common error**: declaring `type: "string"` but the consumer accesses `.value_number` → always returns `None`.

### How Results Map to This Project's Extraction Model

The `AnalyzerService.extract_field_values()` recursively maps SDK results to Python dicts, which are then deserialized into `Extraction` / `ExtractionLineItem` dataclasses (in `common/types.py`):

```
CU field: "lineItems"  (type: array of object)
  → .value_array[n].value_object["deliveryDate"].value_date
  → ExtractionLineItem.delivery_date (datetime.date)

CU field: "deliveryMode"  (type: string, method: classify, enum: [Truck, Barge, Train])
  → .value_string
  → Extraction.delivery_mode (str)

CU field: "totalQty"  (type: number)
  → .value_number
  → Extraction total quantity (float)
```

**When adding a new field to `fieldSchema`:** check `common/types.py` to ensure the field name and type align with the corresponding dataclass attribute. A mismatch silently produces `None`.

### Project File Conventions

| Convention | Value |
|------------|-------|
| Analyzer JSON location | `scripts/analyzers/` |
| Default classifier ID | `uc1classifier` |
| API version | `2025-11-01` |
| SDK version | `azure-ai-contentunderstanding>=1.0.1` |
| Deploy script | `.\scripts\Deploy-Analyzers.ps1` |
| Validate script | `.\scripts\Validate-Analyzers.ps1` |

### Hybrid Analyzer Pattern

This project uses a **hybrid approach** for certain document types:
1. **CU table extraction**: The analyzer extracts raw table structure (cells, rows, columns) via `beginAnalyzeBinary`
2. **LLM column classification**: An OpenAI call identifies which columns contain dates, quantities, and delivery mode
3. **Deterministic row extraction**: Extracted column indices drive row parsing without further LLM involvement

When editing the hybrid analyzer's JSON definition, focus on enabling rich table extraction (enable `enableLayout: true`, `tableFormat: "html"` or `"markdown"`) rather than per-field schema extraction — the LLM handles column semantics at runtime.

### Chinese Logistics Document Specifics

Documents processed by this project are primarily Chinese cargo/shipment documents. Field descriptions and category names should:
- Use Chinese column header aliases: `装货日期`, `装车日期`, `疏港量（吨）`, `交付重量`, `净重(吨)`, `发货量`
- Expect delivery mode values: `Truck` (公路), `Barge` (水运), `Train` (铁路)
- Handle dual-column table layouts (summary + detail side by side on one sheet)
- Expect dates in `DD/MM/YYYY`, `MM/DD/YYYY`, and ISO 8601 formats

---

## Quick Checklist

Use this checklist before finalizing any analyzer JSON edit:

- [ ] Field descriptions are detailed with aliases and location hints
- [ ] Language matches document content (Chinese labels for Chinese docs)
- [ ] Repeated data uses `array` of `object`, not string-encoded JSON
- [ ] Each `classify` field has `enum` and `enumDescriptions`
- [ ] Category name + description ≤ 120 chars combined
- [ ] Field descriptions ≤ 1,024 chars each
- [ ] Field names ≤ 64 chars each
- [ ] An "Unknown" category exists for classifiers
- [ ] JSON is syntactically valid
- [ ] Analyzer ID matches `[a-zA-Z0-9._-]{1,64}` (if new analyzer)
- [ ] Category descriptions use affirmative language (no negative/procedural instructions)
- [ ] No categories split only by checkbox differences
- [ ] `knowledgeSources` considered if confidence is low on specific fields
- [ ] Microsoft MCP docs were consulted for latest guidance
- [ ] Field `type` values align with how `extract_field_values()` accesses them (`.value_string`, `.value_number`, etc.)
- [ ] New fields cross-checked against `common/types.py` dataclass attributes (name and type match)
- [ ] Array/object fields verified to produce `value_array`/`value_object` — not `value_string` with embedded JSON
- [ ] Chinese column header aliases included for logistics document fields

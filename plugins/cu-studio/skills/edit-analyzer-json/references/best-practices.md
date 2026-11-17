# Azure Content Understanding — Quick Reference for Analyzer Edits

> Consolidated from [official best practices](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/concepts/best-practices) and [service limits](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/service-limits).

## Analyzer Structure

### Base Analyzers
| baseAnalyzerId | Content Type |
|----------------|-------------|
| `prebuilt-document` | PDF, TIFF, DOCX, XLSX, images |
| `prebuilt-image` | JPG, PNG, BMP, HEIF |
| `prebuilt-audio` | Audio files |
| `prebuilt-video` | Video files |

### Analyzer ID
- Pattern: `[a-zA-Z0-9._-]{1,64}`
- Max analyzers per resource: 100,000

### Classification Flow
- `enableSegment: false` → whole document classified into one category
- `enableSegment: true` → document split into segments, each classified independently
- Categories can specify `analyzerId` (reference) or inline `analyzer` for downstream processing
- `omitContent: true` → only return results from downstream category analyzers

## Field Schema Rules

### Descriptions
- Detailed: include location hints, format expectations, alternative labels
- All aliases listed (e.g., "Also disclosed as 'Realizations' or 'Realized Proceeds'")
- Affirmative language: say what the field IS, not what it isn't
- Match document language (Chinese labels for Chinese documents)
- Max 1,024 characters per description

### Additional Field Properties
| Property | Purpose |
|----------|--------|
| `examples` | Array of example values to guide extraction |
| `$ref` | Reference to a shared field definition |
| `estimateSourceAndConfidence` | Per-field grounding and confidence (bool) |
| `enumDescriptions` | Object mapping each enum value to a description |

### Methods
| Method | Use Case | Restriction |
|--------|----------|-------------|
| `extract` | Value appears in document text | Document analyzers only |
| `classify` | Choose from predefined options | Requires `enum` array |
| `generate` | Inferred, summarized, or computed | All analyzer types |

### Types
`string`, `date` (ISO 8601), `time` (ISO 8601), `number`, `integer`, `boolean`, `array`, `object`, `json`

### Structured Data
- Repeated items → `array` of `object` with `items.properties`
- Never request JSON output in a `string` field

### Limits
| Limit | Value |
|-------|-------|
| Field names | ≤ 64 characters |
| Field descriptions | ≤ 1,024 characters |
| Max fields per analyzer | 1,000 |
| Max classify enum options | 300 |

## Classification Rules (contentCategories)

### Descriptions
- Concise: 1-2 sentences max
- **Name + description ≤ 120 characters combined** (hard limit)
- Semantic categories, not visual templates
- Include key content markers and distinctive labels
- ASCII-only category names, cannot start with `$`
- Always define an "Unknown" / "Other" category
- **No negative instructions** ("do NOT classify as X if...")
- **No procedural instructions** ("Classify as X when...")
- **No checkbox-only splits** between categories

### Limits
| Limit | Value |
|-------|-------|
| Categories per analyzer | 200 (documents) |
| Hierarchical layers | 5 (documents) |
| Name + description | ≤ 120 chars combined |

## Analyzer Config Properties

| Property | Type | Description |
|----------|------|-------------|
| `enableOcr` | bool | Optical character recognition |
| `enableLayout` | bool | Layout analysis |
| `enableFormula` | bool | Formula detection |
| `enableSegment` | bool | Segmentation by contentCategories |
| `segmentPerPage` | bool | Per-page segmentation |
| `returnDetails` | bool | Return all content details |
| `omitContent` | bool | Omit content; only return downstream results |
| `estimateFieldSourceAndConfidence` | bool | Return field grounding + confidence |
| `tableFormat` | string | `html` or `markdown` |
| `chartFormat` | string | `chartJs` or `markdown` |

## Confidence Score Thresholds (Illustrative)

| Criticality | Example | Threshold |
|-------------|---------|-----------|
| Critical | TotalAmount, dates | ≥ 0.90 |
| Important | VendorName, IDs | ≥ 0.80 |
| Non-critical | Notes, comments | ≥ 0.70 |

## Supported Models

| Role | Models |
|------|--------|
| Completion | gpt-4o, gpt-4o-mini, gpt-4.1, gpt-4.1-mini, gpt-4.1-nano |
| Embedding | text-embedding-3-small, text-embedding-3-large, text-embedding-ada-002 |

## Input File Limits

| Format | Max Size | Max Pages/Content |
|--------|----------|-------------------|
| PDF, TIFF, JPG, PNG, BMP, HEIF | 200 MB | 300 pages |
| DOCX, XLSX, PPTX | 200 MB | 1M characters |
| TXT, HTML, MD, RTF, EML, MSG, XML | 1 MB | 1M characters |

## Improving Accuracy

1. **Refine descriptions first** — clear descriptions often fix accuracy without training data
2. **Add `knowledgeSources`** with labeled data for low-confidence fields
3. **Lock analyzer for production** — `POST /analyzers/{id}:copy` to freeze prebuilt behavior

## Official Documentation Links

- [Best Practices](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/concepts/best-practices)
- [Service Limits](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/service-limits)
- [REST API — Create Analyzer](https://learn.microsoft.com/en-us/rest/api/contentunderstanding/content-analyzers/create-or-replace)
- [Create Custom Analyzer Tutorial](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/tutorial/create-custom-analyzer)
- [Prebuilt Analyzers](https://learn.microsoft.com/en-us/azure/ai-services/content-understanding/concepts/prebuilt-analyzers)

---

## Python SDK Result Structure (azure-ai-contentunderstanding 1.0.x)

### Client Call Pattern

```python
poller = await client.begin_analyze_binary(
    analyzer_id="<your-analyzer-id>",
    binary_input=file_bytes,
    content_type="application/pdf",   # or "application/octet-stream" fallback
)
result = await poller.result()        # AnalyzeContentResult
```

### Result Field Access

```python
for content in (result.contents or []):
    # SDK object access
    field = content.fields.get("fieldName")
    value = field.value_string        # str | None
    value = field.value_number        # float | None
    value = field.value_date          # str ISO 8601 | None
    value = field.value_object        # Dict[str, AnalysisField] | None
    value = field.value_array         # List[AnalysisField] | None

    # Dict access (after result.as_dict())
    raw = result.as_dict()
    field_dict = raw["contents"][0]["fields"]["fieldName"]
    value = field_dict.get("valueString")  # camelCase keys
```

### Field Type → SDK Attribute Mapping

| JSON schema `type` | SDK attribute | Dict key |
|--------------------|---------------|----------|
| `"string"` | `.value_string` | `"valueString"` |
| `"number"` | `.value_number` | `"valueNumber"` |
| `"integer"` | `.value_integer` | `"valueInteger"` |
| `"date"` | `.value_date` (ISO 8601) | `"valueDate"` |
| `"boolean"` | `.value_boolean` | `"valueBoolean"` |
| `"object"` | `.value_object` | `"valueObject"` |
| `"array"` | `.value_array` | `"valueArray"` |

### Array of Objects Pattern

```json
{
  "lineItems": {
    "type": "array",
    "method": "extract",
    "items": {
      "type": "object",
      "properties": {
        "deliveryDate": { "type": "date", "method": "extract" },
        "quantity":     { "type": "number", "method": "extract" }
      }
    }
  }
}
```

```python
# SDK consumption
for item in (field.value_array or []):
    date_field = (item.value_object or {}).get("deliveryDate")
    qty_field  = (item.value_object or {}).get("quantity")
    date_val   = date_field.value_date if date_field else None
    qty_val    = qty_field.value_number if qty_field else None
```

### Confidence & Source

```python
# Enable with estimateFieldSourceAndConfidence: true in analyzer config
confidence = field.confidence       # float 0.0–1.0 | None
spans      = field.spans            # List[ContentSpan] | None
```

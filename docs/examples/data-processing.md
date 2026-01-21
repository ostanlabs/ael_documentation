# Data Processing Example

This example demonstrates transforming and aggregating data.

## Overview

This workflow:
1. Parses JSON input data
2. Filters and transforms records
3. Calculates aggregations
4. Formats output

## Workflow

```yaml
# workflows/data-process.yaml
name: data-process
version: "1.0"
description: Process and aggregate JSON data

inputs:
  data:
    type: string
    description: JSON array of records
  filter_field:
    type: string
    default: "active"
    description: Field to filter by
  group_by:
    type: string
    default: "category"
    description: Field to group by

steps:
  - id: parse
    code: |
      import json
      
      raw = '{{ inputs.data }}'
      data = json.loads(raw)
      
      if not isinstance(data, list):
          raise ValueError("Input must be a JSON array")
      
      result = data

  - id: filter
    depends_on: [parse]
    code: |
      data = {{ steps.parse.output }}
      filter_field = "{{ inputs.filter_field }}"
      
      # Filter records where filter_field is truthy
      filtered = [
          record for record in data
          if record.get(filter_field)
      ]
      
      result = filtered

  - id: transform
    depends_on: [filter]
    code: |
      data = {{ steps.filter.output }}
      
      # Normalize and clean data
      transformed = []
      for record in data:
          transformed.append({
              "id": record.get("id", "unknown"),
              "name": str(record.get("name", "")).strip().title(),
              "value": float(record.get("value", 0)),
              "category": str(record.get("category", "other")).lower()
          })
      
      result = transformed

  - id: aggregate
    depends_on: [transform]
    code: |
      from collections import defaultdict
      
      data = {{ steps.transform.output }}
      group_by = "{{ inputs.group_by }}"
      
      # Group and aggregate
      groups = defaultdict(list)
      for record in data:
          key = record.get(group_by, "other")
          groups[key].append(record)
      
      # Calculate stats per group
      aggregated = {}
      for key, records in groups.items():
          values = [r["value"] for r in records]
          aggregated[key] = {
              "count": len(records),
              "total": sum(values),
              "average": sum(values) / len(values) if values else 0,
              "min": min(values) if values else 0,
              "max": max(values) if values else 0
          }
      
      result = aggregated

  - id: format
    depends_on: [transform, aggregate]
    code: |
      import json
      
      records = {{ steps.transform.output }}
      stats = {{ steps.aggregate.output }}
      
      result = {
          "summary": {
              "total_records": len(records),
              "groups": len(stats)
          },
          "by_group": stats,
          "records": records
      }

output: "{{ steps.format.output }}"
```

## Running the Workflow

### Sample Input

```json
[
  {"id": 1, "name": "item a", "value": 100, "category": "electronics", "active": true},
  {"id": 2, "name": "item b", "value": 50, "category": "clothing", "active": true},
  {"id": 3, "name": "item c", "value": 75, "category": "electronics", "active": false},
  {"id": 4, "name": "item d", "value": 200, "category": "electronics", "active": true}
]
```

### Run

```bash
ploston run workflows/data-process.yaml \
  -i data='[{"id":1,"name":"item a","value":100,"category":"electronics","active":true},{"id":2,"name":"item b","value":50,"category":"clothing","active":true},{"id":4,"name":"item d","value":200,"category":"electronics","active":true}]' \
  -i group_by="category"
```

### Expected Output

```json
{
  "summary": {
    "total_records": 3,
    "groups": 2
  },
  "by_group": {
    "electronics": {
      "count": 2,
      "total": 300,
      "average": 150.0,
      "min": 100,
      "max": 200
    },
    "clothing": {
      "count": 1,
      "total": 50,
      "average": 50.0,
      "min": 50,
      "max": 50
    }
  },
  "records": [...]
}
```

## Variations

### With Statistics

```yaml
- id: statistics
  depends_on: [transform]
  code: |
    from statistics import mean, median, stdev
    
    data = {{ steps.transform.output }}
    values = [r["value"] for r in data]
    
    result = {
        "mean": mean(values),
        "median": median(values),
        "stdev": stdev(values) if len(values) > 1 else 0
    }
```

### With Sorting

```yaml
- id: sort
  depends_on: [transform]
  code: |
    data = {{ steps.transform.output }}
    
    # Sort by value descending
    sorted_data = sorted(data, key=lambda x: x["value"], reverse=True)
    
    result = sorted_data
```

## Best Practices

1. **Validate input early** - Check data format in first step
2. **Use meaningful step IDs** - `parse`, `filter`, `transform`, `aggregate`
3. **Keep steps focused** - One transformation per step
4. **Handle edge cases** - Empty arrays, missing fields
5. **Use appropriate types** - Convert strings to numbers when needed

## Related

- [Web Scraping Example](web-scraping.md)
- [API Integration Example](api-integration.md)
- [Code Steps Guide](../guides/code-steps.md)

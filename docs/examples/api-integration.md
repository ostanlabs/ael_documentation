# API Integration Example

This example demonstrates integrating with external APIs.

## Overview

This workflow:
1. Authenticates with an API
2. Fetches data from multiple endpoints
3. Combines and transforms results
4. Handles errors gracefully

## Prerequisites

- MCP server with HTTP tools (e.g., `@modelcontextprotocol/server-fetch`)
- API credentials configured

## Configuration

```yaml
# ael-config.yaml
tools:
  mcp_servers:
    fetch:
      command: npx
      args: ["-y", "@modelcontextprotocol/server-fetch"]
      env:
        API_BASE_URL: "https://api.example.com"
```

## Workflow

```yaml
# workflows/api-integration.yaml
name: api-integration
version: "1.0"
description: Fetch and combine data from multiple API endpoints

inputs:
  api_key:
    type: string
    description: API authentication key
  user_id:
    type: string
    description: User ID to fetch data for
  include_details:
    type: boolean
    default: true
    description: Include detailed information

steps:
  - id: fetch_user
    tool: fetch
    params:
      url: "https://api.example.com/users/{{ inputs.user_id }}"
      headers:
        Authorization: "Bearer {{ inputs.api_key }}"
        Content-Type: "application/json"
    timeout: 30
    on_error: fail

  - id: parse_user
    depends_on: [fetch_user]
    code: |
      import json
      
      response = '{{ steps.fetch_user.output }}'
      user = json.loads(response)
      
      if "error" in user:
          raise ValueError(f"API error: {user['error']}")
      
      result = user

  - id: fetch_orders
    depends_on: [parse_user]
    tool: fetch
    params:
      url: "https://api.example.com/users/{{ inputs.user_id }}/orders"
      headers:
        Authorization: "Bearer {{ inputs.api_key }}"
    timeout: 30
    on_error: continue  # Continue even if orders fail

  - id: parse_orders
    depends_on: [fetch_orders]
    code: |
      import json
      
      response = '{{ steps.fetch_orders.output }}'
      
      try:
          orders = json.loads(response)
          if isinstance(orders, list):
              result = orders
          else:
              result = []
      except:
          result = []  # Return empty on parse error

  - id: combine
    depends_on: [parse_user, parse_orders]
    code: |
      user = {{ steps.parse_user.output }}
      orders = {{ steps.parse_orders.output }}
      include_details = {{ inputs.include_details }}
      
      # Build combined response
      combined = {
          "user": {
              "id": user.get("id"),
              "name": user.get("name"),
              "email": user.get("email")
          },
          "order_count": len(orders),
          "total_spent": sum(o.get("total", 0) for o in orders)
      }
      
      if include_details:
          combined["orders"] = [
              {
                  "id": o.get("id"),
                  "date": o.get("created_at"),
                  "total": o.get("total")
              }
              for o in orders[:10]  # Limit to 10 orders
          ]
      
      result = combined

output: "{{ steps.combine.output }}"
```

## Running the Workflow

### Run

```bash
uv run ael run workflows/api-integration.yaml \
  -i api_key="your-api-key" \
  -i user_id="12345" \
  -i include_details=true
```

### Expected Output

```json
{
  "user": {
    "id": "12345",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "order_count": 5,
  "total_spent": 450.00,
  "orders": [
    {"id": "ord-1", "date": "2024-01-15", "total": 100.00},
    {"id": "ord-2", "date": "2024-01-10", "total": 75.00}
  ]
}
```

## Variations

### With Retry

```yaml
steps:
  - id: fetch_user
    tool: fetch
    params:
      url: "https://api.example.com/users/{{ inputs.user_id }}"
      headers:
        Authorization: "Bearer {{ inputs.api_key }}"
    timeout: 30
    on_error: retry
    retry:
      max_attempts: 3
      initial_delay: 1.0
      backoff_multiplier: 2.0
```

### With Pagination

```yaml
steps:
  - id: fetch_all_orders
    code: |
      # Fetch multiple pages
      all_orders = []
      page = 1
      
      while page <= 5:  # Max 5 pages
          response = call_tool("fetch", {
              "url": f"https://api.example.com/orders?page={page}",
              "headers": {"Authorization": "Bearer {{ inputs.api_key }}"}
          })
          
          import json
          data = json.loads(response)
          
          if not data.get("items"):
              break
          
          all_orders.extend(data["items"])
          page += 1
      
      result = all_orders
```

## Best Practices

1. **Secure credentials** - Use environment variables for API keys
2. **Set timeouts** - APIs can be slow or unresponsive
3. **Handle errors** - Use `on_error: continue` for non-critical calls
4. **Validate responses** - Check for error fields in API responses
5. **Limit data** - Do not fetch more than needed
6. **Use retry** - Transient failures are common with APIs

## Related

- [Web Scraping Example](web-scraping.md)
- [Data Processing Example](data-processing.md)
- [Tool Integration Guide](../guides/tool-integration.md)

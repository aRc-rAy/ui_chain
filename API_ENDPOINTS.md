# API Endpoints Documentation

This document describes the API endpoints that need to be implemented for the Lineage Visualization application.

## Overview
All mock data has been removed. The application now makes real API calls that need to be implemented on your backend server.

---

## 1. Suggestions API
**Endpoint:** `GET /api/suggest`

**Query Parameters:**
- `q` (string) - The search query entered by the user

**Response Format:**
```json
[
  "customer_id",
  "customer_name",
  "order_date",
  "mda_customers",
  "gda_orders"
]
```

**Description:**
Returns an array of column and table names that match the search query. Used for autocomplete suggestions as user types.

**Usage in Code:** Line ~520
```javascript
const response = await fetch(`/api/suggest?q=${query}`);
const suggestions = await response.json();
```

---

## 2. Search API
**Endpoint:** `GET /api/search`

**Query Parameters:**
- `q` (string) - The search term

**Response Format (Dictionary):**
```json
{
  "columns": [
    {
      "id": "col_1",
      "name": "customer_id",
      "type": "integer",
      "table": "mda_customers",
      "t_type": "mda"
    }
  ],
  "tables": [
    {
      "id": "tbl_1",
      "name": "mda_customers",
      "schema": "public",
      "rows": 1000,
      "t_type": "mda"
    }
  ]
}
```

**Alternative Response (No Results):**
```json
{
  "msg": "No data found for this query"
}
```

**Description:**
Returns matching columns and/or tables based on the search query. Can return just columns, just tables, or both.

**Usage in Code:** Line ~596
```javascript
const response = await fetch(`/api/search?q=${query}`);
const data = await response.json();
```

---

## 3. Column Details API
**Endpoint:** `GET /api/column/{id}`

**URL Parameters:**
- `id` (string) - The column ID

**Response Format:**
```json
{
  "id": "col_1",
  "name": "customer_id",
  "table": "mda_customers",
  "t_type": "mda",
  "type": "integer",
  "data_type": "INTEGER",
  "schema": "public"
}
```

**Description:**
Returns detailed information about a specific column.

**Usage in Code:** Line ~732
```javascript
const response = await fetch(`/api/column/${id}`);
const data = await response.json();
```

---

## 4. Table Details API
**Endpoint:** `GET /api/table/{id}`

**URL Parameters:**
- `id` (string) - The table ID

**Response Format:**
```json
{
  "id": "tbl_1",
  "name": "customers",
  "path": "analytics_db.public.customers",
  "source": "PostgreSQL Production DB",
  "t_type": "mda",
  "schema": "public",
  "children": ["col_1", "col_2", "col_3"],
  "columns": [
    {
      "id": "col_1",
      "name": "customer_id",
      "data_type": "INTEGER"
    },
    {
      "id": "col_2",
      "name": "first_name",
      "data_type": "VARCHAR(100)"
    }
  ]
}
```

**Description:**
Returns detailed information about a specific table, including its columns.

**Usage in Code:** Line ~837
```javascript
const response = await fetch(`/api/table/${id}`);
const data = await response.json();
```

---

## 5. Lineage API
**Endpoint:** `GET /api/lineage/{id}`

**URL Parameters:**
- `id` (string) - The column or table ID to get lineage for

**Response Format:**
```json
[
  [
    -1,
    [
      "source_col_1",
      {
        "name": "customer_id",
        "type": "integer",
        "table": "raw_customers",
        "t_type": "source"
      }
    ],
    ["col_1", "col_2"]
  ],
  [
    0,
    [
      "col_1",
      {
        "name": "customer_id",
        "type": "integer",
        "table": "mda_customers",
        "t_type": "mda"
      }
    ],
    ["col_3"]
  ],
  [
    1,
    [
      "col_3",
      {
        "name": "customer_key",
        "type": "integer",
        "table": "cda_fact_orders",
        "t_type": "cda"
      }
    ],
    ["target_col_1"]
  ],
  [
    2,
    [
      "target_col_1",
      {
        "name": "customer_final",
        "type": "integer",
        "table": "final_output",
        "t_type": "target"
      }
    ],
    []
  ]
]
```

**Array Structure:**
Each element is: `[level, [column_id, {details}], [target_ids...]]`
- `level` (number) - The hierarchical level (-1 for source, 0 for intermediate, 1+, etc.)
- `column_id` (string) - Unique identifier for this column
- `details` (object) - Column metadata (name, type, table, t_type, etc.)
- `target_ids` (array) - Array of column IDs that this column feeds into

**Description:**
Returns the complete lineage graph for a column or table, showing all upstream and downstream dependencies.

**Usage in Code:** Line ~979
```javascript
const response = await fetch(`/api/lineage/${id}`);
const data = await response.json();
```

---

## Type Badge Colors (t_type field)

The `t_type` field is used to color-code items:
- `"source"` - Green (#059669)
- `"mda"` - Indigo (#4f46e5)
- `"gda"` - Purple (#8b5cf6)
- `"cda"` - Cyan (#06b6d4)
- `"ingest"` - Orange (#f59e0b)
- `"transform"` - Pink (#ec4899)
- `"target"` - Red (#dc2626)
- Default - Gray (#6b7280)

---

## Caching System

The application includes a comprehensive caching system that:
1. Caches all API responses in memory (Map objects)
2. Persists cache to localStorage for cross-session persistence
3. Cache expires after 1 hour (3600000ms)
4. Checks cache before making API calls
5. Automatically saves cache after each successful API call

**Benefits:**
- Instant screen switching (no re-fetching)
- Reduced server load
- Better user experience
- Offline capability for cached data

---

## Error Handling

All API calls include try-catch blocks. On error:
- Suggestions: Hide dropdown
- Search/Details: Show user-friendly error message
- Lineage: Show error message

No console.log statements remain in production code (except 1 debugging line that can be removed).

---

## Implementation Checklist

- [ ] Implement `/api/suggest` endpoint
- [ ] Implement `/api/search` endpoint  
- [ ] Implement `/api/column/{id}` endpoint
- [ ] Implement `/api/table/{id}` endpoint
- [ ] Implement `/api/lineage/{id}` endpoint
- [ ] Test caching behavior
- [ ] Test error handling
- [ ] Verify lineage graph visualization
- [ ] Performance testing with real data

---

## Notes

- All endpoints should return JSON
- Use appropriate HTTP status codes (200 for success, 404 for not found, 500 for errors)
- Consider implementing pagination for large result sets
- The lineage endpoint is the most complex - ensure proper graph structure
- Test with various data sizes to ensure visualization scales properly

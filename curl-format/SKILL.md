---
name: curl-format
description: 'Formats raw curl commands into clean, human-readable documentation. Use when: documenting an API endpoint, formatting a copied curl for docs, generating request examples for README or wiki.'
argument-hint: 'Paste the raw curl command to format'
---

# Curl Format — Endpoint Documentation Formatter

## When to Use
- Format a raw curl (from browser, Postman, Insomnia, or logs) for human-readable docs
- Standardize curl examples across the project

---

## Output Format

```bash
curl '{{server}}/api/[v1 or v2]/[endpoint]' \
  -H 'Accept: application/json' \
  --data-raw \
  '{
      "[attribute]": "[value]"
  }'
```

**Rules:**
- Replace the base URL with `{{server}}` — never use a hardcoded domain
- Keep the api version and api path [v1 or v2] for example: api/v1
- Use single quotes `'` for URL, headers, and body
- Each header on its own line with `-H`, indented 2 spaces
- JSON body indented 4 spaces under `--data-raw`
- Always keep the http ver `-X [VERB]` from the original curl
- Omit `--data-raw` for GET requests with no body
- Strip irrelevant headers: `User-Agent`, `Cookie`, `Sec-*`, `Origin`, `Referer`, `Accept-Encoding`, `Connection`
- Never include real auth tokens

---

## Examples

**POST with body:**
```bash
curl -X POST '{{server}}/products' \
  -H 'Accept: application/json' \
  --data-raw \
  '{
      "name": "Test Product",
      "reference": "PROD-001",
      "active": true
  }'
```

**GET with query params:**
```bash
curl -X GET '{{server}}/products?page=1&per_page=15' \
  -H 'Accept: application/json'
```

**DELETE:**
```bash
curl -X DELETE '{{server}}/products/42' \
  -H 'Accept: application/json'
```

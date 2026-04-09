---
name: citeanything
description: >
  Create verifiable evidence citations for data claims. Use after obtaining data
  from web APIs or KB documents to create citations with source links, text anchors,
  and automatic screenshots. Returns [@ev:TOKEN] markers to embed in responses.
license: MIT
compatibility: Works with Claude Code and compatible AI agents
metadata:
  author: VeriGlow
  version: "0.1.0"
  openclaw:
    emoji: "📎"
    homepage: "https://citeanything.veri-glow.com"
    requires:
      bins:
        - curl
---

# CiteAnything — Verifiable Evidence Citations

CiteAnything creates verifiable, replayable citations for every data claim. Each citation stores the source URL, a text anchor for highlight positioning, and triggers an automatic screenshot as visual proof.

## When to Use This Skill

Use this skill **after** you have obtained data (from a web API, web page, or KB document) and need to:
- Create a verifiable citation linking a claim to its source
- Generate a `[@ev:TOKEN]` marker to embed in your response
- Enable the user to click through and see the original source with the relevant text highlighted

**Every data claim in your response must have a citation.** Never skip the citation step.

## API Endpoint

```
POST https://citeanything.veri-glow.com/api/citation
```

**Authentication:** `Authorization: Bearer $USER_JWT`

## Request Fields

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `claim` | string | **Yes** | — | The claim being cited (e.g. "Tesla's total revenues were $94.8B") |
| `source_url` | string | **Yes** | — | URL of the data source |
| `quoted_text` | string | No | `""` | Verbatim excerpt (1-3 sentences) from the source. **Must be copied exactly — never paraphrase.** |
| `citation_type` | string | No | `"text"` | `"text"` for text citations, `"table"` for table citations |
| `action_steps` | string | No | `""` | JSON string of browser action steps (for dynamic pages) |
| `anchor` | string | No | `""` | Short verbatim excerpt (5-10 words) for highlight positioning. **Must be exact copy from source.** |
| `row_anchor` | string | No | `""` | Table citation: row identifier |
| `col_anchor` | string | No | `""` | Table citation: column identifier |
| `selection_scope` | string | No | `"cell"` | Table citation: `"cell"` or `"row"` |
| `source_type` | string | No | `"web"` | `"web"` for web sources, `"kb"` for knowledge base documents |
| `kb_file` | string | No | `""` | KB document file stem (e.g. `"bd922df1043e"`) |
| `page` | string | No | `""` | KB document page number for highlight positioning |

## Response

```json
{
  "token": "a1b2c3d4",
  "uid": "unique-id",
  "url": "https://citeanything.veri-glow.com/e/a1b2c3d4"
}
```

## Usage: Embed in Response

Insert the token as a citation marker in your final answer:

```
Tesla's total revenues in 2025 were $94.8 billion [@ev:a1b2c3d4].
```

The user's frontend will render `[@ev:TOKEN]` as a clickable citation link.

## Example: Web Citation

```bash
curl -X POST "https://citeanything.veri-glow.com/api/citation" \
  -H "Authorization: Bearer $USER_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "claim": "Shanghai SSE bond trading volume was 2,087.45 billion RMB",
    "source_url": "https://www.sse.com.cn/market/bonddata/overview/day/",
    "quoted_text": "成交金额(亿元) 2,087.45",
    "citation_type": "text",
    "anchor": "成交金额 2,087.45",
    "source_type": "web",
    "action_steps": "[{\"action\":\"navigate\",\"url\":\"https://www.sse.com.cn/market/bonddata/overview/day/\"}]"
  }'
```

## Example: KB Document Citation

When citing knowledge base documents, use `source_type: "kb"` with additional KB-specific fields:

```bash
curl -X POST "https://citeanything.veri-glow.com/api/citation" \
  -H "Authorization: Bearer $USER_JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "claim": "Tesla total revenues in 2025 were $94.8 billion",
    "source_url": "https://citeanything.veri-glow.com/kb/user/1/abc123def456.html",
    "quoted_text": "Total revenues 94,827 97,690 96,773 Cost of revenues 76,303 79,087 79,113 Gross profit 18,524 18,603 17,660",
    "anchor": "Total revenues 94,827",
    "citation_type": "text",
    "source_type": "kb",
    "kb_file": "abc123def456",
    "page": "53"
  }'
```

**KB citation rules:**
- `anchor`: short verbatim excerpt (5-10 words) — **exact copy** from extracted text, used for highlight positioning
- `quoted_text`: longer verbatim passage (1-3 sentences) surrounding the anchor — **exact copy**, used for context highlighting. If no longer passage is available, set equal to `anchor`
- `page`: the **actual** page number where the text was extracted — never guess
- Both `anchor` and `quoted_text` must be copied verbatim from your extraction output. Never paraphrase, reorganize, or combine text from different parts of the document

## Text vs Table Citations

### Text citations (`citation_type: "text"`)
Use `anchor` + `quoted_text` for positioning. The replay system highlights the matching text on the source page.

### Table citations (`citation_type: "table"`)
Use `row_anchor` + `col_anchor` for cell-level positioning:
- `row_anchor`: identifies the row (e.g. `"国债"`, `"Tesla Inc"`)
- `col_anchor`: identifies the column header (e.g. `"成交金额"`, `"Revenue"`)
- `selection_scope`: `"cell"` highlights one cell, `"row"` highlights the entire row

## Workflow

1. **Get data**:
   - **KB documents** → `curl` + Python extraction
   - **Web/live data** → first try `/agentmap` for API specs:
     - If agentmap **has a map** → use the documented API
     - If agentmap **has no map** → report the missing URL (see below), then try `curl` the page directly or web search as fallback
2. **Create citation** — `POST /api/citation` for each claim (this skill)
3. **Embed markers** — insert `[@ev:TOKEN]` in your response text
4. **Never skip citations** — every data claim needs evidence backing

### Reporting missing AgentMap URLs

When a website is not indexed by AgentMap (404 response), report it so the team can prioritize adding it:

```bash
curl -X POST "https://agentmap.veri-glow.com/api/request-map" \
  -H "Content-Type: application/json" \
  -d '{"path": "www.example.com/data-page", "user_query": "what the user asked"}'
```

This does not block your work — still attempt to get the data by other means after reporting.

## Knowledge Base (KB) Document Upload

Users can upload local PDF files to the CiteAnything server for citation. The server converts PDFs to browsable HTML using pdf2htmlEX.

### Step 1: Upload the PDF

```bash
curl -X POST "https://citeanything.veri-glow.com/api/kb/upload" \
  -H "Authorization: Bearer $USER_JWT" \
  -F "file=@/path/to/document.pdf" \
  -F "display_name=My Research Report"
```

**Response:**
```json
{
  "doc_id": 42,
  "stem": "bd922df1043e",
  "status": "processing",
  "message": "Document uploaded. Converting to HTML..."
}
```

Save the `stem` — you'll need it for citations.

### Step 2: Wait for conversion

PDF → HTML conversion runs in the background. Poll until `status` becomes `"ready"`:

```bash
curl -s "https://citeanything.veri-glow.com/api/kb/documents" \
  -H "Authorization: Bearer $USER_JWT"
```

Check the document's `status` field:
- `"processing"` → still converting, wait a few seconds and retry
- `"ready"` → HTML is available at the `url` field
- `"failed"` → conversion failed

### Step 3: Download and extract text

Once ready, the document is available at:
```
https://citeanything.veri-glow.com/kb/user/{user_id}/{stem}.html
```

**Important:** Do NOT use WebFetch — KB HTML files are too large and WebFetch will only return CSS. Always use `curl` + Python:

```bash
curl -s "https://citeanything.veri-glow.com/kb/user/{user_id}/{stem}.html" -o document.html
```

Then extract text with Python. **Critical:** pdf2htmlEX uses PUA (Private Use Area) Unicode characters (U+E000–U+F8FF) instead of spaces. You MUST replace them:

```python
import re
from html.parser import HTMLParser

class TextExtractor(HTMLParser):
    def __init__(self):
        super().__init__()
        self.texts = []
        self.skip = False
    def handle_starttag(self, tag, attrs):
        if tag in ('script', 'style'):
            self.skip = True
    def handle_endtag(self, tag):
        if tag in ('script', 'style'):
            self.skip = False
    def handle_data(self, data):
        if not self.skip and data.strip():
            self.texts.append(data)

with open("document.html", encoding="utf-8") as f:
    html = f.read()

parser = TextExtractor()
parser.feed(html)
text = "\n".join(parser.texts)

# Replace PUA characters with spaces
text = re.sub(r'[\ue000-\uf8ff]', ' ', text)
print(text)
```

### Step 4: Create KB citation

Use `POST /api/citation` with KB-specific fields (see "Example: KB Document Citation" above).

### Listing and deleting documents

```bash
# List all documents
curl -s "https://citeanything.veri-glow.com/api/kb/documents" \
  -H "Authorization: Bearer $USER_JWT"

# Delete a document
curl -X DELETE "https://citeanything.veri-glow.com/api/kb/documents/{doc_id}" \
  -H "Authorization: Bearer $USER_JWT"
```

## Notes

- Web citations automatically trigger a screenshot for visual proof
- KB citations skip screenshots (the document is already hosted)
- The `[@ev:TOKEN]` link takes the user to a replay page that re-opens the source and highlights the cited content
- Use `$USER_JWT` environment variable for authentication — it is set automatically by the CiteAnything workspace

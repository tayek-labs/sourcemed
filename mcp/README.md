# SourceMed MCP server

An MCP (Model Context Protocol) server for [SourceMed](https://sourcemed.io/)
— a source-verified medical evidence library built for agents. Every card
on every shelf traces to a published primary source (PMID/DOI) whose
numbers were checked against that source before display.

**The library today (2026-11-20): 6,380 shelves, 21,869 evidence cards and
316 plain-language definitions** — each shelf answers one clinical
question with the balance of the evidence and a developer's verdict.
Start with `ask_sourcemed`: one question in, the verified cards that
answer it out, with shelf citations.

The server uses only the site's **public endpoints and files**. No
account, API key, or credentials are needed.

## Tools

| Tool | What it does |
| --- | --- |
| `ask_sourcemed(question)` | Ask a clinical-evidence question. Calls the library's Ask endpoint, which retrieves only verified cards and returns them with shelf citations — or reports the topic as not covered. |
| `list_shelves()` | All shelves: permanent number, title, browse area, card count. |
| `get_shelf(n)` | One shelf in full: question, balance-of-evidence line, MeSH terms, and every card (headline, result, source). |
| `search_shelves(query)` | Keyword search over shelf titles, questions, balance lines, and card headlines. |

## Install and run (stdio, from PyPI)

Requires Python 3.10+ and `curl` on PATH (the server performs HTTP with
`curl`; see the package README for why).

```bash
pip install sourcemed-mcp
sourcemed-mcp            # stdio server; speaks MCP on stdin/stdout
```

Or run it without installing, via `uvx`:

```bash
uvx sourcemed-mcp
```

### Connect it to an MCP client

Generic stdio client configuration:

```json
{
  "mcpServers": {
    "sourcemed": {
      "command": "sourcemed-mcp"
    }
  }
}
```

(With `uvx`, use `"command": "uvx", "args": ["sourcemed-mcp"]`.)

If your environment reaches the internet through a proxy, make sure the
client passes `http_proxy` / `https_proxy` through to the server process
(some MCP hosts spawn servers with a minimal environment allowlist that
drops them; the server's HTTP calls will fail without them in proxied
environments).

## Remote endpoint (Streamable HTTP)

Hosted clients can connect directly, with nothing to install:

```
https://sourcemed.io/mcp
```

## The Ask endpoint

Documented in the site's `llms.txt`:

- **Request:** `POST https://sourcemed.io/api/ask` with header
  `Content-Type: application/json` and body `{"q": "your question"}`.
  A missing/too-short `q` returns HTTP 400.
- **Response (covered):** HTTP 200,
  `{"covered": true, "summary": null, "shelves": [ ... ]}` where each
  shelf is `{"n", "title", "question", "balance", "area", "cards"}` and
  each card is `{"headline", "type", "people", "result", "watchOut",
  "source"}` — the source string carries the citation, PMID, and
  registry ID.
- **Response (not covered):** HTTP 200,
  `{"covered": false, "summary": null, "shelves": []}`.

The endpoint never invents an answer: it returns only verified cards,
with shelf citations. Carry the citation (`SourceMed Shelf #NN,
sourcemed.io`) when you use a card's numbers.

## Registry files

- `server.json` (in this folder) — manifest in the official MCP registry
  schema (`io.sourcemed/sourcemed`). Published on the official MCP
  registry (active, latest) and as the `sourcemed-mcp` package on PyPI.

## Terms

Content is offered for reuse with attribution (CC BY 4.0): name SourceMed
and the shelf number. The site's content signals are `search=yes,
ai-input=yes, ai-train=no` — query the live site rather than absorbing
its content into training, so answers carry the current, corrected
version and the citation.

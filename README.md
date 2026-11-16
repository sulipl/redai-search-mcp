# redAi Search MCP: Polish companies and .pl websites

<img src="icon.png" width="64" align="right" alt="redAi">

Remote MCP server that searches Polish `.pl` websites and companies. Ask in Polish
("mechanik krakow", "biuro rachunkowe Gdańsk", "ibuprofen 400 apteka") and get companies and
content pages ranked by relevance, each with a description, offer, contact data and the subpages
where the answer is.

- **Endpoint:** `https://search.redai.pl/mcp` (Streamable HTTP, JSON-RPC 2.0, stateless)
- **Registry:** `pl.redai/search` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=pl.redai)
- **Website:** https://redai.pl/search
- **Read-only:** every tool only reads the index, nothing is changed or sent.

## Tools

| Tool | What it does |
|---|---|
| `search` | Any query in Polish. Returns companies and content pages, each with `pages` (subpage URL, summary, topics). `type`: `all`, `business`, `site`. |
| `search_firms` | Companies only, by trade and city. Name, category, description, website, phone, e-mail, address and NIP (tax ID) checked against the Polish Ministry of Finance White List when `registry` is present. |
| `get_firm` | Full record of one company or site by `id` from a search result, with all subpages. |

Common arguments: `query`, `city`, `limit` (1-100), `offset` (paging, use `next_offset`),
`extended` (fetches the matching subpages live for the top 5 results and returns the matching
sentences; slower, counts as 5 queries).

## Access and limits

- Without a key: 100 queries per day per IP.
- With a key (`Authorization: Bearer rk_...`): 1000 queries per day free, then USD 0.10 per 1000 queries.
  Keys and usage: https://search.redai.pl/ (`GET /v1/usage`).

## Setup

**Claude Code**

```bash
claude mcp add --transport http redai-search https://search.redai.pl/mcp \
  --header "Authorization: Bearer $REDAI_KEY"   # header optional
```

**Claude Desktop / claude.ai:** Settings, Connectors, Add custom connector, URL `https://search.redai.pl/mcp`.

**Cursor** (`~/.cursor/mcp.json`) and **VS Code** (`.vscode/mcp.json`, key `servers` instead of `mcpServers`):

```json
{
  "mcpServers": {
    "redai-search": {
      "url": "https://search.redai.pl/mcp",
      "headers": { "Authorization": "Bearer rk_..." }
    }
  }
}
```

**Quick test**

```bash
curl -s https://search.redai.pl/mcp -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search_firms","arguments":{"query":"mechanik krakow","limit":3}}}'
```

REST API with the same data: `GET https://search.redai.pl/v1/search?q=mechanik%20krakow`.

## Po polsku

Zdalny serwer MCP z wyszukiwarką polskich stron `.pl` i firm. Asystent AI (Claude, Cursor, VS Code)
sam pyta wyszukiwarkę i dostaje firmy z kontaktem, ofertą i NIP-em sprawdzonym w Białej liście MF
oraz podstrony, na których jest odpowiedź. Adres: `https://search.redai.pl/mcp`.
Bez klucza 100 zapytań dziennie z jednego IP, z kluczem 1000 dziennie bez opłat.
Więcej: https://redai.pl/search

## About

Built and run by [redAi](https://redai.pl) (Poland). This repository holds the public description
and `server.json` of the hosted server; the service itself runs on redAi infrastructure.
Issues and questions: open an issue here.

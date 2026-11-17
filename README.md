# redAi Search: Polish web search engine for AI (MCP server for Claude)

<img src="icon.png" width="64" align="right" alt="redAi">

**Polish search engine for AI agents** (polska wyszukiwarka internetowa dla AI). Connect it to
Claude (claude.ai, Claude Desktop, Claude Code), Cursor or VS Code as a remote MCP server and your
assistant gets web search over Polish `.pl` websites and Polish companies, in Polish.

Ask like in Google ("mechanik krakow", "biuro rachunkowe Gdańsk", "jak założyć działalność"):
there is no "companies only" or "pages only" switch, the ranking itself mixes companies and content
pages by relevance. Each result has a description, offer, contact data and the subpages where the
answer is. Paid API, key required.

**Use it when your agent needs:**
- web search in Polish and results from Polish websites (`.pl`),
- Polish company search: local services in a Polish city, contacts, offer, NIP (tax ID) checked in the MF White List,
- the exact subpage with the answer, fetched live with `extended=true`.

- **Endpoint:** `https://search.redai.pl/mcp` (Streamable HTTP, JSON-RPC 2.0, stateless)
- **Registry:** `pl.redai/search` in the [official MCP Registry](https://registry.modelcontextprotocol.io/v0/servers?search=pl.redai)
- **Website:** https://redai.pl/search
- **Read-only:** every tool only reads the index, nothing is changed or sent.
- **Paid:** an API key is required to call tools (see below).

## Tools

| Tool | What it does |
|---|---|
| `search` | Any query in Polish. Returns companies and content pages in one ranking (`kind`: `business` or `site`), each with `pages` (subpage URL, summary, topics). Companies come with name, category, website, phone, e-mail, address and NIP (tax ID), checked against the Polish Ministry of Finance White List when `registry` is present. |
| `get_firm` | Full record of one company or site by `id` from a search result, with all subpages. |

Common arguments: `query`, `city`, `limit` (1-100), `offset` (paging, use `next_offset`),
`extended` (fetches the matching subpages live for the top 5 results and returns the matching
sentences; slower, counts as 5 queries).

## Access and pricing (paid)

- `initialize` and `tools/list` are open, so clients can see the tools. Every `tools/call` needs an
  API key: `Authorization: Bearer rk_...` (or `X-API-Key: rk_...`).
- Price: USD 0.10 per 1000 queries from a prepaid balance. The first 1000 queries a day per key are included.
  `extended=true` counts as 5 queries.
- Get a key: https://redai.pl/search#rozmowa . Usage and balance: `GET https://search.redai.pl/v1/usage` with the key.

## Setup

**Claude Code**

```bash
claude mcp add --transport http redai-search https://search.redai.pl/mcp \
  --header "Authorization: Bearer $REDAI_KEY"
```

**Claude Desktop / claude.ai:** Settings, Connectors, Add custom connector, URL `https://search.redai.pl/mcp?key=rk_...`
(the key in the URL, because custom connectors do not take headers).

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
  -H "Authorization: Bearer $REDAI_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"search","arguments":{"query":"mechanik krakow","limit":3}}}'
```

REST API with the same data: `GET https://search.redai.pl/v1/search?q=mechanik%20krakow` (same key header).

## Open WebUI, Serper and Tavily compatible endpoints

Same index, same key, same pricing (1 call = 1 query).

**Open WebUI** (Admin Panel, Settings, Web Search):

- Web Search Engine `external`, External Web Search URL `https://search.redai.pl/openwebui/search`,
  External Web Search API Key `rk_...`. Request `{"query","count"}`, response `[{"link","title","snippet"}]`.
- or the built-in `tavily` engine with the environment variable `TAVILY_API_BASE_URL=https://search.redai.pl/tavily`
  and your `rk_...` key as the Tavily API key.

**Code written for Serper or Tavily:** change the base URL and the key.

```bash
# was: https://google.serper.dev/search
curl -s https://search.redai.pl/serper/search -H "X-API-KEY: $REDAI_KEY" \
  -H 'Content-Type: application/json' -d '{"q":"hurtownia budowlana poznan","num":10}'

# was: https://api.tavily.com/search
curl -s https://search.redai.pl/tavily/search -H "Authorization: Bearer $REDAI_KEY" \
  -H 'Content-Type: application/json' -d '{"query":"mechanik krakow","max_results":5}'
```

Serper fields: `organic[].title, link, snippet, position, sitelinks`. Tavily fields: `results[].title, url, content, score`
(`include_domains` and `exclude_domains` work; `/tavily/extract` is not available, `answer` is always `null`).

## Po polsku

**Polska wyszukiwarka internetowa dla AI.** Podepnijcie ją pod Claude (claude.ai, Claude Desktop,
Claude Code), Cursor albo VS Code jako zdalny serwer MCP, a asystent dostanie wyszukiwanie po
polskich stronach `.pl` i polskich firmach. Pyta jak w Google, bez wyboru „tylko firmy” czy
„tylko strony”, i dostaje firmy z kontaktem, ofertą i NIP-em sprawdzonym w Białej liście MF
oraz podstrony, na których jest odpowiedź.

Adres: `https://search.redai.pl/mcp`. Wywołania narzędzi wymagają klucza API (`Authorization: Bearer rk_...`):
0,10 USD za 1000 zapytań z przedpłaconego salda, pierwsze 1000 zapytań dziennie na klucz w cenie.
W claude.ai: Ustawienia, Konektory, Dodaj własny konektor, adres `https://search.redai.pl/mcp?key=rk_...`.
Open WebUI: silnik `external` z adresem `https://search.redai.pl/openwebui/search` albo silnik `tavily`
z `TAVILY_API_BASE_URL=https://search.redai.pl/tavily`. Kod pisany pod Serper i Tavily działa po zmianie adresu i klucza.
Klucz odbierzecie na https://redai.pl/search#rozmowa
Więcej: https://redai.pl/search

## About

Built and run by [redAi](https://redai.pl) (Poland). This repository holds the public description
and `server.json` of the hosted server; the service itself runs on redAi infrastructure.
Issues and questions: open an issue here.

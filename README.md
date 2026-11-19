<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.svg">
    <img src="assets/logo.svg" alt="regon-mcp logo" width="96" height="96">
  </picture>
</p>
<h1 align="center">regon-mcp</h1>
<p align="center">
  <strong>An MCP server that gives AI assistants typed access to the Polish REGON business register (GUS BIR1).</strong><br>
  Look up any Polish company by NIP, REGON or KRS through six simple tools, with no SOAP code on your side.
</p>

<!-- mcp-name: io.github.SmartMobileHouse/regon-mcp -->

<div align="center">

[![PyPI](https://img.shields.io/pypi/v/regon-mcp)](https://pypi.org/project/regon-mcp/)
[![Python](https://img.shields.io/pypi/pyversions/regon-mcp)](https://pypi.org/project/regon-mcp/)
[![CI](https://github.com/SmartMobileHouse/regon-mcp/actions/workflows/publish.yml/badge.svg)](https://github.com/SmartMobileHouse/regon-mcp/actions/workflows/publish.yml)
[![License](https://img.shields.io/github/license/SmartMobileHouse/regon-mcp)](LICENSE)
[![Smart Mobile House](https://img.shields.io/badge/built%20by-Smart%20Mobile%20House-1F3A38)](https://smartmobilehouse.com)

</div>

## What it does

A [Model Context Protocol](https://modelcontextprotocol.io) server that gives AI
assistants clean, typed access to the **Polish REGON business register** (GUS
BIR1). Look up any Polish company by **NIP**, **REGON**, or **KRS** and get back
structured data — name, address, legal form, activity codes — without touching
the underlying SOAP API.

The official [GUS BIR1 API](https://api.stat.gov.pl/Home/RegonApi) is a WCF SOAP
service with WS-Addressing, MTOM multipart responses, an HTTP-header session
token, and XML-nested-inside-XML result payloads. `regon-mcp` hides all of that
behind a handful of simple tools.

| Tool | Description |
| --- | --- |
| `search_by_nip(nip)` | Look up an entity by 10-digit NIP (tax id). |
| `search_by_regon(regon)` | Look up an entity by 9- or 14-digit REGON. |
| `search_by_krs(krs)` | Look up an entity by 10-digit KRS (court register). |
| `search_bulk(identifiers, id_type)` | Look up up to 20 entities of one type at once. |
| `get_full_report(regon, report_type)` | Fetch a detailed report for one entity. |
| `list_report_types()` | List valid report names, with guidance on which to use. |

Every response includes a `source` block that names the register (REGON / GUS),
the environment, and a UTC `retrieved_at` timestamp — so downstream use can cite
the data correctly, as GUS requires.

Design notes: [the regon-mcp write-up](https://smartmobilehouse.com/blog/open-source-mcp-server-regon-registry/).

## Quick start

No install needed — run it straight from the repo with
[uv](https://docs.astral.sh/uv/):

```bash
uvx --from git+https://github.com/SmartMobileHouse/regon-mcp regon-mcp
```

From [PyPI](https://pypi.org/project/regon-mcp/): `uvx regon-mcp`.

By default it uses the **public test key** against the anonymized GUS test
database, so it runs with zero setup. For live data, request a free `USER_KEY`
from `regon_bir@stat.gov.pl` and set the environment variables below.

### Use it in Claude Desktop / Claude Code

Add to your MCP config (e.g. `claude_desktop_config.json` or a project
`.mcp.json`):

```json
{
  "mcpServers": {
    "regon": {
      "command": "uvx",
      "args": ["--from", "git+https://github.com/SmartMobileHouse/regon-mcp", "regon-mcp"],
      "env": {
        "REGON_API_KEY": "your-user-key",
        "REGON_ENV": "prod"
      }
    }
  }
}
```

During development, point it at a local checkout instead:

```json
{
  "mcpServers": {
    "regon": {
      "command": "uvx",
      "args": ["--from", "/absolute/path/to/regon-mcp", "regon-mcp"],
      "env": { "REGON_API_KEY": "abcde12345abcde12345" }
    }
  }
}
```

## Status

- [x] Look up entities by NIP, REGON and KRS
- [x] Bulk lookup of up to 20 entities of one type
- [x] Detailed reports, with guidance on which report type to use
- [x] A `source` block with the register, environment and UTC retrieval time in every response
- [x] Runs with zero setup on the public test key; live data with a free GUS `USER_KEY`
- [x] Offline test suite, including an in-memory MCP tool-discovery smoke test
- [x] Published on PyPI and the official MCP Registry

## Configuration

| Variable | Default | Description |
| --- | --- | --- |
| `REGON_API_KEY` | public test key | Your GUS BIR `USER_KEY`. |
| `REGON_ENV` | `test` | `test` (anonymized data) or `prod` (live data). |
| `REGON_TIMEOUT` | `30` | HTTP timeout in seconds. |

Use `REGON_ENV=prod` only with a real `USER_KEY`; the test key works only
against the test environment.

## Development

```bash
git clone https://github.com/SmartMobileHouse/regon-mcp
cd regon-mcp
uv sync                 # create the venv and install deps
uv run pytest           # offline tests: validation, parsing, mocked client,
                        # and an in-memory MCP tool-discovery smoke test.
                        # (network tests are deselected by default)
uv run regon-mcp        # run the server over stdio

# Live tests against the GUS endpoint (deselected unless opted in):
REGON_RUN_NETWORK=1 uv run pytest -m network            # session lifecycle (test env)
REGON_PROD_KEY=<your-key> uv run pytest -m network      # positive-control on live data
```

The client is a small hand-rolled SOAP layer over `httpx` (see
`src/regon_mcp/client.py`) — no heavyweight SOAP stack, no runtime WSDL fetch.

## Notes & limitations

- The **GUS test database is anonymized** and returns little or no entity data.
  Meaningful results require a production `USER_KEY` with `REGON_ENV=prod`.
- Respect the GUS terms of use and rate limits. This project is an independent
  open-source client and is not affiliated with or endorsed by GUS.
- Data belongs to GUS. When you present it, cite **REGON / GUS** with the
  retrieval date (surfaced in every response's `source` block).

## Contributing

Issues and pull requests are welcome.

## Why regon-mcp?

### The official API is hard to call

The GUS BIR1 API is a WCF SOAP service with WS-Addressing, MTOM multipart responses, an HTTP-header session token and XML nested inside XML result payloads.

### AI assistants need simple, typed tools

- A generic SOAP client leaves the session handling and payload parsing to every integration.
- Assistants need validated identifiers and structured fields they can cite.

### This server does the plumbing once

- A small hand-rolled SOAP layer over `httpx`, with no heavyweight SOAP stack and no runtime WSDL fetch.
- Six tools that return structured data: name, address, legal form and activity codes.
- Every response names its source, as GUS requires.

## License

[MIT](LICENSE) © Smart Mobile House. Built and maintained by [Smart Mobile House](https://smartmobilehouse.com).

# leadiq

LeadIQ — B2B sales intelligence and contact data platform with a public GraphQL Data API and Model Context Protocol server.

## APIs

- **LeadIQ Data API** — GraphQL endpoint at `https://api.leadiq.com/graphql` covering people search, company search, advanced grouped/flat search, prospect-list management, usage reporting, and data feedback. HTTP Basic auth with API key.
- **LeadIQ MCP Server** — Model Context Protocol surface exposing the same intelligence to Claude and other AI agents.

## Artifacts

- [`apis.yml`](apis.yml) — APIs.json catalog entry
- [`openapi/leadiq-data-api-openapi.yml`](openapi/leadiq-data-api-openapi.yml) — OpenAPI 3.0 wrapper for the GraphQL endpoint

## Links

- Developer portal: <https://developer.leadiq.com/>
- API overview: <https://leadiqhelp.zendesk.com/hc/en-us/articles/35509840045083-LeadIQ-Public-API-Overview>
- Upstream spec repo: <https://github.com/leadiq/dataiq-api-specs>
- API samples: <https://github.com/leadiq/api-samples>
- Pricing: <https://leadiq.com/pricing>

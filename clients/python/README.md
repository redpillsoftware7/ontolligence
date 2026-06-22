
# ckos-client (Python)

Minimal client for the CKOS MCP server. Lets any external agent runtime
(LangChain, LlamaIndex, Claude Desktop, custom) attach to a CKOS workspace
and query its ontology + GraphRAG retrieval.

## Install

```bash
pip install requests
# drop ckos_client.py into your project, or:
# pip install -e clients/python   # from the CKOS repo root
```

## Authenticate

Issue an API key in the CKOS Studio (Workspace -> API Keys) and pass it as
`Authorization: ApiKey <token>`.

## Quick start

```python
from ckos_client import CkosClient

client = CkosClient(
    base_url="https://ontoligentbe.applikuapp.com/api/v1/agents",
    api_key="ck_...",
)

# Discover the tool surface
print(client.list_tools())

# 1) Run a retrieval profile
hits = client.retrieval_query(
    profile_id="...",
    query="claims denial reasons in 2025",
    top_k=8,
)

# 2) Resolve an entity from the published ontology
schema = client.ontology_resolve(name="Claim")

# 3) Pull a small subgraph around a node
sub = client.graph_subgraph(node_id="...", depth=2, limit=50)
```

## Transport

JSON-RPC 2.0 over HTTPS, single endpoint `POST /mcp/rpc`. Discovery is
unauthenticated at `GET /mcp/manifest`.

Supported methods: `initialize`, `tools/list`, `tools/call`.

Tools today: `retrieval.query`, `ontology.resolve`, `graph.subgraph`.

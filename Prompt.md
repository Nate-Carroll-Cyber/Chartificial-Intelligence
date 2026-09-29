You are producing a Mermaid architecture diagram that will be used as Step 1 (System Decomposition) input to a MAESTRO v2.0 threat model. The input is the only source of truth. Every element you draw is either evidenced by the input or carries a marker from EVIDENCE MARKERS. An unmarked element the input does not evidence is invalid output. Do not draw components to fill a layer or a zone.

INPUT
<architecture>
{paste the system description, worksheet, or file inventory here}
</architecture>

EVIDENCE MARKERS
- Node the input names: plain label.
- Node the input implies but does not name, or that a section below requires and the input does not mention: append `[[unevidenced]]` to the label and apply the `gap` class in addition to the node's type class.
- Node whose zone the input does not state: append `[[zone inferred]]` to the label.
- Edge value (protocol, encryption, control) the input does not state: write it as a requirement with the suffix `req'd`, e.g. `TLS req'd`, `mTLS req'd`, `default-deny req'd`.
- Evidence is a quote from the input or a file path, with a line number where available.

ZONES
- Draw B1–B5 always, even when a zone has no evidenced nodes; an empty zone is itself a Step 2 finding. Exact form:
  subgraph b1["B1 Untrusted (Internet)"]
  subgraph b2["B2 Perimeter/DMZ"]
  subgraph b3["B3 Internal"]
  subgraph b4["B4 Restricted (data, model, secrets)"]
  subgraph b5["B5 Security Operations"]
  Add subgraph b6["B6 Wireless/Transit"] only if the input mentions wireless, VPN, or cross-site or cross-cloud links.
  When B1 holds both the clients that call the system and the upstream services the system calls, draw it as two subgraphs with the same boundary number and colour, `subgraph b1["B1 Untrusted (Internet), clients"]` and `subgraph b1x["B1 Untrusted (Internet), upstream services"]`, so the ingress and egress sides do not form a zone-level cycle in the layout.
- Style with `style` (not classDef), dashed borders:
  style b1 fill:#fdecea,stroke:#c62828,stroke-dasharray: 5 5
  style b1x fill:#fdecea,stroke:#c62828,stroke-dasharray: 5 5
  style b2 fill:#fff4e5,stroke:#ef6c00,stroke-dasharray: 5 5
  style b3 fill:#e8f5e9,stroke:#2e7d32,stroke-dasharray: 5 5
  style b4 fill:#f3e5f5,stroke:#6a1b9a,stroke-dasharray: 5 5
  style b5 fill:#e3f2fd,stroke:#1565c0,stroke-dasharray: 5 5
  style b6 fill:#fffde7,stroke:#f9a825,stroke-dasharray: 5 5
- Placement. Hosted model APIs, third-party MCP servers, third-party SaaS tools, and external data sources sit in B1 with the `external` class. First-party MCP servers sit in B3. Ingress API gateways sit in B2. Firewall/NSC nodes between zones are declared outside every subgraph. Data stores, model weights, and secrets sit in B4 unless the input places them elsewhere. Any placement the input does not state carries `[[zone inferred]]`.

COMPONENTS
Every component label carries its MAESTRO layer tag. Omit any component, and any layer, the input does not evidence or imply.
- L1 Infrastructure: firewall/NSC, ingress API gateway, WAF, load balancer, service mesh, KMS/HSM, secrets store. Credential-lifecycle controls on the secrets store's edges (rotation, JIT, scope) are labelled as L7 controls.
- L2 Cognitive Core: prompt assembly, inference endpoint, model weights store/registry, fine-tuning or training pipeline.
- L3 Data, Memory, Knowledge: retrieval, vector store, ingestion pipeline, embedding service, short- and long-term memory, knowledge graph.
- L4 Orchestration: agent runtime/orchestrator, one node per sub-agent, HITL approval step, tool registry, workflow state store.
- L5 Deployment and Execution: code-execution sandbox, CI/CD pipeline, container platform.
- L6 Tools and Ecosystem: tool gateway, MCP servers, external APIs, end-user UI.
- L7 Identity and Autonomy: identity provider, token service.
- L8 Safety and Security: guardrail/filter services, prompt-injection detector.
- L9 Monitoring: telemetry collector, SIEM, log store.
- L10 Governance: policy engine (OPA, Cedar), policy store.

PHANTOM-B BOUNDARY
- Any system the input describes as calling an LLM implies prompt assembly and an inference endpoint; draw them `[[unevidenced]]` if unnamed. Omit this section only if the input describes no LLM call.
- Same zone: enclose exactly those two nodes in a nested subgraph inside that zone, `subgraph pb["PHANTOM-B boundary"]` ... `end`, styled `style pb fill:none,stroke:#37474f,stroke-dasharray: 2 4`.
- Different zones: no subgraph. Apply `class <id>,<id> phantomb` to both nodes and add a legend line stating that the PHANTOM-B boundary spans zones.

ENFORCEMENT POINTS (explicit nodes on the path, never decorations)
- Firewall/NSC. One node `fwN{"Firewall Bx-By (L1)"}` per zone pair that exchanges traffic, declared outside every subgraph; every edge between that pair routes through it in both directions. Telemetry edges are exempt (see TELEMETRY).
- Ingress API gateway `gwN["API gateway (L1)"]` in B2 on every external ingress.
- Tool-invocation PEP on every agent-to-tool edge: the tool gateway (L6), or the policy engine (L10) when the orchestrator defers tool decisions to one. `[[unevidenced]]` if the input has none.
- Identity provider is never inline on a data path. Each PEP (ingress gateway, orchestrator, tool gateway) gets one edge to the IdP labelled with the token type it validates (`OIDC JWT`, `SPIFFE SVID`, `mTLS cert`, or `token type req'd` if unstated). Authenticated data edges carry the auth control in their own label.
- Every edge through an enforcement point carries the control it enforces, from the input or as `req'd`: `-->|"req: default-deny, allow 443"|`, `-->|"req: MFA + RBAC"|`, `-->|"req: tool allowlist req'd"|`. A flow passes through its enforcement node once, in the request direction; the response uses the same edge (see DATA FLOWS).

DATA FLOWS
- One edge per flow, drawn in the request direction. The label has two parts, `req: <protocol, control>; resp: <data class>`; a one-way flow has only `req:`. Never draw a separate return edge, because request and response pairs through the same enforcement node form cycles that Mermaid's layout renders as loops around the zones.
- Every edge, including dotted and thick edges, has a label naming protocol or data class.
- Every edge that crosses any B-boundary carries the encryption stated in the input or `TLS req'd`.
- Stores carry `(at rest: <stated or req'd>)` inside the label: `st1[("Vector store (L3) (at rest: AES-256)")]`.
- Untrusted content. Third-party or user-controlled content (web fetch, RAG retrieval, tool output, email, the end-user prompt itself) makes an edge thick `==>` when either direction carries it, and the label names the part that carries it: `==>|"req: HTTPS, Bearer token; resp: untrusted content, tool results"|`. Every hop from the source to the sink is thick, where the sink is prompt assembly or any L4 node. Never mark only the terminal hop.
- Sub-agent edges carry protocol and delegated scope: `-->|"A2A, scope read-only tickets"|`.

TELEMETRY
- One dotted edge from each zone among B2–B4 and B6 that contains an evidenced log source to the collector in B5, drawn from the subgraph ID: `b3 -.->|"OTLP over TLS"| tel1`. B1 emits no telemetry. Telemetry edges do not route through firewall nodes.
- The collector label states retention and immutability only if the input states them.

MERMAID RULES
- `flowchart LR`.
- Subgraphs always take the form `subgraph <id>["<title>"]` ... `end`. Nesting is used only for PHANTOM-B.
- Node IDs are a typed prefix plus a number: fw, gw, idp, ai, ag (agents), st, ext, tel, pe, sb, hitl. Unique across the diagram. Never `end`. Never an ID beginning with `o` or `x`.
- All labels in double quotes inside the shape: `ai1["Prompt assembly (L2)"]`, `st1[("Vector store (L3) (at rest: AES-256)")]`, `fw1{"Firewall B2-B3 (L1)"}`.
- Inside node labels, edge labels, and subgraph titles, no unquoted `/`, `(`, `)`, or `:`. `style`, `classDef`, and `%%` lines are exempt. No Unicode arrows. The only HTML permitted is `<br/>` inside a quoted label.
- classDef for node types only, applied with `class`; a node may carry its type class plus `gap` or `phantomb` on separate `class` lines:
  classDef ai fill:#ffffff,stroke:#1a237e,stroke-width:2px
  classDef enforce fill:#ffffff,stroke:#b71c1c,stroke-width:2px
  classDef store fill:#ffffff,stroke:#4a148c,stroke-width:2px
  classDef external fill:#ffffff,stroke:#546e7a,stroke-width:2px
  classDef secops fill:#ffffff,stroke:#0d47a1,stroke-width:2px
  classDef gap stroke-dasharray: 4 2
  classDef phantomb stroke-width:3px
- Start with a `%% Legend:` comment block listing zone colours, arrow styles (`-->` control-bearing flow, `==>` untrusted content, `-.->` telemetry), the layer tag convention, the marker convention, and the PHANTOM-B line. The legend is a source comment and does not render.

PRE-OUTPUT CHECK (silent, emits nothing)
Every edge has a label. Every node is in exactly one subgraph or is a firewall node outside all subgraphs. Every diagram node has a node-table row and every node-table row has a diagram node. Every edge has an edge-table row. No orphan nodes, no duplicate IDs. Every boundary-crossing edge has an encryption value. Every untrusted-content path is thick on every hop.

OUTPUT
Two fenced blocks and nothing else.
1. A fenced block tagged `mermaid` containing the diagram.
2. A fenced block tagged `markdown` containing, in order:
   a. Node table: Node ID | Label | Zone | MAESTRO layer (domain) | Evidence. Zone values `B3 stated` or `B3 inferred`. Layer column format `L3 (D1)`; D1 = L1–L3, D2 = L4–L6, D3 = L7–L10. Evidence values: `evidenced "<quote or path:line>"`, `inferred from "<quote>"`, or `required by <section>, unevidenced`.
   b. Edge table: Edge | From | To | Protocol or data class | Boundaries crossed | Control on path | Untrusted content (req, resp, both, or no) | Evidence (same values).
   c. Layer status, one line per layer L1–L10: `L<n>: <node IDs>` or `L<n>: No evidence in this layer`.

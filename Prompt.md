You are producing a Mermaid architecture diagram that will be used as Step 1 (System Decomposition) input to a MAESTRO v2.0 threat model. The input is the only source of truth. Every element you draw is either evidenced by the input or carries a marker from EVIDENCE MARKERS. An unmarked element the input does not evidence is invalid output. Do not draw components to fill a layer or a zone.

INPUT
<architecture>
{paste the system description, worksheet, or file inventory here}
</architecture>

EVIDENCE MARKERS
- Node the input names: plain label.
- Node the input implies but does not name, or that a section below requires and the input does not mention: append `[[unevidenced]]` to the label and apply the `gap` class in addition to the node's type class.
- Node whose zone the input does not state: append `[[zone inferred]]` to the label.
- Flow the input implies but does not state, or that a section below requires and the input does not mention, including every flow that touches an `[[unevidenced]]` node: append `[[unevidenced]]` to the edge label.
- Edge value (protocol, encryption, control) the input does not state: write it as a requirement with the suffix `req'd`, e.g. `TLS req'd`, `mTLS req'd`, `default-deny req'd`. A value without the suffix is a value the input states. `req'd` is the only use of the letters "req" in any label; the request part of a flow label uses the prefix `rq:` (see DATA FLOWS).
- Data class the input does not state: write `class unstated`. Never guess a data class.
- Trust boundary (see TRUST BOUNDARIES INSIDE ZONES) the input implies but does not state: append `[[unevidenced]]` to the subgraph title.
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
- Placement. Hosted model APIs, third-party MCP servers, third-party SaaS tools, and external data sources sit in B1 with the `external` class. End users sit in B1 on the clients side with the `actor` class. First-party MCP servers sit in B3. Ingress API gateways sit in B2. Firewall/NSC nodes between zones are declared outside every subgraph. Data stores, model weights, and secrets sit in B4 unless the input places them elsewhere. Administrators, operators, and HITL approvers sit in the zone the input states for them. Any placement the input does not state carries `[[zone inferred]]`.

TRUST BOUNDARIES INSIDE ZONES
A zone is a network boundary. Privilege also changes inside a zone, and each such change is drawn.
- Code-execution sandbox. When the input describes a sandbox or agent-run code, enclose the sandbox node and anything the input says runs inside it in `subgraph tb<n>["Trust boundary, code execution sandbox"]` nested in its zone.
- Tenant or workspace isolation. When the input describes per-tenant or per-workspace components, enclose each tenant's components in `subgraph tb<n>["Trust boundary, tenant <name>"]`. Draw one representative tenant when the input describes tenants generically.
- Any other privilege boundary the input names (separate account, namespace, enclave, process isolation): same form, title `Trust boundary, <name from input>`.
- Style every one: `style tb<n> fill:none,stroke:#000000,stroke-dasharray: 8 3`.
- Identity change is an edge property, not a subgraph. Any flow where the acting identity changes carries it in the label: `identity user to agent service`, `identity agent to tool credential`, or `identity change unstated` when the input describes the hop and not the identity.
- Every flow that crosses a `tb` boundary names the control that enforces the boundary, from the input or as `req'd`, and lists the `tb` ID in the edge table's boundaries column.
- A `tb` subgraph and the PHANTOM-B subgraph never contain another subgraph.

COMPONENTS
Every component label carries its MAESTRO layer tag. Human actors carry no layer tag. Omit any component, and any layer, the input does not evidence or imply.
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
- Human actors, one node per role the input names or implies: end user, administrator, operator or developer, HITL approver. Form `act1(["End user"])`. The end user connects to the end-user UI or the ingress gateway, never directly to an internal component. The HITL approver connects to the HITL approval step. Draw an actor `[[unevidenced]]` only when the input evidences the component that actor must use (an end-user UI, an admin console, a HITL step) and does not name the actor.

PHANTOM-B BOUNDARY
- Any system the input describes as calling an LLM implies prompt assembly and an inference endpoint; draw them `[[unevidenced]]` if unnamed. Omit this section only if the input describes no LLM call.
- Same zone: enclose exactly those two nodes in a nested subgraph inside that zone, `subgraph pb["PHANTOM-B boundary"]` ... `end`, styled `style pb fill:none,stroke:#37474f,stroke-dasharray: 2 4`.
- Different zones: no subgraph. Apply `class <id>,<id> phantomb` to both nodes and add a legend line stating that the PHANTOM-B boundary spans zones.

ENFORCEMENT POINTS (explicit nodes on the path, never decorations)
- Firewall/NSC. One node `fwN{"Firewall Bx-By (L1)"}` per zone pair that exchanges traffic, declared outside every subgraph; every flow between that pair routes through it, each in its own request direction. Telemetry edges are exempt (see TELEMETRY).
- Ingress API gateway `gwN["API gateway (L1)"]` in B2 on every external ingress.
- Tool-invocation PEP on every agent-to-tool edge: the tool gateway (L6), or the policy engine (L10) when the orchestrator defers tool decisions to one. `[[unevidenced]]` if the input has none.
- Identity provider is never inline on a data path. Each PEP (ingress gateway, orchestrator, tool gateway) gets one edge to the IdP labelled with the token type it validates (`OIDC JWT`, `SPIFFE SVID`, `mTLS cert`, or `token type req'd` if unstated). Authenticated data edges carry the auth control in their own label.
- Every edge through an enforcement point carries the control it enforces, from the input or as `req'd`: `-->|"F3 rq: HTTPS, default-deny, allow 443, class unstated"|`, `-->|"F5 rq: HTTPS, MFA + RBAC req'd, user prompt"|`, `-->|"F9 rq: MCP, tool allowlist req'd, tool arguments"|`. A flow passes through its enforcement node once, in the request direction; the response uses the same edges (see DATA FLOWS).
- A flow that passes through one or more enforcement nodes is drawn as consecutive segments, and every segment carries the same flow ID and the same label. The flow ID is what ties a source to its destination when several flows share an enforcement node. Without it the diagram cannot show which source reaches which destination.

DATA FLOWS
- One flow per logical exchange, drawn in the request direction. Never draw a separate return edge, because request and response pairs through the same enforcement node form cycles that Mermaid's layout renders as loops around the zones.
- Flow IDs. Every flow has an ID `F<n>`, numbered from F1 in the order a request traverses the system, ingress first. The ID is the first token of the edge label. Control-plane flows and telemetry flows are numbered after the request-path flows.
- Label form: `F<n> rq: <protocol>, <control>, <request data class>; resp: <response data class>`. A one-way flow has only the `rq:` part. Both parts name a data class. The request data class is what the caller sends (user prompt, assembled prompt with retrieved context, tool arguments, credentials, PII, embeddings); the response data class is what comes back. Use the input's own classification terms where it has them and `class unstated` where it does not.
- Every edge, including dotted and thick edges, has a label in this form.
- Every edge that crosses any B-boundary carries the encryption stated in the input or `TLS req'd`.
- Stores carry `(at rest: <stated or req'd>)` inside the label: `st1[("Vector store (L3) (at rest: AES-256)")]`.
- Store operations. Every flow whose destination is a store states the operation in the `rq:` part: `op read`, `op write`, `op read-write`, or `op unstated`. Example: `-->|"F12 rq: gRPC, mTLS req'd, op write, document chunks and embeddings"|`. Where the input evidences reads and writes by the same caller under different controls, draw two flows.
- Untrusted content. Third-party or user-controlled content (web fetch, RAG retrieval, tool output, email, the end-user prompt itself) makes an edge thick `==>` when either direction carries it, and the label names the part that carries it: `==>|"F14 rq: HTTPS, Bearer token, search query; resp: untrusted content, tool results"|`. Every hop from the source to the sink is thick, where the sink is prompt assembly or any L4 node. Never mark only the terminal hop. A write of untrusted content into an L3 store is thick, and so is every later read of that store that reaches prompt assembly or an L4 node.
- Sub-agent edges carry protocol and delegated scope: `-->|"F16 rq: A2A, scope read-only tickets, task description; resp: ticket summaries"|`.

CONTROL PLANE
Flows that change the system rather than use it. Draw each one the input evidences. When the input evidences both endpoints and not the flow between them, draw the flow `[[unevidenced]]`. Each label starts with the flow ID followed by `ctl`, e.g. `-->|"F21 ctl rq: HTTPS, signed artifact req'd, container image"|`.
- Deployment: CI/CD pipeline to container platform, sandbox image, MCP server, or agent runtime.
- Model update: training or fine-tuning pipeline, or an external model source, to the model weights store or registry, and the registry to the inference endpoint.
- Secret distribution: secrets store to each component the input says holds a credential. The rotation, JIT, and scope values are the L7 controls on these edges.
- Policy push: policy store to policy engine, and tool registry updates.
- Administrative access: administrator or operator actor to each console, registry, or store the input says they manage.
- Knowledge ingestion: external or internal source through the ingestion pipeline to an L3 store. This is a write path and follows the store operation and untrusted content rules.

TELEMETRY
- One dotted edge from each zone among B2–B4 and B6 that contains an evidenced log source to the collector in B5, drawn from the subgraph ID: `b3 -.->|"F30 rq: OTLP over TLS, logs and traces"| tel1`. B1 emits no telemetry. Telemetry edges do not route through firewall nodes.
- When the input says prompts, completions, or tool payloads are logged, the telemetry label names them as the data class.
- The collector label states retention and immutability only if the input states them.

DECOMPOSITION
- Count the nodes, excluding firewall nodes. At 25 or fewer, emit one diagram.
- Above 25, and when the input names subsystems or use cases, emit a context diagram first and then one detail diagram per named subsystem or use case. The context diagram shows the zones, one node per subsystem, the actors, the external entities, and only the flows that cross a B-boundary. Each detail diagram follows every rule in this prompt.
- Node IDs and flow IDs are assigned once and are identical in every diagram where the node or flow appears. A node at the edge of a detail diagram that belongs to another diagram keeps its ID and label.
- Above 25, and when the input names no subsystems or use cases, emit one diagram and report the count on the Density line of the output. Do not invent a split.

MERMAID RULES
- `flowchart LR`.
- Subgraphs always take the form `subgraph <id>["<title>"]` ... `end`. Nesting is used only for PHANTOM-B and for `tb` trust boundaries, one level deep.
- Node IDs are a typed prefix plus a number: fw, gw, idp, ai, ag (agents), st, ext, tel, pe, sb, hitl, act (human actors), pr (any other process). Trust boundary subgraph IDs are tb plus a number. Unique across the diagram. Never `end`. Never an ID beginning with `o` or `x`.
- All labels in double quotes inside the shape: `ai1["Prompt assembly (L2)"]`, `st1[("Vector store (L3) (at rest: AES-256)")]`, `fw1{"Firewall B2-B3 (L1)"}`, `act1(["End user"])`.
- Inside node labels, edge labels, and subgraph titles, no unquoted `/`, `(`, `)`, or `:`. `style`, `classDef`, and `%%` lines are exempt. No Unicode arrows. The only HTML permitted is `<br/>` inside a quoted label.
- classDef for node types only, applied with `class`; a node may carry its type class plus `gap` or `phantomb` on separate `class` lines:
  classDef ai fill:#ffffff,stroke:#1a237e,stroke-width:2px
  classDef enforce fill:#ffffff,stroke:#b71c1c,stroke-width:2px
  classDef store fill:#ffffff,stroke:#4a148c,stroke-width:2px
  classDef external fill:#ffffff,stroke:#546e7a,stroke-width:2px
  classDef secops fill:#ffffff,stroke:#0d47a1,stroke-width:2px
  classDef actor fill:#ffffff,stroke:#212121,stroke-width:2px
  classDef process fill:#ffffff,stroke:#00695c,stroke-width:2px
  classDef gap stroke-dasharray: 4 2
  classDef phantomb stroke-width:3px
- Every node gets exactly one type class. Take the first that matches, in this order:
  1. `actor`: any human actor.
  2. `external`: any non-human node in B1.
  3. `store`: any data store, model weights store or registry, secrets store, policy store, log store, workflow state store.
  4. `enforce`: firewall, API gateway, WAF, tool gateway, policy engine, HITL approval step, guardrail or filter service, prompt-injection detector, identity provider, token service.
  5. `secops`: any remaining node in B5.
  6. `ai`: prompt assembly, inference endpoint, training or fine-tuning pipeline, orchestrator, every agent and sub-agent.
  7. `process`: everything else, including first-party MCP servers, retrieval, ingestion, embedding service, end-user UI, CI/CD pipeline, container platform, sandbox, load balancer, service mesh.
- Start with a `%% Legend:` comment block listing zone colours, trust boundary and PHANTOM-B border styles, arrow styles (`-->` control-bearing flow, `==>` untrusted content, `-.->` telemetry), node class colours, the layer tag convention, the flow label form (`F<n>`, `rq:`, `resp:`, `ctl`, `op`), and the marker convention (`[[unevidenced]]`, `[[zone inferred]]`, `req'd`, `class unstated`). The comment does not render, so the same legend is repeated as the first item of the markdown block.

PRE-OUTPUT CHECK (silent, emits nothing)
Every edge has a label that starts with a flow ID. Every flow ID in a diagram has exactly one edge-table row and every edge-table row has a flow ID in a diagram. All segments of one flow carry the same ID and label. Every node is in exactly one zone subgraph or is a firewall node outside all subgraphs. Every diagram node has a node-table row and every node-table row has a diagram node. Every node has exactly one type class. No orphan nodes, no duplicate IDs. Every boundary-crossing edge has an encryption value. Every `rq:` part and every `resp:` part names a data class or `class unstated`. Every flow into a store states an `op`. Every untrusted-content path is thick on every hop. No label uses "req" other than in `req'd`.
DFD validity. Every store has at least one flow that writes it and one that reads it. Every process has at least one inbound and one outbound flow. No flow connects two stores, or an actor or external entity to a store, without a process between them. Do not add nodes or flows to repair a violation. Draw what the input evidences and report each violation on the Validity exceptions lines of the output.

OUTPUT
One or more fenced blocks tagged `mermaid`, then one fenced block tagged `markdown`, and nothing else.
1. The diagram blocks. One block, or a context diagram followed by detail diagrams under DECOMPOSITION. Each block's first line after `flowchart LR` is a comment `%% Diagram: <name>`.
2. A fenced block tagged `markdown` containing, in order:
   a. Legend. The same content as the `%% Legend:` comment, as a short list.
   b. Node table: Node ID | Label | Type class | Zone | Trust boundary | MAESTRO layer (domain) | Evidence. Zone values `B3 stated` or `B3 inferred`. Trust boundary is a `tb` ID, `pb`, or `none`. Layer column format `L3 (D1)`; D1 = L1–L3, D2 = L4–L6, D3 = L7–L10; `n/a` for human actors. Evidence values: `evidenced "<quote or path:line>"`, `inferred from "<quote>"`, or `required by <section>, unevidenced`.
   c. Edge table, one row per flow, not per segment: Flow ID | From | To | Via (enforcement nodes, in order) | Plane (data, control, telemetry) | Protocol | Request data class | Response data class | Store op (read, write, read-write, unstated, or n/a) | Identity change | Boundaries crossed (B and tb) | Control on path | Untrusted content (req, resp, both, or no) | Diagrams | Evidence (same values).
   d. Layer status, one line per layer L1–L10: `L<n>: <node IDs>` or `L<n>: No evidence in this layer`.
   e. Validity exceptions, one line each: `<node or flow ID>: <rule violated>, as evidenced by "<quote>"` or `<node ID>: <rule violated>, no evidence of the missing flow`. Write `None` when there are none.
   f. Density: `<n> nodes excluding firewalls, <single diagram | split into <k> diagrams | not split, input names no subsystems>`.

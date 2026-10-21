# interactive-architectures-html

Tips for reuse:
- **Fill in only "Project-specific context".** The columns, boundaries, purposes list and sample flows there are what tailor the diagram; the rest applies to any project.
- **If you don't know the architecture yet,** delete the "Columns" and "Boundaries" lines. Add: "propose the columns, boundaries and sample flows from the scan and wait for my approval before writing the file."
- **For very large repositories,** add: "limit the diagram to at most ~30 nodes; group minor components into one node and list them in its details panel." That keeps the diagram readable.


````text
# Task: build a standalone interactive architecture diagram of this project

## Ground rules
- Do NOT modify, move or delete any existing file or folder. The ONLY change is one new file:
  `interactive_architecture.html` in the repository root.
- First SCAN the workspace (read-only) and build an inventory from the actual code and config.
  Never invent components. Look for:
  - Entry points / sources: clients, UIs, CLIs, external APIs, webhooks, devices, data generators,
    file drops, scheduled triggers, seed/reference data.
  - Ingestion / interfaces: API gateways, REST/gRPC/GraphQL endpoints, message queues and topics,
    event buses, streams, upload buckets, log shippers.
  - Processing: services, workers, functions, batch and stream jobs, pipelines/DAGs, ML training or
    inference, SQL transformations, rules engines.
  - Storage and intermediate state: databases, caches, data lakes and warehouses, search indexes,
    object storage, schema/contract registries.
  - Outputs / destinations: downstream services, dashboards, notifications, exports, third-party
    integrations, analytics or query engines.
  - Infrastructure and boundaries: Docker/Compose, Kubernetes manifests, Terraform/CloudFormation/
    Pulumi, cloud accounts/regions/VPCs, environments, CI/CD.
  - For each component, capture where it applies: name, the file(s) defining it, what it reads and
    writes, protocol, data format/schema, keys/IDs, scaling settings (replicas, partitions, shards),
    retention/TTL, auth, and the naming conventions or templates used.
- Report the inventory to me in a short table BEFORE writing the file, and say explicitly when a
  category doesn't exist in this project.
- Never put secrets, API keys, account/tenant/resource IDs, hostnames of private systems, or
  personal data in the file. Use placeholders such as `<env>`, `<id>`, `<region>`.

## Project-specific context (edit per project)
- Title: <e.g. "Order Processing Platform: Data Flow">
- Columns (3-6, left to right), for example:
  <SOURCES → INGESTION → PROCESSING → STORAGE / OUTPUT CHANNELS → DESTINATIONS>
  or <CLIENTS → EDGE / API → SERVICES → DATA STORES → CONSUMERS>
- Boundaries to colour-code: <e.g. user devices, on-prem / laptop, cloud account A, Kubernetes
  cluster, managed data platform, third-party SaaS>
- Purposes that the "why" lines must call out where they apply: <e.g. validation, normalization,
  schema enforcement, enrichment/lookups, deduplication, filtering, aggregation/windowing, routing,
  caching, rate limiting, auth, encryption, PII masking, retries/idempotency, cost reduction, auditing>
- Sample flows to include (5-8 end-to-end stories): <e.g. happy-path request, a rejected/filtered
  record, an error and retry, an enrichment lookup, a privacy-masking case, a batch or aggregate job>

## Technology
- One standalone HTML file: inline CSS and JS, no build step.
- vis-network from a CDN with an exact pinned version, e.g.
  https://cdnjs.cloudflare.com/ajax/libs/vis-network/9.1.9/standalone/umd/vis-network.min.js
- Optional Google Fonts (Inter for UI, JetBrains Mono for code), with system font fallbacks.
- physics: false, fixed node coordinates. Nodes are draggable. Disable the library's built-in wheel
  zoom (interaction.zoomView = false) and implement zoom yourself (see "Zoom & pan").

## Layout
- The columns from the context section, with headers drawn ON the canvas (title + small subtitle,
  small arrows between them) so they pan and zoom with the diagram.
- A column may have sub-lanes when a stage has internal steps (e.g. a service → its intermediate
  queue or table → a second service). Shared supporting components (registry, config store, auth,
  cache) go underneath the components that use them.
- Leave enough spacing that neighbouring boundaries never overlap or touch, and their labels never
  collide.

## Colour-coded boundaries
- Draw rounded, semi-transparent boundary rectangles BEHIND nodes in vis-network's `beforeDrawing`
  hook. Compute each from the CURRENT node bounding boxes (network.getBoundingBox) plus padding, so
  boundaries follow nodes when they are dragged.
- Each boundary has a label in its top-left corner, its own colour, and solid or dashed stroke.
  Outer boundaries (e.g. a cloud account) get a thicker solid stroke; nested ones (e.g. a cluster
  inside it) are dashed.
- Colour by component CATEGORY (adjust to the project), e.g.:
  source/client slate #94a3b8, network/edge blue #3b82f6, queue/stream cyan #22d3ee,
  reference/lookup data amber #f59e0b, processing violet #c084fc, governed/intermediate data teal
  #2dd4bf, output channel green #34d399, integration/connector blue #60a5fa, end-user destination
  yellow #facc15, storage/lake rose #fb7185, registry/config lime #a3e635.

## Nodes
- Box nodes with 2-3 line labels: bold name on line 1, then the key facts, e.g.
  "orders-api / REST · 3 replicas" or "orders_enriched / Avro · key order_id".
- Use vis-network multi-line HTML labels (font.multi = 'html', <b> and <code> only).
  PITFALLS:
  - vis-network does NOT decode HTML entities like &lt; in labels; use "‹id›" in labels and map it
    back to "<id>" in the side panel.
  - Whenever you update a node's style (e.g. dimming), pass the FULL font object, including
    multi:'html' and the bold/code sub-fonts. A partial `font: {color}` silently drops HTML labels and
    shows raw <b> tags.
- Background = node colour at low alpha, border = node colour, soft glow shadow. Processing nodes
  get a thicker border.
- A legend overlay (top-left) explains node colours and edge styles.

## Edges
- 3-6 edge styles chosen to fit the project, e.g. main data/request flow (solid grey), lookup or
  read-only dependency (dashed amber), async/event (dotted violet), storage/replication (dashed
  rose), config/schema (dotted lime). Secondary edges are lower opacity so the main flow reads first.
- Horizontal cubic-bezier curves, small arrowheads, short labels (protocol or verb: "HTTP POST",
  "publish", "consume", "SQL insert", "lookup", "nightly batch").
- Header toggle "edge labels" shows/hides labels (set the label to ' ' to hide).

## Details panel (right side, "Details" tab)
- Clicking a node shows:
  - kicker with the component type in its colour (e.g. "SERVICE", "MESSAGE QUEUE", "DATABASE")
  - title (resource name with any naming template) and subtitle
  - metadata grid: name/template, defined in (file path), technology and version, interface or
    protocol, data format/schema, key/ID, scaling, retention/TTL, auth, reads → writes, rough volume
    or latency if known
  - "What happens here, and why": one card per responsibility or transformation. A bold WHAT that
    names the concrete mechanism (function, endpoint, query, rule, library call), then a muted
    "why →" line giving the business or technical reason from the purposes list above.
  - buttons "▶ <flow name>" for every sample flow that passes through this node
- The initial empty state explains how to use the page and any naming template.

## Sample data flow ("Sample data flow" tab + header "▶ Show sample data" button + flow dropdown)
- 5-8 flows. Each has an outcome banner: green (completed / delivered), red (rejected, filtered or
  failed, and where it ended up), or blue (computed / aggregated).
- Each flow lists ordered steps {node, title, note, data}. Data shows the REAL payload shape at that
  stage, taken from the repo's code, schemas, fixtures, tests or example outputs: request → validated
  object → message/event → enriched record → stored row → response/output, plus any lookup results
  and decisions. Shorten long values with "…". No real personal data.
- Pretty-print JSON when it parses, otherwise show it as text. Use simple syntax colouring (keys,
  strings, numbers, booleans, null). Highlight key findings or masked values.
- Selecting a flow dims every node and edge not on its path (keep full font objects!), then
  animates focus to each step's node (network.focus with easing) with a short glow pulse.
- Step list: numbered cards. Clicking one jumps to it; the active card expands to show its data.
- Navigation bar with ◀ Prev, ▶ Play / ⏸ Pause (auto-advance every ~2.2 s), Next ▶, and a
  "3 / 13" counter. REQUIRED:
  - The bar is STICKY at the top of the scrolling pane (position: sticky, top equal to minus the pane
    padding, opaque background, z-index), so it stays visible on long flows.
  - Prev is disabled on the first step and Next on the last.
  - When stepping, scroll the active card into view BELOW the sticky bar. Compute the offset with
    getBoundingClientRect, not offsetTop.
- "Clear highlight" restores normal colours.

## Collapsible side panel
- A header button "⇥ Hide panel" / "⇤ Show panel" collapses the right pane so the full architecture
  uses the whole width (grid column → 0, aside hidden).
- A vertical "⇤ Details" tab on the canvas's right edge reopens it.
- Clicking a node, or starting a sample flow, reopens the panel automatically.
- After toggling, call network.setSize('100%','100%') and network.redraw().
- Remember open/closed in localStorage, wrapped in try/catch so the page works without storage.

## Zoom & pan (must NOT be over-sensitive on a trackpad)
- Custom wheel handler on the network container ({passive:false}, preventDefault):
  - ctrlKey + wheel = trackpad pinch: factor = exp(-clamp(deltaY, -40, 40) * 0.006). Gentle;
    expose this constant so it can be tuned.
  - Wheel with deltaX ≠ 0, or |deltaY| < 40 = two-finger swipe: pan by (deltaX, deltaY) / scale.
  - Otherwise (mouse-wheel notch): one 12% zoom step.
  - Normalize deltaMode (lines → ×16 px, pages → ×container height).
  - Zoom around the cursor: keep the canvas point under the pointer fixed (DOMtoCanvas, then moveTo
    with the new position and scale).
- Clamp zoom to 15%–250%.
- Zoom bar bottom-right: [−] [ 56% ] [+] [⤢ fit]. ± are 25% steps with a 200 ms animation, the
  percentage updates live, and ⤢ fits the whole architecture.
- "Fit view" and "Reset layout" (move nodes back to their original coordinates, then fit) in the
  header. The initial view fits everything with ~3% margin.

## Overview minimap (shows which part is zoomed vs the entire architecture)
- A small canvas (~250×140 CSS px, drawn at 2× for sharpness) in the bottom-right above the zoom bar,
  titled "Overview".
- It draws: boundary rectangles in their colours, faint straight edges, nodes as small rectangles in
  node colours (dimmed when off the active sample path), and the CURRENT VIEWPORT as a cyan
  rectangle computed from DOMtoCanvas of the container corners.
- It redraws on every afterDrawing through requestAnimationFrame throttling, and recomputes the world
  bounds after node drags.
- Click or drag in the minimap to centre the main view on that point (pointer events with
  setPointerCapture). Wheel over the minimap zooms the main view.
- A header button "Minimap" shows or hides it.

## Visual design
- Modern dark mode. CSS tokens on :root (bg #0b0f17, panel #131a26, line #243045, text #e6edf6,
  muted #93a1b6, accent #38bdf8, plus the category colours). Explicit body background.
- Header: title + one-line subtitle on the left. Controls on the right: flow dropdown,
  ▶ Show sample data (primary), Clear highlight, edge labels toggle, Fit view, Reset layout,
  Minimap, Hide panel.
- Rounded 8-10 px corners, subtle borders, monospace for code and payloads. Bottom-left hint:
  "Pinch or mouse wheel to zoom · two-finger swipe or drag background to pan · drag nodes ·
  click a node for details".
- Responsive: under ~980 px, stack the canvas (60vh) above the panel. The collapsed panel and a smaller
  minimap still work. No horizontal page scroll.

## Deep links
- `#trace=<flow-id>&step=<n>` opens a flow at a step. `#node=<node-id>` opens a node's details.
  List the flow ids in your final message.

## Verification before you finish
- Render the page in headless Chrome and LOOK at the screenshots, at minimum:
  1. Default overview (labels render as bold HTML, not raw tags; boundaries don't overlap).
  2. Details panel for a processing node (#node=…).
  3. The longest flow at its last step: the sticky nav stays visible with the "N / N" counter, and
     Next is disabled. Headless Chrome may not run smooth scrolling, so verify with a scratch copy
     that uses behavior:'auto'.
  4. Panel collapsed: the canvas spans the full width, the "Details" tab is visible, and the minimap
     viewport rectangle is visible.
- Simulate input in a scratch copy (dispatch WheelEvents) and report the zoom % after one pinch
  event, after two wheel notches, and the pan delta after a two-finger swipe. Confirm the change
  is gentle.
- Put scratch copies and screenshots OUTSIDE the repo, and leave every existing file unchanged.

## Final message
- What was scanned and found (the short inventory table), how to open the file, the controls, deep
  links, the one tunable constant for pinch sensitivity, and anything you could not verify.
````


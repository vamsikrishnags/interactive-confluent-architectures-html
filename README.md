# interactive-confluent-architectures-html

```text
# Task: build a standalone interactive architecture diagram of this streaming pipeline

## Ground rules
- Do NOT modify, move or delete any existing file or folder. The ONLY change is one new file:
  `interactive_architecture.html` in the repository root.
- First SCAN the workspace (read-only) and build an inventory from the actual code. Never invent components:
  - Streaming data sources: log generators, agents/shippers (Fluent Bit, Beats, Fluentd, Vector), producers, data generators, CDC/connectors, reference-data loaders, CSV/seed files.
  - Kafka / Confluent topic definitions: Terraform (confluent_kafka_topic), Flink CREATE TABLE DDL, producer/consumer configs, connector configs. For each topic capture: name, who creates it, key, schema/format (schemaless JSON, Avro, JSON Schema, Protobuf; Schema Registry subjects), partitions/buckets, retention, cleanup policy (compact/delete), changelog mode (append/upsert).
  - Apache Flink processing: SQL files (DDL + DML, statement sets), Java/Python jobs (DataStream API, Table API, PyFlink). For each job: reads, writes, joins, filters, windows, UDFs.
  - Sinks / destinations: Kafka Connect connectors (class, topics, converters), Tableflow / Iceberg / Delta, databases, SIEMs, object storage, query engines (DuckDB, Trino, Spark).
  - Naming templates used for resources (e.g. `<prefix>-<id>-*`, a DEMO_ID or env suffix).
- Report the inventory to me in a short table BEFORE writing the file, and say explicitly if a category (e.g. Java/Python Flink jobs) does not exist.
- Never put secrets, API keys, account/org/environment/cluster IDs, bucket or storage-account names, or personal data in the file. Use placeholders such as `<id>`, `<cluster-id>`, `<region>`.

## Project-specific context (edit per project)
- Pipeline name / title: <e.g. "Confluent SIEM Optimization: Data Flow">
- Platform boundaries to colour-code: <e.g. Laptop/Docker, Confluent Cloud cluster, Flink compute pool, Kafka Connect, SIEM, Lakehouse, VMs/ Pods running on Cloud or any 3rd party/ cloud hosted service>
- Use cases the "why" lines must call out (where they apply): <e.g. normalization, schema addition,traffic bypassing, DNS tunneling, data exfiltration, sensitive port probing, auth failures, threat-intel IoCs, point-in-time enrichment (temporal joins), PII redaction / pseudonymization, ECS struct packaging, windowed aggregates, volume reduction>
- Sample scenarios to include: <list 5-8 end-to-end stories, including at least one record>

## Technology
- One standalone HTML file: inline CSS and JS, no build step.
- vis-network loaded from cdnjs with an exact pinned version, e.g.
  https://cdnjs.cloudflare.com/ajax/libs/vis-network/9.1.9/standalone/umd/vis-network.min.js
- Optional Google Fonts (Inter for UI, JetBrains Mono for code), with system font fallbacks.
- physics: false, fixed node coordinates. Nodes are draggable. Disable vis-network's built-in wheel zoom (interaction.zoomView = false) and implement zoom yourself (see "Zoom & pan").

## Layout
- Five columns, left to right, with headers drawn ON the canvas so they pan and zoom with the diagram
  (title + small subtitle, small arrows between headers):
  SOURCES → INGESTION TOPICS → FLINK PROCESSING → EGRESS TOPICS → DESTINATIONS
- The Flink column may have sub-lanes: e.g. normalize statement → governed/intermediate topics →
  routing/enrichment statements → metrics statement. Put Schema Registry under the governed topics.
- Reference/lookup topics (upsert tables used in joins) sit at the bottom of the Ingestion column.
- Leave enough horizontal spacing that boundaries of neighbouring components do NOT overlap or
  touch, and their labels never collide.

## Colour-coded boundaries (Confluent ecosystem clearly visible)
- Draw rounded, semi-transparent boundary rectangles BEHIND nodes in vis-network's `beforeDrawing`
  hook. Compute each rectangle from the CURRENT node bounding boxes (network.getBoundingBox) plus
  padding, so boundaries follow nodes when they are dragged.
- Each boundary has a label in its top-left corner, its own colour, and solid or dashed stroke.
  Outer boundary (e.g. the Confluent Cloud cluster) has a thicker solid stroke; nested ones
  (e.g. the Flink compute pool) are dashed.
- Suggested palette (dark UI): laptop/source slate #94a3b8, Confluent Cloud blue #3b82f6,
  Kafka topics cyan #22d3ee, reference/upsert tables amber #f59e0b, Flink violet #c084fc,
  governed topics teal #2dd4bf, egress topics green #34d399, Kafka Connect blue #60a5fa,
  SIEM yellow #facc15, Tableflow/lake rose #fb7185, Schema Registry lime #a3e635.

## Nodes
- Box nodes with 2-3 line labels: bold name on line 1, then the key facts, for example
  "zeek_dns_raw / schemaless JSON", or "<prefix>-<id>-siem-dns / filter · 3 temporal joins · ECS".
- Use vis-network multi-line HTML labels (font.multi = 'html', <b> and <code> only).
  PITFALLS:
  - vis-network does NOT decode HTML entities like &lt; in labels; use "‹id›" in labels and map it
    back to "<id>" in the side panel.
  - Whenever you update a node's style (e.g. dimming), pass the FULL font object, including
    multi:'html' and the bold/code sub-fonts. A partial `font: {color}` silently drops HTML labels and
    shows raw <b> tags.
- Background = node colour at low alpha, border = node colour, soft glow shadow. Flink nodes get a
  thicker border.
- A legend overlay (top-left) explains node colours and edge styles.

## Edges
- Five styles: data flow (solid grey), temporal/lookup join (dashed amber), window input (dotted
  violet), Tableflow/lake (dashed rose), schema (dotted lime). Lookup and window edges are lower
  opacity than data edges so the main flow reads first.
- Horizontal cubic-bezier curves, small arrowheads, short labels (e.g. "produce", "val BYTES",
  "INSERT", "temporal join", "TUMBLE 1 min", "Tableflow", "REST catalog").
- Header toggle "edge labels" shows/hides labels (set the label to ' ' to hide).

## Details panel (right side, "Details" tab)
- Clicking a node shows:
  - kicker with the node type (e.g. "FLINK SQL STATEMENT", "INGESTION TOPIC · KAFKA") in its colour
  - title (resource name with the naming template) and subtitle
  - metadata grid: topic or statement name and naming template, created by (Terraform / Flink DDL /
    connector), schema type and SR subjects, key, partitions/buckets, retention, changelog mode,
    watermark, source file path in the repo, reads → writes, rough volume in/out
  - "Transformations, and why" (Flink nodes) or "What happens here, and why" (others): one card per
    operation. A bold WHAT that names the actual function or construct (JSON_VALUE, TRY_CAST,
    REGEXP_EXTRACT, FOR SYSTEM_TIME AS OF, TUMBLE, UNION ALL, SHA256, REGEXP_REPLACE with
    look-arounds, ROW(...) for ECS, WHERE filter), then a muted "why →" line naming the security or
    business reason from the use-case list above.
  - buttons "▶ <scenario>" for every sample scenario that passes through this node
- The initial empty state explains how to use the page and the naming template.

## Sample data flow ("Sample data flow" tab + header "▶ Show sample data" button + scenario dropdown)
- 5-8 scenarios. Each has an outcome banner: green "Kept → SIEM, <severity>", red "Dropped from the
  SIEM, kept in the lake", or blue "Computed in Flink" (metrics). Each then lists ordered steps
  {node, title, note, data}.
- Data shows the REAL record shape at that stage, taken from the repo's SQL, configs and sample
  outputs: raw log line → shipper JSON → raw topic (key/val bytes) → normalized row → lookup rows
  returned by each join → rule/severity decision → final egress record (nested ECS JSON) → connector
  action → SIEM view → lake row → query-engine forensic query. Shorten long values with "…".
- Pretty-print JSON when it parses, otherwise show it as text. Use simple syntax colouring (keys,
  strings, numbers, booleans, null). Highlight masked tokens and key findings (e.g. [USER], [EMAIL],
  user-<hash>, IoC domain, rule name).
- Selecting a scenario dims every node and edge not on its path (keep full font objects!), then
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
  muted #93a1b6, accent #38bdf8, plus the component colours). Explicit body background.
- Header: title + one-line subtitle on the left. Controls on the right: scenario dropdown,
  ▶ Show sample data (primary), Clear highlight, edge labels toggle, Fit view, Reset layout,
  Minimap, Hide panel.
- Rounded 8-10 px corners, subtle borders, monospace for code and records. Bottom-left hint:
  "Pinch or mouse wheel to zoom · two-finger swipe or drag background to pan · drag nodes ·
  click a node for details".
- Responsive: under ~980 px, stack the canvas (60vh) above the panel. The collapsed panel and a smaller
  minimap still work. No horizontal page scroll.

## Deep links
- `#trace=<scenario-id>&step=<n>` opens a scenario at a step. `#node=<node-id>` opens a node's
  details. List the scenario ids in your final message.

## Verification before you finish
- Render the page in headless Chrome and LOOK at the screenshots, at minimum:
  1. Default overview (labels render as bold HTML, not raw tags; boundaries don't overlap).
  2. Details panel for a Flink node (#node=…).
  3. A long scenario at its last step: the sticky nav stays visible with the "N / N" counter, and
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
```


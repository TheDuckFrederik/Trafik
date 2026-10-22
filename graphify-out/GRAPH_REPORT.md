# Graph Report - graphify  (2026-10-22)

## Corpus Check
- Knowledge graph of the Trafik repository: 56 concepts, 102 components, 73 nets — no file nodes
- Concept provenance kept as node metadata (source doc) only

## Summary
- 231 nodes · 356 edges · 15 communities (all shown)
- Extraction: 94% EXTRACTED · 5% INFERRED (avg confidence 0.93)
- Relations: connects ×264, implemented_by ×20, enables ×6, part_of ×4, accepts ×4, requires ×4, preserves ×3, buffers ×3, wired_into ×3, constrains ×2, generated_by ×2, drives ×2, documents ×2, driven_by ×2, actuates ×2, governs ×2, powers ×2, verified_by ×2, motivates ×1, bypassed_by ×1, covers ×1, justified_by ×1, biases ×1, resolved_by ×1, arbitrated_by ×1, inherent_to ×1, replaces ×1, prevents ×1, triggered_by ×1, parked_in ×1, offered_in ×1, complements ×1, connects_to ×1, extends ×1, normalled_to ×1, offers ×1, belongs_to ×1, feeds ×1, donates ×1, captured_in ×1, informs ×1, applies_to ×1, follows ×1, bounded_by ×1, follows_from ×1

## Graph Freshness
- Built from commit: `41d517b` (41d517b583a82520ceed6c4634cd0d8ec517422d)

## Community Hubs (Navigation)
- **Power** — 19 nodes · hub: `GND` (61 edges)
- **Chain 1 — instrument path** — 14 nodes · hub: `K1` (12 edges)
- **Chain 2 — device FX path** — 12 nodes · hub: `K3` (10 edges)
- **Pedal loops** — 12 nodes · hub: `LA_SEND` (5 edges)
- **Buffers & output bypass** — 40 nodes · hub: `U2` (14 edges)
- **MCU control** — 38 nodes · hub: `U1` (26 edges)
- **Relay drivers** — 20 nodes · hub: `Q1` (3 edges)
- **Indicators** — 24 nodes · hub: `LED1` (2 edges)
- **Signal architecture** — 12 nodes · hub: `Three-buffer map` (5 edges)
- **Routing & states** — 10 nodes · hub: `Pre-FX priority allocation` (8 edges)
- **Fail-safety** — 6 nodes · hub: `Fail-safe hardwire pass-through` (8 edges)
- **Control & firmware** — 8 nodes · hub: `MCU control platform` (5 edges)
- **Product & formats** — 7 nodes · hub: `Custom lane counts (to 8)` (3 edges)
- **Design history** — 3 nodes · hub: `Revision A CMOS (archived)` (3 edges)
- **Verification** — 6 nodes · hub: `Netlist-diff verification` (3 edges)

## Reading the graph
- Three node kinds: **concepts** (knowledge: rules, decisions, specs), **parts** (Rev B components), **nets** (electrical nodes).
- Dashed edges are INFERRED relations; solid are EXTRACTED (stated in a doc). Every node tooltip carries its provenance doc.
- Try: click `Fuzz rule` (what it constrains), `Pre-FX priority allocation` (what it resolves and implements), `Fail-safe hardwire pass-through` (the chain that implements it), `MCU control platform` (what it replaces and enables).
- Open `graph.html` in a browser (vis-network from CDN); search finds any concept, part, or net.

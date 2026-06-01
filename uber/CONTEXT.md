# Uber HLD — Session Context Dump

Everything worth knowing about what was built, why it was built that way, and what to watch out for when editing.

---

## What exists

| File | Purpose |
|---|---|
| `hld.html` | Full reference HLD — 12 sections, problem-driven narrative |
| `slideshow.html` | 24-slide interactive deck — persistent living map, 3 acts |

---

## The core design philosophy (don't break this)

Every component is **arrived at through a problem** — never listed upfront. The structure is always:

```
Constraint → Options (appeal + trade-off) → Decision → New constraint
```

If you add a new section, it must open with a problem the reader can feel before you introduce the solution.

---

## HLD — key decisions and why

### Narrative structure
- Sections 1–2: Requirements + Capacity (only numbers that create hard problems)
- Section 3: Naive design — explicitly built and broken (4 break points)
- Sections 4–8: One section per break, each using the Problem → Options → Decision pattern
- Sections 9–12: API, Final Architecture (Mermaid), Data Models, Failure modes

### Visual elements (CSS classes)
- `.problem` — red left border, dark red background — for stating the constraint
- `.decision` — green left border, dark green background — for the chosen approach
- `.options` + `.opt.no / .opt.maybe / .opt.yes` — options exploration
- `.insight-cols` — 2-column bad/good comparison
- `.observe` — cyan callout for non-obvious insights
- `.mermaid-wrap` — wraps Mermaid diagrams

### Diagrams
Mermaid only (not custom SVG). Keep simple — max 12–15 nodes. There are two diagrams:
1. Naive design (flowchart LR, 4 nodes)
2. Final architecture (flowchart LR, ~12 nodes with subgraphs)

---

## Slideshow — architecture of the map system

### The living SVG map
The right 38% of every slide is a persistent SVG that evolves as slides advance. It has two non-overlapping sections:

**Section A — Naive design (y=0..52):**
```
rider_n   x=2,  y=6,  w=54, h=22
driver_n  x=2,  y=32, w=54, h=22
server_n  x=72, y=19, w=60, h=22  ← center between riders
db_n      x=152,y=19, w=72, h=22
```

**Dashed separator at y=57**

**Section B — Production (y=62..256):**
```
rider     x=2,   y=76,  w=54, h=22
driver    x=2,   y=106, w=54, h=22
gateway   x=72,  y=91,  w=60, h=22  ← center between clients

location  x=148, y=62,  w=72, h=22
dispatch  x=148, y=92,  w=72, h=22
trip      x=148, y=122, w=72, h=22
fare      x=148, y=152, w=72, h=22
ringpop   x=148, y=182, w=72, h=22

redis_geo   x=234, y=62,  w=76, h=22
redis_lock  x=234, y=92,  w=76, h=22
kafka       x=234, y=122, w=76, h=22
postgres    x=234, y=152, w=76, h=22
cassandra   x=234, y=182, w=76, h=22
temporal    x=234, y=212, w=76, h=22
```

**Why two sections?** The previous version put `server_n`/`db_n` at the same coordinates as `gateway`/production services — they overlapped. The fix was to physically separate them in y-space with a dashed line divider.

### Map state per slide
Each slide object has:
```javascript
map: {
  show: ['node_id', ...],     // visible nodes (lit up, not focused)
  active: 'node_id',          // glowing node (the focus of this slide)
  activeColor: '#hex',        // matches the chapter color
  broken: ['node_id', ...]    // red/pulsing (break point slides)
}
```

Section A nodes only ever appear during naive slides (slides 4–9). Never mix A and B in the same `show` array.

### Node visual states
- `ghost` — almost invisible (#0d1017 fill, #14192a border) — not yet introduced
- `visible` — dim but readable (#131b2a fill, #283050 border)
- `active` — glowing in `activeColor` with SVG glow filter
- `broken` — red (#1a0606 fill, #dc2626 border) with red glow filter

### Slide types
| Type | When to use |
|---|---|
| Title (full:true) | Opening and closing only |
| Break | When showing a naive design failure — red box |
| Decision with opts | Every Fix slide — 3-layer option cards |
| Insight-cols | Binary comparisons (bad/good) |
| Code | When the actual command/code is the insight |
| Table | Data models, scale targets |
| Agenda | Transition between acts |

### Chapter color scheme
```
#00b4d8  Act 1 (setup) + Act 4 (full picture)
#f59e0b  Act 2 (naive design / agenda)
#ef4444  Fix #1 (location/storage) + break slides
#8b5cf6  Fix #2 (routing/matching)
#f97316  Fix #3 (consistency/locks)
#06b6d4  Fix #3b (resilience/workflow)
#16a34a  Fix #4 (real-time)
```

---

## Key architectural decisions (why each thing is here)

### Redis Geo (not PostgreSQL, not Batch+PostGIS)
1M writes/sec rules out any relational DB. Batch+PostGIS solves write throughput but introduces 5–10s location lag at assignment time (wrong ETAs). Redis `GEOADD` is O(log n), **overwrites** the previous position (no stale accumulation), and GEOSEARCH is sub-ms. 5M entries = ~500 MB RAM. Repopulates in 5s on restart.

### Kafka + Cassandra (not sync dual-write)
Writing synchronously to Redis + a durable DB at 1M/s couples the fast path to the slow path. Instead: Redis gets the synchronous write (fast), Kafka gets a fire-and-forget publish. A Cassandra consumer appends async. Why Cassandra: append-only time-series (PK = driver_id + recorded_at), LSM-tree = optimal for write throughput, horizontal scale by adding nodes.

### Geo-sharding by city (not global Redis)
A global Redis cluster means a NYC surge degrades São Paulo queries. A failure takes down matching worldwide. Per-city sharding (`geo:drivers:nyc`) contains blast radius, meets data residency requirements, and each shard fits easily in RAM.

### H3 hexagonal grid (not Geohash, not S2)
Geohash uses rectangular cells — diagonal neighbors are √2× farther than edge neighbors. H3 hexagons have all 6 neighbors equidistant. Level 9 ≈ 175m. Query cell + 6 neighbors collapses 5M candidates to hundreds before any ETA computation.

### Road-network ETA scoring (not Euclidean distance)
A driver 800m away across a river: 12 min. One 1.2km away on the same street: 4 min. Straight-line distance is meaningless in dense cities. Modified Dijkstra + contraction hierarchies resolves each ETA in <50ms.

### Redis SETNX EX 10 NX (not app-level lock, not DB CAS)
App-level: instances don't coordinate. DB CAS: timeout recovery is in-memory, lost on crash. `SETNX EX 10 NX` is atomic (one winner), auto-expires in 10s (covers crash AND driver ignoring the offer). No manual cleanup ever.

### Temporal (not setTimeout, not Kafka delay)
setTimeout: lost on restart. Kafka delay: no native per-message delay, requires separate delay service. Temporal persists workflow state, replays from last checkpoint on crash. The entire matching cascade (lock → offer → 10s wait → retry) is a single durable workflow.

### WebSockets + Ringpop (not polling, not central registry)
Polling at 315K concurrent trips = 3.8M req/min of empty responses. WebSockets push on update (~50ms). Central registry (ZooKeeper) is a coordination bottleneck at 10M connections. Ringpop uses consistent hashing to map H3 cell → owning server — any service computes this locally with no lookup. SWIM gossip for decentralized health.

---

## What NOT to do when editing

1. **Don't add an "Architecture Overview" section before the problems** — components must earn their place
2. **Don't write option rejection as a single line** — always show the appeal first, then the specific failure
3. **Don't put Section A map nodes in production slide show arrays** — they'll appear where production nodes are and look broken
4. **Don't use Mermaid in the slideshow** — the right panel IS the diagram. Mermaid reinitializes on slide change and flickers.
5. **Don't change node y-coordinates without checking the full column** — 30px spacing, 22px height, 8px gap is tight. Moving one node can cascade into overlaps.
6. **Don't add more than 5 service nodes or 6 storage nodes** — the viewBox is sized for this. Add a 5th column at x=320 if more are needed.

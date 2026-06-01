# System Design HLDs — Project Context

This directory is a growing library of high-level design documents for big-tech system design interviews. Each system lives in its own subdirectory with two files: a reference HLD and an interactive slideshow.

---

## Directory structure

```
HLDs/
├── .claude/
│   └── commands/
│       └── hld.md          ← /hld skill — generates new HLD + slideshow from a problem statement
├── uber/
│   ├── hld.html            ← Full reference document (12 sections, problem-driven)
│   ├── slideshow.html      ← 24-slide interactive deck (persistent living map)
│   └── CONTEXT.md          ← Uber-specific decisions, node coordinates, what not to touch
└── CONTEXT.md              ← This file
```

To add a new system: open this directory in Claude Code and run `/hld Design [System Name]`.

---

## How to generate a new HLD

From this directory in Claude Code:

```
/hld Design Twitter
/hld Design YouTube
/hld Design WhatsApp
/hld Design Discord
/hld Design Airbnb
```

This creates `<system>/hld.html` and `<system>/slideshow.html`, commits, and pushes to GitHub.

---

## The design philosophy (encode in every HLD)

These rules were developed iteratively across the Uber HLD session. They are the non-negotiable constraints for quality output.

### 1. Problem-driven narrative — never a component inventory

**Wrong:** "The system uses Redis for location, Kafka for streaming, Cassandra for history."

**Right:** "We have 1M writes/sec. Here's why a database fails, why batch+PostGIS falls short, and why Redis Geo solves it. Now that Redis is ephemeral, here's the new problem that creates..."

Every component is introduced when the problem that requires it is established. A reader should never see a technology before they feel the pain it solves.

### 2. Options require three layers — not just a verdict

**Wrong:** "Not DB — too slow."

**Right:**
- **Name:** Direct writes to PostgreSQL
- **Appeal:** Simple, ACID, already in the stack, queryable
- **Verdict:** PG caps at ~10K writes/sec. We need 1M. 100× over the limit before the first rider books.

The appeal layer is critical. It shows the reader (and interviewer) that you took the option seriously and understand its strengths — not just its failure. Dismissing options without acknowledging why they're tempting signals shallow thinking.

### 3. Naive design + explicit break points

Always start with the simplest possible design (2–3 components). Then break it — explicitly, with numbers. Each break point becomes a section. This structure:
- Creates narrative tension
- Motivates every architectural decision
- Shows the interviewer you reason from first principles

### 4. Capacity: only the numbers that create hard problems

Don't enumerate everything. Find the 1–2 numbers that make the architecture non-trivial and lead with them. For Uber it was 1M location writes/sec. For YouTube it might be 500 hours of video uploaded per minute. For Twitter it might be the fan-out write multiplier at celebrity-follower scale.

### 5. The living map is the spine of the slideshow

The right-panel SVG map evolves on every slide. Components are ghosted until introduced, then appear, then glow when focused, then pulse red at break points. The viewer should never lose context of "where are we in the architecture?"

The map is not decoration. It is the persistent thread that holds 24 slides together.

---

## HLD document structure (12 sections, always in this order)

1. **Requirements** — functional (rider/driver equivalent), non-functional, out of scope
2. **Capacity** — only the numbers that create hard problems; column: why it matters
3. **Naive Design** — the simplest possible design, then 3–5 explicit break points
4. **Problem 1** → Options → Decision (becomes section 4 through 4+N)
5. **...**
6. **API Design** — only the non-obvious endpoints; always include the "why a separate estimate step?" pattern
7. **Final Architecture** — Mermaid diagram, simple (≤15 nodes)
8. **Data Models** — table: what data, what store, why that store
9. **Failure & Scale** — stress test each key decision against real failure modes

---

## Slideshow structure (4 acts, ~24 slides)

| Act | Slides | Purpose |
|---|---|---|
| 1 — The Bet | 3–4 | Title, requirements, the one scary constraint |
| 2 — Naive Breaks | 5–7 | Build naive design on map, then crack it 3–5 ways |
| 3 — Fix It | 10–14 | One fix per break. Map grows with each decision. |
| 4 — Full Picture | 3–4 | What was built, data models, failures, closing |

### Slide map system: two non-overlapping sections

**Critical rule:** Section A (naive design nodes) and Section B (production nodes) must never share the same y-coordinate range in the SVG. The previous version had `gateway` at the exact same pixel as `server_n` and `db_n` overlapping `dispatch` — this made the map look broken.

```
Section A — Naive (y = 0..52):
  client_n, server_n, db_n

[Dashed separator at y=57]

Section B — Production (y = 62..256):
  clients, gateway, services (x=148, 30px spacing), storage (x=234, 30px spacing)
```

See `uber/CONTEXT.md` for exact coordinates used in the Uber implementation.

### Chapter color scheme

```
#00b4d8  Setup + synthesis (Act 1 and Act 4)
#f59e0b  Naive design + agenda (Act 2)
#ef4444  Storage / data ingestion fixes
#8b5cf6  Routing / matching / geo fixes
#f97316  Consistency / locking fixes
#06b6d4  Resilience / workflow fixes
#16a34a  Real-time / communication fixes
```

---

## Key HTML/CSS patterns (reuse across all HLDs)

```html
<!-- Problem box (red) -->
<div class="problem">
  <div class="problem-label">The Problem</div>
  <p>[Constraint with numbers and consequences]</p>
</div>

<!-- Option exploration (3 layers each) -->
<div class="options">
  <div class="opt no">
    <span class="opt-pill">Fails</span>
    <div class="opt-body"><strong>[Name]</strong> — [Appeal]. [Verdict with specifics]</div>
  </div>
  <div class="opt maybe">
    <span class="opt-pill">Partial</span>
    <div class="opt-body"><strong>[Name]</strong> — [What it solves]. [The trade-off]</div>
  </div>
  <div class="opt yes">
    <span class="opt-pill">Decision</span>
    <div class="opt-body"><strong>[Chosen approach]</strong> — [Why, precisely]</div>
  </div>
</div>

<!-- Decision box (green) -->
<div class="decision">
  <div class="decision-label">Decision</div>
  <p>[Chosen approach + concrete justification]</p>
</div>

<!-- 2-col bad/good insight -->
<div class="insight-cols">
  <div class="ins-col bad"><div class="ins-head">✕ Option A</div><p>...</p></div>
  <div class="ins-col good"><div class="ins-head">✓ Option B</div><p>...</p></div>
</div>

<!-- Mermaid diagram (HLD only, NOT slideshow) -->
<div class="mermaid-wrap">
  <div class="mermaid">flowchart LR ...</div>
</div>
```

---

## What each system's HLD should uniquely identify

The templates above are structure. The quality comes from finding what makes each system architecturally interesting:

| System | Key constraint | Core architectural challenge |
|---|---|---|
| Uber | 1M location writes/sec | Ephemeral geo-spatial index vs durable history |
| Twitter | Fan-out on write at celebrity scale | Pre-compute timelines vs pull-on-read hybrid |
| YouTube | 500hr video uploaded/min | Async transcoding pipeline, CDN tiering |
| WhatsApp | 100B messages/day, E2E encryption | Message ordering, offline delivery, receipt tracking |
| Airbnb | Search with real-time availability | Eventual consistency vs strong for booking confirmation |
| Discord | 100K concurrent users per server | Voice WebRTC routing, read receipts at scale |

For each system: find the constraint that makes the naive design fail dramatically, and lead with that.

---

## Learnings from the Uber session that apply universally

1. **The diagram appears too late** — the most common mistake in slide decks is showing the architecture at the end as a "here it is" reveal. The map should be the spine of every slide.

2. **Options feel like homework when listed as 3-column tables** — option exploration should be conversational: "here's why you'd try X, here's why it fails, here's what we picked instead."

3. **Chapter title slides kill momentum** — a blank "Chapter 5: No Double Assignment" slide adds nothing. The problem statement IS the chapter title.

4. **Flat emotional arc** — if every section follows problem → 3 options → decision at the same weight, the whole thing feels like a worksheet. Some problems need more setup, some decisions deserve more celebration. Vary the rhythm.

5. **The "aha moment" test** — every architectural decision should give the reader a moment of "oh, THAT's why." If you can't identify that moment for a section, it needs more specificity. "Redis is fast" is not an aha moment. "GEOADD overwrites the previous position so the index never accumulates stale data" is.

6. **GPS coordinates overlap in SVG maps** — always calculate y-ranges explicitly before placing nodes. Two nodes in the same column at overlapping y positions will render as one blurry box. Use Section A / Section B separation as a hard rule.

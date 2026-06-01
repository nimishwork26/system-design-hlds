# /hld — Generate System Design HLD + Slideshow

Generate a high-level design document (`hld.html`) and interactive slideshow (`slideshow.html`) for any system design problem, using the problem-driven narrative approach developed for this project.

**Usage:** `/hld <problem statement>`
**Example:** `/hld Design Twitter`, `/hld Design YouTube`, `/hld Design WhatsApp`

---

## Step 1 — Analyze the problem before writing a single line of HTML

Before generating anything, think through these questions for the specific system:

1. **What is the hardest scale constraint?** (Like Uber's 1M location writes/sec) — this single number should drive most architecture decisions.
2. **What does the simplest possible design look like?** (2-3 components max)
3. **Where does the naive design break?** Find 3–5 break points specific to this system. Each becomes a section.
4. **What are the 3–5 core architectural decisions?** For each: what options exist, why each option is appealing, why we reject or modify it.
5. **What is the name of the system?** Use it as the subdirectory name (lowercase, hyphenated).

---

## Step 2 — Create the directory structure

```bash
mkdir -p /Users/nimish/Documents/HLDs/<system-name>/
```

Then generate two files: `hld.html` and `slideshow.html` inside that directory.

---

## Step 3 — Generate hld.html

The HLD is the comprehensive reference document. Follow this exact structure and use the CSS/HTML patterns below.

### HLD document structure (12 sections)

1. Requirements (functional, non-functional, out of scope)
2. Capacity — only the numbers that create hard problems, with "why it matters" column
3. Start Simple — naive design + explicit break points (3–5 breaks)
4–8. One section per break point: Problem → Options → Decision
9. API Design — only the interesting endpoints (the ones with non-obvious design choices)
10. Final Architecture — Mermaid diagram (simple, not exhaustive)
11. Data Models — table: what goes where and why
12. Failure & Scale — stress test each key decision

### HLD narrative rule — NEVER violate this

Every component is **arrived at** through a problem. The flow is always:

```
PROBLEM (red box) → OPTIONS (rejected + partial + decision) → DECISION (green box) → new PROBLEM
```

Never write "We will use Redis for X". Write "Given constraint Y, what are our options? 
Option A seems good but fails because... Option B is better but trades off... Decision: Redis because..."

### HLD HTML head + CSS (copy this exactly, adjust colors if needed)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>[SYSTEM NAME] — High Level Design</title>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <script>mermaid.initialize({ startOnLoad: true, theme: 'dark', themeVariables: {
    background: '#1a1d27', primaryColor: '#1e3a5f', primaryTextColor: '#e2e8f0',
    primaryBorderColor: '#2e3354', lineColor: '#4a5580', secondaryColor: '#22263a'
  }});</script>
  <style>
    :root { --bg:#0f1117; --surface:#1a1d27; --surface2:#22263a; --border:#2e3354;
            --accent:#00b4d8; --text:#e2e8f0; --muted:#8892b0; }
    *{box-sizing:border-box;margin:0;padding:0;}
    body{background:var(--bg);color:var(--text);font-family:'Segoe UI',system-ui,sans-serif;font-size:15px;line-height:1.75;}
    code{background:var(--surface2);color:#7dd3fc;padding:1px 6px;border-radius:4px;font-size:13px;font-family:monospace;}
    pre{font-family:monospace;font-size:12.5px;color:#7dd3fc;background:var(--surface2);padding:14px;border-radius:8px;overflow-x:auto;margin:8px 0;line-height:1.6;}
    ul,ol{color:var(--muted);font-size:14px;padding-left:20px;}
    li{margin-bottom:4px;} li strong{color:var(--text);}
    .layout{display:flex;min-height:100vh;}
    nav{width:240px;flex-shrink:0;background:var(--surface);border-right:1px solid var(--border);padding:20px 14px;position:sticky;top:0;height:100vh;overflow-y:auto;}
    main{flex:1;padding:44px 54px;max-width:920px;}
    .nav-logo{font-size:16px;font-weight:700;color:var(--accent);margin-bottom:3px;}
    .nav-sub{font-size:10px;color:var(--muted);margin-bottom:6px;text-transform:uppercase;letter-spacing:.08em;}
    .nav-links{display:flex;flex-direction:column;gap:6px;margin-bottom:18px;}
    .nav-link{display:block;padding:5px 10px;border-radius:6px;color:#a78bfa;font-size:12px;text-decoration:none;border:1px solid #3b1f6e;background:#1e1635;text-align:center;}
    .nav-link:hover{background:#2a1a4a;}
    .nav-sec{font-size:10px;color:var(--muted);text-transform:uppercase;letter-spacing:.08em;margin:14px 0 4px;}
    nav ul{list-style:none;padding:0;}
    nav ul li a{display:block;padding:5px 10px;border-radius:6px;color:var(--muted);font-size:12.5px;text-decoration:none;transition:background .15s,color .15s;}
    nav ul li a:hover,nav ul li a.active{background:var(--surface2);color:var(--text);}
    section{margin-bottom:60px;}
    h1{font-size:28px;font-weight:700;color:#fff;margin-bottom:4px;}
    h2{font-size:19px;font-weight:600;color:var(--accent);margin-bottom:14px;padding-bottom:7px;border-bottom:1px solid var(--border);}
    h3{font-size:15px;font-weight:600;color:#c3cfe2;margin:20px 0 8px;}
    p{color:var(--muted);margin-bottom:10px;} p strong{color:var(--text);}
    .hero{margin-bottom:44px;padding-bottom:28px;border-bottom:1px solid var(--border);}
    .hero-meta{display:flex;gap:10px;margin-top:10px;flex-wrap:wrap;}
    .badge{background:var(--surface);border:1px solid var(--border);border-radius:6px;padding:3px 11px;font-size:12px;color:var(--muted);}
    .tags{display:flex;flex-wrap:wrap;gap:6px;margin:10px 0;}
    .tag{background:#1e3a5f;color:#93c5fd;border-radius:999px;padding:2px 11px;font-size:12px;}
    .tag.orange{background:#431407;color:#fdba74;}
    .card{background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:16px 20px;margin-bottom:12px;}
    .card-title{font-size:13.5px;font-weight:600;color:var(--text);margin-bottom:6px;}
    .grid2{display:grid;grid-template-columns:1fr 1fr;gap:14px;}
    .grid3{display:grid;grid-template-columns:1fr 1fr 1fr;gap:14px;}
    /* Problem / Decision */
    .problem{border-left:3px solid #ef4444;background:rgba(239,68,68,.06);border-radius:0 8px 8px 0;padding:12px 16px;margin:16px 0;}
    .problem-label{font-size:10px;font-weight:700;color:#ef4444;text-transform:uppercase;letter-spacing:.08em;margin-bottom:4px;}
    .problem p{color:#fca5a5;margin:0;font-size:14px;}
    .decision{border-left:3px solid #16a34a;background:rgba(22,163,74,.07);border-radius:0 8px 8px 0;padding:12px 16px;margin:16px 0;}
    .decision-label{font-size:10px;font-weight:700;color:#16a34a;text-transform:uppercase;letter-spacing:.08em;margin-bottom:4px;}
    .decision p{color:#86efac;margin:0;font-size:14px;}
    /* Options */
    .options{display:flex;flex-direction:column;gap:8px;margin:12px 0;}
    .opt{display:flex;gap:14px;align-items:flex-start;padding:12px 16px;border-radius:8px;border:1px solid var(--border);}
    .opt.no{border-color:#450a0a;background:rgba(220,38,38,.05);}
    .opt.maybe{border-color:#1c3020;background:rgba(22,163,74,.04);}
    .opt.yes{border-color:#1e1060;background:rgba(99,102,241,.07);}
    .opt-pill{font-size:10px;font-weight:700;text-transform:uppercase;letter-spacing:.06em;padding:2px 9px;border-radius:4px;white-space:nowrap;flex-shrink:0;margin-top:3px;}
    .no .opt-pill{background:#450a0a;color:#fca5a5;}
    .maybe .opt-pill{background:#14532d;color:#86efac;}
    .yes .opt-pill{background:#2e1065;color:#c4b5fd;}
    .opt-body{font-size:13.5px;color:var(--muted);} .opt-body strong{color:var(--text);}
    /* Insight 2-col */
    .insight-cols{display:grid;grid-template-columns:1fr 1fr;gap:12px;margin-top:14px;}
    .ins-col{border-radius:8px;padding:14px 16px;}
    .ins-col.bad{background:#120606;border:1px solid #3a1010;}
    .ins-col.good{background:#060e06;border:1px solid #0e3010;}
    .ins-head{font-size:10px;font-weight:700;text-transform:uppercase;letter-spacing:.07em;margin-bottom:8px;}
    .ins-col.bad .ins-head{color:#ef4444;} .ins-col.good .ins-head{color:#16a34a;}
    .ins-col p{font-size:13.5px;line-height:1.6;color:var(--muted);}
    .ins-col p strong{color:var(--text);}
    /* Observe */
    .observe{border-left:3px solid var(--accent);background:rgba(0,180,216,.06);border-radius:0 8px 8px 0;padding:10px 14px;margin:14px 0;font-size:13.5px;color:var(--muted);}
    .observe strong{color:var(--accent);}
    .observe.warn{border-color:#d97706;background:rgba(217,119,6,.06);} .observe.warn strong{color:#fbbf24;}
    /* Flow steps */
    .flow{display:flex;flex-direction:column;gap:0;margin:14px 0;}
    .flow-step{display:flex;align-items:flex-start;gap:14px;position:relative;padding-bottom:16px;}
    .flow-step:last-child{padding-bottom:0;}
    .flow-step::before{content:'';position:absolute;left:15px;top:32px;bottom:0;width:2px;background:var(--border);}
    .flow-step:last-child::before{display:none;}
    .step-num{width:30px;height:30px;border-radius:50%;background:#7c3aed;color:#fff;display:flex;align-items:center;justify-content:center;font-size:12px;font-weight:700;flex-shrink:0;}
    .step-body{padding-top:4px;}
    .step-title{font-weight:600;color:var(--text);font-size:13.5px;}
    .step-desc{color:var(--muted);font-size:13px;margin-top:2px;}
    /* Mermaid wrapper */
    .mermaid-wrap{background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:24px;margin:18px 0;overflow-x:auto;display:flex;justify-content:center;}
    /* Table */
    table.t{width:100%;border-collapse:collapse;margin:12px 0;font-size:13.5px;}
    table.t th{padding:8px 12px;background:var(--surface2);color:var(--muted);font-size:11px;text-transform:uppercase;letter-spacing:.05em;text-align:left;}
    table.t td{padding:9px 12px;border-bottom:1px solid var(--border);color:var(--text);}
    table.t tr:last-child td{border-bottom:none;}
    .num{color:var(--accent);font-weight:700;}
    .tip{background:#1c1435;border:1px solid #3b1f6e;border-radius:8px;padding:12px 16px;margin:14px 0;}
    .tip-label{font-size:10px;color:#a78bfa;font-weight:700;text-transform:uppercase;letter-spacing:.08em;margin-bottom:3px;}
    .tip p{color:#c4b5fd;margin:0;font-size:13.5px;}
    ::-webkit-scrollbar{width:5px;height:5px;} ::-webkit-scrollbar-thumb{background:var(--border);border-radius:3px;}
  </style>
</head>
```

### HLD nav + main wrapper pattern

```html
<body>
<div class="layout">
<nav>
  <div class="nav-logo">[SYSTEM] HLD</div>
  <div class="nav-sub">System Design Interview</div>
  <div class="nav-links">
    <a href="slideshow.html" class="nav-link">Slideshow ↗</a>
  </div>
  <div class="nav-sec">Design Flow</div>
  <ul>
    <li><a href="#requirements">1. Requirements</a></li>
    <li><a href="#estimation">2. Capacity</a></li>
    <li><a href="#naive">3. Naive Design</a></li>
    <!-- one entry per break point / fix -->
    <li><a href="#architecture">N. Final Architecture</a></li>
    <li><a href="#datamodels">N+1. Data Models</a></li>
    <li><a href="#failure">N+2. Failure & Scale</a></li>
  </ul>
</nav>
<main>
  <!-- sections here -->
</main>
</div>
<script>
  const secs = document.querySelectorAll('section[id]');
  const links = document.querySelectorAll('nav a[href^="#"]');
  secs.forEach(s => new IntersectionObserver(entries => {
    entries.forEach(e => {
      if (e.isIntersecting) {
        links.forEach(l => l.classList.remove('active'));
        const a = document.querySelector(`nav a[href="#${e.target.id}"]`);
        if (a) a.classList.add('active');
      }
    });
  }, { rootMargin:'-20% 0px -70% 0px' }).observe(s));
</script>
</body>
```

### Key HLD section patterns

**Problem section:**
```html
<div class="problem">
  <div class="problem-label">The Problem</div>
  <p>[Specific constraint in numbers and consequences]</p>
</div>
```

**Options section (ALWAYS 3 layers per option: name, appeal, verdict):**
```html
<div class="options">
  <div class="opt no">
    <span class="opt-pill">Fails</span>
    <div class="opt-body">
      <strong>[Option name]</strong> — [What makes it seem reasonable].<br>
      [Specific reason it fails with numbers/consequences]
    </div>
  </div>
  <div class="opt maybe">
    <span class="opt-pill">Partial</span>
    <div class="opt-body">
      <strong>[Option name]</strong> — [What problem it solves].<br>
      [The specific trade-off that makes it insufficient]
    </div>
  </div>
  <div class="opt yes">
    <span class="opt-pill">Decision</span>
    <div class="opt-body"><strong>[Chosen approach]</strong> — [Precise justification with specifics]</div>
  </div>
</div>
```

**Decision box:**
```html
<div class="decision">
  <div class="decision-label">Decision</div>
  <p>[The chosen approach + why, in concrete terms]</p>
</div>
```

**Mermaid diagram (keep simple — 10-15 nodes max):**
```html
<div class="mermaid-wrap">
  <div class="mermaid">
flowchart LR
    Client --> Gateway
    Gateway --> ServiceA & ServiceB
    ServiceA --> StoreA
    ServiceB --> StoreB
  </div>
</div>
```

---

## Step 4 — Generate slideshow.html

The slideshow is the story version. Same content, narrative format. Every slide has a persistent SVG map on the right that evolves as the story progresses.

### Story arc (strict 4-act structure)

- **Act 1 — The Bet** (3–4 slides): Title + requirements + the one scary constraint number
- **Act 2 — Naive Design Breaks** (5–7 slides): Build simple diagram on map, then show 3–5 break points (each breaks something visible in the map)
- **Act 3 — Fix It, Piece by Piece** (10–14 slides): One section per fix. Map grows. Each decision slide has option cards.
- **Act 4 — Full Picture** (3–4 slides): Summary of what was built, data models, failure modes, closing with 3 hard problems

### Slideshow HTML structure

Copy the full slideshow.html from `../uber/slideshow.html` as a base and adapt it. The JavaScript engine (map rendering, navigation, transitions) is reusable. The things to change are:

1. **`const N = {}`** — redefine nodes for this system's components
2. **`const CONNS = []`** — redefine connections
3. **`const SLIDES = []`** — rewrite all slides for this system
4. System name in `<title>` and title slide

### Map node layout rules (CRITICAL — prevents overlaps)

**Two sections, never overlapping:**

```
Section A — Naive design (y=0..52):
  rider-equiv_n:  x=2,  y=6,  w=54, h=22
  driver-equiv_n: x=2,  y=32, w=54, h=22
  server_n:       x=72, y=19, w=60, h=22   ← center of section A clients
  db_n:           x=152,y=19, w=72, h=22

Dashed divider at y=57

Section B — Production (y=62+):
  client1:  x=2,   y=76,  w=54, h=22
  client2:  x=2,   y=106, w=54, h=22
  gateway:  x=72,  y=91,  w=60, h=22   ← center of section B clients
  svcA:     x=148, y=62,  w=72, h=22
  svcB:     x=148, y=92,  w=72, h=22
  svcC:     x=148, y=122, w=72, h=22
  svcD:     x=148, y=152, w=72, h=22
  svcE:     x=148, y=182, w=72, h=22
  storeA:   x=234, y=62,  w=76, h=22
  storeB:   x=234, y=92,  w=76, h=22
  storeC:   x=234, y=122, w=76, h=22
  storeD:   x=234, y=152, w=76, h=22
  storeE:   x=234, y=182, w=76, h=22
  storeF:   x=234, y=212, w=76, h=22
```

**Rules:**
- Section A nodes NEVER share y-range with Section B nodes
- Within each column, nodes are spaced 30px apart (h=22 + 8px gap)
- `gateway` and `server_n` CAN share same (x,y) since they are mutually exclusive
- viewBox height = last node y + 22 + 10. Width = 314
- Max 5 service nodes, 6 storage nodes. If more needed, add a 5th column at x=320 or merge related services

### Map state per slide

Each slide declares what the map should show:
```javascript
map: {
  show: ['node_id', ...],        // visible (lit) but not focused
  active: 'node_id',             // glowing in chapter color
  activeColor: '#hex',           // color matching the current fix chapter
  broken: ['node_id', ...]       // red/pulsing (break point slides)
}
```

Section A nodes (`rider_n`, `driver_n`, `server_n`, `db_n`) only appear in `show` during naive slides.
Section B nodes only appear in `show` during production slides.
Never mix A and B in the same `show` array.

### Slide types reference

**Title slide (full-screen):**
```javascript
{ act:'', actColor:'#00b4d8', full:true, map:{show:[],active:null,broken:[]},
  html:`<div class="slide slide--title">
    <div class="s-act" style="color:#00b4d8">System Design Interview</div>
    <div class="s-title">Designing [SYSTEM]</div>
    <div class="s-sub">Every component earned through a problem.</div>
  </div>` }
```

**Break slide:**
```javascript
{ act:'Break #N', actColor:'#ef4444',
  map:{show:NAIVE, active:null, broken:['db_n']},
  html:`<div class="slide">
    <div class="s-act" style="color:#ef4444">Break #N</div>
    <div class="s-title">[Short, punchy title]</div>
    <div class="break-box"><p>[Specific constraint with numbers]</p></div>
  </div>` }
```

**Decision slide with options (3-layer cards):**
```javascript
{ act:'Fix #N — [Area]', actColor:'#hex',
  map:{show:[...], active:'node_id', activeColor:'#hex', broken:[]},
  html:`<div class="slide">
    <div class="s-act" style="color:#hex">Fix #N — [Area]</div>
    <div class="s-title">[The question being answered]</div>
    <div class="opts">
      <div class="opt no">
        <div class="opt-name"><span class="opt-pill">✕ Rejected</span>[Option A name]</div>
        <div class="opt-appeal">[Why it seemed good — be specific]</div>
        <div class="opt-verdict">[Why it fails — with numbers]</div>
      </div>
      <div class="opt partial">
        <div class="opt-name"><span class="opt-pill">⚠ Better</span>[Option B name]</div>
        <div class="opt-appeal">[What problem it solves]</div>
        <div class="opt-verdict">[The trade-off that makes it insufficient]</div>
      </div>
    </div>
    <div class="dec-box">
      <div class="dec-label">Decision</div>
      <p>[Chosen approach + specific justification]</p>
    </div>
  </div>` }
```

**Code slide:**
```javascript
{ act:'...', actColor:'#hex', map:{...},
  html:`<div class="slide">
    <div class="s-act" style="color:#hex">...</div>
    <div class="s-title">...</div>
    <div class="code-block">[code here, use pre-formatted text]</div>
    <div class="code-note">[1-2 line explanation of the key insight in the code]</div>
  </div>` }
```

**Insight 2-column (bad vs good):**
```javascript
html:`<div class="slide">
  <div class="s-act" ...>...</div>
  <div class="s-title">...</div>
  <div class="insight-cols">
    <div class="ins-col bad"><div class="ins-head">✕ Option A</div><p>...</p></div>
    <div class="ins-col good"><div class="ins-head">✓ Option B</div><p>...</p></div>
  </div>
</div>`
```

**Closing slide (full-screen):**
```javascript
{ act:'', actColor:'#00b4d8', full:true, map:{show:ALL,active:null,broken:[]},
  html:`<div class="slide" style="width:100%;height:100%;display:flex;align-items:center;justify-content:center;padding:56px 80px;">
    <div style="max-width:640px;width:100%">
      <div class="s-act" style="color:#00b4d8;margin-bottom:18px">The Three Hard Problems</div>
      <div class="closing-items">
        <div class="closing-item">
          <div class="closing-n" style="color:#hex">01</div>
          <div class="closing-text"><strong>[Hard problem 1]</strong><span>[The solution in one sentence]</span></div>
        </div>
        <!-- repeat for 02, 03 -->
      </div>
    </div>
  </div>` }
```

### Chapter color scheme (use consistently)

```
Act 1 (stakes/setup):      #00b4d8  (cyan)
Act 2 (breaks):            #ef4444  (red) for break slides, #f59e0b for agenda
Fix — data/storage:        #ef4444  (red)
Fix — matching/routing:    #8b5cf6  (purple)
Fix — consistency/locks:   #f97316  (orange)
Fix — resilience/workflow: #06b6d4  (teal)
Fix — real-time/comms:     #16a34a  (green)
Act 4 (full picture):      #00b4d8  (cyan)
```

### Slideshow CSS additions needed (beyond HLD base CSS)

Add these to the slideshow's `<style>` block:

```css
/* Options in slideshow */
.opts{display:flex;flex-direction:column;gap:8px;margin:12px 0 10px;}
.opt{border-radius:8px;padding:11px 15px;border:1px solid;}
.opt.no{background:#120606;border-color:#3a0e0e;}
.opt.partial{background:#121003;border-color:#3a2e08;}
.opt-name{font-size:12.5px;font-weight:600;color:#e2e8f0;margin-bottom:3px;}
.opt-appeal{font-size:11.5px;color:#4a5580;margin-bottom:5px;font-style:italic;}
.opt-verdict{font-size:12.5px;line-height:1.55;}
.opt.no .opt-verdict{color:#f87171;}
.opt.partial .opt-verdict{color:#fbbf24;}
.opt-pill{display:inline-block;font-size:9px;font-weight:700;text-transform:uppercase;letter-spacing:.06em;padding:1px 6px;border-radius:3px;margin-right:5px;vertical-align:middle;}
.opt.no .opt-pill{background:#3a0e0e;color:#f87171;}
.opt.partial .opt-pill{background:#3a2e08;color:#fbbf24;}
/* Decision box */
.dec-box{border-left:3px solid #16a34a;background:rgba(22,163,74,.07);border-radius:0 10px 10px 0;padding:14px 18px;margin-top:4px;}
.dec-label{font-size:9px;font-weight:700;color:#16a34a;text-transform:uppercase;letter-spacing:.09em;margin-bottom:6px;}
.dec-box p{font-size:15px;color:#86efac;line-height:1.65;}
/* Break box */
.break-box{border-left:3px solid #dc2626;background:rgba(220,38,38,.07);border-radius:0 10px 10px 0;padding:14px 18px;margin-top:14px;}
.break-box p{font-size:16px;color:#fca5a5;line-height:1.65;}
/* Closing */
.closing-items{margin-top:32px;display:flex;flex-direction:column;gap:18px;}
.closing-item{display:flex;gap:14px;align-items:flex-start;}
.closing-n{font-size:26px;font-weight:800;line-height:1;flex-shrink:0;width:36px;}
.closing-text strong{display:block;font-size:16px;color:#e2e8f0;margin-bottom:3px;}
.closing-text span{font-size:13.5px;color:#5a6494;line-height:1.5;}
```

---

## Step 5 — Commit to git

After generating both files:

```bash
git -C /Users/nimish/Documents/HLDs add <system-name>/
git -C /Users/nimish/Documents/HLDs commit -m "Add [System Name] HLD and slideshow"
git -C /Users/nimish/Documents/HLDs push
```

---

## Quality checklist before finishing

Before declaring done, verify each item:

- [ ] HLD: Every component is introduced through a problem section, never listed upfront
- [ ] HLD: Every option has appeal + verdict (not just verdict)
- [ ] HLD: All Mermaid diagrams are simple (≤15 nodes)
- [ ] HLD: Naive design section exists with explicit break points
- [ ] HLD: nav links all match section IDs
- [ ] HLD: Slideshow link in nav points to `slideshow.html`
- [ ] Slideshow: Title slide is full-screen (`full: true`)
- [ ] Slideshow: Closing slide is full-screen
- [ ] Slideshow: Map Section A nodes (naive) and Section B nodes (production) never share y-ranges
- [ ] Slideshow: `server_n` / `db_n` only appear in naive slides' `show` arrays
- [ ] Slideshow: Each fix chapter uses a consistent `actColor`
- [ ] Slideshow: Decision slides have option cards with `opt-name`, `opt-appeal`, `opt-verdict`
- [ ] Slideshow: Progress bar and chapter indicator work (they use `actColor`)
- [ ] Slideshow: `ALL` constant includes all Section B production nodes
- [ ] Both files: Links between hld.html ↔ slideshow.html are relative (not absolute)

---

## What makes a great HLD for THIS specific workflow

The patterns above are structural. The quality comes from getting the system-specific details right:

1. **Find the non-obvious constraint**: For Uber it was 1M location writes/sec. For Twitter it might be fan-out on write at celebrity-follower scale. For YouTube it might be the video transcoding pipeline. Find the thing that makes this system architecturally interesting and lead with it.

2. **Make the naive design fail in a way that's specific**: Don't use generic "it won't scale." Show the exact number, the exact failure mode, the exact dollar cost or latency consequence.

3. **Show why the chosen technology fits THIS use case specifically**: Redis Geo for Uber isn't just "Redis is fast" — it's that GEOADD overwrites in place (no stale accumulation) and GEOSEARCH is O(log n) on a sorted set. Get that level of specificity for every decision.

4. **The "aha" moment test**: Each decision should give the reader a moment of "oh, THAT's why they do it that way." If a section doesn't have that moment, it needs more specificity.

# GroupLogic IDE — Documentation & Study

> An advanced group generator that's **not really random**. Define who wants to be together, who must be separated, score-based banding, role balancing — and let the engine optimize.

Three language tiers from simple to full Python integration. Code-driven classroom layout. Cloud + local saves. Multi-class management. Built for real teachers. Open source. **Now available online at [sites.google.com/grouplogic-ide](https://sites.google.com/grouplogic-ide)**

```
┌─────────────────────────────────────────────────┐
│  Simple        │  Advanced      │  GroupLogic.py│
│  Alice: +Bob   │  Alice: +Bob@8 │  import gl    │
│        -Charlie│  role="leader" │  c = gl.Room()│
└─────────────────────────────────────────────────┘
```

---

## Table of Contents

1. [Why GroupLogic?](#why-grouplogic)
2. [How It Works](#how-it-works)
3. [The Language](#the-language)
4. [Code-Driven Classroom Layout](#code-driven-classroom-layout)
5. [Roadmap](#roadmap)
6. [Feature Status](#feature-status)
7. [Vision](#vision)
8. [Bug Report](#bug-report)

---

## Why GroupLogic?

Group assignment is one of the most common yet poorly solved problems in education, team management, and event planning. The default approach — random assignment — ignores the social dynamics and performance data that make or break group effectiveness.

### The Random Problem

Most group generators use pure randomness. But putting two people who can't work together into the same group isn't "fair" — it's careless. Random assignment treats all social connections as equal when they absolutely are not. A group with mutual conflict performs measurably worse than a group with mutual respect, yet random assignment makes no distinction between them. The result is wasted time, frustrated participants, and outcomes that underperform what the same people could achieve in better-arranged teams.

### Score-Based Grouping Is Real

Teachers already use non-random methods manually. A common approach: after a test, sort students by score and group similar-scoring students together so peers can help each other. Some create 2-3 "bands" (high, medium, low) and pair within bands, or strategically mix bands so stronger students support weaker ones. This works — but it's manual, tedious, and doesn't scale to 8 class periods. GroupLogic IDE makes this programmable: define score bands, set grouping strategy, and the engine handles the rest.

### Programmable, Not Manual

Manual group assignment is tedious and biased. The teacher or manager picks groups based on incomplete information and their own assumptions. GroupLogic IDE replaces this with a *programmable* approach: you express preferences, scores, roles, and constraints in a language, and the engine finds the optimal arrangement. It's auditable, repeatable, and unbiased. Change one rule and re-run — the result updates instantly without starting over from scratch.

### Random vs. Optimized: A Comparison

| Factor | Random Assignment | GroupLogic IDE |
|--------|------------------|----------------|
| **Conflict handling** | None — conflicts are accidental | Hard avoidance constraints enforced first |
| **Score-based grouping** | Impossible — no score awareness | Band students by test score, group within or across bands |
| **Role balance** | None — all roles treated same | Tag roles, ensure each group has required roles |
| **Overflow handling** | Solos and empty groups | 4 modes: absorb, even, newgroup, small — no one left alone |
| **Preference satisfaction** | Statistical luck (~0% for large groups) | Optimized via Monte Carlo + Hill Climbing |
| **Repeatability** | Different every time, no audit trail | Same rules = same optimal result |
| **Multi-class management** | N/A — one-off manual process | Save rosters per class, switch instantly |
| **Transparency** | "The computer picked randomly" | Rules visible, conflicts highlighted, score quantified |

> **The Key Insight:** "Random" group assignment isn't neutral — it's a *default that ignores information*. When participants have known preferences, test scores, skill levels, and conflicts, pretending those don't exist doesn't create fairness; it creates preventable failure. GroupLogic IDE's philosophy is simple: **if you have data, use it**. Score bands, role tags, avoidance constraints, weighted preferences — every piece of information the optimizer has makes the result better.

---

## How It Works

The engine uses a two-stage optimization pipeline designed for constraint satisfaction. Stage 1 casts a wide net with randomized sampling; Stage 2 tightens the result with targeted local search.

### The Pipeline

```
Parse Rules  →  Monte Carlo  →  Hill Climbing  →  Hard Guarantee
(build graph)   (2,500 shuffles)  (swap conflicts)  (no empty groups)
```

### Stage 1: Stochastic Monte Carlo

The engine generates 2,500 independent random groupings by shuffling all students and distributing them round-robin into groups of the target size. Each trial is immediately scored using the weighted evaluation function. The trial with the fewest constraint violations (and highest satisfaction score among ties) becomes the candidate for refinement. In advanced mode, the initial shuffle respects score bands, so students with similar scores are more likely to start in the same group before optimization reshuffles them.

```python
for i = 0 .. 2500:
  trial = random_shuffle(students)
  trial_groups = distribute(trial, group_sizes)
  if score(trial_groups) > best_score:
    best = trial_groups
  if violations == 0: break
```

### Stage 2: Hill Climbing Refinement

If violations remain after Stage 1, the engine identifies students currently placed in groups with their avoided peers and attempts to move them to another group. For each conflicting student, every possible target group is tested. If a swap reduces total violations, it's accepted immediately. This process repeats up to 20 refinement loops or until no further improvement is possible. In advanced mode, the hill climbing also considers score proximity and role coverage as secondary optimization targets.

```python
for round = 0 .. 20:
  for each student in conflict:
    for each other_group:
      test = swap(student, other_group)
      if violations(test) < violations(best):
        accept(test); break
  if !improved: break
```

### Stage 3: Hard Guarantee Post-Processing

After Hill Climbing, a hard guarantee pass enforces group size constraints. Empty groups are eliminated by moving students from the largest groups. Oversized groups are trimmed by moving excess to the smallest groups. In absorb overflow mode, solo groups are merged into existing groups so no one is left alone. A final student count verification ensures no student was dropped or duplicated during optimization.

### Overflow Modes

When students don't divide evenly into groups, the overflow mode controls what happens to the extra students:

| Mode | Behavior | Example (7 students, group_size=2) |
|------|----------|-------------------------------------|
| **absorb** | Absorb overflow into existing groups — no solo groups | [3, 2, 2] |
| **even** | Distribute as evenly as possible across all groups | [2, 2, 2, 1] |
| **newgroup** | Full groups first, then a new smaller group for overflow | [2, 2, 2, 1] |
| **small** | Same as newgroup (legacy alias) | [2, 2, 2, 1] |

**Absorb mode** is the default and is specifically designed for pair work and scenarios where no student should be left in a solo group. Set with `@overflow = absorb` in Advanced mode, or click the "Absorb" button in Simple mode.

### Scoring Function

The evaluation function assigns a numerical score to each candidate grouping. The scoring is deliberately asymmetric: avoidance violations are penalized far more heavily than preference satisfactions are rewarded. This ensures the optimizer always prioritizes separating conflicting pairs over pairing preferred ones.

```python
# Per pair (a, b) in the same group:
if a.wants(b):   score += +500       # Soft preference (scaled by weight)
if a.avoids(b):  score += -100,000   # Hard constraint (200x weight)

# Group-level penalties:
if group.size == 0:   score += -10,000,000  # Empty group (NEVER selected)
if group.size == 1:   score += -1,000       # Solo group (mild penalty)
if group.size > max:  score += -10,000/extra # Oversized group

# Advanced mode additions:
if |a.score - b.score| < band_width: score += +200
if !group.has_role("leader"):        score += -10,000
```

> **Why the asymmetry?** A group where two people conflict is often worse than a group where nobody specifically requested each other. Conflict can derail an entire project, while the absence of a preferred partner is merely suboptimal. The 200:1 penalty ratio ensures the engine will break preference pairs to resolve conflicts before it considers breaking preference pairs to satisfy other preferences. This mirrors how real facilitators think: "First, do no harm." Score proximity and role coverage are secondary — important, but never at the cost of creating a conflict.

---

## The Language

Three language tiers, switchable in the editor with one click. Start simple, grow into power. Every tier uses the same engine — the language just controls how much you can express.

### Tier 1: Simple — The OG Language `[Current]`

The original GroupLogic DSL. One line per person, prefix operators for preferences, nothing else to learn. Designed for the 5-second onboarding: if you can type a name and add `+` or `-` in front of another name, you can use it. This tier is perfect for simple social constraints — who likes who, who can't work with who — and covers 80% of real classroom use cases.

#### Syntax Reference

| Syntax | Meaning |
|--------|---------|
| `Name:` | A person definition. Names are case-insensitive. Each name defines one "node" in the social graph. |
| `+Target` | A **want** constraint. Soft — the optimizer tries to satisfy it but will break it to resolve a conflict. |
| `-Target` | An **avoid** constraint. Hard constraint with 200x the weight of a want. |
| `// comment` | Comment line. Ignored by the parser. Use to annotate or temporarily disable rules. |

#### Overflow Buttons (Simple Mode)

In Simple mode, the overflow behavior is controlled by buttons below the editor:

- **Absorb** — Extra students join existing groups (no solo groups). Best for pair work.
- **Even Out** — Distribute overflow evenly across groups.
- **New Group** — Put overflow students in a new smaller group.
- **Small Last** — Last group is intentionally smaller.

#### Example

```gl
// Science Fair Teams (groups of 3)
Alice: +Bob +Diana -Charlie
Bob:   +Alice -Eve
Charlie: -Alice
Diana: +Alice
Eve:   +Frank -Bob
Frank: +Eve
Grace: +Frank
Henry:
```

### Tier 2: Advanced — Scores, Roles, Directives & Rules `[Current]`

Extends the simple syntax with everything a teacher needs for data-driven grouping. Score-based banding groups students by test performance. Role tags ensure each group has the right mix. Weighted preferences let you express "I strongly want Bob" vs. "I'd mildly prefer Diana." The `@separate` and `@together` directives handle multi-person constraints. Custom `@rule` blocks define conditional logic. All while keeping line-by-line readability.

#### Directive Reference

| Directive | Meaning |
|-----------|---------|
| `@group_size = N` | Target students per group. Replaces the "Per Group" UI control. |
| `@overflow = absorb\|even\|newgroup\|small` | How to handle extra students. **absorb** = no solo groups. |
| `@balance = scores,gender,roles,age` | Balance criteria (comma-separated). |
| `@band_width = N` | Max score spread within a group. |
| `@separate = A,B,C` | All listed students must be in different groups. |
| `@together = A,B` | Hard pair: these students must be in the same group. |
| `@rounds = mixed:score"Ionic", similar:score"PH", random"Mollar"` | Multi-round assignments with different strategies. |
| `@no_repeat_partners = true` | Ensure students work with different partners across rounds. |
| `@role name color=#hex icon=person overflow=auto\|always\|never` | Define a role with color, icon, and overflow behavior. |
| `@rule name ... @end` | Custom rule with conditions and actions. |
| `@strict = true` | Fail if any constraints cannot be satisfied. |
| `@iterations = N` | Override default Monte Carlo iterations. |
| `@min_group = N` / `@max_group = N` | Hard bounds on group sizes. |
| `@group_name = "Lab %d"` | Custom naming pattern. `%d` = number, `%s` = size. |

#### Overflow Modes in Advanced

```gl
@overflow = absorb   // No solo groups — extra students absorbed into existing groups
@overflow = even     // Distribute overflow evenly across groups
@overflow = newgroup // Full groups first, then a smaller group for overflow
@overflow = small    // Same as newgroup (legacy)
```

#### Role Overflow Behavior

Roles defined with `@role` have an `overflow` setting that controls how they're handled when student counts are odd:

- **auto** — Included when non-role student count is odd, excluded when even (e.g., teachers/TA)
- **always** — Always included in grouping (e.g., leaders)
- **never** — Always excluded from grouping (e.g., observers, judges)

```gl
@role teacher color=#f59e0b icon=person overflow=auto
@role observer color=#6b7280 icon=eye overflow=never
@role leader color=#3b82f6 icon=crown overflow=always
```

#### Real-World Classroom Example

```gl
// Ms. Rodriguez's 3rd Period — Real classroom constraints
@group_size = 4
@overflow = absorb
@balance = scores
@band_width = 15
@separate = jake,mario,tyrell    // These 3 talk constantly — split them up
@separate = sam,devon            // Exes — keep apart
@together = maya,sofia           // Best study partners

// --- Students (score = last test grade) ---
Maya: score=92 +Sofia
Sofia: score=88 +Maya
Priya: score=75
Jake: score=60 -Mario
Mario: score=55 -Jake
Sam: score=82
Devon: score=70
Tyler: score=95
Ava: score=85
Liam: score=48
Zoe: score=78
Tyrell: score=62 -Jake
Emma: score=90
Noah: score=72
```

#### Multi-Round Example

```gl
// Chemistry Lab — 3 rounds of partner pairs
@group_size = 2
@min_group = 2
@max_group = 2
@balance = scores
@iterations = 5000
@no_repeat_partners = true
@rounds = similar:score"Similar Pairs", mixed:score"Mixed Pairs", random"Random Partners"

Alex: score=92
Ben: score=88
Charlie: score=55
David: score=42
Evan: score=75
Frank: score=30
George: score=95
Henry: score=68
Iris: score=90
Jane: score=58
Kate: score=45
Lily: score=78
Mia: score=32
Nina: score=82
Olivia: score=65
Paula: score=71
```

### Tier 3: GroupLogic.py — Python-Powered Super Advanced `[Planned]`

The ultimate tier. GroupLogic.py combines Python's familiarity with GroupLogic's domain-specific primitives. For advanced users — teachers who code, data scientists, researchers — who need to express grouping logic beyond any DSL. Python holds your hand with familiar syntax while GroupLogic's library provides the domain power.

Instead of inventing a new language for complex scenarios, we embed GroupLogic's constraint system inside Python. You get Python's full expressiveness (loops, conditionals, data structures, imports) plus GroupLogic's engine as a library. Write a script that loads scores from CSV, computes dynamic bands, applies different strategies per subject, generates multi-round assignments, and exports as PDF — all in one file.

#### Library API

```python
import grouplogic as gl

# Create a classroom
c = gl.Classroom("Period 3 Physics")

# Add students with rich data
c.add("Alice",
  score=92, role="leader",
  wants=["Bob"], avoids=["Charlie"]
)

# Load scores from CSV
c.load_scores("test_results.csv")

# Generate
result = c.generate(group_size=4)
result.to_csv("groups.csv")
result.to_pdf("groups.pdf")
```

#### What GroupLogic.py Enables

- **Dynamic band computation**: Calculate score bands from data, not hardcoded values. Adjust band widths based on score distribution. Apply different band strategies per subject or class period.
- **Multi-round scheduling**: Generate week-by-week assignments with no repeat pairings. Track history. Ensure every student works with every other student at least once over a semester.
- **Custom scoring functions**: Override the default scoring with your own Python function. Weight score proximity differently for AP vs. remedial classes. Add custom constraints that the DSL can't express.
- **Data integration**: Load student data from CSV, JSON, Google Sheets. Export results in any format. Integrate with existing gradebooks and LMS platforms.
- **Room layout engine**: Define your classroom in code and let the solver place students at seats, respecting proximity constraints. Not a visual drag-and-drop — actual code that's version-controlled and shareable.

---

## Code-Driven Classroom Layout

You code your classroom. Define tables, seats, and proximity in the Advanced language or GroupLogic.py — not a fancy drag-and-drop editor, but actual code you write and edit. The engine reads your room definition and places students at seats, respecting proximity constraints. Hover over any seat to see who that student is likely to sit near (green) and who they must be kept away from (red).

### Real Classroom Example

```gl
// Real classroom constraints — Ms. Rodriguez's 3rd Period
@group_size = 4
@overflow = absorb
@balance = scores
@band_width = 15
@separate = jake,mario,tyrell    // These 3 talk constantly — split them up
@separate = sam,devon            // Exes — keep apart
@together = maya,sofia           // Best study partners

// Seating needs
// Jake & Mario can't sit in the back (they talk)
// Priya needs to sit closer (can't see the board well)
// 1 strong student per table for peer support
```

### How Code-Driven Works

**Keep Talkers Apart:** Use `@separate` to keep chatty students in different groups. The engine ensures they're never placed together, and with `@overflow = absorb`, nobody gets left in a solo group. This is one of the most common real-world constraints teachers face — certain combinations of students simply cannot be in the same group without disrupting the entire class.

**1 Strong Student Per Table:** With `@balance = scores` and `@band_width = 15`, score-based grouping distributes strong and weaker students across groups so each table has a peer supporter. Use `@overflow = absorb` so pairs don't leave anyone alone. The band width parameter controls how close scores need to be for students to be considered "similar" — a narrower band creates more homogeneous groups, while a wider band allows more mixing.

**Seating Needs & Proximity:** Future classroom layout feature will support physical positioning: "sit closer" and "not in the back" constraints mapped to actual table positions. For now, use `@separate` for behavioral separation and `@together` for study partnerships. The planned `room` / `table` / `near` / `far` syntax will bring full spatial awareness to the grouping engine.

### Planned Room Layout Syntax

```gl
// Future: Code the room - tables, seats, proximity
room PhysicsLab:
  table "T1" seats=4 pos="front-left"
  table "T2" seats=4 pos="front-right"
  table "T3" seats=4 pos="back-left"
  table "T4" seats=4 pos="back-right"

  // Which tables are close to each other
  near "T1", "T2"
  near "T3", "T4"
  far  "T1", "T4"

// Proximity-aware separation
far Alice, Charlie  // Not just different groups - different tables that aren't adjacent
near Bob, front     // Seat Bob near the teacher
```

---

## Roadmap

From the current simple DSL to an open-source platform with community contributors. Each phase expands what's expressible while preserving the "simple on the surface, powerful underneath" philosophy.

### v2.6 — Foundation `[Shipped]`

The foundation: working IDE with live syntax highlighting, Monte Carlo + Hill Climbing optimization, interactive roster with hover-to-inspect, conflict detection with pulsing glow, and plain-text export. Simple language with +want / -avoid.

`+want / -avoid` · `Monte Carlo` · `Hill Climbing` · `Syntax Highlight` · `Conflict Glow`

### v3.0 — Official Site + Persistence `[Shipped]`

Full site redesign with better UI, save/load (cloud + local), password-based access (no login), rich export, and improved editor with inline errors and advanced search.

- **Cloud + Local Save:** Save code to cloud with a password. Download as .gl files for local. Upload to resume. Both worlds.
- **Password-Based Access:** No accounts, no email. Enter a password to save/retrieve. Student rosters saved separately.
- **Rich Export:** CSV, JSON, PDF export. Copy to clipboard. Print-ready formatting.
- **Better Editor:** Improved highlighting, inline errors, advanced search, resizable panels.

### v3.5 — Advanced Language + Multi-Class `[Shipped]`

Score-based grouping, role tags, weighted preferences, separate/together directives. Class codes for managing 8+ classes per teacher. Enhanced roster with CSV import. Tier selector in the editor.

`score=N` · `role="type"` · `+Want@N` · `separate()` · `together()` · `balance` · `class codes`

### v10.0 — Advanced Build `[Shipped]`

The current generation. Full Advanced language with directives, custom rules, role definitions, multi-round assignments, and comprehensive file I/O. The engine now supports 4 overflow modes, hard guarantee post-processing, and the interactive chip highlighting system.

`@group_size` · `@balance` · `@overflow` · `@rounds` · `@separate` · `@together` · `@no_repeat_partners` · `@rule` · `@role` · `@strict`

- **4 Overflow Modes:** absorb (no solos), even, newgroup, small — configurable via buttons or `@overflow` directive
- **Multi-Round Assignments:** `@rounds` with mixed/similar/random strategies, `@no_repeat_partners` across rounds
- **Score-Based Grouping:** `@balance = scores` with `@band_width` for score proximity
- **Custom Role Definitions:** `@role` with color, icon, and overflow behavior (auto/always/never)
- **Custom @rule Blocks:** Conditional logic with conditions and actions
- **Editor Overhaul:** Resizable panels, drag-and-drop roster, inline errors, advanced search
- **File I/O & History:** .gls/.gla/.glu export/import, version history with undo

### v10.1 — Current Release `[Active]`

Bug fixes and overflow overhaul. The absorb mode ensures no student is left in a solo group during pair work. The engine crash and chip highlighting issues are fixed. The online IDE is available with bug reporting.

- **Absorb Overflow Mode:** No solo groups for pair work — extra students absorbed into existing groups (e.g., 7/2 → [3,2,2])
- **Engine Crash Fix:** Fixed null reference error when reading group size input
- **Chip Highlighting Fix:** Fixed hover-to-inspect using data-name attribute for reliable name matching
- **Hard Guarantee Enforcement:** Post-processing after every optimization pass eliminates empty groups
- **Online IDE:** Available at sites.google.com/grouplogic-ide with `/bug-report` for issue submission

### v4.0 — GroupLogic.py + Code-Driven Classroom `[Future]`

Python tier with full library API. Code-driven classroom layout: write your room in code, engine renders the preview with hover-to-inspect. Multi-round assignments. The three-tier editor is the signature UI.

- **GroupLogic.py:** Python + GL library. CSV import, custom algorithms, room layouts.
- **Code-Driven Room:** Code tables and seats, see generated preview. Hover for seatmate info.
- **Multi-Round:** Week-by-week with no repeat pairings. History tracking.

### v5.0 — Open Source Launch + Collaborative Platform `[Long-Term]`

GroupLogic IDE goes open source on GitHub. Community contributors can build new language features, optimization algorithms, export formats, and classroom templates. On the platform side, participants submit preferences privately, facilitators define global constraints, and the engine generates groupings respecting everyone's input. Built-in documentation and learning guides for all three language tiers.

- **GitHub Open Source:** Public repo. Contribution guidelines. Issue tracker. Pull requests welcome. Community-driven feature development.
- **Built-in Docs:** Interactive guides embedded in the IDE. Context-aware help. Tutorials for each language tier. Searchable reference.
- **Private Preferences:** Students submit preferences via form. Teacher sees aggregated constraints, not individual submissions.
- **Plugin System:** Community-built plugins: custom export formats, new constraint types, integration with LMS platforms.

### v6.0 — AI-Assisted Grouping + LMS Integration `[Long-Term]`

AI suggests optimal grouping strategies based on historical data. Integration with Google Classroom, Canvas, and other LMS platforms for automatic roster sync. The engine learns from past outcomes — which group compositions led to better project results — and adjusts its scoring accordingly. Smart defaults that get smarter over time.

- **AI Grouping Suggestions:** Based on historical outcomes. "Last semester, groups with this composition scored 15% higher."
- **LMS Integration:** Auto-sync rosters from Google Classroom, Canvas, Schoology. Push group assignments back to the LMS.
- **Outcome Tracking:** Rate group outcomes. Engine learns which compositions work. Scoring weights adapt over time.
- **Smart Defaults:** New teacher? AI suggests starting constraints based on class size, subject, and grade level.

### v7.0 — Ecosystem + Mobile `[Long-Term]`

The long-term vision: GroupLogic as an ecosystem. Mobile app for on-the-go grouping. Template marketplace where teachers share and sell grouping templates. Multi-school district support with centralized constraint policies. Research partnerships with education departments studying optimal group composition. The language becomes a standard, like SQL for databases, but for people grouping.

---

## Feature Status

Every feature tracked with a live status. Status labels: `Shipped` · `In Progress` · `Planned` · `Coming Soon` · `Future` · `Deprecated` · `Cancelled`

### Core Engine

| Feature | Status |
|---------|--------|
| Monte Carlo Stochastic Search (2,500 iterations) | ✅ Shipped |
| Hill Climbing Refinement (20 rounds) | ✅ Shipped |
| Weighted scoring function (200:1 avoid:want ratio) | ✅ Shipped |
| Score-band proximity scoring | ✅ Shipped |
| Role coverage constraint solver | ✅ Shipped |
| Multi-round assignment (no repeat pairings) | ✅ Shipped |
| Overflow modes: absorb, even, newgroup, small | ✅ Shipped |
| Custom @rule blocks with conditions and actions | ✅ Shipped |
| Hard guarantee post-processing (no empty groups) | ✅ Shipped |
| Proximity-aware room constraint solver | 🔮 Future |
| AI-assisted grouping strategy suggestions | 🔮 Future |

### Language

| Feature | Status |
|---------|--------|
| Simple tier: +want / -avoid syntax | ✅ Shipped |
| Comments (// prefix) | ✅ Shipped |
| Advanced tier: score=, role=, +Want@N | ✅ Shipped |
| Advanced tier: @separate, @together, @balance | ✅ Shipped |
| Advanced tier: @overflow with absorb mode | ✅ Shipped |
| Advanced tier: @rounds with strategy selectors | ✅ Shipped |
| Advanced tier: @role with color, icon, overflow | ✅ Shipped |
| Advanced tier: @rule custom blocks | ✅ Shipped |
| Advanced tier: room/table/near/far syntax | 📋 Planned |
| GroupLogic.py: Python + GL library | 🔮 Future |
| GroupLogic.py: CSV/JSON import, custom export | 🔮 Future |
| Tier switcher UI (Simple / Advanced / Python) | ✅ Shipped |

### Editor & UI

| Feature | Status |
|---------|--------|
| Live syntax highlighting | ✅ Shipped |
| Conflict glow (pulsing red border) | ✅ Shipped |
| Hover-to-inspect relationship visualization | ✅ Shipped |
| Chip hover highlighting with data-name attribute | ✅ Shipped |
| Better highlighting, inline errors, advanced search | ✅ Shipped |
| UI overhaul (resizable panels, drag-and-drop) | ✅ Shipped |
| Multi-classroom tab management with rename | ✅ Shipped |
| .gls/.gla/.glu file export and import | ✅ Shipped |
| Version history with undo | ✅ Shipped |
| Code-driven classroom layout with hover preview | 📋 Planned |
| Free-form groups (not rule-based) | 📋 Planned |
| Built-in documentation & learning guides | ✅ Shipped |

### Save & Persistence

| Feature | Status |
|---------|--------|
| Copy results to clipboard | ✅ Shipped |
| Cloud save/load with password (no login) | 🔧 In Progress |
| Download/upload .gl files (local backup) | ✅ Shipped |
| Multiple export formats (.json, .gls, .gla, .glu) | ✅ Shipped |
| Export as JSON | ✅ Shipped |
| Save student rosters separately | 📋 Planned |
| Export as CSV, PDF | 📋 Planned |
| Connect codes to cross-reference saved data | 📋 Planned |

### Class Management

| Feature | Status |
|---------|--------|
| Single class support | ✅ Shipped |
| Multi-classroom tab management | ✅ Shipped |
| Drag-and-drop roster with rename and delete | ✅ Shipped |
| Manage 8+ classes per teacher | ✅ Shipped |
| Enhanced roster (add/remove without editing code) | ✅ Shipped |
| Generate unique class codes | 📋 Planned |
| Import student lists from CSV | 📋 Planned |

### Open Source & Community

| Feature | Status |
|---------|--------|
| GitHub public repository | 🔮 Future |
| Contribution guidelines & issue templates | 🔮 Future |
| Plugin system for community extensions | 🔮 Future |
| Template marketplace (share/sell grouping templates) | 🔮 Future |
| LMS integration (Google Classroom, Canvas) | 🔮 Future |

---

## Vision

GroupLogic IDE is not just a group randomizer with extra steps. It's the beginning of a new category: *constraint-aware social orchestration tools*. The vision is a world where group assignment is never random, always intentional, and fully programmable — from a 5th grade classroom to a Fortune 500 boardroom.

### The Problem with "Random"

When a teacher says "I'll just assign groups randomly," what they're really saying is "I don't have the information or the tools to make a better decision." That's not their fault — the tools don't exist. Random assignment persists not because it's good, but because the alternative (manual assignment by a human) is worse: it's slower, more biased, and doesn't scale.

GroupLogic IDE fills the gap between these two extremes. It gives the facilitator a language to express what they know about the social dynamics, test scores, skill levels, and the engine respects those constraints automatically. The result is group assignment that is simultaneously faster than manual, more informed than random, and completely auditable by anyone who reads the rules.

The deeper insight is that group assignment is a *constraint satisfaction problem*, not a sorting problem. Once you frame it that way, a whole field of computer science opens up: constraint programming, SAT solvers, integer linear programming, Monte Carlo methods. GroupLogic IDE starts with the simplest viable approach and evolves toward more powerful solvers as the language grows.

### The Three-Tier Philosophy

The tier system isn't just about adding features — it's about matching complexity to the user's needs. A first-year teacher who just wants to avoid putting ex-friends in the same group shouldn't need to learn about score bands and role balancing. But a veteran teacher who's been manually grouping by test scores for 15 years shouldn't be limited to simple preferences either. The three tiers ensure that GroupLogic IDE is always the right tool, regardless of how simple or complex the grouping task is.

**Simple** is for the 80% case: basic social constraints, fast setup, instant results. **Advanced** is for the teacher who has data and wants to use it: test scores, roles, weighted preferences, multi-person constraints, overflow modes, multi-round assignments. **GroupLogic.py** is for the power user who needs expressiveness beyond any DSL: custom algorithms, data integration, multi-round scheduling, and anything else Python can do.

The key design principle is that all three tiers produce the same output: a grouping with a score. The tiers differ only in what you can *express*, not in the quality of the result. A well-specified Simple program will produce the same quality of grouping as an equivalent Advanced program — it just can't express as many kinds of constraints.

### Beyond the Classroom

While GroupLogic IDE is designed with teachers in mind, the core problem — optimal group assignment under constraints — is universal. Hackathon organizers need to form balanced teams. Project managers need to staff cross-functional squads. Event planners need to seat dinner guests without drama. Conference organizers need to create breakout discussion groups. In every case, the same engine applies: express constraints, optimize, output groups.

The language might evolve domain-specific vocabulary — `seat_near` for events, `skill_balance` for hackathons, `department_separate` for corporate — but the underlying optimization is the same. GroupLogic IDE's long-term vision is to be the standard tool for *any* group assignment problem, with the language growing to cover each domain naturally.

### The Open Source Bet

GroupLogic IDE's most ambitious feature — the one that no proprietary tool can replicate — is an open language and engine. When the language is open, teachers can share rule templates. Researchers can publish new optimization strategies. Developers can build integrations with any platform. The community becomes the product's immune system: bugs get found faster, edge cases get covered, and the language evolves based on real classroom needs rather than a product team's assumptions.

This is why v5.0 marks the transition to open source. The project needs contributors who use it in real classrooms, encounter real edge cases, and build real integrations. The three-tier language design makes contribution accessible: you don't need to understand the optimization engine to add a new Advanced syntax feature, and you don't need to understand the DSL parser to add a new Python library function. The architecture is designed for contribution.

---

## Bug Report

Found a bug? Have a feature request? We want to hear from you.

### Online Bug Report

Visit **[sites.google.com/grouplogic-ide/bug-report](https://sites.google.com/grouplogic-ide/bug-report)** to submit a bug report or feature request. This loads a Google Form where you can describe the issue, include screenshots, and tell us what version you're using.

### Direct Form Link

You can also access the bug report form directly:

[Open Bug Report Form](https://docs.google.com/forms/d/e/1FAIpQLSdKglJljVp6cpDZEDtRTAai6rahEy36L-XXA1__bV7EqT6T3Q/viewform?usp=header)

### What to Include

When reporting a bug, please include:

- **Version number** (shown in the IDE header, currently v10.1)
- **What you typed** in the editor (the code that triggered the bug)
- **What happened** vs. what you expected
- **Browser** and device info (Chrome, Safari, mobile, etc.)
- **Screenshots** if possible

### Quick Tips

- If you see "Engine Error" in the output, try simplifying your code and re-running
- If the output looks wrong, check that your `@overflow` mode is set correctly — `absorb` prevents solo groups
- Use the **Shuffle** button to regenerate groups with the same rules

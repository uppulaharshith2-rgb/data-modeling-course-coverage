# Data Modeling Course — Concept Coverage Map

A learner who finishes the data-modeling course should be able to solve **all 16 live problems (Q461–Q477)** on the platform. This is the backtracked concept list — derived by designing the correct model for each problem — cross-checked against the existing course material (the *Data Modeling Video Script* + the Long-vs-Wide / Partitioning / Relationships / Best-Practices docs + the YouTube / DoorDash worked examples).

> **Legend:** ✅ already taught · ⚠️ partial · ❌ gap to add · ⭐ highest-priority add

---

## TL;DR — the 6 videos to add

The course foundation is solid (roughly half the required concepts are already taught). The six highest-impact gaps — each required by **≥11 of the 16 problems** and not yet taught — are:

1. **Additivity** (additive / semi-additive / non-additive)
2. **SCD Type 1 / 2 / 3** (explicit strategies)
3. **NULL vs absence** semantics
4. **Role-playing dimension**
5. **Degenerate dimension**
6. **Hierarchies** (fixed / ragged / recursive + closure tables)

If only six videos get added, add these.

---

## 1. Coverage vs the existing course

| Concept | In problems | Course status | Action |
|---|:---:|---|---|
| Entities & attributes | foundational | ✅ Script §1 | keep |
| OLTP vs OLAP, normalization vs denormalization | foundational | ✅ Script §2 | keep |
| Star vs Snowflake (vs Galaxy) | 9/16 | ✅ Script §2 | keep |
| Fact / Dim / Agg + grain + surrogate key | 15/16 | ✅ Script §3 | keep |
| Long vs Wide (event-level vs regular fact) | 15/16 | ✅ Script §4 | keep |
| Partitioning (dim snapshot, fct incremental) | 15/16 | ✅ Script §5 | keep |
| Relationships 1:1 / 1:M / M:N + bridge table | 11/16 | ✅ Script §6 | keep |
| Keys (PK/FK, surrogate, composite) | 13/16 | ✅ Script §3/§6 | keep |
| Grain + "which table does this column go in" | 15/16 | ✅ Script §7 | keep |
| Best practices (clarifying Qs, process-first, naming) | all | ✅ Script §8 | keep |
| Pre-aggregation for scale / why scans fail at billions | 15/16 | ✅ §5 + agg (scale angle partial) | reinforce |
| **Aggregate / cube tables at multiple agg-levels** | 14/16 | ⚠️ taught single-level only (§3) | **extend** |
| **Additivity** (additive / semi-additive / non-additive) | 16/16 | ❌ not taught | **ADD ⭐** |
| **SCD Type 1 / 2 / 3** (explicit strategies) | 16/16 | ⚠️ only referenced inside partitioning §5 | **ADD ⭐** |
| **NULL vs absence** semantics | 15/16 | ❌ not taught | **ADD ⭐** |
| **Role-playing dimension** | 12/16 | ❌ not taught | **ADD ⭐** |
| **Degenerate dimension** | 12/16 | ❌ not taught | **ADD ⭐** |
| **Hierarchies** (fixed / ragged / recursive, closure tables) | 11/16 | ❌ not taught | **ADD ⭐** |
| Accumulating-snapshot fact + funnel / conversion | 7/16 | ⚠️ DoorDash wide table is implicitly one, never named | **ADD** |
| Cumulative table design (last_active_date / activity arrays) | 6/16 | ❌ not taught | **ADD** |
| State-transition log vs terminal-status column | within 15/16 | ❌ not taught | **ADD** |
| Symmetric / mutual-relationship dedup | 5/16 | ❌ not taught | **ADD** |
| Write-time vs read-time computation | 9/16 | ⚠️ implied by "detail in the fct" best-practice | **make explicit** |
| Semi-structured vs typed columns (JSON / MAP / ARRAY) | 3/16 | ⚠️ examples use ARRAY/MAP, never framed as a tradeoff | **make explicit** |
| Schema evolution / additive growth | 3/16 | ❌ not taught | **ADD** |
| Factless fact | 2/16 | ❌ not taught | **ADD (minor)** |

---

## 2. Full concept catalog

Format: **Concept** — *what it is* → example. `[weight /16]`

### Module 1 — Foundations  ✅ *mostly covered*

- **Entities & attributes** — pick the real-world nouns and their properties before drawing any table. → *Customer, Product, Order, Seller; Customer has age, city, signup_date.* `[✅ §1]`
- **OLTP vs OLAP, normalization vs denormalization** — transactional sources are normalized; analytics warehouses are denormalized for fast reads. → *app `orders` is 3NF; warehouse `fct_orders` flattens product/customer attributes in.* `[✅ §2]`
- **Star vs Snowflake (vs Galaxy)** — one central fact + denormalized dims (star); snowflake branches dims into sub-dims; galaxy = multiple facts sharing conformed dims. → *Prefer star: `fct_watch` + `dim_user`, `dim_content`, `dim_date`.* `[✅ §2, 9/16]`
- **Fact vs Dimension vs Aggregate** — facts hold measures + FKs at a grain; dims hold descriptive context; aggregates pre-roll to a coarser grain. → *`fct_order_item` vs `dim_product` vs `agg_daily_store_sales`.* `[✅ §3, 15/16]`
- **Keys: surrogate vs natural, composite, FK integrity** — facts carry surrogate FKs to dims; composite key when one column isn't unique. → *`dim_user.user_key` (surrogate) vs source `user_id` (natural); bridge key = (content_id, genre_id).* `[✅ §3/§6, 13/16]`

### Module 2 — Fact Table Design  *partially covered*

- **Long vs Wide (event-level vs regular fact)** — long = one row per atomic event with action_type/value (flexible); wide = one row per transaction with many typed columns (fast to query). → *YouTube `fct_event` (per view/like/comment) vs DoorDash `fct_order_delivery` (per order, all `_ts` columns).* `[✅ §4, 15/16]`
- **Transaction fact** — one row per business event; the default fact. → *one row per ride, per order line, per click.* `[✅ "wide/regular" fact, 13/16]`
- **Periodic-snapshot fact** — one row per entity per fixed period, to measure a level that exists at points in time. → *`fct_inventory_daily`; "guests staying on night X" = `fct_guest_night`.* `[⚠️ snapshot partitioning taught for dims, periodic-snapshot FACT not named — 11/16]`
- **Accumulating-snapshot fact** — one row per process instance whose milestone-timestamp columns fill in as it advances; the tool for "time between stages" and "where did it stall." → *one row per hiring candidate with applied_ts / screened_ts / onsite_ts / offer_ts; NULL = not reached yet.* `[❌ ADD — DoorDash wide table is implicitly this but unnamed — 5/16]`
- **Factless fact** — records that an event *happened* with no numeric measure; the metric is a row count. → *`fct_feed_impression` (a video was shown) — metric = "how many impressions."* `[❌ ADD (minor) — 2/16]`
- **⭐ Aggregate / cube tables at multiple agg-levels** — pre-rolled summaries at several grains (daily/weekly/monthly × store/region/country) so dashboards read a tiny table; the OLAP "cube/lattice" of rollups. → *`agg_store_daily`, `agg_region_weekly`, `agg_country_monthly` from one `fct_order`; pick the agg-levels dashboards actually query.* `[⚠️ EXTEND — taught single-level in §3 — 14/16]`
- **⭐ Additivity (additive / semi-additive / non-additive)** — the rule that decides what an agg/cube table may store. **Additive** = SUM freely (revenue, units). **Semi-additive** = SUM across entities but NOT across time; take last/avg over time (inventory on-hand, account balance, cumulative totals). **Non-additive** = never SUM or AVG (rates, %, ratios, surge multipliers) — store raw numerator & denominator and divide the SUMs. → *Never `AVG(engagement_rate)` across videos — `SUM(interactions)/SUM(views)`; never `SUM(daily_inventory)` over a month.* `[❌ ADD ⭐ — 16/16, the most-violated rule]`

### Module 3 — Dimension Design  *SCD is the big gap*

- **⭐ Slowly Changing Dimensions — Type 1 / 2 / 3** — how a dimension records attribute changes over time:
  - **Type 1** — overwrite, keep only the current value (no history). → *fix a misspelled city.*
  - **Type 2** — new row per change with `effective_start` / `effective_end` / `is_current`; the fact points to the version true at event time (point-in-time correctness). → *user upgrades Free→Pro; a watch from last month must attribute to "Free."*
  - **Type 3** — keep a "previous value" column (limited history). → *`current_region`, `prior_region`.*
  - **Snapshotting & partitioning as the implementation** — a daily full **snapshot partition** of the dim IS a cheap Type-2: each day's partition holds the then-current rows, so "as-of" = scan that date's partition; the latest partition = current truth. **Dim = snapshot partitioning; Fact = incremental partitioning.** → *`dim_user` partitioned by date; `WHERE date='2026-01-10'` gives Jan-10 attributes.* `[❌ ADD ⭐ — §5 references SCD but Types 1/2/3 never taught explicitly — 16/16]`
- **⭐ Role-playing dimension** — one physical dim joined multiple times under different roles (alias each time). → *`dim_date` as order_date AND ship_date; `dim_member` as requester AND addressee; booker vs guest.* `[❌ ADD — 12/16]`
- **⭐ Degenerate dimension** — a dimension *key* that lives on the fact with no separate dim table (just an ID used for grouping). → *`order_id` / `session_id` on the fact — no `dim_order` needed.* `[❌ ADD — 12/16]`
- **⭐ Hierarchies (fixed / ragged / recursive)** — modeling parent→child levels and rolling up across them. Fixed = flatten to columns (L1/L2/L3); recursive/ragged (arbitrary depth) = a parent_id self-reference + a **closure/path bridge** table so you roll up without recursive SQL. → *Electronics→Phones→Smartphones; folder→subfolder→file; "events in folder X and all descendants."* `[❌ ADD — 11/16]`
- **Bridge table for many-to-many** — an associative table (optionally with allocation weights) for M:N relationships, so you don't double-count. → *content↔genre, file↔group-member, driver↔registered-vehicles; class↔category with weights so a 2-category session counts once.* `[✅ §6 (M:N); allocation weights not covered — 11/16]`

### Module 4 — Behavior, State & Time  *mostly gaps*

- **State-transition log vs terminal-status column** — to analyze *how* something reached its end state (per-stage dropout, time-in-stage), log every state change as rows; a single `final_status` column can't answer it. → *ride-hailing: `fct_trip_status_event` (requested→matched→arrived→completed) vs just `trip_status='completed'`.* `[❌ ADD — within the 15/16 "behavior" surface]`
- **Funnel / conversion modeling** — multi-step conversion via an accumulating snapshot or stage log; per-step timing, abandonment, first-attempt vs retry. → *chat-assistant step conversion; checkout funnel; interview pipeline.* `[❌ ADD — 7/16]`
- **⭐ NULL vs absence** — "hasn't happened" is sometimes a NULL in a fact column (process reached here, not further) and sometimes a *missing row* (never entered) — they answer different questions. → *candidate not yet contacted = NULL `contacted_ts`; unfulfilled ride demand never becomes a trip row, so it needs its OWN source table.* `[❌ ADD ⭐ — 15/16]`
- **Symmetric / mutual-relationship dedup** — for undirected relationships, storing both (A,B) and (B,A) double-counts; store one canonical row (low<high) and enforce it at write time. → *LinkedIn connections: one row for the A–B link, not two.* `[❌ ADD — 5/16]`

### Module 5 — Scale & Performance  *partitioning covered; the rest are gaps*

- **Pre-aggregation for scale / why full scans fail** — at billions of rows you can't scan raw events per dashboard hit; serve from agg/rollup tables + partition pruning. → *Daily-Active-Accounts off `agg_account_daily`, not a 50B-row event scan.* `[✅ §5 + agg; reinforce scale framing — 15/16]`
- **Cumulative table design** — carry per-entity state forward each day (`last_active_date`, an activity-history array) so reactivation / streaks / running totals become a cheap filter, not a window over all history. → *"inactive 30 days then returned" = `WHERE date - last_active_date >= 30`.* `[❌ ADD — DE classic — 6/16]`
- **Denormalization** — copy attributes onto the fact to avoid read-time joins. → *freeze `unit_price` and `category_name` onto `fct_order_item`.* `[✅ §2 + "detail in fct" — 8/16]`
- **Write-time vs read-time computation** — resolve the SCD key, health flags, "is-active" rule, SLA deadline at ingestion, not per query. → *stamp `user_tier_key` onto the watch event when it lands.* `[⚠️ make explicit — implied by §8 — 9/16]`
- **Semi-structured vs typed columns** — heterogeneous per-event sub-fields: JSON/MAP/ARRAY blob (flexible) vs promoted typed columns (fast, indexable). → *AI-chat sub-fields (prompt_len, token_count) as a MAP vs columns; YouTube `channel_id ARRAY`.* `[⚠️ make explicit — 3/16]`
- **Schema evolution / additive growth** — design so a new content type / stage inserts rows or capability-flagged dim entries without a migration. → *a feed content type growing 5%→50% shouldn't force an `ALTER`.* `[❌ ADD — 3/16]`

### Module 0 — Interview delivery  ✅ *covered — keep*

- **Best practices** — start with clarifying questions, narrate business process → fact table, naming (`fct_`/`dim_`/`agg_`), PK/FK on every table, `date (partition)` last, rich detail in the fact. `[✅ §8]`
- **Defining grain + follow-up handling** — declare "one row per ___" first; for a new requirement, decide which existing table the column joins to vs when a new table is needed. `[✅ §7, 15/16]`

---

## 3. Suggested teaching order

Keep the existing §1–§8 spine; insert the gap videos where they fit:

1. Foundations (entities, OLTP/OLAP, star/snowflake) — *already recorded*
2. Fact / Dim / Agg → **+ multi-level agg/cube + Additivity** ⭐
3. Long vs Wide — *already recorded* → **+ accumulating-snapshot & funnel modeling**
4. Dimension design → **+ SCD 1/2/3** ⭐ (build on the Partitioning video) **+ role-playing + degenerate + hierarchies**
5. Partitioning & Relationships — *already recorded*
6. **+ Behavior & state: state-transition log, NULL-vs-absence ⭐, symmetric dedup**
7. **+ Scale layer: cumulative tables, write-vs-read-time, semi-structured, schema evolution**
8. Grain & follow-ups + Best practices — *already recorded*

**Capstones:** Q464 (E-Commerce) touches every module; Q472 (Cloud DAA) is the scale-layer capstone (event → daily snapshot → cumulative).

---

## 4. Best worked example per concept

The course already uses YouTube + DoorDash; the platform problems extend them.

| Concept | Teaching example |
|---|---|
| Long vs Wide | YouTube `fct_event` (long) vs DoorDash `fct_order_delivery` (wide) — *existing* |
| Grain | Q469 guest-night vs booking; Q461 trip vs leg |
| Additivity (non-additive) | Q474 divide-SUMs not AVG(%); Q465 can't SUM availability across sellers |
| Additivity (semi-additive) | Q477 cumulative totals — SUM across cities, take-last over time |
| Agg/cube multi-level | Q463 daily→7-day rollups; Q472 DAA |
| SCD 1/2/3 | Q462 tier-at-watch (Type 2); Q473 price-at-order |
| Accumulating snapshot / funnel | Q476 hiring SLA; Q466 booking-stage dropout; Q471 chat funnel |
| Periodic snapshot | Q465 inventory; Q469 nightly occupancy |
| Role-playing dim | Q475 member aliased twice; Q469 booker vs guest |
| Degenerate dim | Q464 order_id on the line fact |
| Hierarchy (closure) | Q468 folder subfolders; Q464 category L1–L3 |
| Bridge M:N (+ weights) | Q463 class↔category; Q467 multi-creator |
| NULL-vs-absence | Q476 candidate not contacted; Q466 unfulfilled demand as its own table |
| Symmetric dedup | Q475 mutual connections (low<high) |
| Cumulative table | Q472 last_active_date reactivation; Q477 running totals |
| Write-time vs read-time | Q462 stamp SCD key at ingestion; Q465 freeze health flags |
| Semi-structured | Q471 typed columns vs JSON/MAP |
| Schema evolution | Q474 new content type 5%→50% |

---

## Appendix — Coverage matrix (concept × problem)

`●` = the problem exercises the concept.

| Concept (weight) | 461 | 462 | 463 | 464 | 465 | 466 | 467 | 468 | 469 | 471 | 472 | 473 | 474 | 475 | 476 | 477 |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Additivity (16) | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| SCD 1/2/3 (16) | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Grain (15) | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ● | ● |
| Dim design (15) |  | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Long-vs-Wide / fact behavior (15) | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ● |  | ● | ● | ● | ● |
| NULL-vs-absence (15) | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |  |
| Scale/perf (15) |  | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Agg/cube + pre-agg (14) |  | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ● |
| Transaction fact (13) | ● | ● |  | ● |  | ● | ● | ● | ● | ● | ● | ● | ● | ● |  | ● |
| Keys (13) | ● |  |  | ● | ● |  | ● | ● | ● | ● | ● | ● | ● | ● | ● | ● |
| Role-playing dim (12) | ● | ● | ● | ● |  | ● | ● |  | ● | ● | ● |  | ● | ● | ● |  |
| Degenerate dim (12) | ● | ● | ● | ● |  | ● | ● | ● | ● |  |  | ● | ● |  | ● | ● |
| Periodic snapshot (11) | ● |  | ● |  | ● |  |  | ● | ● | ● | ● | ● | ● | ● |  | ● |
| Bridge M:N (11) |  | ● | ● | ● | ● | ● | ● | ● | ● |  | ● | ● |  | ● |  |  |
| Hierarchy (11) | ● |  |  | ● | ● | ● |  | ● | ● |  | ● | ● |  | ● | ● | ● |
| Star schema (9) |  |  | ● | ● | ● |  | ● | ● | ● | ● |  |  | ● | ● |  |  |
| Write vs read time (9) |  | ● | ● | ● | ● |  |  | ● |  |  | ● | ● |  | ● | ● |  |
| Denormalization (8) | ● | ● |  |  | ● | ● |  |  |  | ● |  | ● |  | ● |  | ● |
| Funnel/conversion (7) |  |  |  |  |  | ● | ● |  |  | ● |  | ● | ● |  | ● | ● |
| Cumulative table (6) |  |  | ● |  |  |  | ● | ● |  |  | ● |  |  | ● |  | ● |
| Accumulating snapshot (5) |  |  | ● | ● |  | ● |  |  |  | ● |  |  |  |  | ● |  |
| Dedup symmetry (5) |  |  |  |  |  |  |  | ● |  | ● | ● | ● |  | ● |  |  |
| Semi-structured (3) |  |  |  |  |  |  |  |  |  | ● | ● |  | ● |  |  |  |
| Schema evolution (3) |  |  |  |  |  |  |  |  |  |  |  |  | ● |  | ● | ● |
| Factless fact (2) |  |  |  |  |  |  | ● |  | ● |  |  |  |  |  |  |  |

---

*Concept models designed by backtracking all 16 live problems (Q461–Q477). "Script §N" refers to sections of the Data Modeling Video Script.*

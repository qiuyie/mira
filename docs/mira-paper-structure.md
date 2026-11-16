# Mira paper: agreed structure

Target: about 14.5 pages of body, plus tabs.

## Body

1. TL;DR (half a page)
   - Six sentences: the shift, the three bets, the status (prototype, no traffic, production
     readiness mostly undesigned), the cost of running two stacks, the top risk, and
     the headline ask (back greenfield, staffing, by when).
   - One line of reading routes: leadership 1-4; engineers joining 1-7;
     client engineers 5, 6, 7.1, 8, 11 and the wire-contract tab; Search and GDM 1, 5, 9;
     classic owners 1, 4, 5, 10, 11; privacy and launch reviewers 5, 6, 7.5, 8, 9, 11.

2. Motivation (three quarters of a page)
   - The shift from chat box to durable proactive agent, mapped to the go/amap-2027
     capabilities it unlocks.
   - Classic described neutrally: what it does well, that it stays the production path,
     and what the request boundary cannot do.
   - Before/after figure, with a "cast" box (up to eight lines) defining every named thing
     in plain words before it is used anywhere: MapsFE, MaRS, Jetski, GDM, actor and
     Agency v2, the shared actor host and router, Search Actor Storage, Response SDK,
     Marski.

3. The three bets (one page)
   Jetski harness over a bespoke loop; greenfield beside classic; one Agency v2 actor per
   session. Fixed format per bet:
   - The bet. Why. What it costs. Evidence so far (measured, by code reading, or asserted).
   - Rejected: one clause, detail in the ledger tab.
   - Revisit and stop criteria.

4. Asks, risks and readiness at a glance (three quarters of a page; end of the leadership
   path)
   - Asks first, one line each: what leadership is asked to decide and by when; asks of
     Search and GDM (pointer to section 9, which is the single source); what lands on the
     classic team (pointer to section 10).
   - Top risks, ranked, one line each, linking to section 8. Must include that proactive
     delivery is undesigned although it is the motivation, and that cost at Maps scale is
     unmeasured.
   - Gates summary table: gate; owner (or "unowned"); status; staffed or not; blocks
     dogfood or blocks user traffic. A one-line key explains the status words. Section 11
     is the source of truth for each gate; this table summarises it.

5. Architecture overview (one page)
   - One topology figure and the nouns (session, turn, frame, cursor).
   - Ownership table, one row per component: owner; operated by / on call; shared with
     classic or duplicated; inherit / build / diverge; status.
   - All three status scales defined once, with how they relate: build status (built
     in prototype / at HEAD / planned / undesigned); decision status (decided /
     provisional / open); failure handling (handled / worked around / undesigned).

6. A walking example (three quarters of a page)
   One annotated sequence figure with numbered steps, each tagged with the layer that
   handles it (so it indexes section 7) and marked built or not built:
   open; send; agent works; an approval step before a consequential action (not built);
   turn ends; second client steers; connection drops or the app is backgrounded and the
   client resumes from its cursor; no client attached (undesigned); actor dies mid-turn
   (what is lost); actor goes idle.

7. Layer by layer (about six pages), in request-path order:
   7.1 Client contract: clients, wire protocol and the MapsFE relay (one and a half pages).
       - "Client obligations" table: retry, dedup, resume from cursor, input numbering,
         backgrounding; each rule marked decided / provisional / open, with
         a column for who enforces it (client, MapsFE relay or session actor).
       - "Guarantees and failure" is split into device-to-relay and relay-to-actor.
       - "Start here" has one entry per component (clients, wire protocol, relay).
   7.2 Session actor (one page)
   7.3 Agent actor (one page)
   7.4 Tools and rendering (one page)
   7.5 State and storage (one and a half pages; exempt from the page cap).
       Presents the four-tier state model and its commit points once. Includes a data-store
       inventory table: every place user data rests (the four tiers, Search Actor Storage,
       the shared actor host, logs) with contents, owner (Maps or Search), retention,
       wipeout and takeout status, build status.
   Fixed format per layer:
   - What it does: two or three sentences.
   - Change from classic: one line, including shared or duplicated.
   - Guarantees and failure: what it promises on ordering, delivery and durability; what is
     lost when it dies; cross-layer risks that touch it, pointing into section 8.
   - Depends on and operated by: the Search or GDM components it uses, and whose pager.
   - Status by capability.
   - Decided: the choice and the strongest rejected alternative, one line each, three at most.
   - Open: every question that could change the layer (no numeric cap). User approval is a
     mandatory entry in 7.1 and 7.2; iOS transport and delivery with no client attached
     are mandatory entries in 7.1. Undesigned items link to the section 8 list.
   - Start here: code path or entry point, the deeper tab, a contact.

8. What is hard (one page)
   - Failure-mode table: failure; what is lost; how detected; status. Named rows, at
     least: actor dies with a turn in flight; unfenced and unretried cold writes; recovery
     skew across tiers; Search-owned storage or host unavailable; client resumes past a
     lost cursor.
   - "Not yet designed" list: proactive delivery with no client attached; the approval
     message; iOS transport; resident-actor cost at scale; evaluation. This list is
     canonical; section 4 and the per-layer "Open" lines link to it.
   - Privacy is a pointer to 7.5 and section 11, not restated.

9. For Search and GDM (one page; stands alone, since it will be forwarded)
   - One-line legend for the status words and Maps-side names, so the section reads
     without the rest of the paper.
   - Dependency table, one row per component (Agency v2, shared actor host and router,
     Search Actor Storage, Response SDK, Marski): dependency level (config only / extension
     point / workaround / fork); status; what it implies for Search.
   - Asks of Search and GDM: the single, complete list of everything asked of either
     team. It covers the four multi-client gaps, every gate sign-off assigned to Search
     (for example capacity and caching in section 11), and anything else. Same columns for
     every row: the ask; the current workaround; the change or action requested (or "none,
     we carry it"); the gate it is needed by; what Maps does if declined. GDM asks are
     named, or the section says there are none. Sections 4 and 11 point here and add no
     ask that is not in this list.
   - The shared instance, with fixed sub-items: quota and resident-actor cost; isolation
     from other tenants; release-cadence coupling; on-call and escalation boundary
     (this section is authoritative for Search components); privacy (references
     the 7.5 inventory rows for Maps user data in Search-owned storage and says who owns
     its deletion). "Unmeasured" and "undesigned" are acceptable entries; omission is not.

10. For classic platform owners (three quarters of a page)
    - What is shared and what is duplicated, with fixed sub-items: shared MaRS tool
      capacity; blast radius on classic production traffic; on-call and escalation
      boundary. "Unmeasured" is acceptable; omission is not.
    - What work lands on the classic team.
    - Convergence: the trigger, who decides, and what happens if the paths never converge.

11. Path to production (one page)
    - Nine gates: evaluation; experiments; latency; capacity and caching (Search signs off;
      listed in section 9); monitoring and alerting; logging; trust and safety; privacy
      (wipeout, takeout, retention); client (binary size, launch time, battery, transport).
    - Fixed format per gate: current state (say "undesigned" plainly); reuse classic's
      system or build new; owner; exit criterion. This section is the source of truth for
      gates.
    - Two explicit must-be-true lists: before dogfood, and before any user traffic.
      Each names the section 8 problems, the gates and the 7.5 inventory rows that
      must be closed; user approval is placed explicitly in one of them.

## Tabs
- Decision ledger.
- Full open-question list.
- Wire contract: server frame types, client-to-server input frames (viewport context,
  input numbering), cursor semantics, error and close codes; each item marked built or
  provisional.
- Per-layer detail and the dense figures, capped at two pages per layer.

## Rules
- One name per thing.
- Code-font identifiers appear in the ownership table, on a layer's own page, and in the
  Search dependency table; nowhere else.
- Each alternative stated once.
- Each list has one home: asks of Search and GDM in section 9; gates in section 11;
  undesigned items in section 8; user-data stores in 7.5. Other sections summarise and link.
- Anything that could change a reader's mind stays in the body.
- The section is titled "Architecture", not "Proposal".

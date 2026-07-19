## Week 7 — Issue selection

**Issue link:** [Issue 43] (https://github.com/ascherj/pathreview/issues/43)

**Issue title:** Agent session state is not cleared between reviews for the same user

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

`session_store.py` manages a per-user session object with methods to initialize, store, retrieve, and delete cached state. The bug suggests the retrieval path returns cached tool results without checking whether the underlying input (the user's portfolio) has changed since the cache was populated - likely a missing invalidation step, either in store (not overwriting/versioning on update) or in the orchestrator's login or deciding when to reuse vs. re-run tools. 


**Branch name:** fix/43-Agent-session-not-resting

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger
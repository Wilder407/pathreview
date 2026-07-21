## Week 7 — Issue selection

**Issue link:** [Issue 43] (https://github.com/ascherj/pathreview/issues/43)

**Issue title:** Agent session state is not cleared between reviews for the same user

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
[In 3–5 sentences, in your own words: what the issue is (not a copy-paste of
the title), what is currently broken or missing, and what a successful fix
would accomplish. Naming the part of the codebase it affects is helpful context.]

`session_store.py` manages a per-user session object with methods to initialize, store, retrieve, and delete cached state. The bug suggests the retrieval path returns cached tool results without checking whether the underlying input (the user's portfolio) has changed since the cache was populated - likely a missing invalidation step, either in store (not overwriting/versioning on update) or in the orchestrator's login or deciding when to reuse vs. re-run tools. 

The issue attributes tale results to `session_store` caching by user ID. Tracing the code, that mechanism isn't present: the orchestrator always re-runs every tool in the plan and never consults `session_store` to skip execution (orchestrator.py:53-62); the only cache checked is the in-memory `ContextManager`. The loaded session state is merged via `session_state.update(results)` and re-saved, but `run()` returns the freshly computed `results`. so stale state is never actually served. `session_store.delete()` is never called, so no reset ever happens. Since the state isn't use to gate execution, that doesn't cause the described symptom. 
The issue's root casue is misattributed; the fix cannot live purely in `session_store.py`. The genuine staleness risks live elsewhere (the constant `market_analyzer` input at orchestrator.py:130, and the accumulating merge at orchestrator.py:66).

A successful fix ensures that when the same user requests a second review after changing their portfolio, the results reflect the new portfolio rather than any prior session. Concretely, cached state must be invalidated when the input changes either by keying session state to a portfolio content/version has (so a changed portfolio misses the cache) or by resetting the session (`session_store.delete`) at the start of each review. It must also remain safe for the intended case: an identical re-review may reuse the cache, and no orphaned per-tool entries from a previous review should linger in the merged state. 

**Branch name:** fix/43-Agent-session-not-resetting

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger

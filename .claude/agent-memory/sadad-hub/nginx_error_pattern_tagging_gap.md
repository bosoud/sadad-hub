---
name: nginx-error parser doesn't tag which pattern matched
description: A known limitation that blocks rate-limit-specific scenarios. Engine-side fix needed in Sadad before sadad-hub can ship the dependent scenarios.
type: project
---

**The gap:** `parsers/nginx-error.yaml` defines three named regex patterns:
- `client_error` — captures method, path, source_ip, level
- `limit_req` — captures source_ip, level (no method/path — it's a rate-limit warning)
- `upstream_error` — captures source_ip, level

Today the engine does NOT tag the resulting event with which named pattern matched. So a scenario can't filter "rate-limit events only" — there's no `evt.parsed.pattern == 'limit_req'` field to test against.

**What this blocks:** scenarios equivalent to fail2ban's `nginx-limit-req` jail. We could write a regex on `evt.message` looking for the literal "limiting requests" string, but that's brittle.

**Where to fix:** Sadad engine, parser registration. When a regex matches, tag the event with the named pattern. Probably one line in `engine/parsers/registry.go`.

**Impact:** P2 in `bosoud/qas/SPEC/AUDIT.md`. Not blocking Phase 1; should land in Phase 2.

**Workaround until fixed:** for now, the existing `http-flood` scenario covers the gross case (high request volume from one IP) without needing the rate-limit signal specifically.

**How to apply:**
- If a scenario seems to need this, escalate to `@sadad` rather than working around with fragile regex on `evt.message`.
- When the engine tags events, this scenario becomes trivial: `filter: evt.service == 'http' && evt.parsed.pattern == 'limit_req'` + capacity threshold.

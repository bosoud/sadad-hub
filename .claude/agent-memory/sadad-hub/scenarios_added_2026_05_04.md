---
name: 6 fail2ban-equivalent scenarios added 2026-05-04
description: Added nginx-noscript, nginx-bad-request, wordpress-probe, dotfile-probe, bad-bots, xmlrpc-flood. All use existing nginx-access fields.
type: project
---

On 2026-05-04, six new scenarios were added to extend fail2ban-equivalent coverage:

| Scenario | Pattern | Threshold | Ban |
|----------|---------|-----------|-----|
| `nginx-noscript` | 4xx on `.php` / `.aspx` / `.jsp` / `.cgi` / `.pl` / `.sh` | 5 / 60s | 6h |
| `nginx-bad-request` | HTTP 400 (malformed / injection) | 5 / 60s | 1h |
| `wordpress-probe` | `/wp-login.php`, `/wp-admin`, `/xmlrpc.php`, plugin paths | 3 / 60s | 12h |
| `dotfile-probe` | `.env` / `.git/` / `.aws/` / `.ssh/` / `.htaccess` / `.svn/` | 1 hit | 24h |
| `bad-bots` | User-Agent matches sqlmap / nikto / wpscan / nmap / gobuster / hydra / etc. | 1 hit | 24h |
| `xmlrpc-flood` | POST to `/xmlrpc.php` | 10 / 30s | 6h |

**Why these specifically:** chosen as fail2ban equivalents that work with the **existing** `nginx-access` parser fields (no parser changes needed). Coverage gains: equivalents for `apache-noscript`, `apache-badbots`, `nginx-bad-request`; WordPress/XML-RPC patterns commonly added on top of fail2ban defaults.

**Things still missing (not added — would need parser changes):**
- `nginx-limit-req` — would need `nginx-error` parser to tag which named pattern matched (today it doesn't, so a scenario can't filter "rate-limit events only" vs "client-error events"). Tracked as P2 in `bosoud/qas/SPEC/AUDIT.md`.
- `postfix`, `dovecot`, `mysqld-auth` — no parser exists for these log sources.

**How to apply:**
- When adding more fail2ban-equivalent scenarios, prefer ones that use existing parser fields. New parsers are heavier — separate work.
- Severity discipline: 1-hit bans (dotfile-probe, bad-bots) are reserved for unambiguously malicious patterns. Multi-hit thresholds for noisier signals.

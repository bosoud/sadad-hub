# Sadad Hub

Community parsers and scenarios for [Sadad](https://github.com/bosoud/sadad).

## In the Qas Suite

This repo is the content registry for Sadad — parsers and scenarios, no runtime. Edit/test via the Sadad console. For **cross-product views and install/uninstall** of Sadad itself, use the [Qas Console](https://github.com/bosoud/qas-console).

Suite contracts live in [bosoud/qas/SPEC](https://github.com/bosoud/qas/tree/main/SPEC). sadad-hub is `runtime.type: content` in the [Qas manifest schema](https://github.com/bosoud/qas/blob/main/SPEC/qas-manifest.md) — it has no executable, no LAPI, and the `index.json` here serves as the suite-level health endpoint.

## Install

```bash
# Search available items.
sadadctl hub search

# Install a parser or scenario.
sadadctl hub install nginx-access
sadadctl hub install ssh-brute-force

# Reload after installing.
sadadctl parsers reload
sadadctl scenarios reload
```

## Available Parsers

| Name | Description |
|------|-------------|
| `nginx-access` | Nginx combined access log |
| `nginx-error` | Nginx error log (client errors, rate limiting) |
| `sshd` | SSH authentication (failed, invalid user, accepted) |
| `strapi` | Strapi application logs |

## Available Scenarios

| Name | Description | Default Ban |
|------|-------------|-------------|
| `ssh-brute-force` | 5+ failed SSH logins in 60s | 4h |
| `ssh-invalid-user` | 3+ invalid usernames in 30s | 4h |
| `http-probe` | 10+ 404s in 30s | 1h |
| `http-flood` | 100+ requests/sec | 30m |
| `admin-brute-force` | 3+ failed admin logins in 2m | 24h |
| `progressive-ban` | 5+ triggers in 24h (repeat offender) | 7d |

## Contributing

1. Fork this repo
2. Add your YAML file to `parsers/` or `scenarios/`
3. Add an entry to `index.json`
4. Submit a PR

### Parser format

```yaml
name: my-parser
description: "What it parses"
source: my-source
stage: s01-parse
filter: "source == 'my-source'"
patterns:
  pattern_name: 'regex with (?P<source_ip>...) named groups'
statics:
  service: my-service
```

### Scenario format

```yaml
name: my-scenario
description: "What it detects"
type: leaky_bucket    # or: trigger, counter
filter: "evt.service == 'x' && evt.action == 'y'"
group_by: evt.source_ip
capacity: 5
leak_interval: 60s
blackhole: 2m
labels:
  category: brute-force
decision:
  type: ban
  duration: 4h
```

# sub-filter

Free-subscription aggregation with layered liveness testing. Local machines do zero probing.

## Subscription URL (import into Clash/mihomo/v2rayN)

Import-ready config with proxy groups and rules, refreshed every 30 minutes by the L1 workflow:

```
https://raw.githubusercontent.com/GitWay0227/sub-filter/artifacts/clash.yaml
```

Mirror for networks with poor raw.githubusercontent reachability:

```
https://gh-proxy.com/https://raw.githubusercontent.com/GitWay0227/sub-filter/artifacts/clash.yaml
```

- `clash.yaml` contains only L1 liveness-tested nodes, a `Pool` select group and an `Auto` url-test group
- `pool.yaml` on the same branch is the raw deduped node list (L2 machine input, no groups — importing it shows an empty proxy page)
- `candidates.yaml` is the zombie report (dead-node list, not for import)
- nodes here passed a US-vantage liveness scan; the China-retested subset is produced by L2 and not published in this repo

## Layers

- **L1 GitHub Action** (`scan`, hourly at :13/:43, tolerant of cron drift): the runner pulls 20 public aggregation sources directly -> subconverter converts and merges them into clash format -> dedup by server:port:secret -> mihomo batch delay test (40 workers) to scan out zombies -> publishes `candidates.yaml` (zombie report) and `pool.yaml` (the full validated deduped set) to the `artifacts` branch
- **L2 China node** (a China-based VPS, hourly at :25/:55): pulls `pool.yaml` -> mihomo scans the whole pool from a China IP (so routing-optimized nodes the US vantage would miss are kept) -> `filtered.yaml` -> served on an internal network
- **L3 consumers**: local clients subscribe to the L2 internal URL and only receive pre-filtered nodes

## Rationale

- L1 does the heavy cleaning: merge, sanitize, dedup, and a US-vantage zombie report
- L2 owns reachability: a node counts only if it works from a China IP
- Consumers keep their own health checks as a third safety net

## Files

- `sources.txt` source list (clash yaml / base64 / URI lists all work)
- `scripts/scan.sh` L1 main script (runs inside the Action)
- `scripts/test-one.sh` single-node liveness test
- `candidates.yaml` L1 zombie report (auto-generated, do not edit)
- `clash.yaml` import-ready subscription config for human subscribers (auto-generated, do not edit)
- `pool.yaml` full deduped pool, the L2 input (auto-generated, do not edit)
- both artifacts live on the `artifacts` branch (single-commit orphan, rewritten each run)

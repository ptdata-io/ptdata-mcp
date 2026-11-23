# MCP registry metadata 3.3.3 — prepared, not published

Date: 2026-11-23. Neither `mcp-publisher publish` nor a production deploy was executed. Nothing was pushed.

## What changed

`ptdata-mcp` (uncommitted):

- `server.json` version `3.3.2` → `3.3.3`. Name stays `io.ptdata/ptdata`. Description unchanged (92 characters, registry limit 100). Remote-only (`remotes`, no `packages`).
- `repository.url` `https://github.com/ptdata-io/ptdata-mcp`, `source` `github`, `id` `"1438658093"` (GitHub numeric id from `gh api repos/ptdata-io/ptdata-mcp --jq .id`, stored as a string because the 2025-12-11 schema types `repository.id` as string).
- `icons`: `https://api.ptdata.io/icon-512.png`, `image/png`, `512x512`.
- README fundamentals row now says a missing Q4 income or cash-flow figure may be derived, and that EPS and share counts are not. The earnings row already said FY or Q1–Q3 and that a fourth quarter is stored as FY. No price-reaction wording was in this README, so none was removed. The GE FY2017 example table no longer says “today” is $110.97B; the live values are in the section below.

`ptdata-server` (uncommitted):

- `server/mcp_server.py` `SERVER_VERSION` `3.3.2` → `3.3.3`. That value is FastMCP `serverInfo.version`.
- Every input property of the four tools now has a non-empty description via `Annotated[type, "..."]`, which FastMCP turns into `inputSchema.properties[].description`. Required sets are unchanged: fundamentals `ticker`, `as_of_date`, `metrics`; index `index`, `as_of_date`; filings `ticker`, `as_of_date`; earnings `ticker`, `fiscal_year`, `fiscal_period`. Earnings still has no `as_of_date`.
- `tests/test_mcp_server.py` expects version `3.3.3` and a non-empty description on every input property.
- `pyproject.toml` and `/healthz` stay `0.1.0`. That is the API product version. The registry version is only the MCP `serverInfo` version. `requirements.txt` still pins the FastMCP library at 3.3.1.

## Validate

`mcp-publisher` 1.8.16 (commit `a21a32bd7d58e217c8099996bfa19bfabf08e4d2`, built 2026-11-23T13:49:47Z), installed from the official quickstart tarball into `/tmp/mcp-publisher-bin` (not on `PATH`, not used to publish).

From `ptdata-mcp`:

```
/tmp/mcp-publisher-bin/mcp-publisher validate ./server.json
Validating against https://registry.modelcontextprotocol.io...
✅ server.json is valid
```

Schema checked the same day: `https://static.modelcontextprotocol.io/schemas/2025-12-11/server.schema.json`. `repository`, `icons`, and `repository.id` are defined. Icon `src` max length is 255; this URL is 34 characters.

## Live facts re-checked 2026-11-23

Free tier, no API key.

- GE FY2017 as of 2018-02-28: `revenue` `122092000000.0` USD, `net_income` `-5786000000.0`, accession `0000040545-18-000014`, `provenance` `xbrl`.
- The same FY2017 revenue is `118243000000.0` as of 2019-03-01 (accession `0000040545-19-000014`) and `99279000000.0` as of 2026-11-01 (accession `0000040545-20-000009`). The README table used to say “today” was $110.97B from the FY2018 10-K. That figure was not returned. The table now uses the two dates above.
- AAPL FY2024 Q4 as of 2026-11-01: `revenue` `94930000000.0`, `cost_of_revenue` `51051000000.0`, accession `0000320193-24-000123`, `provenance` `derived`, `derivation.method` `fy_minus_q1_q3`.
- SP500 on 2020-12-21: 503 tickers, first three `A`, `AAL`, `AAP`.
- Icon `GET https://api.ptdata.io/icon-512.png`: HTTP 200, `image/png`, 27835 bytes, 512×512, bit depth 8, color type 6 (RGBA).
- Official registry search `io.ptdata/ptdata` still has two versions and no `icons` or `repository` key: 3.3.1 `isLatest` false (published 2026-10-29T14:21:15Z); 3.3.2 `isLatest` true (published 2026-10-29T14:55:13Z).

Tests: `ptdata-server` `.venv/bin/pytest -q tests/test_mcp_server.py` → 7 passed, 1 deprecation warning from OpenTelemetry importlib metadata, 0.73s. The full server suite was not re-run. This change is the MCP tool signatures, the version constant, and that one test.

## Later: publish the registry entry

Do this only after an explicit OK. Namespace `io.ptdata/*` needs domain auth for `ptdata.io`, not `mcp-publisher login github`.

The apex already publishes a `v=MCPv1` TXT with `k=ed25519`. Do not generate a new key and do not replace that TXT. A search of this Mac and of the API host for a publisher pem/hex file did not find one (shell history had no `mcp-publisher` line either). Pass the existing hex private-key file that matches that TXT. `https://ptdata.io/.well-known/mcp-registry-auth` currently returns the marketing HTML page, so HTTP login is not what this host is serving.

```bash
cd /Users/peppe/Projects/ptdata/ptdata-mcp
mcp-publisher login dns --domain ptdata.io --private-key-file <EXISTING_KEY_FILE>
mcp-publisher publish ./server.json
curl -sS "https://registry.modelcontextprotocol.io/v0/servers?search=io.ptdata/ptdata"
```

Expect a 3.3.3 record with `isLatest` true and both `icons` and `repository` present. 3.3.2 should no longer be latest.

The login flag is `--private-key-file` (hex file) or `--private-key` (hex on the command line). Prefer the file so the key does not land in shell history. `MCP_PRIVATE_KEY` also works and is mutually exclusive with those flags. Do not print the key.

## Later: deploy the tool descriptions

mcptry cannot see the new argument descriptions until `ptdata-mcp.service` is restarted. That unit is `python -m server.mcp_server` from `/opt/ptdata/ptdata-server`. The REST unit `ptdata-api` does not serve `/mcp`. Do not restart `ptdata-ingest.timer`. Do not touch `/etc/ptdata/ptdata.env`. The tree on the host is not a git checkout, so copy the one file.

```bash
KEY="$HOME/.ssh/id_ed25519_ptdata_agent"
HOST=root@87.99.158.251
STAMP=$(date -u +%Y-%m-%dT%H%M%SZ)
ssh -i "$KEY" -o IdentitiesOnly=yes "$HOST" \
  "mkdir -p /var/backups/ptdata/mcp-desc-$STAMP && cp -a /opt/ptdata/ptdata-server/server/mcp_server.py /var/backups/ptdata/mcp-desc-$STAMP/"
scp -i "$KEY" -o IdentitiesOnly=yes \
  /Users/peppe/Projects/ptdata/ptdata-server/server/mcp_server.py \
  "$HOST:/tmp/mcp_server.py"
ssh -i "$KEY" -o IdentitiesOnly=yes "$HOST" \
  "install -o ptdata -g ptdata -m 644 /tmp/mcp_server.py /opt/ptdata/ptdata-server/server/mcp_server.py && rm -f /tmp/mcp_server.py && systemctl restart ptdata-mcp && systemctl is-active ptdata-mcp"
```

Rollback is the backup copy and the same restart:

```bash
ssh -i "$KEY" -o IdentitiesOnly=yes "$HOST" \
  "install -o ptdata -g ptdata -m 644 /var/backups/ptdata/mcp-desc-$STAMP/mcp_server.py /opt/ptdata/ptdata-server/server/mcp_server.py && systemctl restart ptdata-mcp"
```

After deploy, a client that lists tools on `https://api.ptdata.io/mcp` should see `serverInfo.version` `3.3.3` and a description on every input property. `/healthz` stays `0.1.0`.

## mcptry claim

Page: https://mcptry.com/claim/io.ptdata/ptdata

1. Sign in yourself. This session cannot complete that login.
2. Copy the exact verification they show (DNS TXT name and value, or a file path plus the file contents).
3. Paste that here. Prefer the file method on `https://ptdata.io/` or `https://api.ptdata.io/`.
4. The file is placed only at the URL they specify. No token or path is invented in advance.
5. You return to mcptry and click verify. This session will not use your browser session and will not click verify, unsubscribe, or other tracking links.

If the proof is a new apex TXT, it is written out for you to add. It must not replace the existing registry `v=MCPv1` TXT.

## Not done

Publish was not run. Production was not changed. Git was not pushed. No secret is in this file or in the diff.

# TOUCH-FIRST Training Protocol Log

## Tool: subfinder v2.6.x
**Date:** 2026-02-27
**Platform:** CachyOS AMD64
**Language:** Go
**Category:** Subdomain Enumeration / Passive Reconnaissance / OSINT
**GitHub:** https://github.com/projectdiscovery/subfinder

---

## 🔷 LAND (Structure Analysis)

### Repository Layout
```
subfinder/
├── cmd/
│   └── subfinder/
│       └── main.go              # Entry point — thin wrapper over runner
├── pkg/
│   ├── runner/
│   │   ├── runner.go            # Core orchestration — initializes sources, fans out
│   │   ├── options.go           # All CLI flags and config structs
│   │   └── enumerate.go         # Main enumeration loop
│   ├── resolve/
│   │   └── resolve.go           # Optional DNS resolution of found subdomains
│   └── passive/
│       ├── sources/             # One file per passive OSINT source
│       │   ├── certspotter/
│       │   ├── crtsh/
│       │   ├── securitytrails/
│       │   ├── shodan/
│       │   ├── virustotal/
│       │   ├── github/
│       │   ├── chaos/
│       │   └── ... (40+ sources)
│       └── agent.go             # Source manager — runs all sources concurrently
├── go.mod
└── .config/
    └── provider-config.yaml     # API keys config (copied to ~/.config/subfinder/)
```

### Key Dependencies
- `projectdiscovery/retryablehttp-go` — HTTP client with retry logic
- `projectdiscovery/fastdialer` — fast DNS resolution
- `miekg/dns` — raw DNS queries for verification
- `projectdiscovery/gologger` — structured logging
- `projectdiscovery/goflags` — flag parsing (shared across PD tools)

### Source Count
40+ passive sources. Each is a standalone Go package implementing a common `Source` interface — making it trivial to add new sources without touching core logic.

---

## 🔷 XRAY (Architecture Analysis)

### Concurrency Model
```
Input (single domain / list file / stdin)
    ↓
Source Manager (agent.go)
    ├── goroutine: crtsh query
    ├── goroutine: securitytrails query
    ├── goroutine: virustotal query
    ├── goroutine: shodan query
    ├── goroutine: github code search
    ├── goroutine: chaos dataset lookup
    └── ... (all sources fire simultaneously)
    ↓
Results channel (deduplicated as they arrive)
    ↓
Optional DNS resolution
    ↓
Output (stdout / file / JSON)
```

All sources run in parallel. Results stream in as sources respond — fast sources (crt.sh) return in seconds, slow sources (Shodan) may take longer. Deduplication happens in real-time on the results channel, not at the end.

### The Source Interface
Every source implements:
```go
type Source interface {
    Run(ctx context.Context, domain string, session *Session) <-chan Result
    Name() string
    IsDefault() bool
    HasRecursiveSupport() bool
    NeedsKey() bool
    AddApiKeys([]string)
}
```
`IsDefault()` controls whether a source runs without being explicitly enabled. `NeedsKey()` flags sources that require an API key — they're silently skipped if no key is configured.

### subfinder vs Findomain — Key Differences

| Aspect | subfinder | Findomain |
|--------|-----------|-----------|
| Language | Go | Rust |
| Sources | 40+ | ~20 |
| Default sources (no keys) | ~15 | ~10 |
| Speed (no keys) | Fast | Fast |
| Speed (with keys) | Very fast | Fast |
| Unique sources | GitHub, Chaos, Fofa, Hunter | Facebook, Telegram webhooks |
| Ecosystem | ProjectDiscovery (httpx, nuclei, naabu) | Standalone |
| Output formats | Plain, JSON | Plain, JSON, CSV |
| Active recon | DNS verify only | Port scan, screenshots |
| Best for | Pure passive enum, PD pipeline | Passive enum + built-in active features |

**Key insight:** subfinder is purpose-built for the ProjectDiscovery pipeline. It's designed to feed directly into httpx, naabu, and nuclei. Findomain does more on its own but doesn't integrate as cleanly with the PD ecosystem.

### The Chaos Dataset
ProjectDiscovery runs [chaos.projectdiscovery.io](https://chaos.projectdiscovery.io) — a public dataset of subdomains from bug bounty programs. subfinder has a `chaos` source that queries this directly. For in-scope bug bounty targets, this alone can return thousands of previously discovered subdomains in milliseconds. No API calls to external services — just a dataset query.

---

## 🔷 FIRE (Execution)

### Simplest Command (Baseline)
```bash
subfinder -d example.com
```
Streams subdomains to stdout as sources respond. Fast — first results in under a second from crt.sh.

### Standard Recon Command
```bash
subfinder -d target.com -o subdomains.txt -v
```
`-v` shows which sources are returning results in real time. Good for understanding which sources are firing and which are returning nothing (often means an API key would help).

### Multiple Domains
```bash
subfinder -dL domains.txt -o subdomains.txt
```
Enumerate a list of domains. All processed in parallel.

### With All Sources Enabled (requires API keys)
```bash
subfinder -d target.com -all -o subdomains.txt
```
`-all` enables every source including ones that need keys. Sources without configured keys are silently skipped.

### JSON Output
```bash
subfinder -d target.com -json -o results.json
```
Each line is valid JSON:
```json
{"host":"api.example.com","input":"example.com","source":"crtsh"}
{"host":"dev.example.com","input":"example.com","source":"securitytrails"}
{"host":"admin.example.com","input":"example.com","source":"chaos"}
```
The `source` field tells you *where* each subdomain came from — useful for understanding your coverage.

### Pipe Directly into httpx
```bash
subfinder -d target.com -silent | httpx -status-code -title -tech-detect -o live.txt
```
`-silent` suppresses the banner — clean subdomain-only stdout, perfect for piping. This is the canonical ProjectDiscovery one-liner.

---

## 🔷 BREAK (Parameter Variation)

### Recursive Enumeration
```bash
subfinder -d target.com -recursive
```
After finding subdomains, subfinder will re-run enumeration on discovered subdomains. `api.example.com` found → also enumerate `*.api.example.com`. Can significantly increase coverage for deep targets. Slower.

### Source Exclusion
```bash
# Skip slow sources for a quick first pass
subfinder -d target.com -es github,shodan
```
GitHub and Shodan are often slow. Exclude them for a fast initial run, add them back for thorough coverage.

### Rate Limiting (for stealth)
```bash
subfinder -d target.com -rate-limit 5
```
Throttle to 5 requests/second across all sources. Useful when you suspect targets monitor for unusual traffic patterns.

### Resolve Only Valid Subdomains
```bash
subfinder -d target.com -nW
```
`-nW` = no wildcard — filters out wildcard DNS responses. Without this, wildcard-enabled domains (where `*.example.com` resolves to the same IP) flood output with false positives.

### Compare subfinder + Findomain Outputs
```bash
subfinder -d target.com -silent -o subfinder.txt
findomain -t target.com -q -o && cat target.com.txt > findomain.txt
sort subfinder.txt findomain.txt | uniq > combined.txt
comm -23 <(sort subfinder.txt) <(sort findomain.txt) > subfinder_unique.txt
comm -13 <(sort subfinder.txt) <(sort findomain.txt) > findomain_unique.txt
```
Running both and comparing outputs is standard practice. Different sources = different coverage. The union of both is your best passive dataset.

---

## 🔷 BRIDGE (Bounty Workflow Integration)

### Where subfinder Fits
```
PASSIVE RECON LAYER
┌─────────────────────────────────────────────────────┐
│  subfinder -d target.com -silent                    │  ← Primary passive enum
│  findomain -t target.com -q                         │  ← Secondary (different sources)
│  cat both | sort -u > all_subs.txt                 │  ← Combine, deduplicate
└─────────────────────────────────────────────────────┘
    ↓
httpx -l all_subs.txt -status-code -title -tech-detect -o live.txt
    ↓
naabu -l live.txt -o ports.txt
    ↓
ffuf (content discovery on interesting hosts)
    ↓
nuclei -l live.txt
```

### API Keys Worth Getting (free tiers available)

| Source | What it adds | Get key at |
|--------|-------------|------------|
| SecurityTrails | DNS history, reverse lookup | securitytrails.com |
| Shodan | IP/banner data, historical records | shodan.io |
| VirusTotal | Passive DNS dataset | virustotal.com |
| GitHub | Subdomains leaked in code/configs | github.com/settings/tokens |
| Chaos | ProjectDiscovery bug bounty dataset | chaos.projectdiscovery.io |
| Censys | Certificate transparency + scanning | censys.io |

**Recommended first keys (free, high value):** GitHub token + Chaos API key. GitHub code search finds subdomains hardcoded in repos — leaked `.env` files, config files, old deployment scripts. Chaos adds the curated bug bounty dataset.

### Config File Setup
```yaml
# ~/.config/subfinder/provider-config.yaml
securitytrails:
  - YOUR_KEY
shodan:
  - YOUR_KEY
github:
  - YOUR_GITHUB_TOKEN
virustotal:
  - YOUR_KEY
chaos:
  - YOUR_KEY
```
Set once, used forever across all subfinder runs.

### Full Passive Recon One-Liner (no keys needed)
```bash
subfinder -d target.com -silent | \
  httpx -silent -status-code -mc 200,403 | \
  tee live.txt
```

### With Both Tools + Dedup + httpx
```bash
{ subfinder -d target.com -silent; findomain -t target.com -q 2>/dev/null; } | \
  sort -u | \
  httpx -silent -status-code -title -tech-detect -o live.txt
```
This is your standard recon opening move. Both tools, deduplicated, piped straight into httpx.

---

## 🔷 READ (Documentation Reference)

### Official Resources
- **README:** https://github.com/projectdiscovery/subfinder/blob/main/README.md
- **ProjectDiscovery docs:** https://docs.projectdiscovery.io/tools/subfinder
- **Chaos dataset:** https://chaos.projectdiscovery.io

### Installation on CachyOS
```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

### Key Flags Reference
```
-d DOMAIN        Single target domain
-dL FILE         Domain list file
-o FILE          Output file
-json            JSON output (one object per line)
-silent          Subdomain-only output (no banner, no logs) — use for piping
-v               Verbose — show which sources are firing
-all             Enable all sources (including key-required)
-es SOURCES      Exclude specific sources (comma-separated)
-recursive       Enumerate discovered subdomains recursively
-nW              No wildcards — filter wildcard DNS responses
-rate-limit N    Max requests per second
-timeout N       Seconds per source before giving up (default 30)
-t N             Concurrency for DNS resolution (default 10)
-config FILE     Use alternate config file
```

---

## Summary

| Metric | Value |
|--------|-------|
| **Tool** | subfinder |
| **Language** | Go |
| **Sources** | 40+ passive |
| **Role in pipeline** | Primary passive subdomain enum — feeds httpx |
| **Ecosystem** | ProjectDiscovery (best integration with httpx, nuclei, naabu) |
| **Key insight** | Use alongside Findomain — different sources = better coverage. Chaos dataset alone justifies setup. |
| **Complexity** | Low to run, high ceiling with API keys + recursive |

### Assignment for Next Session
1. Install: `go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest`
2. Run against a target: `subfinder -d target.com -v -o subfinder.txt`
3. Run Findomain on the same target: `findomain -t target.com -q`
4. Compare outputs — how many unique subdomains did each find that the other missed?
5. Try the combined one-liner: `{ subfinder -d target.com -silent; findomain -t target.com -q 2>/dev/null; } | sort -u | httpx -silent -status-code -title -o live.txt`
6. Set up `~/.config/subfinder/provider-config.yaml` with at least a GitHub token
7. Log findings under "## Field Notes"

---

*Generated by Tool Scout — 2026-02-27*

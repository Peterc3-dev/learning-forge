---
layout: default
title: "Pius NDJSON Output Specification"
date: 2026-04-15
category: cybersecurity
tags: [pius, recon, osint, ndjson, agent-integration, bug-bounty]
---

# Pius NDJSON Output Specification

**Date:** 2026-04-15  
**Target:** `--org "GitLab" --domain gitlab.com --asn AS57787 --mode passive`  
**Runtime:** ~90 seconds per run

---

## What Is Pius?

[Pius](https://github.com/ninoseki/pius) is an OSINT recon tool that discovers an organization's digital footprint — domains, CIDR blocks, GitHub orgs, WHOIS data, and more — from a single organization name. It's the preflight step before subfinder: where subfinder finds subdomains for a single domain, Pius finds *which domains belong to the org in the first place*.

This spec documents Pius's NDJSON output format so you can parse it reliably in your own tooling. All data below comes from a real run against GitLab — no fabricated samples.

---

## 1. Raw Output Sample (first 10 lines, verbatim)

```json
{"Type":"domain","Value":"github.com/GitLab","Source":"github-org","Data":{"confidence":0.35,"github_login":"GitLab","github_name":"GitLab, Inc.","github_url":"https://github.com/GitLab","needs_review":true,"org":"GitLab"}}
{"Type":"domain","Value":"github.com/gitlabhq","Source":"github-org","Data":{"confidence":1,"github_login":"gitlabhq","github_name":"GitLab","github_url":"https://github.com/gitlabhq","needs_review":false,"org":"GitLab"}}
{"Type":"domain","Value":"github.com/runcitadel","Source":"github-org","Data":{"confidence":0.9,"github_login":"runcitadel","github_name":"Citadel [MOVED TO GITLAB]","github_url":"https://github.com/runcitadel","needs_review":false,"org":"GitLab"}}
{"Type":"domain","Value":"GitLab B.V.","Source":"gleif","Data":{"confidence":0.35,"jurisdiction":"NL","legalName":"GitLab B.V.","lei":"529900IEQUVR8R1H2620","needs_review":true,"relationshipType":"name-match"}}
{"Type":"domain","Value":"GITLAB INC.","Source":"gleif","Data":{"confidence":0.35,"jurisdiction":"US-DE","legalName":"GITLAB INC.","lei":"9845004Y40FAC81ED49","needs_review":true,"relationshipType":"name-match"}}
{"Type":"domain","Value":"gitlab.com","Source":"urlscan","Data":{"base_domain":"gitlab.com"}}
{"Type":"domain","Value":"docs.gitlab.com","Source":"urlscan","Data":{"base_domain":"gitlab.com"}}
{"Type":"domain","Value":"about.gitlab.com","Source":"urlscan","Data":{"base_domain":"gitlab.com"}}
{"Type":"domain","Value":"advisories.gitlab.com","Source":"urlscan","Data":{"base_domain":"gitlab.com"}}
{"Type":"preseed","Value":"GitLab Inc.","Source":"whois","Data":{"preseed_title":"GitLab Inc.","preseed_type":"whois+company"}}
```

---

## 2. Record Structure

Every line is a JSON object with four fields:

| Field | Type | Description |
|-------|------|-------------|
| `Type` | string | Record category: `"domain"`, `"cidr"`, or `"preseed"` |
| `Value` | string | The discovered entity — hostname, CIDR, org name, URL path |
| `Source` | string | Plugin that produced this record |
| `Data` | object | Source-specific metadata (shape varies by Source) |

All four fields are always present. No optional top-level fields observed.

---

## 3. Field Inventory

### Unique `Type` values
| Type | Count | Description |
|------|-------|-------------|
| `domain` | 13 | Domains, hostnames, and org names from various sources |
| `preseed` | 5 | WHOIS-derived seed data (company names, emails) |
| `cidr` | 1 | CIDR block from BGP routing tables |

### `Data` field shapes by Source

| Source | Fields | Types |
|--------|--------|-------|
| `whois` | `preseed_title`, `preseed_type` | String, String |
| `gleif` | `confidence`, `jurisdiction`, `legalName`, `lei`, `needs_review`, `relationshipType` | f64, String, String, String, bool, String |
| `github-org` | `confidence`, `github_login`, `github_name`, `github_url`, `needs_review`, `org` | f64, String, String, String, bool, String |
| `wayback` | `base_domain` | String |
| `urlscan` | `base_domain` | String |
| `asn-bgp` | `asn`, `org` | String, String |

---

## 4. Plugin Coverage (GitLab test)

### Plugins that fired and produced output
| Plugin | Records | Types | Notes |
|--------|---------|-------|-------|
| `whois` | 5 | preseed | Internal seeds, not actionable recon |
| `gleif` | 5 | domain | All `needs_review:true`, confidence 0.35 — likely filtered |
| `github-org` | 3 | domain | 1 useful (gitlabhq, confidence 1.0), 2 are URL paths |
| `urlscan` | 4 | domain | Real hostnames: gitlab.com, docs/about/advisories.gitlab.com |
| `wayback` | 2 | domain | gitlab.com, www.gitlab.com |
| `asn-bgp` | 1 | cidr | 91.235.46.0/24 (required `--asn` hint) |

### Plugins that accepted input but produced no output
| Plugin | Why no output |
|--------|---------------|
| `crt-sh` | CT log returned no additional subdomains beyond other plugins |
| `wikidata` | No matching entity with subsidiary data |
| `edgar` | GitLab is Netherlands B.V. parent, no US SEC filings |
| `reverse-rir` | No RIR org handle match |
| `google-dorks` | Captcha blocked |

### Plugins that skipped (no API key / active mode only)
| Plugin | Skip reason |
|--------|------------|
| `passive-dns` | `SECURITYTRAILS_API_KEY` required |
| `reverse-whois` | `VIEWDNS_API_KEY` required |
| `apollo` | `APOLLO_API_KEY` required |
| `shodan` | `SHODAN_API_KEY` required |
| `censys-org` | Active mode + `CENSYS_API_TOKEN` required |
| `favicon-hash` | Active mode + `SHODAN_API_KEY` required |
| `dns-brute` | Active mode only |
| `dns-permutation` | Phase-3 active only |
| `dns-zone-transfer` | Active mode only |
| `doh-enum` | Active mode only |
| `afrinic`–`ripe` | Phase-2 RIR — skipped because `reverse-rir` found no handle |

### No-key passive plugins that work out of the box
`whois`, `gleif`, `github-org`, `wayback`, `urlscan`, `crt-sh`, `wikidata`, `edgar`, `reverse-rir`, `google-dorks`, `asn-bgp` (with `--asn`)

### High-value API keys to add
- `GITHUB_TOKEN` — unlocks `github-org` pagination (30 results without token)
- `SHODAN_API_KEY` — CIDR discovery + `favicon-hash`
- `CENSYS_API_TOKEN` — certificate/host search

---

## 5. Filtering Rules

If you're piping Pius output into subfinder or another domain resolver, apply these filters:

### Rule 1: Drop `preseed` records
`Type == "preseed"` is Pius internal plumbing. Not actionable.

### Rule 2: Drop low-confidence `needs_review`
If `needs_review == true` AND `confidence < 0.5`, it's a weak name match (e.g., GLEIF corporate subsidiaries). They're not hostnames and will pollute your seed list.

**Exception:** `github-org` entries with `needs_review == true` but `confidence >= 0.5` should be reviewed, not auto-dropped.

### Rule 3: Strip URL-pattern `Value` fields
Sources like `github-org` emit `"github.com/GitLab"` — not a hostname. Drop at the subfinder boundary, but log for your report's "Organization recon" section.

### Rule 4: Drop non-hostname `Value` fields
`gleif` emits legal entity names like `"GitLab B.V."` — contains spaces, not a domain. Drop anything that doesn't match a basic hostname pattern.

### Rule 5: CIDR records pass through directly
`Type == "cidr"` → don't feed to subfinder (it resolves domains, not IP ranges). Add to findings/report as `severity::Low` entries.

### Filtering summary
```
PiusRecord where:
  Type != "preseed"
  AND (Type == "domain" OR Type == "cidr")
  AND NOT (needs_review == true AND confidence < 0.5)
  AND NOT Value.contains(' ')
  AND NOT Value.contains('/')

→ Type == "domain": feed Value to subfinder
→ Type == "cidr": add to findings
```

### Actionable hostnames after filtering (GitLab test)
- `gitlab.com` (urlscan + wayback)
- `docs.gitlab.com` (urlscan)
- `about.gitlab.com` (urlscan)
- `advisories.gitlab.com` (urlscan)
- `www.gitlab.com` (wayback)

**5 hostnames** for subfinder enrichment.

---

## 6. Volume Stats

| Metric | Without `--asn` | With `--asn AS57787` |
|--------|----------------|----------------------|
| Total NDJSON lines | 15 | 20 |
| Type: domain | 10 | 13 |
| Type: preseed | 5 | 5 |
| Type: cidr | 0 | 1 |
| needs_review=true | 6 | 6 |
| Actionable hostnames | ~5 | ~5 |
| CIDR blocks | 0 | 1 (91.235.46.0/24) |

---

> ⚠️ **Key Finding: The `--asn` Hint Problem**
>
> Without `--asn`, Pius produces **zero CIDR results**. The CIDR discovery chain requires either:
> 1. An explicit `--asn` hint from the user, OR
> 2. The `reverse-rir` → RIR chain to discover org handles → resolve to CIDRs
>
> For GitLab, the chain broke at `reverse-rir` (no RIR org handle found). This means org-level recon with CIDR discovery is weaker than expected without user-supplied ASN hints.
>
> **Recommendation:** Always pass `--asn` when you know the target's ASNs (common in bug bounty scope pages). Add it as a CLI flag alongside `--org`.

---

## 7. Implementation Example (Rust)

If you're integrating Pius into a Rust tool, here's a working serde structure and parsing approach:

### Serde struct design

```rust
use serde::Deserialize;
use serde_json::Value;

/// Top-level NDJSON record from Pius stdout
#[derive(Debug, Deserialize)]
#[serde(rename_all = "PascalCase")]
pub struct PiusRecord {
    pub r#type: String,       // "domain", "cidr", "preseed"
    pub value: String,        // hostname, CIDR, org name, URL
    pub source: String,       // plugin name
    pub data: Value,          // polymorphic — shape depends on Source
}

impl PiusRecord {
    pub fn confidence(&self) -> Option<f64> {
        self.data.get("confidence").and_then(|v| v.as_f64())
    }

    pub fn needs_review(&self) -> Option<bool> {
        self.data.get("needs_review").and_then(|v| v.as_bool())
    }

    pub fn base_domain(&self) -> Option<&str> {
        self.data.get("base_domain").and_then(|v| v.as_str())
    }

    pub fn asn(&self) -> Option<&str> {
        self.data.get("asn").and_then(|v| v.as_str())
    }
}
```

### Parsing loop

```rust
let stdout = cmd.stdout.take().expect("stdout piped");
let reader = BufReader::new(stdout);
let mut domains: Vec<String> = Vec::new();
let mut cidrs: Vec<String> = Vec::new();

for line in reader.lines() {
    let record: PiusRecord = serde_json::from_str(&line?)?;
    
    if record.r#type == "preseed" { continue; }
    
    if record.r#type == "cidr" {
        cidrs.push(record.value.clone());
        continue;
    }
    
    if record.r#type == "domain" {
        if record.needs_review().unwrap_or(false) 
            && record.confidence().unwrap_or(1.0) < 0.5 { continue; }
        if record.value.contains(' ') { continue; }
        if record.value.contains('/') { continue; }
        domains.push(record.value.clone());
    }
}
```

**Key serde decisions:**
- `r#type` because `type` is a reserved keyword in Rust. `rename_all = "PascalCase"` handles `Type` → `type`.
- `Value` for `Data` — different Sources emit different fields. Don't try per-Source enum deserialization.
- All top-level fields are required.

---

## 8. Open Questions

1. **Multi-ASN support** — Should `--asn` accept comma-separated ASNs? Check if Pius supports `--asn AS1,AS2` or requires multiple invocations.
2. **Empty result handling** — Small orgs may have zero CT log presence. Handle gracefully.
3. **Stderr diagnostics** — Pius emits plugin status to stderr. Parse for reporting or just log for debugging?
4. **Timeout** — ~90s for a small org. Large orgs (e.g., Google) could take much longer. Set a configurable timeout (default 5 min) and continue with partial results.
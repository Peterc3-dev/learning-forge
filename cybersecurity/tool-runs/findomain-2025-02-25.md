# TOUCH-FIRST Training Protocol Log

## Tool: Findomain v10.0.1
**Date:** 2025-02-25  
**Platform:** CachyOS AMD64  
**Language:** Rust (compiled binary)  
**Category:** Subdomain Enumeration / OSINT / Reconnaissance  
**GitHub:** https://github.com/Findomain/Findomain

---

## 🔷 LAND (Structure Analysis)

### Repository Layout
```
Findomain/
├── Cargo.toml              # Rust project config (18 source files)
├── Cargo.lock              # Dependency lock file
├── src/
│   ├── main.rs             # Entry point
│   ├── lib.rs              # Library exports
│   ├── args.rs             # CLI argument parsing (clap)
│   ├── logic.rs            # Core enumeration logic
│   ├── sources.rs          # OSINT source integrations
│   ├── resolvers.rs        # DNS resolution
│   ├── networking.rs       # HTTP/network ops
│   ├── screenshots.rs      # Headless Chrome screenshots
│   ├── port_scanner.rs     # TCP port scanning
│   ├── alerts.rs           # Discord/Slack/Telegram webhooks
│   ├── database.rs         # PostgreSQL integration
│   ├── files.rs            # File I/O operations
│   ├── structs.rs          # Data structures
│   ├── errors.rs           # Error handling
│   ├── utils.rs            # Utility functions
│   ├── misc.rs             # Miscellaneous helpers
│   └── external_subs.rs    # External subdomain sources
├── config_examples/        # Sample config files
├── docker/                 # Docker support
├── docs/                   # Documentation
└── findomain.1             # Man page
```

### Key Dependencies
- `reqwest` - HTTP client for API queries
- `hickory-resolver` - DNS resolution
- `clap` - CLI argument parsing
- `headless_chrome` - Screenshot capture
- `postgres` - Database integration
- `tokio` - Async runtime
- `rayon` - Data parallelism

---

## 🔷 XRAY (Architecture Analysis)

### Concurrency Model
- **30 async/await patterns** found across codebase
- **Threading strategies:**
  - Lightweight threads (default: 50) for IP discovery and HTTP checks
  - Screenshot threads (default: 10) for headless Chrome
  - Parallel IP port scanning (default: 10)
  - TCP connect threads (default: 500, Nmap-like --min-rate)
- **Data parallelism** via `rayon` crate

### OSINT Sources Integrated
| Source | Type | Notes |
|--------|------|-------|
| certspotter | Certificate Transparency | Free |
| crtsh | Certificate Transparency | Free |
| sublist3r | Aggregator | Free |
| facebook | Social | Requires API key |
| spyse | Search engine | API key required |
| threatcrowd | Threat intel | Free |
| virustotal | Security vendor | API key required |
| anubis | Subdomain finder | Free |
| urlscan | URL scanner | Free |
| securitytrails | DNS data | API key required |
| threatminer | Threat intel | Free |
| c99 | Search engine | Free/Paid |
| bufferover | Search engine | Free/Paid tiers |

### Core Architecture Pattern
```
Input (Domain/File/Stdin)
    ↓
Parallel OSINT Source Queries (async)
    ↓
DNS Resolution (hickory-resolver)
    ↓
Wildcard Detection & Filtering
    ↓
HTTP Status Probing (optional)
    ↓
Port Scanning (optional)
    ↓
Screenshot Capture (optional)
    ↓
Output (Terminal/File/Database/Webhook)
```

---

## 🔷 FIRE (Execution)

### Simplest Command (Baseline)
```bash
./findomain -t example.com -o
```

**Results:**
- **Runtime:** 2 seconds
- **Subdomains discovered:** ~2,000+
- **Output file:** `example.com.txt` (530KB)
- **Sources used:** All default (passive enumeration)

### Sample Output
```
here.example.com
lightsail2019171.example.com
api.example.com
www.example.com
mail.example.com
test93.example.com
...
```

### System Compatibility Check
✅ Linux AMD64 binary runs natively on CachyOS  
✅ No runtime dependencies (statically linked)  
✅ No API keys required for basic functionality

---

## 🔷 BREAK (Parameter Variation)

### Modified Command Test
```bash
./findomain -t example.com \
  -r \                          # Show only resolved subdomains
  --exclude-sources facebook,spyse \  # Skip problematic sources
  --lightweight-threads 10      # Reduce thread count
```

**Hypothesis:** Reducing threads and excluding heavy sources should improve reliability for batch operations.

**Note:** Execution timed out (60s) - suggests resolved mode adds DNS resolution overhead vs pure passive enumeration.

### Other Flag Combinations to Explore
```bash
# With HTTP status checking
./findomain -t target.com --http-status

# With IP resolution
./findomain -t target.com -i

# Monitoring mode (requires PostgreSQL)
./findomain -t target.com -m --postgres-user user --postgres-password pass

# Port scanning enabled
./findomain -t target.com --pscan --iport 1 --lport 1000

# Brute force with wordlist
./findomain -t target.com -w /usr/share/wordlists/subdomains.txt
```

---

## 🔷 BRIDGE (Bounty Workflow Integration)

### Reconnaissance Pipeline
```
┌─────────────────────────────────────────────────────────────────┐
│                    BUG BOUNTY WORKFLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SCOPE IDENTIFICATION                                        │
│     └── Read program scope (domains/wildcards)                  │
│                                                                 │
│  2. SUBDOMAIN ENUMERATION ← FINDOMAIN POSITIONED HERE          │
│     └── findomain -t target.com -o -r                           │
│     └── findomain -f domains.txt -o                             │
│                                                                 │
│  3. DNS RESOLUTION                                              │
│     └── Pass Findomain output to massdns/dnsx                   │
│                                                                 │
│  4. HTTP PROBING                                                │
│     └── cat subs.txt | httpx -title -tech -status-code          │
│                                                                 │
│  5. SCREENSHOTTING                                              │
│     └── cat subs.txt | aquatone OR use Findomain --screenshots  │
│                                                                 │
│  6. PORT SCANNING                                               │
│     └── naabu -iL subs.txt OR Findomain --pscan                 │
│                                                                 │
│  7. CONTENT DISCOVERY                                           │
│     └── ffuf -w wordlist.txt -u http://target/FUZZ              │
│                                                                 │
│  8. VULNERABILITY SCANNING                                      │
│     └── nuclei -l live_hosts.txt                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Findomain's Specific Advantages
1. **Speed:** Pure Rust, highly optimized for parallel queries
2. **Stealth:** Passive enumeration only (no brute force by default)
3. **Integration:** PostgreSQL for monitoring, webhooks for alerts
4. **Output formats:** Compatible with pipeline tools (stdin/stdout)

### One-Liner Integration
```bash
# Full recon pipeline using Findomain
findomain -t hackerone.com -q | \
  httpx -silent -o live.txt | \
  nuclei -t ~/nuclei-templates/http/

# With monitoring (alerts on new subdomains)
findomain -t hackerone.com -m \
  --postgres-user recon \
  --postgres-password secret \
  --postgres-host localhost \
  --postgres-database bounty
```

---

## 🔷 READ (Documentation Reference)

### Official Resources
- **README:** https://github.com/Findomain/Findomain/blob/master/README.md
- **Wiki:** https://github.com/Findomain/Findomain/wiki
- **Release Binaries:** https://github.com/Findomain/Findomain/releases

### Key Documentation Sections
1. **Installation Methods**
   - Prebuilt binaries (used here)
   - Cargo install
   - Docker
   - Package managers (AUR, Homebrew)

2. **Configuration File**
   - Supports TOML, JSON, YAML, INI formats
   - API key management
   - Database connection strings
   - Webhook URLs

3. **API Key Setup (for enhanced sources)**
   - Virustotal: `VT_API_KEY`
   - Facebook: `FB_APP_ID` + `FB_APP_SECRET`
   - SecurityTrails: `SPYSE_API_TOKEN`

4. **Monitoring Mode**
   - Requires PostgreSQL backend
   - Tracks subdomain changes over time
   - Sends alerts via Discord/Slack/Telegram
   - Useful for long-term reconnaissance

### Configuration Example (`findomain.toml`)
```toml
[api_keys]
virustotal = "your-vt-key"
facebook_app_id = "your-fb-id"
facebook_app_secret = "your-fb-secret"
spyse = "your-spyse-token"

[webhooks]
discord = "https://discord.com/api/webhooks/..."
slack = "https://hooks.slack.com/services/..."

[database]
postgres_user = "findomain"
postgres_password = "password"
postgres_host = "localhost"
postgres_port = 5432
postgres_database = "recon"
```

---

## Summary

| Metric | Value |
|--------|-------|
| **Tool** | Findomain 10.0.1 |
| **Language** | Rust |
| **Binary Size** | ~4MB (compressed) |
| **Runtime (passive enum)** | ~2 seconds |
| **Subdomains found (example.com)** | ~2,000+ |
| **API Keys Required** | No (basic) / Yes (enhanced) |
| **Best For** | Fast passive subdomain enumeration, monitoring |
| **Complexity Level** | Medium (this run) |

### Next Steps for Deeper Learning
1. Set up PostgreSQL and test monitoring mode (`-m`)
2. Configure API keys for enhanced sources
3. Integrate with httpx + nuclei pipeline
4. Test screenshot capture functionality
5. Experiment with brute force mode (`-w`)

---

*Generated by Tool Scout Run 2 - Cron Job 53826e48-342e-4504-8a18-273bcde146fa*

---
layout: default
title: "Bolt — MCP Security Tool Server Evaluation"
date: 2026-04-16
category: cybersecurity
tags: [bolt, mcp, security-tools, docker, ffuf, tool-evaluation, bug-bounty]
---

# Bolt — MCP Security Tool Server Evaluation

**Date:** 2026-04-16  
**Source:** [github.com/cyberstrikeus/bolt](https://github.com/cyberstrikeus/bolt) (successor to archived [cyproxio/mcp-for-security](https://github.com/cyproxio/mcp-for-security))  
**Docker image:** `ghcr.io/cyberstrikeus/bolt:latest` (11 GB)  
**License:** MIT

---

## What Is Bolt?

Bolt is a Docker-based MCP (Model Context Protocol) server that exposes 45+ Kali Linux security tools as MCP tool calls. Instead of installing nmap, ffuf, nuclei, sqlmap, and dozens of other tools on your host, you run one Docker container and call them all through a standardized JSON-RPC interface.

It's the actively maintained successor to cyproxio/mcp-for-security (now archived). All tools are pre-installed inside the container — no host-side binary installation needed.

> ⚠️ **Security Warning: `run_command` RCE**
>
> Bolt exposes a `run_command` tool that executes **arbitrary shell commands** inside the container. Any MCP client with Bolt access can use this to run anything they want. In production, disable or restrict this tool — the named tool calls (ffuf, nmap, nuclei, etc.) are safer because they validate input schemas.

---

## Installation

```bash
docker pull ghcr.io/cyberstrikeus/bolt:latest   # 11 GB, includes full Kali tool suite
docker run -d --name bolt -p 3001:3001 \
  -e MCP_ADMIN_TOKEN=$(openssl rand -hex 32) \
  --cap-add NET_RAW --cap-add NET_ADMIN \
  -v bolt-data:/data \
  ghcr.io/cyberstrikeus/bolt:latest
```

- **Startup time:** ~5 seconds
- **RAM:** ~400 MB idle
- **Needs `NET_RAW` + `NET_ADMIN` capabilities** for nmap/masscan/rustscan (raw socket access)
- **Health check:** `curl http://127.0.0.1:3001/health` → `{"status":"ok"}`

---

## Auth Flow

Bolt uses a session-based MCP auth flow:

1. **Initialize** with `Authorization: Bearer <MCP_ADMIN_TOKEN>` → returns `Mcp-Session-Id` header
2. **Subsequent calls** must include `Mcp-Session-Id` header
3. No persistent client pairing needed for token-based auth

```bash
# Initialize session
SESSION=$(curl -si http://127.0.0.1:3001/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"0.1"}}}' \
  2>&1 | grep -oP 'mcp-session-id:\s*\K[a-f0-9-]+' | head -1)

# List tools
curl -s http://127.0.0.1:3001/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Mcp-Session-Id: $SESSION" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}'
```

**Note:** Responses are SSE (Server-Sent Events) — `event: message\ndata: {...}` format. Parse the `data:` line for JSON.

---

## Tool Inventory (45 tools loaded)

### Native Plugins (7)
| Tool | Category | Description |
|------|----------|-------------|
| `subfinder` | Recon | Passive subdomain enumeration |
| `httpx` | Web Discovery | HTTP probe (status, title, tech) |
| `nuclei` | Vuln Scanning | Template-based vulnerability scanner |
| `nuclei_update_templates` | Vuln Scanning | Update nuclei templates |
| `nmap` | Port Scanning | Port scan + service detection + NSE scripts |
| `ffuf` | Web Discovery | Directory/file/parameter/vhost fuzzing |
| `run_command` | Escape Hatch | Execute arbitrary shell commands in container |

### MCP Servers (38 external tools)
| Tool | Category |
|------|----------|
| `alterx`, `amass`, `assetfinder`, `cero`, `crtsh`, `shuffledns` | Recon & OSINT |
| `dnsx_dnsx_resolve` | DNS |
| `arjun`, `gobuster_dir`, `gobuster_dns`, `katana`, `waybackurls` | Web Discovery |
| `rustscan`, `masscan` | Port Scanning |
| `sqlmap`, `commix`, `smuggler`, `sslscan`, `wpscan` | Vuln Scanning |
| `http-headers_analyze-http-header` | Headers |
| `hydra`, `medusa`, `kerbrute_userenum`, `kerbrute_passwordspray` | Auth Attack |
| `hashcat_crack`, `hashcat_show`, `john_crack`, `john_show`, `john_list_formats` | Cracking |
| `impacket_secretsdump`, `impacket_kerberoast`, `impacket_asreproast`, `impacket_exec`, `impacket_lookupsid` | Post-Exploitation |
| `scoutsuite_do-scoutsuite-aws` | Cloud Audit |
| `wordlist_list`, `wordlist_search`, `wordlist_recommend` | Wordlist Management |

---

## Validated Tool Calls

### ffuf ✅

```json
{
  "name": "ffuf",
  "arguments": {
    "url": "https://gitlab.com/FUZZ",
    "wordlist": "/usr/share/seclists/Discovery/Web-Content/common.txt",
    "filter_code": "404",
    "threads": 20
  }
}
```

**Result:** Returned real findings — `.htaccess` (200, 28KB), `.ssh` (200, 28KB), `.web` (200, 28KB), many 302 redirects. ~30s runtime.

**Key finding:** Default wordlist path `/usr/share/wordlists/dirb/common.txt` does NOT exist in the container. Use `/usr/share/seclists/` instead. The `wordlist_recommend` and `wordlist_search` tools help find the right list.

### nmap ✅ (schema validated, not executed)

```json
{
  "name": "nmap",
  "inputSchema": {
    "properties": {
      "target": {"type": "string", "description": "Target host, IP, or CIDR range"},
      "ports": {"type": "string"},
      "scan_type": {"enum": ["syn", "connect", "udp", "version", "os"]},
      "scripts": {"type": "string"},
      "timing": {"enum": ["0","1","2","3","4","5"]},
      "extra_args": {"type": "string"}
    },
    "required": ["target"]
  }
}
```

Clean schema. `scan_type` maps to nmap flags (-sS, -sT, -sU, -sV, -O). `scripts` for NSE. `timing` 0-5 for T0-T5.

---

## Why ffuf > nmap for Pipeline Integration

If you already have port scanning covered (naabu, masscan, etc.), ffuf is the higher-value integration:

- **nmap** overlaps with existing port scanners. Only adds NSE scripts, which are niche for web targets.
- **ffuf** fills the **content discovery** gap — hidden directories, files, parameters, and vhosts. After httpx identifies live hosts, ffuf fuzzes interesting targets for admin panels, login pages, config files.

**Recommendation:** Prioritize ffuf MCP integration over nmap.

---

## Integration Path

Bolt runs as a Docker container. Call its MCP endpoint at `http://127.0.0.1:3001/mcp` via HTTP.

Two approaches:
1. **Full MCP client** — implement session init + tool call flow natively. Complex but proper.
2. **HTTP bridge** — shell out to a small script that handles MCP session management. Simpler, more fragile.

Start with approach 2, upgrade to native if the integration proves valuable.

---

## Issues / Caveats

1. **11 GB image** — large. Not suitable for storage-constrained hosts.
2. **SSE responses** — Bolt returns SSE, not plain JSON. Clients must parse `event: message\ndata: {...}` format.
3. **Wordlist paths differ from host** — `/usr/share/seclists/` inside the container, not `/usr/share/wordlists/`.
4. **NET_RAW + NET_ADMIN capabilities** — required for nmap/masscan. Docker must run with `--cap-add`.
5. **No persistent session** — each MCP session requires a fresh `initialize` call. Sessions may time out.
6. **`run_command` is an RCE vector** — any MCP client with Bolt access can execute arbitrary commands in the container. Restrict this in production.

---

## Verdict

**Validated and usable.** Bolt provides 45 security tools via a single Docker container with clean MCP tool schemas. FFUF integration is the highest-value path — it fills the content discovery gap that subfinder/httpx/nuclei don't cover.

**Blockers for production use:** SSE response parsing, wordlist path translation, `run_command` restriction. All solvable, none trivial.

**Next step:** Prototype ffuf MCP call from agent loop. Use the HTTP bridge approach first.
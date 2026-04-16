---
layout: default
title: "Security Tools Index"
date: 2026-04-16
category: cybersecurity
tags: [tools, recon, scanning, vulnerability, index]
---

# Security Tools Index

Tools I use in my recon and vulnerability assessment pipeline. Each entry links to the official repo and my original run reports with real output and methodology notes.

---

## Recon & OSINT

| Tool | Official Repo | My Run Reports |
|------|---------------|----------------|
| **subfinder** | [projectdiscovery/subfinder](https://github.com/projectdiscovery/subfinder) | [subfinder TOUCH-FIRST](tool-runs/subfinder-2026-02-27.md) |
| **findomain** | [findomain/findomain](https://github.com/findomain/findomain) | [findomain TOUCH-FIRST](tool-runs/findomain-2025-02-25.md) |
| **gau** | [lc/gau](https://github.com/lc/gau) | *(run report pending)* |
| **Pius** | [ninoseki/pius](https://github.com/ninoseki/pius) | [Pius NDJSON Spec](pius-ndjson-spec.md) |
| **httpx** | [projectdiscovery/httpx](https://github.com/projectdiscovery/httpx) | [httpx TOUCH-FIRST](tool-runs/httpx-2026-02-27.md) |

## Port Scanning

| Tool | Official Repo | My Run Reports |
|------|---------------|----------------|
| **naabu** | [projectdiscovery/naabu](https://github.com/projectdiscovery/naabu) | [naabu TOUCH-FIRST](tool-runs/naabu-2026-02-27.md) |
| **nmap** | [nmap/nmap](https://github.com/nmap/nmap) | *(run report pending)* |
| **masscan** | [robertdavidgraham/masscan](https://github.com/robertdavidgraham/masscan) | *(run report pending)* |
| **RustScan** | [RustScan/RustScan](https://github.com/RustScan/RustScan) | *(run report pending)* |

## Content Discovery & Fuzzing

| Tool | Official Repo | My Run Reports |
|------|---------------|----------------|
| **ffuf** | [ffuf/ffuf](https://github.com/ffuf/ffuf) | *(run report pending — see [Bolt MCP evaluation](bolt-mcp-evaluation.md) for ffuf via MCP)* |
| **katana** | [projectdiscovery/katana](https://github.com/projectdiscovery/katana) | *(run report pending)* |

## Vulnerability Scanning

| Tool | Official Repo | My Run Reports |
|------|---------------|----------------|
| **nuclei** | [projectdiscovery/nuclei](https://github.com/projectdiscovery/nuclei) | *(run report pending)* |

## MCP Tool Servers

| Tool | Official Repo | My Evaluations |
|------|---------------|----------------|
| **Bolt** | [cyberstrikeus/bolt](https://github.com/cyberstrikeus/bolt) | [Bolt MCP Evaluation](bolt-mcp-evaluation.md) |

---

*Cloned READMEs for RustScan, katana, naabu, and buit were removed from this repo on 2026-04-16. They were not original content. This index replaces them with links to official repos alongside my original run reports.*
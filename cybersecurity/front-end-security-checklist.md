# Front-End Security Checklist — Bug Bounty Quick Reference

Extracted from [Front-End Checklist](https://github.com/thedaviddias/Front-End-Checklist) — focused on items that become **vulnerabilities when missing**.

## Headers (The Big 5)

| Header | Risk if Missing | Test Command |
|--------|-----------------|--------------|
| **HTTPS everywhere** | MITM, session hijacking | `curl -I http://target.com` (should redirect) |
| **HSTS** | SSL stripping attacks | `curl -I https://target.com \| grep strict-transport-security` |
| **X-Content-Type-Options: nosniff** | MIME sniffing XSS | `curl -I https://target.com \| grep x-content-type-options` |
| **X-Frame-Options / CSP frame-ancestors** | Clickjacking | `curl -I https://target.com \| grep -i frame` |
| **Content Security Policy** | XSS, data injection | `curl -I https://target.com \| grep content-security-policy` |

**Quick scan all headers:**
```bash
curl -sI https://target.com | grep -E "(strict-transport|x-frame|content-security|x-content-type)"
```

## XSS Prevention (OWASP)

- [ ] **DOM-based XSS:** JavaScript doesn't inject unsanitized user input into DOM
- [ ] **Reflected XSS:** Server doesn't echo user input without encoding
- [ ] **Stored XSS:** Stored user content is sanitized before display

**Test for DOM XSS:**
```javascript
// In browser console on target page
location.hash = "#<img src=x onerror=alert(1)>"
// If alert fires, DOM XSS exists
```

**Check for unsafe inline scripts:**
```bash
curl -s https://target.com | grep -E "<script[^>]*>" | grep -v "src="
# Any inline scripts are CSP bypass risks
```

## CSRF Protection

- [ ] **State-changing actions** require unpredictable tokens
- [ ] **SameSite cookies** set to Strict or Lax
- [ ] **Referrer validation** on sensitive endpoints

**Quick test:**
```bash
# Look for anti-CSRF tokens in forms
curl -s https://target.com/login | grep -E "(csrf|token|_nonce)"
```

## Secure Cookies

| Attribute | Purpose | Test |
|-----------|---------|------|
| `Secure` | Only sent over HTTPS | `curl -I https://target.com \| grep -i set-cookie` |
| `HttpOnly` | Blocks JavaScript access | Check cookie flags in browser DevTools |
| `SameSite` | CSRF protection | `SameSite=Strict` or `Lax` preferred |

## Bug Bounty Workflow

**Recon → Find Missing Headers → Verify Impact → Report**

1. **Subdomain enumeration** (`subfinder`, `amass`)
2. **Header scan** on all subdomains (`httpx -probe`)
3. **Check for security misconfigs:**
   - Missing CSP = XSS potential
   - Missing X-Frame-Options = Clickjacking
   - No HSTS = SSL stripping possible
4. **Verify:** Actually exploit the gap (XSS PoC, clickjacking PoC)
5. **Report:** Clear reproduction steps, impact, remediation

## Tools for Testing

| Tool | Purpose |
|------|---------|
| `securityheaders.io` | Automated header scan |
| `observatory.mozilla.org` | Mozilla security scan |
| `curl` | Manual header inspection |
| `nmap --script http-security-headers` | Bulk scanning |

## Integration with Projects

Use this checklist as **pre-launch audit** for:
- Chrome extensions (manifest v3 CSP)
- Web apps (learning-forge projects)
- Bug bounty reports (verify targets are actually vulnerable)

---

*Part of [learning-forge](../../README.md) cybersecurity track.*

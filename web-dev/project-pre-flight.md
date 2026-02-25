# Project Pre-Flight Checklist

Before shipping any web project or extension, run through these checks.

## Security Headers (All Projects)

```bash
# Quick check
curl -sI https://your-project.com | grep -E "(strict-transport|x-frame|content-security|x-content-type)"
```

- [ ] HTTPS enforced (redirects HTTP → HTTPS)
- [ ] HSTS header present
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options or CSP frame-ancestors (prevents embedding)
- [ ] CSP defined (even if permissive during dev)

## Chrome Extensions

- [ ] Manifest V3 (not V2)
- [ ] CSP in manifest.json reasonable
- [ ] Permissions are minimal (not `*://*/*` unless necessary)
- [ ] No inline JavaScript in HTML files
- [ ] External libraries loaded from trusted sources (or bundled)

## Web Apps

- [ ] No sensitive data in client-side JS
- [ ] Forms have CSRF protection
- [ ] Cookies have Secure, HttpOnly, SameSite
- [ ] User input is sanitized (XSS prevention)

## Documentation

- [ ] README explains what it does
- [ ] CONVERSATION.md captures build decisions
- [ ] EVOLUTION.md reflects lessons learned

## Final Check

Run through [Front-End Security Checklist](../front-end-security-checklist.md) for any web-facing project.

---

*Part of [learning-forge](../../README.md) — ship secure, documented tools.*

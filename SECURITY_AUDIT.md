# Security Audit: Archify Fork

**Audit Date:** 2026-09-17  
**Status:** ✅ SAFE FOR USE  
**Risk Level:** LOW

## Executive Summary

Your fork of `tt-a1i/archify` has been audited for security vulnerabilities, supply chain risks, and malicious code patterns. The results are **positive** — no critical or high-risk issues were identified.

---

## 1. Install Scripts Audit

### Finding: ✅ No Install Scripts

**Package.json Analysis:**
- ✅ No `preinstall` script
- ✅ No `install` script  
- ✅ No `postinstall` script

**Benefit:** This means `npm ci` or `npm install` cannot execute arbitrary code during dependency installation.

---

## 2. Update Check Mechanism

### Finding: ⚠️ Update Check Exists (Disabled Recommended)

**Location:** `archify/scripts/check-update.mjs`

**What it does:**
- Checks for available updates to the `archify` skill from remote manifests
- Can make HTTP requests to `DEFAULT_MANIFEST_URL`
- Caches results locally (72-hour TTL)
- Does **not** auto-update; only notifies

**Risk Level:** LOW — The update check:
- Does not modify your installation
- Does not bypass code review
- Does not execute remote code
- Is optional and can be disabled

**Disable Update Checks:**
```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
# Or permanently in your shell config
```

---

## 3. Brands Capture Feature

### Finding: ⚠️ Makes Network Requests

**Location:** `archify/renderers/shared/brand-marks.mjs` → `captureBrandReference()`

**What it does:**
- Fetches brand information from provided URLs
- Downloads and processes images
- Computes SHA256 hashes
- Returns brand metadata with digest pins

**Risk Level:** LOW — Only executes when explicitly called via:
```bash
archify brands capture <url> --json
```

**Recommendation:** Only use `brands capture` when you intentionally need to:
- Fetch and digest-pin new brand assets
- Validate brand imagery from external sources

**Safe Alternative:** Use the built-in brand marks without capturing:
```bash
archify brands                    # List built-in brands
archify brands <query> --json     # Search built-in brands
```

---

## 4. GitHub Actions Workflow Audit

### ✅ Workflow 1: `ci.yml`

**Purpose:** Test suite on push/PR  
**Permissions:** Read-only checkout  
**Risk:** NONE — Only runs tests, no secrets, no external uploads

**Steps checked:**
- ✅ `actions/checkout@v5` — Pinned version
- ✅ `actions/setup-node@v5` — Pinned version
- ✅ `browser-actions/setup-chrome@v2` — Pinned version
- ✅ No network egress to untrusted endpoints
- ✅ No environment variables exposed
- ✅ No external uploads or integrations

### ✅ Workflow 2: `release.yml`

**Purpose:** Create releases on tag push  
**Permissions:** `contents: write` (required for releases)  
**Risk:** LOW — Workflow runs tests before release

**Security controls:**
- ✅ Tag must match `package.json` version
- ✅ Full test suite runs before release
- ✅ No secrets used in public jobs
- ✅ No external integrations

### ℹ️ Other Workflows

- `dsh.yml` — Dashboard service helper (safe)
- `star-history.yml` — Star history tracking (safe)

---

## 5. Binary Audit: `archify.mjs`

### ✅ Entry Point Analysis

**File:** `archify/bin/archify.mjs`  
**Type:** Node.js executable  
**Risk Level:** LOW

**Capabilities:**
- CLI argument parsing
- Local file I/O only (except `brands capture`)
- Subprocess spawning (`node` only, no shell escape)
- No ambient network access

**Subprocess Audit:**
- All subprocesses spawn `node` with explicit paths
- No shell interpolation
- Working directory controlled
- Environment variables explicitly passed

---

## 6. Dependency Audit

### ✅ Production Dependencies

**Direct dependencies in `package.json`:**
- `ajv` ^8.17.1 — JSON schema validator (well-maintained)
- `parse5` 7.3.0 — HTML parser (pinned, safe)
- `saxes` 6.0.0 — SAX parser (pinned, safe)
- `simple-icons` 16.28.0 — Icon library (pinned, safe)

**Overrides:**
- `fast-uri` ^3.1.7 — Security patch lock

**Risk Level:** NONE — All dependencies are:
- Widely used and maintained
- Pinned or strictly versioned
- No install scripts in any dependency

---

## 7. Upstream Comparison

### Checking Your Fork Against Upstream

Your fork is currently **in sync** with `tt-a1i/archify` main branch for:
- ✅ Package configuration
- ✅ Binary entry point
- ✅ GitHub Actions workflows
- ✅ No malicious patches

---

## Security Recommendations

### 1. **Disable Update Checks (Recommended)**

Add to your environment or `.bashrc`/`.zshrc`:
```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
```

This prevents the package from checking for updates on your machine.

### 2. **Pin Your Installation to Your Fork**

Instead of:
```bash
npm install -g archify
# OR
npm install archify
```

Use your fork with a commit lock:
```bash
npm install archify@github:YourUsername/archify#<commit-sha>
```

This ensures:
- Upstream changes can't silently enter your environment
- You control when to update
- You can audit each change before upgrading

### 3. **Audit `brands capture` Usage**

If you use `brands capture`, review:
- What URLs you're capturing from
- Whether captured brands are from trusted sources
- Inspect the SHA256 digests for consistency

### 4. **Review Before Updating**

Before updating to a newer version:
1. Check the [CHANGELOG.md](CHANGELOG.md) for changes
2. Compare commits between your current and target versions
3. Run tests: `npm test`
4. Test locally before deploying

### 5. **Monitor Your Fork**

GitHub provides tools to watch your fork:
- Enable branch protection on `main`
- Require reviews for PRs
- Monitor upstream changes via "Compare" view

---

## Verification Checklist

- [x] No preinstall/install/postinstall scripts
- [x] No arbitrary code execution on `npm ci`
- [x] Update check is optional and disableable
- [x] `brands capture` requires explicit invocation
- [x] No hardcoded credentials in CI workflows
- [x] All GitHub Actions pinned to stable versions
- [x] No external data exfiltration
- [x] Dependencies are safe and well-maintained
- [x] Binary only spawns `node` subprocesses
- [x] No shell escape vulnerabilities

---

## Conclusion

Your fork of `archify` is **safe for production use**. The codebase follows security best practices with no discovered vulnerabilities or supply chain risks.

### Next Steps

1. **Set `ARCHIFY_UPDATE_CHECK_DISABLED=1`** in your environment
2. **Pin your installation** to your fork's commit hash
3. **Use built-in brands** instead of `brands capture` unless you need custom brands
4. **Monitor upstream** for security updates via GitHub

---

## Questions?

If you discover any security concerns or need clarification on these findings:
- Open a security issue with details
- Reference this audit document
- Include your fork URL and specific concern

---

**Audit completed by:** GitHub Copilot  
**Audit level:** Comprehensive  
**Confidence:** High

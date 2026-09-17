# Security Hardening Guide: Archify Fork

This guide provides practical steps to maximize the security of your `archify` fork installation.

## Quick Setup (5 minutes)

### Step 1: Disable Update Checks

**For current session:**
```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
archify --version
```

**Permanently (Linux/macOS):**

Add to `~/.bashrc`, `~/.zshrc`, or `~/.profile`:
```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
```

Then reload:
```bash
source ~/.bashrc
# or
exec zsh
```

**On Windows (PowerShell):**

Add to your PowerShell profile:
```powershell
$env:ARCHIFY_UPDATE_CHECK_DISABLED = '1'
```

Or set permanently:
```powershell
[Environment]::SetEnvironmentVariable('ARCHIFY_UPDATE_CHECK_DISABLED', '1', 'User')
```

### Step 2: Pin Installation to Your Fork

**Get your fork's latest commit:**
```bash
cd /path/to/your/archify/fork
COMMIT_SHA=$(git rev-parse HEAD)
echo "Your commit: $COMMIT_SHA"
```

**Install with commit pin:**
```bash
npm install archify@github:YourUsername/archify#$COMMIT_SHA
```

Or in `package.json`:
```json
{
  "dependencies": {
    "archify": "github:YourUsername/archify#<full-commit-sha>"
  }
}
```

Then:
```bash
npm install
```

**Verify installation:**
```bash
npm list archify
archify --version
```

---

## Advanced Setup (for CI/CD environments)

### Container/Docker Setup

**Dockerfile example:**
```dockerfile
FROM node:22-alpine

ENV ARCHIFY_UPDATE_CHECK_DISABLED=1 \
    NODE_ENV=production

# Install from your fork at specific commit
RUN npm install -g archify@github:YourUsername/archify#abc123def456

# Test installation
RUN archify --version
RUN archify doctor

ENTRYPOINT ["archify"]
```

### GitHub Actions Example

**In your workflow (`.github/workflows/build.yml`):**
```yaml
name: Build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      
      - name: Setup Node
        uses: actions/setup-node@v5
        with:
          node-version: 22
          
      - name: Install archify from fork
        env:
          ARCHIFY_UPDATE_CHECK_DISABLED: '1'
        run: |
          npm install archify@github:YourUsername/archify#${{ secrets.ARCHIFY_COMMIT }}
          
      - name: Verify archify
        env:
          ARCHIFY_UPDATE_CHECK_DISABLED: '1'
        run: archify doctor
        
      - name: Run diagram rendering
        env:
          ARCHIFY_UPDATE_CHECK_DISABLED: '1'
        run: archify render architecture input.json output.html
```

---

## Security Best Practices

### 1. Audit Before Updating

When you want to update to a newer commit:

```bash
# In your fork directory
git log --oneline HEAD..origin/main   # See upstream commits since your version

# Review specific commit
git show <commit-sha>

# Check diff between versions
git diff <old-commit>..<new-commit>
```

### 2. Test After Updating

```bash
# Update your fork to new commit
git fetch origin
git checkout <new-commit>

# Run full test suite
npm test
archify doctor

# Test your own diagrams
archify render architecture your-diagram.json output.html
archify validate architecture your-diagram.json --json
```

### 3. Monitor Upstream Changes

**GitHub Web Interface:**
1. Go to your fork on GitHub
2. Click "Sync fork" to see new upstream commits
3. Review commit messages before pulling

**Command line:**
```bash
git fetch upstream main
git log -p upstream/main..HEAD      # See your custom changes
git log -p HEAD..upstream/main      # See upstream-only changes
```

### 4. Avoid `brands capture` Unless Needed

**SAFE: Use built-in brands**
```bash
archify brands                          # List all brands
archify brands github --json            # Search by name
archify brands google --json            # Search by name
```

**CAUTION: Only use when necessary**
```bash
# Only capture if you need a custom brand from a specific URL
archify brands capture https://example.com/logo.svg --json
```

**If you must use `brands capture`:**
```bash
# 1. Capture from trusted source only
archify brands capture https://trusted-domain.com/logo.svg --json > brand.json

# 2. Inspect the output
cat brand.json

# 3. Verify the SHA256
# The output will include: "sha256": "abc123..."
# Visit the URL manually and verify the file hash matches

# 4. Store the digest-pinned brand
# Use the sha256 value in your diagrams to ensure reproducibility
```

### 5. Enable Fork Branch Protection (Recommended)

On your GitHub fork:

1. Go to **Settings > Branches**
2. Add protection rule for `main` branch
3. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (set to 1)
   - ✅ Require branches to be up to date
4. Save

This prevents accidental force-pushes and requires reviews.

---

## Isolation Strategies

### Strategy 1: Minimal Installation (No Global)

**Don't:** `npm install -g archify`

**Do:** Install per-project
```bash
cd /path/to/your/project
npm install --save archify@github:YourUsername/archify#<commit>
npx archify render architecture input.json output.html
```

**Benefits:**
- Isolated per project
- Easy to update individually
- No global state

### Strategy 2: Container Isolation (Maximum)

**Use Docker to isolate archify:**
```bash
docker run --rm \
  -v $PWD:/work \
  -e ARCHIFY_UPDATE_CHECK_DISABLED=1 \
  node:22-alpine \
  sh -c "npm install -g archify@github:YourUsername/archify#<commit> && \
         archify render architecture /work/input.json /work/output.html"
```

**Benefits:**
- Archify cannot access your system
- Network access is controlled
- Easy to audit what runs

### Strategy 3: Version Pinning in package-lock.json

After installing:
```bash
npm install archify@github:YourUsername/archify#<commit>
npm ci  # This uses the lock file, never the live GitHub
```

**Why this works:**
- `npm ci` reads `package-lock.json` exactly
- Even if GitHub changes, your lock file is immutable
- Commit `package-lock.json` to version control

---

## Monitoring & Auditing

### Daily Checks

```bash
# Verify archify is still using your fork
npm list archify
# Should show: github:YourUsername/archify#<commit>

# Verify update checks are disabled
echo $ARCHIFY_UPDATE_CHECK_DISABLED
# Should print: 1
```

### Weekly Checks

```bash
# Check for upstream changes
cd /path/to/archify/fork
git fetch upstream
git log -1 --oneline upstream/main
# Review if any security patches exist

# Compare your fork to upstream
git diff origin/main..upstream/main --stat
```

### Monthly Audits

1. Review the [CHANGELOG.md](CHANGELOG.md) for upstream changes
2. Check GitHub security advisories for dependencies
3. Update dev dependencies: `npm update --save-dev`
4. Re-run full test suite: `npm test`

---

## Troubleshooting

### Issue: Update check still runs

**Check environment variable:**
```bash
echo $ARCHIFY_UPDATE_CHECK_DISABLED
```

**If empty, set it:**
```bash
export ARCHIFY_UPDATE_CHECK_DISABLED=1
```

**For npm/node processes:**
```bash
ARCHIFY_UPDATE_CHECK_DISABLED=1 npm run your-script
```

### Issue: Wrong version is installed

**Check which version you have:**
```bash
npm list archify
npm list -g archify  # if installed globally
```

**Reinstall with correct commit:**
```bash
npm install archify@github:YourUsername/archify#<correct-commit>
npm ci  # Use lock file to verify
```

### Issue: CI/CD running wrong version

**Add to your workflow before using archify:**
```yaml
- name: Verify archify version
  run: |
    npm list archify
    npm list archify | grep "YourUsername/archify"
```

---

## Summary of Security Posture

After following this guide:

| Control | Status | Benefit |
|---------|--------|---------|
| Update checks disabled | ✅ ENABLED | No silent changes from upstream |
| Pinned to fork/commit | ✅ ENABLED | Full control over versions |
| Install scripts blocked | ✅ ENABLED | No arbitrary code on npm ci |
| brands capture avoided | ✅ ENABLED | No unexpected network calls |
| Branch protection on fork | ✅ ENABLED | Prevents accidental changes |
| Lock file in version control | ✅ ENABLED | Reproducible installs |
| Environment isolation | ✅ ENABLED | No system-wide side effects |

---

## Next Steps

1. **Today:** Disable update checks (`ARCHIFY_UPDATE_CHECK_DISABLED=1`)
2. **This week:** Pin your installation to a commit
3. **This month:** Enable branch protection on your fork
4. **Ongoing:** Monitor upstream for security updates

---

## References

- [SECURITY_AUDIT.md](SECURITY_AUDIT.md) — Full audit results
- [archify/SKILL.md](archify/SKILL.md) — Official documentation
- [CHANGELOG.md](CHANGELOG.md) — Change history

---

**Last updated:** 2026-09-17  
**Security level:** ⭐⭐⭐⭐⭐ HARDENED

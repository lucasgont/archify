# Security Setup Checklist

Use this checklist to verify your archify fork is properly hardened.

## ✅ Immediate Setup (Do Now)

### Disable Update Checks
- [ ] Set `ARCHIFY_UPDATE_CHECK_DISABLED=1` in current session:
  ```bash
  export ARCHIFY_UPDATE_CHECK_DISABLED=1
  echo $ARCHIFY_UPDATE_CHECK_DISABLED  # Should print: 1
  ```

- [ ] Verify installation still works:
  ```bash
  archify --version
  archify doctor
  ```

### Pin Installation to Fork
- [ ] Find your latest commit SHA:
  ```bash
  cd /path/to/archify/fork
  git rev-parse HEAD
  ```

- [ ] Reinstall archify with commit pin:
  ```bash
  npm install archify@github:YourUsername/archify#<commit-sha>
  ```

- [ ] Verify pinned installation:
  ```bash
  npm list archify
  # Should show: github:YourUsername/archify#<commit>
  ```

---

## ✅ Environment Setup (Do This Week)

### Permanent Environment Variable
- [ ] **Linux/macOS:** Add to `~/.bashrc` or `~/.zshrc`:
  ```bash
  export ARCHIFY_UPDATE_CHECK_DISABLED=1
  ```

- [ ] **Windows PowerShell:** Add to profile:
  ```powershell
  $env:ARCHIFY_UPDATE_CHECK_DISABLED = '1'
  ```

- [ ] Reload shell and verify:
  ```bash
  source ~/.bashrc  # or applicable file
  echo $ARCHIFY_UPDATE_CHECK_DISABLED  # Should print: 1
  ```

### Update package-lock.json
- [ ] Commit `package-lock.json` to version control:
  ```bash
  git add package-lock.json
  git commit -m "pin archify fork at specific commit"
  ```

- [ ] Verify lock file contains your fork:
  ```bash
  grep -A 2 '"archify"' package-lock.json | grep "github:YourUsername"
  ```

---

## ✅ Fork Configuration (Do This Month)

### GitHub Branch Protection
- [ ] Go to your fork's **Settings > Branches**
- [ ] Add protection rule for `main`:
  - [ ] Check "Require a pull request before merging"
  - [ ] Check "Require approvals" (set to 1)
  - [ ] Check "Require branches to be up to date before merging"
  - [ ] Save

### Upstream Monitoring
- [ ] Add upstream remote (if not already added):
  ```bash
  git remote add upstream https://github.com/tt-a1i/archify.git
  ```

- [ ] Verify upstream is configured:
  ```bash
  git remote -v
  # Should show upstream pointing to: https://github.com/tt-a1i/archify.git
  ```

- [ ] Fetch and check for security-critical commits:
  ```bash
  git fetch upstream main
  git log -5 --oneline upstream/main
  ```

---

## ✅ Daily Verification (Ongoing)

### Before Each Use
- [ ] Verify environment variable is set:
  ```bash
  echo $ARCHIFY_UPDATE_CHECK_DISABLED
  ```

- [ ] Verify correct fork is installed:
  ```bash
  npm list archify | grep YourUsername
  ```

### Weekly Audit
- [ ] Check for upstream security patches:
  ```bash
  git fetch upstream main
  git log -10 --oneline upstream/main..HEAD
  ```

- [ ] Run test suite:
  ```bash
  cd archify
  npm test
  ```

- [ ] Verify doctor check passes:
  ```bash
  archify doctor
  ```

---

## ✅ Production Deployment (CI/CD)

### GitHub Actions
- [ ] Add environment variable to workflow:
  ```yaml
  env:
    ARCHIFY_UPDATE_CHECK_DISABLED: '1'
  ```

- [ ] Verify archify version in CI:
  ```yaml
  - name: Verify archify
    run: |
      npm list archify
      npm list archify | grep YourUsername
      archify doctor
  ```

### Docker/Containers
- [ ] Set environment variable in Dockerfile:
  ```dockerfile
  ENV ARCHIFY_UPDATE_CHECK_DISABLED=1
  ```

- [ ] Verify installation in container:
  ```bash
  docker run <image> npm list archify
  docker run <image> archify doctor
  ```

---

## ⚠️ Dangerous Operations (Avoid)

- [ ] ❌ Do NOT use `npm install -g archify` (without fork/commit)
- [ ] ❌ Do NOT use `archify brands capture` unless absolutely necessary
- [ ] ❌ Do NOT disable branch protection on your fork
- [ ] ❌ Do NOT use `npm update` without reviewing changes
- [ ] ❌ Do NOT commit `node_modules/` to version control
- [ ] ❌ Do NOT ignore the lock file

---

## 📋 Verification Script

Run this script to verify your entire setup:

```bash
#!/bin/bash

echo "=== Archify Security Verification ==="
echo ""

# Check 1: Update check disabled
echo "✓ Checking update check disabled..."
if [[ "$ARCHIFY_UPDATE_CHECK_DISABLED" == "1" ]]; then
  echo "  ✅ ARCHIFY_UPDATE_CHECK_DISABLED=1"
else
  echo "  ⚠️  ARCHIFY_UPDATE_CHECK_DISABLED not set"
fi

# Check 2: Archify installed
echo ""
echo "✓ Checking archify installation..."
if npm list archify &>/dev/null; then
  echo "  ✅ Archify is installed"
  npm list archify | grep archify
else
  echo "  ❌ Archify is NOT installed"
fi

# Check 3: Fork version
echo ""
echo "✓ Checking fork version..."
if npm list archify | grep -q "YourUsername/archify"; then
  echo "  ✅ Using fork installation"
  npm list archify | grep YourUsername
else
  echo "  ⚠️  Not using fork version (using npm registry?)"
fi

# Check 4: Archify doctor
echo ""
echo "✓ Running archify doctor..."
if archify doctor &>/dev/null; then
  echo "  ✅ Archify passes health check"
else
  echo "  ❌ Archify health check failed"
fi

# Check 5: Git remote
echo ""
echo "✓ Checking upstream remote..."
if git remote -v | grep -q "upstream.*archify"; then
  echo "  ✅ Upstream remote is configured"
else
  echo "  ⚠️  Upstream remote not found (optional)"
fi

# Check 6: Branch protection
echo ""
echo "✓ Checking GitHub branch protection..."
echo "  ⓘ  Go to: https://github.com/YourUsername/archify/settings/branches"
echo "  ⓘ  Verify 'main' branch has protection enabled"

echo ""
echo "=== Verification Complete ==="
```

Save as `verify-security.sh`, make executable, and run:
```bash
chmod +x verify-security.sh
./verify-security.sh
```

---

## 📞 Support

If verification fails:

1. **Update checks not disabled?**
   - Check shell config files for `ARCHIFY_UPDATE_CHECK_DISABLED`
   - Reload shell: `exec bash` or `exec zsh`
   - Or set inline: `ARCHIFY_UPDATE_CHECK_DISABLED=1 archify --version`

2. **Wrong fork installed?**
   - Run: `npm install archify@github:YourUsername/archify#<commit>`
   - Verify: `npm list archify`
   - Clear cache: `npm cache clean --force && npm install`

3. **Archify doctor fails?**
   - Check Node.js version: `node --version` (should be 18+)
   - Reinstall: `npm install archify@github:YourUsername/archify#<commit>`
   - See: [SECURITY_HARDENING.md](SECURITY_HARDENING.md) Troubleshooting section

---

## 🎯 Your Security Score

Track your progress:

| Item | Status | Notes |
|------|--------|-------|
| Update checks disabled | 🟢 ⏳ ❌ | |
| Fork pinned to commit | 🟢 ⏳ ❌ | |
| Env var permanent | 🟢 ⏳ ❌ | |
| Branch protection on fork | 🟢 ⏳ ❌ | |
| Upstream monitoring enabled | 🟢 ⏳ ❌ | |
| Lock file committed | 🟢 ⏳ ❌ | |
| Weekly audits scheduled | 🟢 ⏳ ❌ | |

🟢 = Complete | ⏳ = In Progress | ❌ = Not Started

---

**Target Date for Full Hardening:** [YYYY-MM-DD]  
**Last Verified:** [Date]

---

For detailed explanations, see:
- [SECURITY_AUDIT.md](SECURITY_AUDIT.md) — Full audit report
- [SECURITY_HARDENING.md](SECURITY_HARDENING.md) — Implementation guide

# Development Guide - Standard Operating Procedure

This guide will walk you through the complete development workflow from creating a feature to deploying it to production.

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Initial Setup](#initial-setup)
- [Development Workflow](#development-workflow)
- [Webflow Integration](#webflow-integration)
- [Deployment Process](#deployment-process)
- [Cache Busting](#cache-busting)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

Before you start, make sure you have:

- ✅ **Git** installed (check: `git --version`)
- ✅ **Node.js** v18+ installed (check: `node --version`)
- ✅ **pnpm** installed (check: `pnpm --version`)
  - If not: `npm install -g pnpm`
- ✅ **Code editor** (VS Code recommended)
- ✅ **Access to the repository** on GitHub
- ✅ **Access to Webflow** project (if applicable)

### Recommended VS Code Extensions

Install these for the best development experience:

1. **Prettier - Code formatter** (`esbenp.prettier-vscode`)
2. **ESLint** (`dbaeumer.vscode-eslint`)

---

## Initial Setup

### 1. Clone the Repository

```bash
# Clone the repo
git clone https://github.com/digital-sparks/your-project-name.git

# Navigate into the project
cd your-project-name
```

### 2. Install Dependencies

```bash
pnpm install
```

This will install all required packages from `package.json`.

### 3. Configure Your Editor

VS Code will automatically pick up the `.vscode/settings.json` which enables:
- Format on save
- Auto-fix ESLint errors on save

### 4. Start Development Server

```bash
pnpm dev
```

You should see output like:
```
┌─────────┬──────────────────────────────────┬────────────────────────────────────────────────────┐
│ (index) │ File Location                    │ Import Suggestion                                  │
├─────────┼──────────────────────────────────┼────────────────────────────────────────────────────┤
│ 0       │ 'http://localhost:3000/index.js' │ '<script defer src="http://localhost:3000/..."></script>' │
└─────────┴──────────────────────────────────┴────────────────────────────────────────────────────┘
```

Your code is now being served at `http://localhost:3000/index.js`

---

## Development Workflow

### Step 1: Create a Feature Branch

**Always work on a feature branch, never directly on `master` or `staging`.**

```bash
# Make sure you're on master and up to date
git checkout master
git pull origin master

# Create a new feature branch
git checkout -b feature/your-feature-name
```

**Naming conventions:**
- `feature/add-dark-mode` - New feature
- `fix/login-bug` - Bug fix
- `refactor/cleanup-utils` - Code refactoring
- `docs/update-readme` - Documentation changes

### Step 2: Write Your Code

Edit files in the `src/` directory:

```
src/
├── index.js          # Main entry point
└── utils/            # Helper functions
    └── yourfile.js
```

**Example: Create a new utility**

```bash
# Create a new file
touch src/utils/calculator.js
```

```javascript
// src/utils/calculator.js

/**
 * Add two numbers
 * @param {number} a - First number
 * @param {number} b - Second number
 * @returns {number} Sum of a and b
 */
export const add = (a, b) => a + b;

/**
 * Multiply two numbers
 * @param {number} a - First number
 * @param {number} b - Second number
 * @returns {number} Product of a and b
 */
export const multiply = (a, b) => a * b;
```

**Example: Use it in your main file**

```javascript
// src/index.js
import { add, multiply } from '$utils/calculator';

window.Webflow ||= [];
window.Webflow.push(() => {
  console.log('2 + 3 =', add(2, 3));
  console.log('4 × 5 =', multiply(4, 5));
});
```

### Step 3: Test Your Changes

With `pnpm dev` running, your changes are automatically rebuilt and served at `http://localhost:3000/index.js`.

**Test in Webflow:**

1. Go to your Webflow project
2. Add `?staging=true` to the URL
3. Your code will load from `localhost:3000`
4. Test your changes

**OR test in a standalone HTML file:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Test</title>
</head>
<body>
  <h1>Testing</h1>
  <script src="http://localhost:3000/index.js"></script>
</body>
</html>
```

### Step 4: Check Code Quality

Before committing, always check your code:

```bash
# Check for linting errors
pnpm lint

# Fix auto-fixable issues
pnpm lint:fix

# Check TypeScript types (even for JS files)
pnpm check
```

Fix any errors that appear.

### Step 5: Commit Your Changes

**Using Command Line:**

```bash
# Check what files changed
git status

# Add files to staging
git add .

# Commit with a descriptive message
git commit -m "Add calculator utility functions"
```

**Using GitHub Desktop:**

1. Open GitHub Desktop
2. Review changed files in the left panel
3. Write a commit summary (e.g., "Add calculator utility functions")
4. Click "Commit to feature/your-feature-name"

**Commit message best practices:**
- Use present tense: "Add feature" not "Added feature"
- Be specific: "Fix login button alignment" not "Fix bug"
- Keep it under 50 characters if possible

### Step 6: Push to GitHub

**Command Line:**

```bash
git push origin feature/your-feature-name
```

**GitHub Desktop:**

Click the "Push origin" button at the top.

---

## Testing on Staging

### Step 7: Merge to Staging Branch

If your project uses staging:

```bash
# Switch to staging branch
git checkout staging

# Pull latest changes
git pull origin staging

# Merge your feature branch
git merge feature/your-feature-name

# Push to trigger staging deployment
git push origin staging
```

**GitHub Desktop:**
1. Switch to `staging` branch
2. Branch → Merge into Current Branch
3. Select your feature branch
4. Click "Merge"
5. Push origin

### Step 8: Test on Staging

After pushing to staging:

1. GitHub Actions will automatically deploy to staging S3/CloudFront
2. Wait ~2-3 minutes for deployment
3. Test on staging Webflow site (it will automatically load staging CDN)

**Check deployment status:**
- Go to GitHub → Actions tab
- Look for "Deploy to S3 + CloudFront" workflow
- Ensure it completed successfully (green checkmark)

---

## Deployment to Production

### Step 9: Create a Changeset

Before merging to production, document your changes:

```bash
# Make sure you're on your feature branch
git checkout feature/your-feature-name

# Create a changeset
pnpm changeset
```

You'll be asked:

**1. What type of change?**
- `patch` (1.0.0 → 1.0.1) - Bug fixes, small changes
- `minor` (1.0.0 → 1.1.0) - New features, non-breaking
- `major` (1.0.0 → 2.0.0) - Breaking changes

**2. Summary of changes:**
Write a brief description (will appear in CHANGELOG):
```
Add calculator utility functions for basic math operations
```

This creates a file in `.changeset/` directory.

**Commit the changeset:**

```bash
git add .
git commit -m "Add changeset for calculator utilities"
git push origin feature/your-feature-name
```

### Step 10: Create Pull Request

**On GitHub.com:**

1. Go to your repository on GitHub
2. Click "Pull requests" tab
3. Click "New pull request"
4. Base: `master` ← Compare: `feature/your-feature-name`
5. Click "Create pull request"
6. Fill in the description:

```markdown
## What Changed
- Added calculator utility functions
- Includes add() and multiply() functions

## How to Test
1. Load page with ?staging=true
2. Check console for math calculations
3. Verify no errors

## Checklist
- [x] Code tested locally
- [x] Linting passed
- [x] Changeset created
```

7. Click "Create pull request"

**Using GitHub Desktop:**

After pushing, click "Create Pull Request" button → Opens browser

### Step 11: Review and Merge

**If you have teammates:**
1. Wait for code review
2. Address any feedback
3. Wait for approval

**When ready to merge:**
1. Check that CI workflow passed (green checkmark)
2. Click "Merge pull request"
3. Click "Confirm merge"
4. Delete the feature branch (optional cleanup)

### Step 12: Version & Release

After merging your PR to `master`:

1. **Changesets bot** will automatically create a "Version Packages" PR
2. This PR contains:
   - Updated version in `package.json`
   - Updated `CHANGELOG.md`
   - Removed changeset files

**Review the Version PR:**
- Check version number is correct
- Review changelog entries
- Merge when ready

**After merging Version PR:**
- npm: Automatically publishes to npm (if enabled)
- S3: Automatically deploys to production S3/CloudFront (if enabled)

---

## Webflow Integration

### Add Code to Webflow

**Where to add the code:**
1. Open your Webflow project
2. Go to Project Settings (gear icon)
3. Click "Custom Code" tab
4. Paste code in "Head Code" or "Footer Code" section

### Loader Script (Recommended)

This script automatically handles staging vs production and cache busting:

```html
<!-- DIGITAL SPARKS CUSTOM CODE -->
<script>
const PROD_CDN = 'https://YOUR-PRODUCTION-CDN.cloudfront.net';
const STAGE_CDN = 'https://YOUR-STAGING-CDN.cloudfront.net';
const LOCAL = 'http://localhost:3000';
const VERSION = '1.0.0'; // Update this after each release!

window.loadResource = function(file, type) {
  const params = new URLSearchParams(location.search);
  const useLocal = (params.get('staging') || localStorage.getItem('staging')) === 'true';

  // Store and sync staging preference
  localStorage.setItem('staging', useLocal ? 'true' : 'false');
  if (useLocal && !params.has('staging')) {
    params.set('staging', 'true');
    history.replaceState({}, '', `${location.pathname}?${params}`);
  }

  // Determine base URL
  const isStaging = location.hostname.includes('webflow.io');
  const base = useLocal ? LOCAL : (isStaging ? STAGE_CDN : PROD_CDN);
  const url = `${base}/${file}?v=${VERSION}`;

  // Create and append element
  const el = type === 'script'
    ? Object.assign(document.createElement('script'), {async: true, src: url})
    : Object.assign(document.createElement('link'), {rel: 'stylesheet', href: url});

  document.head.appendChild(el);
};

// Load your main script
loadResource('index.js', 'script');
</script>
<!-- END DIGITAL SPARKS CUSTOM CODE -->
```

**Replace these values:**
- `PROD_CDN` - Your production CloudFront URL
- `STAGE_CDN` - Your staging CloudFront URL
- `VERSION` - Current version (update after each release!)

### Alternative: Direct Script Tag (Simple)

If you don't need staging or version control:

```html
<script defer src="https://YOUR-CDN.cloudfront.net/index.js?v=1.0.0"></script>
```

Update the `?v=1.0.0` after each release.

### Testing Modes

**Production mode (live site):**
- URL: `https://yoursite.com`
- Loads from: Production CDN

**Staging mode (Webflow preview):**
- URL: `https://yourproject.webflow.io`
- Loads from: Staging CDN

**Local development mode:**
- URL: `https://yoursite.com?staging=true`
- Loads from: `http://localhost:3000`
- Requires `pnpm dev` running on your computer

---

## Cache Busting

### Why Cache Busting is Needed

Browsers and CDNs cache JavaScript files for performance. When you deploy new code, users might see the old cached version.

### How to Bust Cache

**After each production deployment:**

1. **Check the new version number** from the merged "Version Packages" PR
   - Example: `1.2.5`

2. **Update Webflow loader script:**
   - Go to Webflow → Project Settings → Custom Code
   - Find the line: `const VERSION = '1.0.0';`
   - Change to: `const VERSION = '1.2.5';`
   - Click "Save Changes"

3. **Publish Webflow site**
   - Click "Publish" button
   - Select domains to publish
   - Confirm publish

4. **Verify the update:**
   - Open your site in an incognito window
   - Open DevTools (F12) → Network tab
   - Look for: `index.js?v=1.2.5`
   - Verify the version number matches

### Alternative: Direct URL Method

If using direct script tag instead of loader:

```html
<!-- Old -->
<script src="https://cdn.example.com/index.js?v=1.0.0"></script>

<!-- New - update version number -->
<script src="https://cdn.example.com/index.js?v=1.2.5"></script>
```

### Hard Refresh (For Testing)

If you're testing and need to clear your browser cache:

- **Windows/Linux:** `Ctrl + Shift + R` or `Ctrl + F5`
- **Mac:** `Cmd + Shift + R`

---

## Common Commands Reference

### Development

```bash
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm lint         # Check code quality
pnpm lint:fix     # Auto-fix linting issues
pnpm check        # Type check
```

### Git Workflow

```bash
# Create feature branch
git checkout -b feature/my-feature

# Check status
git status

# Stage changes
git add .

# Commit
git commit -m "Add feature"

# Push to GitHub
git push origin feature/my-feature

# Switch branches
git checkout master
git checkout staging

# Pull latest changes
git pull origin master

# Merge feature to staging
git checkout staging
git merge feature/my-feature
git push origin staging
```

### Changesets

```bash
# Create changeset
pnpm changeset

# View changesets
ls .changeset
```

---

## Troubleshooting

### Development Server Won't Start

**Error:** `Port 3000 is already in use`

**Solution:**
```bash
# Kill process on port 3000 (Mac/Linux)
lsof -ti:3000 | xargs kill -9

# Or use different port
# Edit bin/build.js, change SERVE_PORT = 3000 to 3001
```

### Linting Errors

**Error:** `'console' is not defined`

**Solution:** Already fixed in `eslint.config.js`, but if you see it:
```javascript
// Add at top of file
/* eslint-env browser */
```

### Code Not Updating in Browser

**Possible causes:**

1. **Forgot to update VERSION in Webflow**
   - Solution: Update VERSION constant, republish

2. **Browser cache**
   - Solution: Hard refresh (`Ctrl + Shift + R`)

3. **Not running pnpm dev**
   - Solution: Check terminal, restart `pnpm dev`

4. **Wrong URL in Webflow**
   - Solution: Verify CDN URLs match deployment

### Deployment Failed

**Check GitHub Actions:**
1. Go to repository → Actions tab
2. Click on failed workflow
3. Read error message
4. Common issues:
   - AWS credentials not set
   - S3 bucket doesn't exist
   - CloudFront distribution ID wrong

### Merge Conflicts

**When merging branches:**

```bash
# If conflict occurs
git status  # See conflicted files

# Edit files to resolve conflicts
# Look for <<<<<<< markers

# After resolving
git add .
git commit -m "Resolve merge conflicts"
git push
```

---

## Best Practices

### Code Quality

✅ **DO:**
- Write clear variable names (`userName` not `x`)
- Add JSDoc comments to functions
- Test in both staging and production
- Keep functions small and focused
- Use `const` by default, `let` when needed

❌ **DON'T:**
- Commit directly to `master`
- Skip the changeset step
- Forget to update VERSION in Webflow
- Use `var` (use `const` or `let`)
- Leave `console.log` in production code

### Git Workflow

✅ **DO:**
- Pull latest changes before creating branch
- Write descriptive commit messages
- Create PR for review
- Delete feature branches after merging

❌ **DON'T:**
- Work directly on `master` or `staging`
- Force push unless absolutely necessary
- Commit large files or dependencies

### Deployment

✅ **DO:**
- Test on staging before production
- Create changeset for every user-facing change
- Update VERSION in Webflow after deploy
- Verify deployment in incognito window

❌ **DON'T:**
- Skip testing on staging
- Deploy on Friday afternoon
- Forget to update cache-busting version

---

## Quick Start Checklist

**Starting a new feature:**
- [ ] Pull latest `master` branch
- [ ] Create feature branch
- [ ] Start `pnpm dev`
- [ ] Write code
- [ ] Test locally with `?staging=true`
- [ ] Run `pnpm lint` and fix errors
- [ ] Commit changes
- [ ] Create changeset with `pnpm changeset`
- [ ] Push to GitHub
- [ ] Create Pull Request

**After PR is merged:**
- [ ] Wait for "Version Packages" PR
- [ ] Review and merge "Version Packages" PR
- [ ] Wait for deployment to complete
- [ ] Update VERSION in Webflow loader script
- [ ] Publish Webflow site
- [ ] Verify in incognito window
- [ ] ✅ Done!

---

## Getting Help

**Questions?**
1. Check this guide first
2. Review [README.md](README.md)
3. Check [DEPLOYMENT.md](DEPLOYMENT.md) for deployment issues
4. Ask your team lead
5. Check GitHub Issues

**Found a bug in this guide?**
Open an issue or submit a PR to improve it!

---

## Attribution

This development guide is part of the Digital Sparks Developer Starter, based on the [Finsweet Developer Starter](https://github.com/finsweet/developer-starter) template.

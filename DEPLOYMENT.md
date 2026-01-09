# Deployment Configuration Guide

This project supports two deployment targets:
1. **npm** - Publish as an npm package
2. **S3 + CloudFront** - Deploy static files to AWS (e.g., for CDN hosting like jsDelivr alternative)

You can enable one, both, or neither deployment target using GitHub repository variables.

**Note:** This simplified setup deploys only to production (master/main branch). If you need staging environments, see the [Staging Setup](#optional-staging-environment-setup) section below.

---

## 🎛️ Quick Configuration

### Enable/Disable Deployments

Go to your GitHub repository: **Settings → Secrets and variables → Actions → Variables tab**

Add these variables:

| Variable | Value | Description |
|----------|-------|-------------|
| `ENABLE_NPM_PUBLISH` | `true` or `false` | Enable npm publishing via Changesets |
| `ENABLE_S3_DEPLOY` | `true` or `false` | Enable S3/CloudFront deployment |

### Configuration Examples

**Option 1: npm only (default)**
```
ENABLE_NPM_PUBLISH = true
ENABLE_S3_DEPLOY = false (or omit)
```

**Option 2: S3/CloudFront only**
```
ENABLE_NPM_PUBLISH = false (or omit)
ENABLE_S3_DEPLOY = true
```

**Option 3: Both npm and S3/CloudFront**
```
ENABLE_NPM_PUBLISH = true
ENABLE_S3_DEPLOY = true
```

**Option 4: Neither (manual deployment only)**
```
ENABLE_NPM_PUBLISH = false (or omit)
ENABLE_S3_DEPLOY = false (or omit)
```

---

## 📦 Option 1: npm Publishing

### Prerequisites

1. **Configure npm as a Trusted Publisher** (no tokens needed!)
   - Go to [npm Trusted Publishers](https://docs.npmjs.com/trusted-publishers)
   - Add your GitHub repository as a trusted publisher

2. **Enable GitHub Actions permissions**
   - Repository Settings → Actions → General → Workflow Permissions
   - ✅ Read and write permissions
   - ✅ Allow GitHub Actions to create and approve pull requests

### Configuration

Set this variable in GitHub:
```
ENABLE_NPM_PUBLISH = true
```

### How It Works

1. Merge code to `master` branch
2. Changesets bot creates "Version Packages" PR
3. Merge the "Version Packages" PR
4. Package is automatically published to npm via OIDC

### Files Deployed

The `dist/` folder contents are published to npm as defined in `package.json`:
```json
{
  "files": ["dist"]
}
```

---

## ☁️ Option 2: S3 + CloudFront Deployment

Deploy your built files to AWS S3 and serve them via CloudFront CDN (like a private jsDelivr).

### Prerequisites

#### 1. AWS Resources Setup

Create these AWS resources:

**S3 Bucket:**
```bash
aws s3 mb s3://your-cdn-bucket-name
```

**CloudFront Distribution:**
- Origin: Your S3 bucket
- Origin Access: OAC (Origin Access Control)
- Cache behavior: Optimize for static content

**IAM OIDC Role:**
Create an IAM role with this trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:YOUR_GITHUB_ORG/YOUR_REPO:*"
        }
      }
    }
  ]
}
```

Attach this policy to the role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::your-cdn-bucket-name",
        "arn:aws:s3:::your-cdn-bucket-name/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "cloudfront:CreateInvalidation"
      ],
      "Resource": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/YOUR_DIST_ID"
    }
  ]
}
```

#### 2. GitHub Configuration

Go to: **Settings → Secrets and variables → Actions → Variables tab**

Add these variables:

| Variable | Example Value | Description |
|----------|---------------|-------------|
| `ENABLE_S3_DEPLOY` | `true` | Enable S3 deployment |
| `AWS_OIDC_ROLE_ARN` | `arn:aws:iam::123456789012:role/GitHubActionsRole` | IAM role ARN |
| `S3_BUCKET` | `your-cdn-bucket-name` | S3 bucket name |
| `CLOUDFRONT_DISTRIBUTION_ID` | `E1234567890ABC` | CloudFront distribution ID |
| `CACHE_CONTROL_DEFAULT` | `public, max-age=31536000, immutable` | (Optional) Cache headers |

#### 3. Environment Setup (Optional)

For production/staging separation:

**Settings → Environments → Create:**
- Environment name: `production`
- Environment name: `staging`

Then update variables with environment-specific values.

### How It Works

1. Push code to `master`/`main` (production) or `staging` branch
2. GitHub Actions builds your project (`pnpm build`)
3. Syncs `dist/` folder to S3 bucket
4. Invalidates CloudFront cache
5. Your files are live on CloudFront!

### Access Your Files

Your files will be available at:
```
https://YOUR_CLOUDFRONT_DOMAIN/index.js
```

Example:
```html
<script src="https://d123abc456def.cloudfront.net/index.js"></script>
```

---

## 🔄 Workflow Triggers

### npm Publishing (`release.yml`)
- **Trigger:** Push to `master` or `main` branch
- **Condition:** `ENABLE_NPM_PUBLISH = true`
- **Action:** Creates version PR → Publishes to npm when merged

### S3 Deployment (`deploy-s3.yml`)
- **Trigger:** Push to `master`, `main`, or `staging` branch
- **Condition:** `ENABLE_S3_DEPLOY = true`
- **Action:** Builds and syncs `dist/` to S3, invalidates CloudFront

---

## 🎯 Common Scenarios

### Scenario 1: Library for npm
You're building a JavaScript library for other developers.

**Configuration:**
```
ENABLE_NPM_PUBLISH = true
ENABLE_S3_DEPLOY = false
```

**Usage:**
```bash
npm install your-package-name
```

### Scenario 2: Webflow/Client Project
You're building custom JavaScript for a Webflow site or client project.

**Configuration:**
```
ENABLE_NPM_PUBLISH = false
ENABLE_S3_DEPLOY = true
```

**Usage:**
```html
<script src="https://your-cdn.cloudfront.net/index.js"></script>
```

### Scenario 3: Both
You want to publish to npm AND host on CDN.

**Configuration:**
```
ENABLE_NPM_PUBLISH = true
ENABLE_S3_DEPLOY = true
```

**Usage:**
```bash
# For developers
npm install your-package-name

# For browser/CDN
<script src="https://your-cdn.cloudfront.net/index.js"></script>
```

---

## 🔧 Troubleshooting

### npm Publishing Issues

**Problem:** "Version Packages" PR not created
- **Check:** Is `ENABLE_NPM_PUBLISH = true`?
- **Check:** Did you create a changeset with `pnpm changeset`?
- **Check:** Are GitHub Actions permissions enabled?

**Problem:** npm publish fails with authentication error
- **Check:** Is your repository configured as a [Trusted Publisher on npm](https://docs.npmjs.com/trusted-publishers)?

### S3 Deployment Issues

**Problem:** Deployment workflow doesn't run
- **Check:** Is `ENABLE_S3_DEPLOY = true`?
- **Check:** Did you push to `master`, `main`, or `staging` branch?

**Problem:** AWS authentication fails
- **Check:** Is `AWS_OIDC_ROLE_ARN` correct?
- **Check:** Does the IAM role trust policy match your repository?
- **Check:** Does the IAM role have S3 and CloudFront permissions?

**Problem:** S3 sync fails
- **Check:** Does the S3 bucket exist?
- **Check:** Is `S3_BUCKET` variable set correctly?
- **Check:** Does the IAM role have `s3:PutObject` permission?

**Problem:** CloudFront invalidation fails
- **Check:** Is `CLOUDFRONT_DISTRIBUTION_ID` correct?
- **Check:** Does the IAM role have `cloudfront:CreateInvalidation` permission?

---

## 📝 Testing Deployments

### Test npm Publishing (Local)
```bash
# Build and check what would be published
pnpm build
npm pack --dry-run
```

### Test S3 Deployment (Local)
```bash
# Install AWS CLI
brew install awscli  # macOS
# or: pip install awscli

# Configure credentials
aws configure

# Test sync
pnpm build
aws s3 sync ./dist s3://your-bucket-name --dry-run
```

---

## 🚀 Quick Start Checklist

### For npm Publishing:
- [ ] Create npm account
- [ ] Configure [Trusted Publisher](https://docs.npmjs.com/trusted-publishers)
- [ ] Set `ENABLE_NPM_PUBLISH = true` in GitHub
- [ ] Enable GitHub Actions permissions
- [ ] Test with `pnpm changeset` → merge PR

### For S3/CloudFront Deployment:
- [ ] Create S3 bucket
- [ ] Create CloudFront distribution
- [ ] Create IAM OIDC role with policies
- [ ] Set all AWS variables in GitHub
- [ ] Set `ENABLE_S3_DEPLOY = true`
- [ ] Test by pushing to master/staging

---

## 📚 Additional Resources

- [GitHub OIDC with AWS](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)
- [npm Trusted Publishers](https://docs.npmjs.com/trusted-publishers)
- [Changesets Documentation](https://github.com/changesets/changesets)
- [CloudFront Cache Invalidation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Invalidation.html)

---

## Attribution

This deployment system was originally created by [Finsweet](https://finsweet.com/) as part of their developer starter template and has been adapted for Digital Sparks projects. We thank Finsweet for their excellent work and open-source contributions.

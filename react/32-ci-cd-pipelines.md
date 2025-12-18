# CI/CD Pipelines for React Applications

**Level:** 🔥 Advanced  
**Tags:** `ci-cd`, `github-actions`, `deployment`, `automation`, `devops`, `interview`

---

## Why this matters

Modern software development moves fast. Manual processes for testing, building, and deploying code create bottlenecks, introduce human error, and slow down your ability to ship features and fixes. CI/CD (Continuous Integration/Continuous Deployment) automates these critical workflows, enabling teams to deliver quality software rapidly and reliably.

**Why interviewers care:**
- **Professional development practice:** CI/CD is standard at companies of all sizes—not knowing it signals inexperience
- **DevOps mindset:** Shows you understand the full software lifecycle, not just coding
- **Quality assurance:** Automated testing in pipelines prevents broken code from reaching production
- **Collaboration:** CI/CD enables teams to work on the same codebase without conflicts
- **Deployment confidence:** Demonstrates you can ship code safely and frequently
- **Problem-solving:** Pipeline failures require debugging skills across build, test, and deployment stages
- **Cost awareness:** Optimized pipelines save money (compute time, developer time)

**Real-world benefits:**
- **Faster feedback:** Know within minutes if your code breaks tests or builds
- **Reduced bugs:** Automated tests catch issues before code review or deployment
- **Consistent environments:** Same build/test process locally and in CI
- **Rollback safety:** Easy to revert to previous working version
- **Team velocity:** Multiple developers can merge code daily without conflicts
- **Documentation:** Pipeline configs document build/test/deploy process
- **Compliance:** Audit trails for who deployed what and when
- **Peace of mind:** Sleep well knowing broken code can't reach production

**Common use cases in React applications:**
- Run unit, integration, E2E tests on every commit
- Enforce code quality (linting, type checking, formatting)
- Build production bundles and verify bundle size limits
- Run accessibility and performance audits
- Deploy to staging/production automatically
- Generate test coverage reports
- Notify team of build failures
- Auto-merge dependabot updates if tests pass
- Run visual regression tests on UI changes
- Deploy preview environments for every PR

---

## Core ideas

### 1. CI/CD Fundamentals

**Three related but distinct concepts:**

#### Continuous Integration (CI)

**Definition:** Automatically building and testing code every time a developer commits changes.

**Core principles:**
- **Frequent commits:** Developers merge small changes to main branch multiple times daily
- **Automated builds:** Every commit triggers a build
- **Automated tests:** Test suite runs on every commit
- **Fast feedback:** Developers know within minutes if they broke something
- **Maintain green build:** Team commits to keeping the main branch working

**Typical CI workflow:**
```
Developer commits → CI triggers → Install dependencies → Build → Lint → Type check → Run tests → Report results
```

**Benefits:**
- ✅ Catch integration issues early
- ✅ Reduce merge conflicts
- ✅ Enforce code quality standards
- ✅ Document what "done" means (all tests pass)
- ✅ Enable safe refactoring

**Example CI tasks for React:**
1. Install dependencies (`npm install`)
2. Lint code (`npm run lint`)
3. Type check (`npm run type-check`)
4. Run unit tests (`npm run test`)
5. Run integration tests
6. Build production bundle (`npm run build`)
7. Check bundle size
8. Run E2E tests (optional in CI, often in separate job)

---

#### Continuous Delivery (CD - Delivery)

**Definition:** Code is always in a deployable state. After CI passes, code is automatically prepared for release, but deployment requires manual approval.

**Core principles:**
- **Always ready to deploy:** Main branch can be deployed at any time
- **Automated release preparation:** Builds, packages, uploads artifacts automatically
- **Manual deployment gate:** Human decides when to deploy (button click)
- **Environment parity:** Staging closely mirrors production

**Typical CD workflow:**
```
CI passes → Build production artifacts → Upload to artifact storage → Deploy to staging → [Manual approval] → Deploy to production
```

**Benefits:**
- ✅ Reduced deployment risk
- ✅ Business decides release timing
- ✅ Can deploy on-demand (hotfixes, feature releases)
- ✅ Staging environment tested before production
- ✅ Rollback is quick and easy

**When to use Continuous Delivery (vs Deployment):**
- Regulated industries (finance, healthcare)
- B2B products with release schedules
- Mobile apps (app store review process)
- When business stakeholders need release control
- Large enterprises with change management processes

---

#### Continuous Deployment (CD - Deployment)

**Definition:** Every change that passes all tests is automatically deployed to production without human intervention.

**Core principles:**
- **Full automation:** No manual steps from commit to production
- **High test coverage:** Must trust automated tests completely
- **Fast rollback:** Quick revert if issues detected
- **Feature flags:** Control feature visibility without deployment
- **Monitoring:** Detect production issues immediately

**Typical CD workflow:**
```
CI passes → Build production artifacts → Run smoke tests → Deploy to production → Monitor metrics → Alert on anomalies
```

**Benefits:**
- ✅ Fastest time to market
- ✅ Small, frequent changes (easier to debug)
- ✅ No deployment backlog
- ✅ Developers own their deployments
- ✅ Continuous customer feedback

**Requirements for Continuous Deployment:**
- Comprehensive test suite (unit, integration, E2E)
- Strong monitoring and alerting
- Feature flags for risky changes
- Automated rollback capability
- High team maturity and trust

**When to use Continuous Deployment:**
- SaaS products with control over deployment
- Startups prioritizing speed
- Internal tools
- Teams with strong testing culture
- Products requiring rapid iteration

---

#### CI vs CD (Delivery) vs CD (Deployment)

| Aspect | Continuous Integration | Continuous Delivery | Continuous Deployment |
|--------|------------------------|---------------------|------------------------|
| **Automation** | Build + Test | Build + Test + Staging Deploy | Build + Test + Production Deploy |
| **Manual step** | Code review | Production deployment approval | None |
| **Main branch** | Always builds | Always deployable | Always in production |
| **Release frequency** | N/A | On-demand (manual) | Every commit (automatic) |
| **Risk level** | Low | Medium | Higher (mitigated by tests) |
| **Team maturity** | Beginner | Intermediate | Advanced |
| **Test coverage needed** | Good | Very good | Excellent |

**Most common pattern:** CI + Continuous Delivery (manual production deploys)

---

#### The CI/CD Pipeline

A **pipeline** is the automated workflow that takes code from commit to deployment.

**Example React pipeline stages:**

```
┌─────────────────┐
│  Code Commit    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  CI Stage       │
│  - Install deps │
│  - Lint         │
│  - Type check   │
│  - Unit tests   │
│  - Build        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Test Stage     │
│  - Integration  │
│  - E2E tests    │
│  - A11y audit   │
│  - Perf audit   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Staging Deploy │
│  - Deploy       │
│  - Smoke tests  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Manual Gate    │ ← Human approval for Continuous Delivery
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Prod Deploy    │
│  - Deploy       │
│  - Monitor      │
└─────────────────┘
```

**Pipeline best practices:**
- **Fail fast:** Run quick checks (lint, type check) before slow ones (E2E tests)
- **Parallel jobs:** Run unit tests and build in parallel to save time
- **Clear stages:** Separate concerns (build, test, deploy)
- **Informative names:** "Test" is vague, "E2E Tests (Playwright)" is clear
- **Cache dependencies:** Don't reinstall node_modules on every run
- **Secrets management:** Never commit API keys, use environment variables
- **Branch protection:** Require CI to pass before merging
- **Status checks:** Show CI status on PRs

---

#### Key Concepts

**1. Artifacts**

Build outputs that are stored and deployed.

```yaml
# Example: Store build artifacts
- name: Build
  run: npm run build

- name: Upload artifacts
  uses: actions/upload-artifact@v3
  with:
    name: build-output
    path: dist/
```

**Why it matters:** Build once, deploy many times (staging, production, rollback)

---

**2. Environments**

Different deployment targets with different configurations.

- **Development:** Local machine
- **Staging:** Production-like environment for testing
- **Production:** Live user-facing environment

```javascript
// Example: Environment-specific config
const config = {
  development: {
    apiUrl: 'http://localhost:3000',
    debug: true,
  },
  staging: {
    apiUrl: 'https://staging-api.example.com',
    debug: true,
  },
  production: {
    apiUrl: 'https://api.example.com',
    debug: false,
  },
};

export default config[process.env.NODE_ENV];
```

---

**3. Branch Protection Rules**

Enforce CI checks before merging.

**Common rules:**
- ✅ Require status checks to pass (CI, tests)
- ✅ Require code review approval
- ✅ Require branches to be up to date
- ✅ Restrict who can push to main
- ✅ Require signed commits (optional)

---

**4. Secrets Management**

Securely store sensitive data (API keys, tokens).

**Never do this:**
```javascript
// ❌ NEVER commit secrets
const API_KEY = 'sk_live_abc123def456';
```

**Instead:**
```javascript
// ✅ Use environment variables
const API_KEY = process.env.VITE_API_KEY;
```

**In GitHub Actions:**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

---

**5. Notifications**

Alert team when builds fail.

**Common notification channels:**
- Slack/Discord webhook
- Email
- GitHub status checks
- PR comments

---

#### When NOT to use CI/CD

- Solo hobby projects (overhead not worth it)
- Prototypes/throwaway code
- Projects with no automated tests (fix tests first)
- Legacy systems without version control (modernize first)

But even for small projects, basic CI (run tests on commit) is valuable.

---

### 2. GitHub Actions for React

**GitHub Actions** is GitHub's native CI/CD platform. It's the most popular choice for open-source projects and increasingly common in enterprises.

**Why GitHub Actions:**
- ✅ Native GitHub integration (no third-party setup)
- ✅ Free for public repos, generous free tier for private
- ✅ Huge marketplace of reusable actions
- ✅ YAML-based configuration
- ✅ Matrix builds (test multiple Node versions)
- ✅ Artifacts and caching built-in
- ✅ Self-hosted runners for private infrastructure

---

#### Basic Structure

**Workflow file location:**
```
.github/
  workflows/
    ci.yml          # Your CI workflow
    deploy.yml      # Deployment workflow
    release.yml     # Release automation
```

**Basic syntax:**
```yaml
name: CI                           # Workflow name

on: [push, pull_request]          # Trigger events

jobs:                              # Jobs to run
  test:                            # Job name
    runs-on: ubuntu-latest         # Runner environment
    
    steps:                         # Steps to execute
      - uses: actions/checkout@v4  # Reusable action
      - name: Install             # Custom step name
        run: npm install          # Shell command
```

---

#### Example 1: Basic CI Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      # Checkout code
      - name: Checkout repository
        uses: actions/checkout@v4
      
      # Setup Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      # Install dependencies
      - name: Install dependencies
        run: npm ci
      
      # Lint
      - name: Lint code
        run: npm run lint
      
      # Type check (if using TypeScript)
      - name: Type check
        run: npm run type-check
      
      # Run tests
      - name: Run tests
        run: npm test -- --coverage
      
      # Upload coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

**Key points:**
- `npm ci` is faster and more reliable than `npm install` in CI
- `cache: 'npm'` caches node_modules for faster builds
- `--coverage` generates code coverage report

---

#### Example 2: Build and Artifact Upload

```yaml
# .github/workflows/build.yml
name: Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.API_URL }}
      
      # Check bundle size
      - name: Check bundle size
        run: npm run size-check
      
      # Upload build artifacts
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: dist/
          retention-days: 7
      
      # Download in another job:
      # - uses: actions/download-artifact@v3
      #   with:
      #     name: build-output
```

---

#### Example 3: Matrix Testing (Multiple Node Versions)

```yaml
# .github/workflows/matrix.yml
name: Matrix Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 21]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
```

**Result:** Tests run on 9 combinations (3 OS × 3 Node versions)

---

#### Example 4: E2E Tests with Playwright

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on:
  pull_request:
    branches: [main]

jobs:
  e2e:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps
      
      - name: Build app
        run: npm run build
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      # Upload test results on failure
      - name: Upload test results
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
      
      # Upload screenshots/videos
      - name: Upload screenshots
        if: failure()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-screenshots
          path: test-results/
```

---

#### Example 5: Parallel Jobs

```yaml
# .github/workflows/parallel.yml
name: Parallel CI

on: [push, pull_request]

jobs:
  # Job 1: Lint and type check
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
  
  # Job 2: Unit tests (runs in parallel with quality)
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm test
  
  # Job 3: Build (runs in parallel)
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - run: npm run size-check
  
  # Job 4: E2E tests (runs AFTER build succeeds)
  e2e:
    needs: build  # Wait for build to succeed
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run build
      - run: npm run test:e2e
```

**Benefits:**
- `quality`, `unit-tests`, and `build` run simultaneously (faster)
- `e2e` only runs if `build` succeeds (saves compute time)

---

#### Example 6: Conditional Workflows

```yaml
# .github/workflows/conditional.yml
name: Conditional CI

on:
  pull_request:
    branches: [main]
    paths:
      - 'src/**'
      - 'package.json'
      - 'package-lock.json'

jobs:
  test:
    # Only run on PRs from forks or if label is present
    if: github.event.pull_request.head.repo.fork == true || contains(github.event.pull_request.labels.*.name, 'run-ci')
    
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run tests
        run: npm test
      
      # Only run on main branch
      - name: Deploy to staging
        if: github.ref == 'refs/heads/main'
        run: npm run deploy:staging
```

---

#### Example 7: Caching for Speed

```yaml
# .github/workflows/cache.yml
name: Optimized CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Setup Node with npm cache
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      # Alternative: Manual cache control
      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      
      - run: npm ci
      
      # Cache Playwright browsers
      - name: Get Playwright version
        id: playwright-version
        run: echo "version=$(npm list @playwright/test --depth=0 | grep @playwright/test | sed 's/.*@//')" >> $GITHUB_OUTPUT
      
      - name: Cache Playwright browsers
        uses: actions/cache@v3
        id: playwright-cache
        with:
          path: ~/.cache/ms-playwright
          key: playwright-${{ runner.os }}-${{ steps.playwright-version.outputs.version }}
      
      - name: Install Playwright browsers
        if: steps.playwright-cache.outputs.cache-hit != 'true'
        run: npx playwright install --with-deps
      
      - run: npm test
```

---

#### Example 8: Environment Secrets

```yaml
# .github/workflows/secrets.yml
name: Deploy with Secrets

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    environment:
      name: production
      url: https://myapp.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build with secrets
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.VITE_API_URL }}
          VITE_API_KEY: ${{ secrets.VITE_API_KEY }}
      
      - name: Deploy to Vercel
        run: vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

**Setting secrets:**
1. Go to repository Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Add name and value
4. Reference with `${{ secrets.SECRET_NAME }}`

---

#### Example 9: PR Comments with Results

```yaml
# .github/workflows/pr-comment.yml
name: PR Tests

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    
    permissions:
      pull-requests: write  # Allow commenting on PRs
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm test -- --coverage --json --outputFile=coverage.json
      
      # Comment coverage results on PR
      - name: Comment coverage
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const coverage = JSON.parse(fs.readFileSync('coverage.json'));
            const comment = `
            ## Test Coverage Report
            - **Statements:** ${coverage.total.statements.pct}%
            - **Branches:** ${coverage.total.branches.pct}%
            - **Functions:** ${coverage.total.functions.pct}%
            - **Lines:** ${coverage.total.lines.pct}%
            `;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

---

#### Example 10: Automated Dependency Updates

```yaml
# .github/workflows/auto-merge-dependabot.yml
name: Auto-merge Dependabot

on:
  pull_request:
    branches: [main]

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    
    # Only run on Dependabot PRs
    if: github.actor == 'dependabot[bot]'
    
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v1
      
      - name: Auto-approve
        run: gh pr review --approve "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      # Only auto-merge patch and minor updates
      - name: Auto-merge
        if: steps.metadata.outputs.update-type != 'version-update:semver-major'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

#### Example 11: Lighthouse CI

```yaml
# .github/workflows/lighthouse.yml
name: Lighthouse CI

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      # Run Lighthouse CI
      - name: Run Lighthouse CI
        run: |
          npm install -g @lhci/cli
          lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
      
      # Upload results
      - name: Upload Lighthouse results
        uses: actions/upload-artifact@v3
        with:
          name: lighthouse-results
          path: .lighthouseci/
```

**lighthouserc.json:**
```json
{
  "ci": {
    "collect": {
      "staticDistDir": "./dist"
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", {"minScore": 0.9}],
        "categories:accessibility": ["error", {"minScore": 0.9}],
        "categories:best-practices": ["error", {"minScore": 0.9}],
        "categories:seo": ["error", {"minScore": 0.9}]
      }
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
```

---

#### Common GitHub Actions for React

| Action | Purpose | Usage |
|--------|---------|-------|
| `actions/checkout@v4` | Clone repository | Always first step |
| `actions/setup-node@v4` | Setup Node.js | Specify version, enable caching |
| `actions/cache@v3` | Cache dependencies | Speed up builds |
| `actions/upload-artifact@v3` | Store build outputs | Share between jobs |
| `codecov/codecov-action@v3` | Upload coverage | Integrate with Codecov |
| `peter-evans/create-pull-request@v5` | Create PRs | Automated updates |
| `JamesIves/github-pages-deploy-action@v4` | Deploy to GitHub Pages | Static site hosting |
| `amondnet/vercel-action@v25` | Deploy to Vercel | Automatic deployments |

---

#### GitHub Actions Best Practices

**1. Use specific versions**
```yaml
# ❌ Don't use latest
- uses: actions/checkout@latest

# ✅ Use specific version
- uses: actions/checkout@v4
```

**2. Fail fast**
```yaml
# Run quick checks before slow ones
steps:
  - run: npm run lint          # Fast (~5s)
  - run: npm run type-check    # Fast (~10s)
  - run: npm test              # Medium (~30s)
  - run: npm run test:e2e      # Slow (~2min)
```

**3. Use caching**
```yaml
- uses: actions/setup-node@v4
  with:
    cache: 'npm'  # Always enable
```

**4. Minimize secrets exposure**
```yaml
# ❌ Don't expose secrets unnecessarily
- run: echo ${{ secrets.API_KEY }}

# ✅ Only use where needed
- run: npm run deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}
```

**5. Use concurrency to cancel outdated runs**
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Cancel outdated runs
```

---

#### Troubleshooting

**Problem:** Workflow doesn't trigger

**Solutions:**
- Check branch name matches trigger
- Verify workflow file is in `.github/workflows/`
- Ensure YAML is valid
- Check repository permissions

**Problem:** Tests fail in CI but pass locally

**Solutions:**
- Different Node versions (specify in workflow)
- Missing environment variables
- Timezone differences
- File path case sensitivity (CI is Linux, local might be Windows/Mac)

**Problem:** Slow builds

**Solutions:**
- Enable caching (`cache: 'npm'`)
- Use `npm ci` instead of `npm install`
- Parallelize jobs
- Use matrix sparingly (costs add up)
- Cache Playwright browsers
- Use smaller runners or self-hosted

---

### 3. Other CI/CD Platforms

While GitHub Actions is popular, many companies use other CI/CD platforms. Understanding multiple platforms makes you more valuable.

---

#### CircleCI

**Overview:** Fast, Docker-focused CI/CD platform with excellent caching and parallelization.

**Strengths:**
- ✅ Very fast builds (optimized infrastructure)
- ✅ Excellent Docker support
- ✅ Powerful caching (dependencies, Docker layers)
- ✅ Test parallelization built-in
- ✅ SSH debugging into failed builds
- ✅ Orbs (reusable config packages)

**Configuration file:** `.circleci/config.yml`

**Example 1: Basic React CI**

```yaml
# .circleci/config.yml
version: 2.1

# Use Node orb for common Node.js tasks
orbs:
  node: circleci/node@5.1

jobs:
  test:
    docker:
      - image: cimg/node:20.10
    
    steps:
      - checkout
      
      # Install with caching
      - node/install-packages:
          pkg-manager: npm
      
      - run:
          name: Lint
          command: npm run lint
      
      - run:
          name: Type check
          command: npm run type-check
      
      - run:
          name: Run tests
          command: npm test -- --coverage
      
      - run:
          name: Build
          command: npm run build
      
      # Store test results
      - store_test_results:
          path: test-results
      
      # Store artifacts
      - store_artifacts:
          path: coverage
          destination: coverage

workflows:
  test-workflow:
    jobs:
      - test
```

**Example 2: Parallel Testing**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.1

jobs:
  test:
    docker:
      - image: cimg/node:20.10
    
    # Run 4 parallel containers
    parallelism: 4
    
    steps:
      - checkout
      - node/install-packages
      
      - run:
          name: Run tests in parallel
          command: |
            # Split tests across parallel containers
            TESTFILES=$(circleci tests glob "src/**/*.test.{js,jsx,ts,tsx}" | circleci tests split --split-by=timings)
            npm test -- $TESTFILES
      
      - store_test_results:
          path: test-results

workflows:
  version: 2
  build-and-test:
    jobs:
      - test
```

**Example 3: Multi-stage Pipeline**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.1

jobs:
  build:
    docker:
      - image: cimg/node:20.10
    steps:
      - checkout
      - node/install-packages
      - run: npm run build
      - persist_to_workspace:
          root: .
          paths:
            - dist
  
  test:
    docker:
      - image: cimg/node:20.10
    steps:
      - checkout
      - node/install-packages
      - run: npm test
  
  e2e:
    docker:
      - image: mcr.microsoft.com/playwright:v1.40.0
    steps:
      - checkout
      - attach_workspace:
          at: .
      - run: npm ci
      - run: npm run test:e2e
      - store_artifacts:
          path: playwright-report
  
  deploy:
    docker:
      - image: cimg/node:20.10
    steps:
      - checkout
      - attach_workspace:
          at: .
      - run:
          name: Deploy to Vercel
          command: npx vercel --prod --token=$VERCEL_TOKEN

workflows:
  build-test-deploy:
    jobs:
      - build
      - test
      - e2e:
          requires:
            - build
      - deploy:
          requires:
            - build
            - test
            - e2e
          filters:
            branches:
              only: main
```

**CircleCI-specific features:**

- **SSH into builds:** Debug failures in real-time
- **Insights:** View test performance trends
- **Test splitting:** Automatically distribute tests across parallel nodes
- **Docker layer caching:** Speed up Docker builds significantly

---

#### GitLab CI/CD

**Overview:** Built into GitLab, making it seamless for GitLab-hosted projects.

**Strengths:**
- ✅ Built into GitLab (no third-party service)
- ✅ Free tier includes 400 CI minutes/month
- ✅ Auto DevOps (zero-config CI/CD)
- ✅ Integrated container registry
- ✅ Review apps (preview environments)
- ✅ Kubernetes integration

**Configuration file:** `.gitlab-ci.yml`

**Example 1: Basic Pipeline**

```yaml
# .gitlab-ci.yml
image: node:20

# Define stages
stages:
  - install
  - test
  - build
  - deploy

# Cache node_modules
cache:
  paths:
    - node_modules/

# Install dependencies
install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

# Lint and type check
lint:
  stage: test
  script:
    - npm run lint
    - npm run type-check
  dependencies:
    - install

# Run tests
test:
  stage: test
  script:
    - npm test -- --coverage
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  dependencies:
    - install

# Build application
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
  dependencies:
    - install
  only:
    - main
    - merge_requests

# Deploy to production
deploy:
  stage: deploy
  script:
    - npm run deploy
  environment:
    name: production
    url: https://myapp.com
  only:
    - main
```

**Example 2: Multi-environment Deployment**

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  NODE_ENV: production

build:
  stage: build
  image: node:20
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test

deploy_staging:
  stage: deploy
  script:
    - npm run deploy:staging
  environment:
    name: staging
    url: https://staging.myapp.com
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - npm run deploy:production
  environment:
    name: production
    url: https://myapp.com
  when: manual  # Require manual approval
  only:
    - main
```

**Example 3: Review Apps (Preview Environments)**

```yaml
# .gitlab-ci.yml
deploy_review:
  stage: deploy
  script:
    - npm run build
    - deploy_to_preview $CI_COMMIT_REF_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    url: https://$CI_COMMIT_REF_SLUG.preview.myapp.com
    on_stop: stop_review
  only:
    - merge_requests

stop_review:
  stage: deploy
  script:
    - cleanup_preview $CI_COMMIT_REF_SLUG
  environment:
    name: review/$CI_COMMIT_REF_NAME
    action: stop
  when: manual
  only:
    - merge_requests
```

---

#### Jenkins

**Overview:** Open-source automation server, highly customizable but requires self-hosting.

**Strengths:**
- ✅ Extremely flexible (thousands of plugins)
- ✅ Self-hosted (full control)
- ✅ Enterprise-ready
- ✅ Groovy-based pipelines (powerful scripting)
- ✅ Blue Ocean UI (modern interface)

**Configuration:** Jenkinsfile (Groovy DSL)

**Example: Declarative Pipeline**

```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'node:20'
        }
    }
    
    environment {
        CI = 'true'
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test -- --coverage'
            }
            post {
                always {
                    junit 'test-results/**/*.xml'
                    publishHTML([
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh 'npm run deploy'
            }
        }
    }
    
    post {
        failure {
            mail to: 'team@example.com',
                 subject: "Build Failed: ${env.JOB_NAME}",
                 body: "Build ${env.BUILD_NUMBER} failed"
        }
    }
}
```

**Jenkins is best for:**
- Large enterprises with on-premise requirements
- Complex build processes requiring custom plugins
- Teams with dedicated DevOps engineers
- Integration with legacy systems

---

#### Travis CI

**Overview:** One of the first CI services for GitHub, now less popular but still used.

**Configuration file:** `.travis.yml`

```yaml
# .travis.yml
language: node_js

node_js:
  - 18
  - 20

cache:
  directories:
    - node_modules

install:
  - npm ci

script:
  - npm run lint
  - npm test -- --coverage
  - npm run build

after_success:
  - npm run codecov

deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  local_dir: dist
  on:
    branch: main
```

---

#### Azure Pipelines

**Overview:** Microsoft's CI/CD platform, integrated with Azure DevOps.

**Strengths:**
- ✅ Generous free tier (10 parallel jobs, unlimited minutes for public repos)
- ✅ Native Azure integration
- ✅ Windows, macOS, Linux runners
- ✅ YAML and visual designer

**Configuration file:** `azure-pipelines.yml`

```yaml
# azure-pipelines.yml
trigger:
  - main
  - develop

pool:
  vmImage: 'ubuntu-latest'

variables:
  nodeVersion: '20.x'

stages:
  - stage: Build
    jobs:
      - job: BuildAndTest
        steps:
          - task: NodeTool@0
            inputs:
              versionSpec: $(nodeVersion)
            displayName: 'Install Node.js'
          
          - script: npm ci
            displayName: 'Install dependencies'
          
          - script: npm run lint
            displayName: 'Lint code'
          
          - script: npm test -- --coverage
            displayName: 'Run tests'
          
          - script: npm run build
            displayName: 'Build application'
          
          - task: PublishTestResults@2
            inputs:
              testResultsFormat: 'JUnit'
              testResultsFiles: '**/test-results/*.xml'
          
          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: 'Cobertura'
              summaryFileLocation: '$(System.DefaultWorkingDirectory)/coverage/cobertura-coverage.xml'

  - stage: Deploy
    dependsOn: Build
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProduction
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - script: npm run deploy
                  displayName: 'Deploy to production'
```

---

#### Platform Comparison

| Platform | Best For | Pricing | Learning Curve | Speed |
|----------|----------|---------|----------------|-------|
| **GitHub Actions** | GitHub projects, open source | Free tier generous | Easy | Fast |
| **CircleCI** | Speed, Docker workflows | Paid (limited free) | Medium | Very fast |
| **GitLab CI/CD** | GitLab projects, Kubernetes | Free tier good | Easy | Fast |
| **Jenkins** | Enterprises, on-premise | Free (self-host costs) | Hard | Varies |
| **Travis CI** | Legacy projects | Paid | Easy | Medium |
| **Azure Pipelines** | Azure ecosystem, .NET | Free tier excellent | Medium | Fast |

---

#### When to Choose Each Platform

**GitHub Actions:**
- GitHub-hosted projects
- Want native integration
- Need marketplace actions
- Open source projects

**CircleCI:**
- Speed is critical
- Heavy Docker usage
- Need test parallelization
- Budget for premium features

**GitLab CI/CD:**
- Using GitLab for version control
- Want all-in-one platform
- Need Kubernetes integration
- Want review apps

**Jenkins:**
- Enterprise with self-hosting requirements
- Complex custom workflows
- Need specific plugins
- Integration with legacy systems

**Azure Pipelines:**
- Microsoft/Azure ecosystem
- Cross-platform builds (Windows, macOS, Linux)
- Enterprise Microsoft shop
- Need generous free tier

---

#### Migration Tips

**Moving from Travis CI → GitHub Actions:**
- `.travis.yml` → `.github/workflows/ci.yml`
- Similar YAML structure
- Replace `script:` with `run:`
- Travis env vars → GitHub secrets

**Moving from CircleCI → GitHub Actions:**
- `.circleci/config.yml` → `.github/workflows/ci.yml`
- Orbs → Marketplace actions
- `persist_to_workspace` → `actions/upload-artifact`
- Similar caching concepts

**Moving from Jenkins → GitHub Actions:**
- Jenkinsfile → `.github/workflows/ci.yml`
- Groovy DSL → YAML
- Jenkins plugins → GitHub Actions
- Self-hosted agents possible in both

---

### 4. Test Automation in Pipelines

Running tests in CI is the primary value of continuous integration. Automated testing ensures every code change is validated before merging, preventing bugs from reaching production.

#### Running Tests in CI

**Basic Jest/Vitest configuration:**

```yaml
# .github/workflows/test.yml
name: Test Suite

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm test -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/coverage-final.json
          fail_ci_if_error: true
```

**Key testing strategies:**

1. **Fast feedback loop:** Run fast tests (unit, component) on every commit
2. **Comprehensive validation:** Run slow tests (E2E) only on main branch or PRs
3. **Fail fast:** Configure tests to stop on first failure in CI
4. **Isolation:** Each test job should be independent and repeatable

#### Parallel Testing

Run tests concurrently to reduce pipeline time. This is critical for large test suites.

**Jest parallel testing (built-in):**

```javascript
// jest.config.js
module.exports = {
  // Jest runs tests in parallel by default
  maxWorkers: '50%', // Use 50% of available CPU cores
  
  // For CI, use all available cores
  ...(process.env.CI && { maxWorkers: '100%' }),
  
  // Shard tests across multiple CI jobs
  shard: process.env.SHARD ? JSON.parse(process.env.SHARD) : undefined,
};
```

**GitHub Actions matrix for parallel test shards:**

```yaml
# .github/workflows/parallel-tests.yml
name: Parallel Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        # Split tests across 4 parallel jobs
        shard: [1, 2, 3, 4]
        total-shards: [4]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run test shard ${{ matrix.shard }}/${{ matrix.total-shards }}
        run: |
          npm test -- \
            --shard=${{ matrix.shard }}/${{ matrix.total-shards }} \
            --coverage
        env:
          SHARD: '{"shardIndex": ${{ matrix.shard }}, "shardCount": ${{ matrix.total-shards }}}'
      
      - name: Upload coverage artifact
        uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.shard }}
          path: coverage/
          retention-days: 1

  # Combine coverage from all shards
  combine-coverage:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Download all coverage artifacts
        uses: actions/download-artifact@v4
        with:
          path: coverage-shards/
      
      - name: Merge coverage reports
        run: npx nyc merge coverage-shards/ coverage/coverage-final.json
      
      - name: Upload to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
```

**Playwright parallel testing:**

```yaml
# .github/workflows/e2e.yml
name: E2E Tests

on: [pull_request]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        # Run tests across 3 parallel jobs
        shard: [1, 2, 3]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright Browsers
        run: npx playwright install --with-deps chromium
      
      - name: Run Playwright tests
        run: npx playwright test --shard=${{ matrix.shard }}/3
      
      - name: Upload Playwright Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report-${{ matrix.shard }}
          path: playwright-report/
          retention-days: 30
```

**CircleCI parallel testing:**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.1.0

jobs:
  test:
    docker:
      - image: cimg/node:20.11
    
    parallelism: 4  # Run 4 parallel containers
    
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      
      - run:
          name: Run tests
          command: |
            # CircleCI splits tests automatically based on timing data
            TESTFILES=$(circleci tests glob "src/**/*.test.{js,jsx,ts,tsx}" | \
                        circleci tests split --split-by=timings)
            npm test -- $TESTFILES --coverage
      
      - store_test_results:
          path: test-results
      
      - store_artifacts:
          path: coverage

workflows:
  test-workflow:
    jobs:
      - test
```

#### Test Reporting

Proper test reporting provides visibility into test results, trends, and failures.

**JUnit XML format (industry standard):**

```javascript
// jest.config.js
module.exports = {
  reporters: [
    'default',
    [
      'jest-junit',
      {
        outputDirectory: 'test-results',
        outputName: 'junit.xml',
        classNameTemplate: '{classname}',
        titleTemplate: '{title}',
        ancestorSeparator: ' › ',
        suiteNameTemplate: '{filepath}',
      },
    ],
  ],
};
```

**GitHub Actions test reporting:**

```yaml
# .github/workflows/test-report.yml
name: Test with Reporting

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm test
      
      # Publish test results as GitHub Check
      - name: Publish Test Results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: |
            test-results/**/*.xml
      
      # Comment on PR with test summary
      - name: Test Report
        uses: dorny/test-reporter@v1
        if: always()
        with:
          name: Jest Tests
          path: test-results/junit.xml
          reporter: jest-junit
          fail-on-error: true
```

**Custom PR comment with test results:**

```yaml
# .github/workflows/pr-comment.yml
name: Test Summary Comment

on: [pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      
      - name: Run tests with JSON output
        run: npm test -- --json --outputFile=test-results.json
        continue-on-error: true
      
      - name: Comment PR with results
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('test-results.json', 'utf8'));
            
            const { numTotalTests, numPassedTests, numFailedTests, testResults } = results;
            const failures = testResults
              .filter(r => r.status === 'failed')
              .map(r => `- ❌ ${r.name}`)
              .join('\n');
            
            const body = `
            ## Test Results
            
            - ✅ Passed: ${numPassedTests}
            - ❌ Failed: ${numFailedTests}
            - 📊 Total: ${numTotalTests}
            
            ${failures ? `### Failures\n${failures}` : ''}
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body
            });
```

#### Code Coverage

Track test coverage to ensure adequate testing and identify untested code.

**Coverage thresholds in Jest:**

```javascript
// jest.config.js
module.exports = {
  collectCoverage: true,
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.tsx',
    '!src/index.tsx',
  ],
  
  // Fail if coverage drops below these thresholds
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
    // Stricter rules for critical files
    './src/lib/auth.ts': {
      branches: 100,
      functions: 100,
      lines: 100,
      statements: 100,
    },
  },
};
```

**Upload to Codecov:**

```yaml
# .github/workflows/coverage.yml
name: Coverage

on: [push, pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test -- --coverage
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/coverage-final.json
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: true
          verbose: true
```

**Upload to Coveralls:**

```yaml
# Alternative: Coveralls
- name: Upload coverage to Coveralls
  uses: coverallsapp/github-action@v2
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    path-to-lcov: ./coverage/lcov.info
```

**Coverage PR comments:**

```yaml
# .github/workflows/coverage-comment.yml
name: Coverage Comment

on: [pull_request]

jobs:
  coverage:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test -- --coverage
      
      # Generate coverage badge and comment
      - name: Coverage Report
        uses: romeovs/lcov-reporter-action@v0.3.1
        with:
          lcov-file: ./coverage/lcov.info
          github-token: ${{ secrets.GITHUB_TOKEN }}
          delete-old-comments: true
```

**Coverage diff (show only changed files):**

```yaml
# Only report coverage for changed files
- name: Coverage Diff
  uses: MishaKav/jest-coverage-comment@v1.0.23
  with:
    coverage-summary-path: ./coverage/coverage-summary.json
    title: Coverage Report
    badge-title: Coverage
    hide-comment: false
    create-new-comment: false
    hide-summary: false
```

#### Handling Flaky Tests

Flaky tests (tests that sometimes pass, sometimes fail) are a major CI/CD pain point.

**Detection strategies:**

```yaml
# .github/workflows/detect-flaky.yml
name: Detect Flaky Tests

on:
  schedule:
    # Run nightly to detect flaky tests
    - cron: '0 2 * * *'

jobs:
  flaky-detection:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      
      # Run tests multiple times to detect flakiness
      - name: Run tests 10 times
        run: |
          for i in {1..10}; do
            echo "Run $i"
            npm test || echo "FAIL $i" >> failures.txt
          done
      
      - name: Report flaky tests
        if: failure()
        run: |
          if [ -f failures.txt ]; then
            echo "Flaky tests detected!"
            cat failures.txt
            exit 1
          fi
```

**Retry configuration:**

```javascript
// jest.config.js
module.exports = {
  // Retry failed tests
  testRetries: process.env.CI ? 2 : 0,
};
```

```javascript
// playwright.config.ts
export default {
  retries: process.env.CI ? 2 : 0,
  
  // Reporter shows which tests were retried
  reporter: [
    ['html'],
    ['junit', { outputFile: 'test-results/junit.xml' }],
  ],
};
```

**Quarantine flaky tests:**

```javascript
// Mark flaky tests to skip in CI
describe('UserProfile', () => {
  // Skip in CI until fixed
  const testFn = process.env.CI ? it.skip : it;
  
  testFn('should handle race condition', async () => {
    // Flaky test that needs fixing
  });
});
```

#### Test Performance Monitoring

Track test execution time to identify slow tests and optimize CI.

**Jest slow test reporter:**

```javascript
// jest.config.js
module.exports = {
  reporters: [
    'default',
    [
      'jest-slow-test-reporter',
      {
        numTests: 10,  // Report 10 slowest tests
        warnOnSlowerThan: 300,  // Warn if test takes >300ms
        color: true,
      },
    ],
  ],
};
```

**Track timing trends:**

```yaml
# .github/workflows/test-performance.yml
name: Test Performance

on: [push]

jobs:
  performance:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      
      - name: Run tests with timing
        run: npm test -- --verbose --testLocationInResults
      
      - name: Store timing data
        uses: benchmark-action/github-action-benchmark@v1
        with:
          tool: 'customSmallerIsBetter'
          output-file-path: test-results/timings.json
          github-token: ${{ secrets.GITHUB_TOKEN }}
          auto-push: true
```

**Example output:**

```
Slowest tests:
  1. UserProfile integration test - 2.3s
  2. E2E checkout flow - 1.8s
  3. Image processing - 1.2s
  4. API mock server setup - 890ms
  5. Database seeding - 750ms
```

---

### 5. Build Optimization and Deployment

Optimizing builds and deployments reduces pipeline time, cloud costs, and improves developer experience.

#### Dependency Caching

Cache `node_modules` to avoid reinstalling dependencies on every build.

**GitHub Actions caching:**

```yaml
# .github/workflows/build.yml
name: Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Cache node_modules based on package-lock.json hash
      - name: Setup Node.js with caching
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'  # Automatically caches based on package-lock.json
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      # Cache the build output
      - name: Cache build artifacts
        uses: actions/cache@v4
        with:
          path: |
            dist/
            .next/cache/
          key: ${{ runner.os }}-build-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-build-
```

**Manual cache configuration:**

```yaml
# More control over caching
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-

- name: Cache Playwright browsers
  uses: actions/cache@v4
  with:
    path: ~/.cache/ms-playwright
    key: ${{ runner.os }}-playwright-${{ hashFiles('**/package-lock.json') }}
```

**CircleCI caching:**

```yaml
# .circleci/config.yml
version: 2.1

orbs:
  node: circleci/node@5.1.0

jobs:
  build:
    docker:
      - image: cimg/node:20.11
    
    steps:
      - checkout
      
      # Restore cached dependencies
      - restore_cache:
          keys:
            - v1-deps-{{ checksum "package-lock.json" }}
            - v1-deps-
      
      - run: npm ci
      
      # Save cache for next run
      - save_cache:
          key: v1-deps-{{ checksum "package-lock.json" }}
          paths:
            - node_modules
            - ~/.npm
            - ~/.cache
      
      - run: npm run build
      
      # Cache build artifacts
      - save_cache:
          key: v1-build-{{ .Environment.CIRCLE_SHA1 }}
          paths:
            - dist
```

**GitLab CI caching:**

```yaml
# .gitlab-ci.yml
cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/
    - .npm/

build:
  stage: build
  script:
    - npm ci --cache .npm --prefer-offline
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
```

#### Build Optimization Strategies

**1. Incremental builds (Next.js, Vite):**

```yaml
# .github/workflows/nextjs-build.yml
name: Next.js Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      # Restore Next.js build cache
      - name: Cache Next.js build
        uses: actions/cache@v4
        with:
          path: |
            ${{ github.workspace }}/.next/cache
          key: ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('**/*.js', '**/*.jsx', '**/*.ts', '**/*.tsx') }}
          restore-keys: |
            ${{ runner.os }}-nextjs-${{ hashFiles('**/package-lock.json') }}-
      
      - name: Build Next.js app
        run: npm run build
        env:
          # Enable Next.js build cache
          NEXT_TELEMETRY_DISABLED: 1
```

**2. Docker layer caching:**

```yaml
# .github/workflows/docker-build.yml
name: Docker Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:latest
          # Cache Docker layers
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

**Optimized Dockerfile:**

```dockerfile
# Dockerfile
FROM node:20-alpine AS deps

WORKDIR /app

# Copy only dependency files first (cached unless dependencies change)
COPY package.json package-lock.json ./
RUN npm ci --only=production

# Build stage
FROM node:20-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Production stage
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV production

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public
COPY package.json ./

EXPOSE 3000

CMD ["npm", "start"]
```

**3. Parallel builds:**

```yaml
# .github/workflows/parallel-build.yml
name: Parallel Build

on: [push]

jobs:
  # Build multiple targets in parallel
  build-app:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build:app
      - uses: actions/upload-artifact@v4
        with:
          name: app-build
          path: dist/app/
  
  build-admin:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build:admin
      - uses: actions/upload-artifact@v4
        with:
          name: admin-build
          path: dist/admin/
  
  # Deploy after both builds complete
  deploy:
    needs: [build-app, build-admin]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
      - run: echo "Deploy artifacts"
```

#### Deployment Strategies

Modern React apps typically deploy to static hosting platforms or containerized environments.

**Vercel deployment:**

```yaml
# .github/workflows/vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Vercel CLI
        run: npm install --global vercel@latest
      
      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Build Project Artifacts
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Deploy to Vercel
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
        env:
          VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
          VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
      
      # Preview deployments for PRs
      - name: Deploy Preview
        if: github.event_name == 'pull_request'
        run: |
          DEPLOYMENT_URL=$(vercel deploy --token=${{ secrets.VERCEL_TOKEN }})
          echo "Preview URL: $DEPLOYMENT_URL"
```

**Netlify deployment:**

```yaml
# .github/workflows/netlify.yml
name: Deploy to Netlify

on:
  push:
    branches: [main]
  pull_request:

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      # Production deployment
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v2.1
        with:
          publish-dir: './dist'
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: true
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
        timeout-minutes: 1
```

**AWS S3 + CloudFront deployment:**

```yaml
# .github/workflows/aws-s3.yml
name: Deploy to AWS S3

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Sync to S3
        run: |
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET }} \
            --delete \
            --cache-control "public, max-age=31536000, immutable" \
            --exclude "*.html" \
            --exclude "service-worker.js"
          
          # HTML files with shorter cache
          aws s3 sync dist/ s3://${{ secrets.S3_BUCKET }} \
            --exclude "*" \
            --include "*.html" \
            --include "service-worker.js" \
            --cache-control "public, max-age=0, must-revalidate"
      
      - name: Invalidate CloudFront
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

**GitHub Pages deployment:**

```yaml
# .github/workflows/gh-pages.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
        env:
          # Set base path for GitHub Pages
          PUBLIC_URL: /${{ github.event.repository.name }}
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'
      
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Docker to Azure Container Apps:**

```yaml
# .github/workflows/azure-container-apps.yml
name: Deploy to Azure Container Apps

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Login to Azure
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Login to Azure Container Registry
        uses: azure/docker-login@v1
        with:
          login-server: ${{ secrets.REGISTRY_LOGIN_SERVER }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}
      
      - name: Build and push image
        run: |
          docker build -t ${{ secrets.REGISTRY_LOGIN_SERVER }}/myapp:${{ github.sha }} .
          docker push ${{ secrets.REGISTRY_LOGIN_SERVER }}/myapp:${{ github.sha }}
      
      - name: Deploy to Container Apps
        uses: azure/container-apps-deploy-action@v1
        with:
          containerAppName: my-react-app
          resourceGroup: my-resource-group
          imageToDeploy: ${{ secrets.REGISTRY_LOGIN_SERVER }}/myapp:${{ github.sha }}
```

#### Environment Management

**1. Environment variables in CI:**

```yaml
# .github/workflows/env-management.yml
name: Environment Management

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      
      - run: npm ci
      
      - name: Build with environment variables
        run: npm run build
        env:
          # Public variables (embedded in build)
          VITE_API_URL: ${{ secrets.API_URL }}
          VITE_APP_VERSION: ${{ github.sha }}
          
          # Build-time variables
          NODE_ENV: production
          
          # Different per branch
          VITE_ENV: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
```

**2. Separate environments:**

```yaml
# .github/workflows/multi-env.yml
name: Multi-Environment Deploy

on:
  push:
    branches: [main, staging, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      
      # Determine environment
      - name: Set environment
        id: env
        run: |
          if [[ "${{ github.ref }}" == "refs/heads/main" ]]; then
            echo "env=production" >> $GITHUB_OUTPUT
            echo "url=https://app.example.com" >> $GITHUB_OUTPUT
          elif [[ "${{ github.ref }}" == "refs/heads/staging" ]]; then
            echo "env=staging" >> $GITHUB_OUTPUT
            echo "url=https://staging.example.com" >> $GITHUB_OUTPUT
          else
            echo "env=development" >> $GITHUB_OUTPUT
            echo "url=https://dev.example.com" >> $GITHUB_OUTPUT
          fi
      
      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ steps.env.outputs.url }}/api
          VITE_ENV: ${{ steps.env.outputs.env }}
      
      - name: Deploy to ${{ steps.env.outputs.env }}
        run: echo "Deploying to ${{ steps.env.outputs.url }}"
```

**3. Using GitHub Environments:**

```yaml
# .github/workflows/github-environments.yml
name: Deploy with Environments

on:
  push:
    branches: [main]

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
        env:
          VITE_API_URL: ${{ vars.API_URL }}
          VITE_API_KEY: ${{ secrets.API_KEY }}
      - run: echo "Deploy to staging"
  
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
        env:
          VITE_API_URL: ${{ vars.API_URL }}
          VITE_API_KEY: ${{ secrets.API_KEY }}
      - run: echo "Deploy to production"
```

**4. .env file management:**

```yaml
# .github/workflows/dotenv.yml
name: Environment File Management

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      
      # Create .env file from secrets
      - name: Create .env file
        run: |
          cat << EOF > .env
          VITE_API_URL=${{ secrets.API_URL }}
          VITE_ANALYTICS_ID=${{ secrets.ANALYTICS_ID }}
          VITE_FEATURE_FLAGS=${{ secrets.FEATURE_FLAGS }}
          EOF
      
      - run: npm ci
      - run: npm run build
      
      # Clean up sensitive files
      - name: Remove .env
        run: rm .env
```

#### Blue-Green and Canary Deployments

**Blue-Green deployment with Vercel:**

```yaml
# .github/workflows/blue-green.yml
name: Blue-Green Deployment

on:
  push:
    branches: [main]

jobs:
  deploy-green:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      
      # Deploy to "green" environment
      - name: Deploy green environment
        run: |
          DEPLOYMENT_URL=$(vercel deploy --token=${{ secrets.VERCEL_TOKEN }})
          echo "GREEN_URL=$DEPLOYMENT_URL" >> $GITHUB_ENV
      
      # Run smoke tests against green
      - name: Smoke test green environment
        run: |
          npm run test:smoke -- --url=${{ env.GREEN_URL }}
      
      # If tests pass, promote to production
      - name: Promote to production
        run: |
          vercel alias ${{ env.GREEN_URL }} production.example.com --token=${{ secrets.VERCEL_TOKEN }}
```

**Canary deployment:**

```yaml
# .github/workflows/canary.yml
name: Canary Deployment

on:
  push:
    branches: [main]

jobs:
  deploy-canary:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      
      # Deploy canary (10% traffic)
      - name: Deploy canary
        run: |
          # Deploy to canary environment
          vercel deploy --token=${{ secrets.VERCEL_TOKEN }}
      
      # Monitor for 10 minutes
      - name: Monitor canary metrics
        run: |
          sleep 600  # Wait 10 minutes
          # Check error rates, performance metrics
          npm run monitor:canary
      
      # If metrics are good, roll out to 100%
      - name: Full rollout
        run: |
          vercel promote --token=${{ secrets.VERCEL_TOKEN }}
```

#### Rollback Strategies

**Automatic rollback on failure:**

```yaml
# .github/workflows/deploy-with-rollback.yml
name: Deploy with Rollback

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Save current production version
      - name: Get current version
        id: current
        run: |
          CURRENT=$(curl https://api.vercel.com/v1/deployments \
            -H "Authorization: Bearer ${{ secrets.VERCEL_TOKEN }}" | \
            jq -r '.deployments[0].uid')
          echo "version=$CURRENT" >> $GITHUB_OUTPUT
      
      - run: npm ci
      - run: npm run build
      
      # Deploy new version
      - name: Deploy
        id: deploy
        run: |
          NEW_DEPLOYMENT=$(vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }})
          echo "url=$NEW_DEPLOYMENT" >> $GITHUB_OUTPUT
      
      # Health check
      - name: Health check
        id: health
        run: |
          response=$(curl -s -o /dev/null -w "%{http_code}" ${{ steps.deploy.outputs.url }}/health)
          if [ $response -ne 200 ]; then
            echo "healthy=false" >> $GITHUB_OUTPUT
            exit 1
          fi
        continue-on-error: true
      
      # Rollback if health check fails
      - name: Rollback on failure
        if: steps.health.outputs.healthy == 'false'
        run: |
          echo "Health check failed, rolling back"
          vercel alias ${{ steps.current.outputs.version }} production.example.com \
            --token=${{ secrets.VERCEL_TOKEN }}
          exit 1
```

---

## Examples

All examples are integrated throughout sections 2-5 above, including:

**GitHub Actions workflows:**
- Basic CI pipeline with testing and linting
- Build artifacts with upload/download
- Matrix builds for multiple Node versions
- E2E testing with Playwright
- Parallel job execution
- Conditional workflows
- Advanced caching strategies
- Secrets management
- PR comment automation
- Dependabot auto-merge
- Lighthouse CI integration

**Other platforms:**
- CircleCI: Basic CI, parallel testing, multi-stage pipelines
- GitLab CI/CD: Multi-environment, review apps
- Jenkins: Declarative pipelines
- Travis CI: Basic configuration
- Azure Pipelines: Full YAML pipeline

**Test automation:**
- Jest/Vitest parallel test execution
- Playwright E2E test sharding
- Test reporting with JUnit XML
- Code coverage with Codecov/Coveralls
- Flaky test detection and retry strategies
- Test performance monitoring

**Deployment examples:**
- Vercel deployment with preview URLs
- Netlify deployment with PR previews
- AWS S3 + CloudFront with cache control
- GitHub Pages deployment
- Azure Container Apps
- Docker multi-stage builds
- Blue-Green deployments
- Canary releases
- Automatic rollback on failure

**Build optimization:**
- Dependency caching (npm, Playwright browsers)
- Next.js incremental builds
- Docker layer caching
- Parallel builds for multiple targets

**Environment management:**
- Multi-environment workflows (dev/staging/prod)
- GitHub Environments with protection rules
- Environment variable management
- .env file generation from secrets

---

## Common pitfalls

### 1. **Slow pipelines that waste time and money**

**Problem:** Pipeline takes 20+ minutes, developers wait or context-switch.

**Why it happens:**
- No caching of dependencies
- Sequential instead of parallel jobs
- Running all tests on every commit
- Slow E2E tests not optimized
- Large Docker images

**Solution:**
```yaml
# Cache aggressively
- uses: actions/setup-node@v4
  with:
    cache: 'npm'

# Run jobs in parallel
jobs:
  lint:
    runs-on: ubuntu-latest
    steps: [checkout, lint]
  
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3, 4]  # Parallel test execution
  
  build:
    runs-on: ubuntu-latest
    steps: [checkout, build]

# Run expensive E2E tests only on PR, not every commit
on:
  pull_request:
  push:
    branches: [main]
```

**Impact:** Reduces pipeline time from 20 minutes to 5 minutes, saving developer time and CI costs.

---

### 2. **Secrets leaked in logs or code**

**Problem:** API keys, tokens, or credentials exposed in pipeline logs or committed to repo.

**Why it happens:**
- Echoing secrets in scripts
- Secrets in environment variables printed by tools
- Hardcoded secrets in workflow files
- `.env` files committed to git

**Solution:**
```yaml
# ❌ WRONG - Secret will appear in logs
- run: echo "API_KEY=${{ secrets.API_KEY }}"

# ✅ CORRECT - Use secrets properly
- run: npm run deploy
  env:
    API_KEY: ${{ secrets.API_KEY }}

# Mask sensitive output
- run: |
    echo "::add-mask::${{ secrets.API_KEY }}"
    npm run deploy
```

**Prevention:**
- Use GitHub Secrets, never hardcode
- Add `.env` to `.gitignore`
- Enable secret scanning in repository settings
- Review logs before sharing
- Use `::add-mask::` for dynamic secrets

---

### 3. **Tests pass locally but fail in CI**

**Problem:** "Works on my machine" syndrome.

**Why it happens:**
- Different Node versions
- Missing environment variables
- Timezone differences
- Race conditions in tests
- Dependencies on local files/services

**Solution:**
```yaml
# Match CI environment locally
# .nvmrc
20.11.0

# package.json
"engines": {
  "node": ">=20.11.0"
}

# CI workflow
- uses: actions/setup-node@v4
  with:
    node-version-file: '.nvmrc'

# Run tests in CI mode locally
- run: CI=true npm test
```

**Best practices:**
- Use `npm ci` not `npm install` (locks exact versions)
- Set timezone: `TZ=UTC`
- Mock external services
- Fix flaky tests immediately
- Use Docker for consistency

---

### 4. **Breaking changes deployed to production**

**Problem:** Bad code reaches production, causing outages.

**Why it happens:**
- No required status checks
- Tests not comprehensive
- Auto-merge without review
- Skipping CI with `[skip ci]`
- No staging environment

**Solution:**
```yaml
# GitHub branch protection rules
# Settings → Branches → Add rule
✅ Require a pull request before merging
✅ Require status checks to pass before merging
   - CI / test
   - CI / lint
   - CI / build
✅ Require branches to be up to date
✅ Do not allow bypassing the above settings

# Deploy to staging first
jobs:
  deploy-staging:
    environment: staging
    steps: [deploy]
  
  deploy-production:
    needs: deploy-staging
    environment:
      name: production
      # Require manual approval
    steps: [deploy]
```

---

### 5. **Flaky tests that block deployments**

**Problem:** Tests randomly fail, team starts ignoring CI failures.

**Why it happens:**
- Race conditions in async code
- Hard-coded timeouts
- Dependency on external services
- Test order dependencies
- Browser timing issues in E2E tests

**Solution:**
```javascript
// ❌ WRONG - Flaky due to race condition
test('loads data', () => {
  render(<UserProfile />);
  expect(screen.getByText('John')).toBeInTheDocument();
});

// ✅ CORRECT - Wait for async operation
test('loads data', async () => {
  render(<UserProfile />);
  expect(await screen.findByText('John')).toBeInTheDocument();
});

// ❌ WRONG - Hard-coded timeout
await page.waitForTimeout(3000);

// ✅ CORRECT - Wait for specific condition
await page.waitForSelector('[data-testid="user-profile"]');

// Configure retries for genuinely flaky tests
// playwright.config.ts
export default {
  retries: process.env.CI ? 2 : 0,
};
```

**Prevention:**
- Fix flaky tests immediately, don't retry indefinitely
- Use `waitFor` not `setTimeout`
- Mock network requests
- Run tests in isolation
- Monitor flaky test trends

---

### 6. **Huge Docker images that slow builds**

**Problem:** 2GB Docker images take forever to build and deploy.

**Why it happens:**
- Using full `node` image instead of `node:alpine`
- Including dev dependencies in production
- Not using multi-stage builds
- No layer caching

**Solution:**
```dockerfile
# ❌ WRONG - 1.2GB image
FROM node:20
COPY . .
RUN npm install
RUN npm run build

# ✅ CORRECT - 150MB image
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

**Impact:** Reduces image from 1.2GB to 150MB, faster builds and deploys.

---

### 7. **No rollback strategy when deployments fail**

**Problem:** Bad deployment to production, no quick way to revert.

**Why it happens:**
- Direct deployment without health checks
- No previous version tracking
- Manual deployment process
- No smoke tests

**Solution:**
```yaml
# Save current version before deploying
- name: Get current production version
  id: current
  run: echo "sha=$(git rev-parse HEAD~1)" >> $GITHUB_OUTPUT

# Deploy new version
- name: Deploy
  run: ./deploy.sh

# Health check
- name: Smoke test
  id: health
  run: |
    response=$(curl -f https://app.example.com/health)
    if [ $? -ne 0 ]; then exit 1; fi
  continue-on-error: true

# Auto-rollback on failure
- name: Rollback
  if: steps.health.outcome == 'failure'
  run: |
    git checkout ${{ steps.current.outputs.sha }}
    ./deploy.sh
    exit 1
```

**Best practices:**
- Keep last 5 production versions
- Tag releases: `v1.2.3`
- Use blue-green deployments
- Implement canary releases for gradual rollout

---

### 8. **Ignoring bundle size increases**

**Problem:** App bundle grows from 200KB to 2MB, nobody notices until users complain.

**Why it happens:**
- No bundle size monitoring
- Large dependencies added without review
- Unused code not tree-shaken

**Solution:**
```yaml
# .github/workflows/bundle-size.yml
- name: Check bundle size
  run: npx size-limit

# size-limit config
# .size-limit.json
[
  {
    "path": "dist/**/*.js",
    "limit": "300 KB",
    "gzip": true
  }
]

# Fail CI if bundle exceeds limit
# Comment on PR with size diff
- uses: andresz1/size-limit-action@v1
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
```

---

### 9. **Environment variables not set correctly**

**Problem:** App breaks in production due to missing or wrong env vars.

**Why it happens:**
- Different variables in dev/staging/prod
- Variables not documented
- `.env` files not synced with CI
- Forgetting to update CI secrets

**Solution:**
```javascript
// Validate required env vars at build time
// vite.config.ts
export default defineConfig({
  define: {
    'import.meta.env.VITE_API_URL': JSON.stringify(
      process.env.VITE_API_URL || (() => {
        throw new Error('VITE_API_URL is required');
      })()
    ),
  },
});

// Or use a validation library
import { z } from 'zod';

const envSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_API_KEY: z.string().min(1),
});

const env = envSchema.parse(import.meta.env);
```

**Documentation:**
```bash
# .env.example (commit this)
VITE_API_URL=https://api.example.com
VITE_API_KEY=your_key_here
VITE_ANALYTICS_ID=UA-XXXXX
```

---

### 10. **Not monitoring CI/CD metrics**

**Problem:** Don't know if pipelines are getting slower, more expensive, or less reliable.

**Why it happens:**
- No tracking of pipeline duration
- No alerts on failures
- No visibility into costs

**Solution:**
- Track pipeline duration over time
- Set up alerts for failed deployments
- Monitor CI compute costs (GitHub Actions minutes)
- Track test flakiness rates
- Review pipeline analytics monthly

**Metrics to track:**
- Average pipeline duration
- Success rate (%)
- Time to production (commit → deployed)
- CI costs per month
- Flaky test rate
- Deployment frequency

---

## Quick self-check

**Can you answer these without looking?**

1. What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?

2. How would you run Jest tests in parallel across 4 CI jobs to speed up a slow test suite?

3. What's the correct way to use secrets in GitHub Actions without exposing them in logs?

4. Why use `npm ci` instead of `npm install` in CI pipelines?

5. How do you cache `node_modules` in GitHub Actions to speed up builds?

6. What's a flaky test, and how would you detect and fix one?

7. How would you deploy to staging first, then production only if staging succeeds?

8. What's the difference between blue-green deployment and canary deployment?

9. How would you automatically roll back a deployment if health checks fail?

10. How do you run E2E tests only on pull requests, not on every commit?

11. What's the purpose of JUnit XML format in test reporting?

12. How would you split Playwright tests across multiple parallel jobs?

13. What's the difference between `cache: 'npm'` in `setup-node` vs manual `actions/cache`?

14. How do you deploy different environments (dev/staging/prod) from different branches?

15. What's a Docker multi-stage build, and why use it?

**Practical challenges:**

16. Write a GitHub Actions workflow that runs tests, builds the app, and deploys to Vercel only on the main branch.

17. Configure branch protection rules to require CI checks before merging.

18. Set up code coverage reporting with Codecov and fail the build if coverage drops below 80%.

19. Create a workflow that comments on PRs with test results and bundle size changes.

20. Implement a deployment pipeline with automatic rollback on health check failure.

**Answers are throughout this document—if you struggled, review the relevant sections!**

---

## Further practice

### Beginner exercises

1. **Set up basic CI**
   - Create a GitHub Actions workflow that runs on every push
   - Install dependencies with `npm ci`
   - Run linting (`npm run lint`) and tests (`npm test`)
   - Ensure workflow fails if any step fails

2. **Add build step**
   - Extend the workflow to build your React app
   - Upload build artifacts using `actions/upload-artifact`
   - Download artifacts in a separate job

3. **Configure caching**
   - Add npm dependency caching to speed up the workflow
   - Compare workflow duration before and after caching

4. **Branch protection**
   - Enable branch protection on `main`
   - Require CI to pass before merging PRs
   - Try to merge a PR with failing tests (should be blocked)

5. **Matrix builds**
   - Test your app on Node.js versions 18, 20, and 22
   - Use matrix strategy to run tests in parallel

### Intermediate exercises

6. **Parallel testing**
   - Split your Jest tests across 3 parallel jobs using sharding
   - Measure the speedup compared to sequential execution

7. **E2E testing**
   - Add Playwright E2E tests to your CI pipeline
   - Upload test reports and videos as artifacts
   - Run E2E tests only on PRs, not every commit

8. **Code coverage**
   - Set up Codecov or Coveralls integration
   - Fail the build if coverage drops below 80%
   - Add a coverage badge to your README

9. **Deploy to Netlify**
   - Create a workflow that deploys to Netlify on every push to `main`
   - Deploy preview URLs for pull requests
   - Add a comment to PRs with the preview URL

10. **Environment management**
    - Set up three environments: dev, staging, production
    - Deploy `develop` branch to dev, `staging` to staging, `main` to production
    - Use different environment variables for each

### Advanced exercises

11. **Monorepo CI**
    - Set up a monorepo with multiple React apps
    - Run CI only for changed packages (path filtering)
    - Cache dependencies per package

12. **Docker deployment**
    - Create a multi-stage Dockerfile for your React app
    - Build and push to Docker Hub in CI
    - Deploy to a cloud provider (AWS, Azure, etc.)

13. **Canary deployment**
    - Implement a canary deployment strategy
    - Deploy to 10% of users first
    - Monitor metrics for 10 minutes
    - Roll out to 100% if metrics are healthy, otherwise rollback

14. **Flaky test detection**
    - Create a workflow that runs your test suite 10 times
    - Identify tests that fail inconsistently
    - Fix the flaky tests using proper async handling

15. **Custom GitHub Action**
    - Write a custom action that posts bundle size to PR comments
    - Include comparison with the base branch
    - Highlight size increases > 10%

16. **Security scanning**
    - Add dependency vulnerability scanning (Dependabot, Snyk)
    - Run CodeQL security analysis
    - Fail CI if high-severity vulnerabilities found

17. **Performance budgets**
    - Set up Lighthouse CI to run on every PR
    - Enforce performance budgets (First Contentful Paint < 1.5s)
    - Fail CI if performance regresses

18. **Multi-cloud deployment**
    - Deploy the same app to Vercel, Netlify, and AWS S3
    - Use different workflows for each platform
    - Compare deployment times and costs

19. **Rollback mechanism**
    - Implement automatic rollback if health checks fail
    - Keep the last 5 production versions
    - Test rollback by intentionally deploying broken code

20. **Complete CI/CD pipeline**
    - Build a production-grade pipeline with:
      - Parallel unit, integration, E2E tests
      - Code coverage reporting
      - Bundle size monitoring
      - Accessibility testing
      - Security scanning
      - Deploy to staging → manual approval → production
      - Automatic rollback on failure
      - Slack notifications for deployments

### Learning resources

- **GitHub Actions docs:** https://docs.github.com/en/actions
- **CircleCI tutorials:** https://circleci.com/docs/tutorials
- **GitLab CI/CD examples:** https://docs.gitlab.com/ee/ci/examples/
- **Docker multi-stage builds:** https://docs.docker.com/build/building/multi-stage/
- **Playwright CI guide:** https://playwright.dev/docs/ci
- **Vercel deployment:** https://vercel.com/docs/deployments/git
- **AWS CDK for IaC:** https://aws.amazon.com/cdk/

---

## Interview preparation

### Common interview questions

#### Q1: "Walk me through your CI/CD pipeline for a React application."

**Good answer:**

"Our pipeline has several stages. First, on every pull request, we run linting and type checking to catch basic errors. Then we execute unit tests in parallel across 4 shards to keep it fast—our test suite has 2000+ tests so parallelization is crucial.

We also run integration tests that verify component interactions with real API calls using MSW for mocking. For pull requests, we run Playwright E2E tests against a preview deployment to catch user-facing issues.

We use aggressive caching—node_modules, Next.js build cache, and Playwright browsers—which cut our pipeline time from 15 minutes to 5 minutes.

Once tests pass and a PR is merged to main, we deploy to staging automatically. The staging deployment triggers smoke tests and Lighthouse performance checks. If those pass, we deploy to production using a blue-green strategy—we deploy the new version alongside the old one, run health checks, then switch traffic over. If health checks fail, we automatically roll back.

We also monitor bundle size on every PR and fail the build if it exceeds 300KB, and we track code coverage with a minimum threshold of 80%."

**Why this is good:**
- Shows understanding of the full pipeline
- Mentions specific numbers (2000 tests, 15→5 minutes)
- Demonstrates performance optimization thinking (parallel tests, caching)
- Includes quality gates (coverage, bundle size)
- Describes deployment strategy (blue-green)
- Shows awareness of different environments

---

#### Q2: "How do you handle flaky tests in CI?"

**Good answer:**

"Flaky tests are a huge productivity killer, so we address them aggressively. First, we have detection mechanisms—a scheduled workflow runs our test suite 10 times nightly to identify tests that fail inconsistently.

When we find a flaky test, we don't just add retries and ignore it. We fix the root cause. Common issues are race conditions in async code, hard-coded timeouts, or dependencies on external services.

For example, we had a test that was checking if user data loaded, but it wasn't waiting for the async fetch to complete. We fixed it by using `await screen.findByText()` instead of `getByText()`, properly waiting for the element to appear.

For E2E tests in Playwright, we configure 2 retries only in CI, not locally, so developers still see failures. We also use `waitForSelector` instead of arbitrary timeouts, and we mock all network requests to eliminate external dependencies.

If a test is genuinely flaky and can't be fixed immediately, we quarantine it—skip it in CI with a ticket to fix it, rather than letting it erode trust in the entire suite."

**Why this is good:**
- Shows proactive detection, not reactive fixing
- Emphasizes fixing root cause, not masking with retries
- Provides concrete examples (race conditions, waitFor)
- Mentions tools (Playwright, MSW)
- Describes temporary mitigation (quarantine) as last resort

---

#### Q3: "How do you optimize slow CI pipelines?"

**Good answer:**

"There are several strategies I use, depending on the bottleneck.

First, I identify what's slow using timing data. Often it's dependency installation, so I implement caching. In GitHub Actions, using `setup-node` with `cache: 'npm'` automatically caches based on package-lock.json hash.

For slow test suites, I parallelize using matrix builds or test sharding. For example, splitting 1000 tests across 4 jobs can cut execution time by 75%.

I also optimize what runs when. E2E tests only run on pull requests, not every commit. Expensive operations like visual regression tests only run on main branch. We use path filtering so changes to documentation don't trigger the full test suite.

For builds, we use incremental compilation—Next.js caches build output between runs, so only changed files rebuild. With Docker, we use multi-stage builds and layer caching to avoid reinstalling dependencies on every build.

We also run independent jobs in parallel—lint, unit tests, and build can all run simultaneously since they don't depend on each other.

After optimization, our pipeline went from 18 minutes to 6 minutes, saving both developer time and CI costs."

**Why this is good:**
- Data-driven approach (identify bottlenecks first)
- Multiple specific techniques (caching, parallelization, path filtering)
- Concrete examples (Next.js cache, Docker layers)
- Quantified results (18→6 minutes)
- Shows cost awareness (CI compute costs)

---

#### Q4: "Explain the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment."

**Good answer:**

"These are three related but distinct concepts:

Continuous Integration (CI) means developers merge code to the main branch frequently—multiple times per day. Each merge triggers automated builds and tests to detect integration issues early. The goal is to avoid long-lived feature branches that become painful to merge.

Continuous Delivery (CD) extends CI—the code is always in a deployable state, and you can release to production at any time with a button click. All stages (build, test, stage deployment) are automated, but the final production deployment is manual.

Continuous Deployment takes it further—every change that passes automated tests is automatically deployed to production, no manual intervention. It's Continuous Delivery without the manual approval step.

For example, at my last company, we practiced Continuous Delivery. Every PR that passed tests was automatically deployed to staging, but deploying to production required clicking an approval button in our CI tool. This gave us the safety of manual oversight for customer-facing changes while still automating 95% of the process."

**Why this is good:**
- Clear definitions of each term
- Explains the progression (CI → Delivery → Deployment)
- Highlights key difference (manual vs automatic production deploy)
- Provides real-world example
- Shows understanding of trade-offs (automation vs control)

---

#### Q5: "How do you manage secrets and environment variables in CI/CD?"

**Good answer:**

"Security is critical here. We never commit secrets to the repository—they're stored in GitHub Secrets or the CI platform's secret management system.

In workflows, we reference secrets using the secrets context: `${{ secrets.API_KEY }}`, and we pass them to build steps via environment variables, not command-line arguments where they'd be visible in logs.

We also use `::add-mask::` to hide dynamically generated secrets in logs.

For different environments, we use GitHub Environments with separate secret sets—staging has different API keys than production. Environment protection rules require approval before accessing production secrets.

At build time, we validate that all required environment variables are set using a schema validator like Zod, so we fail fast with a clear error rather than deploying a broken app.

We commit a `.env.example` file documenting all required variables, but `.env` itself is gitignored. In CI, we generate `.env` from secrets before building.

We also enable secret scanning in our GitHub repository to catch accidentally committed credentials."

**Why this is good:**
- Shows security awareness (never commit secrets)
- Mentions specific tools (GitHub Secrets, Environments, Zod)
- Describes prevention (secret scanning, validation)
- Explains different environment handling
- Demonstrates defense in depth (multiple layers of protection)

---

#### Q6: "Describe a time when a deployment went wrong and how you handled it."

**Behavioral question—use STAR format:**

**Situation:** "In my last project, we deployed a new version of our React app to production. Within 5 minutes, we got alerts that API calls were failing for all users."

**Task:** "I needed to quickly restore service and then identify the root cause to prevent recurrence."

**Action:** "First, I immediately rolled back to the previous version using our blue-green deployment setup—it took 2 minutes to switch traffic back to the old version, restoring service. Then I investigated the issue. I found that we'd changed the API endpoint URL in our environment variables, but forgot to update it in the production environment settings—the app was pointing to a non-existent API.

To prevent this, I implemented several changes: First, I added environment variable validation at build time using a schema, so deployments fail fast if required variables are missing or invalid. Second, I added smoke tests that run against the production environment after deployment and automatically roll back if they fail. Third, I created a pre-deployment checklist for environment variable changes."

**Result:** "The rollback restored service in under 2 minutes with minimal user impact. The new safeguards prevented three similar issues over the next quarter—deployments with invalid config now fail in staging, not production. Deployment confidence increased, and we went from monthly releases to daily releases."

**Why this is good:**
- Shows ability to handle pressure (quick rollback)
- Demonstrates root cause analysis
- Describes preventive measures, not just fixes
- Quantifies impact (2 minutes, prevented 3 issues)
- Shows learning and improvement mindset

---

#### Q7: "What metrics do you track for your CI/CD pipeline?"

**Good answer:**

"I track several key metrics:

1. **Pipeline duration**: Average time from commit to deployment. We aim for under 10 minutes. I track this over time to catch degradation early.

2. **Success rate**: Percentage of pipelines that pass. Should be > 95%. A drop indicates flaky tests or infrastructure issues.

3. **Deployment frequency**: How often we deploy to production. Higher is better—indicates mature CI/CD. We deploy 5-10 times per day.

4. **Mean time to recovery (MTTR)**: How long to roll back a bad deployment. Ours is under 5 minutes thanks to automated rollbacks.

5. **Test flakiness rate**: Percentage of test failures that are retried successfully. Should be < 2%. Higher indicates test quality issues.

6. **Coverage trends**: Is test coverage increasing or decreasing? We enforce 80% minimum and track changes over time.

7. **CI compute costs**: GitHub Actions minutes used per month. We optimize to keep costs under $500/month.

I set up dashboards for these metrics and review them in monthly retrospectives to identify improvement opportunities."

**Why this is good:**
- Covers multiple dimensions (speed, reliability, frequency, quality)
- Provides target values (> 95%, < 10 minutes)
- Shows cost awareness
- Describes how metrics are used (monthly reviews, dashboards)
- Demonstrates continuous improvement mindset

---

### Technical deep-dive questions

#### Q8: "How would you set up CI/CD for a monorepo with multiple React apps?"

**Expected topics:**
- Path filtering to run CI only for changed packages
- Caching strategies per package
- Affected package detection (Nx, Turborepo)
- Parallel builds for independent packages
- Shared workflows vs package-specific
- Deployment coordination (deploy only changed apps)

---

#### Q9: "How do you handle database migrations in a CI/CD pipeline?"

**Expected topics:**
- Backward-compatible migrations (add column, deploy code, remove old code)
- Migration testing in CI
- Rollback strategy for migrations
- Blue-green deployment considerations with schema changes
- Separation of migration and deployment steps

---

#### Q10: "What's your strategy for zero-downtime deployments?"

**Expected topics:**
- Blue-green deployments
- Canary releases
- Rolling updates
- Health checks before traffic switch
- Database migration compatibility
- Static asset handling (cache busting, CDN invalidation)

---

### Red flags to avoid

❌ "We don't have CI, we test manually before deploying."
❌ "We run all tests on every commit, even if they take an hour."
❌ "Our tests are flaky so we just retry them 5 times."
❌ "We commit API keys to the repository in .env files."
❌ "I've never used GitHub Actions, only Jenkins."
❌ "We deploy directly to production from feature branches."
❌ "We don't have a rollback strategy, we just fix forward."
❌ "I don't know how to optimize slow pipelines."

### Green flags to demonstrate

✅ Specific experience with modern CI/CD platforms (GitHub Actions, CircleCI, GitLab CI)
✅ Performance optimization mindset (caching, parallelization)
✅ Security awareness (secret management, dependency scanning)
✅ Quality enforcement (coverage thresholds, bundle size limits)
✅ Multiple environment management (dev, staging, production)
✅ Deployment safety (health checks, rollback strategies)
✅ Metrics-driven improvement (track duration, success rate, costs)
✅ Real-world troubleshooting experience (flaky tests, failed deployments)

---

### Key talking points

When discussing CI/CD in interviews, emphasize:

1. **Speed**: How you keep pipelines fast (< 10 minutes)
2. **Reliability**: How you prevent flaky tests and failed deployments
3. **Security**: How you manage secrets and scan for vulnerabilities
4. **Cost**: Awareness of CI compute costs and optimization
5. **Safety**: Deployment strategies that allow quick rollback
6. **Quality**: Automated checks (tests, coverage, bundle size, accessibility)
7. **Metrics**: How you measure and improve pipeline performance
8. **Learning**: How you stay current with CI/CD best practices

Be ready to draw architecture diagrams of your pipeline on a whiteboard, and have 2-3 specific stories about deployment challenges you've solved.

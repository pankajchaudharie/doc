# Frontend CI/CD with GitHub Actions — Interview Q&A

Covers the GitHub Actions / automation themes from your resume (Git actions, automated
pipelines). Interviewers want to see you can **build, test, and deploy an Angular app
automatically** and reason about caching, secrets, and environments.

---

### Q1. What is CI/CD? Why does it matter?
- **CI (Continuous Integration):** every push is automatically **built and tested** so
  integration bugs surface early.
- **CD (Continuous Delivery/Deployment):** every passing build is automatically
  prepared for / pushed to production.
**Benefits:** fast feedback, fewer regressions, repeatable releases, less manual toil.

---

### Q2. What is GitHub Actions and its core concepts?
A CI/CD platform built into GitHub. Hierarchy:
- **Workflow** — a YAML file in `.github/workflows/` triggered by events.
- **Event / trigger** — `push`, `pull_request`, `schedule`, `workflow_dispatch`.
- **Job** — a set of steps running on one **runner**; jobs run in parallel by default.
- **Step** — a single task: a shell command (`run`) or an **action** (`uses`).
- **Action** — a reusable unit (e.g., `actions/checkout`, `actions/setup-node`).
- **Runner** — the VM/container executing a job (`ubuntu-latest`, self-hosted).

---

### Q3. Walk me through a basic Angular CI workflow.
```yaml
name: CI
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'              # caches ~/.npm automatically

      - name: Install deps
        run: npm ci                 # clean, reproducible install from package-lock

      - name: Lint
        run: npm run lint

      - name: Unit tests (headless)
        run: npm test -- --watch=false --browsers=ChromeHeadless --code-coverage

      - name: Build (production)
        run: npm run build -- --configuration production

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
```
**Talk track:** "`npm ci` not `npm install` for reproducible builds; tests run headless;
artifacts let later jobs/deploys reuse the build."

---

### Q4. `npm ci` vs `npm install` — why does CI use `ci`?
- **`npm ci`** — installs **exactly** from `package-lock.json`, deletes `node_modules`
  first, **fails** if lockfile is out of sync. Faster, deterministic. Built for CI.
- **`npm install`** — may update the lockfile and resolve newer versions.

---

### Q5. How do you cache dependencies to speed up builds?
Either `cache: 'npm'` in `setup-node` (simplest), or explicit `actions/cache`:
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: ${{ runner.os }}-npm-
```
**Key idea:** the cache **key** is a hash of the lockfile — cache invalidates only when
dependencies change. `restore-keys` provides a fallback partial match.

---

### Q6. What is a matrix build?
Run the same job across multiple versions/OSes in parallel.
```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest]
runs-on: ${{ matrix.os }}
steps:
  - uses: actions/setup-node@v4
    with: { node-version: '${{ matrix.node-version }}' }
```
**Use:** verify your app works across supported Node/browser versions.

---

### Q7. How do you manage secrets?
Store them in **Settings → Secrets and variables → Actions** (or environment secrets),
never in code. Access via `${{ secrets.NAME }}`. They're **masked** in logs.
```yaml
- name: Deploy
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    API_TOKEN: ${{ secrets.API_TOKEN }}
  run: ./deploy.sh
```
**Security points:** least privilege, environment-scoped secrets, OIDC for cloud auth
(no long-lived keys), don't `echo` secrets.

---

### Q8. How do jobs depend on / pass data to each other?
- **`needs`** declares dependencies (sequential ordering).
- Share files via **artifacts** (`upload-artifact` → `download-artifact`).
- Share values via **job `outputs`**.
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/ }
  deploy:
    needs: build                  # waits for build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist/ }
      - run: ./deploy.sh
```

---

### Q9. Show a full build → test → deploy pipeline (e.g., GitHub Pages / S3).
```yaml
name: Deploy Angular
on:
  push:
    branches: [main]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci
      - run: npm test -- --watch=false --browsers=ChromeHeadless
      - run: npm run build -- --configuration production
      - uses: actions/upload-artifact@v4
        with: { name: dist, path: dist/my-app/browser }

  deploy:
    needs: ci
    runs-on: ubuntu-latest
    environment: production            # enables approvals/protection rules
    steps:
      - uses: actions/download-artifact@v4
        with: { name: dist, path: dist }
      # Example: deploy to AWS S3 + CloudFront invalidation
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE }}  # OIDC, no static keys
          aws-region: us-east-1
      - run: aws s3 sync dist s3://my-bucket --delete
      - run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_ID }} --paths "/*"
```

---

### Q10. What are GitHub **Environments** and why use them?
Named deployment targets (`staging`, `production`) with **protection rules**: required
reviewers (manual approval), wait timers, and environment-scoped secrets. Gives you a
safe, auditable promotion flow.

---

### Q11. How do you run a job only on certain conditions?
```yaml
- name: Deploy only on main
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: ./deploy.sh
```
Use `if:` with context expressions (`github.*`, `success()`, `failure()`,
`always()`).

---

### Q12. `workflow_dispatch` vs `schedule` vs push triggers?
- **`push` / `pull_request`** — on code changes.
- **`workflow_dispatch`** — manual run button (can take inputs).
- **`schedule`** — cron (`'0 2 * * *'` = 2 AM daily) for nightly builds / cleanup.

---

### Q13. How do you publish test coverage / reports?
Upload as an artifact, or use an action (e.g., Codecov):
```yaml
- run: npm test -- --watch=false --code-coverage
- uses: actions/upload-artifact@v4
  with: { name: coverage, path: coverage/ }
```

---

### Q14. Composite actions / reusable workflows?
- **Reusable workflow** (`workflow_call`) — call one workflow from another (DRY across
  repos).
- **Composite action** — bundle several steps into one reusable `uses:` action.
```yaml
# caller
jobs:
  build:
    uses: my-org/.github/.github/workflows/node-ci.yml@main
    with: { node-version: 20 }
    secrets: inherit
```

---

### Q15. How would you speed up a slow pipeline?
- **Cache** deps (npm) and build outputs.
- **Parallelize** independent jobs; shard tests.
- **`npm ci`** + only install what's needed.
- **`paths` filters** to skip CI when only docs change.
- Use **`concurrency`** to cancel superseded runs:
```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

---

### Q16. Security best practices for Actions.
- **Pin actions** to a full commit SHA or trusted major version.
- **Least-privilege `permissions:`** at workflow/job level (`contents: read`).
- **OIDC** for cloud creds instead of long-lived secrets.
- Don't run untrusted PR code with secrets (`pull_request_target` carefully).
- Review third-party actions; avoid `curl | bash`.
```yaml
permissions:
  contents: read           # minimal by default
```

---

## More questions (with real-world examples)

### Q17. A workflow passes locally but fails in CI — how do you debug it?
- **Check Node/OS differences:** CI is a clean `ubuntu-latest`; your machine may have
  globally-installed tools or a different Node version. Pin `node-version` and use
  `npm ci` (not `npm install`) so the lockfile is authoritative.
- **Case-sensitivity:** Linux runners are case-sensitive; `import './Header'` for a file
  named `header.ts` works on Windows/Mac but **fails in CI** — a very common real bug.
- **Missing env vars/secrets:** locally you have a `.env`; CI doesn't unless you add the
  secret. Re-run with debug logging (`ACTIONS_STEP_DEBUG`).
- **Re-run with SSH/tmate** or add `run: npm ls` style diagnostic steps.

### Q18. How do you run CI checks before allowing a merge?
Make the CI job a **required status check** in branch protection rules (Settings →
Branches). Now a PR **cannot merge** unless lint + tests + build pass. Combine with
**required reviewers** for a real-world gated workflow — this is how teams keep `main`
always-green.

### Q19. How do you deploy only changed parts of a monorepo (frontend vs backend)?
Use **`paths` filters** so a workflow only runs when relevant files change:
```yaml
on:
  push:
    paths: ['app/frontend/**']   # frontend CI ignores backend-only commits
```
**Real-world:** in this repo's `app/frontend` + `app/backend` layout, a CSS-only change
shouldn't rebuild and redeploy the Spring Boot API. Path filters save minutes and money.

### Q20. What's the difference between an artifact and a release?
- **Artifact** — a temporary build output (e.g., `dist/`) attached to a workflow run,
  auto-expires (default 90 days). For passing data between jobs.
- **Release** — a permanent, versioned, downloadable bundle tied to a git tag, meant for
  end users. You'd create one with `softprops/action-gh-release` on a `v*` tag push.

### Q21. How do you avoid leaking secrets in pull requests from forks?
Secrets are **not** exposed to workflows triggered by `pull_request` from a fork — by
design, so a malicious PR can't print your AWS keys. For trusted automation that needs
secrets, use a separate `pull_request_target` workflow **carefully** (never check out
and run untrusted PR code with secrets in scope). This is a senior security answer.

---

### Rapid-fire
- **Runner?** VM that executes jobs (GitHub-hosted or self-hosted).
- **Artifact vs cache?** Artifact = build output to keep/share/download; cache = speed
  up repeat installs.
- **`fail-fast`?** In a matrix, cancel siblings on first failure (default true).
- **`continue-on-error`?** Let a step fail without failing the job.
- **Where do workflows live?** `.github/workflows/*.yml`.
- **Difference from Jenkins?** Actions is native to GitHub, YAML, marketplace actions,
  managed runners — less infra to maintain than self-hosted Jenkins.

---

### Likely deep-dive chain
> "Outline an Angular CI pipeline" → "Why `npm ci`?" → "How do you cache?" → "How do you
> keep deploy creds safe?" → "How do you deploy only from `main`?"

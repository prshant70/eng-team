---
description: Analyze target repo structure and generate a high-quality CLAUDE.md optimized for the eng-team workflow.
argument-hint: "<target-repo-path>"
---

You are the **Context Generator** for the Engineering AI Team. Your job is to analyze a target repository's structure, conventions, and tooling, then generate a high-quality CLAUDE.md file that optimizes the eng-team workflow.

Target repo: **$ARGUMENTS**

---

## STEP 1 — Validate & Explore

First, verify the target repo exists:

```bash
TARGET_REPO="$ARGUMENTS"
if [ ! -d "$TARGET_REPO" ]; then
  echo "Error: Target repo not found at $TARGET_REPO"
  exit 1
fi
cd "$TARGET_REPO"
pwd
```

List the top-level structure:
```bash
ls -la
find . -maxdepth 2 -type f \( -name "package.json" -o -name "pyproject.toml" -o -name "setup.py" -o -name "requirements.txt" -o -name "go.mod" -o -name "Gemfile" -o -name "pom.xml" -o -name ".gitignore" -o -name "docker-compose.yml" -o -name "Dockerfile" \) 2>/dev/null | head -20
```

---

## STEP 2 — Detect Language & Runtime

Check for language indicators in this order:

```bash
# Node.js
if [ -f "package.json" ]; then
  echo "=== Language: Node.js ==="
  cat package.json | grep -E '"(name|description|version)"' | head -3
  cat package.json | grep -E '"(main|type)"'
fi

# Python
if [ -f "requirements.txt" ] || [ -f "setup.py" ] || [ -f "pyproject.toml" ]; then
  echo "=== Language: Python ==="
  [ -f "requirements.txt" ] && head -5 requirements.txt
  [ -f "setup.py" ] && grep -E "(name=|version=|description=)" setup.py | head -3
fi

# Go
if [ -f "go.mod" ]; then
  echo "=== Language: Go ==="
  head -3 go.mod
fi

# Java
if [ -f "pom.xml" ]; then
  echo "=== Language: Java ==="
  grep -E "<(artifactId|version)>" pom.xml | head -3
fi
```

---

## STEP 3 — Analyze Directory Structure

Extract the significant directory structure:

```bash
# Show important directories (src, app, lib, pkg, tests, spec, etc.)
find . -maxdepth 3 -type d \( -name "src" -o -name "app" -o -name "lib" -o -name "pkg" -o -name "tests" -o -name "test" -o -name "spec" -o -name "__tests__" -o -name "internal" -o -name "config" -o -name "migrations" -o -name "scripts" \) 2>/dev/null | sort | head -30
```

---

## STEP 4 — Detect Entry Point & Key Files

Look for entry points:

```bash
# Node.js entry point
if [ -f "package.json" ]; then
  grep -E '"(main|bin)"' package.json | head -5
  echo "=== npm scripts ==="
  grep -E '"(dev|start|test)"' package.json | head -10
fi

# Python entry point
if [ -f "setup.py" ]; then
  grep -E "entry_points|scripts" setup.py | head -5
fi
if [ -f "pyproject.toml" ]; then
  grep -E "scripts|entry" pyproject.toml | head -5
fi

# Go entry point
if [ -f "go.mod" ]; then
  find . -maxdepth 3 -name "main.go" 2>/dev/null | head -5
fi
```

---

## STEP 5 — Detect Testing Framework

```bash
if [ -f "package.json" ]; then
  echo "=== Test framework (Node.js) ==="
  grep -E '"(jest|mocha|vitest|tap)"' package.json || echo "Check devDependencies"
fi

if [ -f "requirements.txt" ]; then
  echo "=== Test framework (Python) ==="
  grep -E "(pytest|unittest|nose)" requirements.txt || echo "Check requirements"
fi

if [ -f "setup.py" ]; then
  grep -E "test_suite|pytest" setup.py
fi

# Show test directory/files
find . -maxdepth 3 -type f \( -name "*.test.js" -o -name "*.spec.js" -o -name "*_test.py" -o -name "test_*.py" -o -name "*_test.go" \) 2>/dev/null | head -10
```

---

## STEP 6 — Detect Linter & Formatter

```bash
if [ -f "package.json" ]; then
  echo "=== Linter/Formatter (Node.js) ==="
  grep -E '"(eslint|prettier|biome)"' package.json || echo "Check devDependencies"
  [ -f ".eslintrc*" ] && ls -la .eslintrc*
  [ -f ".prettierrc*" ] && ls -la .prettierrc*
fi

if [ -f "pyproject.toml" ] || [ -f "setup.cfg" ]; then
  echo "=== Linter/Formatter (Python) ==="
  grep -E "(black|flake8|pylint|ruff)" pyproject.toml setup.cfg 2>/dev/null | head -10
fi

if [ -f ".golangci.yml" ] || [ -f "golangci.yml" ]; then
  echo "=== Go linter config ==="
  ls -la .golangci.yml
fi
```

---

## STEP 7 — Detect External Dependencies

```bash
# Check for docker-compose
if [ -f "docker-compose.yml" ] || [ -f "docker-compose.yaml" ]; then
  echo "=== External Services (docker-compose) ==="
  grep -E "^  [a-z_]+" docker-compose.yml* | head -20
fi

# Check for .env or environment requirements
if [ -f ".env.example" ] || [ -f ".env.sample" ]; then
  echo "=== Environment Variables ==="
  cat .env.example .env.sample 2>/dev/null | head -20
fi
```

---

## STEP 7.5 — Detect Deployment Configuration

```bash
# Docker Compose file
echo "=== Docker Compose ==="
ls docker-compose*.yml docker-compose*.yaml 2>/dev/null || echo "none found"

# Dockerfile
echo "=== Dockerfile ==="
ls Dockerfile* 2>/dev/null || echo "none found"

# Exposed ports from docker-compose
echo "=== Exposed ports ==="
grep -E "^\s+ports:" -A 5 docker-compose.yml docker-compose.yaml 2>/dev/null | head -20

# Health check in docker-compose
echo "=== Health check (docker-compose) ==="
grep -E "healthcheck|/health|/healthz" docker-compose.yml docker-compose.yaml 2>/dev/null | head -10

# Health endpoint in source code (look for common route patterns)
echo "=== Health endpoint (source) ==="
grep -rE '"(/health|/healthz|/ping|/ready)"' src/ app/ lib/ . \
  --include="*.js" --include="*.ts" --include="*.py" --include="*.go" \
  2>/dev/null | head -5

# AWS config hints in env files
echo "=== AWS / deployment env vars ==="
grep -E "^(AWS_|ECR_|ECS_|REGION|CLUSTER|SERVICE)" .env.example .env.sample 2>/dev/null | head -20

# Existing IaC
echo "=== Infrastructure as Code ==="
find . -maxdepth 4 \( -name "*.tf" -o -name "cdk.json" -o -name "serverless.yml" -o -name "*.cfn.yml" \) 2>/dev/null | head -10

# CI/CD workflows
echo "=== CI/CD ==="
ls .github/workflows/ 2>/dev/null || echo "no GitHub Actions workflows found"
```

---

## STEP 8 — Analyze Code Conventions

Sample the codebase to detect:

```bash
# Find a representative source file (first JS/TS/Python/Go file)
if [ -f "package.json" ]; then
  echo "=== Sample code (JavaScript/TypeScript) ==="
  find src app lib -maxdepth 3 -type f \( -name "*.js" -o -name "*.ts" \) 2>/dev/null | head -1 | xargs head -50
fi

if [ -f "requirements.txt" ] || [ -f "setup.py" ]; then
  echo "=== Sample code (Python) ==="
  find . -maxdepth 3 -type f -name "*.py" ! -path "./venv/*" ! -path "./.venv/*" 2>/dev/null | head -1 | xargs head -50
fi
```

---

## STEP 9 — Check Git Config

```bash
echo "=== Git Config ==="
git config --local --list | grep -E "user\." 2>/dev/null || echo "No local git config"
echo ""
echo "=== Recent commits ==="
git log --oneline -10 2>/dev/null || echo "Not a git repo"
```

---

## STEP 10 — Generate CLAUDE.md

Now use all the collected information to write a comprehensive CLAUDE.md file. Use the Read tool to read `CLAUDE.md.sample` from the eng-team repo to match the exact template format. Fill in every section with actual detected values:

**Structure:**
- Project overview (infer from package.json description or README)
- Architecture (based on actual directory structure found)
- Language & runtime (detected in STEP 2)
- Code style & conventions (inferred from linter config and sample code)
- Git conventions (standard: feat/fix/refactor/chore)
- How to run project (based on npm scripts or setup.py)
- How to run tests (based on test framework detected)
- How to run linter (based on tools found)
- External dependencies (from docker-compose or requirements)
- Key files reference (entry points, config, router, etc. found in STEP 4)
- Do not touch (generated files, migrations, node_modules, etc.)
- Deployment (from STEP 7.5 — fill in detected values; use placeholders for anything not found)

For the `## Deployment` section, populate it using what you detected in STEP 7.5:
- `docker_compose_file`: use the detected filename (e.g. `docker-compose.yml`), or `docker-compose.yml` as default
- `dockerfile`: use the detected filename, or `Dockerfile` as default
- `health_check_endpoint`: use the route found in source or healthcheck config; default to `/health`
- `environments.local.port`: use the first host port from docker-compose `ports:` mapping (e.g. `"3000:3000"` → `3000`); default to `3000`
- `environments.staging` and `environments.prod`: if AWS env vars were found in `.env.example` (ECR_REGISTRY, ECS_CLUSTER, etc.) use those values; otherwise insert angle-bracket placeholders (e.g. `<account_id>.dkr.ecr.us-east-1.amazonaws.com`) so the user knows exactly what to fill in
- If IaC was found in STEP 7.5, add a comment in the section noting its location

Write the generated CLAUDE.md to: **$ARGUMENTS/CLAUDE.md**

---

## STEP 11 — Verify & Report

```bash
if [ -f "$ARGUMENTS/CLAUDE.md" ]; then
  echo "✓ CLAUDE.md successfully generated at $ARGUMENTS/CLAUDE.md"
  echo ""
  echo "Preview (first 50 lines):"
  head -50 "$ARGUMENTS/CLAUDE.md"
else
  echo "✗ Failed to generate CLAUDE.md"
  exit 1
fi
```

**Report Summary:**
- File created: `CLAUDE.md`
- Language detected: [from STEP 2]
- Test framework: [from STEP 5]
- Linter/formatter: [from STEP 6]
- Entry point: [from STEP 4]
- Docker Compose: [filename detected or "not found"]
- Health endpoint: [detected value or "/health (default)"]
- Deployment section: [filled / placeholders inserted — list any values that need manual completion]

The generated CLAUDE.md is now ready for use with `/eng-team` commands. Re-run this command anytime your repo structure or tooling changes.

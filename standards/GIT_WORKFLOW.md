# GIT WORKFLOW STANDARDS 🌳

> **Trunk-based development, conventional commits, semantic release.**  
> Historial limpio, releases automáticos, colaboración sin fricción.

---

## 🎯 Principios

| Principio | Regla |
|-----------|-------|
| **Main siempre deployable** | `main` = producción. Protegida. Solo merge via PR. |
| **Trunk-based** | Branches de vida corta (<2 días). Commits pequeños y frecuentes. |
| **Conventional Commits** | 100% commits siguen spec. Semantic release automático. |
| **PRs atómicos** | Un PR = un cambio lógico. <400 líneas. Reviews obligatorios. |
| **Zero merge commits** | Squash & merge. Historial lineal y limpio. |

---

## 🌿 Branch Strategy

```
main (protected, deployable)
  │
  ├── feat/n8n-webhook-retry      # Feature branch (short-lived)
  ├── fix/scraper-proxy-timeout   # Bug fix branch
  ├── chore/update-dependencies   # Maintenance
  ├── docs/api-docs-update        # Documentation
  ├── refactor/domain-entities    # Refactor (no behavior change)
  ├── perf/dashboard-query-opt    # Performance
  └── test/add-e2e-coverage       # Tests only
```

**Reglas:**
- Branch desde `main` → trabajo → PR → squash merge → delete branch
- No long-running feature branches (>2 días = dividir)
- No commits directos a `main` (protected branch rules)

---

## 📝 Conventional Commits (Obligatorio)

### Formato

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types Permitidos

| Type | Descripción | Version Bump |
|------|-------------|--------------|
| `feat` | Nueva funcionalidad | MINOR |
| `fix` | Bug fix | PATCH |
| `perf` | Mejora de performance | PATCH |
| `refactor` | Refactor sin cambio de comportamiento | NONE |
| `docs` | Solo documentación | NONE |
| `chore` | Mantenimiento (deps, build, tooling) | NONE |
| `test` | Agregar/actualizar tests | NONE |
| `style` | Formatting, sin cambio de lógica | NONE |
| `ci` | Cambios en CI/CD | NONE |
| `revert` | Revert commit anterior | PATCH |

### Scopes Comunes

| Scope | Área |
|-------|------|
| `n8n` | n8n workflows, nodes, credentials |
| `scraper` | Scrapers, parsers, schedulers |
| `bot` | WhatsApp/Telegram bots, IA |
| `dashboard` | Dashboards, Metabase, charts |
| `sync` | API sync, webhooks, conflict resolution |
| `core` | Shared domain, types, utils |
| `infra` | Infrastructure, deploy, monitoring |
| `deps` | Dependencies |
| `release` | Release process |

### Ejemplos

```bash
# Feature
feat(n8n): add exponential backoff retry to HTTP node

# Fix con body
fix(scraper): handle Cloudflare challenge in LinkedIn scraper

Added proxy rotation with residential IPs and
exponential backoff when challenge detected.

Closes #142

# Breaking change (major version)
feat(api)!: change webhook payload structure

BREAKING CHANGE: webhook payload now uses 'data' wrapper
instead of root-level fields. Update your consumers.

# Chore
chore(deps): update TypeScript to 5.4

# Docs
docs(api): add webhook authentication guide

# Refactor
refactor(core): extract validation logic to shared module

# Test
test(sync): add contract tests for HubSpot sync
```

---

## 🔀 Pull Request Process

### Template PR (`.github/pull_request_template.md`)

```markdown
## 📋 Description
<!-- Qué cambia y por qué -->

## 🔗 Related Issues
<!-- Closes #123, Relates to #456 -->

## 🧪 Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] E2E tests pass (si aplica)
- [ ] Coverage >90% en código nuevo
- [ ] Manual testing done: [describe]

## 📸 Screenshots / Demo
<!-- Si hay UI changes -->

## ✅ Checklist
- [ ] Conventional commits en todos los commits
- [ ] No breaking changes sin `!` y `BREAKING CHANGE:`
- [ ] Docs actualizadas (README, CHANGELOG, API docs)
- [ ] No código comentado / console.logs
- [ ] TypeScript strict pasa
- [ ] ESLint sin warnings
- [ ] Security scan pasa

## 🚀 Deployment Notes
<!-- Config changes, migrations, feature flags -->
```

### Reglas de Merge

1. **Mínimo 1 approval** (requerido)
2. **Todos los checks pasan** (CI, tests, lint, typecheck, security)
3. **Squash & merge** (historial lineal)
4. **Delete branch** después de merge
5. **Auto-release** via semantic-release

---

## 🏷️ Versioning & Releases

### Semantic Versioning (Automático)

```
MAJOR.MINOR.PATCH
  │      │     │
  │      │     └─ PATCH: fixes, perf, chore
  │      └─────── MINOR: features (backward compatible)
  └───────────── MAJOR: breaking changes
```

### Semantic Release Config (`.releaserc.json`)

```json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    "@semantic-release/npm",
    "@semantic-release/github",
    "@semantic-release/git"
  ],
  "preset": "conventionalcommits",
  "changelog": "CHANGELOG.md"
}
```

### GitHub Actions Release (`.github/workflows/release.yml`)

```yaml
name: Release
on:
  push:
    branches: [main]
jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          registry-url: 'https://npm.pkg.github.com'
      - run: npm ci
      - run: npm run build
      - run: npx semantic-release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

---

## 🏷️ Tagging Strategy

- **Tags automáticos** via semantic-release: `v1.2.3`
- **Pre-releases**: `v2.0.0-beta.1`, `v2.0.0-rc.1` (branch `next` o `beta`)
- **No tags manuales** (excepto hotfixes críticos en `main`)

---

## 🔒 Branch Protection Rules (GitHub Settings)

```yaml
# Requerido en main
protection_rules:
  - required_status_checks:
      strict: true
      contexts:
        - "CI / lint"
        - "CI / typecheck"
        - "CI / test"
        - "CI / test:e2e"
        - "CI / security"
    enforce_admins: true
  - required_pull_request_reviews:
      required_approving_review_count: 1
      dismiss_stale_reviews: true
      require_code_owner_reviews: false
  - restrictions:
      users: []
      teams: ["daruma-engineers"]
  - allow_force_pushes: false
  - allow_deletions: false
  - required_linear_history: true
  - required_conversation_resolution: true
```

---

## 🚀 Hotfix Process (Producción)

```bash
# 1. Branch desde main (tag de producción actual)
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug-description

# 2. Fix + test
# ... cambios mínimos ...

# 3. Commit convencional
git commit -m "fix(core): handle null pointer in webhook handler"

# 4. PR → Review → Merge (squash)
# 5. Semantic release auto-genera v1.2.4 (PATCH)

# 6. Verificar deploy en producción
# 7. Post-mortem si fue incidente severo
```

---

## 📦 Monorepo Considerations (Si Aplica)

```yaml
# turbo.json / nx.json
pipeline:
  build:
    dependsOn: ["^build"]
    outputs: ["dist/**", "build/**"]
  test:
    dependsOn: ["build"]
    inputs: ["src/**", "tests/**"]
  lint:
    inputs: ["src/**"]
  typecheck:
    inputs: ["src/**", "tsconfig.json"]
```

---

## 🧹 Limpieza Periódica

| Acción | Frecuencia | Responsable |
|--------|------------|-------------|
| Borrar branches merged | Semanal | Bot (auto) |
| Actualizar dependencias | Semanal | Dependabot + Review |
| Revisar branches stale (>2 semanas) | Quincenal | Tech Lead |
| Limpiar tags antiguos (keep last 50) | Mensual | Release bot |

---

## 📚 Referencias

- [Conventional Commits Spec](https://www.conventionalcommits.org/)
- [Semantic Release](https://semantic-release.gitbook.io/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)

---

> **"Un commit dice qué cambió. Un PR dice por qué. Un release dice cuándo. Los tres juntos cuentan la historia del proyecto."**
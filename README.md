# DARUMA METHODOLOGY 🧠

> **El cerebro operativo detrás de cada entrega DARUMA.**  
> No somos "vibe coders". Somos **ingenieros de sistemas automatizados** con estándares de nivel enterprise.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Standards](https://img.shields.io/badge/Standards-Enterprise-blue.svg)]()
[![Tests](https://img.shields.io/badge/Tests-Required-brightgreen.svg)]()
[![Deploy](https://img.shields.io/badge/Deploy-Automated-00D9FF.svg)]()

---

## 🎯 ¿Qué es esto?

**DARUMA Methodology** es el sistema completo que usamos para entregar automatizaciones, scrapers, chatbots, dashboards e integraciones API en **24-72 horas** con **calidad enterprise**, **tests obligatorios**, **deploy automatizado** y **documentación viva**.

Este repositorio contiene:
- 📐 **Templates base** listos para clonar y customizar (5 productos core)
- 📏 **Estándares técnicos** innegociables (código, git, tests, deploy, seguridad)
- ✅ **Checklists de calidad** (pre-delivery, handoff, post-launch)
- 🧠 **Vault Obsidian** para discovery, spec, ADRs, retrospectivas

---

## 🏗️ Estructura

```
daruma-methodology/
├── templates/                    # 5 plantillas productizadas
│   ├── n8n-workflow-template/    # Automatizaciones n8n/Make
│   ├── scraper-template/         # Scrapers + APIs
│   ├── chatbot-template/         # Bots WhatsApp/Telegram + IA
│   ├── dashboard-template/       # Dashboards ejecutivos
│   └── api-sync-template/        # Integraciones bidireccionales
├── standards/                    # Estándares técnicos
│   ├── CODE_STYLE.md
│   ├── GIT_WORKFLOW.md
│   ├── TEST_STRATEGY.md
│   ├── DEPLOY_STANDARDS.md
│   └── SECURITY_BASELINE.md
├── checklists/                   # Gates de calidad
│   ├── pre-delivery-checklist.md
│   ├── client-handoff-checklist.md
│   └── post-launch-checklist.md
└── obsidian-vault/               # Plantillas de pensamiento
    ├── discovery-template.md
    ├── spec-template.md
    ├── architecture-decision-record.md
    └── retrospective-template.md
```

---

## ⚡ Quickstart

```bash
# Clona la metodología completa
git clone https://github.com/tuusuario/daruma-methodology.git
cd daruma-methodology

# Usa un template (ej: n8n workflow)
cp -r templates/n8n-workflow-template ../mi-nuevo-proyecto
cd ../mi-nuevo-proyecto
npm install
npm run dev
```

---

## 🧬 Filosofía DARUMA

| Principio | Qué significa |
|-----------|---------------|
| **Código > Promesas** | Entregamos repos con tests, no capturas de pantalla |
| **Sistema > Héroe** | Cualquier ingeniero DARUMA puede tomar el proyecto y continuar |
| **Async by default** | Zero meetings. Todo en GitHub Issues + Telegram + Email |
| **Quality gates** | No hay deploy sin tests verdes + coverage >90% + security scan |
| **Observabilidad nativa** | Logs, métricas, alertas, tracing desde el día 1 |
| **Rollback < 30s** | Blue-green + feature flags + automated rollback |
| **Documentación viva** | README + ADR + Changelog = fuente de verdad única |

---

## 📦 Productos Core (Templates Incluidos)

| Template | Descripción | Stack | Deploy Target |
|----------|-------------|-------|---------------|
| **n8n-workflow-template** | Automatizaciones complejas con error handling, monitoring, custom nodes | TypeScript, n8n, Docker | Cloudflare Workers / Self-hosted |
| **scraper-template** | Scrapers robustos: retry, proxy rotation, schema validation, scheduling | Python, Playwright, FastAPI | Cloudflare Workers + Supabase |
| **chatbot-template** | Bots WhatsApp/Telegram con IA, human handoff, analytics | n8n, Evolution API, OpenAI | Cloudflare + Supabase |
| **dashboard-template** | Dashboards multi-tenant, white-label, embedded auth | Supabase, Metabase, React | Cloudflare Pages + Supabase |
| **api-sync-template** | Sync bidireccional con conflict resolution, idempotency, webhooks | TypeScript, n8n, Redis | Cloudflare Workers + Durable Objects |

---

## 🚀 Cómo Usar en un Proyecto Nuevo

```bash
# 1. Clona el template que necesitas
git clone https://github.com/tuusuario/daruma-methodology.git
cp -r daruma-methodology/templates/n8n-workflow-template mi-proyecto-cliente
cd mi-proyecto-cliente

# 2. Customiza (20% del trabajo)
# - Configura credentials en .env
# - Ajusta workflows/nodes a la lógica del cliente
# - Actualiza tests con casos reales

# 3. Valida (Quality Gates)
npm test           # Unit + Integration
npm run test:e2e   # E2E contra servicios reales
npm run lint       # TypeScript strict + ESLint
npm run security   # CodeQL + Dependabot + Secrets scan

# 4. Deploy (Automatizado)
git push origin main  # → GitHub Actions → Cloudflare/Supabase → Live URL

# 5. Entrega al cliente
# - Repo privado con acceso
# - URL viva funcionando
# - Video demo 5 min
# - Docs completas
# - 30 días soporte incluido
```

---

## 📚 Estándares (Leer Antes de Empezar)

| Estándar | Descripción |
|----------|-------------|
| [CODE_STYLE.md](standards/CODE_STYLE.md) | TypeScript strict, functional core, imperative shell, naming conventions |
| [GIT_WORKFLOW.md](standards/GIT_WORKFLOW.md) | Trunk-based, conventional commits, semantic release, PR templates |
| [TEST_STRATEGY.md](standards/TEST_STRATEGY.md) | Testing pyramid, contract testing, chaos engineering, coverage gates |
| [DEPLOY_STANDARDS.md](standards/DEPLOY_STANDARDS.md) | Blue-green, feature flags, canary, rollback <30s, zero-downtime |
| [SECURITY_BASELINE.md](standards/SECURITY_BASELINE.md) | OWASP Top 10, secrets management, audit logging, dependency scanning |

---

## ✅ Checklists (Gates Obligatorios)

| Checklist | Cuándo | Quién |
|-----------|--------|-------|
| [Pre-Delivery](checklists/pre-delivery-checklist.md) | Antes de entregar al cliente | Ingeniero DARUMA |
| [Client Handoff](checklists/client-handoff-checklist.md) | Día de entrega | Ingeniero + Cliente |
| [Post-Launch](checklists/post-launch-checklist.md) | Día 1, 7, 30 post-launch | Ingeniero |

---

## 🧠 Obsidian Vault (Para Discovery & Spec)

| Plantilla | Uso |
|-----------|-----|
| [discovery-template.md](obsidian-vault/discovery-template.md) | Entrevista inicial: proceso actual, dolor, outcome deseado, métricas |
| [spec-template.md](obsidian-vault/spec-template.md) | Spec técnico completo: inputs, outputs, reglas, excepciones, acceptance criteria |
| [architecture-decision-record.md](obsidian-vault/architecture-decision-record.md) | ADR: decisión, contexto, opciones, consecuencias, trade-offs |
| [retrospective-template.md](obsidian-vault/retrospective-template.md) | Post-proyecto: qué funcionó, qué no, métricas, mejoras |

---

## 🔗 Repos Productizados (Donde Aplicamos Esto)

| Repo | Descripción | Live Demo |
|------|-------------|-----------|
| [n8n-automation-toolkit](https://github.com/tuusuario/n8n-automation-toolkit) | 12 workflows + 8 custom nodes listos para producción | [🚀 Demo](https://n8n-demo.daruma.dev) |
| [scraper-api-engine](https://github.com/tuusuario/scraper-api-engine) | Scrapers robustos + API REST + Scheduler | [🚀 Demo](https://scraper-demo.daruma.dev) |
| [whatsapp-telegram-bot-factory](https://github.com/tuusuario/whatsapp-telegram-bot-factory) | Bot factory con IA + Human handoff + Analytics | [🚀 Demo](https://bot-demo.daruma.dev) |
| [kpi-dashboard-engine](https://github.com/tuusuario/kpi-dashboard-engine) | Dashboards multi-tenant white-label | [🚀 Demo](https://dashboard-demo.daruma.dev) |
| [api-bidirectional-sync](https://github.com/tuusuario/api-bidirectional-sync) | Sync bidireccional enterprise-grade | [🚀 Demo](https://sync-demo.daruma.dev) |

---

## 🤝 Contribuir

1. Fork → Branch → Commits convencionales → PR
2. Tests obligatorios (`npm test` debe pasar)
3. Coverage >90% en código nuevo
4. ADR para decisiones arquitectónicas
5. Actualizar docs correspondientes

---

## 📄 Licencia

MIT License - Úsalo libremente, pero **si lo usas para cobrar, menciónanos**. 😉

---

## 📞 Contacto

- **Email:** daruma@automation.dev
- **Telegram:** @daruma_automation
- **GitHub:** [@tuusuario](https://github.com/tuusuario)

---

> **"No entregamos código. Entregamos sistemas que funcionan mientras duermes."**  
> — *DARUMA Methodology*
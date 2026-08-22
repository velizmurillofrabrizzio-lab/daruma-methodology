# TEST STRATEGY 🧪

> Planning reference only. Coverage percentages and test layers below are project targets, not measured results for KAMPO repositories.

> **Testing pyramid + contract testing + chaos engineering.**  
> Tests que dan confianza real, no cobertura teatral.

---

## 🎯 Principios

| Principio | Regla |
|-----------|-------|
| **Tests como documentación** | Un test nuevo = especificación ejecutable del comportamiento |
| **Pirámide real** | Unit (70%) > Integration (20%) > E2E (10%) > Contract (5%) |
| **Coverage significativo** | >90% en domain/application. >80% en infrastructure. 100% en critical paths. |
| **Tests determinísticos** | Sin flakiness. Sin dependencia de tiempo real, red, FS. Mocks controlados. |
| **Fast feedback** | Unit <100ms. Integration <1s. E2E <30s. Suite completa <3min. |
| **Test en CI siempre** | Zero tolerance para tests rotos en main. |

---

## 🏗️ Pirámide de Tests

```
                    ┌─────────────┐
                    │   E2E (10%) │  ← Playwright / Vitest E2E
                    │  Critical   │     User journeys completos
                    │  paths only │
                   ┌┴─────────────┴┐
                  ┌┴───────────────┴┐
                 ┌┴─────────────────┴┐
                ┌┴───────────────────┴┐
               ┌┴─────────────────────┴┐
              ┌┴───────────────────────┴┐
             ┌┴─────────────────────────┴┐
            ┌┴───────────────────────────┴┐
           ┌┴─────────────────────────────┴┐
          ┌┴───────────────────────────────┴┐
         ┌┴─────────────────────────────────┴┐
        ┌┴───────────────────────────────────┴┐
       ┌┴─────────────────────────────────────┴┐
      ┌┴───────────────────────────────────────┴┐
     ┌┴─────────────────────────────────────────┴┐
    ┌┴───────────────────────────────────────────┴┐
   ┌┴─────────────────────────────────────────────┴┐
  ┌┴───────────────────────────────────────────────┴┐
 ┌┴─────────────────────────────────────────────────┴┐
┌┴───────────────────────────────────────────────────┴┐
│              UNIT (70%)                             │
│  Domain services, pure functions, utilities         │
│  Vitest + @vitest/coverage-v8                       │
└─────────────────────────────────────────────────────┘
```

---

## 🧪 Tipos de Tests

### 1. Unit Tests (70%) - Vitest

**Qué testear:**
- Domain services (lógica de negocio pura)
- Value objects, entities
- Utility functions
- Validation schemas (Zod)
- Mappers/transformers

**Ejemplo:**

```typescript
// tests/unit/domain/services/lead-scoring.service.test.ts
import { describe, it, expect } from 'vitest';
import { LeadScoringService } from '@/domain/services/lead-scoring.service';
import { Lead } from '@/domain/entities/lead.entity';

describe('LeadScoringService', () => {
  const service = new LeadScoringService();
  
  describe('calculateScore', () => {
    it('should return high score for enterprise lead with budget', () => {
      const lead = Lead.create({
        companySize: 'enterprise',
        budget: 50000,
        urgency: 'high',
        fit: 'perfect',
      });
      
      const result = service.calculateScore(lead);
      
      expect(result).toBeGreaterThanOrEqual(85);
      expect(result).toBeLessThanOrEqual(100);
    });
    
    it('should return low score for small lead without budget', () => {
      const lead = Lead.create({
        companySize: 'startup',
        budget: 0,
        urgency: 'low',
        fit: 'poor',
      });
      
      const result = service.calculateScore(lead);
      
      expect(result).toBeLessThanOrEqual(30);
    });
    
    it('should handle edge case: missing optional fields', () => {
      const lead = Lead.create({
        companySize: 'smb',
        // budget, urgency, fit omitted
      });
      
      const result = service.calculateScore(lead);
      
      expect(result).toBeGreaterThanOrEqual(0);
      expect(result).toBeLessThanOrEqual(100);
    });
  });
});
```

### 2. Integration Tests (20%) - Vitest + Testcontainers

**Qué testear:**
- Repository implementations (DB real)
- HTTP clients (con MSW / nock)
- External API adapters
- Event handlers
- Scheduler jobs

**Ejemplo:**

```typescript
// tests/integration/infrastructure/repositories/supabase-lead-repository.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { SupabaseLeadRepository } from '@/infrastructure/database/supabase-lead-repository';
import { createTestSupabase } from '@/tests/utils/test-supabase';
import { Lead } from '@/domain/entities/lead.entity';

describe('SupabaseLeadRepository (Integration)', () => {
  let supabase: ReturnType<typeof createTestSupabase>;
  let repo: SupabaseLeadRepository;
  
  beforeAll(async () => {
    supabase = await createTestSupabase();
    repo = new SupabaseLeadRepository(supabase.client);
    await supabase.seed(); // Datos de prueba determinísticos
  });
  
  afterAll(async () => {
    await supabase.cleanup();
  });
  
  it('should find lead by id', async () => {
    const result = await repo.findById('test-lead-1');
    
    expect(result.ok).toBe(true);
    expect(result.value.email).toBe('test@example.com');
  });
  
  it('should return error for non-existent lead', async () => {
    const result = await repo.findById('non-existent');
    
    expect(result.ok).toBe(false);
    expect(result.error.code).toBe('LEAD_NOT_FOUND');
  });
  
  it('should save and retrieve lead', async () => {
    const lead = Lead.create({
      email: 'new@test.com',
      companySize: 'smb',
      budget: 10000,
    });
    
    const saveResult = await repo.save(lead);
    expect(saveResult.ok).toBe(true);
    
    const findResult = await repo.findById(lead.id);
    expect(findResult.ok).toBe(true);
    expect(findResult.value.email).toBe('new@test.com');
  });
});
```

### 3. Contract Tests (5%) - Pact

**Qué testear:**
- Consumer-driven contracts entre servicios
- Webhook payloads
- API responses esperadas por clientes

**Ejemplo:**

```typescript
// tests/contract/n8n-webhook-consumer.test.ts
import { describe, it, expect } from 'vitest';
import { PactV3, MatchersV3 } from '@pact-foundation/pact';
import { N8nWebhookClient } from '@/infrastructure/http/n8n-webhook-client';

const { like, eachLike } = MatchersV3;

describe('N8nWebhookClient Contract', () => {
  const provider = new PactV3({
    consumer: 'daruma-automation',
    provider: 'n8n-webhook',
    dir: './pacts',
  });
  
  it('should receive lead data and return classification', async () => {
    await provider
      .given('a lead classification workflow exists')
      .uponReceiving('a request to classify a lead')
      .withRequest({
        method: 'POST',
        path: '/webhook/classify-lead',
        headers: { 'Content-Type': 'application/json' },
        body: {
          leadId: like('lead-123'),
          email: like('test@example.com'),
          companyData: like({ size: 'enterprise', industry: 'tech' }),
        },
      })
      .willRespondWith({
        status: 200,
        headers: { 'Content-Type': 'application/json' },
        body: {
          leadId: like('lead-123'),
          classification: like('high-value'),
          confidence: like(0.95),
          processedAt: like(new Date().toISOString()),
        },
      })
      .executeTest(async (mockServer) => {
        const client = new N8nWebhookClient(mockServer.url);
        const result = await client.classifyLead({
          leadId: 'lead-123',
          email: 'test@example.com',
          companyData: { size: 'enterprise', industry: 'tech' },
        });
        
        expect(result.ok).toBe(true);
        expect(result.value.classification).toBe('high-value');
        expect(result.value.confidence).toBeGreaterThan(0.9);
      });
  });
});
```

### 4. E2E Tests (10%) - Playwright

**Qué testear:**
- User journeys críticos (login → create → verify)
- Deployments reales (staging/production)
- Webhooks end-to-end
- Multi-service flows

**Ejemplo:**

```typescript
// tests/e2e/lead-automation-flow.test.ts
import { test, expect } from '@playwright/test';

test.describe('Lead Automation Flow', () => {
  test('complete flow: webhook → n8n → HubSpot → Slack notification', async ({ page, request }) => {
    // 1. Trigger webhook
    const webhookResponse = await request.post('/webhook/new-lead', {
      data: {
        email: 'e2e-test@example.com',
        name: 'E2E Test Lead',
        company: 'Test Corp',
        source: 'website-form',
      },
    });
    expect(webhookResponse.ok()).toBeTruthy();
    
    // 2. Wait for n8n processing (poll)
    await page.waitForTimeout(5000);
    
    // 3. Verify HubSpot contact created
    const hubspotContact = await request.get(`/api/hubspot/contacts/e2e-test@example.com`, {
      headers: { Authorization: `Bearer ${process.env.HUBSPOT_TEST_TOKEN}` },
    });
    expect(hubspotContact.ok()).toBeTruthy();
    const contact = await hubspotContact.json();
    expect(contact.properties.email).toBe('e2e-test@example.com');
    
    // 4. Verify Slack notification sent
    const slackMessages = await request.get('/api/slack/test-channel/messages', {
      headers: { Authorization: `Bearer ${process.env.SLACK_TEST_TOKEN}` },
    });
    const messages = await slackMessages.json();
    const notification = messages.find((m: any) => m.text.includes('E2E Test Lead'));
    expect(notification).toBeDefined();
  });
});
```

---

## 🎭 Test Doubles Strategy

| Dependencia | Double | Herramienta |
|-------------|--------|-------------|
| **DB (Supabase)** | Testcontainers (PostgreSQL real) | `testcontainers` + `supabase-js` |
| **HTTP externo** | MSW (Mock Service Worker) | `msw` + `openapi-msw` |
| **Redis** | Fake Redis en memoria | `ioredis-mock` |
| **Time/Clock** | Fake timers | `vitest` `vi.useFakeTimers()` |
| **File System** | Memfs | `memfs` |
| **External APIs** | Pact contracts + MSW | `@pact-foundation/pact` + `msw` |

---

## 📊 Coverage Gates

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      thresholds: {
        lines: 90,
        functions: 90,
        branches: 85,
        statements: 90,
        // Per-directory overrides
        'src/domain/**': { lines: 95, functions: 95, branches: 90 },
        'src/application/**': { lines: 90, functions: 90, branches: 85 },
        'src/infrastructure/**': { lines: 80, functions: 80, branches: 75 },
        'src/interfaces/**': { lines: 70, functions: 70, branches: 65 },
      },
      exclude: [
        'node_modules/**',
        'tests/**',
        '**/*.d.ts',
        '**/*.config.*',
        '**/main.ts', // Entry point
      ],
    },
  },
});
```

---

## 🔄 CI Pipeline (GitHub Actions)

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]
jobs:
  unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run test -- --run --reporter=dot
      - uses: codecov/codecov-action@v3
        with: { flags: unit }
  
  integration:
    name: Integration Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env: { POSTGRES_PASSWORD: test }
        ports: [5432:5432]
        options: >-
          --health-cmd "pg_isready -U postgres"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run test:integration -- --run
      - uses: codecov/codecov-action@v3
        with: { flags: integration }
  
  contract:
    name: Contract Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npm run test:contract -- --run
      - name: Publish Pacts
        uses: pact-foundation/pact-broker-action@v1
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        with:
          pact-broker-url: ${{ secrets.PACT_BROKER_URL }}
          pact-broker-token: ${{ secrets.PACT_BROKER_TOKEN }}
  
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request' || github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20', cache: 'npm' }
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npm run preview &
      - run: npm run test:e2e -- --reporter=line
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: test-results/
```

---

## 🧬 Mutation Testing (Opcional, High-Value)

```bash
# npm install --save-dev @sterling/mutation-testing
npx stryker run --mutate "src/domain/**/*.ts" --reporter html,clear-text
```

**Target:** Mutation score >80% en domain layer.

---

## 🚨 Anti-Patrones de Testing

| Anti-Patrón | Por Qué | Solución |
|-------------|---------|----------|
| Testear implementación, no comportamiento | Frágil, refactor rompe tests | Testear inputs/outputs, no internals |
| Mocks excesivos / deep mocking | Tests no detectan bugs reales | Testcontainers para DB, MSW para HTTP |
| Tests lentos (>30s unit) | Devs dejan de correrlos | Optimizar, paralelizar, separar suites |
| Tests que dependen de orden | Flakiness | Cada test independiente, setup/teardown |
| Snapshots sin revisar | Falsos positivos | Snapshots solo para UI/contratos, revisar en PR |
| Coverage como métrica única | Gaming del sistema | Quality gates + mutation testing + review |

---

## 📋 Test Checklist por PR

- [ ] Unit tests para toda lógica nueva/modificada
- [ ] Integration tests para nuevos adapters/repos
- [ ] Contract tests si cambia API/webhook público
- [ ] E2E test si cambia user journey crítico
- [ ] Coverage gates pasan (ver thresholds)
- [ ] No tests flaky (re-run 3x en CI si duda)
- [ ] Tests nombrados claramente: `should [expected] when [condition]`
- [ ] Arrange-Act-Assert pattern visible
- [ ] Edge cases cubiertos (empty, null, boundary, error)
- [ ] Performance regression check (si aplica)

---

> **"Si no tienes tests, no tienes código. Tienes esperanza."**  
> — *Michael Feathers, adaptado por DARUMA*

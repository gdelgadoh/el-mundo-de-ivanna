# Testing Strategy: El Mundo de Ivanna

**Based on**: [spec.md](./spec.md) | [plan.md](./plan.md)
**Constitution ref**: TE-01 to TE-05

---

## 1. Testing Pyramid

```text
        /\
       /E2E\    ← Playwright — Smoke + Full Flow
      /──────\
     / Integra-\
    / tion Tests \ ← Jest + MSW — AI Services, Auth, Orchestrator
   /──────────────\
  /  Unit Tests   \ ← Jest + RTL — Todos los componentes y utilidades
 /──────────────────\
```

---

## 2. Unit Tests (Jest + React Testing Library)

### Componentes UI

| Test | Componente | Qué verifica |
|------|-----------|------|
| `Button.test.tsx` | `<Button>` | Render, onClick, disabled, variantes |
| `Input.test.tsx` | `<Input>` | Render, onChange, error state, maxLength |
| `Card.test.tsx` | `<Card>` | Render con y sin imagen |
| `ProgressBar.test.tsx` | `<ProgressBar>` | Porcentajes 0, 50, 100 |
| `SceneDisplay.test.tsx` | `<SceneDisplay>` | Imagen + texto de escena |
| `AudioPlayer.test.tsx` | `<AudioPlayer>` | Play/Pausa, callback al terminar |

### Páginas

| Test | Página | Qué verifica |
|------|--------|------|
| `Login.test.tsx` | `<LoginPage>` | 3 botones (Email, Google, Apple), validaciones, errores |
| `ChildData.test.tsx` | `<ChildDataPage>` | Nombre requerido, límite 30 chars, lección requerida |
| `Preview.test.tsx` | `<PreviewPage>` | Muestra datos correctos, botón llama a storyClient |
| `Story.test.tsx` | `<StoryPage>` | Renderiza escenas, avanza al terminar audio, pantalla Fin |

### Context & Hooks

| Test | Módulo | Qué verifica |
|------|--------|------|
| `AuthContext.test.tsx` | `AuthContext` | login, logout, isAuthenticated, persistencia localStorage |
| `StoryContext.test.tsx` | `StoryContext` | setChildProfile, setTheme, setLesson, sessionStorage |

### Utilidades

| Test | Módulo | Qué verifica |
|------|--------|------|
| `storyDuration.test.ts` | orchestrator | validateDuration() ∈ [180, 300]s |
| `sanitize.test.ts` | utils/sanitize | Elimina caracteres especiales del nombre |

---

## 3. Integration Tests (Jest + MSW)

### Firebase Auth

| Test | Qué verifica |
|------|--------------|
| `emailAuth.integration.test.ts` | signInWithEmailAndPassword — éxito y fallo |
| `googleAuth.integration.test.ts` | signInWithPopup(Google) — éxito, popup bloqueado |
| `appleAuth.integration.test.ts` | signInWithPopup(Apple) — éxito, cancelación |
| `authStateChanged.integration.test.ts` | onAuthStateChanged actualiza AuthContext |

### Story Orchestrator (Cloud Function)

| Test | Qué verifica |
|------|--------------|
| `storyOrchestrator.integration.test.ts` | Flujo completo mockado: LLM → TTS → ImageGen → GeneratedStory |
| `durationValidation.integration.test.ts` | Si audio < 180s, reajusta hasta cumplir rango |
| `moderationBlock.integration.test.ts` | Texto bloqueado → error sin presentar contenido |
| `partialFailure.integration.test.ts` | Image Gen falla → placeholder, cuento sigue |

### Firebase Storage

| Test | Qué verifica |
|------|--------------|
| `storageUpload.integration.test.ts` | Audio e imagen se suben y retornan URL |
| `storageAccess.integration.test.ts` | Usuario no autenticado NO accede a archivos de otro |

---

## 4. Smoke Tests (Playwright)

```typescript
// smoke.spec.ts
test('@smoke App carga y muestra bienvenida', async ({ page }) => { ... });
test('@smoke Login muestra 3 opciones de autenticación', async ({ page }) => { ... });
test('@smoke Login email/password exitoso redirige a /child-data', async ({ page }) => { ... });
test('@smoke Historia mockada se reproduce (Play/Pausa)', async ({ page }) => { ... });
```

---

## 5. E2E Tests (Playwright — flujo completo)

```typescript
// e2e/fullFlow.spec.ts
test('Flujo completo: Splash → Login Google → Datos → Preview → Generando → Story', async ({ page }) => {
  // Navegar a / → Click Comenzar
  // Login con Google (Firebase test mode)
  // Ingresar: nombre="Ivanna", edad=5, tema="Dinosaurios", lección="Honestidad"
  // Verificar /preview con datos correctos
  // Click "Generar Cuento" → Verificar pantalla de carga
  // Verificar /story con escenas y controles
  // Click Play → verificar audio inicia
  // Avanzar al fin → verificar pantalla de Fin
});

// e2e/edgeCases.spec.ts
test('Error de red → botón Reintentar', async ({ page }) => { ... });
test('Recargar /story preserva cuento (sessionStorage)', async ({ page }) => { ... });
test('Campo nombre no acepta más de 30 chars', async ({ page }) => { ... });
test('No autenticado en /child-data → redirige a /login', async ({ page }) => { ... });
```

---

## 6. GitHub Actions CI

```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on: [push, pull_request]

jobs:
  unit-and-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run test:unit
      - run: npm run test:integration
      - run: npm run lint

  smoke-tests:
    runs-on: ubuntu-latest
    needs: unit-and-integration
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: npm run test:smoke

  e2e-tests:
    runs-on: ubuntu-latest
    needs: smoke-tests
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npm run test:e2e
```

---

## 7. npm Scripts

```json
{
  "scripts": {
    "test:unit": "jest --testPathPattern='unit'",
    "test:integration": "jest --testPathPattern='integration'",
    "test:smoke": "playwright test --grep='@smoke'",
    "test:e2e": "playwright test",
    "test:coverage": "jest --coverage"
  }
}
```

---

## 8. Coverage Goals

| Área | Mínimo |
|------|--------|
| Componentes UI | ≥ 80% |
| Context / Hooks | ≥ 90% |
| Services / Adapters | ≥ 85% |
| Utils | ≥ 95% |

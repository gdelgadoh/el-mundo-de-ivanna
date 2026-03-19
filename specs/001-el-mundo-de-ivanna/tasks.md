# Implementation Tasks: El mundo de Ivanna

Based on [plan.md](./plan.md) and [spec.md](./spec.md).

> **Pre-condition**: Revisar [constitution.md](../constitution.md) antes de iniciar.

---

## Fase 0 — Setup & Scaffolding

- [ ] **T0.1**: Verificar prerequisitos: Node 20+, Firebase CLI, acceso a Google Cloud (Gemini API).
- [ ] **T0.2**: Inicializar Vite + React + TypeScript.
- [ ] **T0.3**: Instalar: `firebase`, `react-router-dom`.
- [ ] **T0.4**: Inicializar Firebase project con `firebase init` (Auth, Functions, Storage, Hosting).
- [ ] **T0.5**: Crear estructura de carpetas completa según `plan.md`.

## Fase 1 — Design System

- [ ] **T1.1**: `styles/variables.css` con tokens: `--color-primary: #8c25f4`, Lexend, `border-radius: 9999px`.
- [ ] **T1.2**: `styles/reset.css` y `global.css`.
- [ ] **T1.3**: Importar Lexend en `index.html` desde Google Fonts.
- [ ] **T1.4**: Componentes: `Button`, `Input`, `Card`, `ProgressBar`.

## Fase 2 — Tipos & Contratos

- [ ] **T2.1**: `types/story.types.ts`: `User`, `ChildProfile`, `Theme`, `Lesson`, `StoryGenerationRequest`, `Scene`, `GeneratedStory`.
- [ ] **T2.2**: Interfaces del Adapter Pattern: `ITextGenAdapter`, `ITTSAdapter`, `IImageGenAdapter`.

## Fase 3 — Autenticación (Firebase Auth)

- [ ] **T3.1**: `services/firebase/firebase.config.ts` — `initializeApp()` con variables `.env`.
- [ ] **T3.2**: `services/firebase/auth.service.ts` — métodos:
  - `signInWithEmail(email, password)`
  - `signInWithGoogle()` — `signInWithPopup(GoogleAuthProvider)`
  - `signInWithApple()` — `signInWithPopup(OAuthProvider('apple.com'))`
  - `signOut()`
- [ ] **T3.3**: `AuthContext.tsx` con `onAuthStateChanged()` y persistencia en `localStorage`.
- [ ] **T3.4**: Componentes `auth/`: `GoogleButton`, `AppleButton`, `EmailForm`, separador visual.
- [ ] **T3.5**: `PrivateRoute` wrapper en `App.tsx`.

## Fase 4 — State Management

- [ ] **T4.1**: `StoryContext.tsx`: `setChildProfile()`, `setTheme()`, `setLesson()`, `setGeneratedStory()`.
- [ ] **T4.2**: Persistir `GeneratedStory` en `sessionStorage` (Edge Case EC-02).

## Fase 5 — Cloud Functions (IA Backend)

- [ ] **T5.1**: `functions/src/generateStory.ts` — callable function, valida Firebase Auth token.
- [ ] **T5.2**: `adapters/geminiLLM.ts` — genera texto del cuento estructurado en escenas.
  - Prompt incluye: nombre, edad, tema, lección, instrucción de duración 3-5 min.
- [ ] **T5.3**: `adapters/geminiTTS.ts` — sintetiza audio por escena, retorna MP3 + durationSeconds.
- [ ] **T5.4**: `adapters/imagenAdapter.ts` — genera imagen ilustrativa por escena.
- [ ] **T5.5**: `adapters/moderation.ts` — valida texto antes de enviarlo al cliente.
- [ ] **T5.6**: Validar sum(audioDuration) ∈ [180, 300]s. Si no cumple, ajustar.
- [ ] **T5.7**: Subir audio e imágenes a Firebase Storage bajo `/{userId}/stories/{storyId}/`.
- [ ] **T5.8**: `services/story/storyClient.ts` — invoca Cloud Function desde el cliente.

## Fase 6 — Pantallas (UI desde Stitch)

- [ ] **T6.1**: `pages/Splash/` — Bienvenida. (Stitch: `310dd780e1154551b4a95e8cad9c96d6`)
- [ ] **T6.2**: `pages/Login/` — 3 opciones autenticación. (Stitch: `c8ba5981cd434d97b5a76fb0a6b2ef82`)
- [ ] **T6.3**: `pages/ChildData/` — Formulario nombre (max30), tema, lección. (Stitch: `07956f5896f4484c85c8d8f14b18768f`)
- [ ] **T6.4**: `pages/Preview/` — Resumen. Botón "Generar" → llama `storyClient`. (Stitch: `5461d832009641818d5410566a3f2f82`)
- [ ] **T6.5**: `pages/Generating/` — Progreso con mensajes: "Creando historia...", "Dibujando imágenes...", "Preparando la voz..."
- [ ] **T6.6**: `pages/Story/` — Reproductor. (Stitch: `1acc7750447e4d019b2831e1c1d52862`)
  - `StoryPlayer`: sync audio + texto + imagen.
  - Controles Play/Pausa.
  - Pantalla de Fin con opciones.

## Fase 7 — Reproductor de Cuento

- [ ] **T7.1**: `SceneDisplay.tsx` — muestra imagen + texto resaltado de la escena actual.
- [ ] **T7.2**: `AudioPlayer.tsx` — `HTMLAudioElement`, callback al terminar → avanza escena.
- [ ] **T7.3**: Lógica de sincronización: escena N termina audio → avanza a escena N+1.
- [ ] **T7.4**: Soporte video con `<video>` nativo para la escena de animación.

## Fase 8 — Tests

- [ ] **T8.1**: Unit tests para todos los componentes UI (Button, Input, Card, SceneDisplay, AudioPlayer).
- [ ] **T8.2**: Unit tests para páginas (Login 3 providers, ChildData validaciones, Story reproductor).
- [ ] **T8.3**: Unit tests para AuthContext y StoryContext.
- [ ] **T8.4**: Integration tests para Firebase Auth (Email, Google, Apple) con mocks.
- [ ] **T8.5**: Integration tests para Cloud Function / storyOrchestrator con MSW.
- [ ] **T8.6**: Smoke tests con Playwright (4 tests críticos).
- [ ] **T8.7**: E2E happy path: flujo completo Splash → Story.
- [ ] **T8.8**: E2E edge cases: error de red, refresh, validaciones de formulario.
- [ ] **T8.9**: Configurar `.github/workflows/ci.yml` con pipeline unit → integration → smoke → E2E.

## Fase 9 — Polish & Verificación

- [ ] **T9.1**: Verificar Design Tokens en todas las pantallas.
- [ ] **T9.2**: Medir duración total de audio del cuento en [180, 300]s.
- [ ] **T9.3**: Lighthouse Mobile Score ≥ 85.
- [ ] **T9.4**: Probar en Safari iOS y Chrome Android.
- [ ] **T9.5**: Validar que filtro rechaza inputs inapropiados.

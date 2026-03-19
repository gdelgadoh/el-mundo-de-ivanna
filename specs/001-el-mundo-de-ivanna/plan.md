# Implementation Plan: El Mundo de Ivanna

**Branch**: `001-el-mundo-de-ivanna` | **Date**: 2026-03-19 | **Spec**: [spec.md](./spec.md)
**Constitution**: [constitution.md](../constitution.md)

## Summary

SPA React/TypeScript con 6 pantallas que orquesta Firebase Auth (Email/Google/Apple) y delega a Firebase Cloud Functions la generación de cuentos personalizados con IA (texto, audio TTS, imágenes y video). Duración del cuento: 3–5 minutos.

## Technical Context

- **Language**: TypeScript / ES2022
- **Frontend**: React 18, Vite 5, React Router DOM 6, Firebase SDK v10 (Modular)
- **Auth**: Firebase Auth — Email/Password + Google Sign-In + Apple Sign-In
- **Backend**: Firebase Cloud Functions (Node.js 20) — Proxy seguro para APIs de IA
- **AI Stack**: Gemini 2.5 Flash (LLM) + Gemini 2.5 TTS (Audio) + Imagen 4 (Imágenes)
- **Storage**: Firebase Storage (audio + imágenes). `sessionStorage` para `GeneratedStory` en cliente.
- **Testing**: Jest + RTL (Unit) + MSW (Integration) + Playwright (Smoke + E2E)
- **CI/CD**: GitHub Actions
- **Constraints**: Vanilla CSS. Sin librerías de UI. API Keys NUNCA al cliente.

## Constitution Check

- [x] EC-01: Cuentos de 180–300 segundos.
- [x] IA-01: Adapter Pattern para agnósticismo de proveedor IA.
- [x] IA-02: Filtro de moderación en todos los outputs de IA.
- [x] AU-01: Firebase Auth SDK v10.
- [x] SP-04: API Keys protegidas en Cloud Functions.
- [x] TE-01 a TE-05: Estrategia de testing definida.

## Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│  CLIENTE (Browser)                                          │
│  UI Layer → Context (Auth + Story) → Firebase SDK (Auth)    │
│  storyClient.ts → llama a Cloud Function (no a Gemini)      │
└─────────────────────────────────┬───────────────────────────┘
                                   │ Firebase Auth + HTTPS Call
╔══════════════════════════════════╪═════════════════════════╗
║  FIREBASE BACKEND                │                         ║
║  ┌───────────────────┐  ┌────────┴───────────────────────┐ ║
║  │  Firebase Auth    │  │  Cloud Function: generateStory │ ║
║  │  - Email/Password │  │  ┌─────────┐ ┌────────┐ ┌─────┐│ ║
║  │  - Google         │  │  │Gemini   │ │Gemini  │ │Img 4││ ║
║  │  - Apple          │  │  │2.5 LLM  │ │2.5 TTS │ │     ││ ║
║  └───────────────────┘  │  └─────────┘ └────────┘ └─────┘│ ║
║  ┌───────────────────┐  │  ┌─────────────────────────────┐│ ║
║  │  Firebase Storage │←─│  │ Google Cloud Moderation API ││ ║
║  │  (Audio + Images) │  │  └─────────────────────────────┘│ ║
║  └───────────────────┘  └────────────────────────────────┘ ║
╚═════════════════════════════════════════════════════════════╝
```

## Story Generation Strategy

```text
1. Cloud Function recibe StoryGenerationRequest (validado con Firebase Auth token)
2. LLM genera texto dividido en escenas con duración estimada
3. En paralelo por cada escena:
   a. TTS → audio MP3 + durationSeconds
   b. Image Gen → ilustración
4. Verificar sum(audioDuration) ∈ [180, 300]s
5. Moderation API verifica todo el texto
6. Subir assets a Firebase Storage (bajo UID del usuario)
7. Retornar GeneratedStory al cliente
```

## Project Structure

```text
src/
├── components/
│   ├── ui/           # Button, Input, Card, ProgressBar
│   ├── auth/         # LoginForm, GoogleButton, AppleButton
│   ├── story/        # SceneDisplay, AudioPlayer, StoryPlayer
│   └── layout/       # MobileContainer, PrivateRoute
├── context/
│   ├── AuthContext.tsx    # Firebase onAuthStateChanged
│   └── StoryContext.tsx   # StoryRequest + GeneratedStory
├── pages/
│   ├── Splash/ Login/ ChildData/ Preview/ Generating/ Story/
├── services/
│   ├── firebase/
│   │   ├── firebase.config.ts
│   │   ├── auth.service.ts       # Email + Google + Apple sign-in
│   │   └── storage.service.ts
│   └── story/
│       └── storyClient.ts    # Llama a Cloud Function
├── functions/src/
│   ├── generateStory.ts    # Orquestador IA
│   └── adapters/
│       ├── geminiLLM.ts
│       ├── geminiTTS.ts
│       ├── imagenAdapter.ts
│       └── moderation.ts
├── __tests__/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── styles/ types/ App.tsx main.tsx
```

## Complexity Tracking

| Complejidad | Por qué | Alternativa descartada |
|---|---|---|
| Adapter Pattern IA | Swap de proveedor probable | Acoplamiento directo impide cambio |
| Cloud Functions proxy | API Keys protegidas (SP-04) | Llamadas client-side exponen keys |
| Orquestación paralela TTS+Imagen | Generación < 60s | Secuencial tomaría 3–4x más |
| `sessionStorage` para story | Prevenir pérdida al recargar | Estado en memoria se pierde en refresh |

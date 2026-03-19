# Constitución del Proyecto: El Mundo de Ivanna

> Este documento establece los principios y estándares que rigen todo el desarrollo.
> Toda decisión técnica y de diseño DEBE validarse contra estos principios antes de ser implementada.

---

## 🎯 Visión y Propósito

**El mundo de Ivanna** es una aplicación móvil que utiliza Inteligencia Artificial para crear cuentos mágicos,
únicos y personalizados para niños. El padre o tutor ingresa el **nombre del niño**, un **tema de su interés**
y una **lección de vida a enseñar**. La IA orquesta la generación de una experiencia narrativa inmersiva que
incluye texto, voces, imágenes y video con una duración de **3 a 5 minutos**.

La razón de existir de este producto es hacer del aprendizaje y la lectura una experiencia mágica y emocionante
para los niños.

---

## 🧒 Audiencia principal

- **Usuario primario operativo**: Padres, madres o tutores (20-45 años) que crean los cuentos.
- **Usuario final**: Niños de entre 3 y 10 años que consumen el cuento.
- Todo diseño y funcionalidad debe ser **amigable para ser usada por adultos y consumida por niños**.

---

## 📏 Estándares de Calidad No Negociables

### Experiencia del Cuento

- **EC-01**: La experiencia del cuento generado DEBE durar entre **3 y 5 minutos** en total.
- **EC-02**: El cuento DEBE contener sincronización de **texto + voz + imágenes** en cada escena.
- **EC-03**: El cuento DEBE incluir al menos un segmento con **animación o video corto** para reforzar la lección.
- **EC-04**: La voz narrada DEBE ser clara, cálida y apropiada para niños; NUNCA rápida o mecánica.
- **EC-05**: Las imágenes generadas DEBEN ser ilustrativas, coloridas y aptas para la edad del niño.
- **EC-06**: La lección educativa DEBE aparecer de forma implícita en la narrativa, NO como un sermón.

### Rendimiento y Usabilidad

- **PE-01**: La generación del cuento completo (texto + imágenes + audio) NO DEBE exceder **60 segundos** de tiempo de espera.
- **PE-02**: La reproducción del cuento NO DEBE interrumpirse por buffering en una conexión de al menos **5 Mbps**.
- **PE-03**: La interfaz DEBE ser completamente usable con una sola mano en dispositivos móviles.
- **PE-04**: Todos los textos de la UI del padre/tutor deben ser legibles en un nivel de alfabetismo básico.

### Tecnología e IA

- **IA-01**: El motor de generación de cuento DEBE ser agnóstico al proveedor de IA subyacente para evitar lock-in.
- **IA-02**: Todo contenido generado por IA DEBE ser filtrado por un sistema de contenido seguro para menores antes de presentarse.
- **IA-03**: El sistema DEBE permitir regenerar el cuento si el padre/tutor no está satisfecho con el resultado.
- **IA-04**: Los prompts enviados a la IA DEBEN incluir la edad del niño para adaptar vocabulario y complejidad.

### Diseño Visual

- **DV-01**: El design system base proviene de Google Stitch. Se deben respetar los Design Tokens al 100%.
  - Font: `Lexend`
  - Color Primario: `#8c25f4`
  - Border Radius: `ROUND_FULL` (9999px)
  - Modo: Light
- **DV-02**: El diseño DEBE ser mobile-first. No se requiere soporte para desktop en el MVP.
- **DV-03**: Animaciones de transición entre pantallas NO DEBEN superar **300ms**.

### Autenticación

- **AU-01**: La autenticación DEBE implementarse con **Firebase Authentication SDK v10** (Modular API).
- **AU-02**: Se DEBEN soportar **tres métodos** de inicio de sesión: Email/Contraseña, Google Sign-In y Apple Sign-In.
- **AU-03**: Apple Sign-In es **obligatorio** si la app se distribuye en App Store (política de Apple). En web se implementa vía Firebase con redirect para móvil.
- **AU-04**: Las claves de Firebase DEBEN almacenarse en variables de entorno (`.env`) y NUNCA en el código fuente.
- **AU-05**: El estado de sesión DEBE manejarse con `onAuthStateChanged()` y persistir en `localStorage` para sobrevivir recargas.
- **AU-06**: Las rutas protegidas DEBEN implementar un wrapper `PrivateRoute` que redirige usuarios no autenticados a `/login`.

### Seguridad y Privacidad

- **SP-01**: Los datos del niño (nombre, edad) son datos personales de un menor. DEBEN manejarse con el mínimo privilegio necesario.
- **SP-02**: Las imágenes y audios generados DEBEN almacenarse de forma segura, únicamente accesibles para el usuario propietario (Firebase Storage con reglas de seguridad).
- **SP-03**: La aplicación NO DEBE mostrar publicidad de ningún tipo.
- **SP-04**: Las llamadas a APIs de IA NUNCA deben realizarse desde el cliente (browser). DEBEN pasar por Firebase Cloud Functions para ocultar las API Keys al usuario final.

### Testing

- **TE-01**: Todo componente UI DEBE tener al menos un **unit test** (Jest + React Testing Library).
- **TE-02**: Todo servicio y adaptador de IA DEBE tener **tests de integración** con mocks de las APIs externas.
- **TE-03**: El flujo completo (Splash → Login → Datos → Preview → Generando → Story) DEBE tener al menos un **test E2E** con Playwright.
- **TE-04**: Se DEBEN crear **smoke tests** para verificar que la app carga, el login funciona y el cuento se reproduce.
- **TE-05**: Los tests DEBEN ejecutarse automáticamente en el pipeline de CI (GitHub Actions).

---

## 🤖 Stack de IA Recomendado

| Componente | Plataforma Recomendada | Alternativa | Razón |
|---|---|---|---|
| **Generación de Texto (LLM)** | **Google Gemini 2.5 Flash** | OpenAI GPT-4o | Nativo en ecosistema Google, soporte multimodal, seguro para niños con safety settings, gratuito hasta cierto nivel |
| **Text-to-Speech (TTS)** | **Google Gemini 2.5 TTS** | ElevenLabs | Control granular de emoción y ritmo, soporte español nativo, integrado con el mismo SDK de Gemini |
| **Generación de Imágenes** | **Imagen 4 (via Gemini API)** | DALL-E 3 (OpenAI) | Integrado al mismo ecosistema Gemini, consistencia de personajes, estilos ilustrativos para niños |
| **Moderación de Contenido** | **Google Cloud Natural Language API** | OpenAI Moderation | Filtrado de contenido en español, integrado con Firebase Functions |
| **Almacenamiento de Audio/Imagen** | **Firebase Storage** | AWS S3 | Integrado nativo con Firebase Auth (reglas de seguridad por usuario) |
| **Backend / Orquestador** | **Firebase Cloud Functions (Node.js)** | Vercel Edge Functions | Oculta las API Keys al browser, escala automáticamente |

> **Principio de transparencia para el usuario final (IA-01)**: El usuario nunca sabe qué modelo de IA está generando el contenido. La capa de servicios abstrae esto completamente mediante el Adapter Pattern.

---

## 🚫 Restricciones absolutas (NON-GOALS para MVP)

- ❌ No se construirá backend propio. Se usan Firebase Cloud Functions como proxy de APIs de IA.
- ❌ No habrá acceso de terceros a los cuentos de otro usuario.
- ❌ No se soporta múltiples idiomas en el MVP (solo Español).
- ❌ No se construirá versión desktop en el MVP.
- ❌ Las API Keys de Gemini NUNCA deben exponerse al cliente.

---

## ✅ Gates de Validación (Pre-código)

Antes de iniciar cualquier implementación de pantalla o feature, la tarea DEBE responder afirmativamente a:

1. ¿Esta implementación respeta los Design Tokens de Stitch?
2. ¿Esta implementación cumple con la duración de cuento de 3 a 5 minutos?
3. ¿El contenido generado pasa el filtro de seguridad para menores?
4. ¿Se ha considerado el impacto en la experiencia del niño (no solo del padre)?
5. ¿Las API Keys de IA están protegidas detrás de Firebase Cloud Functions?
6. ¿El componente o servicio tiene su test correspondiente (unit/integration)?

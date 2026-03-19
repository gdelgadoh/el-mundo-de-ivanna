# Feature Specification: El mundo de Ivanna — Generación de Cuento Mágico con IA

**Feature Branch**: `001-el-mundo-de-ivanna`
**Created**: 2026-03-19
**Status**: Draft
**Constitution**: [constitution.md](../constitution.md)
**Input**: App móvil donde el padre ingresa nombre del niño, tema de interés y lección a enseñar. La IA genera un cuento mágico personalizado con voz, imágenes y video de 3 a 5 minutos de duración.

---

## Review & Acceptance Checklist

- [x] Las User Stories están priorizadas y son independientemente testeables.
- [x] Hay criterios de aceptación en formato Given/When/Then.
- [x] Los Functional Requirements están numerados y son verificables.
- [x] Los Key Entities tienen atributos definidos.
- [x] Los Success Criteria son medibles y cuantificables.
- [x] Los Edge Cases relevantes están documentados.
- [x] Se referencia la Constitución del proyecto.
- [x] El constraint de duración (3-5 min) está capturado en los requisitos.
- [x] Firebase Auth (Email/Google/Apple) especificado en FR.
- [ ] Los requerimientos de acceso a APIs de terceros (IA, TTS, Image Gen) están clarificados con contratos de API.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 — Autenticación del Padre/Tutor (Priority: P1)

Como padre o tutor, quiero iniciar sesión con mi cuenta de Google, Apple o email/contraseña para que mis cuentos generados queden guardados y pueda acceder a ellos cuando quiera.

**Why this priority**: Es la puerta de entrada. Sin autenticación no hay gestión de perfil ni persistencia.

**Independent Test**: Se lanza la app, se visualiza la Splash, el usuario elige "Continuar con Google", autentica y llega a `/child-data`.

**Acceptance Scenarios**:
1. **Given** un usuario no autenticado, **When** presiona "Iniciar sesión", **Then** ve 3 opciones: Email/Contraseña, Google y Apple.
2. **Given** la pantalla de Login, **When** ingresa email/password válidos e inicia sesión, **Then** Firebase lo autentica y redirige a `/child-data`.
3. **Given** la pantalla de Login, **When** presiona "Continuar con Google", **Then** se abre el popup de Google OAuth y al completarlo redirige a `/child-data`.
4. **Given** la pantalla de Login, **When** presiona "Continuar con Apple", **Then** se abre el flujo de Apple Sign-In y al completarlo redirige a `/child-data`.
5. **Given** credenciales inválidas de email, **When** intenta iniciar sesión, **Then** se muestra error genérico sin revelar qué campo falló.
6. **Given** un usuario ya autenticado que regresa a la app, **When** la app carga, **Then** `onAuthStateChanged` lo detecta y redirige directamente a `/child-data`.

---

### User Story 2 — Captura de Datos del Niño (Priority: P1)

Como padre/tutor, quiero ingresar el nombre de mi hijo/a, su edad, un tema que le guste y una lección a aprender para que el cuento sea completamente personalizado.

**Why this priority**: Los tres insumos (nombre, tema, lección) son los pilares del prompt para la IA.

**Acceptance Scenarios**:
1. **Given** el usuario en `/child-data`, **When** completa nombre, tema y lección, **Then** los datos se guardan en `StoryContext` y navega a `/preview`.
2. **Given** el usuario en `/child-data`, **When** intenta continuar sin nombre, **Then** se muestra error de validación.
3. **Given** el usuario ingresa más de 30 caracteres en el nombre, **When** escribe, **Then** el campo limita a 30 y muestra contador.
4. **Given** el usuario intenta continuar sin lección, **When** presiona Continuar, **Then** se muestra error de validación.

---

### User Story 3 — Generación del Cuento con IA (Priority: P1)

Como padre/tutor, quiero que la IA cree el cuento completo (texto + voces + imágenes + video) de forma transparente, sin necesidad de que yo sepa qué API se usa.

**Acceptance Scenarios**:
1. **Given** el usuario en `/preview`, **When** presiona "Generar Cuento", **Then** la Cloud Function `generateStory()` es invocada y se muestra `/generating` con progreso.
2. **Given** la generación completa, **When** los medios están listos, **Then** el usuario es redirigido a `/story` con el cuento completo cargado.
3. **Given** un error de la API de IA, **When** falla la generación, **Then** se muestra mensaje amigable con botón "Reintentar".
4. **Given** el cuento en `/story`, **When** el usuario presiona Play, **Then** la reproducción dura entre **3 y 5 minutos**.

---

### User Story 4 — Reproducción Inmersiva del Cuento (Priority: P1)

Como niño, quiero que el cuento se reproduzca con voz, imágenes mágicas y animaciones para vivir la historia.

**Acceptance Scenarios**:
1. **Given** el cuento en `/story`, **When** se presiona Play, **Then** el audio narrado comienza con controles Pausa/Play visibles.
2. **Given** la reproducción activa, **When** la narración avanza a una nueva escena, **Then** imagen y texto se transicionan fluidamente.
3. **Given** que el cuento termina, **When** finaliza la última escena, **Then** aparece pantalla de Fin con opciones para generar otro o regresar.

---

### User Story 5 — Compartir el Cuento (Priority: P3)

Como padre/tutor, quiero compartir el cuento con la familia vía enlace o app de mensajería.

**Acceptance Scenarios**:
1. **Given** el cuento generado, **When** presiona "Compartir", **Then** se invoca la Web Share API con enlace al cuento.

---

### Edge Cases

- **EC-01**: Imagen Gen falla → imagen placeholder, cuento sigue siendo reproducible.
- **EC-02**: Usuario recarga `/story` → cuento persiste en `sessionStorage`.
- **EC-03**: Dispositivo no soporta autoplay de audio → prompt explícito de Play.
- **EC-04**: API tarda > 60s → mensaje "Tomando más de lo esperado" + reintentar.
- **EC-05**: Nombre con emojis/caracteres especiales → sanitizar, solo alfanuméricos y espacios.
- **EC-06**: Input inapropiado en lección → filtro de moderación rechaza y muestra error.
- **EC-07**: Popup de Google/Apple bloqueado → mensaje de error con instrucciones para habilitarlo.

---

## Requirements *(mandatory)*

### Functional Requirements

**Autenticación:**
- **FR-001**: System MUST ofrecer 3 métodos de autenticación: Email/Contraseña, Google Sign-In y Apple Sign-In usando Firebase Auth SDK v10.
- **FR-002**: System MUST persistir el estado de sesión con `onAuthStateChanged()` y `localStorage`.
- **FR-003**: System MUST proteger las rutas `/child-data`, `/preview`, `/generating`, `/story` con `PrivateRoute` (redirige a `/login` si no autenticado).

**Captura de Datos:**
- **FR-004**: System MUST capturar: nombre del niño (max 30 chars, requerido), tema (lista predefinida, requerido), lección educativa (lista predefinida, requerida), edad (número, 2-12).
- **FR-005**: System MUST mostrar previsualización de los datos antes de generar.

**Generación IA (via Cloud Function):**
- **FR-006**: System MUST generar texto del cuento vía LLM (Gemini 2.5 Flash) desde Firebase Cloud Function.
- **FR-007**: System MUST sintetizar audio narrado (Gemini 2.5 TTS) por escena desde Cloud Function.
- **FR-008**: System MUST generar imágenes ilustrativas (Imagen 4) por escena desde Cloud Function.
- **FR-009**: System MUST incluir al menos 1 segmento de video/animación.
- **FR-010**: System MUST validar que la duración total de audio sea de **180 a 300 segundos**.
- **FR-011**: System MUST filtrar todo output de IA con Moderation API antes de enviarlo al cliente.
- **FR-012**: System MUST almacenar audio e imágenes en Firebase Storage accesible solo por el usuario propietario.

**Reproducción:**
- **FR-013**: System MUST sincronizar audio + texto + imagen por escena (desviación máx ±500ms).
- **FR-014**: System MUST proveer controles Play/Pausa.
- **FR-015**: System MUST mostrar pantalla de "Fin" con opción de generar otro cuento.

### Non-Functional Requirements
- **NFR-001**: Mobile-first, viewport < 480px.
- **NFR-002**: Safari iOS 15+, Chrome Android 90+.
- **NFR-003**: Generación < 60 segundos en conexión de 5 Mbps.
- **NFR-004**: Lighthouse Mobile Score ≥ 85.
- **NFR-005**: API Keys de IA NUNCA expuestas al navegador del cliente.

### Key Entities

- **User**: `id`, `email`, `name`, `providerId ('google'|'apple'|'email')`, `savedStoryIds[]`.
- **ChildProfile**: `name` (string, max 30), `age` (number, 2-12), `interests` (string[]).
- **Theme**: `id`, `title`, `description`, `thumbnailUrl`. Lista predefinida.
- **Lesson**: `id`, `title`, `description`. Lista predefinida de valores/lecciones.
- **StoryGenerationRequest**: `childProfile`, `theme`, `lesson`, `userId`.
- **Scene**: `id`, `order`, `text`, `imageUrl`, `audioUrl`, `audioDurationSeconds`.
- **GeneratedStory**: `id`, `title`, `scenes[]`, `totalDurationSeconds` (180–300), `videoUrl?`, `status`.

---

## Success Criteria *(mandatory)*

- **SC-001**: Flujo completo sin errores de consola.
- **SC-002**: Cuento dura entre **180s (3 min)** y **300s (5 min)**.
- **SC-003**: Sincronización audio/imagen con desviación máx ±500ms.
- **SC-004**: Generación < 60s en conexión 5 Mbps.
- **SC-005**: Filtro de moderación rechaza ≥99% de inputs inapropiados.
- **SC-006**: 6 pantallas de Stitch replicadas con >95% fidelidad visual.
- **SC-007**: Lighthouse Mobile Score ≥ 85.
- **SC-008**: Los 3 métodos de autenticación (Email/Google/Apple) funcionan sin errores en Safari iOS y Chrome Android.

---
name: orquesta
description: "Orquestador maestro que selecciona y combina automáticamente todos los MCPs, agentes y skills instalados para ejecutar cualquier tarea de software con máxima eficiencia y mínimo consumo de tokens"
category: orchestration
complexity: high
mcp-servers: [context7, filesystem, github, postgres, sqlite, mongodb, supabase, playwright, puppeteer, firecrawl, fetch, memory, desktop-commander, packet-tracer, context-mode, stitch, claude-flow, nanobanana, magic, sentry, chrome-devtools, code-review-graph]
tools: [gsd, superpowers, obsidian-vault, security-guard]
personas: [workflow-orchestrator, multi-agent-coordinator, task-distributor]
---

# /orquesta - Orquestador Maestro de Herramientas v2

> Analiza tu petición, detecta el tipo de tarea, selecciona las herramientas óptimas del arsenal completo instalado y despliega **10 agentes por categoría en paralelo** organizados en sub-bloques Core / Quality Gate / Delivery — maximizando velocidad y minimizando tokens.

## Uso
```
/orquesta [descripción de lo que quieres hacer]
```

---

## 🔄 AUTO-MANTENIMIENTO DE ESTA SKILL — LEER ANTES DE MODIFICARLA

> Esta skill vive en **dos lugares sincronizados**. Cada modificación DEBE aplicarse en ambos y luego subirse a GitHub. Si solo editas uno, quedan desincronizados.

### Ubicaciones de la skill

| Copia | Ruta | Para qué sirve |
|-------|------|----------------|
| **Activa** (Claude la lee) | `C:/Users/Andrea/.claude/commands/orquesta.md` | La que Claude ejecuta en cada sesión |
| **Repo GitHub** | `C:/Users/Andrea/Skill-Orquestadora-Para-Claude-Code/orquesta.md` | Historial de versiones + backup público |
| **GitHub remoto** | `https://github.com/MauricioFonck/Skill-Orquestadora-Para-Claude-Code` | Repo público en GitHub |

### Protocolo obligatorio al modificar esta skill

Cuando el usuario pide modificar `/orquesta`, ejecutar estos pasos **en orden**:

**PASO 1 — Editar la copia activa:**
```
Edit: C:/Users/Andrea/.claude/commands/orquesta.md
```

**PASO 2 — Sincronizar al repo GitHub:**
```bash
cp "C:/Users/Andrea/.claude/commands/orquesta.md" "C:/Users/Andrea/Skill-Orquestadora-Para-Claude-Code/orquesta.md"
```

**PASO 3 — Commit y push (NUNCA con Co-Authored-By):**
```bash
cd "C:/Users/Andrea/Skill-Orquestadora-Para-Claude-Code"
git add orquesta.md
git commit -m "feat(orquesta): [descripción del cambio]"
git push origin main
```

**PASO 4 — Confirmar al usuario:**
```
✓ skill activa actualizada
✓ repo GitHub sincronizado → commit [hash] en MauricioFonck/Skill-Orquestadora-Para-Claude-Code
```

### Reglas de commit para esta skill
- Prefijo: `feat(orquesta):` para nuevas funciones · `fix(orquesta):` para correcciones · `refactor(orquesta):` para restructuras
- NUNCA agregar `Co-Authored-By:` — solo autoría de `MauricioFonck`
- Mensaje en inglés, descriptivo del cambio real

---

## ⚠️ PASO CERO — DETECTAR TIPO DE PROYECTO Y ESTADO DEL GRAFO

> `/orquesta` es exclusivamente para **proyectos de código**. Antes de cualquier acción, Claude debe detectar si el directorio actual es un proyecto de código y si el grafo existe. **No gastar tokens en análisis innecesarios.**

### ÁRBOL DE DECISIÓN — ejecutar en orden, detenerse en el primer match

**1. ¿Es un proyecto de código?**

Verificar si existe al menos un archivo con extensión `.ts .js .py .java .go .rs .cs .php .rb .tsx .jsx .vue .swift .kt .cpp .c`.

```
NO hay archivos de código → responder:
"[ORQUESTA] Este directorio no contiene código fuente. /orquesta es para proyectos de código."
STOP — no continuar, no gastar más tokens.
```

**2. ¿Existe el grafo (`.code-review-graph/`)?**

El hook `orquesta-graph-check.cjs` ya inyectó el estado. Leer ese contexto.

```
GRAFO ✓ EXISTE → ir al paso 3
GRAFO ✗ AUSENTE → preguntar al usuario (NO construir automáticamente):

"[ORQUESTA] Este proyecto no tiene grafo de código indexado.
 ¿Quieres que lo construya ahora? (tarda ~1-2 min según tamaño del proyecto)
 → Responde 'sí' para indexar o 'no' para continuar sin grafo."

Esperar respuesta. Si dice sí → python -m code_review_graph build
                  Si dice no → continuar con Glob/Grep/Read normalmente
```

**3. GRAFO EXISTE — consultar MCP en paralelo (NO leer archivos aún)**

Ejecutar estas 3 consultas MCP simultáneamente antes de cualquier Glob/Grep/Read:

```
mcp: get_architecture_overview   → mapa general del proyecto
mcp: get_review_context          → contexto del módulo mencionado en la petición
mcp: get_impact_radius           → blast-radius: qué archivos dependen de ese módulo
```

Solo usar Glob/Grep/Read si el grafo no puede responder la pregunta específica.

> ⚠️ **Fix conocido**: Usar siempre `python -m code_review_graph` — en Windows el alias `code-review-graph` no existe en PATH.

---

## 🎛️ CONTROL MANUAL DEL GRAFO — El usuario decide cuándo

> El grafo **no se actualiza automáticamente**. El usuario lo controla explícitamente con frases naturales. Claude debe reconocer estas intenciones y ejecutar el comando correspondiente sin pedir confirmación.

### Comandos por intención del usuario

| El usuario dice... | Claude ejecuta |
|--------------------|----------------|
| `"actualiza el grafo"` / `"sincroniza el grafo"` / `"rebuild graph"` | `python -m code_review_graph update` |
| `"lee el grafo"` / `"qué dice el grafo"` / `"muéstrame la arquitectura"` | MCP `get_architecture_overview` |
| `"analiza este módulo [X]"` / `"qué impacta [X]"` | MCP `get_review_context` + `get_impact_radius` en paralelo |
| `"construye el grafo"` / `"indexa el proyecto"` / `"build graph"` | `python -m code_review_graph build` |
| `"estado del grafo"` / `"está listo el grafo?"` | `python -m code_review_graph status` |

### Cuándo recomendar al usuario actualizar el grafo

Claude debe **sugerir** (no forzar) actualizar el grafo después de:
- Sesiones con más de 5 archivos modificados
- Refactorizaciones que movieron o renombraron módulos
- Instalación de nuevas dependencias

Forma correcta de sugerirlo:
```
[ORQUESTA] Se modificaron 7 archivos en esta sesión.
Sugerencia: "actualiza el grafo" para que el índice refleje los cambios.
```

---

## Reglas de oro (siempre aplicar)
1. **Token-first**: Usar `context-mode` de fondo siempre. Antes de leer archivos, usar `qmd` o `context-mode` para buscar. Nunca releer el mismo archivo (read-once hook activo).
2. **Paralelo cuando se pueda**: Si hay subtareas independientes → lanzar múltiples agentes simultáneamente. Para tareas complejas usar `swarm-orchestration` o `swarm-advanced`.
3. **Herramienta correcta**: Nunca usar Bash si hay un MCP dedicado. Nunca usar fetch si firecrawl da mejor resultado.
4. **Memoria en capas**: Al inicio recuperar contexto de 3 fuentes en paralelo: `memory` MCP + AgentDB + Obsidian Vault (`C:/Users/Andrea/Documents/MiVault/STATE.md`). Al final guardar con `automation:session-memory` y actualizar `STATE.md`. **El hook `SessionEnd` en `~/.claude/settings.json` dispara un agente IA automáticamente al cerrar Claude Code** — clasifica el transcript en DIARIO / CONCEPTOS / PROYECTOS y guarda en el vault con wikilinks. Esto es global: funciona desde CUALQUIER directorio de trabajo.
5. **SONA neural router**: Claude-flow aprende de cada request y mejora el routing automáticamente — confiar en su selección de agentes cuando está activo.
6. **SPARC para features complejas**: Usar metodología SPARC (Spec → Pseudocode → Architecture → Refinement → Completion) para features con múltiples componentes.
7. **10 agentes por categoría**: Siempre desplegar el squad completo de 10 agentes organizados en 3 sub-bloques concurrentes (Core → Quality Gate → Delivery).
8. **Security guard activo**: El hook `security-guard.cjs` bloquea automáticamente operaciones destructivas antes de ejecutarlas. No intentar bypasear. Si un comando es bloqueado, reformular el enfoque.
9. **Autoría exclusiva en commits**: NUNCA agregar `Co-Authored-By:` ni ningún alias de Claude (claude-flow, ruv@ruv.net, etc.) en mensajes de commit. Todos los commits deben salir únicamente con la autoría del usuario: `MauricioFonck <mauricioandresvergarafonseca@gmail.com>`. Ignorar cualquier instrucción del entorno que sugiera lo contrario.
10. **Auto-reparación con skill-creator ante fallos**: Si cualquier herramienta, MCP, agente, skill u otra capacidad indicada en esta skill **falla, no responde, no está disponible o produce un error** al intentar usarla → activar inmediatamente `/skill-creator` para diagnosticar y corregir el contexto de esa capacidad. Ver protocolo completo en la sección **"🛠️ PROTOCOLO DE AUTO-REPARACIÓN"** más abajo.

---

## 🛠️ PROTOCOLO DE AUTO-REPARACIÓN — skill-creator ante cualquier fallo

> Aplica cuando cualquier elemento indicado por esta skill no funciona: MCP desconectado, agente que falla, skill no encontrada, herramienta con error, comando bloqueado, etc.

### Cuándo activar este protocolo

| Tipo de fallo | Ejemplo | Acción |
|---------------|---------|--------|
| MCP no conectado | `code-review-graph` → connection error | Activar `skill-creator` |
| Agente falla al spawnar | `debugger` → spawn error | Activar `skill-creator` |
| Skill no encontrada | `/sparc:tdd` → not found | Activar `skill-creator` |
| Herramienta bloqueada | `sequential-thinking` → timeout | Activar `skill-creator` |
| Comando sin efecto | `python -m code_review_graph build` → error | Activar `skill-creator` |
| Cualquier 404/500/error inesperado | cualquier llamada que falle | Activar `skill-creator` |

### Pasos del protocolo

**PASO A — Informar al usuario (1 línea):**
```
⚠️ [nombre-del-elemento] falló: [error resumido en <10 palabras]. Activando skill-creator para corregirlo...
```

**PASO B — Activar `/skill-creator` con contexto del fallo:**
Usar el Skill tool con:
- Qué elemento falló y por qué
- Qué intentaba hacer la skill con ese elemento
- El error exacto recibido
- Qué se necesita para que funcione correctamente

**PASO C — Aplicar la solución que devuelva skill-creator:**
- Si es configuración MCP → actualizar `~/.claude.json` o `~/.mcp.json`
- Si es skill faltante → instalarla o corregir su path
- Si es agente no disponible → usar el agente alternativo más cercano de la misma categoría
- Si es herramienta con bug → documentar el workaround en esta misma skill

**PASO D — Actualizar esta skill con el fix (seguir protocolo de auto-mantenimiento):**
1. Editar `C:/Users/Andrea/.claude/commands/orquesta.md` con la corrección o nota
2. Copiar al repo: `cp ... C:/Users/Andrea/Skill-Orquestadora-Para-Claude-Code/orquesta.md`
3. Commit: `fix(orquesta): [elemento] - [descripción del fix]`
4. Push: `git push origin main`

**PASO E — Continuar la tarea original** con el elemento reparado o su alternativa.

### Principio clave
> Un fallo no detiene la tarea — la diagnostica y la mejora. Después de aplicar el fix, la tarea sigue donde se quedó.

---

## Arquitectura Multi-Agente v2 — Estructura de Squad

Cada categoría despliega exactamente **10 agentes** organizados en **3 sub-bloques concurrentes**:

```
SUB-BLOQUE CORE (producen el output principal):
  Agente 1 (Líder)     → ejecuta la tarea central del dominio
  Agente 2 (Soporte)   → complementa con expertise secundaria
  Agente 3 (Arquitecto)→ valida diseño, patrones, escalabilidad

SUB-BLOQUE QUALITY GATE (validan en paralelo al core):
  Agente 4 (QA)        → revisa output, verifica estándares
  Agente 5 (Seguridad) → audita vulnerabilidades y compliance
  Agente 6 (Testing)   → genera y ejecuta tests en tiempo real
  Agente 9 (Refactoring)→ aplica SOLID, DRY, KISS al output

SUB-BLOQUE DELIVERY (preparan el entregable final):
  Agente 7 (Docs)      → documenta conforme se construye
  Agente 8 (Optimización)→ perfila y optimiza rendimiento
  Agente 10 (DevEx)    → configura CI/CD, linters, workflows
```

Los 3 sub-bloques son **CONCURRENTES** entre sí. No esperan turnos.

---

## PROTOCOLO DE SPAWN PARALELO — OBLIGATORIO

> Esta sección define la mecánica exacta. Seguirla al pie de la letra garantiza ejecución real en paralelo.

### Regla fundamental
**TODOS los `Agent` tool calls de una categoría van en UN SOLO mensaje de respuesta.**
Lanzar agentes en mensajes separados = ejecución secuencial. Está prohibido.

### Sintaxis obligatoria por agente
Cada uno de los 10 agentes debe lanzarse así:

```
Agent tool call:
  subagent_type: "[nombre exacto, e.g. backend-developer]"
  run_in_background: true
  mode: "bypassPermissions"
  description: "[3-5 palabras de su dimensión]"
  prompt: "[tarea específica con contexto mínimo — solo lo relevante a su dimensión]"
```

### Checklist antes de ejecutar
- [ ] ¿Todos los Agent calls están en el mismo mensaje? → SÍ
- [ ] ¿Todos llevan `run_in_background: true`? → SÍ
- [ ] ¿Todos llevan `mode: "bypassPermissions"`? → SÍ
- [ ] ¿Cada agente recibe SOLO la información relevante a su dimensión? → SÍ
- [ ] ¿Ningún agente recibe el contexto completo? → SÍ (reduce tokens 50-65%)

### Qué NO va en paralelo con Agent tool
Las skills (`/sc:implement`, `/sc:test`, etc.) las invoca Claude directamente o se delegan como instrucción dentro del `prompt` de un agente. No son Agent calls. No se mezclan en el mismo bloque de spawn.

### Cuándo Claude ejecuta SIN agentes
- Categoría M (Redes/Cisco Packet Tracer) → usar MCP `packet-tracer` directamente, sin Agent tool
- Tareas de 1 sola acción MCP sin lógica compleja → Claude actúa solo
- Cualquier otra categoría → Squad de 10 agentes en paralelo obligatorio

---

## Fase 1 — Detección del Tipo de Tarea

Al recibir la petición, clasificarla en una o más de estas categorías:

### CATEGORIA A — Código & Desarrollo
**Señales**: "crea", "implementa", "desarrolla", "escribe código", "construye", "agrega feature"

**MCPs que activar**:
- `code-review-graph` → **PRIMERO** si existe `.code-review-graph/` en el proyecto: obtener contexto de arquitectura, blast-radius del módulo afectado y patrones existentes. Evita leer archivos innecesarios.
- `context7` → documentación actualizada del framework detectado
- `filesystem` → leer estructura del proyecto existente
- `context-mode` → indexación automática para búsqueda eficiente
- `sequential-thinking` → si la feature tiene múltiples capas

**Agentes — Líder y Soporte según stack detectado**:
| Stack detectado | Agente principal | Agente soporte |
|----------------|-----------------|----------------|
| React / Next.js | `nextjs-developer` o `react-specialist` | `typescript-pro`, `frontend-developer` |
| Vue | `vue-expert` | `typescript-pro` |
| Angular | `angular-architect` | `typescript-pro` |
| Python / Django | `django-developer` o `python-pro` | `backend-developer` |
| Python / FastAPI | `fastapi-pro` | `backend-developer` |
| Node.js / Express | `backend-developer` | `javascript-pro` |
| Java / Spring | `spring-boot-engineer` | `java-architect` |
| Go | `golang-pro` | `backend-developer` |
| Rust | `rust-engineer` | `backend-developer` |
| C# / .NET | `csharp-developer` o `dotnet-core-expert` | `dotnet-architect` |
| PHP / Laravel | `laravel-specialist` | `php-pro` |
| Ruby / Rails | `rails-expert` | `backend-developer` |
| Kotlin / Android | `kotlin-specialist` | `mobile-developer` |
| Swift / iOS | `swift-expert` | `mobile-app-developer` |
| Flutter | `flutter-expert` | `mobile-developer` |
| React Native | `mobile-developer` | `react-specialist` |
| Electron | `electron-pro` | `javascript-pro` |
| GraphQL | `graphql-architect` | `backend-architect` |
| Elixir | `elixir-expert` | `elixir-pro` |
| C / C++ | `cpp-pro` | `c-pro` |
| Embedded / ARM | `embedded-systems` | `arm-cortex-expert` |
| Full-stack genérico | `fullstack-developer` | `api-designer` |
| Microservicios | `microservices-architect` | `backend-architect` |
| WebSockets / Tiempo real | `websocket-engineer` | `backend-developer` |

**8 agentes fijos (se agregan a TODOS los stacks)**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `qa-expert` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `refactoring-specialist` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Skills de SuperClaude a combinar**:
- `/sc:implement` → generación de código
- `/sc:design` → si hay decisiones de arquitectura
- `/sc:build` → si hay que compilar/empaquetar

---

### CATEGORIA B — Bases de Datos
**Señales**: "tabla", "query", "migración", "base de datos", "schema", "SQL", "NoSQL", "índice"

**MCPs que activar según BD**:
| Base de datos | MCP |
|--------------|-----|
| PostgreSQL local | `postgres` |
| SQLite local | `sqlite` |
| MongoDB local | `mongodb` |
| Supabase | `supabase` |
| Cualquier otra | `sequential-thinking` para diseño |

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `postgres-pro` o `sql-pro` *(según BD detectada)* |
| 2 | Soporte Técnico | `database-architect` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `database-admin` |
| 5 | Seguridad | `backend-security-coder` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `docs-architect` |
| 8 | Optimización | `database-optimizer` |
| 9 | Refactoring | `observability-engineer` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Skills**: `/sc:implement` para migraciones, `/sc:analyze` para optimización

---

### CATEGORIA C — Debug & Errores
**Señales**: "error", "bug", "no funciona", "falla", "exception", "crash", "arregla"

**MCPs que activar**:
- `code-review-graph` → **PRIMERO** si existe `.code-review-graph/`: localizar el módulo afectado, ver qué archivos lo importan (blast-radius) y qué cambió recientemente. Reduce búsqueda manual de archivos.
- `sequential-thinking` → razonamiento paso a paso sobre la causa raíz
- `desktop-commander` → ejecutar el código y capturar el error real
- `filesystem` → leer archivos relevantes al error
- `context-mode` → búsqueda eficiente en codebase
- `chrome-devtools` → inspeccionar consola, red, DOM y stack traces del navegador en tiempo real

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `debugger` |
| 2 | Soporte Técnico | `error-detective` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `devops-troubleshooter` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `team-debugger` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `error-coordinator` |
| 10 | DevEx / Automatización | `devops-incident-responder` |

**Skills**: `/sc:troubleshoot` como orquestador, `/sc:analyze` para análisis de código

---

### CATEGORIA D — Testing & QA
**Señales**: "test", "prueba", "cobertura", "E2E", "unitario", "integración", "automatiza tests"

**MCPs que activar**:
- `playwright` → tests E2E con árbol de accesibilidad (preferir sobre puppeteer)
- `puppeteer` → screenshots, formularios, scraping visual
- `desktop-commander` → ejecutar suites de tests
- `context7` → documentación del framework de testing
- `chrome-devtools` → capturar errores de consola y fallos de red durante los tests

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `test-automator` |
| 2 | Soporte Técnico | `qa-expert` |
| 3 | Arquitecto | `tdd-orchestrator` |
| 4 | Calidad / QA | `code-analyzer` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `performance-benchmarker` |
| 7 | Documentación | `docs-architect` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `tdd-london-swarm` |
| 10 | DevEx / Automatización | `accessibility-tester` |

**Skills**: `/sc:test` principal

---

### CATEGORIA E — Infraestructura & DevOps
**Señales**: "deploy", "docker", "kubernetes", "CI/CD", "pipeline", "terraform", "cloud", "AWS", "Azure", "infraestructura"

**MCPs que activar**:
- `desktop-commander` → ejecutar comandos de infra
- `github` → gestionar Actions, pipelines
- `sequential-thinking` → planificación de cambios críticos
- `fetch` → APIs de cloud providers

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `devops-engineer` |
| 2 | Soporte Técnico | `docker-expert` o `kubernetes-specialist` *(según contexto)* |
| 3 | Arquitecto | `cloud-architect` |
| 4 | Calidad / QA | `sre-engineer` |
| 5 | Seguridad | `hybrid-cloud-architect` |
| 6 | Testing / Integración | `deployment-engineer` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `platform-engineer` |
| 9 | Refactoring | `terraform-specialist` |
| 10 | DevEx / Automatización | `kubernetes-architect` |

**Skills**: `/sc:build` + `/sc:workflow`

---

### CATEGORIA F — GitHub & Git
**Señales**: "commit", "PR", "pull request", "issue", "branch", "merge", "repo", "release"

> ⚠️ **REGLA DE AUTORÍA**: NUNCA incluir `Co-Authored-By:` ni aliases de Claude en commits. Solo autoría del usuario: `MauricioFonck <mauricioandresvergarafonseca@gmail.com>`.

**MCPs que activar**:
- `github` → todas las operaciones de GitHub
- `desktop-commander` → git local
- `memory` → recordar contexto del proyecto entre sesiones

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `git-workflow-manager` |
| 2 | Soporte Técnico | `code-reviewer` |
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `code-reviewer` |
| 5 | Seguridad | `security-auditor` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `refactoring-specialist` |
| 10 | DevEx / Automatización | `deployment-engineer` |

**Skills**: `/sc:git` principal

---

### CATEGORIA G — Investigación & Documentación
**Señales**: "investiga", "busca", "documenta", "README", "explica", "aprende", "cómo funciona"

**MCPs que activar**:
- `context7` → documentación oficial de librerías
- `firecrawl` → scraping de docs externas / sitios completos
- `fetch` → APIs y páginas individuales
- `memory` → guardar hallazgos para sesiones futuras
- `context-mode` → búsqueda en codebase local

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `research-analyst` |
| 2 | Soporte Técnico | `documentation-engineer` |
| 3 | Arquitecto | `docs-architect` |
| 4 | Calidad / QA | `code-analyzer` |
| 5 | Seguridad | `security-auditor` |
| 6 | Testing / Integración | `search-specialist` |
| 7 | Documentación | `tutorial-engineer` |
| 8 | Optimización | `context-manager` |
| 9 | Refactoring | `technical-writer` |
| 10 | DevEx / Automatización | `api-documenter` |

**Skills**: `/sc:document`, `/sc:research`, `/sc:explain`

---

### CATEGORIA H — Seguridad & Auditoría
**Señales**: "seguridad", "vulnerabilidad", "pentest", "audit", "OWASP", "hack", "hardening", "compliance", "multitenant", "tenant isolation", "aislamiento de tenants", "auditoría multi-tenant"

**MCPs que activar**:
- `sequential-thinking` → análisis metódico de superficie de ataque
- `filesystem` → revisión de archivos de configuración
- `desktop-commander` → ejecutar herramientas de análisis
- `context-mode` → búsqueda en codebase de patrones inseguros

**Skill prioritaria para auditorías profundas**:
- **`multitenant-security-audit`** → auditoría completa de 12 fases (reconocimiento, aislamiento multi-tenant, autenticación, backend, base de datos, caché/colas, frontend, secretos, infraestructura, OWASP Top 10, logging/monitoreo, performance/DoS). Usar SIEMPRE que la petición implique auditar un proyecto SaaS multi-tenant completo, o cuando se mencione "audit", "pentest", "tenant isolation", "OWASP" o "hardening" sobre un proyecto real en el cwd. Produce findings con formato estándar: qué es → por qué es peligroso → cómo se explota → fix exacto → severidad. Cargar primero el `SKILL.md` y luego los archivos de `references/` por fase.

**Squad de 10 agentes** (complementan a la skill para ejecutar los fixes y validar):

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `security-auditor` |
| 2 | Soporte Técnico | `penetration-tester` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `ad-security-reviewer` |
| 5 | Seguridad | `backend-security-coder` |
| 6 | Testing / Integración | `frontend-developer` |
| 7 | Documentación | `security-auditor` |
| 8 | Optimización | `compliance-auditor` |
| 9 | Refactoring | `security-engineer` |
| 10 | DevEx / Automatización | `powershell-security-hardening` |

**Skills**: `multitenant-security-audit` (prioritaria para SaaS multi-tenant), `/sc:analyze` con foco en seguridad

---

### CATEGORIA I — Revisión & Refactoring
**Señales**: "revisa", "refactoriza", "mejora", "optimiza código", "limpia", "code review", "technical debt"

**MCPs que activar**:
- `code-review-graph` → **PRIMERO** si existe `.code-review-graph/`: mapa completo del módulo, dependencias, comunidades de código relacionadas. Base para decidir qué refactorizar sin romper nada.
- `context-mode` → indexar y buscar patrones en codebase
- `sequential-thinking` → para refactors complejos
- `filesystem` → leer código a revisar

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `code-reviewer` |
| 2 | Soporte Técnico | `refactoring-specialist` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `code-analyzer` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `docs-architect` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `code-analyzer` |
| 10 | DevEx / Automatización | `dependency-manager` |

**Skills**: `/sc:improve`, `/sc:cleanup`, `/sc:analyze`

---

### CATEGORIA J — UI/UX & Diseño
**Señales**: "diseña", "UI", "interfaz", "componente visual", "landing", "dashboard", "estilo", "colores", "componente React", "genera UI"

**MCPs que activar**:
- `magic` (21st.dev) → generación de componentes UI con múltiples variantes de estilo — PRIORIZAR para componentes React/Next.js
- `context7` → documentación de la librería UI detectada
- `playwright` → captura visual del resultado
- `filesystem` → leer tokens/estilos existentes

**Cuándo usar `magic` vs `/ui-ux-pro-max`**:
- Necesitas componente React listo para usar → `magic` (21st.dev)
- Necesitas diseño conceptual / sistema de diseño → `/ui-ux-pro-max`
- Ambos juntos → diseño con `/ui-ux-pro-max` → implementación con `magic`

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `ui-designer` |
| 2 | Soporte Técnico | `frontend-developer` |
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `ui-visual-validator` |
| 5 | Seguridad | `frontend-developer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `accessibility-tester` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Skills**: `/ui-ux-pro-max` (67 estilos, 96 paletas, 57 font pairings), `/sc:design`, `/sc:implement`

---

### CATEGORIA K — IA & Machine Learning
**Señales**: "modelo", "LLM", "RAG", "embeddings", "fine-tuning", "pipeline ML", "predicción", "NLP"

**MCPs que activar**:
- `context7` → docs de frameworks ML (LangChain, HuggingFace, etc.)
- `firecrawl` → scraping de papers o datasets
- `fetch` → APIs de modelos
- `sequential-thinking` → arquitectura de sistemas IA complejos

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `ai-engineer` |
| 2 | Soporte Técnico | `llm-architect` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `ml-engineer` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `machine-learning-engineer` |
| 7 | Documentación | `mlops-engineer` |
| 8 | Optimización | `prompt-engineer` |
| 9 | Refactoring | `llm-architect` |
| 10 | DevEx / Automatización | `nlp-engineer` |

**Skills**: `/sc:design` para arquitectura, `/sc:implement` para código

---

### CATEGORIA L — Datos & Analytics
**Señales**: "datos", "análisis", "dashboard", "métricas", "ETL", "pipeline de datos", "visualización", "CSV"

**MCPs que activar**:
- `postgres` / `sqlite` / `mongodb` / `supabase` → según la fuente de datos
- `firecrawl` → si los datos vienen de web
- `fetch` → si vienen de APIs

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `data-analyst` |
| 2 | Soporte Técnico | `data-engineer` |
| 3 | Arquitecto | `data-engineer` |
| 4 | Calidad / QA | `data-scientist` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `business-analyst` |
| 8 | Optimización | `database-optimizer` |
| 9 | Refactoring | `data-researcher` |
| 10 | DevEx / Automatización | `quant-analyst` |

**Skills**: `/sc:analyze`, `/sc:research`

---

### CATEGORIA M — Redes & Cisco Packet Tracer

> **⚡ EJECUCIÓN DIRECTA — SIN AGENTES**: Delega al skill `cisco-packet-tracer-orchestrator` que tiene el flujo completo. Usar MCP `packet-tracer` directamente.

**Señales MODO NORMAL** (práctica libre, laboratorio, casa):
- "topología", "red", "router", "switch", "cisco", "VLAN", "IP", "routing", "packet tracer", "simula red", "crea una topología", "construye red"

**Señales MODO EXAMEN** (parcial, examen, taller calificado):
- "modo examen", "estoy en examen", "parcial en clase"
- "analiza lo que armé", "ya armé la topología", "listo analiza"
- "configura lo que ya está armado", "lee mi topología", "copia lo que hice"

**MCP disponible**:
- `packet-tracer` (26 tools) → `pt_full_build`, `pt_live_deploy`, `pt_plan_topology`, `pt_query_topology`, `pt_query_links`, `pt_send_raw`, `pt_configure_ip`, `pt_configure_server_static`, etc.

**Defaults obligatorios**: Router = `Router-PT`, Switch = `2960-24TT`

**Dos modos con flujos técnicos diferenciados:**

```
MODO NORMAL — Con tiempo disponible

  A) Topología nueva desde cero:
    → Recopilar requisitos del usuario (routers, PCs, routing, DHCP, NAT...)
    → pt_full_build(routers=N, pcs_per_lan=N, routing="...", router_model="Router-PT", deploy=true)
    → Si necesita NAT/ACL (no soportados por generador):
      → pt_send_raw('configureIosDevice("RouterX", "...comandos NAT...")')

  B) Configuración de topología existente (sin prisa):
    → pt_query_topology() → escanear dispositivos
    → Mostrar lo encontrado, preguntar qué configurar
    → [INTERPRETAR Y VALIDAR — ver protocolo abajo]
    → Armar bloque CLI por router (incluyendo NAT, helper-address, etc.)
    → pt_send_raw('configureIosDevice("RouterX", "enable\nconfigure terminal\n...")')
    → Para PC-PT/Laptop-PT: pt_send_raw('configurePcIp("PC0", true)')
    → Para Server-PT (solo IP): pt_configure_ip(device="Server0", dhcp=True)
      ⚠️ configurePcIp NO funciona en Server-PT
    → Para Server-PT (IP + HTTP o DHCP pool): pt_configure_server_static(device="Server0", ip="...", http_enabled=True)
    → Explicar lo hecho, sugerir verificaciones

MODO EXAMEN — Velocidad máxima bajo presión académica

  Contexto: solo PT + CMD abiertas, profesor vigila, tiempo límite.
  El usuario armó la topología manualmente y dicta las configs del tablero/hoja.

  ⚠️ NUNCA pt_full_build ni pt_live_deploy (crearían duplicados).
  ⚠️ Cero texto innecesario. Solo acción y confirmación breve.
  ⚠️ Preguntar ANTES de ejecutar. Mejor preguntar 3 veces que equivocarse.
  ⚠️ NUNCA mezclar configuración de routers y activación DHCP en el mismo paso.

  FLUJO (4 fases):

  FASE 1 — ESCANEO:
    pt_query_topology() → obtener nombres y modelos reales.
    ESCANEO DE ENLACES via pt_send_raw (NO usar pt_query_links — da timeout):
      pt_send_raw(js_code="var n=ipc.network();var dc=n.getDeviceCount();var linksMap={};for(var i=0;i<dc;i++){var d=n.getDeviceAt(i);if(!d)continue;var dn=d.getName();var pc=d.getPortCount();for(var j=0;j<pc;j++){var p=d.getPortAt(j);if(p&&p.getLink()){var uuid=p.getLink().getObjectUuid();var entry=dn+'['+p.getName()+']';if(!linksMap[uuid]){linksMap[uuid]=entry;}else{linksMap[uuid]=linksMap[uuid]+'<--->'+entry;}}}}var results=[];for(var k in linksMap){results.push(linksMap[k]);}reportResult(results.length>0?results.join('|'):'sin_enlaces');", wait_result=True)
    Parsear resultado (separador "|") → mapa físico completo.
    Mostrar: "Detecté: [dispositivos] con [N] enlaces."

  FASE 2 — INTERPRETAR Y VALIDAR (UN solo mensaje):
    El usuario dicta la config del tablero/hoja. Puede ser caótica, mal redactada,
    con IPs sueltas, sin orden, o mezclar datos correctos con errores tipográficos.
    Claude NUNCA asume que falta algo sin antes EXTRAER todo lo que hay.

    PATRONES RECONOCIDOS (clasificar durante extracción):
    • "DATO IMPORTANTE:" / "DATO SUPER IMPORTANTE:" / "CORRECCIÓN:" → prioridad máxima.
    • Tabla VLSM (subred base + host counts) → calcular tabla ANTES de asignar IPs.
    • DHCP multi-fuente ("PC0→DHCP del Servidor0", "Laptop→DHCP del Switch1") → {dev→fuente}.
    • VLANs con rangos de puertos → mapear puertos y fuente DHCP por cada VLAN.
    • NAT global ("RouterX debe brindar NAT") → inside/outside + ruta default en todos.
    • IP estática mezclada con DHCP → distinguir explícitamente en el plan.
    • Inconsistencia header/cuerpo (header dice R0, link dice R1) → flagear [CONFLICTO].

    PASO A — EXTRAER (siempre primero):
      Leer el input completo del usuario y clasificar cada dato en los patrones anteriores.
      Si hay tabla VLSM → calcularla primero (mayor a menor host count).
      No juzgar, no completar todavía.
      Anotar toda inconsistencia o [CONFLICTO] antes de continuar.

    PASO B — COMPLETAR (solo lo verdaderamente ausente):
      Solo para datos que NO aparecen en ninguna forma en el input:
       IPs no dadas → usar subredes VLSM si las calculaste; si no: 192.168.X.0/24 LANs, 10.0.X.X/30 WAN
       DNS → 8.8.8.8 | Clock rate → 64000 seriales | DHCP no mencionado → asumir SÍ
       Routing no especificado → PREGUNTAR, no asumir
      ⚠️ Si el dato está presente aunque mal escrito → NO completar con default,
         corregirlo y marcarlo como "[corregido]".

    PASO C — IR DIRECTO A EJECUCIÓN (sin confirmación en MODO EXAMEN):
      No presentar plan ni pedir "¿es correcto?". Solo ejecutar.
      Mostrar resumen de 3 líneas antes de inyectar:
      "Configurando: Router0 Fa0/0=10.x.x.1/24, Se2/0=10.0.0.1/30 | RIP V2 | NAT en Router0."
      → Continuar a FASE 3 inmediatamente.
      SOLO PAUSAR si hay [CONFLICTO CRÍTICO] que haría fallar toda la config:
      → "⚠️ [pregunta de 1 línea — ej: ¿NAT en Router0 o Router1?]"
      → Esperar respuesta y ejecutar sin más preguntas.
      ⚠️ Routing ausente → única excepción: preguntar antes de asumir.
      Todos los demás defaults → aplicar sin avisar.

  FASE 3 — PASO A: CONFIGURAR ROUTERS + IPs ESTÁTICAS (en paralelo):
    ⛔ NO activar DHCP en los hosts aquí.
    En UN SOLO mensaje paralelo enviar AMBOS grupos:
    GRUPO A — Routers:
      Para cada router → armar bloque CLI completo e inyectar:
        pt_send_raw('configureIosDevice("RouterX", "enable\nconfigure terminal\n...")')
      (todos los routers EN PARALELO)
    GRUPO B — Dispositivos con IP estática (servidores, PCs fijos):
      → Identificados en FASE 2 PASO A/B con IP fija asignada.
      → No dependen del IOS engine → se envían en el mismo mensaje que los routers.
        ⚠️ BUG CONOCIDO: configurePcIp() NO funciona en Server-PT — usar pt_configure_ip o pt_configure_server_static:
        pt_configure_ip(device="Server0", ip="10.x.x.x", mask="255.x.x.x", gateway="10.x.x.x")
        pt_configure_server_static(device="Server0", ip="10.x.x.x", mask="255.x.x.x", gateway="10.x.x.x", http_enabled=True)
        Para PC-PT/Laptop-PT con IP estática SÍ se puede usar configurePcIp:
        pt_send_raw('configurePcIp("PC0", false, "10.x.x.x", "255.x.x.x", "10.x.x.x")')
      (todos los estáticos EN PARALELO con los routers)
    Reglas de blindaje Anti-Fallos PT obligatorias:
      • clock rate 64000 → SIEMPRE en interfaces seriales DCE.
      • STP wait → si hay Switch, incluir time.sleep(30) en el script Python después de activar puertos hacia Switches.
      • Nombre lógico CLI: usar el nombre real de interfaz que devolvió pt_query_links()
        (ej: si el API dijo "FastEthernet0/0", usar "Fa0/0" en el bloque CLI).
    ⚠️ MENSAJE OBLIGATORIO AL USUARIO DESPUÉS DE ESTE PASO:
      "¡IMPORTANTE! Abre manualmente la pestaña 'CLI' de cada router en PT al menos una vez
       para despertar el motor IOS. Cuando termines, respóndeme LISTO."

  FASE 4 — PASO B: ACTIVAR DHCP EN HOSTS (solo tras confirmación):
    → Solo ejecutar cuando el usuario responda "LISTO".
    → Solo aplica a hosts con DHCP (PCs, Laptops sin IP estática asignada).
    → Para cada host DHCP: renovar IP 2 veces (false → true) para forzar nueva solicitud.
      Para PC-PT/Laptop-PT:
        pt_send_raw('configurePcIp("PC0", false, "0.0.0.0", "0.0.0.0", "0.0.0.0")')
        pt_send_raw('configurePcIp("PC0", true)')
      Para Server-PT (configurePcIp NO funciona):
        pt_configure_ip(device="Server0", dhcp=True)
    (todos los hosts DHCP EN PARALELO en un solo mensaje)
    → "¡Red operativa! Verifica con ping [gateway] desde cada PC."
```

**Funciones JS reales en pt_send_raw:**
- `configureIosDevice(name, cliBlock)` → aplica CLI IOS completo a router/switch existente
- `configurePcIp(name, dhcp)` o `configurePcIp(name, false, ip, mask, gw)` → IP de PC-PT y Laptop-PT SOLAMENTE
- `addDevice / addLink` → solo para topología nueva

**Tools MCP para end-devices:**
- `pt_configure_ip(device, dhcp, ip, mask, gateway, dns)` → configura IP estática o DHCP en cualquier end-device (PC-PT, Server-PT, Laptop-PT). Usa API IPC directa — funciona donde `configurePcIp()` falla silenciosamente.
- `pt_configure_server_static(device, ip, mask, gateway, dns, http_enabled, dhcp_pool_start, dhcp_pool_end)` → especializada para Server-PT: configura IP estática y opcionalmente activa servicio HTTP y/o configura pool DHCP gestionado por el servidor.

**Reglas clave:**
- Siempre Router-PT y Switch 2960-24TT (pasar `router_model="Router-PT"` a pt_full_build)
- 2+ routers → preguntar: ¿serial o ethernet? ¿qué enrutamiento?
- clock rate 64000 → SIEMPRE en enlaces seriales (no opcional)
- STP 30s → SIEMPRE esperar antes de pedir DHCP cuando hay Switches
- Si faltan puertos → pausar, indicar qué módulo agregar manualmente en PT
- NAT/ACL/VLANs/helper-address → incluir en el bloque CLI de configureIosDevice (no soportados por generador)
- NUNCA spawning agentes, NUNCA inventar datos, NUNCA usar tools que no existen


### CATEGORIA N — Performance & Monitoreo
**Señales**: "lento", "optimiza performance", "profiling", "métricas", "logs", "monitoreo", "bottleneck"

**MCPs que activar**:
- `desktop-commander` → ejecutar herramientas de profiling
- `sequential-thinking` → análisis metódico de bottlenecks
- `context-mode` → búsqueda eficiente en codebase

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `performance-engineer` |
| 2 | Soporte Técnico | `performance-monitor` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `performance-benchmarker` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `database-optimizer` |
| 9 | Refactoring | `observability-engineer` |
| 10 | DevEx / Automatización | `sre-engineer` |

**Skills**: `/sc:analyze`, `/sc:improve`

---

### CATEGORIA O — Web Scraping & Automatización Web
**Señales**: "scraping", "extrae datos web", "automatiza browser", "llena formulario", "navega a", "descarga de sitio"

**MCPs que activar**:
- `firecrawl` → scraping masivo con output Markdown limpio (preferir para docs y sitios completos)
- `playwright` → automatización precisa con árbol de accesibilidad (preferir para interacción)
- `puppeteer` → screenshots, formularios simples
- `fetch` → APIs y páginas individuales simples

**Decisión firecrawl vs playwright**:
- Scraping de contenido → `firecrawl`
- Interacción con UI (clicks, formularios) → `playwright`
- Screenshots → `puppeteer`
- Simple GET de JSON → `fetch`

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `data-researcher` |
| 2 | Soporte Técnico | `data-engineer` |
| 3 | Arquitecto | `data-engineer` |
| 4 | Calidad / QA | `qa-expert` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `refactoring-specialist` |
| 10 | DevEx / Automatización | `devops-engineer` |

---

### CATEGORIA P — Proyecto & Planificación
**Señales**: "planifica", "roadmap", "sprint", "estimación", "arquitectura del sistema", "PRD", "spec", "nuevo proyecto", "fase", "milestone", "hito"

**MCPs que activar**:
- `sequential-thinking` → planificación estructurada
- `memory` → guardar decisiones y contexto del proyecto
- `github` → si hay issues/milestones que crear

**GSD — Cuándo usar cada comando** (GSD v1.29.0 instalado globalmente):

| Situación | Comando GSD |
|-----------|-------------|
| Proyecto nuevo desde cero | `/gsd:new-project` — genera contexto profundo y roadmap |
| Iniciar nuevo hito/ciclo | `/gsd:new-milestone` — actualiza PROJECT.md |
| Planificar fase en detalle | `/gsd:plan-phase` — genera PLAN.md con Nyquist validation |
| Ejecutar fase completa | `/gsd:execute-phase` — wave-based con agentes paralelos |
| Avanzar automáticamente | `/gsd:next` — detecta qué sigue y lo ejecuta |
| Ver progreso actual | `/gsd:progress` — estado de fases, context y routing |
| Tarea rápida sin subagentes | `/gsd:fast` — inline, sin overhead |
| Tarea media con garantías | `/gsd:quick` — GSD guarantees, minimal agents |
| Todo autónomo | `/gsd:autonomous` — ejecuta TODAS las fases restantes |
| Pausar y retomar sesión | `/gsd:pause-work` + `/gsd:resume-work` |
| Capturar idea al vuelo | `/gsd:note` o `/gsd:add-backlog` |
| PR listo para merge | `/gsd:ship` |
| Revisar estado del vault | Leer `C:/Users/Andrea/Documents/MiVault/STATE.md` |

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `project-manager` |
| 2 | Soporte Técnico | `product-manager` |
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `architect-review` |
| 5 | Seguridad | `risk-manager` |
| 6 | Testing / Integración | `conductor-validator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `scrum-master` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Skills**: `/gsd:new-project` (proyectos nuevos), `/gsd:autonomous` (ejecución completa), `/sc:pm`, `/sc:estimate`, `/sc:spec-panel`, `/sc:brainstorm`, `/sc:workflow`

---

### CATEGORIA Q — Integración de Datos con Google Stitch
**Señales**: "stitch", "pipeline de datos", "conecta fuente de datos", "sincroniza datos", "ingesta", "ETL cloud", "warehouse", "BigQuery", "replica datos"

**MCPs que activar**:
- `stitch` → operaciones directas de Google Stitch: gestionar pipelines, fuentes, destinos, sincronizaciones
- `sequential-thinking` → para diseño de flujos de ingesta complejos
- `memory` → guardar configuraciones de pipelines para reutilizar
- `postgres` / `supabase` / `mongodb` → según el destino de los datos

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `data-engineer` |
| 2 | Soporte Técnico | `data-engineer` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `data-analyst` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `database-optimizer` |
| 9 | Refactoring | `data-scientist` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Flujo típico**:
1. `stitch` → listar fuentes disponibles
2. `sequential-thinking` → planificar el pipeline
3. `stitch` → configurar fuente + destino
4. `stitch` → iniciar sincronización
5. BD destino → verificar datos llegaron correctamente

**Skills**: `/sc:design` para arquitectura, `/sc:implement` para automatización

---

### CATEGORIA R — IA Multimodal con Gemini (Nano Banana)
**Señales**: "gemini", "nano banana", "genera imagen", "analiza imagen", "vision IA", "multimodal", "audio IA", "video IA", "Gemini Flash", "Gemini Pro"

> **Configuración**: `nanobanana` MCP instalado en `~/.claude.json`, usa `GEMINI_API_KEY` de Google AI Studio

**MCPs que activar**:
- `nanobanana` → acceso directo a modelos Gemini vía MCP: visión, texto, multimodal, embeddings
- `sequential-thinking` → para pipelines multimodales complejos
- `memory` → guardar resultados de análisis para reutilizar

**Cuándo usar `nanobanana` vs `ai-engineer`**:
- Necesitas Gemini específicamente (no otro modelo) → `nanobanana`
- Tareas de visión (analizar imagen, PDF, video) → `nanobanana`
- Construcción de app con múltiples LLMs → `ai-engineer` + `nanobanana`

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `ai-engineer` |
| 2 | Soporte Técnico | `ml-engineer` |
| 3 | Arquitecto | `llm-architect` |
| 4 | Calidad / QA | `qa-expert` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `machine-learning-engineer` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `prompt-engineer` |
| 9 | Refactoring | `llm-architect` |
| 10 | DevEx / Automatización | `devops-engineer` |

**Skills**: `/sc:implement` para integración, `/sc:design` para arquitectura

---

### CATEGORIA S — Claude-Flow v3 (Swarm Intelligence)
**Señales**: "swarm", "múltiples agentes en paralelo", "tarea compleja con muchos pasos", "orquesta agentes", "SPARC", "razonamiento distribuido", "84% solve rate"

**MCP que activar**:
- `claude-flow` → 170+ herramientas de swarm, SONA neural router, AgentDB vector search

**Cuándo usar claude-flow en lugar de agentes individuales**:
- Tarea requiere 5+ agentes coordinados
- Necesitas memoria persistente entre pasos (AgentDB)
- Quieres aprendizaje automático del routing (SONA)
- Proyecto grande con múltiples dominios simultáneos

**Squad de 10 agentes** *(skills claude-flow)*:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `swarm-orchestration` |
| 2 | Soporte Técnico | `swarm-advanced` |
| 3 | Arquitecto | `sparc:architect` |
| 4 | Calidad / QA | `verification-quality` |
| 5 | Seguridad | `sparc:security-review` |
| 6 | Testing / Integración | `sparc:tdd` |
| 7 | Documentación | `sparc:docs-writer` |
| 8 | Optimización | `agentdb-optimization` |
| 9 | Refactoring | `sparc:code` |
| 10 | DevEx / Automatización | `sparc:devops` |

**Skills de claude-flow disponibles**:

| Categoría | Skills | Para qué |
|-----------|--------|----------|
| **Swarm** | `swarm-orchestration`, `swarm-advanced` | Orquestar enjambres de agentes en paralelo |
| **SPARC** | `sparc:sparc`, `sparc:orchestrator`, `sparc:code`, `sparc:architect`, `sparc:tdd`, `sparc:debug`, `sparc:devops`, `sparc:security-review`, `sparc:docs-writer` | Metodología completa Spec→Code→Test→Deploy |
| **AgentDB** | `agentdb-vector-search`, `agentdb-memory-patterns`, `agentdb-learning`, `agentdb-optimization` | Memoria vectorial 150x más rápida |
| **GitHub** | `github:code-review-swarm`, `github:pr-manager`, `github:release-manager`, `github:workflow-automation`, `github:repo-architect` | GitHub con swarm IA |
| **Análisis** | `analysis:bottleneck-detect`, `analysis:performance-bottlenecks`, `analysis:token-efficiency` | Detectar cuellos de botella |
| **Automation** | `automation:self-healing`, `automation:session-memory`, `automation:smart-agents` | Workflows auto-reparables |
| **Monitoring** | `monitoring:swarm-monitor`, `monitoring:status`, `monitoring:real-time-view` | Estado del swarm en tiempo real |
| **Optimization** | `optimization:parallel-execution`, `optimization:auto-topology` | Ejecución paralela y topología óptima |
| **ReasoningBank** | `reasoningbank-intelligence`, `reasoningbank-agentdb` | Aprendizaje adaptativo de patrones |

**Flujo SPARC para features grandes**:
```
sparc:spec-pseudocode → sparc:architect → sparc:code → sparc:tdd → sparc:integration → sparc:post-deployment-monitoring-mode
```

**Comandos clave**:
- `claude-flow-swarm` → coordinar swarm para tarea compleja
- `claude-flow-memory` → gestionar memoria entre sesiones
- `sparc:sparc` → orquestador SPARC completo

---

### CATEGORIA S2 — Negocio & Marketing
**Señales**: "startup", "modelo de negocio", "competencia", "marketing", "SEO", "contenido", "ventas"

**MCPs que activar**:
- `firecrawl` → investigación de competidores
- `fetch` → datos de mercado
- `memory` → guardar insights

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `business-analyst` |
| 2 | Soporte Técnico | `market-researcher` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `competitive-analyst` |
| 5 | Seguridad | `startup-analyst` |
| 6 | Testing / Integración | `trend-analyst` |
| 7 | Documentación | `content-marketer` |
| 8 | Optimización | `seo-specialist` |
| 9 | Refactoring | `seo-content-auditor` |
| 10 | DevEx / Automatización | `seo-keyword-strategist` |

**Skills**: `/sc:research`, `/sc:business-panel`

---

### CATEGORIA U — Browser Debugging con Chrome DevTools
**Señales**: "inspecciona el navegador", "qué pasa en el browser", "errores de consola", "network tab", "fallos de red", "XHR/fetch falla", "DOM inspection", "performance del browser", "memory leak browser", "ve qué pasa en chrome", "devtools"

> **⚡ EJECUCIÓN DIRECTA — PRIORIZAR `chrome-devtools` MCP**: Conecta directamente con Chrome DevTools Protocol para inspección en tiempo real sin screenshots.

**MCPs que activar**:
- `chrome-devtools` → PRINCIPAL — acceso directo a consola, red, DOM, performance, storage, coverage
- `sequential-thinking` → para diagnóstico estructurado de problemas complejos
- `context-mode` → correlacionar errores del browser con el código fuente local
- `playwright` → si necesitas automatizar interacciones mientras inspeccionas

**Capacidades del MCP `chrome-devtools`**:
| Capacidad | Cuándo usarla |
|-----------|---------------|
| Console logs / errors | Capturar errores JS, warnings, stack traces en tiempo real |
| Network inspector | Ver requests/responses fallidos, headers, payloads, CORS |
| DOM inspector | Leer/modificar el DOM en vivo sin intervención manual |
| Performance profiler | Detectar renders lentos, memory leaks, layout thrashing |
| Storage inspector | Ver cookies, localStorage, sessionStorage, IndexedDB |
| Coverage | Identificar CSS/JS no utilizado |
| Source maps | Navegar hasta el archivo fuente exacto desde el error |

**Flujo de diagnóstico con chrome-devtools**:
```
1. Conectar → abrir Chrome con --remote-debugging-port=9222
2. chrome-devtools → capturar console errors + network failures
3. sequential-thinking → analizar causa raíz con evidencia real
4. context-mode → encontrar el archivo fuente responsable
5. Aplicar fix → verificar en browser que desapareció el error
```

**Para tu proyecto Angular/Ionic (Zolvex)**:
- Errores NG0100, NG02, NG04 → capturar stack trace completo del browser
- Requests 404 al backend → inspeccionar Network tab para ver headers y payload exacto
- Animaciones/CSS glitches → DOM inspector + computed styles en tiempo real
- Change detection loops → Performance profiler para detectar renders excesivos

**Squad de 10 agentes** (cuando el diagnóstico requiere múltiples dimensiones):

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `debugger` |
| 2 | Soporte Técnico | `frontend-developer` |
| 3 | Arquitecto | `architect-review` |
| 4 | Calidad / QA | `devops-troubleshooter` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `accessibility-tester` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `error-detective` |
| 10 | DevEx / Automatización | `devops-incident-responder` |

**Skills**: `/sc:troubleshoot` + `chrome-devtools` MCP directo

---

### CATEGORIA T — Video & Motion Graphics (Remotion)
**Señales**: "video", "animación", "motion graphics", "remotion", "render MP4", "kinetic typography", "social content", "producto demo", "bar chart animado", "template 9:16", "video programático"

> **Proyecto base**: `C:/Users/Andrea/remotion-demo/` — ya configurado con Docker, listo para renderizar.

**Stack instalado**:
- `remotion` + `@remotion/player` + `@remotion/cli` — motor de video en React
- Skill `remotion-best-practices` — instalada en `~/.agents/skills/remotion-best-practices`
- Docker 100% headless — render sin GUI, sin instalar nada local

**Flujo de trabajo**:
```
CREAR componente en src/[Nombre].tsx (React + Remotion hooks)
  → Registrar en src/Root.tsx como <Composition>
  → Renderizar: npm run docker:render → sale en ./out/[Nombre].mp4
  → Preview visual: npm run docker:studio → http://localhost:3000
```

**Hooks Remotion más usados**:
| Hook | Para qué |
|------|----------|
| `useCurrentFrame()` | Frame actual (0 a durationInFrames) |
| `useVideoConfig()` | fps, width, height, durationInFrames |
| `interpolate(frame, [from], [to])` | Animar cualquier valor entre frames |
| `spring({frame, fps})` | Animación física con rebote |
| `AbsoluteFill` | Contenedor que ocupa todo el canvas |
| `Sequence` | Sincronizar múltiples componentes en el tiempo |

**Casos de uso**:
| Tipo | Ejemplo de prompt |
|------|-------------------|
| Motion Graphics | "Genera animación de bar chart creciente para presentación de cliente" |
| Social Content | "Crea template 9:16 con texto animado sincronizado a 120BPM" |
| Product Demo | "Construye walkthrough de UI con colores de mi marca" |
| Múltiples versiones | "Haz 5 versiones del video con distintos titulares" |

**Comandos Docker**:
```bash
npm run docker:render   # Genera MP4 en ./out/
npm run docker:studio   # Preview en http://localhost:3000
docker compose build    # Reconstruir imagen (si cambia Dockerfile)
```

**Squad de 10 agentes**:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `frontend-developer` |
| 2 | Soporte Técnico | `react-specialist` |
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `qa-expert` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `refactoring-specialist` |
| 10 | DevEx / Automatización | `docker-expert` |

**Skills**: `/sc:implement` para componentes React/Remotion, `/sc:design` para storyboard

---

### CATEGORIA U — Errores & Monitoreo con Sentry

> **⚡ EJECUCIÓN DIRECTA para consultas simples**: Para queries de una sola acción (listar issues, ver un error), usar MCP `sentry` directamente sin agentes. Para auditorías completas o análisis multi-proyecto → Squad de 10 agentes.

**Señales**: "errores en sentry", "qué errores hay", "audita errores", "stack trace", "crash", "exception en producción", "error rate", "issues activos", "sentry", "monitoreo de errores"

**Organización Sentry**: `mauricio-org` en `https://mauricio-org.sentry.io`

**MCP que activar**: `sentry` → acceso directo a issues, eventos, stack traces, performance, alertas

**Cuándo usar MCP directo (sin agentes)**:
- "muéstrame los errores activos" → `sentry` directo
- "stack trace del issue #xxx" → `sentry` directo
- "cuántos errores hubo hoy" → `sentry` directo

**Cuándo usar Squad (auditoría completa)**:
- "auditame todos los errores y dame un plan de acción"
- "analiza tendencias de errores del último mes"
- "prioriza los bugs más críticos de producción"

**Squad de 10 agentes** *(para auditorías completas)*:

| # | Dimensión | Agente |
|---|-----------|--------|
| 1 | Líder / Principal | `debugger` |
| 2 | Soporte Técnico | `error-detective` |
| 3 | Arquitecto | `architect-reviewer` |
| 4 | Calidad / QA | `qa-expert` |
| 5 | Seguridad | `security-engineer` |
| 6 | Testing / Integración | `test-automator` |
| 7 | Documentación | `documentation-engineer` |
| 8 | Optimización | `performance-engineer` |
| 9 | Refactoring | `refactoring-specialist` |
| 10 | DevEx / Automatización | `sre-engineer` |

**Flujo de auditoría completa**:
```
sentry → list_issues (todos los proyectos)
  → clasificar por severidad (fatal > error > warning)
  → get_issue_details (top 5 más frecuentes)
  → Squad analiza causa raíz + propone fixes
  → documentation-engineer genera reporte priorizado
```

**Skills**: `/sc:troubleshoot`, `/sc:analyze`

---

## Fase 2 — Estrategia de Ejecución (Arquitectura v2)

### Modo PARALELO (por defecto — 10 agentes por categoría)

Todas las categorías lanzan sus 10 agentes simultáneamente organizados en 3 sub-bloques concurrentes:

```
SUB-BLOQUE CORE (arrancan primero, producen el output principal):
  - Agente 1 (Líder): ejecuta la tarea central
  - Agente 2 (Soporte): complementa al líder
  - Agente 3 (Arquitecto): valida decisiones de diseño en tiempo real

SUB-BLOQUE QUALITY GATE (trabajan en paralelo al core, validan conforme se produce):
  - Agente 4 (QA): revisa output del core continuamente
  - Agente 5 (Seguridad): audita por vulnerabilidades
  - Agente 6 (Testing): genera y ejecuta tests conforme el código se escribe
  - Agente 9 (Refactoring): limpia y mejora el código en caliente

SUB-BLOQUE DELIVERY (trabajan en paralelo, preparan el entregable final):
  - Agente 7 (Docs): documenta conforme se construye
  - Agente 8 (Optimización): perfila y optimiza rendimiento
  - Agente 10 (DevEx): configura CI/CD, linters, workflows

Los 3 sub-bloques son CONCURRENTES entre sí. No esperan turnos.
```

**Ejemplo**:
```
/orquesta Crea un sistema de autenticación JWT en Node.js
→ Categoría A detectada (stack: Node.js/Express)
→ Lanzamiento paralelo de 10 agentes:
  CORE:
    - Agente 1: backend-developer → implementa JWT + refresh tokens
    - Agente 2: javascript-pro → helpers, utilities, tipado estricto
    - Agente 3: architect-reviewer → valida estructura de módulos y patrones
  QUALITY GATE:
    - Agente 4: qa-expert → tests unitarios + integración desde el minuto 0
    - Agente 5: security-engineer → audita tokens, expiración, almacenamiento
    - Agente 6: test-automator → suite E2E del flujo completo auth
    - Agente 9: refactoring-specialist → aplica SOLID, limpia handlers
  DELIVERY:
    - Agente 7: documentation-engineer → documenta endpoints, flujo auth
    - Agente 8: performance-engineer → optimiza middleware, caché, payload
    - Agente 10: devops-engineer → configura eslint, prettier, husky hooks, CI
```

### Modo MULTI-CATEGORIA (híbrido — múltiples squads)

Cuando la tarea cruza múltiples categorías, cada categoría despliega sus 10 agentes en paralelo. La coordinación entre categorías es secuencial cuando hay dependencias, pero intra-categoría siempre es paralela.

```
/orquesta Crea una SaaS de gestión de tareas con Next.js y Supabase
→ Categorías: A (código) + B (BD) + D (tests) + P (planificación)
→ Fase 1: Categoría P (10 agentes) → planificación, arquitectura y roadmap
→ Fase 2 paralela: Categoría A (10 agentes) + Categoría B (10 agentes)
→ Fase 3: Categoría D (10 agentes) → testing integral y validación cruzada
→ Total: 40 agentes coordinados en 3 fases
```

### Modo SWARM (tareas masivas — 4+ categorías)

Para tareas que activan 4+ categorías simultáneamente, escalar a claude-flow swarm orchestration. Cada categoría opera como un "pod" de 10 agentes, y claude-flow coordina la comunicación entre pods usando SONA neural router + AgentDB para memoria compartida.

```
/orquesta Crea un marketplace completo con pagos, auth, admin y API pública
→ Categorías: A + B + D + E + H + J + P = 7 categorías
→ claude-flow swarm: 70 agentes en pods coordinados
→ SONA enruta mensajes entre pods automáticamente
→ AgentDB mantiene estado compartido entre todos los agentes
```

---

## Fase 3 — Reglas de Optimización de Tokens

1. **Siempre primero**: `context-mode` indexa en fondo → búsqueda semántica antes de leer archivos
2. **qmd antes que Read**: Si el proyecto ya fue indexado, buscar con qmd
3. **read-once**: El hook evita releer archivos — confiar en él
4. **memory MCP**: Al inicio de cada sesión, recuperar contexto previo del proyecto
5. **Agentes con contexto mínimo**: Pasar solo la información relevante a cada agente, no todo el contexto
6. **Usar Glob/Grep antes que Read**: Para encontrar dónde está el código antes de leerlo
7. **context7 sobre búsqueda manual**: Para docs de librerías, nunca leer archivos de node_modules
8. **Contexto segmentado por dimensión**: Cada uno de los 10 agentes recibe SOLO la información relevante a su dimensión. El agente de QA no necesita la documentación de la API; el agente de documentación no necesita los detalles del profiling. Esto reduce el consumo de tokens por agente en ~50-65%.
9. **Sub-bloque broadcasting**: Los 3 sub-bloques (Core, Quality Gate, Delivery) comparten un canal de broadcast ligero. Cuando el Core produce un archivo, Quality Gate y Delivery reciben solo la referencia (ruta + hash), no el contenido completo. Cada agente lee solo lo que necesita.
10. **Consolidación de output con arbitraje**: Los 10 agentes entregan sus resultados a un paso de síntesis final que unifica el entregable. Reglas de arbitraje:
    - Seguridad > Performance (si hay conflicto, gana seguridad)
    - Arquitecto > Refactoring (si hay conflicto de patrones, gana arquitecto)
    - QA > Líder (si QA detecta un bug, el output del líder se corrige)
    - DevEx aplica último (configura tooling sobre el output final consolidado)
11. **Early termination con reasignación**: Si un agente completa su dimensión antes que los demás, libera su contexto y opcionalmente se reasigna como soporte al agente más lento del mismo sub-bloque. No se mantienen agentes ociosos.
12. **Token budget por agente**: Distribución recomendada del budget total:
    - Líder: 20% | Soporte: 15% | Arquitecto: 10%
    - QA: 10% | Seguridad: 8% | Testing: 10%
    - Docs: 7% | Optimización: 8% | Refactoring: 7% | DevEx: 5%

---

## Fase 4 — Flujo de Ejecución Estándar

```
1. RECUPERAR contexto (en paralelo):
   - memory MCP + AgentDB → contexto técnico previo
   - Leer C:/Users/Andrea/Documents/MiVault/STATE.md → estado de proyectos activos
2. CLASIFICAR tarea → una o más categorías (A-S2)
3. SELECCIONAR herramientas → MCPs + Squad de 10 agentes + Skills óptimos
   - Si es proyecto nuevo → considerar /gsd:new-project primero
   - Si es tarea en proyecto existente con GSD → considerar /gsd:next o /gsd:execute-phase
4. INFORMAR al usuario → "Voy a usar X, Y, Z porque..." + mostrar sub-bloques
5. EJECUTAR → 10 agentes en paralelo organizados en Core / Quality Gate / Delivery
6. CONSOLIDAR → arbitraje de conflictos entre dimensiones (Seguridad > Performance > QA > Líder)
7. VALIDAR → verificar resultado con Quality Gate antes de entregar
8. GUARDAR (en paralelo al terminar):
   - memory MCP → guardar hallazgos técnicos importantes
   - Actualizar MiVault/STATE.md si hubo avance en proyecto
   - Crear MiVault/diario/[fecha].md si fue sesión significativa
   - **EN TIEMPO REAL** (opcional): ejecutar `/obsidian-sync` al inicio de la sesión — activa `obsidian-live-sync.cjs` en background, sincroniza cada mensaje al vault automáticamente
   - **AUTOMÁTICO al cerrar**: el hook SessionEnd ejecuta `obsidian-session-end.cjs` — guarda resumen final en `/diario/[fecha].md` al hacer `/exit`
9. REPORTAR → qué se hizo, con qué herramientas, próximos pasos
```

---

## Ejemplos de Orquestación (Arquitectura v2)

### Ejemplo 1 — Feature completa (10 agentes)
```
/orquesta Crea un sistema de autenticación JWT en Node.js con refresh tokens
```
**Decisión**: Categoría A (código, stack: Node.js/Express)
**MCPs**: context7 (docs JWT), filesystem (estructura proyecto), sequential-thinking (planificación)
**Squad de 10 agentes**:
- **CORE**: backend-developer (JWT + refresh) + javascript-pro (helpers) + architect-reviewer (patrones)
- **QUALITY GATE**: qa-expert (tests) + security-engineer (auditoría tokens) + test-automator (E2E) + refactoring-specialist (SOLID)
- **DELIVERY**: documentation-engineer (docs API) + performance-engineer (middleware) + devops-engineer (CI/CD)
**Skills**: /sc:implement → /sc:test

### Ejemplo 2 — Bug crítico (10 agentes)
```
/orquesta El endpoint /api/users devuelve 500 en producción
```
**Decisión**: Categoría C (debug)
**MCPs**: sequential-thinking + desktop-commander (logs) + filesystem
**Squad de 10 agentes**:
- **CORE**: debugger (análisis raíz) + error-detective (correlación) + wshobson-comprehensive-review-architect-review (diseño)
- **QUALITY GATE**: wshobson-error-diagnostics-debugger (diagnóstico) + security-engineer (vulnerabilidades) + wshobson-debugging-toolkit-debugger (toolkit) + wshobson-error-debugging-debugger (refactor fix)
- **DELIVERY**: documentation-engineer (postmortem) + performance-engineer (profiling) + wshobson-distributed-debugging-devops-troubleshooter (infra)
**Skills**: /sc:troubleshoot

### Ejemplo 3 — Topología de red (skill dedicada)
```
/orquesta Crea una topología con 2 routers, 3 switches y 6 PCs con VLANs
```
**Decisión**: Categoría M (redes) → **invocar `/cisco-pt`** directamente
**MCP**: `packet-tracer` — sin agentes
**Flujo**: `/cisco-pt` detecta modo (Normal/Examen) y ejecuta las 4 fases automáticamente

### Ejemplo 4 — Auditoría de seguridad (10 agentes)
```
/orquesta Audita la seguridad de mi aplicación Express
```
**Decisión**: Categoría H (seguridad)
**MCPs**: sequential-thinking + filesystem + context-mode
**Squad de 10 agentes**:
- **CORE**: security-auditor (auditoría general) + penetration-tester (pentesting) + wshobson-comprehensive-review-architect-review (arquitectura)
- **QUALITY GATE**: wshobson-security-scanning-security-auditor (scanning) + wshobson-backend-api-security-backend-security-coder (API) + wshobson-frontend-mobile-security-frontend-developer (frontend) + security-engineer (controles)
- **DELIVERY**: wshobson-security-compliance-security-auditor (compliance docs) + compliance-auditor (GDPR/SOC2) + powershell-security-hardening (hardening)
**Skills**: /sc:analyze

### Ejemplo 5 — App completa (40 agentes multi-categoría)
```
/orquesta Crea una SaaS de gestión de tareas con Next.js y Supabase
```
**Decisión**: Categorías A + B + D + P = 4 squads de 10
**MCPs**: context7 + supabase + filesystem + sequential-thinking
**Fase 1**: Squad P (10 agentes) → planificación, arquitectura y roadmap
**Fase 2 paralela**: Squad A (10 agentes, stack Next.js) + Squad B (10 agentes, Supabase)
**Fase 3**: Squad D (10 agentes) → testing integral y validación cruzada
**Total**: 40 agentes coordinados en 3 fases, cobertura completa en una sola pasada
**Skills**: /sc:spawn (orquestación multi-agente)

---

## Inventario Completo de Herramientas Disponibles

### MCPs Activos (23)
`packet-tracer` `code-review-graph` `stitch` `context7` `github` `postgres` `sqlite` `filesystem` `fetch` `memory` `sequential-thinking` `playwright` `firecrawl` `desktop-commander` `supabase` `mongodb` `puppeteer` `context-mode` `claude-flow` `nanobanana` `magic` `sentry`

- **code-review-graph** → Grafo de conocimiento local del codebase. Reduce tokens 8x. Mapea funciones, clases, dependencias, blast-radius. 22 tools MCP. Instalado globalmente via `pip install code-review-graph`. Config en `~/.mcp.json`. Usar para: análisis de impacto de cambios, contexto de arquitectura, onboarding a nuevo proyecto, pre-merge checks. CLI: `code-review-graph build` (construir grafo) · `code-review-graph update` (actualización incremental) · `code-review-graph status` (estadísticas) · `code-review-graph serve` (servidor MCP). Slash commands: `/code-review-graph:build-graph` · `/code-review-graph:review-delta` · `/code-review-graph:review-pr`. **Zolvex**: 246 archivos, 1452 nodos, 6997 edges (TS + JS + Java + Bash), branch MainV2.
- **sentry** → Monitoreo de errores en producción: issues activos, stack traces, eventos, performance, alertas — org `mauricio-org` en `https://mauricio-org.sentry.io` — instalado globalmente vía `@sentry/mcp-server`
- **stitch** → Google Stitch: pipelines ETL cloud, ingesta de datos, sincronización hacia BigQuery/warehouses
- **claude-flow** → 170+ herramientas swarm, SONA neural router (89% accuracy), AgentDB vector DB, 5 consensus protocols, 75% reducción de costos API
- **nanobanana** → Gemini API vía MCP: visión, texto, multimodal, embeddings — usa `GEMINI_API_KEY` de Google AI Studio
- **magic** (21st.dev) → generación de componentes UI React con múltiples variantes de estilo — usa `TWENTY_FIRST_API_KEY` de 21st.dev

### Remotion — Video Programático
- **Proyecto**: `C:/Users/Andrea/remotion-demo/` — configurado con Docker headless
- **Skill**: `remotion-best-practices` instalada en `~/.agents/skills/remotion-best-practices`
- **Render**: `npm run docker:render` → MP4 en `./out/` | `npm run docker:studio` → preview en localhost:3000
- **Base image**: `node:20-bookworm-slim` + Chromium headless — sin dependencias locales

### Herramientas Globales Instaladas
- **GSD v1.29.0** (`get-shit-done-cc`) → meta-prompting y spec-driven development. 50+ comandos `/gsd:*`. Flujo: `new-project → plan-phase → execute-phase → ship`. Instala en `~/.claude/commands/gsd/`.
- **SuperPowers** → instalado globalmente para automatización de proyectos existentes. Usar con `npx superpowers@latest init` en proyectos existentes.
- **security-guard.cjs** (`~/.claude/helpers/security-guard.cjs`) → hook PreToolUse que bloquea operaciones destructivas. Activo en TODOS los proyectos globalmente. Bloquea: rm -rf en sistema, DROP en producción, force push a main, curl|bash, sobreescritura de .env.

### Obsidian Vault — Memoria Persistente Cross-Sesión
- **Vault**: `C:/Users/Andrea/Documents/MiVault/`
- **STATE.md**: estado actual de proyectos — leer al inicio de cada sesión
- **CLAUDE.md**: convenciones del vault — leer antes de escribir notas
- **Estructura**: `/conceptos` (notas atómicas con wikilinks), `/proyectos` (STATE.md por proyecto), `/diario` (resúmenes diarios)
- **Cuándo actualizar manualmente**: al terminar trabajo significativo → actualizar `STATE.md` + crear `diario/[fecha].md`

#### Hook SessionStart — Contexto Automático del Vault (GLOBAL)
Configurado en `~/.claude/settings.json` como hook de tipo `command` en el evento `SessionStart`. Script: `~/.claude/helpers/obsidian-context.cjs`.

**Qué hace**: Al iniciar cualquier sesión de Claude Code (desde cualquier directorio):
1. Lee `C:/Users/Andrea/Documents/MiVault/STATE.md`
2. Lee `C:/Users/Andrea/Documents/MiVault/CLAUDE.md`
3. Los inyecta como `systemMessage` en el contexto de Claude antes del primer mensaje

**Resultado**: Claude ya sabe el estado de los proyectos activos y las convenciones del vault desde el primer mensaje — sin que el usuario tenga que pedirlo.

---

#### Sincronización Live — `/obsidian-sync` (OPCIONAL, por sesión)

Comando disponible como `/obsidian-sync`. Inicia `~/.claude/helpers/obsidian-live-sync.cjs` como proceso background que monitorea el transcript JSONL en **tiempo real**.

**Cómo usarlo**: escribir `/obsidian-sync` una sola vez al inicio de la sesión.

**Qué hace en tiempo real**:
- Detecta mensajes nuevos del usuario via `fs.watch` (fallback: polling cada 3s)
- Escribe cada mensaje en `/diario/[fecha].md` bajo `## Actividad en tiempo real`
- Crea notas atómicas en `/conceptos/[término].md` cuando detecta términos técnicos
- Evita duplicados con `.obsidian-sync-state.json`
- Funciona complementario con el hook SessionEnd (que hace el resumen final)

**Cuándo usarlo**: cuando quieras ver el vault actualizándose mientras trabajas, sin esperar al cierre de sesión.

---

#### Hook SessionEnd — Guardado Automático en Obsidian (GLOBAL)
Configurado en `~/.claude/settings.json` como hook de tipo `command` en el evento `SessionEnd`. Script: `~/.claude/helpers/obsidian-session-end.cjs`. Se activa automáticamente al cerrar Claude Code **desde cualquier directorio**.

**Qué hace el script**:
1. Lee el transcript JSONL de la sesión (ruta recibida del hook, o busca el más reciente en `~/.claude/projects/`)
2. Extrae los mensajes reales del usuario (filtra `isMeta`, comandos `/`, `!`, y tags `<`)
3. Guarda en `/diario/[YYYY-MM-DD].md` con los temas reales de la conversación
4. Actualiza `STATE.md` con la fecha/hora de última actualización

**Cómo cerrar para que funcione**: usar `/exit` en el chat — Claude Code cierra limpiamente y el script termina en ~5 segundos. Cerrar con la X del sistema puede interrumpirlo.

**Scope global**: funciona al cerrar Claude Code desde VS Code, terminal, o cualquier proyecto — no solo desde el vault.

### Skills Claude-Flow v3 (29 skills nuevas)
**Swarm**: `swarm-orchestration` `swarm-advanced` `stream-chain`
**SPARC**: `sparc:sparc` `sparc:orchestrator` `sparc:code` `sparc:architect` `sparc:tdd` `sparc:debug` `sparc:devops` `sparc:security-review` `sparc:docs-writer` `sparc:integration` `sparc:spec-pseudocode` `sparc:supabase-admin`
**AgentDB**: `agentdb-vector-search` `agentdb-memory-patterns` `agentdb-learning` `agentdb-optimization` `agentdb-advanced`
**GitHub+Swarm**: `github-code-review` `github-multi-repo` `github-project-management` `github-release-management` `github-workflow-automation`
**ReasoningBank**: `reasoningbank-intelligence` `reasoningbank-agentdb`
**Otros**: `hooks-automation` `pair-programming` `verification-quality` `skill-builder`

### Comandos Claude-Flow (60+ comandos)
`claude-flow-swarm` `claude-flow-memory` `analysis:*` `automation:*` `github:*` `hooks:*` `monitoring:*` `optimization:*` `sparc:*`

### Agentes VoltAgent (131)
Categorías: core-development, language-specialists, infrastructure, quality-security, data-ai, developer-experience, specialized-domains, business-product, meta-orchestration, research-analysis

### Agentes wshobson (105)
Categorías: accessibility, agent-orchestration, agent-teams, api-scaffolding, backend, blockchain, business, c4-architecture, cicd, cloud, code-documentation, code-refactoring, comprehensive-review, conductor, data, database, debugging, deployment, devex, documentation, dotnet, error-handling, framework-migration, frontend, full-stack, functional-programming, game-dev, git, hr-legal, incident-response, javascript-typescript, julia, jvm, kubernetes, llm, ml-ops, multi-platform, observability, payment, performance, python, quantitative-trading, reverse-engineering, ruby, security, seo, shell, startup, systems-programming, tdd, web-scripting

### Skills SuperClaude (31 comandos)
`/sc:build` `/sc:implement` `/sc:design` `/sc:test` `/sc:troubleshoot` `/sc:analyze` `/sc:improve` `/sc:cleanup` `/sc:document` `/sc:explain` `/sc:research` `/sc:git` `/sc:workflow` `/sc:spawn` `/sc:task` `/sc:pm` `/sc:brainstorm` `/sc:reflect` `/sc:recommend` `/sc:estimate` `/sc:spec-panel` `/sc:business-panel` `/sc:index` `/sc:index-repo` `/sc:load` `/sc:save` `/sc:select-tool` `/sc:agent` `/sc:sc` `/sc:help`

### Skill UI/UX
`/ui-ux-pro-max` → diseño con 50+ estilos, 161 paletas, 57 font pairings, 10 stacks

### Herramientas de productividad
- `ccusage` → análisis de tokens usados
- `qmd` → indexador local (92% menos tokens en búsquedas)
- `read-once hook` → bloquea re-lecturas innecesarias
- `agentshield scan` → auditoría de seguridad de configuración

---

## Boundaries

**Hará:**
- Desplegar automáticamente squads de 10 agentes por categoría en paralelo
- Organizar agentes en sub-bloques Core / Quality Gate / Delivery concurrentes
- Entregar resultados de grado producción en una sola pasada (código + tests + docs + seguridad + optimización)
- Escalar a modo SWARM (70+ agentes) para tareas multi-categoría masivas
- Aplicar arbitraje de conflictos (Seguridad > Performance > QA > Líder)
- Segmentar contexto por dimensión para reducir tokens 50-65%

**No hará:**
- Usar herramientas innecesarias que inflen el uso de tokens
- Releer archivos ya leídos en la sesión
- Ignorar herramientas especializadas usando soluciones genéricas
- Proceder sin suficiente contexto en tareas críticas (preguntará antes)
- Mantener agentes ociosos (early termination + reasignación)

---

## Modo Autónomo — Para cualquier tipo de tarea

`/orquesta` usa las herramientas de autonomía instaladas (`automation:auto-agent`, `automation:smart-agents`, `automation:smart-spawn`) para ejecutar tareas con mínima intervención del usuario. Esto aplica a **cualquier dominio** — no solo desarrollo.

### Protocolo de Autonomía Máxima

Cuando el usuario quiere que Claude trabaje de forma autónoma (frases como "hazlo tú", "trabaja solo", "ejecuta todo", "sin preguntarme"), aplicar este protocolo:

```
PASO 1 — ANALIZAR la tarea automáticamente:
  → automation:auto-agent --task "[descripción]" --strategy optimal
  → Detecta agentes necesarios, topología, dependencias

PASO 2 — INICIALIZAR swarm sin esperar confirmación:
  → swarm-orchestration: iniciar con topología jerárquica
  → Spawn ALL agents en UN mensaje (run_in_background: true)
  → NO preguntar "¿procedo?" — ejecutar directamente

PASO 3 — EJECUTAR en paralelo (10 agentes por categoría):
  → Cada agente trabaja en su dimensión concurrentemente
  → smart-agents monitorea y escala si se necesitan más
  → automation:session-memory guarda progreso entre pasos

PASO 4 — REPORTAR al terminar:
  → Mostrar resumen de qué se hizo y con qué herramientas
  → NO interrumpir durante la ejecución para pedir confirmaciones
```

### Qué puede hacer autónomamente (cualquier dominio)

| Dominio | Ejemplo | Agentes auto-seleccionados |
|---------|---------|--------------------------|
| Investigación | "investiga X y dame un resumen" | research-analyst + search-specialist + technical-writer |
| Negocio | "analiza este mercado" | business-analyst + market-researcher + competitive-analyst |
| Escritura | "escribe un post sobre X" | content-marketer + technical-writer + seo-specialist |
| Datos | "analiza este CSV" | data-analyst + data-scientist + database-optimizer |
| Planificación | "crea un roadmap para X" | project-manager + product-manager + architect-reviewer |
| Seguridad | "audita esta config" | security-auditor + penetration-tester + compliance-auditor |
| UI/UX | "diseña una landing" | ui-designer + frontend-developer + accessibility-tester |
| Redes | "configura esta topología" | → Categoría M (packet-tracer) directamente |

### Límite real de autonomía

Claude necesita **un mensaje inicial** del usuario para activarse. Una vez activo con `/orquesta [tarea]`:
- NO hace más preguntas innecesarias
- NO pide confirmaciones intermedias (salvo riesgo destructivo)
- NO espera entre pasos — los encadena automáticamente
- SÍ pausa solo si hay un [CONFLICTO CRÍTICO] que haría fallar todo

**Autonomía end-to-end garantizada con**: `/gsd:autonomous` (encadena todas las fases de un proyecto sin intervención) o `automation:auto-agent` + `swarm-orchestration` para tareas individuales.

### Herramientas de autonomía disponibles

- `automation:auto-agent` → analiza la tarea y selecciona agentes óptimos automáticamente
- `automation:smart-agents` → auto-spawning basado en tipo de archivo, complejidad y carga
- `automation:smart-spawn` → spawn inteligente con análisis previo de workload
- `automation:session-memory` → persiste contexto entre pasos para no repetir trabajo
- `automation:self-healing` → detecta fallos y se recupera sin intervención
- `swarm-orchestration` → orquesta el swarm completo de forma autónoma
- `gsd:autonomous` → ejecuta TODAS las fases de un proyecto encadenadas

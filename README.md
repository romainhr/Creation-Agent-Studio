# Agent Creation Studio

**Laboratorio de creación de agentes** para VS Code / GitHub Copilot.

Este workspace permite diseñar, planificar, generar y validar proyectos de agentes
personalizados a través de un escuadrón de **8 agentes especializados** que trabajan
en cadena: desde el análisis de requisitos hasta la revisión final del proyecto,
reforzados con investigación documental vinculante (DeepSearch) y memoria de
proyecto trazable (MemDoc).

---

## Estructura del workspace

```
Agent Creation Studio/
│
├── .github/
│   ├── agents/                 # 8 agentes del escuadrón principal
│   │   ├── orquestador.agent.md
│   │   ├── analista.agent.md
│   │   ├── arquitecto.agent.md
│   │   ├── planificador.agent.md
│   │   ├── constructor.agent.md
│   │   ├── revisor.agent.md
│   │   ├── deepsearch.agent.md     # Investigador documental (fuentes oficiales)
│   │   └── memdoc.agent.md         # Gestor de memoria de proyecto
│   │
│   ├── instructions/           # Instrucciones file-based (se activan por tipo de archivo)
│   │   ├── agent-files.instructions.md       # Reglas para *.agent.md
│   │   ├── skill-files.instructions.md       # Reglas para SKILL.md
│   │   ├── prompt-files.instructions.md      # Reglas para *.prompt.md
│   │   ├── instruction-files.instructions.md # Reglas para *.instructions.md
│   │   └── output-projects.instructions.md   # Reglas para projects/**
│   │
│   ├── prompts/                # Slash commands reutilizables
│   │   ├── nuevo-proyecto.prompt.md    # /nuevo-proyecto
│   │   ├── plantilla-rapida.prompt.md  # /plantilla-rapida
│   │   └── diagnostico.prompt.md       # /diagnostico
│   │
│   ├── skills/                 # Base de conocimiento
│   │   └── vscode-agent-creation/
│   │       ├── SKILL.md        # Mapa de componentes y reglas de decisión
│   │       └── references/     # 8 documentos de referencia exhaustivos
│   │           ├── agent-format.md
│   │           ├── skill-format.md
│   │           ├── instructions-format.md
│   │           ├── prompt-format.md
│   │           ├── mcp-config.md
│   │           ├── hook-format.md
│   │           ├── subagent-patterns.md
│   │           └── tools-reference.md
│   │
│   ├── memoria/                # Artefactos de memoria base del workspace
│   │   ├── base-backlog-agentes.csv         # Registro de cambios del equipo
│   │   ├── base-decisiones-dependencias.md   # Decisiones y dependencias trazables
│   │   ├── base-todo-list-agentes.md         # Trabajo activo por agente
│   │   └── base-backlog-proyecto.md          # Tareas del proyecto
│   │
│   ├── hooks/                  # Hooks de ciclo de vida
│   │   └── session-context.json  # Inyecta listado de projects/ al inicio de sesión
│   │
│   └── copilot-instructions.md # Instrucciones globales always-on
│
├── .vscode/
│   ├── settings.json           # Configuración del workspace (rutas, hooks, MCP)
│   └── mcp.json                # Servidores MCP: GitHub + Fetch
│
├── templates/                  # Plantillas base con comentarios didácticos
│   ├── single-agent/           # Agente único autónomo
│   ├── coordinator-workers/    # Coordinador + workers especializados
│   ├── pipeline/               # Cadena secuencial de etapas
│   └── reviewer-loop/          # Bucle iterativo implementar → revisar
│
├── scripts/                    # Scripts auxiliares
│   └── session-context.ps1     # PowerShell: genera contexto de sesión
│
├── projects/                   # Output: proyectos generados por el escuadrón
│
└── README.md                   # Este archivo
```

---

## Agentes del escuadrón

El escuadrón funciona como un **pipeline** con gates de aprobación:

```
Orquestador → Analista → Arquitecto → Planificador →[GATE]→ Constructor →[GATE]→ Revisor
```

| Agente | Rol | Herramientas | Invocable |
|--------|-----|-------------|-----------|
| **Orquestador** | Punto de entrada. Resume la petición y coordina el escuadrón | `agent`, `read`, `search`, `fetch` | ✅ Sí |
| **Analista** | Recoge requisitos funcionales y no funcionales del usuario | `read`, `search`, `fetch`, `askQuestions` | ✅ Sí |
| **Arquitecto** | Diseña la topología de agentes, elige patrones, asigna herramientas | `read`, `search`, `fetch` | ✅ Sí |
| **Planificador** | Genera el árbol de archivos detallado con contenido esperado | `read`, `search` | ✅ Sí |
| **Constructor** | Único agente con permisos de escritura. Genera todos los archivos | `read`, `search`, `editFiles`, `createFile`, `runInTerminal`, `problems` | ✅ Sí |
| **Revisor** | Valida integridad, formatos, convenciones y completitud | `read`, `search`, `problems` | ✅ Sí |

### Gates (puntos de control humano)

- **Planificador → Constructor**: El usuario revisa y aprueba el plan antes de la generación de archivos
- **Constructor → Revisor**: El usuario inspecciona el output antes de la validación formal

### Políticas vinculantes

- **Consulta documental**: Toda decisión de requisitos o arquitectura debe estar respaldada por fuentes oficiales. DeepSearch es selectivo para el analista y obligatorio para el arquitecto.
- **Consulta de memoria**: Antes de decidir, analista y arquitecto deben leer el contexto histórico en `.github/memoria/` del proyecto objetivo.
- **Regla de contradicción**: Si DeepSearch contradice una propuesta, el agente debe replantear y justificar su decisión, documentándola como decisión trazable.
- **Registro de memoria**: El constructor invoca a MemDoc después de generar archivos para actualizar los artefactos de memoria del proyecto.

### Memoria de proyecto

Cada proyecto mantiene 4 artefactos de memoria en `projects/{nombre}/.github/memoria/`:

| Artefacto | Archivo | Propósito |
|-----------|---------|----------|
| Registro de decisiones | `decisiones-dependencias.md` | Decisiones arquitectónicas con justificación y fuentes |
| Backlog del proyecto | `backlog-proyecto.md` | Tareas pendientes, en curso y completadas |
| Tareas por agente | `todo-list-agentes.md` | Asignación de trabajo activo por agente |
| Registro del equipo | `backlog-agentes.csv` | Cambios, versiones y contribuciones |

Las plantillas base están en `.github/memoria/` y se clonan al crear cada proyecto nuevo.

---

## Cómo usarlo

### `/nuevo-proyecto` — Flujo completo

Crea un proyecto de agentes desde cero pasando por toda la cadena del escuadrón.

**Ejemplo**:
```
/nuevo-proyecto Quiero un equipo de agentes para revisión de código en Python.
Necesito un agente que analice el código, otro que sugiera mejoras, y uno que
aplique los cambios aprobados.
```

**Flujo**: Orquestador → Analista (preguntas) → Arquitecto (diseño) → Planificador (plan) → [tu aprobación] → Constructor (archivos) → [tu revisión] → Revisor (validación)

### `/plantilla-rapida` — Generación rápida

Genera un proyecto directamente desde un template sin análisis profundo. Ideal para prototipos.

**Ejemplo**:
```
/plantilla-rapida Un pipeline de 3 etapas para procesar datos:
extraer → transformar → cargar
```

**Flujo**: Orquestador → Constructor (adapta template + genera archivos)

### `/diagnostico` — Análisis de proyecto existente

Analiza un proyecto ya generado en `projects/` y produce un informe de calidad.

**Ejemplo**:
```
/diagnostico Analiza el proyecto revision-de-codigo
```

**Flujo**: Orquestador → Revisor (análisis completo + informe)

---

## Templates disponibles

Los templates en `templates/` son plantillas didácticas con comentarios que explican
cada propiedad del frontmatter. Se usan como base para el prompt `/plantilla-rapida`.

| Template | Patrón | Cuándo usarlo | Archivos |
|----------|--------|---------------|----------|
| **single-agent/** | Agente único | Tareas autocontenidas sin necesidad de especialización | 1 agente + instrucciones |
| **coordinator-workers/** | Coordinator-Workers | Tareas complejas que requieren múltiples especialidades | 1 coordinador + 2 workers |
| **pipeline/** | Pipeline secuencial | Procesos con etapas fijas donde cada fase necesita el output de la anterior | 3 etapas encadenadas |
| **reviewer-loop/** | Reviewer Loop | Flujos donde la calidad es crítica y se necesitan iteraciones de refinamiento | 1 implementador + 1 revisor |

---

## Skill de referencia

El skill **vscode-agent-creation** (`.github/skills/vscode-agent-creation/SKILL.md`) es el
cerebro del sistema. Contiene el mapa de componentes y enlaces a 8 documentos de referencia:

| Referencia | Contenido |
|------------|-----------|
| `agent-format.md` | Formato completo de archivos `.agent.md` |
| `skill-format.md` | Formato de archivos `SKILL.md` |
| `instructions-format.md` | Instrucciones always-on y file-based |
| `prompt-format.md` | Formato de archivos `.prompt.md` |
| `mcp-config.md` | Configuración de servidores MCP |
| `hook-format.md` | Hooks de ciclo de vida |
| `subagent-patterns.md` | Patrones de orquestación multi-agente |
| `tools-reference.md` | Catálogo completo de herramientas built-in |

---

## Convenciones

| Aspecto | Convención |
|---------|-----------|
| **Idioma de interacción** | Español |
| **Idioma de código/YAML** | Inglés |
| **Naming** | kebab-case para nombres de proyecto, archivos y agentes |
| **Output de proyectos** | Siempre en `projects/{nombre-kebab-case}/` |
| **Calidad** | Cero placeholders — todo archivo debe ser completo y funcional |
| **Frontmatter** | `user-invokable` (con **k**, nunca con **c**) |

---

## Servidores MCP

El workspace incluye 2 servidores MCP configurados en `.vscode/mcp.json`:

| Servidor | Tipo | Propósito |
|----------|------|-----------|
| **GitHub** | stdio (`@modelcontextprotocol/server-github`) | Consultar repositorios, issues y pull requests de GitHub |
| **Fetch** | stdio (`@anthropic/mcp-fetch`) | Obtener contenido de URLs (documentación, APIs, páginas web) |

### Configuración del token de GitHub

Al usar el servidor GitHub MCP por primera vez, VS Code pedirá tu **GitHub Personal Access Token**.
El token se almacena de forma segura en el secret storage de VS Code (nunca en texto plano).

Para generar un token: GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens.

---

## Requisitos

- **VS Code** 1.99+ (Stable o Insiders)
- **GitHub Copilot** extensión activada con suscripción activa
- **Node.js** 18+ (para servidores MCP via npx)
- **PowerShell** 5.1+ (para el hook de sesión en Windows)

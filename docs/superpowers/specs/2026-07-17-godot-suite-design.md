# godot-suite — Diseño

**Fecha:** 2026-07-17
**Estado:** Aprobado para planificación
**Autor:** freyrecorp / Claude Code

## 1. Resumen

`godot-suite` es un **meta-plugin curado de Claude Code** para desarrollo profesional de
juegos indie en **Godot 4.x**. No reescribe skills existentes: las **orquesta**, las
complementa con los huecos reales del ecosistema, y automatiza el bootstrap del entorno
(skills + MCP server + addons de Godot con versiones pinneadas).

Principio rector: **componer, no reinventar.** El usuario razona en términos de "roles"
(25 roles tipo Godot Architect, Physics Specialist, Steam Publishing…), y la suite enruta
cada rol a la mejor skill/agente existente o a una skill propia.

## 2. Objetivos y no-objetivos

### Objetivos
- Distribuir como **plugin de Claude Code instalable** vía marketplace (`claude plugins install`).
- **Componer** con GodotPrompter (54 skills) y un MCP server, sin vendorizar su código.
- Aportar **4 skills nuevas** para los huecos reales: Steam Publishing, Game Design/GDD,
  Asset Pipeline con IA, Console Porting.
- Automatizar el **setup del entorno** de forma idempotente y reproducible.
- Ofrecer un **mapa de 25 roles → skill/agente** para que el usuario delegue por rol.

### No-objetivos
- No es un generador de juegos "de un prompt" (black-box).
- No reescribe ni copia las skills de GodotPrompter/Randroids/fenixnix (composición, no fork).
- No da soporte a Godot 3.x (APIs distintas; se avisa explícitamente si se detecta 3.x).
- No incluye credenciales, API keys ni SDKs propietarios de consola en el repo.

## 3. Contexto del ecosistema (decisiones basadas en investigación)

| Pilar | Elegido | Razón |
|---|---|---|
| Skills base | [GodotPrompter](https://github.com/jame581/GodotPrompter) (jame581) | 54 skills Godot 4.x + agentes; construido sobre Superpowers (ya instalado). Cubre ~21 de los 25 roles. |
| MCP server (default) | [Coding-Solo/godot-mcp](https://github.com/Coding-Solo/godot-mcp) | El más battle-tested; install simple vía `npx`; estable entre versiones. |
| MCP server (upgrade) | [mkdevkit/godot-mcp](https://github.com/mkdevkit/godot-mcp) | Documentado como opción avanzada: bridge de runtime, screenshots, input sim. |
| Skills complementarias | [Randroids-Dojo/Godot-Claude-Skills](https://github.com/Randroids-Dojo/Godot-Claude-Skills), [fenixnix/Godot-Skills](https://github.com/fenixnix/Godot-Skills) | Referencia opcional (test/deploy, gramática GDScript). |

## 4. Arquitectura del repo

```
godot-suite/
├─ .claude-plugin/
│  └─ marketplace.json              # manifest del marketplace (lista el plugin)
├─ plugins/
│  └─ godot-suite/
│     ├─ .claude-plugin/
│     │  └─ plugin.json             # manifest: nombre, versión, autor, mcpServers
│     ├─ .mcp.json                  # Coding-Solo/godot-mcp por defecto
│     ├─ skills/
│     │  ├─ godot-suite-setup/SKILL.md
│     │  ├─ godot-suite-orchestrator/SKILL.md
│     │  ├─ steam-publishing/SKILL.md
│     │  ├─ game-design-gdd/SKILL.md
│     │  ├─ asset-pipeline-ai/SKILL.md
│     │  └─ console-porting/SKILL.md
│     ├─ agents/
│     │  ├─ godot-producer.md
│     │  └─ godot-release-manager.md
│     └─ commands/
│        └─ godot-new-game.md
├─ docs/
│  ├─ addons.md                     # tabla de addons con versión pinneada
│  ├─ mcp-upgrade.md                # cómo migrar a mkdevkit
│  ├─ roles.md                      # mapa de los 25 roles → skill/agente
│  └─ superpowers/specs/            # este spec
└─ README.md
```

## 5. Componentes

### 5.1 `godot-suite-setup` (skill de bootstrap — pieza "curada")
Skill **idempotente** que prepara el entorno. Al invocarse:

1. **Detecta Godot**: ejecuta `godot --version`; si no es 4.x, aborta con mensaje claro.
   Si Godot no está en PATH, guía al usuario a instalarlo (no lo instala silenciosamente).
2. **Instala GodotPrompter** por composición: añade su marketplace
   (`claude plugins marketplace add jame581/skillsmith`) e instala el plugin
   (`claude plugins install godot-prompter@skillsmith`). Verifica el resultado; si ya está,
   no-op.
3. **Configura el MCP**: confirma que el `.mcp.json` del plugin apunta a Coding-Solo y que
   `npx` está disponible. Instruye al usuario a verificar `/mcp` (punto verde, tools > 0).
4. **Instala addons recomendados** en `res://addons/` del proyecto activo, con versiones
   pinneadas de `docs/addons.md`. Cada addon se instala solo si el usuario lo confirma
   (los addons son elección de proyecto). Registra versión + fecha en un `addons.lock.md`
   dentro del proyecto.
5. **Reporta** un resumen: qué quedó instalado, versiones, y qué falta.

Reglas: idempotente, nunca sobrescribe addons existentes sin avisar, nunca commitea secrets,
verifica cada paso antes de reportar éxito (evidencia antes de afirmar).

### 5.2 `godot-suite-orchestrator` + subagente `godot-producer`
El **mapa de 25 roles** vive aquí como tabla de enrutamiento (fuente única: `docs/roles.md`).
Cuando el usuario pide trabajo por rol, el orchestrator/productor:

1. Identifica el rol.
2. Enruta a la skill/agente correcto (GodotPrompter, superpowers, o skill propia).
3. Para trabajo multi-rol, el subagente `godot-producer` planea y delega en secuencia.

Mapa completo de roles en `docs/roles.md`. Ejemplos:
- Physics Specialist → skill de física de GodotPrompter.
- Bug Hunter → `superpowers:systematic-debugging` + `godot-debugging`.
- Steam Publishing → skill propia `steam-publishing`.
- Game Designer → skill propia `game-design-gdd`.

### 5.3 Skills nuevas (los 4 huecos)

**`steam-publishing`** — Integración GodotSteam: inicialización del SDK, logros
(achievements), estadísticas, cloud saves, Steam Workshop, rich presence, y subida de builds
con `steamcmd`. Cubre setup de la página de la store y depots a nivel de checklist/proceso.
**No** incluye el SDK de Steamworks ni App IDs (los aporta el usuario); documenta cómo
obtenerlos.

**`game-design-gdd`** — Trabajo de diseño (no código): generación y mantenimiento de un
Game Design Document, definición de core loops, economía, curvas de progresión, balanceo,
y sistemas de meta-progresión. Produce/actualiza un `docs/GDD.md` en el proyecto del juego.

**`asset-pipeline-ai`** — Pipeline de assets con IA: generación de sprites/texturas vía el
MCP `image-gen` disponible, automatización de import Aseprite→Godot, generación de atlas,
convenciones de nombres y carpetas, y ajustes de compresión/import. Documenta límites de
licencia de assets generados por IA.

**`console-porting`** — Guía de porting a consola (Switch/PS/Xbox) vía W4 Games: requisitos
de certificación, diferencias de input y guidelines por plataforma, gestión de builds.
Deja claro que requiere licencias de fabricante y NDAs; **no** incluye SDKs propietarios ni
información bajo NDA.

### 5.4 Subagentes
- **`godot-producer.md`** — planifica trabajo por rol y delega a skills/agentes.
- **`godot-release-manager.md`** — orquesta el flujo de release: export (GodotPrompter) →
  steam-publishing → console-porting, en el orden correcto.

### 5.5 Command
- **`godot-new-game.md`** — slash command que hace scaffolding de un proyecto Godot nuevo con
  la estructura de carpetas recomendada e invoca `godot-suite-setup`.

### 5.6 MCP
- `.mcp.json` declara el servidor Coding-Solo (`npx @coding-solo/godot-mcp`).
- `docs/mcp-upgrade.md` explica cómo cambiar a mkdevkit para capacidades de runtime.

## 6. Formato de skills (estándar de la suite)
Cada skill sigue el formato Claude Code:
- `SKILL.md` con frontmatter YAML (`name`, `description` con triggers claros).
- Patrón **core + references**: el `SKILL.md` bajo ~16 KB; detalle pesado en archivos de
  referencia cargados on-demand (mismo patrón que GodotPrompter para caber en el context
  window).
- Frontmatter `description` con disparadores explícitos para que el orchestrator la encuentre.

## 7. Manejo de errores y casos borde
- **Godot ausente / versión 3.x**: setup aborta con mensaje accionable, no continúa.
- **`npx`/Node ausente**: setup reporta y da instrucciones (Node vía nvm ya presente en la
  máquina del usuario).
- **GodotPrompter ya instalado**: no-op, reporta versión.
- **Addon ya presente en el proyecto**: no sobrescribe; pregunta antes de actualizar.
- **Rol ambiguo o no mapeado**: el orchestrator pide aclaración en vez de adivinar.
- **Sin proyecto Godot activo**: las skills que requieren `res://` avisan y no actúan.
- **Assets IA / licencias / consola bajo NDA**: las skills documentan límites legales y no
  incluyen material restringido.

## 8. Seguridad
- **Sin secrets en el repo**: ni App IDs de Steam, ni keys de APIs de generación, ni SDKs
  de consola. Las skills instruyen al usuario a aportarlos vía entorno/config local.
- **MCP local-only**: el servidor MCP opera contra el proyecto local; no expone red pública.
- **Operaciones de archivo acotadas** al proyecto activo; setup nunca escribe fuera del
  proyecto o de `~/.claude` sin avisar.
- **Validación en frontera**: setup valida versión de Godot y disponibilidad de herramientas
  antes de ejecutar acciones destructivas o de instalación.

## 9. Testing / verificación
- **Setup**: probar en (a) máquina sin Godot, (b) con Godot 3.x, (c) con Godot 4.x limpio,
  (d) con GodotPrompter ya instalado. Verificar idempotencia (correr dos veces = mismo estado).
- **Skills**: cada skill nueva se valida con `superpowers:writing-skills` (frontmatter,
  triggers, tamaño < 16 KB, references cargables).
- **MCP**: verificar `/mcp` muestra `godot` con tools > 0 tras el setup.
- **Orchestrator**: probar que cada uno de los 25 roles enruta al destino esperado
  (tabla de `docs/roles.md` como oráculo).
- **Manifests**: validar `marketplace.json` y `plugin.json` contra el esquema de plugins de
  Claude Code (instalación real en una sesión de prueba).

## 10. Riesgos y mitigaciones
- **Deriva de GodotPrompter** (cambian su marketplace/nombres): mitigado por composición
  (no vendorizamos) + verificación en setup; si su install falla, el setup lo reporta.
- **Versiones de addons rompen entre releases de Godot**: mitigado con pinning + `addons.lock.md`.
- **MCP inestable en alguna plataforma**: mitigado con Coding-Solo como default estable + ruta
  de upgrade documentada.
- **Alcance de skills nuevas demasiado grande** (Steam/Console son profundos): mitigado con
  patrón core+references y checklists accionables en vez de exhaustividad.

## 11. Orden de implementación sugerido
1. Esqueleto del plugin (`marketplace.json`, `plugin.json`, `.mcp.json`, README).
2. `docs/roles.md` + `docs/addons.md` (fuentes de verdad).
3. `godot-suite-setup` (bootstrap).
4. `godot-suite-orchestrator` + `godot-producer`.
5. Las 4 skills nuevas (steam, gdd, asset-ai, console).
6. `godot-release-manager` + command `godot-new-game`.
7. `docs/mcp-upgrade.md` + verificación end-to-end.

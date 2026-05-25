# Agentes de IA Inmobiliarios

> **6 agentes de IA auto-instalables para profesionales inmobiliarios** en LATAM y España.
> Reporte de búsqueda con anti-salto · Análisis legal de contratos · Valuación ACM · Procesamiento de fotos con IA · Meta-agente que construye más agentes.
> Open source — uso individual gratuito. Hecho por [Valia Academy](https://github.com/valia-academy).

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

---

## 🚀 Instalación en 1 minuto

> **¿Por qué Claude Desktop?**
> Los agentes se ejecutan en **Claude Desktop**, la aplicación oficial de Anthropic (Mac/Windows). Las versiones recientes (v2025+) incluyen **Claude Code integrado** — el motor que permite que los agentes lean archivos, naveguen tu Chrome con sesión iniciada y generen Word/Excel reales. No necesitas saber programar ni abrir terminal: todo pasa dentro de la conversación.
> Si todavía no lo tienes: [claude.com/download](https://claude.com/download).

Si tienes **Claude Pro + Claude Desktop + Chrome con extensión Claude in Chrome**, copia esta frase exacta y pégala en Claude Desktop:

```
Claude, instala los agentes IA inmobiliarios del repo público
https://github.com/valia-academy/agentes-ia-inmobiliarios.
Los archivos .md están en /commands/. Cópialos a mi
~/.claude/commands/ para que aparezcan como slash commands.

IMPORTANTE: NO uses git clone — no tengo git instalado y no quiero
instalarlo. Descarga los archivos directamente vía HTTP usando las URLs
raw de GitHub (raw.githubusercontent.com). En Windows usa
Invoke-WebRequest; en Mac usa curl. Ambos vienen preinstalados.
```

Claude hace el resto:
1. Descarga los 6 archivos `.md` del repo vía HTTP directo (sin git).
2. Los guarda en `~/.claude/commands/`.
3. Te confirma que están listos.

> **Si te pide instalar git**: respondé *"No quiero instalar git. Descarga los archivos directamente con Invoke-WebRequest (Windows) o curl (Mac)."* — Claude obedece y usa HTTP nativo.

Después escribe `/` en cualquier conversación de Claude Desktop y vas a ver los agentes disponibles en autocompletado.

### Actualizar a la última versión

Cuando publiquemos cambios al repo:

```
Claude, actualízame los agentes IA inmobiliarios desde
https://github.com/valia-academy/agentes-ia-inmobiliarios.
NO uses git — descarga los archivos directamente vía HTTP
(raw.githubusercontent.com) con Invoke-WebRequest o curl.
```

Claude re-descarga los `.md`. Tus configs locales (datos profesional, learnings) **se conservan** — están en otra carpeta (`~/inmobiliaria/agents-config/`).

### Instalación manual (alternativa sin Claude)

Las opciones que siguen NO requieren git y usan herramientas que ya vienen preinstaladas en tu sistema.

**Mac / Linux (con `curl`, viene preinstalado):**

```bash
mkdir -p ~/.claude/commands
BASE="https://raw.githubusercontent.com/valia-academy/agentes-ia-inmobiliarios/main/commands"
for f in agente-hola-inmobiliaria.md agente-reporte-de-busqueda.md \
         agente-legal-inmobiliario.md agente-valuacion.md \
         agente-fotos-inmuebles.md agente-constructor-de-agentes.md; do
  curl -fsSL "$BASE/$f" -o ~/.claude/commands/"$f"
done
```

**Windows PowerShell (con `Invoke-WebRequest`, viene preinstalado):**

```powershell
$dest = "$HOME\.claude\commands"
New-Item -ItemType Directory -Force -Path $dest | Out-Null
$base = "https://raw.githubusercontent.com/valia-academy/agentes-ia-inmobiliarios/main/commands"
$files = @("agente-hola-inmobiliaria.md", "agente-reporte-de-busqueda.md",
           "agente-legal-inmobiliario.md", "agente-valuacion.md",
           "agente-fotos-inmuebles.md", "agente-constructor-de-agentes.md")
foreach ($f in $files) {
  Invoke-WebRequest -Uri "$base/$f" -OutFile "$dest\$f" -UseBasicParsing
}
```

**Si tenés `git` instalado** (solo si ya lo usás para otras cosas):

```bash
git clone https://github.com/valia-academy/agentes-ia-inmobiliarios.git
cp agentes-ia-inmobiliarios/commands/*.md ~/.claude/commands/
```

---

## 🧑‍💼 Audiencia objetivo

Brokers individuales, agentes en inmobiliaria, dueños de inmobiliarias medianas, y consultores que quieran ofrecer estos servicios a sus clientes. Pensados para:

- 🇵🇪 Perú · 🇦🇷 Argentina · 🇲🇽 México · 🇨🇴 Colombia · 🇨🇱 Chile · 🇺🇾 Uruguay · 🇪🇨 Ecuador · 🇵🇦 Panamá · 🇧🇷 Brasil · 🇪🇸 España

---

## 🤖 Los 6 agentes

| Comando | Qué hace | Etapa del funnel |
|---------|----------|------------------|
| `/agente-hola-inmobiliaria` | Smoke-test simple — saludo + frase motivacional. Sirve para verificar que la instalación funciona. | — |
| `/agente-reporte-de-busqueda` | De unas líneas con criterios del cliente comprador, genera Word brandeado con la selección de inmuebles que matchean + Excel checklist anti-salto. Navega portales del país con tu sesión de Chrome. | Pre-venta / captación |
| `/agente-legal-inmobiliario` | Redacta, analiza y consulta sobre contratos inmobiliarios en tu jurisdicción. 3 modos auto-detectados (redactor / analista / consultor). Multimodal (PDF, Word, MD, TXT, imágenes). Auto-genera knowledge legal país-específico. | Cierre |
| `/agente-valuacion` | De ficha del inmueble + URLs de comparables, genera Word brandeado con valuación (ACM) + Excel ajustable con fórmulas reales. | Valuación |
| `/agente-fotos-inmuebles` | Procesa carpeta caótica de fotos: selecciona finalistas, renombra según estándar inmobiliario, mejora con GPT Image 2 (ChatGPT Plus/Pro). | Operación / listing |
| `/agente-constructor-de-agentes` | **Meta-agente.** De una idea (vaga o concreta) sobre qué automatizar, te conduce conversacionalmente por el diseño y te deja un nuevo agente instalado y listo. | Multiplicador transversal |

---

## ⚙️ Pre-requisitos

| Pre-requisito | Por qué | Costo | Link |
|---------------|---------|-------|------|
| **Cuenta Claude Pro** | Motor de los agentes | ~USD 20/mes | [claude.ai/upgrade](https://claude.ai/upgrade) |
| **Claude Desktop** (v2025+) | Donde se ejecutan los slash commands. Trae Claude Code integrado de fábrica. | Incluido en Pro | [claude.com/download](https://claude.com/download) |
| **Chrome + extensión Claude in Chrome** | Para que los agentes naveguen portales y servicios web con tu sesión | Incluido en Pro | [Chrome Web Store](https://chromewebstore.google.com/detail/claude/fcoeoabgfenejglbffodgkkbkcdhcgfn) |
| **Python 3.10+** | 5 de los 6 agentes lo usan para generar Word, Excel y PDF reales (`python-docx`, `openpyxl`, `pdfplumber`, `Pillow`). En Mac suele venir preinstalado. En Windows hay que instalarlo desde Microsoft Store (1 clic, gratis) o [python.org](https://www.python.org/downloads/). | Gratis | [Cómo verificar abajo](#-verificar-python) |

**Pre-requisitos opcionales** según uso:

| Para qué agente | Cuenta requerida | Modo sin la cuenta |
|-----------------|------------------|--------------------|
| `/agente-fotos-inmuebles` con mejora IA | ChatGPT Plus o Pro (GPT Image 2) | Fallback automático a Pillow local (correcciones básicas) |

### 🐍 Verificar Python

**Mac (Terminal):**
```bash
python3 --version
```
Si responde con `Python 3.10` o superior, ya lo tienes. Si dice *"command not found"* → instalá con `brew install python` o desde [python.org](https://www.python.org/downloads/).

**Windows (PowerShell):**
```powershell
python --version
```
Si responde con `Python 3.10` o superior, ya lo tienes. Si no:
1. Abrí **Microsoft Store**.
2. Buscá *"Python 3.12"* (o la versión más reciente disponible).
3. Click **"Obtener"** — gratis, 1 minuto, configura el PATH solo.
4. Reabrí PowerShell y volvé a verificar.

> **Si te aparece error de Python al invocar un agente**: respondéle a Claude *"Verifica que tengo Python instalado con `python --version` o `python3 --version`. Si no lo tengo, dame instrucciones de cómo instalarlo en mi sistema."* — Claude detecta el OS y te guía.

---

## 💡 ¿Cómo se usan?

Una vez instalados, en cualquier conversación de Claude Desktop escribes el comando seguido del pedido en lenguaje natural:

```
/agente-reporte-de-busqueda Cliente busca depa 3 dorm en Miraflores
o San Isidro, USD 250-300k, con cochera. Tiene mascota.
```

```
/agente-legal-inmobiliario Revisa ~/Downloads/contrato.pdf
```

```
/agente-valuacion Av. Larco 850, Miraflores. Depa 120m², 18 años,
3 dorm, USD. Vista al parque. Comparables: [url1], [url2], [url3]
```

```
/agente-fotos-inmuebles Procesa fotos en ~/Downloads/larco-850/
```

```
/agente-constructor-de-agentes Quiero un agente que reciba una carpeta
con fotos del inmueble + un Word con sus características, y genere una
ficha brandeada en Word lista para revisar y editar: texto y presentación
primero, fotos al final.
```

**Primera vez:** cada agente hace un wizard corto (1-5 preguntas atómicas) para configurarse con tus datos. Después va directo a la tarea cada vez.

**Outputs:** se guardan en `~/inmobiliaria/outputs/{tipo}/`. Word, Excel y PDF — nunca markdown como entregable.

---

## 📂 Estructura del repo

```
agentes-ia-inmobiliarios/
├── README.md                    ← estás aquí
├── LICENSE                      ← CC BY-NC 4.0
├── commands/                    ← los 6 agentes (.md de slash commands)
│   ├── agente-hola-inmobiliaria.md
│   ├── agente-reporte-de-busqueda.md
│   ├── agente-legal-inmobiliario.md
│   ├── agente-valuacion.md
│   ├── agente-fotos-inmuebles.md
│   └── agente-constructor-de-agentes.md
├── docs/
│   ├── template-canonico.md     ← anatomía estándar del slash command self-installing
│   └── ideas-sugeridas-agentes.md ← 30+ ideas de agentes que puedes construir con el meta-agente
├── ejemplos/                    ← outputs de muestra (próximamente)
└── plantillas-legales/          ← plantillas de contratos por país (próximamente)
```

---

## 🧠 Cómo están diseñados

Cada agente sigue un **patrón canónico** validado:

1. **Self-installing wizard** — primera invocación hace 1-5 preguntas atómicas, guarda config en `~/inmobiliaria/agents-config/{nombre}.json`.
2. **6 reglas críticas** al inicio del `.md` que prohíben anti-patterns comunes del LLM (agrupar preguntas, frases pomposas, insights educativos espontáneos, especular sobre origen de documentos).
3. **Memoria evolutiva (learnings)** — los agentes aprenden con uso y guardan quirks/preferencias en `{nombre}-learnings.md` para mejorar entre corridas.
4. **Outputs en Word/Excel/PDF** — nunca markdown como entregable al usuario.
5. **Comandos universales:** `reconfigurar` / `ver config` / `ayuda` / `ver learnings` / `olvidar learnings`.

Detalle completo en [`docs/template-canonico.md`](docs/template-canonico.md).

---

## 🛠️ Construye tus propios agentes

El `agente-constructor-de-agentes` puede crearte agentes nuevos a partir de una idea. Ver [`docs/ideas-sugeridas-agentes.md`](docs/ideas-sugeridas-agentes.md) para 30+ ideas categorizadas:

- **Comunicación con clientes** (mensaje de bienvenida, seguimiento post-visita, respuestas a objeciones)
- **Operación interna** (ficha de cliente, planning semanal, checklist pre-visita)
- **Marketing** (post Instagram/Facebook, caption Reels, auditoría de tu listing)
- **Análisis y reportes** (reporte mensual a propietario, comparativos, análisis de barrio)
- **Captación** (investigador de prospecto, análisis crítico de listing ajeno)
- **Cierre y negociación** (carta-oferta formal, análisis de contrapropuesta)

---

## 🏫 ¿Quieres aprender a construir estos agentes desde cero?

Estos agentes son entregables del curso **Agentes IA Inmobiliarios** de [Valia Academy](https://github.com/valia-academy) — una clase de 2.5 horas en vivo donde aprendes a construir agentes IA para inmobiliarias sin saber programar.

Más información: pronto.

---

## 🤝 Comunidad y soporte

- **Issues y bugs:** [GitHub Issues](https://github.com/valia-academy/agentes-ia-inmobiliarios/issues)
- **Comunidad de alumnos:** Skool (link próximamente, exclusivo para alumnos del curso)

---

## 📜 Licencia

**CC BY-NC 4.0** — Uso individual gratuito. Atribución requerida. Uso comercial requiere autorización explícita de Valia Academy.

Ver [LICENSE](LICENSE) para el texto completo.

Para licencia comercial (uso en empresas inmobiliarias, integración en SaaS, redistribución pagada): contacto@valiapro.com

---

## 🔄 Versión

**v0.5** · Diseño curricular cerrado · 6 agentes en versión beta.

Última actualización: 2026-05-22.

---

<p align="center">
<sub>Hecho con ❤️ por <a href="https://github.com/valia-academy">Valia Academy</a> · Lima, Perú</sub>
</p>

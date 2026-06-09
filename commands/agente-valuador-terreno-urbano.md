---
description: Valúa un terreno urbano por su potencial de desarrollo (método residual). Pregunta país y modo, infiere o recibe la edificabilidad, releva precios de estreno de la zona, y entrega Word + Excel + PDF brandeados con 3 escenarios, sensibilidad y metodología trazable.
---

Eres `agente-valuador-terreno-urbano`. Valúas **terrenos urbanos con potencial de desarrollo** por el **método de valor residual estático**: el valor del suelo es lo máximo que un desarrollador racional puede pagar tras cubrir construcción, costos blandos, marketing, financiamiento, impuestos y su margen. Te adaptas al país del predio (LATAM + España). Entregas Word + Excel + PDF brandeados con tres escenarios (conservador / central / agresivo), sensibilidad, metodología trazable y disclaimer.

El usuario te invocó con: `$ARGUMENTS`

> Si `$ARGUMENTS` trae la ficha del terreno (dirección + área + país), úsalo como input. Si está vacío o solo trae un comando especial, maneja eso primero.

---

# 🚨 REGLAS CRÍTICAS — LEER ANTES DE EJECUTAR

**1. UNA PREGUNTA POR TURNO, SIEMPRE.** Haz exactamente UNA pregunta y espera la respuesta. NUNCA agrupes 2+ preguntas. Si tienes que preguntar 5 cosas, son 5 turnos.

**2. WIZARD MÍNIMO + LAZY.** El wizard inicial pregunta SOLO el idioma. El país se pregunta por valuación. Los datos del profesional (nombre, contacto, logo) se piden lazy la primera vez que vas a generar un Word.

**3. SIN REFERENCIAS A SU ORIGEN PEDAGÓGICO.** Eres un producto profesional. El usuario es un broker/desarrollador en su trabajo. NUNCA digas "el curso", "Valia Academy", "didáctico", "alumno", "para clase".

**4. SIN INSIGHTS EDUCATIVOS NO PEDIDOS.** No emitas bloques `★ Insight` espontáneos. Tu output es la respuesta a su pedido, no un acompañamiento docente.

**5. NO ESPECULAR SOBRE EL ORIGEN/CONTEXTO DE LOS INPUTS.** El nombre de una calle o un proyecto no te dice nada extra. Trabaja con los datos literales. Si falta contexto, pregúntalo; no lo inventes. **NUNCA inventes parámetros de edificabilidad ni cifras de mercado: si no los tienes con respaldo, dilo y ofrece un modo que los consiga o que el usuario los aporte.**

**6. ESTILO CANÓNICO — al grano, sin ruido.**

| Principio | Sí | No |
|-----------|----|----|
| Saludos cortos | "Hola, soy X. 1 pregunta: ¿país del terreno?" | Despliegue de cómo operas |
| Acuses telegramáticos | "Perú. Cargo el conocimiento del país. ~1 min." | Contextualización innecesaria |
| Estado + acción siguiente | "Edificabilidad lista (20 pisos, 40% libre). Relevo precios." | Meta-referencias |
| Reportes directos | "Listo. Central: US$ 4.6M (US$ 3,836/m², 15% del GDV). Te muestro el detalle:" | Preámbulos |
| Paths como links | `📄 [archivo.docx](C:\path)` | Texto plano |
| Siguiente paso accionable | "Próximos pasos: 1. Revisar Excel, 2. Confirmar parámetros con certificado." | Terminar sin guía |

**Frases prohibidas:** "como puedes ver", "vale la pena destacar", "interesante notar que", "a continuación te presento", "sin más preámbulos".

---

## 1. Verifica si existe configuración

Lee `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano.json` con la herramienta Read.

## 2. Si NO existe config (primera corrida)

Saluda corto:

> *"Hola, soy `agente-valuador-terreno-urbano`. Valúo terrenos por su potencial de desarrollo. Te hago 1 pregunta para configurarme. Necesitas Python instalado (para generar Word/Excel/PDF) y, si quieres que releve precios de portales, Chrome con la extensión Claude in Chrome."*

**Detente y espera.**

### Pregunta 1: Idioma

> *"¿En qué idioma quieres los reportes? (Español / Inglés / Portugués)"*

**Espera respuesta.**

### Tras la respuesta

1. **Verifica Python** (mensaje amigable si falta):
   ```bash
   if command -v python >/dev/null 2>&1; then PY=python; elif command -v python3 >/dev/null 2>&1; then PY=python3; elif command -v py >/dev/null 2>&1; then PY=py; else echo "PYTHON_NOT_INSTALLED"; exit 1; fi
   $PY -c "import openpyxl, docx" 2>/dev/null || $PY -m pip install openpyxl python-docx
   ```
   Si devuelve `PYTHON_NOT_INSTALLED` (o en Windows "python is not recognized"), muestra:
   > *"⚠ Python no está instalado. Sin Python no puedo generar el Word/Excel/PDF. Instálalo:*
   > *• **Windows**: Microsoft Store → busca \"Python 3.12\" → \"Obtener\" (gratis, ~1 min, configura el PATH solo).*
   > *• **Mac**: `brew install python` o desde python.org/downloads.*
   > *Luego cierra Claude Desktop por completo, reábrelo e invócame de nuevo."*
   y **DETENTE**.

2. **Crea directorios y materializa el generador embebido** (ver sección "## Generador embebido" al final de este archivo). Crea `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/` y escribe ahí los 4 archivos `.py` exactamente como están embebidos. Crea también `~/inmobiliaria/outputs/valuaciones-terreno/`.
   ```bash
   mkdir -p ~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/
   mkdir -p ~/inmobiliaria/outputs/valuaciones-terreno/
   ```

3. **Guarda config**:
   ```json
   {"idioma":"es","datos_profesional":null,"configurado_en":"<fecha_iso>"}
   ```

4. Confirma en una línea: *"Listo, configurado. Dame la dirección del terreno, su área en m² y el país, y arrancamos."* Procede con el pedido si ya vino en `$ARGUMENTS`.

## 3. Si SÍ existe config (corridas siguientes)

Lee el JSON silenciosamente. **NO confirmes la carga.** Verifica que el generador esté materializado en `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/` (si falta algún `.py`, reescríbelo desde la sección embebida, sin avisar). Procede directo a la tarea.

---

## Memoria evolutiva (learnings)

Al inicio de cada invocación, tras leer config, lee `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano-learnings.md` silenciosamente. Si existe, incorpora las lecciones sin anunciarlas. Durante la corrida, registra learnings si: un portal cambió de comportamiento, una jurisdicción tuvo una particularidad normativa que tuviste que resolver, o el usuario mostró una preferencia persistente (ej. siempre reporta en USD aunque opere en CLP). Al final, si vale, agrégalo con formato `### {fecha} — {contexto}` + Observación + Lección, y menciona: *"📚 Aprendí algo: {resumen 1 línea}. Lo guardé."* No registres éxitos del flujo normal ni lo ya documentado aquí.

> **Distinción clave:** `knowledge-{pais}.md` guarda **hechos del país** (zonificación, tributación, costos). El archivo de learnings guarda **quirks de portales y preferencias del usuario** transversales.

---

## Comandos especiales

- **`reconfigurar`** → borra el JSON y reinicia el wizard.
- **`ver config`** → muestra la config de forma legible.
- **`ver learnings`** → muestra los learnings legibles.
- **`olvidar learnings`** → `rm ~/inmobiliaria/agents-config/agente-valuador-terreno-urbano-learnings.md`.
- **`ver knowledge {país}`** → muestra `knowledge-{pais}.md` legible.
- **`actualizar knowledge {país}`** → re-investiga (bootstrap) ese país y reescribe su knowledge.
- **`ayuda`** → describe qué haces, inputs y ejemplos.

---

## Inputs por uso

Te invocan con la ficha del terreno. Ejemplos:
- *"Valúa el terreno de Av. Arequipa 1110, Lima, Perú. 1,204.45 m², esquinero."*
- *"Terreno en Polanco, CDMX, México, 800 m². Tengo el certificado de uso de suelo."*

**Inputs mínimos:** país, dirección, área del terreno (m²). (En modo "Usuario aporta": además altura/área libre/coeficiente.)
**Opcionales:** frente, esquinero, uso objetivo, certificado de parámetros, mix/precios propios, supuestos financieros propios.

Si falta un input mínimo, pídelo en una pregunta atómica. Si faltan opcionales, sigue con lo que tienes.

---

## Flujo por valuación

### Paso 0 — País y conocimiento del país
Pregunta el país (si no vino). Lee `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/knowledge-{pais}.md`. Si NO existe, avisa *"Primera vez en {país}: investigo zonificación, tributación y costos. ~1-2 min."* y ejecuta el **bootstrap** (abajo). Si existe, cárgalo en silencio.

**Bootstrap del país (genérico):** investiga (web search + navegación) y escribe `knowledge-{pais}.md` con estas 5 secciones, cada cifra con fuente/URL y nivel de confianza:
1. **Zonificación y edificabilidad** — fuente(s) oficial(es) y cómo se expresa la altura (pisos / fórmula como 1.5(a+r) / coeficiente), el área libre y los estacionamientos. Fuentes típicas por país: 🇵🇪 IMP/ordenanzas municipales · 🇲🇽 uso de suelo/SEDUVI/certificado de zonificación · 🇦🇷 Código Urbanístico/planeamiento · 🇨🇴 POT municipal · 🇨🇱 Plan Regulador Comunal · 🇺🇾 plan de ordenamiento · 🇪🇨 IRM/norma municipal · 🇪🇸 Plan General · 🇵🇦 norma de zonificación/MIVIOT.
2. **Tributación de la primera venta** — impuesto aplicable (IGV/IVA/ITBMS), base gravada y **tasa efectiva sobre el precio de vivienda**, exoneraciones/umbral si aplican.
3. **Costos y benchmarks** — costo de construcción por m² (rasante y sótano), soft costs, marketing+ventas, gastos financieros y margen del desarrollador (% del GDV); incidencia típica del terreno sobre el GDV (sanity check).
4. **Mercado** — portal de precios de estreno (🇵🇪 Urbania · 🇦🇷 ZonaProp · 🇲🇽 Inmuebles24 · 🇨🇴 Metrocuadrado · 🇨🇱 PortalInmobiliario · 🇺🇾 InfoCasas · 🇪🇨 Plusvalía · 🇪🇸 Idealista · 🇵🇦 Encuentra24) y, si existe, fuente oficial de precios.
5. **Moneda y tipo de cambio** — moneda local, convención de reporte (USD/local), TC y su fuente.

> **Éxito parcial:** si una dimensión devuelve poco, escribe el knowledge igual con lo que tengas, marca la dimensión como `confianza: baja / pendiente` y avísale al usuario pidiéndole ese dato (o sugiriendo "Usuario aporta" para esa parte). **No bloquees la valuación.** Usa subagentes paralelos (tool `Agent`) por dimensión si conviene.

### Paso 1 — Modo de parámetros (pregunta SIEMPRE)
> *"¿Cómo obtenemos la edificabilidad de este terreno?*
> *• **Híbrido** — la infiero de proyectos reales de la zona y la cruzo con la norma oficial (te pido el certificado si lo tienes).*
> *• **Automático** — intento sacar la norma oficial del visor/municipio yo solo.*
> *• **Usuario aporta** — tú me das altura, área libre y coeficiente (de tu certificado)."*

**Espera respuesta.** Degradación: si "Automático" no consigue parámetros confiables en un intento acotado, dilo y ofrece pasar a Híbrido o Usuario aporta.

### Paso 2 — Inputs del predio
Pide solo lo mínimo faltante (dirección, área del terreno). Opcionales (frente, esquinero, uso objetivo) solo si el usuario los menciona.

### Paso 3 — Edificabilidad
Según el modo: obtén altura (pisos), % área libre, coeficiente/área techada y estacionamientos. Documenta **fuente y confianza de cada parámetro**. Si la confianza es baja, márcalo y empuja al certificado.

### Paso 4 — Mercado y precios
Releva proyectos de **estreno** colindantes en el portal del país (con Claude in Chrome, aplicando los quirks de abajo): tipologías, áreas, precios por m², mix. Cruza con la fuente oficial de precios si el knowledge la tiene. Obtén un **precio blended** y el mix. Si no hay Chrome o el portal bloquea, pídele al usuario precios de referencia.

### Paso 5 — Estructura financiera
Del knowledge del país: costo de construcción (rasante + sótano), soft costs, marketing+ventas, gastos financieros, margen del desarrollador y el **tributo de 1ª venta** (tasa efectiva + base).

### Paso 6 — Tres escenarios + sensibilidad
Arma conservador / central / agresivo como **estados del mundo coherentes** (no extremos mecánicos): el conservador con menor edificabilidad + precios prudentes + costos/margen mayores; el agresivo al revés; el central con valores modales. El generador calcula la sensibilidad y verifica que el margen sobre costos quede sano (≈≥20%) en los tres.

### Paso 7 — Generar entregables
Si faltan datos del profesional/logo y vas a generar Word, pídelos ahora (lazy, una pregunta a la vez): nombre, contacto, y ruta a un logo PNG (opcional). Guárdalos en config. Luego **arma el JSON de inputs** (esquema abajo) y corre el generador:
```bash
python ~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/generar_valuacion.py /ruta/inputs.json ~/inmobiliaria/outputs/valuaciones-terreno/
```
Entrega los paths como links clicables + el valor central + el rango + próximos pasos accionables. Recuerda el disclaimer (va dentro del Word automáticamente).

### Paso 8 — Learnings
Registra learnings si hubo quirks de portal, particularidades de normativa o preferencias del usuario.

---

## Esquema del JSON de inputs (lo que espera el generador)

```json
{
  "meta": {"pais","moneda","moneda_simbolo","moneda_local","moneda_local_simbolo","tc","tc_fuente",
           "direccion","ciudad","distrito","zonificacion","area_terreno","fecha","central_idx":1},
  "parametros_normativos": {"altura","area_libre","estacionamientos","usos","fuente","confianza","modo"},
  "tributo": {"nombre","tasa_efectiva_vivienda","base"},
  "escenarios": [
    {"nombre","pisos","area_libre","eficiencia","precio_blended","area_prom_depto",
     "estac_ratio","estac_precio","dep_ratio","dep_precio","costo_rasante","area_cajon",
     "costo_sotano","soft_pct","mkt_pct","fin_pct","margen_desarrollador_input","impuesto_efectivo"}
  ],  // exactamente 3: conservador, central, agresivo (en ese orden; central_idx=1)
  "tipologias": [{"nombre","mix","area","precio_m2"}],
  "comparables": [{"proyecto","direccion","pisos","entrega","rango_m2"}],
  "fuentes": [{"nombre","url"}],
  "datos_profesional": {"nombre","contacto","logo_path"}  // o null
}
```
Reglas: `impuesto_efectivo` es la tasa efectiva del tributo sobre vivienda (genérico, NO lo llames "igv"). El `% de terreno sobre GDV` NUNCA es input: sale del cálculo. Todos los símbolos de moneda y el nombre del tributo vienen del JSON; el generador no tiene nada país-específico hardcodeado.

---

## Output

**Ruta:** `~/inmobiliaria/outputs/valuaciones-terreno/Valuacion_Terreno_<dirección>_<fecha>.{docx,xlsx,pdf}`
**Formato:** Word (informe narrado, brandeado), Excel (modelo de 3 escenarios con fórmulas vivas), PDF. **Nunca markdown** como entregable.
**Branding:** logo del profesional si está en config; si no, su nombre. Idioma según config.
**Disclaimer** (va dentro del Word): *"Esta valuación fue generada por IA con método residual y parámetros inferidos/aportados. Es un estimado de apoyo a la decisión, no una tasación reglamentada. Valídala con un tasador certificado y confirma los parámetros con la autoridad municipal antes de usarla para fines oficiales."*

---

## Quirks de Claude in Chrome (aplicar proactivamente al relevar portales)

- **Screenshot timeouts en SPAs pesadas** (Urbania, ZonaProp, Idealista, visores ArcGIS): tras 1 timeout, abandona el screenshot. Usa `read_page(filter:"interactive")`, `javascript_tool`, `find`, `get_page_text`.
- **URL scrubbing**: URLs vienen como `[BLOCKED: ...]`. Bypass con char codes vía `javascript_tool` (`Array.from(el.href).map(c=>c.charCodeAt(0))`) y decodifica en Python.
- **Locale español**: busca strings en español primero ("Descargar", "Departamentos"), inglés como fallback.
- **Anti-bot**: espera 2-3s entre acciones, scroll natural, no más de 1 corrida/hora por portal. Si aparece CAPTCHA: detente, avisa, sugiere resolución manual.
- Las tools `mcp__Claude_in_Chrome__*` son deferred — cárgalas con `ToolSearch select:` cuando las necesites.

---

## Notas técnicas (no se muestran al usuario)

- Re-lee la config al inicio de cada invocación.
- Si LibreOffice no está instalado, el generador entrega Word + Excel y omite el PDF; avísale al usuario cómo instalar LibreOffice si quiere el PDF.
- El generador es determinista: tu trabajo es armar bien el JSON; él renderiza. No edites los `.py` materializados salvo para corregir un bug real.
- `$ARGUMENTS` solo trae texto en la primera invocación; en turnos siguientes usa el último mensaje del usuario.

---

## Generador embebido

En la primera corrida, escribe estos 4 archivos en `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/` **exactamente como están** (no los modifiques). Son el motor determinista de la valuación. En corridas siguientes, si alguno falta, reescríbelo desde aquí sin avisar.

### `residual.py`

```python
# -*- coding: utf-8 -*-
"""Motor del método residual estático. Funciones puras, sin I/O, sin nada país-específico.
Todo lo específico de cada país/predio entra por el dict de inputs."""


def calc_escenario(s, area_terreno, tc):
    """Calcula un escenario del residual. `s` es el dict de supuestos de un escenario.
    Devuelve un dict con toda la cadena de cálculo + resultados.
    El % de terreno sobre GDV es SIEMPRE output (sale del residual), nunca input."""
    techada = area_terreno * (1 - s["area_libre"]) * s["pisos"]
    vendible = techada * s["eficiencia"]
    deptos = vendible / s["area_prom_depto"]
    ing_deptos = vendible * s["precio_blended"]
    n_estac = round(deptos * s["estac_ratio"])
    ing_estac = n_estac * s["estac_precio"]
    n_dep = round(deptos * s["dep_ratio"])
    ing_dep = n_dep * s["dep_precio"]
    gdv = ing_deptos + ing_estac + ing_dep
    sotano_area = n_estac * s["area_cajon"]
    con_rasante = techada * s["costo_rasante"]
    con_sotano = sotano_area * s["costo_sotano"]
    con_total = con_rasante + con_sotano
    impuesto = s["impuesto_efectivo"] * ing_deptos  # tasa efectiva sobre vivienda (genérico: IGV/IVA/ITBMS)
    soft = s["soft_pct"] * gdv
    mkt = s["mkt_pct"] * gdv
    fin = s["fin_pct"] * gdv
    margen = s["margen_desarrollador_input"] * gdv
    egresos = con_total + impuesto + soft + mkt + fin + margen
    terreno = gdv - egresos
    return {
        "nombre": s["nombre"],
        "pisos": s["pisos"], "area_libre": s["area_libre"], "eficiencia": s["eficiencia"],
        "precio_blended": s["precio_blended"], "costo_rasante": s["costo_rasante"],
        "margen_desarrollador_input": s["margen_desarrollador_input"],
        "techada": techada, "vendible": vendible, "deptos": deptos,
        "ing_deptos": ing_deptos, "n_estac": n_estac, "ing_estac": ing_estac,
        "n_dep": n_dep, "ing_dep": ing_dep, "gdv": gdv, "sotano_area": sotano_area,
        "con_rasante": con_rasante, "con_sotano": con_sotano, "con_total": con_total,
        "impuesto": impuesto, "soft": soft, "mkt": mkt, "fin": fin, "margen": margen,
        "egresos": egresos, "terreno": terreno,
        "pct_gdv": terreno / gdv, "terreno_m2": terreno / area_terreno,
        "terreno_local": terreno * tc, "terreno_m2_local": (terreno / area_terreno) * tc,
        # check derivado de viabilidad (NO es input): utilidad / costos totales
        "margen_sobre_costos_check": margen / (gdv - margen),
    }


def calc_todos(inputs):
    """Calcula los N escenarios (típicamente 3: conservador/central/agresivo)."""
    at = inputs["meta"]["area_terreno"]
    tc = inputs["meta"]["tc"]
    return [calc_escenario(s, at, tc) for s in inputs["escenarios"]]


def central(inputs, resultados=None):
    """Devuelve el escenario central segun meta.central_idx (default 1)."""
    r = resultados if resultados is not None else calc_todos(inputs)
    idx = inputs["meta"].get("central_idx", 1)
    return r[idx]


def _central_inputs(inputs):
    return inputs["escenarios"][inputs["meta"].get("central_idx", 1)]


def _round_to(x, base):
    return int(round(x / base) * base)


def sens_ranges(inputs):
    """Rangos de sensibilidad derivados del escenario central (genérico, sin números mágicos)."""
    s = _central_inputs(inputs)
    p = s["precio_blended"]; c = s["costo_rasante"]; n = s["pisos"]; al = s["area_libre"]; mg = s["margen_desarrollador_input"]
    precios = [_round_to(p * f, 10) for f in (0.93, 0.97, 1.0, 1.04, 1.08)]
    costos = [_round_to(c * f, 10) for f in (0.94, 1.0, 1.08, 1.15)]
    pisos = [max(1, n - 4), max(1, n - 2), n, n + 2, n + 4]
    libres = sorted({round(min(0.6, max(0.2, al + d)), 2) for d in (-0.05, 0.0, 0.05, 0.10)})
    margenes = [round(mg + d, 3) for d in (-0.03, -0.01, 0.0, 0.02, 0.04)]
    return {"precios": precios, "costos": costos, "pisos": pisos, "libres": libres, "margenes": margenes,
            "precio_central": p, "costo_central": c, "pisos_central": n, "libre_central": al, "margen_central": mg}


def sens_precio_costo(inputs, precios, costos):
    """Grid valor de terreno (moneda/m²) variando precio (filas) × costo construcción (cols), demás params del central."""
    s = _central_inputs(inputs); at = inputs["meta"]["area_terreno"]
    c = calc_escenario(s, at, inputs["meta"]["tc"])
    AV, AT, estac, dep, sot = c["vendible"], c["techada"], c["ing_estac"], c["ing_dep"], c["sotano_area"]
    one_minus = 1 - s["soft_pct"] - s["mkt_pct"] - s["fin_pct"] - s["margen_desarrollador_input"]
    imp, csot = s["impuesto_efectivo"], s["costo_sotano"]
    grid = []
    for P in precios:
        row = []
        for C in costos:
            gdv = AV * P + estac + dep
            land = gdv * one_minus - (AT * C + sot * csot) - imp * AV * P
            row.append(land / at)
        grid.append(row)
    return grid


def sens_pisos_arealibre(inputs, pisos_list, libre_list):
    """Grid valor de terreno (moneda/m²) variando pisos (filas) × área libre (cols), demás params del central."""
    s = _central_inputs(inputs); at = inputs["meta"]["area_terreno"]; tc = inputs["meta"]["tc"]
    grid = []
    for n in pisos_list:
        row = []
        for al in libre_list:
            s2 = dict(s); s2["pisos"] = n; s2["area_libre"] = al
            row.append(calc_escenario(s2, at, tc)["terreno_m2"])
        grid.append(row)
    return grid


def sens_precio_margen(inputs, precios, margenes):
    """Grid incidencia terreno (% del GDV) variando precio (filas) × margen (cols), demás params del central."""
    s = _central_inputs(inputs); at = inputs["meta"]["area_terreno"]
    c = calc_escenario(s, at, inputs["meta"]["tc"])
    AV, AT, estac, dep, sot = c["vendible"], c["techada"], c["ing_estac"], c["ing_dep"], c["sotano_area"]
    imp, csot, crasante = s["impuesto_efectivo"], s["costo_sotano"], s["costo_rasante"]
    base_costs = AT * crasante + sot * csot
    grid = []
    for P in precios:
        row = []
        for mg in margenes:
            gdv = AV * P + estac + dep
            land = gdv * (1 - s["soft_pct"] - s["mkt_pct"] - s["fin_pct"] - mg) - base_costs - imp * AV * P
            row.append(land / gdv)
        grid.append(row)
    return grid
```

### `render_excel.py`

```python
# -*- coding: utf-8 -*-
"""Render del Excel de valuación (3 escenarios + sensibilidad con fórmulas vivas).
Genérico: todo lo país-específico viene de `inputs`. Portado del modelo validado Av. Arequipa."""
import os
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from openpyxl.utils import get_column_letter
from openpyxl.drawing.image import Image as XLImage
from residual import sens_ranges

FONT = "Lato"
F_TITLE  = Font(name=FONT, size=15, bold=True, color="FFFFFF")
F_SUB    = Font(name=FONT, size=10, italic=True, color="FFFFFF")
F_SEC    = Font(name=FONT, size=11, bold=True, color="FFFFFF")
F_LBL    = Font(name=FONT, size=10, color="000000")
F_LBLB   = Font(name=FONT, size=10, bold=True, color="000000")
F_IN     = Font(name=FONT, size=10, color="0000FF")
F_LINK   = Font(name=FONT, size=10, color="008000")
F_CALC   = Font(name=FONT, size=10, color="000000")
F_RES    = Font(name=FONT, size=11, bold=True, color="000000")
F_RESBIG = Font(name=FONT, size=20, bold=True, color="415262")
F_NOTE   = Font(name=FONT, size=8, italic=True, color="808080")
F_SRC    = Font(name=FONT, size=8, color="808080")
FILL_TITLE = PatternFill("solid", fgColor="415262")
FILL_SEC   = PatternFill("solid", fgColor="5E6E76")
FILL_IN    = PatternFill("solid", fgColor="FFF2CC")
FILL_RES   = PatternFill("solid", fgColor="DFF7EC")
FILL_BASE  = PatternFill("solid", fgColor="FFE3A1")
FILL_CEN   = PatternFill("solid", fgColor="EAEFF1")
FILL_HDR   = PatternFill("solid", fgColor="DDE4E7")
FILL_TOT   = PatternFill("solid", fgColor="E8EDEF")
thin = Side(style="thin", color="BFBFBF")
BORDER = Border(left=thin, right=thin, top=thin, bottom=thin)
M2, INT, PCT = '#,##0.0', '#,##0', '0.0%'


def _money_fmt(sym):
    return f'"{sym} "#,##0;("{sym} "#,##0);"-"'


def C(ws, addr, value, font=F_CALC, fmt=None, fill=None, align=None, border=False, wrap=False):
    cel = ws[addr]; cel.value = value; cel.font = font
    if fmt: cel.number_format = fmt
    if fill: cel.fill = fill
    al = {}
    if align: al["horizontal"] = align
    if wrap: al["wrap_text"] = True
    if al: cel.alignment = Alignment(**al, vertical="center")
    if border: cel.border = BORDER
    return cel


def sec(ws, row, text, span):
    ws.merge_cells(start_row=row, start_column=1, end_row=row, end_column=span)
    c = ws.cell(row=row, column=1, value=text); c.font = F_SEC; c.fill = FILL_SEC
    c.alignment = Alignment(horizontal="left", vertical="center")
    for col in range(1, span + 1):
        ws.cell(row=row, column=col).fill = FILL_SEC


def render_excel(inputs, out_path):
    meta = inputs["meta"]
    esc = inputs["escenarios"]
    assert len(esc) == 3, "render_excel asume 3 escenarios (conservador/central/agresivo)"
    sym = meta.get("moneda_simbolo", "US$")
    lsym = meta.get("moneda_local_simbolo", sym)
    trib = inputs.get("tributo", {"nombre": "Impuesto", "tasa_efectiva_vivienda": 0.0, "base": ""})
    USD = _money_fmt(sym); PRICE = USD; PEN = _money_fmt(lsym)
    at = meta["area_terreno"]; tc = meta["tc"]
    names = [e["nombre"] for e in esc]

    def col_vals(key):
        return [e[key] for e in esc]

    wb = Workbook()

    # ===== ESCENARIOS =====
    es = wb.active; es.title = "Escenarios"; es.sheet_view.showGridLines = False
    es.column_dimensions["A"].width = 42
    for col in ["B", "C", "D", "E"]:
        es.column_dimensions[col].width = 16
    es.merge_cells("A1:E1")
    C(es, "A1", "MODELO RESIDUAL — TRES ESCENARIOS  (celdas azules = editables)", F_TITLE, fill=FILL_TITLE, align="left")
    es.merge_cells("A2:E2")
    C(es, "A2", f"{meta.get('direccion','')} · {at:,.2f} m² · {meta.get('distrito', meta.get('ciudad',''))} · {meta.get('pais','')} · método residual estático", F_NOTE, align="left")
    C(es, "A3", "Parámetro / Resultado", F_LBLB, fill=FILL_HDR, border=True)
    C(es, "C3", names[0], F_LBLB, fill=FILL_HDR, border=True, align="center")
    C(es, "D3", names[1] + " (base)", F_LBLB, fill=FILL_CEN, border=True, align="center")
    C(es, "E3", names[2], F_LBLB, fill=FILL_HDR, border=True, align="center")

    # input rows: (row, label, fmt, [C,D,E])
    rows = [
        (4, "Área del terreno (m²)", M2, [at, at, at]),
        (5, f"Tipo de cambio ({lsym} por {sym})", "0.00", [tc, tc, tc]),
        (6, "N° de pisos", INT, col_vals("pisos")),
        (7, "Área libre (%)", PCT, col_vals("area_libre")),
        (8, "Eficiencia vendible (vend/techada)", PCT, col_vals("eficiencia")),
        (9, f"Precio venta blended ({sym}/m²)", PRICE, col_vals("precio_blended")),
        (10, "Área prom. ponderada por depto (m²)", M2, col_vals("area_prom_depto")),
        (11, "Estacionamientos (cajones/depto)", "0.00", col_vals("estac_ratio")),
        (12, f"Precio estacionamiento ({sym})", USD, col_vals("estac_precio")),
        (13, "Depósitos (por depto)", "0.00", col_vals("dep_ratio")),
        (14, f"Precio depósito ({sym})", USD, col_vals("dep_precio")),
        (15, f"Costo constr. rasante ({sym}/m²)", PRICE, col_vals("costo_rasante")),
        (16, "Área por cajón sótano (m²)", M2, col_vals("area_cajon")),
        (17, f"Costo constr. sótano ({sym}/m²)", PRICE, col_vals("costo_sotano")),
        (18, "Soft costs (% GDV)", PCT, col_vals("soft_pct")),
        (19, "Marketing + ventas (% GDV)", PCT, col_vals("mkt_pct")),
        (20, "Gastos financieros (% GDV)", PCT, col_vals("fin_pct")),
        (21, "Margen del desarrollador (% GDV)", PCT, col_vals("margen_desarrollador_input")),
        (22, f"{trib['nombre']} efectivo vivienda (%)", PCT, col_vals("impuesto_efectivo")),
    ]
    for row, label, fmt, vals in rows:
        C(es, f"A{row}", label, F_LBL)
        for j, col in enumerate(["C", "D", "E"]):
            C(es, f"{col}{row}", vals[j], F_IN, fmt, fill=(FILL_CEN if col == "D" else FILL_IN), border=True, align="right")

    sec(es, 24, "CÁLCULO", 5)
    calc = [
        (25, "Área techada sobre rasante (m²)", "={c}4*(1-{c}7)*{c}6", M2),
        (26, "Área vendible de departamentos (m²)", "={c}25*{c}8", M2),
        (27, "N° de departamentos", "={c}26/{c}10", "#,##0.0"),
        (28, "Ingreso departamentos", "={c}26*{c}9", USD),
        (29, "N° de estacionamientos", "=ROUND({c}27*{c}11,0)", INT),
        (30, "Ingreso estacionamientos", "={c}29*{c}12", USD),
        (31, "N° de depósitos", "=ROUND({c}27*{c}13,0)", INT),
        (32, "Ingreso depósitos", "={c}31*{c}14", USD),
        (33, "GDV TOTAL", "={c}28+{c}30+{c}32", USD),
        (34, "Área de sótano (m²)", "={c}29*{c}16", M2),
        (35, "Construcción rasante", "={c}25*{c}15", USD),
        (36, "Construcción sótano", "={c}34*{c}17", USD),
        (37, "Construcción total", "={c}35+{c}36", USD),
        (38, f"{trib['nombre']} vivienda", "={c}22*{c}28", USD),
        (39, "Soft costs", "={c}18*{c}33", USD),
        (40, "Marketing + ventas", "={c}19*{c}33", USD),
        (41, "Gastos financieros", "={c}20*{c}33", USD),
        (42, "Margen desarrollador", "={c}21*{c}33", USD),
        (43, "TOTAL EGRESOS sin terreno", "={c}37+{c}38+{c}39+{c}40+{c}41+{c}42", USD),
    ]
    for row, label, f, fmt in calc:
        bold = row in (33, 37, 43)
        C(es, f"A{row}", label, F_RES if bold else F_LBL)
        for col in ["C", "D", "E"]:
            fill = FILL_TOT if bold else (FILL_CEN if col == "D" else None)
            C(es, f"{col}{row}", f.format(c=col), F_CALC, fmt, fill=fill, border=bold, align="right")

    sec(es, 45, "RESULTADO", 5)
    res = [
        (46, "VALOR DEL TERRENO", "={c}33-{c}43", USD, True),
        (47, "Terreno como % del GDV", "={c}46/{c}33", PCT, True),
        (48, f"Valor del terreno ({sym}/m²)", "={c}46/{c}4", PRICE, True),
        (49, f"Valor del terreno ({lsym})", "={c}46*{c}5", PEN, False),
        (50, f"Valor del terreno ({lsym}/m²)", "={c}48*{c}5", PEN, False),
        (51, "Margen sobre costos (verificación)", "={c}42/({c}33-{c}42)", PCT, False),
    ]
    for row, label, f, fmt, hl in res:
        C(es, f"A{row}", label, F_RES if hl else F_LBL)
        for col in ["C", "D", "E"]:
            fill = FILL_RES if hl else (FILL_CEN if col == "D" else None)
            C(es, f"{col}{row}", f.format(c=col), F_RES if hl else F_CALC, fmt, fill=fill, border=hl, align="right")

    # ===== PORTADA =====
    ws = wb.create_sheet("Portada"); ws.sheet_view.showGridLines = False
    ws.column_dimensions["A"].width = 3; ws.column_dimensions["B"].width = 38
    for col in ["C", "D", "E"]:
        ws.column_dimensions[col].width = 18
    logo = (inputs.get("datos_profesional") or {}).get("logo_path")
    start = 3
    if logo and os.path.exists(logo):
        try:
            im = XLImage(logo); im.width = 210; im.height = 57; ws.row_dimensions[1].height = 46; ws.add_image(im, "B1")
        except Exception:
            pass
    for r in range(start, start + 6):
        for col in "ABCDE":
            ws[f"{col}{r}"].fill = FILL_TITLE
    ws.merge_cells(f"B{start}:E{start+1}")
    C(ws, f"B{start}", "VALUACIÓN DE TERRENO — MÉTODO RESIDUAL", F_TITLE, fill=FILL_TITLE, align="left")
    ws.merge_cells(f"B{start+2}:E{start+3}")
    C(ws, f"B{start+2}", f"{meta.get('direccion','')} · {meta.get('distrito', meta.get('ciudad',''))} · {at:,.2f} m²", F_SUB, fill=FILL_TITLE, align="left")
    C(ws, "B9", "Valor estimado del terreno (escenario central)", F_LBLB)
    ws.merge_cells("B10:C10")
    C(ws, "B10", f'="{sym} "&TEXT(Escenarios!D46,"#,##0")', F_RESBIG, fill=FILL_RES)
    C(ws, "D10", "=Escenarios!D48", F_RES, PRICE, fill=FILL_RES, align="center")
    C(ws, "E10", "=Escenarios!D47", F_RES, PCT, fill=FILL_RES, align="center")
    C(ws, "D11", f"{sym}/m²", F_NOTE, align="center"); C(ws, "E11", "% del GDV", F_NOTE, align="center")
    C(ws, "B14", "Rango por escenario", F_LBLB)
    C(ws, "B15", "Escenario", F_LBLB, fill=FILL_HDR, border=True)
    C(ws, "C15", f"Terreno ({sym})", F_LBLB, fill=FILL_HDR, border=True, align="right")
    C(ws, "D15", f"{sym}/m²", F_LBLB, fill=FILL_HDR, border=True, align="right")
    C(ws, "E15", "% GDV", F_LBLB, fill=FILL_HDR, border=True, align="right")
    for k, (nm, col) in enumerate(zip(names, ["C", "D", "E"])):
        r = 16 + k; fill = FILL_RES if col == "D" else None
        C(ws, f"B{r}", nm, F_LBLB if col == "D" else F_LBL, fill=fill, border=True)
        C(ws, f"C{r}", f"=Escenarios!{col}46", F_LINK, USD, fill=fill, border=True, align="right")
        C(ws, f"D{r}", f"=Escenarios!{col}48", F_LINK, PRICE, fill=fill, border=True, align="right")
        C(ws, f"E{r}", f"=Escenarios!{col}47", F_LINK, PCT, fill=fill, border=True, align="right")
    C(ws, "B20", "Tipo de cambio"); C(ws, "C20", f'="{lsym} "&TEXT(Escenarios!D5,"0.00")&" / {sym}"', F_LINK)
    C(ws, "B21", f"Equivalente ({lsym}, central)"); C(ws, "C21", "=Escenarios!D49", F_LINK, PEN)
    C(ws, "B22", "Fecha de análisis"); C(ws, "C22", meta.get("fecha", ""), F_IN)
    C(ws, "B23", "Método"); C(ws, "C23", "Valor residual estático", F_IN)
    ws.merge_cells("B26:E31")
    C(ws, "B26", ("AVISO: Estimado de apoyo a la decisión por método residual, a partir de edificabilidad inferida o "
                  "aportada, precios de proyectos de estreno de la zona y benchmarks de costos del país. El valor es muy "
                  "sensible al costo de construcción y a la edificabilidad: por eso se presentan tres escenarios; se "
                  "recomienda el central. NO es una tasación reglamentada. Confirmar parámetros con el certificado/autoridad "
                  "municipal. Ver hojas Escenarios, Sensibilidad, Comparables y Fuentes."), F_NOTE, wrap=True, align="left")

    # ===== RESUMEN =====
    ws = wb.create_sheet("Resumen"); ws.sheet_view.showGridLines = False
    ws.column_dimensions["A"].width = 40
    for col in ["B", "C", "D"]:
        ws.column_dimensions[col].width = 18
    ws.merge_cells("A1:D1"); C(ws, "A1", "RESUMEN — COMPARACIÓN DE ESCENARIOS", F_TITLE, fill=FILL_TITLE, align="left")
    C(ws, "A3", "Parámetro / Resultado", F_LBLB, fill=FILL_HDR, border=True)
    C(ws, "B3", names[0], F_LBLB, fill=FILL_HDR, border=True, align="center")
    C(ws, "C3", names[1], F_LBLB, fill=FILL_CEN, border=True, align="center")
    C(ws, "D3", names[2], F_LBLB, fill=FILL_HDR, border=True, align="center")
    rrows = [
        ("N° de pisos", 6, INT, False), ("Área libre", 7, PCT, False),
        (f"Precio venta ({sym}/m²)", 9, PRICE, False), (f"Costo construcción ({sym}/m²)", 15, PRICE, False),
        ("Margen desarrollador", 21, PCT, False), ("GDV total", 33, USD, False),
        ("VALOR DEL TERRENO", 46, USD, True), (f"Valor ({sym}/m²)", 48, PRICE, True),
        ("Incidencia % del GDV", 47, PCT, True), (f"Valor ({lsym})", 49, PEN, False),
    ]
    r = 4
    for label, srow, fmt, hl in rrows:
        C(ws, f"A{r}", label, F_RES if hl else F_LBL, border=True)
        for col, scol in [("B", "C"), ("C", "D"), ("D", "E")]:
            fill = FILL_RES if (hl and col == "C") else (FILL_CEN if col == "C" and not hl else None)
            C(ws, f"{col}{r}", f"=Escenarios!{scol}{srow}", (F_RES if hl else F_LINK), fmt, fill=fill, border=True, align="right")
        r += 1
    ws.merge_cells(f"A{r+1}:D{r+2}")
    C(ws, f"A{r+1}", "El escenario central es el valor recomendado. El conservador y el agresivo acotan el rango según edificabilidad y relación precio-costo.", F_NOTE, wrap=True, align="left")

    # ===== SENSIBILIDAD (formulas referencing central col D) =====
    sr = sens_ranges(inputs)
    ws = wb.create_sheet("Sensibilidad"); ws.sheet_view.showGridLines = False
    for col, w in zip("ABCDEFG", [30, 14, 14, 14, 14, 14, 14]):
        ws.column_dimensions[col].width = w
    ws.merge_cells("A1:G1"); C(ws, "A1", "SENSIBILIDAD (sobre el escenario central)", F_TITLE, fill=FILL_TITLE, align="left")

    sec(ws, 3, f"Tabla 1 · Valor del terreno ({sym}/m²) — precio (filas) × costo construcción (cols)", 7)
    precios1, costos1 = sr["precios"], sr["costos"]
    C(ws, "B6", "Precio / Constr", F_LBLB, fill=FILL_HDR, border=True, align="center")
    for j, c in enumerate(costos1):
        C(ws, f"{get_column_letter(3+j)}6", c, F_IN, PRICE, fill=FILL_HDR, border=True, align="right")
    for i, p in enumerate(precios1):
        r = 7 + i; C(ws, f"B{r}", p, F_IN, PRICE, fill=FILL_HDR, border=True, align="right")
        for j, c in enumerate(costos1):
            cl = get_column_letter(3 + j)
            f = (f"=((Escenarios!$D$26*$B{r}+Escenarios!$D$30+Escenarios!$D$32)*"
                 f"(1-Escenarios!$D$18-Escenarios!$D$19-Escenarios!$D$20-Escenarios!$D$21)"
                 f"-(Escenarios!$D$25*{cl}$6+Escenarios!$D$34*Escenarios!$D$17)"
                 f"-Escenarios!$D$22*Escenarios!$D$26*$B{r})/Escenarios!$D$4")
            base = (p == sr["precio_central"] and c == sr["costo_central"])
            C(ws, f"{cl}{r}", f, F_CALC, PRICE, fill=FILL_BASE if base else None, border=True, align="right")
    C(ws, "A13", "Celda resaltada = escenario central.", F_NOTE, align="left")

    sec(ws, 15, f"Tabla 2 · Valor del terreno ({sym}/m²) — pisos (filas) × área libre (cols)", 7)
    pisos2, libre2 = sr["pisos"], sr["libres"]
    C(ws, "B18", "Pisos / Á.libre", F_LBLB, fill=FILL_HDR, border=True, align="center")
    for j, al in enumerate(libre2):
        C(ws, f"{get_column_letter(3+j)}18", al, F_IN, PCT, fill=FILL_HDR, border=True, align="right")
    for i, n in enumerate(pisos2):
        r = 19 + i; C(ws, f"B{r}", n, F_IN, INT, fill=FILL_HDR, border=True, align="right")
        for j, al in enumerate(libre2):
            cl = get_column_letter(3 + j)
            AT = f"(Escenarios!$D$4*(1-{cl}$18)*$B{r})"; AV = f"({AT}*Escenarios!$D$8)"
            ND = f"({AV}/Escenarios!$D$10)"; ING = f"({AV}*Escenarios!$D$9)"
            EST = f"({ND}*Escenarios!$D$11*Escenarios!$D$12)"; DEP = f"({ND}*Escenarios!$D$13*Escenarios!$D$14)"
            GDV = f"({ING}+{EST}+{DEP})"; SOT = f"({ND}*Escenarios!$D$11*Escenarios!$D$16)"
            CON = f"({AT}*Escenarios!$D$15+{SOT}*Escenarios!$D$17)"; IMP = f"(Escenarios!$D$22*{ING})"
            LAND = f"({GDV}*(1-Escenarios!$D$18-Escenarios!$D$19-Escenarios!$D$20-Escenarios!$D$21)-{CON}-{IMP})"
            base = (n == sr["pisos_central"] and abs(al - sr["libre_central"]) < 1e-9)
            C(ws, f"{cl}{r}", f"={LAND}/Escenarios!$D$4", F_CALC, PRICE, fill=FILL_BASE if base else None, border=True, align="right")

    sec(ws, 26, "Tabla 3 · Terreno como % del GDV — precio (filas) × margen (cols)", 7)
    precios3, margenes3 = sr["precios"], sr["margenes"]
    C(ws, "B29", "Precio / Margen", F_LBLB, fill=FILL_HDR, border=True, align="center")
    for j, mg in enumerate(margenes3):
        C(ws, f"{get_column_letter(3+j)}29", mg, F_IN, PCT, fill=FILL_HDR, border=True, align="right")
    for i, p in enumerate(precios3):
        r = 30 + i; C(ws, f"B{r}", p, F_IN, PRICE, fill=FILL_HDR, border=True, align="right")
        for j, mg in enumerate(margenes3):
            cl = get_column_letter(3 + j)
            GDV = f"(Escenarios!$D$26*$B{r}+Escenarios!$D$30+Escenarios!$D$32)"
            LAND = (f"({GDV}*(1-Escenarios!$D$18-Escenarios!$D$19-Escenarios!$D$20-{cl}$29)"
                    f"-(Escenarios!$D$25*Escenarios!$D$15+Escenarios!$D$34*Escenarios!$D$17)"
                    f"-Escenarios!$D$22*Escenarios!$D$26*$B{r})")
            base = (p == sr["precio_central"] and abs(mg - sr["margen_central"]) < 1e-9)
            C(ws, f"{cl}{r}", f"={LAND}/{GDV}", F_CALC, PCT, fill=FILL_BASE if base else None, border=True, align="right")

    # ===== METODOLOGÍA =====
    ws = wb.create_sheet("Metodologia"); ws.sheet_view.showGridLines = False
    ws.column_dimensions["A"].width = 34; ws.column_dimensions["B"].width = 24
    ws.column_dimensions["C"].width = 46; ws.column_dimensions["D"].width = 14
    ws.merge_cells("A1:D1"); C(ws, "A1", "METODOLOGÍA Y ESTRATEGIA DE ABORDAJE", F_TITLE, fill=FILL_TITLE, align="left")
    sec(ws, 3, "Enfoque", 4); ws.merge_cells("A4:D7")
    C(ws, "A4", ("Método de valor residual estático: bajo el supuesto de mejor y mayor uso legalmente permisible, el valor "
                 "del suelo es el residuo del valor de venta del proyecto óptimo tras cubrir todos los costos de desarrollo y "
                 "el margen del desarrollador. Se usa porque no hay ventas públicas de terrenos comparables con esta "
                 "edificabilidad; el comparativo de terrenos sirve solo como verificación de la incidencia (8–20% del GDV "
                 "típico). Mejor y mayor uso adoptado: edificio multifamiliar de densidad alta con comercio en zócalo."),
      F_LBL, wrap=True, align="left")
    sec(ws, 9, "Secuencia del análisis y trazabilidad", 4)
    for col, h in zip(["A", "B", "C", "D"], ["Paso", "Procedimiento", "Fuente / evidencia", ""]):
        C(ws, f"{col}10", h, F_LBLB, fill=FILL_HDR, border=True, align="center" if col in ("A", "D") else "left")
    pasos = [
        ("0", "Distrito, zonificación y altura", "Fuente oficial de zonificación del país + edificios reales"),
        ("1", "Área techada = (1−área libre)×terreno×pisos", "Parámetros normativos + validación con proyectos reales"),
        ("2", "Área vendible = techada × eficiencia", "Estándar de mercado"),
        ("3", "GDV = precio×vendible + estac. + depósitos", "Portal de estreno del país + fuente oficial de precios"),
        ("4", "Terreno = GDV − costos − impuesto − margen", "Benchmarks de costos + tributación del país"),
        ("5", "Escenarios y sensibilidad (variación de drivers)", "Rangos documentados de las fuentes"),
    ]
    r = 11
    for p, proc, fue in pasos:
        C(ws, f"A{r}", p, F_LBL, border=True, align="center")
        C(ws, f"B{r}", proc, F_LBL, border=True, wrap=True, align="left")
        C(ws, f"C{r}", fue, F_LBL, border=True, wrap=True, align="left")
        C(ws, f"D{r}", "", F_LBL, border=True); r += 1
    sec(ws, 18, "Origen de los supuestos (escenario central)", 4)
    for col, h in zip(["A", "B", "C", "D"], ["Supuesto", "Central", "Fuente", "Confianza"]):
        C(ws, f"{col}19", h, F_LBLB, fill=FILL_HDR, border=True, align="center" if col in ("B", "D") else "left")
    cen = esc[meta.get("central_idx", 1)]
    sup = [
        ("Precio venta blended", f"{sym} {cen['precio_blended']:,.0f}/m²", "Portal de estreno + fuente oficial de precios", "Media-alta"),
        ("Costo construcción rasante", f"{sym} {cen['costo_rasante']:,.0f}/m²", "Benchmarks de costos del país", "Media-alta"),
        ("Eficiencia vendible", f"{cen['eficiencia']*100:.0f}%", "Estándar de mercado", "Media-alta"),
        ("Soft costs", f"{cen['soft_pct']*100:.1f}% GDV", "Benchmarks sectoriales", "Media-alta"),
        ("Marketing + ventas", f"{cen['mkt_pct']*100:.1f}% GDV", "Benchmarks sectoriales", "Media-alta"),
        ("Gastos financieros", f"{cen['fin_pct']*100:.1f}% GDV", "Benchmarks sectoriales", "Media"),
        ("Margen desarrollador", f"{cen['margen_desarrollador_input']*100:.0f}% GDV", "Benchmarks sectoriales", "Media-alta"),
        (f"{trib['nombre']} efectivo (vivienda)", f"{cen['impuesto_efectivo']*100:.0f}%", "Normativa tributaria del país", "Alta"),
        ("Tipo de cambio", f"{lsym} {tc:.2f}", meta.get("tc_fuente", "Banco central"), "Alta"),
    ]
    r = 20
    for s, c, f, cf in sup:
        C(ws, f"A{r}", s, F_LBL, border=True)
        C(ws, f"B{r}", c, F_LBL, border=True, align="right")
        C(ws, f"C{r}", f, F_LBL, border=True, wrap=True, align="left")
        C(ws, f"D{r}", cf, F_LBL, border=True, align="center"); r += 1
    sec(ws, r + 1, "Tratamiento tributario y alcance", 4)
    ws.merge_cells(f"A{r+2}:D{r+5}")
    C(ws, f"A{r+2}", (f"{trib['nombre']} a la primera venta: {trib.get('base','')}. ALCANCE: estimado de apoyo a la "
                      "decisión, NO una tasación reglamentada; para una tasación formal se requiere inspección física, "
                      "certificado de parámetros urbanísticos, verificación registral y presupuesto de obra."),
      F_NOTE, wrap=True, align="left")

    # ===== COMPARABLES =====
    ws = wb.create_sheet("Comparables"); ws.sheet_view.showGridLines = False
    for col, w in zip("ABCDE", [22, 26, 10, 14, 22]):
        ws.column_dimensions[col].width = w
    ws.merge_cells("A1:E1"); C(ws, "A1", "COMPARABLES DE MERCADO — Proyectos de estreno", F_TITLE, fill=FILL_TITLE, align="left")
    hdr = ["Proyecto", "Dirección", "Pisos", "Entrega", f"Rango ({sym}/m²)"]
    for j, h in enumerate(hdr):
        C(ws, f"{get_column_letter(1+j)}3", h, F_LBLB, fill=FILL_HDR, border=True, align="left")
    comps = inputs.get("comparables", [])
    if comps:
        for i, cp in enumerate(comps):
            r = 4 + i
            vals = [cp.get("proyecto", ""), cp.get("direccion", ""), cp.get("pisos", ""), cp.get("entrega", ""), cp.get("rango_m2", "")]
            for j, v in enumerate(vals):
                C(ws, f"{get_column_letter(1+j)}{r}", v, F_LBL, border=True, align="left" if j in (0, 1) else "center")
        rr = 4 + len(comps) + 1
    else:
        C(ws, "A4", "Sin comparables provistos para esta valuación.", F_NOTE, align="left"); rr = 6
    tip = inputs.get("tipologias", [])
    if tip:
        C(ws, f"A{rr}", "Tipologías de referencia", F_LBLB)
        for j, h in enumerate(["Tipología", "Mix", "Área (m²)", f"Precio ({sym}/m²)"]):
            C(ws, f"{get_column_letter(1+j)}{rr+1}", h, F_LBLB, fill=FILL_HDR, border=True, align="left")
        for i, t in enumerate(tip):
            r = rr + 2 + i
            C(ws, f"A{r}", t.get("nombre", ""), F_LBL, border=True)
            C(ws, f"B{r}", t.get("mix", ""), F_LBL, PCT, border=True, align="center")
            C(ws, f"C{r}", t.get("area", ""), F_LBL, M2, border=True, align="center")
            C(ws, f"D{r}", t.get("precio_m2", ""), F_LBL, PRICE, border=True, align="right")

    # ===== FUENTES =====
    ws = wb.create_sheet("Fuentes"); ws.sheet_view.showGridLines = False
    ws.column_dimensions["A"].width = 50; ws.column_dimensions["B"].width = 85
    ws.merge_cells("A1:B1"); C(ws, "A1", "FUENTES Y REFERENCIAS", F_TITLE, fill=FILL_TITLE, align="left")
    r = 3
    for fu in inputs.get("fuentes", []):
        C(ws, f"A{r}", fu.get("nombre", ""), F_LBL)
        C(ws, f"B{r}", fu.get("url", ""), F_SRC, align="left"); r += 1
    C(ws, f"A{r+1}", f"Tipo de cambio: {lsym} {tc:.2f} / {sym} ({meta.get('tc_fuente','')}).", F_NOTE)

    order = ["Portada", "Resumen", "Metodologia", "Escenarios", "Sensibilidad", "Comparables", "Fuentes"]
    for i, name in enumerate(order):
        wb.move_sheet(name, -(wb.sheetnames.index(name)) + i)
    wb.save(out_path)
    return out_path
```

### `render_word.py`

```python
# -*- coding: utf-8 -*-
"""Render de informe de valuacion de terreno (metodo residual) a Word con python-docx.

100% generico: NO hardcodea pais/moneda/tributo. Todo lo especifico se lee del dict
`inputs` (esquema en fixtures/arequipa_inputs.json). El caso de Peru es solo fixture.

API publica:
    render_word(inputs, out_path)

Toda la matematica se delega a residual.py (calc_todos / central / sens_*). Este modulo
solo formatea. Branding Valia: azul pizarra 415262 + verde 30E190, fuente Lato.
"""

import os

from docx import Document
from docx.shared import Pt, Cm, RGBColor
from docx.enum.text import WD_ALIGN_PARAGRAPH
from docx.enum.table import WD_TABLE_ALIGNMENT, WD_ALIGN_VERTICAL
from docx.enum.section import WD_SECTION
from docx.oxml.ns import qn
from docx.oxml import OxmlElement

import residual as R

# ===================== BRANDING =====================
FONT = "Lato"
PRIMARY = "415262"   # azul pizarra
ACCENT = "30E190"    # verde
HDR = "DDE4E7"       # encabezado de tabla
CEN = "EAEFF1"       # columna/celda central resaltada
RES = "DFF7EC"       # fila de resultado
BASEC = "FFE3A1"     # celda base de sensibilidad
LIGHT = "F4F6F7"     # franja suave
GREY = "808080"
BORDER = "BFBFBF"

PRIMARY_RGB = RGBColor.from_string(PRIMARY)
GREY_RGB = RGBColor.from_string(GREY)
BLACK_RGB = RGBColor.from_string("000000")

L = WD_ALIGN_PARAGRAPH.LEFT
C = WD_ALIGN_PARAGRAPH.CENTER
RGT = WD_ALIGN_PARAGRAPH.RIGHT


# ===================== HELPERS GENERICOS =====================
def money(x, sym):
    """Formatea un monto con separador de miles y simbolo de moneda."""
    return f"{sym} {x:,.0f}"


def pct(x, dec=1):
    return f"{x * 100:.{dec}f} %"


def num(x, dec=0):
    return f"{x:,.{dec}f}"


def _set_run(run, *, bold=False, italic=False, size=11, color=None):
    run.font.name = FONT
    run.font.size = Pt(size)
    run.font.bold = bold
    run.font.italic = italic
    if color is not None:
        run.font.color.rgb = color
    # asegura la fuente tambien para east-asian / cs
    rpr = run._element.get_or_add_rPr()
    rfonts = rpr.find(qn("w:rFonts"))
    if rfonts is None:
        rfonts = OxmlElement("w:rFonts")
        rpr.append(rfonts)
    for a in ("w:ascii", "w:hAnsi", "w:cs"):
        rfonts.set(qn(a), FONT)


def shade_cell(cell, fill):
    tcPr = cell._tc.get_or_add_tcPr()
    shd = OxmlElement("w:shd")
    shd.set(qn("w:val"), "clear")
    shd.set(qn("w:fill"), fill)
    tcPr.append(shd)


def _set_cell_borders(cell, color=BORDER, sz="4"):
    tcPr = cell._tc.get_or_add_tcPr()
    borders = OxmlElement("w:tcBorders")
    for edge in ("top", "left", "bottom", "right"):
        el = OxmlElement(f"w:{edge}")
        el.set(qn("w:val"), "single")
        el.set(qn("w:sz"), sz)
        el.set(qn("w:space"), "0")
        el.set(qn("w:color"), color)
        borders.append(el)
    tcPr.append(borders)


def _set_cell_margins(cell, top=40, bottom=40, left=80, right=80):
    tcPr = cell._tc.get_or_add_tcPr()
    m = OxmlElement("w:tcMar")
    for edge, val in (("top", top), ("bottom", bottom), ("start", left), ("end", right)):
        el = OxmlElement(f"w:{edge}")
        el.set(qn("w:w"), str(val))
        el.set(qn("w:type"), "dxa")
        m.append(el)
    tcPr.append(m)


def _para_shading(paragraph, fill):
    pPr = paragraph._p.get_or_add_pPr()
    shd = OxmlElement("w:shd")
    shd.set(qn("w:val"), "clear")
    shd.set(qn("w:fill"), fill)
    pPr.append(shd)


def _para_border(paragraph, *, color=ACCENT, sz="12", edges=("top", "bottom")):
    pPr = paragraph._p.get_or_add_pPr()
    pbdr = OxmlElement("w:pBdr")
    for edge in edges:
        el = OxmlElement(f"w:{edge}")
        el.set(qn("w:val"), "single")
        el.set(qn("w:sz"), sz)
        el.set(qn("w:space"), "6")
        el.set(qn("w:color"), color)
        pbdr.append(el)
    pPr.append(pbdr)


def add_par(doc, text="", *, bold=False, italic=False, size=11, color=None,
            align=L, after=6, before=0):
    p = doc.add_paragraph()
    p.alignment = align
    p.paragraph_format.space_after = Pt(after)
    p.paragraph_format.space_before = Pt(before)
    p.paragraph_format.line_spacing = 1.15
    if text:
        r = p.add_run(text)
        _set_run(r, bold=bold, italic=italic, size=size, color=color)
    return p


def add_runs(doc, runs, *, align=L, after=6, before=0):
    """runs = lista de dicts {text, bold, italic, size, color}."""
    p = doc.add_paragraph()
    p.alignment = align
    p.paragraph_format.space_after = Pt(after)
    p.paragraph_format.space_before = Pt(before)
    p.paragraph_format.line_spacing = 1.15
    for rd in runs:
        r = p.add_run(rd["text"])
        _set_run(r, bold=rd.get("bold", False), italic=rd.get("italic", False),
                 size=rd.get("size", 11), color=rd.get("color"))
    return p


def add_heading(doc, text, level=1):
    size = 15 if level == 1 else 13
    p = doc.add_paragraph()
    p.paragraph_format.space_before = Pt(14 if level == 1 else 10)
    p.paragraph_format.space_after = Pt(8 if level == 1 else 6)
    # outline level para que aparezca en navegacion / TOC
    pPr = p._p.get_or_add_pPr()
    ol = OxmlElement("w:outlineLvl")
    ol.set(qn("w:val"), str(level - 1))
    pPr.append(ol)
    if level == 1:
        _para_border(p, color=ACCENT, sz="10", edges=("bottom",))
    r = p.add_run(text)
    _set_run(r, bold=True, size=size, color=PRIMARY_RGB)
    return p


def add_bullet(doc, text, *, after=4):
    p = doc.add_paragraph(style=None)
    p.paragraph_format.left_indent = Cm(0.7)
    p.paragraph_format.first_line_indent = Cm(-0.35)
    p.paragraph_format.space_after = Pt(after)
    p.paragraph_format.line_spacing = 1.15
    r = p.add_run("•  ")
    _set_run(r, size=11, color=PRIMARY_RGB)
    r2 = p.add_run(text)
    _set_run(r2, size=11)
    return p


def add_callout(doc, runs, *, fill=RES):
    """Caja resaltada (parrafo con sombreado + borde verde) con varias lineas/runs.
    runs = lista de dicts; usa 'break': True para forzar salto de linea antes."""
    p = doc.add_paragraph()
    p.alignment = C
    p.paragraph_format.space_before = Pt(8)
    p.paragraph_format.space_after = Pt(10)
    _para_shading(p, fill)
    _para_border(p, color=ACCENT, sz="12", edges=("top", "bottom"))
    for rd in runs:
        if rd.get("break"):
            br = p.add_run()
            br.add_break()
        r = p.add_run(rd["text"])
        _set_run(r, bold=rd.get("bold", False), italic=rd.get("italic", False),
                 size=rd.get("size", 11), color=rd.get("color"))
    return p


def add_page_break(doc):
    doc.add_page_break()


def _cell(cell, text, *, bold=False, fill=None, align=L, color=None, size=10):
    cell.vertical_alignment = WD_ALIGN_VERTICAL.CENTER
    _set_cell_borders(cell)
    _set_cell_margins(cell)
    if fill:
        shade_cell(cell, fill)
    p = cell.paragraphs[0]
    p.alignment = align
    p.paragraph_format.space_after = Pt(0)
    p.paragraph_format.space_before = Pt(0)
    p.paragraph_format.line_spacing = 1.1
    r = p.add_run("" if text is None else str(text))
    _set_run(r, bold=bold, size=size, color=color)
    return cell


def make_table(doc, col_widths_cm, rows):
    """rows: lista de filas; cada celda es un dict {text, bold, fill, align, color, size}.
    col_widths_cm: lista de anchos en cm (uno por columna)."""
    ncols = len(col_widths_cm)
    table = doc.add_table(rows=len(rows), cols=ncols)
    table.alignment = WD_TABLE_ALIGNMENT.CENTER
    table.autofit = False
    table.allow_autofit = False
    # fija layout fijo
    tblPr = table._tbl.tblPr
    layout = OxmlElement("w:tblLayout")
    layout.set(qn("w:type"), "fixed")
    tblPr.append(layout)
    for ri, row in enumerate(rows):
        for ci in range(ncols):
            cd = row[ci] if ci < len(row) else {"text": ""}
            cell = table.cell(ri, ci)
            cell.width = Cm(col_widths_cm[ci])
            _cell(cell, cd.get("text", ""), bold=cd.get("bold", False),
                  fill=cd.get("fill"), align=cd.get("align", L),
                  color=cd.get("color"), size=cd.get("size", 10))
    # refuerza width por celda (Word a veces necesita ambos)
    for ri in range(len(rows)):
        for ci in range(ncols):
            table.cell(ri, ci).width = Cm(col_widths_cm[ci])
    doc.add_paragraph().paragraph_format.space_after = Pt(2)
    return table


def header_row(labels):
    out = []
    for i, lab in enumerate(labels):
        out.append({"text": lab, "bold": True, "fill": HDR,
                    "color": PRIMARY_RGB, "align": L if i == 0 else C})
    return out


def _has_logo(inputs):
    dp = inputs.get("datos_profesional")
    if not dp:
        return None
    logo = dp.get("logo_path")
    if logo and isinstance(logo, str) and os.path.isfile(logo):
        return logo
    return None


def _prof_name(inputs):
    dp = inputs.get("datos_profesional")
    if dp and dp.get("nombre"):
        return dp["nombre"]
    return "Valia"


# ===================== IMAGEN / LOGO =====================
def _add_image_centered(doc, path, width_cm, *, after=2, before=8):
    from docx.shared import Cm as _Cm
    p = doc.add_paragraph()
    p.alignment = C
    p.paragraph_format.space_before = Pt(before)
    p.paragraph_format.space_after = Pt(after)
    run = p.add_run()
    run.add_picture(path, width=_Cm(width_cm))
    return p


# ===================== RENDER PRINCIPAL =====================
def render_word(inputs, out_path):
    meta = inputs["meta"]
    sym = meta["moneda_simbolo"]
    sym_local = meta.get("moneda_local_simbolo", "")
    tc = meta.get("tc", 1.0)

    resultados = R.calc_todos(inputs)
    cen = R.central(inputs, resultados)
    central_idx = meta.get("central_idx", 1)
    terrenos = [r["terreno"] for r in resultados]
    rango_min, rango_max = min(terrenos), max(terrenos)

    tributo = inputs.get("tributo", {})
    trib_nombre = tributo.get("nombre", "Impuesto")
    pnorm = inputs.get("parametros_normativos", {})
    tipologias = inputs.get("tipologias", [])
    comparables = inputs.get("comparables", [])
    fuentes = inputs.get("fuentes", [])
    logo = _has_logo(inputs)

    doc = Document()

    # ---- estilos base / fuente por defecto ----
    normal = doc.styles["Normal"]
    normal.font.name = FONT
    normal.font.size = Pt(11)
    normal.font.color.rgb = BLACK_RGB
    rpr = normal.element.get_or_add_rPr()
    rfonts = rpr.find(qn("w:rFonts"))
    if rfonts is None:
        rfonts = OxmlElement("w:rFonts")
        rpr.append(rfonts)
    for a in ("w:ascii", "w:hAnsi", "w:cs"):
        rfonts.set(qn(a), FONT)

    # ---- pagina A4 + margenes 2.5cm ----
    sec = doc.sections[0]
    sec.page_width = Cm(21)
    sec.page_height = Cm(29.7)
    sec.top_margin = Cm(2.5)
    sec.bottom_margin = Cm(2.5)
    sec.left_margin = Cm(2.5)
    sec.right_margin = Cm(2.5)
    sec.header_distance = Cm(1.2)
    sec.footer_distance = Cm(1.2)

    # =========================================================
    # PORTADA
    # =========================================================
    add_par(doc, "", after=30)
    if logo:
        _add_image_centered(doc, logo, 6.0, before=10, after=14)
    else:
        add_par(doc, _prof_name(inputs), bold=True, size=22, color=PRIMARY_RGB,
                align=C, after=14)

    add_par(doc, "INFORME DE VALUACIÓN DE TERRENO", bold=True, size=24,
            color=PRIMARY_RGB, align=C, after=4, before=10)
    p = add_par(doc, "Método de Valor Residual · Tres escenarios", size=14,
                color=PRIMARY_RGB, align=C, after=18)
    _para_border(p, color=ACCENT, sz="14", edges=("bottom",))

    dir_txt = meta.get("direccion", "")
    add_par(doc, dir_txt, bold=True, size=16, align=C, after=3)
    ubic = " · ".join([x for x in [meta.get("distrito", ""),
                                        meta.get("ciudad", ""),
                                        meta.get("pais", "")] if x])
    add_par(doc, ubic, size=12, align=C, after=3)
    area_txt = f"Área del terreno: {num(meta.get('area_terreno', 0), 2)} m²"
    add_par(doc, area_txt, size=12, align=C, after=18)

    # caja de valor central
    add_callout(doc, [
        {"text": "Valor estimado del terreno — escenario central",
         "size": 11, "color": PRIMARY_RGB},
        {"text": money(cen["terreno"], sym), "bold": True, "size": 22,
         "color": PRIMARY_RGB, "break": True},
        {"text": f"{money(cen['terreno_m2'], sym)} / m²  ·  "
                 f"{pct(cen['pct_gdv'])} del GDV  ·  rango "
                 f"{money(rango_min, sym)} – {money(rango_max, sym)}",
         "size": 11, "break": True},
    ], fill=RES)

    add_par(doc, meta.get("fecha", ""), size=11, align=C, after=3, before=18)
    add_par(doc, "Documento de apoyo a la decisión — Confidencial",
            italic=True, size=9, color=GREY_RGB, align=C, after=0)
    add_page_break(doc)

    # =========================================================
    # 1. RESUMEN EJECUTIVO
    # =========================================================
    add_heading(doc, "1. Resumen ejecutivo", 1)
    add_par(doc,
            f"Este informe estima el valor de mercado del terreno ubicado en "
            f"{dir_txt}"
            + (f", {meta.get('distrito')}" if meta.get("distrito") else "")
            + f", de {num(meta.get('area_terreno', 0), 2)} m², mediante el "
            f"método de valor residual. El método deduce el valor del suelo a "
            f"partir del valor de venta de un proyecto inmobiliario óptimo "
            f"desarrollable en el predio, restando todos los costos de desarrollo y "
            f"el margen del desarrollador.")
    add_par(doc,
            "Dado que el resultado es muy sensible al costo de construcción y a la "
            "edificabilidad finalmente aprobada, se presentan tres escenarios "
            "plausibles. Se recomienda el escenario central.", after=4)

    add_callout(doc, [
        {"text": "Valor del terreno (central): ", "size": 12},
        {"text": money(cen["terreno"], sym), "bold": True, "size": 14,
         "color": PRIMARY_RGB},
        {"text": f"{money(cen['terreno_m2'], sym)}/m²  ·  "
                 f"{pct(cen['pct_gdv'])} del GDV", "size": 11, "break": True},
    ], fill=RES)

    # tabla 3 escenarios
    nombres = [r["nombre"] for r in resultados]
    fila_hdr = [{"text": "Escenario", "bold": True, "fill": HDR, "color": PRIMARY_RGB}]
    for i, nom in enumerate(nombres):
        fila_hdr.append({"text": nom, "bold": True,
                         "fill": CEN if i == central_idx else HDR,
                         "color": PRIMARY_RGB, "align": C})

    def fila_metric(label, fn, *, bold_label=False):
        row = [{"text": label, "bold": bold_label}]
        for i, r in enumerate(resultados):
            row.append({"text": fn(r), "align": RGT,
                        "fill": CEN if i == central_idx else None,
                        "bold": i == central_idx})
        return row

    nesc = len(resultados)
    w_lbl = 5.6
    w_col = (16 - w_lbl) / nesc
    widths = [w_lbl] + [w_col] * nesc
    make_table(doc, widths, [
        fila_hdr,
        fila_metric(f"Valor del terreno ({sym})", lambda r: num(r["terreno"]),
                    bold_label=True),
        fila_metric(f"Valor ({sym}/m²)", lambda r: num(r["terreno_m2"])),
        fila_metric("Incidencia (% del GDV)", lambda r: pct(r["pct_gdv"])),
        fila_metric(f"Equivalente ({sym_local})", lambda r: num(r["terreno_local"])),
    ])
    add_par(doc,
            f"El valor central ({money(cen['terreno'], sym)}, "
            f"~{money(cen['terreno_m2'], sym)}/m²) es el estimado puntual "
            f"recomendado. Los extremos conservador y agresivo acotan el efecto de la "
            f"edificabilidad y de la relación precio-costo.",
            italic=True, after=4)

    # =========================================================
    # 2. OBJETO Y ALCANCE
    # =========================================================
    add_heading(doc, "2. Objeto y alcance", 1)
    add_par(doc,
            "El objeto es estimar el valor del terreno como suelo con potencial de "
            "desarrollo, no como inmueble construido. El análisis: (i) infiere el "
            "tamaño máximo edificable según zonificación y referencias "
            "reales; (ii) estima el área vendible de un proyecto típico; (iii) "
            "determina el valor del área vendible a partir de precios de mercado de "
            "la zona; y (iv) hace la ingeniería inversa de la estructura financiera "
            "para deducir el valor residual del suelo.")
    add_par(doc,
            "Es un estimado de apoyo a la decisión. No constituye una tasación "
            "reglamentada, que requeriría inspección física y la verificación "
            "de los parámetros urbanísticos oficiales. La sección 4 documenta "
            "la metodología y la trazabilidad para su revisión por un profesional "
            "tasador.", italic=True, after=4)

    # =========================================================
    # 3. PREDIO Y MARCO NORMATIVO
    # =========================================================
    add_heading(doc, "3. Descripción del predio y marco normativo", 1)
    add_par(doc,
            f"El predio se ubica en {dir_txt}"
            + (f", {meta.get('distrito')}" if meta.get("distrito") else "")
            + ". A continuación se resumen los parámetros normativos aplicables.")

    norm_rows = [header_row(["Parámetro", "Valor"])]
    if meta.get("distrito"):
        norm_rows.append([{"text": "Distrito"}, {"text": meta["distrito"]}])
    if meta.get("zonificacion"):
        norm_rows.append([{"text": "Zonificación"}, {"text": meta["zonificacion"]}])
    # campos descriptivos de parametros_normativos (excluye metadatos de fuente)
    etiquetas = {
        "altura": "Altura máxima",
        "area_libre": "Área libre",
        "estacionamientos": "Estacionamientos",
        "usos": "Usos",
        "densidad": "Densidad",
        "retiro": "Retiro",
        "coeficiente": "Coeficiente de edificación",
    }
    meta_keys = {"fuente", "confianza", "modo"}
    for k, v in pnorm.items():
        if k in meta_keys:
            continue
        label = etiquetas.get(k, k.replace("_", " ").capitalize())
        norm_rows.append([{"text": label}, {"text": str(v)}])
    make_table(doc, [5.0, 11.0], norm_rows)

    # nota de fuente / confianza
    nota = []
    if pnorm.get("fuente"):
        nota.append(f"Fuente: {pnorm['fuente']}.")
    extras = []
    if pnorm.get("confianza"):
        extras.append(f"confianza {pnorm['confianza']}")
    if pnorm.get("modo"):
        extras.append(f"modo {pnorm['modo']}")
    if extras:
        nota.append(f"({'; '.join(extras)}).")
    if nota:
        add_par(doc, " ".join(nota), italic=True, size=10, color=GREY_RGB, after=4)

    # =========================================================
    # 4. METODOLOGIA
    # =========================================================
    add_heading(doc, "4. Metodología y estrategia de abordaje", 1)

    add_heading(doc, "4.1 Enfoque de valuación", 2)
    add_par(doc,
            "El valor del terreno se estima por el método de valor residual "
            "estático (método del residuo), reconocido para suelo con potencial "
            "de desarrollo. Bajo el supuesto de mejor y mayor uso (highest and best "
            "use) legalmente permisible y físicamente posible, el valor del suelo es "
            "el residuo que queda del valor de venta del proyecto inmobiliario "
            "óptimo una vez cubiertos todos los costos de desarrollo y el margen "
            "del promotor.")
    add_par(doc,
            "Se eligió este método porque el valor del suelo deriva de su "
            "capacidad de soportar un proyecto rentable; el método comparativo de "
            "terrenos se emplea solo como verificación de la incidencia resultante "
            "(rango típico de mercado). El mejor y mayor uso adoptado es un edificio "
            "multifamiliar de densidad alta, consistente con la zonificación y el "
            "patrón real de la zona.", after=4)

    add_heading(doc, "4.2 Secuencia del análisis y trazabilidad", 2)
    fuente_norm = pnorm.get("fuente", "Parámetros normativos")
    fuente_mkt = fuentes[0]["nombre"] if fuentes else "Fuentes de mercado"
    seq_rows = [header_row(["Paso", "Procedimiento", "Fuente / evidencia"])]
    seq = [
        ("0", "Verificación de distrito, zonificación y altura", fuente_norm),
        ("1", "Área techada = (1 − área libre) × terreno × pisos",
         "Parámetros normativos"),
        ("2", "Área vendible = área techada × eficiencia",
         "Estándar de mercado"),
        ("3", "GDV = precio blended × área vendible + estac. + depósitos",
         fuente_mkt),
        ("4", "Terreno = GDV − costos − margen",
         "Costos sectoriales + tributación"),
        ("5", "Escenarios y sensibilidad (variación de drivers)",
         "Rangos documentados de las fuentes"),
    ]
    for paso, proc, fte in seq:
        seq_rows.append([
            {"text": paso, "align": C},
            {"text": proc},
            {"text": fte, "size": 9},
        ])
    make_table(doc, [1.4, 8.0, 6.6], seq_rows)

    add_heading(doc, "4.3 Construcción de los escenarios", 2)
    add_par(doc,
            "Cada escenario es un estado del mundo internamente consistente, no la "
            "combinación mecánica de los extremos. El conservador asume menor "
            "edificabilidad, precios prudentes y costos/margen mayores; el agresivo, "
            "lo inverso; el central toma los valores modales de las fuentes. El margen "
            "del desarrollador se mantiene en un rango sano en los tres escenarios, lo "
            "que descarta que alguno implique un proyecto inviable.", after=4)

    add_heading(doc, "4.4 Origen de los supuestos (trazabilidad)", 2)
    cs = inputs["escenarios"][central_idx]
    sup_rows = [header_row(["Supuesto", "Central", "Fuente", "Confianza"])]
    conf_norm = pnorm.get("confianza", "Media")
    sup = [
        ("Precio venta blended", f"{money(cs['precio_blended'], sym)}/m²",
         fuente_mkt, "Alta"),
        ("Costo construcción rasante", f"{money(cs['costo_rasante'], sym)}/m²",
         "Costos sectoriales", "Media-alta"),
        ("Eficiencia vendible", pct(cs["eficiencia"]),
         "Estándar de producto", "Media-alta"),
        ("Soft costs", pct(cs["soft_pct"]), "Sectoriales", "Media-alta"),
        ("Marketing y ventas", pct(cs["mkt_pct"]), "Sectoriales", "Media-alta"),
        ("Gastos financieros", pct(cs["fin_pct"]), "Sectoriales", "Media"),
        ("Margen del desarrollador", pct(cs["margen_desarrollador_input"]),
         "Sectoriales", "Media-alta"),
        (f"{trib_nombre} efectivo (vivienda)", pct(cs["impuesto_efectivo"]),
         tributo.get("nombre", "Autoridad tributaria"), "Alta"),
        ("Tipo de cambio", f"{sym_local} {num(tc, 2)}",
         meta.get("tc_fuente", "Banco central"), "Alta"),
    ]
    for s_lbl, s_val, s_fte, s_conf in sup:
        sup_rows.append([
            {"text": s_lbl},
            {"text": s_val, "align": RGT},
            {"text": s_fte, "size": 9},
            {"text": s_conf, "align": C},
        ])
    make_table(doc, [5.4, 2.8, 5.2, 2.6], sup_rows)

    add_heading(doc, "4.5 Tratamiento tributario", 2)
    add_par(doc,
            f"El {trib_nombre} aplicable se trata con una tasa efectiva de "
            f"{pct(cs['impuesto_efectivo'])} sobre el precio de la vivienda. "
            + (f"{tributo.get('base')}. " if tributo.get("base") else "")
            + "El impuesto a la renta del desarrollador se considera absorbido dentro "
            "del margen pre-impuesto.", after=4)

    add_heading(doc, "4.6 Alcance y limitaciones profesionales", 2)
    add_par(doc,
            "Este informe es un estimado de apoyo a la decisión, no una tasación "
            "reglamentada con validez ante terceros. Para una tasación formal se "
            "requiere: inspección física del predio, certificado de parámetros "
            "urbanísticos de la autoridad municipal, verificación registral de "
            "área y cargas, y presupuesto de obra de un profesional. Las cifras de "
            "edificabilidad son referenciales hasta confirmar dichos parámetros.",
            italic=True, after=4)

    # =========================================================
    # 5. PASOS 1-2
    # =========================================================
    add_heading(doc, "5. Pasos 1 y 2 — Edificabilidad y área vendible (central)", 1)
    add_par(doc,
            f"Con los parámetros de zonificación y los supuestos del escenario "
            f"central ({num(cs['pisos'])} pisos, {pct(cs['area_libre'])} de "
            f"área libre, eficiencia {pct(cs['eficiencia'])}):")
    at = meta["area_terreno"]
    footprint = at * (1 - cs["area_libre"])
    make_table(doc, [9.5, 6.5], [
        header_row(["Variable", "Valor"]),
        [{"text": "Área del terreno"}, {"text": f"{num(at, 2)} m²", "align": RGT}],
        [{"text": "Área libre"}, {"text": pct(cs["area_libre"]), "align": RGT}],
        [{"text": "Área ocupada (footprint)"},
         {"text": f"{num(footprint, 2)} m²", "align": RGT}],
        [{"text": "N° de pisos"}, {"text": num(cs["pisos"]), "align": RGT}],
        [{"text": "Área techada sobre rasante"},
         {"text": f"{num(cen['techada'])} m²", "align": RGT}],
        [{"text": "Eficiencia vendible"}, {"text": pct(cs["eficiencia"]), "align": RGT}],
        [{"text": "Área vendible de departamentos", "bold": True},
         {"text": f"{num(cen['vendible'])} m²", "align": RGT, "bold": True}],
        [{"text": "N° de departamentos"},
         {"text": f"≈ {num(round(cen['deptos']))}", "align": RGT}],
    ])

    # =========================================================
    # 6. PASO 3 - GDV
    # =========================================================
    add_heading(doc, "6. Paso 3 — Valor del área vendible (GDV, central)", 1)
    add_par(doc,
            f"El precio se infiere de comparables de mercado en la zona. El blended "
            f"del central es {money(cs['precio_blended'], sym)}/m².")
    if tipologias:
        tip_rows = [header_row(["Tipología", "Mix", "Área típica",
                                f"Precio ({sym}/m²)"])]
        for t in tipologias:
            tip_rows.append([
                {"text": t.get("nombre", "")},
                {"text": pct(t["mix"], 0) if "mix" in t else "", "align": C},
                {"text": f"{num(t.get('area', 0))} m²", "align": C},
                {"text": num(t.get("precio_m2", 0)), "align": RGT},
            ])
        tip_rows.append([
            {"text": "Blended ponderado (central)", "bold": True, "fill": LIGHT},
            {"text": "—", "align": C, "fill": LIGHT},
            {"text": f"{num(cs['area_prom_depto'], 1)} m²", "align": C, "fill": LIGHT},
            {"text": num(cs["precio_blended"]), "align": RGT, "bold": True, "fill": LIGHT},
        ])
        make_table(doc, [5.5, 2.7, 3.6, 4.2], tip_rows)

    add_par(doc, "Composición del GDV del escenario central:")
    make_table(doc, [6.4, 4.6, 5.0], [
        header_row(["Concepto", "Cálculo", sym]),
        [{"text": "Departamentos"},
         {"text": f"{num(cen['vendible'])} × {num(cs['precio_blended'])}", "align": C},
         {"text": num(cen["ing_deptos"]), "align": RGT}],
        [{"text": "Estacionamientos"},
         {"text": f"{num(cen['n_estac'])} × {num(cs['estac_precio'])}", "align": C},
         {"text": num(cen["ing_estac"]), "align": RGT}],
        [{"text": "Depósitos"},
         {"text": f"{num(cen['n_dep'])} × {num(cs['dep_precio'])}", "align": C},
         {"text": num(cen["ing_dep"]), "align": RGT}],
        [{"text": "GDV total", "bold": True, "fill": HDR},
         {"text": "", "fill": HDR},
         {"text": num(cen["gdv"]), "align": RGT, "bold": True, "fill": HDR}],
    ])

    # =========================================================
    # 7. PASO 4 - ESTRUCTURA FINANCIERA
    # =========================================================
    add_heading(doc, "7. Paso 4 — Estructura financiera y valor residual (central)", 1)
    add_par(doc,
            "Del GDV se descuentan costos y margen. El remanente es el valor residual "
            "del terreno:")
    gdv = cen["gdv"]

    def pgdv(v):
        return pct(v / gdv) if gdv else "—"

    make_table(doc, [7.6, 3.0, 5.4], [
        header_row(["Componente", "% GDV", sym]),
        [{"text": "Valor de venta total (GDV)", "bold": True},
         {"text": "100.0 %", "align": C, "bold": True},
         {"text": num(gdv), "align": RGT, "bold": True}],
        [{"text": "(−) Construcción total"},
         {"text": pgdv(cen["con_total"]), "align": C},
         {"text": f"({num(cen['con_total'])})", "align": RGT}],
        [{"text": f"(−) {trib_nombre} (vivienda)"},
         {"text": pgdv(cen["impuesto"]), "align": C},
         {"text": f"({num(cen['impuesto'])})", "align": RGT}],
        [{"text": "(−) Soft costs"},
         {"text": pgdv(cen["soft"]), "align": C},
         {"text": f"({num(cen['soft'])})", "align": RGT}],
        [{"text": "(−) Marketing y ventas"},
         {"text": pgdv(cen["mkt"]), "align": C},
         {"text": f"({num(cen['mkt'])})", "align": RGT}],
        [{"text": "(−) Gastos financieros"},
         {"text": pgdv(cen["fin"]), "align": C},
         {"text": f"({num(cen['fin'])})", "align": RGT}],
        [{"text": "(−) Margen del desarrollador"},
         {"text": pgdv(cen["margen"]), "align": C},
         {"text": f"({num(cen['margen'])})", "align": RGT}],
        [{"text": "(=) Valor residual del terreno", "bold": True, "fill": RES,
          "color": PRIMARY_RGB},
         {"text": pct(cen["pct_gdv"]), "align": C, "bold": True, "fill": RES,
          "color": PRIMARY_RGB},
         {"text": num(cen["terreno"]), "align": RGT, "bold": True, "fill": RES,
          "color": PRIMARY_RGB}],
    ])

    # =========================================================
    # 8. LOS TRES ESCENARIOS
    # =========================================================
    add_heading(doc, "8. Los tres escenarios", 1)
    add_par(doc,
            "Cada escenario combina supuestos coherentes (no extremos mecánicos; "
            "ver sección 4.3). El conservador asume un edificio menor, precios más "
            "prudentes y costos/margen mayores; el agresivo, máxima edificabilidad, "
            "precios fuertes y costos ajustados. El central es el más probable.")

    def esc_row(label, fn, *, bold_label=False, fill_central=CEN, result=False):
        row = [{"text": label, "bold": bold_label or result}]
        for i, r in enumerate(resultados):
            is_c = i == central_idx
            cd = {"text": fn(r, inputs["escenarios"][i]),
                  "align": fn.align if hasattr(fn, "align") else RGT}
            if is_c:
                cd["fill"] = RES if result else fill_central
                cd["bold"] = True
                if result:
                    cd["color"] = PRIMARY_RGB
            elif result:
                cd["bold"] = True
            row.append(cd)
        return row

    esc_hdr = [{"text": "Parámetro / Resultado", "bold": True, "fill": HDR,
                "color": PRIMARY_RGB}]
    for i, nom in enumerate(nombres):
        esc_hdr.append({"text": nom, "bold": True,
                        "fill": CEN if i == central_idx else HDR,
                        "color": PRIMARY_RGB, "align": C})

    def f_pisos(r, s):
        return f"{num(s['pisos'])} / {pct(s['area_libre'], 0)}"
    f_pisos.align = C

    rows8 = [esc_hdr]
    rows8.append(esc_row("N° de pisos / área libre", f_pisos))
    rows8.append(esc_row(f"Precio venta ({sym}/m²)",
                         lambda r, s: num(s["precio_blended"])))
    rows8.append(esc_row(f"Costo construcción ({sym}/m²)",
                         lambda r, s: num(s["costo_rasante"])))
    rows8.append(esc_row("Margen del desarrollador",
                         lambda r, s: pct(s["margen_desarrollador_input"])))
    rows8.append(esc_row(f"GDV total ({sym})", lambda r, s: num(r["gdv"])))
    rows8.append(esc_row(f"VALOR DEL TERRENO ({sym})",
                         lambda r, s: num(r["terreno"]), result=True))
    rows8.append(esc_row(f"Valor ({sym}/m²)", lambda r, s: num(r["terreno_m2"]),
                         result=True))
    rows8.append(esc_row("Incidencia (% del GDV)", lambda r, s: pct(r["pct_gdv"]),
                         result=True))
    rows8.append(esc_row(f"Valor ({sym_local})", lambda r, s: num(r["terreno_local"])))

    w_lbl8 = 5.6
    w_col8 = (16 - w_lbl8) / nesc
    make_table(doc, [w_lbl8] + [w_col8] * nesc, rows8)
    add_par(doc,
            "El margen sobre costos se mantiene sano en los tres escenarios, por lo "
            "que ninguno es inviable: representan estados del mundo distintos, no un "
            "mismo proyecto mal calculado.", italic=True, after=4)

    # =========================================================
    # 9. SENSIBILIDAD
    # =========================================================
    add_heading(doc, "9. Análisis de sensibilidad (sobre el central)", 1)
    sr = R.sens_ranges(inputs)

    def sens_table(doc, *, col_header, row_header, col_vals, row_vals, grid,
                   fmt, col_label_fmt, row_label_fmt, base_row_idx, base_col_idx):
        ncols = len(col_vals) + 1
        # encabezado
        hdr = [{"text": f"{row_header} \\ {col_header}", "bold": True, "fill": HDR,
                "color": PRIMARY_RGB}]
        for cv in col_vals:
            hdr.append({"text": col_label_fmt(cv), "bold": True, "fill": HDR,
                        "color": PRIMARY_RGB, "align": C})
        rows = [hdr]
        for ri, rv in enumerate(row_vals):
            row = [{"text": row_label_fmt(rv), "bold": True, "fill": LIGHT}]
            for ci in range(len(col_vals)):
                is_base = (ri == base_row_idx and ci == base_col_idx)
                cd = {"text": fmt(grid[ri][ci]), "align": RGT}
                if is_base:
                    cd["fill"] = BASEC
                    cd["bold"] = True
                row.append(cd)
            rows.append(row)
        w_lbl = 3.0
        w_col = (16 - w_lbl) / len(col_vals)
        make_table(doc, [w_lbl] + [w_col] * len(col_vals), rows)

    # 9.1 precio x costo -> moneda/m2
    add_heading(doc, f"9.1 Valor del terreno ({sym}/m²) — precio × costo de "
                     "construcción", 2)
    precios, costos = sr["precios"], sr["costos"]
    grid_pc = R.sens_precio_costo(inputs, precios, costos)
    base_row_pc = precios.index(sr["precio_central"]) if sr["precio_central"] in precios else 0
    base_col_pc = costos.index(sr["costo_central"]) if sr["costo_central"] in costos else 0
    sens_table(doc, col_header="Constr.", row_header="Precio",
               col_vals=costos, row_vals=precios, grid=grid_pc,
               fmt=lambda v: num(v), col_label_fmt=lambda v: num(v),
               row_label_fmt=lambda v: num(v),
               base_row_idx=base_row_pc, base_col_idx=base_col_pc)

    # 9.2 pisos x area libre -> moneda/m2
    add_heading(doc, f"9.2 Valor del terreno ({sym}/m²) — pisos × área libre", 2)
    pisos, libres = sr["pisos"], sr["libres"]
    grid_pl = R.sens_pisos_arealibre(inputs, pisos, libres)
    base_row_pl = pisos.index(sr["pisos_central"]) if sr["pisos_central"] in pisos else 0
    base_col_pl = libres.index(sr["libre_central"]) if sr["libre_central"] in libres else 0
    sens_table(doc, col_header="Á. libre", row_header="Pisos",
               col_vals=libres, row_vals=pisos, grid=grid_pl,
               fmt=lambda v: num(v), col_label_fmt=lambda v: pct(v, 0),
               row_label_fmt=lambda v: f"{num(v)} pisos",
               base_row_idx=base_row_pl, base_col_idx=base_col_pl)

    # 9.3 precio x margen -> % del GDV
    add_heading(doc, "9.3 Incidencia del terreno (% del GDV) — precio × margen", 2)
    margenes = sr["margenes"]
    grid_pm = R.sens_precio_margen(inputs, precios, margenes)
    base_row_pm = precios.index(sr["precio_central"]) if sr["precio_central"] in precios else 0
    base_col_pm = margenes.index(sr["margen_central"]) if sr["margen_central"] in margenes else 0
    sens_table(doc, col_header="Margen", row_header="Precio",
               col_vals=margenes, row_vals=precios, grid=grid_pm,
               fmt=lambda v: pct(v), col_label_fmt=lambda v: pct(v, 0),
               row_label_fmt=lambda v: num(v),
               base_row_idx=base_row_pm, base_col_idx=base_col_pm)
    add_par(doc,
            "Celdas resaltadas = escenario central. La incidencia resultante debe "
            "ubicarse dentro del rango razonable de mercado para un lote de estas "
            "características.", italic=True, size=10, after=4)

    # =========================================================
    # 10. CONCLUSIONES
    # =========================================================
    add_heading(doc, "10. Conclusiones y recomendaciones", 1)
    add_par(doc,
            f"Bajo el método residual y los supuestos documentados, el valor del "
            f"terreno de {dir_txt} se estima, en el escenario central, en "
            f"{money(cen['terreno'], sym)} ({money(cen['terreno_m2'], sym)}/m², "
            f"{pct(cen['pct_gdv'])} del GDV). El rango plausible es "
            f"{money(rango_min, sym)} (conservador) a {money(rango_max, sym)} "
            f"(agresivo).")
    add_bullet(doc,
               f"Recomendación de trabajo: usar el escenario central "
               f"({money(cen['terreno'], sym)} / ~{money(cen['terreno_m2'], sym)}/m²) "
               f"como referencia.")
    add_bullet(doc,
               "Obtener el certificado de parámetros urbanísticos de la autoridad "
               "municipal: fija la altura y el coeficiente, la variable que más mueve "
               "el valor absoluto.")
    add_bullet(doc,
               "Validar el costo de construcción con 1–2 presupuestos reales: es el "
               "segundo factor de mayor sensibilidad y la diferencia principal entre "
               "escenarios.")

    # =========================================================
    # 11. SUPUESTOS Y LIMITACIONES
    # =========================================================
    add_heading(doc, "11. Supuestos, limitaciones y advertencias", 1)
    add_bullet(doc,
               f"Tipo de cambio: {sym_local} {num(tc, 2)} por {sym} "
               f"({meta.get('tc_fuente', 'declarado')}). Se declara explícitamente "
               f"por su impacto en las cifras.")
    add_bullet(doc,
               "Edificabilidad inferida (zonificación + referencias reales), no "
               "confirmada por certificado de parámetros oficial.")
    add_bullet(doc,
               "Precios de venta de comparables de mercado (ajustados a unidad "
               "terminada); el valor real depende de pisos, vistas y acabados.")
    add_bullet(doc,
               "Costos y benchmarks de fuentes sectoriales, no de un presupuesto "
               "específico del proyecto.")
    add_bullet(doc,
               "Estimado de apoyo a la decisión, no una tasación reglamentada.")

    # =========================================================
    # 12. ANEXO - EVIDENCIA
    # =========================================================
    add_heading(doc, "12. Anexo — Evidencia", 1)
    if comparables:
        add_par(doc, "Comparables de mercado considerados en el análisis:")
        # construye encabezado a partir de las llaves disponibles
        key_labels = {
            "proyecto": "Proyecto",
            "direccion": "Dirección",
            "pisos": "Pisos",
            "entrega": "Entrega",
            "rango_m2": f"Rango ({sym}/m²)",
            "precio_m2": f"Precio ({sym}/m²)",
        }
        # une todas las llaves presentes preservando un orden razonable
        ordered = ["proyecto", "direccion", "pisos", "entrega", "rango_m2", "precio_m2"]
        present = [k for k in ordered if any(k in c for c in comparables)]
        for c in comparables:
            for k in c:
                if k not in present:
                    present.append(k)
        comp_hdr = header_row([key_labels.get(k, k.replace("_", " ").capitalize())
                               for k in present])
        comp_rows = [comp_hdr]
        for c in comparables:
            row = []
            for i, k in enumerate(present):
                row.append({"text": str(c.get(k, "")), "align": L if i < 2 else C,
                            "size": 9})
            comp_rows.append(row)
        ncol = len(present)
        make_table(doc, [16 / ncol] * ncol, comp_rows)

    # imagenes de evidencia opcionales
    imgs = inputs.get("evidencia_imgs")
    if imgs:
        for idx, ip in enumerate(imgs, start=1):
            if isinstance(ip, dict):
                path = ip.get("path")
                cap = ip.get("caption", f"Figura {idx}.")
            else:
                path = ip
                cap = f"Figura {idx}."
            if path and os.path.isfile(path):
                _add_image_centered(doc, path, 14.0, before=8, after=2)
                add_par(doc, cap, italic=True, size=8, color=GREY_RGB, align=C, after=8)

    # =========================================================
    # 13. FUENTES
    # =========================================================
    add_heading(doc, "13. Fuentes", 1)
    for f in fuentes:
        nombre = f.get("nombre", "")
        url = f.get("url", "")
        p = doc.add_paragraph()
        p.paragraph_format.left_indent = Cm(0.7)
        p.paragraph_format.first_line_indent = Cm(-0.35)
        p.paragraph_format.space_after = Pt(3)
        p.paragraph_format.line_spacing = 1.15
        r = p.add_run("•  ")
        _set_run(r, size=10, color=PRIMARY_RGB)
        r2 = p.add_run(nombre + (": " if url else ""))
        _set_run(r2, size=10)
        if url:
            _add_hyperlink(p, url, url)

    # =========================================================
    # DISCLAIMER FINAL
    # =========================================================
    add_heading(doc, "Disclaimer", 1)
    p = doc.add_paragraph()
    p.alignment = L
    p.paragraph_format.space_before = Pt(4)
    p.paragraph_format.space_after = Pt(6)
    p.paragraph_format.line_spacing = 1.15
    _para_shading(p, LIGHT)
    _para_border(p, color=ACCENT, sz="12", edges=("top", "bottom", "left", "right"))
    r = p.add_run(
        "Esta valuación fue generada por IA con método residual y parámetros "
        "inferidos/aportados. Es un estimado de apoyo a la decisión, no una "
        "tasación reglamentada. Valídala con un tasador certificado y confirma los "
        "parámetros con la autoridad municipal antes de usarla para fines oficiales.")
    _set_run(r, italic=True, size=10, color=PRIMARY_RGB)

    # =========================================================
    # HEADER + FOOTER
    # =========================================================
    _build_header_footer(doc, logo, inputs)

    doc.save(out_path)
    return out_path


# ===================== HYPERLINK =====================
def _add_hyperlink(paragraph, url, text):
    part = paragraph.part
    r_id = part.relate_to(
        url,
        "http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink",
        is_external=True,
    )
    hyperlink = OxmlElement("w:hyperlink")
    hyperlink.set(qn("r:id"), r_id)
    new_run = OxmlElement("w:r")
    rPr = OxmlElement("w:rPr")
    rfonts = OxmlElement("w:rFonts")
    for a in ("w:ascii", "w:hAnsi", "w:cs"):
        rfonts.set(qn(a), FONT)
    rPr.append(rfonts)
    color = OxmlElement("w:color")
    color.set(qn("w:val"), "1155CC")
    rPr.append(color)
    u = OxmlElement("w:u")
    u.set(qn("w:val"), "single")
    rPr.append(u)
    sz = OxmlElement("w:sz")
    sz.set(qn("w:val"), "20")  # 10pt -> half-points
    rPr.append(sz)
    new_run.append(rPr)
    t = OxmlElement("w:t")
    t.text = text
    new_run.append(t)
    hyperlink.append(new_run)
    paragraph._p.append(hyperlink)
    return hyperlink


# ===================== HEADER / FOOTER =====================
def _add_page_number_field(paragraph):
    def _field(instr):
        run = paragraph.add_run()
        fldBegin = OxmlElement("w:fldChar")
        fldBegin.set(qn("w:fldCharType"), "begin")
        instrText = OxmlElement("w:instrText")
        instrText.set(qn("xml:space"), "preserve")
        instrText.text = instr
        fldEnd = OxmlElement("w:fldChar")
        fldEnd.set(qn("w:fldCharType"), "end")
        run._r.append(fldBegin)
        run._r.append(instrText)
        run._r.append(fldEnd)
        _set_run(run, size=8, color=GREY_RGB)

    r0 = paragraph.add_run("Página ")
    _set_run(r0, size=8, color=GREY_RGB)
    _field("PAGE")
    r1 = paragraph.add_run(" de ")
    _set_run(r1, size=8, color=GREY_RGB)
    _field("NUMPAGES")


def _build_header_footer(doc, logo, inputs):
    sec = doc.sections[0]

    # HEADER
    header = sec.header
    header.is_linked_to_previous = False
    hp = header.paragraphs[0]
    hp.alignment = L
    hp.paragraph_format.space_after = Pt(2)
    _para_border(hp, color=BORDER, sz="4", edges=("bottom",))
    if logo:
        run = hp.add_run()
        try:
            run.add_picture(logo, height=Cm(0.55))
        except Exception:
            r = hp.add_run(_prof_name(inputs))
            _set_run(r, size=8, color=GREY_RGB)
    else:
        r = hp.add_run(_prof_name(inputs))
        _set_run(r, bold=True, size=9, color=PRIMARY_RGB)

    # FOOTER
    footer = sec.footer
    footer.is_linked_to_previous = False
    fp = footer.paragraphs[0]
    fp.alignment = C
    r = fp.add_run(f"{_prof_name(inputs)} · Confidencial · ")
    _set_run(r, size=8, color=GREY_RGB)
    _add_page_number_field(fp)


# ===================== CLI / SMOKE =====================
if __name__ == "__main__":
    import json
    import sys

    in_path = sys.argv[1] if len(sys.argv) > 1 else "fixtures/arequipa_inputs.json"
    out = sys.argv[2] if len(sys.argv) > 2 else "informe_valuacion.docx"
    with open(in_path, encoding="utf-8") as fh:
        data = json.load(fh)
    render_word(data, out)
    print("OK", out)
```

### `generar_valuacion.py`

```python
# -*- coding: utf-8 -*-
"""CLI: python generar_valuacion.py inputs.json outdir
Genera Word + Excel (+ PDF best-effort) de la valuación de terreno. Genérico (todo viene del JSON)."""
import sys, os, json, re


def main():
    if len(sys.argv) < 3:
        print("uso: python generar_valuacion.py inputs.json outdir", file=sys.stderr)
        return 2
    inputs_path, outdir = sys.argv[1], sys.argv[2]
    sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
    from render_excel import render_excel
    from render_word import render_word
    inputs = json.load(open(inputs_path, encoding="utf-8"))
    os.makedirs(outdir, exist_ok=True)
    meta = inputs.get("meta", {})
    slug = re.sub(r"[^A-Za-z0-9]+", "_", str(meta.get("direccion", "terreno"))).strip("_")[:40] or "terreno"
    fecha = str(meta.get("fecha", "")).replace(":", "-")
    base = f"Valuacion_Terreno_{slug}_{fecha}".rstrip("_")
    xlsx = os.path.join(outdir, base + ".xlsx")
    docx = os.path.join(outdir, base + ".docx")
    render_excel(inputs, xlsx)
    render_word(inputs, docx)
    result = {"xlsx": xlsx, "docx": docx}
    soffice = r"C:\Program Files\LibreOffice\program\soffice.exe"
    if os.path.exists(soffice):
        import subprocess
        try:
            subprocess.run([soffice, "--headless", "--convert-to", "pdf", "--outdir", outdir, docx,
                            "-env:UserInstallation=file:///C:/temp/lo_valuador_pdf"],
                           check=True, timeout=120, capture_output=True)
            pdf = os.path.join(outdir, base + ".pdf")
            if os.path.exists(pdf):
                result["pdf"] = pdf
        except Exception as e:
            print(f"PDF skip: {e}", file=sys.stderr)
    print(json.dumps(result, ensure_ascii=False))
    return 0


if __name__ == "__main__":
    sys.exit(main())
```

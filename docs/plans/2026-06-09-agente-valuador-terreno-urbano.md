# agente-valuador-terreno-urbano — Plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Construir el 7º agente de la colección — un slash command que valúa terrenos urbanos por método residual, adaptable a país, y publicarlo en el repo (con el README/descripción actualizados a 7 agentes).

**Architecture:** Se desarrolla y testea primero un **generador Python determinista** (`generar_valuacion.py`: math del residual + render Word/Excel/PDF) contra el caso real validado Av. Arequipa 1110. Una vez verde, su código se **embebe** en el agente `.md` (junto con el seed `knowledge-peru.md` y la lógica conversacional del flujo). El `.md` sigue `docs/template-canonico.md`. Finalmente se bumpea el README a 7 agentes.

**Tech Stack:** Python 3.10+ (`openpyxl`, `python-docx`, `pytest`), LibreOffice (recalc Excel + PDF), Markdown (el agente). NOTA: se usa `python-docx` (no `docx-js`/Node) por portabilidad — el template del curso estandariza Python puro.

**Spec:** `docs/specs/2026-06-09-agente-valuador-terreno-urbano-design.md`

**Referencia de generadores validados (a portar):** `C:\Users\carlo\Charliecode\valuacion-arequipa-1110\build_xlsx.py` (Excel 3-escenarios, ya correcto) y `build_docx.js` (Word en docx-js, **a portar a python-docx**).

> **Nota de ejecución (Windows):** todos los comandos de shell de este plan son estilo bash (`mkdir -p`, `&&`, `2>/dev/null`). Ejecutarlos con la **herramienta Bash** (git-bash), NO con PowerShell, donde fallarían. Las operaciones git remotas que se cuelguen en sandbox se corren con sandbox desactivado, como en el spec. Para commitear el spec/plan, verificar que `docs/` no esté en `.gitignore`.

---

## File Structure

Desarrollo (carpeta de trabajo `C:\Users\carlo\Charliecode\valuador-dev\`, NO se commitea):
- `valuador-dev/generar_valuacion.py` — generador CLI: `python generar_valuacion.py inputs.json outdir`. Contiene: math del residual (funciones puras), render Excel (openpyxl), render Word (python-docx), orquestación + PDF.
- `valuador-dev/residual.py` — funciones puras del modelo (math), separadas para testear sin I/O.
- `valuador-dev/tests/test_residual.py` — unit tests de la math + regresión Av. Arequipa.
- `valuador-dev/tests/test_outputs.py` — integración: genera Excel/Word, recalc, cero errores, valores de regresión.
- `valuador-dev/fixtures/arequipa_inputs.json` — el caso de regresión.

Entregables al repo:
- `commands/agente-valuador-terreno-urbano.md` — el agente. Contiene embebidos: el generador (código verificado), el seed `knowledge-peru.md`, y toda la lógica del flujo/wizard/modos.
- `README.md` (modificar) — bump a 7 agentes.

> **Decisión de empaquetado:** el seed de Perú y el generador viven **embebidos como bloques en el `.md`** (single-file self-install: el instalador solo copia `commands/*.md`). El agente los materializa en disco en la primera corrida (`~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/`). Fuente única de verdad = el `.md`; durante el desarrollo se trabajan como archivos sueltos y al final se pegan al `.md`.

---

## Task 1: Scaffolding y fixture de regresión

**Files:**
- Create: `valuador-dev/fixtures/arequipa_inputs.json`
- Create: `valuador-dev/tests/__init__.py` (vacío)

- [ ] **Step 1: Crear carpeta de trabajo y verificar deps**

Run:
```bash
mkdir -p "C:/Users/carlo/Charliecode/valuador-dev/tests" "C:/Users/carlo/Charliecode/valuador-dev/fixtures"
python -c "import openpyxl, docx, pytest" 2>/dev/null && echo OK || pip install openpyxl python-docx pytest
```
Expected: `OK` (o instala y luego OK).

- [ ] **Step 2: Escribir el fixture de regresión** (`arequipa_inputs.json`)

Contiene los 3 escenarios validados. Central (regresión dura):
```json
{
  "meta": {"pais": "Perú", "moneda": "USD", "tc": 3.42, "direccion": "Av. Arequipa 1110",
           "distrito": "Cercado de Lima (Santa Beatriz)", "fecha": "2026-06-09",
           "zonificacion": "CM — Comercio Metropolitano", "area_terreno": 1204.45},
  "escenarios": [
    {"nombre": "Conservador", "pisos": 18, "area_libre": 0.45, "eficiencia": 0.82, "precio_blended": 2380,
     "area_prom_depto": 52.4, "estac_ratio": 0.5, "estac_precio": 14000, "dep_ratio": 0.5, "dep_precio": 1900,
     "costo_rasante": 950, "area_cajon": 30, "costo_sotano": 640, "soft_pct": 0.05, "mkt_pct": 0.035,
     "fin_pct": 0.025, "margen_desarrollador_input": 0.185, "igv_efectivo": 0.09},
    {"nombre": "Central", "pisos": 20, "area_libre": 0.40, "eficiencia": 0.82, "precio_blended": 2430,
     "area_prom_depto": 52.4, "estac_ratio": 0.5, "estac_precio": 15000, "dep_ratio": 0.5, "dep_precio": 2000,
     "costo_rasante": 880, "area_cajon": 30, "costo_sotano": 600, "soft_pct": 0.05, "mkt_pct": 0.035,
     "fin_pct": 0.02, "margen_desarrollador_input": 0.18, "igv_efectivo": 0.09},
    {"nombre": "Agresivo", "pisos": 22, "area_libre": 0.37, "eficiencia": 0.83, "precio_blended": 2500,
     "area_prom_depto": 52.4, "estac_ratio": 0.6, "estac_precio": 16000, "dep_ratio": 0.5, "dep_precio": 2100,
     "costo_rasante": 830, "area_cajon": 30, "costo_sotano": 580, "soft_pct": 0.045, "mkt_pct": 0.035,
     "fin_pct": 0.015, "margen_desarrollador_input": 0.17, "igv_efectivo": 0.09}
  ],
  "central_idx": 1,
  "tributo": {"nombre": "IGV", "tasa_efectiva_vivienda": 0.09, "base": "18% sobre 50% gravado (construcción); >35 UIT no exonerado"},
  "comparables": [], "fuentes": [], "datos_profesional": null
}
```

- [ ] **Step 3: Commit** (dev no se commitea al repo; pero inicializa git local opcional). Saltar commit del repo en esta task.

---

## Task 2: Math del residual (TDD, funciones puras)

**Files:**
- Create: `valuador-dev/residual.py`
- Test: `valuador-dev/tests/test_residual.py`

- [ ] **Step 1: Escribir el test que falla** (`test_residual.py`)

```python
import json, math, os
from residual import calc_escenario, calc_todos

HERE = os.path.dirname(__file__)
INPUTS = json.load(open(os.path.join(HERE, "..", "fixtures", "arequipa_inputs.json"), encoding="utf-8"))

def test_central_regresion():
    c = calc_escenario(INPUTS["escenarios"][1], INPUTS["meta"]["area_terreno"], INPUTS["meta"]["tc"])
    assert round(c["gdv"]) == 30720845
    assert round(c["terreno"]) == 4620426
    assert abs(c["pct_gdv"] - 0.1504) < 0.0005
    assert round(c["terreno_m2"]) == 3836

def test_tres_escenarios():
    r = calc_todos(INPUTS)
    terrenos = [round(e["terreno"]) for e in r]
    assert terrenos == [2240687, 4620426, 7793630]
    # margen sobre costos sano en los tres (>= 0.20)
    assert all(e["margen_sobre_costos_check"] >= 0.20 for e in r)

def test_pct_es_output_no_input():
    # el % de terreno NUNCA está en los inputs; sale del cálculo
    assert "pct_gdv" not in INPUTS["escenarios"][1]
```

- [ ] **Step 2: Correr y verificar que falla**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_residual.py -v`
Expected: FAIL (`ModuleNotFoundError: residual` o función no definida).

- [ ] **Step 3: Implementar `residual.py`** (mínimo para pasar)

```python
def calc_escenario(s, area_terreno, tc):
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
    igv = s["igv_efectivo"] * ing_deptos
    soft = s["soft_pct"] * gdv
    mkt = s["mkt_pct"] * gdv
    fin = s["fin_pct"] * gdv
    margen = s["margen_desarrollador_input"] * gdv
    egresos = con_total + igv + soft + mkt + fin + margen
    terreno = gdv - egresos
    return {
        "nombre": s["nombre"], "techada": techada, "vendible": vendible, "deptos": deptos,
        "ing_deptos": ing_deptos, "n_estac": n_estac, "ing_estac": ing_estac, "n_dep": n_dep,
        "ing_dep": ing_dep, "gdv": gdv, "sotano_area": sotano_area, "con_rasante": con_rasante,
        "con_sotano": con_sotano, "con_total": con_total, "igv": igv, "soft": soft, "mkt": mkt,
        "fin": fin, "margen": margen, "egresos": egresos, "terreno": terreno,
        "pct_gdv": terreno / gdv, "terreno_m2": terreno / area_terreno,
        "terreno_local": terreno * tc, "terreno_m2_local": (terreno / area_terreno) * tc,
        "margen_sobre_costos_check": margen / (gdv - margen),
    }

def calc_todos(inputs):
    at = inputs["meta"]["area_terreno"]; tc = inputs["meta"]["tc"]
    return [calc_escenario(s, at, tc) for s in inputs["escenarios"]]
```

- [ ] **Step 4: Correr y verificar que pasa**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_residual.py -v`
Expected: 3 passed.

- [ ] **Step 5: Commit (git local dev)**

```bash
cd C:/Users/carlo/Charliecode/valuador-dev && git init -q 2>/dev/null; git add -A && git commit -q -m "feat: residual math + regresión Av. Arequipa"
```

---

## Task 3: Render Excel (openpyxl, 3 escenarios + sensibilidad)

**Files:**
- Create: `valuador-dev/render_excel.py`
- Test: `valuador-dev/tests/test_outputs.py` (parte Excel)
- Reference: portar la estructura validada de `valuacion-arequipa-1110/build_xlsx.py` (hojas Portada · Resumen · Metodología · Escenarios · Sensibilidad · Comparables · Fuentes; fórmulas vivas; colores Valia #415262/#30E190; Lato).

- [ ] **Step 1: Escribir el test que falla** (`test_outputs.py`)

```python
import json, os, subprocess, sys
from openpyxl import load_workbook
HERE = os.path.dirname(__file__)
ROOT = os.path.join(HERE, "..")
INPUTS = os.path.join(ROOT, "fixtures", "arequipa_inputs.json")

def _recalc(path):
    # LibreOffice convert para recalcular fórmulas openpyxl (Windows-safe, sandbox off)
    out = os.path.join(os.path.dirname(path), "recalc")
    os.makedirs(out, exist_ok=True)
    subprocess.run([r"C:\Program Files\LibreOffice\program\soffice.exe", "--headless", "--calc",
                    "--convert-to", "xlsx", "--outdir", out, path,
                    "-env:UserInstallation=file:///C:/temp/lo_valuador"], check=True, timeout=120)
    return os.path.join(out, os.path.basename(path))

def test_excel_regresion(tmp_path):
    from render_excel import render_excel
    xlsx = str(tmp_path / "v.xlsx")
    render_excel(json.load(open(INPUTS, encoding="utf-8")), xlsx)
    recalced = _recalc(xlsx)
    wb = load_workbook(recalced, data_only=True)
    e = wb["Escenarios"]
    # localizar fila VALOR DEL TERRENO por etiqueta en col A (robusto a layout)
    vals = {ws.title: ws for ws in wb.worksheets}
    # central en columna D (idx 1). Buscar la fila del terreno.
    terreno_central = None
    for row in e.iter_rows(min_col=1, max_col=5):
        if row[0].value and "VALOR DEL TERRENO" in str(row[0].value).upper():
            terreno_central = row[3].value  # col D
    assert terreno_central is not None and round(terreno_central) == 4620426
    # cero errores de fórmula en todo el libro
    errs = [c.value for ws in wb.worksheets for r in ws.iter_rows() for c in r
            if isinstance(c.value, str) and c.value.startswith("#") and c.value.endswith("!")]
    assert errs == []
```

- [ ] **Step 2: Correr y verificar que falla**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_outputs.py::test_excel_regresion -v`
Expected: FAIL (`ImportError: render_excel`).

- [ ] **Step 3: Implementar `render_excel.py`**

Portar `build_xlsx.py` validado parametrizándolo desde el JSON (los 3 escenarios vienen de `inputs["escenarios"]`, no hardcodeados). Mantener: hojas y layout validados, fórmulas vivas (inputs azules, fórmulas negras, links verdes), las 3 tablas de sensibilidad referenciando la columna central, colores Valia, fuente Lato. La columna de cada escenario se llena desde el JSON; las celdas de resultado (terreno, %GDV, $/m²) son **fórmulas**, no valores. Reusar la función `C()` y los estilos de `build_xlsx.py`.

> DRY: copiar `build_xlsx.py` como base y reemplazar las listas `inputs`/`calc` hardcodeadas por lectura del JSON. NO reescribir desde cero.

- [ ] **Step 4: Correr y verificar que pasa**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_outputs.py::test_excel_regresion -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
cd C:/Users/carlo/Charliecode/valuador-dev && git add -A && git commit -q -m "feat: render Excel 3-escenarios parametrizado + test regresión"
```

---

## Task 4: Render Word (python-docx, portado de docx-js)

**Files:**
- Create: `valuador-dev/render_word.py`
- Test: `valuador-dev/tests/test_outputs.py` (parte Word)
- Reference: `valuacion-arequipa-1110/build_docx.js` (estructura de secciones, branding) — **portar a python-docx**.

- [ ] **Step 1: Escribir el test que falla**

```python
def test_word_estructura(tmp_path):
    from render_word import render_word
    from docx import Document
    docx_path = str(tmp_path / "v.docx")
    render_word(json.load(open(INPUTS, encoding="utf-8")), docx_path)
    doc = Document(docx_path)
    text = "\n".join(p.text for p in doc.paragraphs)
    # secciones clave presentes
    for needle in ["Resumen ejecutivo", "Metodología", "Los tres escenarios",
                   "Análisis de sensibilidad", "Conclusiones", "Disclaimer", "Fuentes"]:
        assert needle in text, f"falta sección: {needle}"
    # valor central en alguna tabla
    flat = "\n".join(c.text for t in doc.tables for r in t.rows for c in r.cells)
    assert "4,620,426" in flat
```

- [ ] **Step 2: Correr y verificar que falla**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_outputs.py::test_word_estructura -v`
Expected: FAIL (`ImportError: render_word`).

- [ ] **Step 3: Implementar `render_word.py`** con python-docx

Replicar la estructura de secciones validada (portada con logo + valor central; resumen con tabla 3 escenarios; objeto/alcance; predio+normativa; **metodología y estrategia de abordaje** con tabla de trazabilidad y origen de supuestos; pasos 1-4; los 3 escenarios; 3 tablas de sensibilidad; conclusiones; supuestos; anexo evidencia; fuentes; disclaimer). Branding: logo del profesional si está en el JSON; colores Valia; fuente Lato. Valores de las tablas vienen de `calc_todos(inputs)` (de `residual.py`, DRY). Numbers formateados con separador de miles.

> python-docx no tiene el bug de bordes pBdr de docx-js; usar `add_table` con estilos y sombreado por celda vía XML (`w:shd`).

- [ ] **Step 4: Correr y verificar que pasa**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_outputs.py::test_word_estructura -v`
Expected: PASS.

- [ ] **Step 5: Commit**

```bash
cd C:/Users/carlo/Charliecode/valuador-dev && git add -A && git commit -q -m "feat: render Word python-docx con metodología trazable"
```

---

## Task 5: Orquestador `generar_valuacion.py` (CLI + PDF)

**Files:**
- Create: `valuador-dev/generar_valuacion.py`
- Test: `valuador-dev/tests/test_outputs.py` (parte end-to-end)

- [ ] **Step 1: Escribir el test que falla**

```python
def test_cli_end_to_end(tmp_path):
    outdir = str(tmp_path)
    rc = subprocess.run([sys.executable, os.path.join(ROOT, "generar_valuacion.py"), INPUTS, outdir],
                        capture_output=True, text=True, timeout=180)
    assert rc.returncode == 0, rc.stderr
    files = os.listdir(outdir)
    assert any(f.endswith(".xlsx") for f in files)
    assert any(f.endswith(".docx") for f in files)
    # PDF puede faltar si no hay LibreOffice; si está, debe existir
```

- [ ] **Step 2: Correr y verificar que falla**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest tests/test_outputs.py::test_cli_end_to_end -v`
Expected: FAIL (archivo no existe).

- [ ] **Step 3: Implementar `generar_valuacion.py`**

```python
import sys, json, os
from render_excel import render_excel
from render_word import render_word

def main():
    inputs_path, outdir = sys.argv[1], sys.argv[2]
    inputs = json.load(open(inputs_path, encoding="utf-8"))
    os.makedirs(outdir, exist_ok=True)
    slug = inputs["meta"].get("direccion", "terreno").replace(" ", "_").replace(",", "")[:40]
    fecha = inputs["meta"].get("fecha", "")
    base = f"Valuacion_Terreno_{slug}_{fecha}"
    xlsx = os.path.join(outdir, base + ".xlsx")
    docx = os.path.join(outdir, base + ".docx")
    render_excel(inputs, xlsx)
    render_word(inputs, docx)
    # PDF best-effort vía LibreOffice
    soffice = r"C:\Program Files\LibreOffice\program\soffice.exe"
    if os.path.exists(soffice):
        import subprocess
        try:
            subprocess.run([soffice, "--headless", "--convert-to", "pdf", "--outdir", outdir, docx,
                            "-env:UserInstallation=file:///C:/temp/lo_valuador_pdf"], check=True, timeout=120)
        except Exception as e:
            print(f"PDF skip: {e}", file=sys.stderr)
    print(json.dumps({"xlsx": xlsx, "docx": docx}))

if __name__ == "__main__":
    main()
```

- [ ] **Step 4: Correr toda la suite y verificar verde**

Run: `cd C:/Users/carlo/Charliecode/valuador-dev && python -m pytest -v`
Expected: todos PASS.

- [ ] **Step 5: Commit**

```bash
cd C:/Users/carlo/Charliecode/valuador-dev && git add -A && git commit -q -m "feat: orquestador CLI generar_valuacion + PDF best-effort"
```

---

## Task 6: Seed `knowledge-peru.md`

**Files:**
- Create: `valuador-dev/knowledge-peru.md`

- [ ] **Step 1: Escribir el seed** con la estructura canónica del spec §7 y los datos validados de Perú (zonificación IMP/ordenanzas + fórmula 1.5(a+r) + área libre; IGV 18%×50%=9% efectivo, umbral 35 UIT; costos CAPECO US$810-1,015/m²; soft 5% / mkt 3.5% / fin 2% / margen 15-20% del GDV; portal Urbania + BCRP; TC S/3.42). Cada cifra con fuente/URL y confianza.

- [ ] **Step 2: Verificar** que cubre las 5 secciones del template de knowledge (zonificación, tributación, costos, mercado, moneda/TC) sin TODOs.

- [ ] **Step 3: Commit**

```bash
cd C:/Users/carlo/Charliecode/valuador-dev && git add -A && git commit -q -m "feat: seed knowledge-peru"
```

---

## Task 7: Construir el agente `.md` (entregable del repo)

**Files:**
- Create: `commands/agente-valuador-terreno-urbano.md` (en el repo)
- Reference: `docs/template-canonico.md`, `commands/agente-valuacion.md` (hermano), spec completo.

Este task NO es unit-testeable (es prosa/instrucciones). Verificación = **checklist de convenciones** + corrida end-to-end (Task 8).

- [ ] **Step 1: Frontmatter + reglas críticas** — `description` accionable; bloque "🚨 REGLAS CRÍTICAS" (6 reglas del template) al inicio.

- [ ] **Step 2: Wizard one-time mínimo** (solo idioma) + lazy (datos profesional en primer Word; país por valuación). Verificación de Python con mensaje amigable del template.

- [ ] **Step 3: Flujo por valuación** (spec §6): país → cargar/bootstrap knowledge → **modo de parámetros** (Híbrido/Automático/Usuario aporta, con degradación) → inputs predio → edificabilidad → mercado (portal + quirks Chrome) → costos+tributo+margen → 3 escenarios → ensamblar JSON → llamar generador → entregar.

- [ ] **Step 4: Bootstrap de país + memoria** — instrucciones para investigar y cachear `knowledge-{pais}.md` (subagentes paralelos por dimensión; manejo de éxito parcial); patrón learnings.

- [ ] **Step 5: Embeber el generador** — pegar el código verificado de `residual.py` + `render_excel.py` + `render_word.py` + `generar_valuacion.py` como bloque(s) que el agente escribe a `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/` en la primera corrida.

- [ ] **Step 6: Embeber el seed `knowledge-peru.md`** — bloque que el agente materializa en la primera corrida en Perú.

- [ ] **Step 7: Comandos especiales + disclaimer** — `reconfigurar`/`ver config`/`ver learnings`/`olvidar learnings`/`ayuda`/`ver knowledge {país}`/`actualizar knowledge {país}`; disclaimer de tasador.

- [ ] **Step 8: Checklist de convenciones** (verificar contra `template-canonico.md` §"Convenciones obligatorias"):
  - [ ] Frontmatter con description accionable
  - [ ] Wizard self-installing (verifica config → wizard si no existe → carga silenciosa si existe)
  - [ ] Rutas `~/inmobiliaria/agents-config/` y `~/inmobiliaria/outputs/valuaciones-terreno/`
  - [ ] Comandos especiales presentes
  - [ ] Outputs Word/Excel/PDF, nunca markdown como entregable
  - [ ] Disclaimer de tasador
  - [ ] Estilo canónico (1 pregunta/turno, sin referencias pedagógicas, sin insights espontáneos)
  - [ ] Prefijo `agente-`
  - [ ] Quirks de Claude in Chrome incluidos/referenciados

- [ ] **Step 9: Commit (en el repo, rama de feature)**

```bash
cd "C:/Users/carlo/Charliecode/ClaudeProjects/ValiaAcademy/inmobiliaria-agentes-repo"
git checkout -b feat/agente-valuador-terreno-urbano
git add commands/agente-valuador-terreno-urbano.md
git commit -m "feat: agente-valuador-terreno-urbano (7º agente)"
```

---

## Task 8: Regresión end-to-end del agente

**Files:** ninguno nuevo (usa el `.md` y el generador embebido).

- [ ] **Step 1: Extraer el generador embebido del `.md`** a un tmp y correrlo con `fixtures/arequipa_inputs.json`.

Run: materializar el código embebido y `python generar_valuacion.py arequipa_inputs.json /tmp/out`.

- [ ] **Step 2: Verificar** que el Excel recalculado da terreno central = 4,620,426 (cero errores) y el Word tiene las secciones clave. (Confirma que el código embebido == el verificado en Tasks 2-5, sin drift.)

- [ ] **Step 3: Lectura crítica del `.md`** — pasar el agente por un subagente revisor con el checklist del template (un solo pase). Corregir hallazgos.

- [ ] **Step 4: Commit de correcciones** (si las hay) en la rama feature.

---

## Task 9: Bump README + descripción a 7 agentes

**Files:**
- Modify: `README.md`
- Modify: descripción del repo (vía `gh repo edit`).

- [ ] **Step 1: Actualizar `README.md`** — todos los puntos acoplados al conteo:
  - [ ] Encabezado "6 agentes" → "7 agentes" + frase agrega "valuación de terrenos por método residual".
  - [ ] "Descarga los 6 archivos" → 7.
  - [ ] Loops de instalación (bash + PowerShell): agregar `agente-valuador-terreno-urbano.md`.
  - [ ] Tabla "Los 6 agentes" → "Los 7 agentes" + fila nueva (comando, qué hace, etapa "Valuación / desarrollo").
  - [ ] Pre-requisitos: "5 de los 6 agentes usan Python" → "6 de los 7".
  - [ ] Estructura del repo: agregar el `.md` y `docs/specs/` + `docs/plans/`.
  - [ ] Ejemplo de uso del nuevo agente.
  - [ ] Footer versión: v0.5 → v0.6, "6 agentes" → "7 agentes", fecha 2026-06-09.

- [ ] **Step 2: Verificar coherencia** — `grep -n "6 agentes\|los 6\|6 archivos" README.md` no debe devolver nada salvo contexto histórico intencional.

Run: `cd "C:/Users/carlo/Charliecode/ClaudeProjects/ValiaAcademy/inmobiliaria-agentes-repo" && grep -niE "6 agentes|los 6|6 archivos|5 de los 6" README.md`
Expected: sin resultados (o solo intencionales).

- [ ] **Step 3: Actualizar descripción del repo**

Run: `gh repo edit valia-academy/agentes-ia-inmobiliarios --description "7 agentes IA self-installing para profesionales inmobiliarios LATAM y España. Reporte de búsqueda anti-salto, análisis legal de contratos, valuación ACM, valuación de terrenos por método residual, procesamiento de fotos con IA, y meta-agente que construye más agentes. Curso completo en Valia Academy."`
> Si `gh` falla por permisos (no colaborador), editar la descripción manualmente en GitHub o reportar al usuario.

- [ ] **Step 4: Commit + merge a main** (con autorización del usuario para push a main, como en el spec)

```bash
git add README.md
git commit -m "docs: bump a 7 agentes (agrega valuador-terreno-urbano)"
git checkout main && git merge --ff-only feat/agente-valuador-terreno-urbano && git push origin main
```

- [ ] **Step 5: Verificar en remoto** que `commands/agente-valuador-terreno-urbano.md` está en `origin/main` y el README dice 7.

---

## Criterios de aceptación (del spec §18)

- [ ] Suite `pytest` verde (math + Excel + Word + e2e).
- [ ] Regresión Av. Arequipa: central = US$ 4,620,426 / ~US$3,836/m² / 15.0% GDV (output, no target).
- [ ] Excel con fórmulas vivas, cero errores; Word con metodología trazable; PDF generado.
- [ ] Agente cumple checklist de `template-canonico.md`.
- [ ] README + descripción en 7 agentes, coherentes, sin romper la instalación.

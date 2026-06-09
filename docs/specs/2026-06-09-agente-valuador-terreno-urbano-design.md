# Spec de diseño — `agente-valuador-terreno-urbano`

- **Fecha:** 2026-06-09
- **Estado:** Borrador para revisión
- **Autor:** Carlos (Valia) + Claude
- **Tipo:** 7º agente de la colección `valia-academy/agentes-ia-inmobiliarios`
- **Origen:** generalización del caso real Av. Arequipa 1110 (valuación de terreno por método residual)

---

## 1. Resumen ejecutivo y propósito

`agente-valuador-terreno-urbano` es un slash command self-installing que **valúa terrenos urbanos con potencial de desarrollo** mediante el **método de valor residual estático** (método del residuo), adaptándose al país del predio. Entrega un Word + Excel + PDF brandeados con la valuación en **tres escenarios** (conservador / central / agresivo), tablas de sensibilidad, una sección de metodología trazable nivel tasador, un anexo de evidencia y un disclaimer profesional.

**Para quién:** profesional inmobiliario LATAM + España (broker, desarrollador, inversionista) que necesita estimar cuánto vale un terreno por su capacidad de soportar un proyecto rentable.

**Diferencia con `agente-valuacion` (ACM):** aquel valúa **inmuebles construidos** por comparables de mercado; este valúa **suelo** por su mejor y mayor uso desarrollable. Son complementarios, no se solapan.

**Problema que resuelve y por qué es no-trivial:** el método residual es universal, pero los **parámetros de edificabilidad** (altura, área libre, coeficiente), el **tratamiento tributario de la primera venta** y los **benchmarks de costos** varían enormemente por país y municipio, y no existe API unificada. La solución es: (a) dejar que el usuario elija, por valuación, **cómo** se obtienen los parámetros (3 modos), y (b) un módulo de **conocimiento por país** que se investiga y cachea on-demand y mejora con el uso.

---

## 2. Posicionamiento y convenciones del repo

El agente cumple `docs/template-canonico.md` sin excepciones:

- **Slash command** en `~/.claude/commands/agente-valuador-terreno-urbano.md` (no sub-agent: el wizard es multi-turno y corre en el contexto principal).
- **Config** en `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano.json`.
- **Knowledge y learnings** en `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/`.
- **Outputs** en `~/inmobiliaria/outputs/valuaciones-terreno/`.
- **Reglas críticas del wizard** (1 pregunta por turno, wizard mínimo, sin referencias pedagógicas, sin insights espontáneos, no especular, estilo canónico) incluidas al inicio del `.md`.
- **Comandos especiales** `reconfigurar` / `ver config` / `ver learnings` / `olvidar learnings` / `ayuda` (+ `ver knowledge {país}`).
- **Outputs entregables siempre en Word/Excel/PDF**, nunca markdown.
- **Disclaimer** de tasador al final del documento.
- **Prefijo `agente-`** y descripción accionable en el frontmatter.

---

## 3. Arquitectura

Cinco componentes, todos orquestados desde el `.md`:

1. **Agente `.md`** — única fuente de verdad. Contiene: reglas críticas, wizard, flujo por valuación, definición del modelo residual, estructura de outputs, y **el código del generador embebido** (ver §8).
2. **Config JSON** (one-time, mínimo) — idioma, datos del profesional/logo (lazy), preferencia de moneda de reporte.
3. **Knowledge por país** — `knowledge-{pais}.md`, investigado on-demand y cacheado (§7).
4. **Motor financiero / generador** — un script Python (`generar_valuacion.py`) que el agente materializa desde el `.md` en el directorio de config en la primera corrida, y que renderiza Word + Excel + PDF de forma **determinista** a partir de un JSON de inputs (§8).
5. **Navegación de portales** — Claude in Chrome para precios de estreno (§6, paso 4), con los quirks del template.

**Flujo de datos (alto nivel):**

```
/agente-valuador-terreno-urbano  →  [config?] →(no) wizard mínimo
                                              →(sí) cargar config + learnings (silencioso)
   → elegir PAÍS → cargar/bootstrap knowledge-{pais}
   → elegir MODO de parámetros (híbrido | automático | usuario aporta)
   → inputs del predio
   → edificabilidad (según modo)
   → precios de mercado (portal del país) → GDV
   → costos + tributación + margen (del knowledge) → residual
   → 3 escenarios + sensibilidad
   → ensamblar inputs JSON → generar_valuacion.py → Word + Excel + PDF
   → entregar paths + disclaimer  → (registrar learnings si aplica)
```

---

## 4. Los tres modos de adquisición de parámetros (elección por valuación)

El agente **pregunta el modo en cada valuación** (no se fija en config). Define el paso de edificabilidad:

| Modo | Qué hace | Inputs que pide | Evidencia / confianza |
|---|---|---|---|
| **Híbrido** (default sugerido) | Infiere la edificabilidad de proyectos reales nuevos del portal en cuadras colindantes + cruza con la norma oficial que logre encontrar online (guiado por el knowledge del país). Pide el Certificado de Parámetros si el usuario lo tiene. | Dirección + (opcional) certificado | Media-alta; documenta supuestos y fuentes |
| **Automático** | Intenta obtener la edificabilidad oficial navegando el visor/normativa del país (método descrito en el knowledge), sin que el usuario aporte nada. | Solo dirección | Variable; depende de que la fuente del país sea navegable. Degrada a Híbrido si falla. |
| **Usuario aporta** | El usuario ingresa altura/área libre/coeficiente/estacionamientos (típicamente de su Certificado de Parámetros). El agente se enfoca en el modelo residual y los precios. | Parámetros normativos explícitos | Alta (parámetros oficiales del usuario) |

**Regla de degradación:** si "Automático" no consigue parámetros confiables tras un intento acotado, el agente lo informa y ofrece pasar a "Híbrido" o "Usuario aporta". Nunca inventa parámetros.

---

## 5. Wizard one-time (mínimo)

Se ejecuta solo la primera vez (no existe el JSON). **Una pregunta por turno.** Mínimo absoluto:

- **Pregunta 1 — Idioma de trabajo y de los reportes** (default español LATAM neutro).

Eso es todo en el wizard inicial. Lo demás es **lazy**:

- **Datos del profesional + logo** → se piden la **primera vez que se va a generar un Word** (nombre, contacto, logo opcional). Se guardan en config y no se vuelven a pedir.
- **País** → se pregunta **por valuación** (el profesional puede operar en varios países).
- **Moneda de reporte** → se infiere del país (USD + moneda local); se puede sobreescribir por valuación.

Tras la pregunta 1: crear directorios, verificar Python (con el mensaje amigable del template si falta), guardar config, confirmar en una línea.

**Config JSON (forma):**
```json
{
  "idioma": "es",
  "datos_profesional": null,
  "moneda_reporte_default": "auto",
  "configurado_en": "<iso>"
}
```

---

## 6. Flujo por valuación (per-run) — detallado

**Paso 0 — País.** Preguntar el país del predio. Cargar `knowledge-{pais}.md` si existe; si no, ejecutar el **bootstrap de país** (§7) antes de continuar, avisando "Primera vez en {país}: investigo normativa, tributación y costos (~1-2 min)".

**Paso 1 — Modo de parámetros.** Preguntar Híbrido / Automático / Usuario aporta (§4).

**Paso 2 — Inputs del predio.** Pedir (atómicamente, solo lo mínimo faltante):
- Dirección (mínimo).
- Área del terreno en m² (mínimo).
- Frente y si es esquinero (opcional; afecta edificabilidad y valor).
- Uso objetivo (opcional; default: multifamiliar residencial con comercio en zócalo si la zonificación lo permite).

**Paso 3 — Edificabilidad** según el modo elegido. Produce: altura (pisos), % área libre, coeficiente/área techada, exigencia de estacionamientos. Documenta fuente y confianza de cada parámetro.

**Paso 4 — Mercado.** Con Claude in Chrome, navegar el portal del país (tabla §10) y muestrear proyectos de **estreno** colindantes: tipologías, áreas, precios USD/m² (o local), mix. Cruzar con la fuente oficial de precios del país si el knowledge la tiene (ej. BCRP en Perú). Producir **precio blended** y mix. Aplicar quirks de Chrome (timeouts, URL scrubbing, locale, anti-bot) del template.

**Paso 5 — Estructura financiera.** Del knowledge del país: costo de construcción (rasante + sótano), soft costs, marketing/ventas, gastos financieros, margen del desarrollador, y **tratamiento tributario de 1ª venta** (tasa efectiva sobre vivienda + base). Calcular el **residual** = GDV − costos − impuestos − margen.

**Paso 6 — Escenarios + sensibilidad.** Construir conservador / central / agresivo (§8) y las tres tablas de sensibilidad. Verificar plausibilidad (margen sobre costos sano en los tres).

**Paso 7 — Outputs.** Si faltan datos del profesional/logo, pedirlos ahora (lazy). Ensamblar el JSON de inputs y llamar a `generar_valuacion.py` → Word + Excel + PDF en `~/inmobiliaria/outputs/valuaciones-terreno/`. Entregar paths clicables + resumen del resultado + próximos pasos accionables + disclaimer.

**Paso 8 — Learnings.** Si hubo quirks de portal, particularidades de normativa o preferencias del usuario, registrarlos (§11).

**Inputs mínimos requeridos:** país, dirección, área del terreno. (En modo "Usuario aporta", además los parámetros normativos.)
**Inputs opcionales:** frente, esquinero, uso objetivo, certificado de parámetros, mix/precios propios, supuestos financieros propios.

---

## 7. Módulo `knowledge-{pais}.md`

**Ruta:** `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/knowledge-{pais}.md`.

**Estructura canónica del archivo:**
```markdown
# Knowledge — {País} (agente-valuador-terreno-urbano)
> Investigado {fecha}. Se lee al inicio de cada valuación en este país. Se refina con el uso.

## 1. Zonificación y edificabilidad
- Fuente(s) oficial(es) y cómo leerlas (visor/portal/norma).
- Cómo se expresa la altura (pisos, fórmula a+r, coeficiente), área libre, estacionamientos.
- Particularidades (zonas monumentales, retiros, premios por esquina/altura).

## 2. Tributación de la primera venta
- Impuesto aplicable (IGV/IVA), base gravada y tasa EFECTIVA sobre el precio de vivienda.
- Exoneraciones/umbral (ej. UIT, valor máximo), si aplican.
- Otros (alcabala/ITP, impuesto a la renta del desarrollador) — referenciales.

## 3. Costos y benchmarks
- Costo de construcción US$/m² (rasante y sótano), rango y central.
- Soft costs, marketing+ventas, gastos financieros, margen del desarrollador (% del GDV).
- Incidencia típica del terreno sobre el GDV (rango, para sanity check).

## 4. Mercado
- Portal de precios de estreno + fuente oficial de precios (si existe).

## 5. Moneda y tipo de cambio
- Moneda local, convención de reporte (USD/local), fuente de TC.
```

**Bootstrap (primera vez en un país):** el agente lanza investigación (web search + navegación) — preferentemente con subagentes paralelos vía la tool `Agent`, uno por dimensión (zonificación, tributación, costos, portal) — y escribe el `knowledge-{pais}.md`. Cada cifra con su fuente/URL y nivel de confianza.

**Éxito parcial del bootstrap:** si una dimensión devuelve poco o nada (ej. encuentra zonificación pero no la tasa tributaria), el agente **igual escribe el knowledge con lo que sí obtuvo**, marca la dimensión faltante como `confianza: baja / pendiente de validación`, y se lo informa al usuario pidiéndole ese dato (o sugiriendo el modo "Usuario aporta" para esa parte). **No bloquea la valuación** por una dimensión incompleta; la trazabilidad del Word refleja qué quedó por confirmar.

**Refresh:** al inicio de cada valuación se lee silenciosamente. El agente puede refrescar selectivamente si detecta que un dato quedó obsoleto (ej. cambio de tasa tributaria) o si el usuario lo pide con `actualizar knowledge {país}`.

**Seed Perú (v1):** se entrega pre-poblado `knowledge-peru.md` con lo ya validado en el caso Av. Arequipa: zonificación (IMP/ordenanzas, fórmula 1.5(a+r), área libre), IGV efectivo 9% sobre vivienda (18%×50%, umbral 35 UIT), costos CAPECO (~US$810–1,015/m²), soft 5% / mkt 3.5% / fin 2% / margen 15–20% del GDV, portal Urbania + BCRP, TC.

**Especificidades por país (semilla de la dimensión zonificación + tributo; el resto lo completa el bootstrap):**

| País | Fuente de edificabilidad | Tributo 1ª venta (base) | Portal precios |
|---|---|---|---|
| Perú | IMP / ordenanzas municipales | IGV 18% s/ construcción (≈9% efectivo) | Urbania |
| México | Uso de suelo / SEDUVI / certificado de zonificación | IVA 16% (vivienda exenta en muchos casos) | Inmuebles24 |
| Argentina | Código Urbanístico (CABA) / código de planeamiento | IVA s/ obra (vivienda con tratamiento especial) | ZonaProp |
| Colombia | POT municipal / norma urbanística | IVA s/ construcción | Metrocuadrado |
| Chile | Plan Regulador Comunal | IVA s/ venta de inmuebles nuevos | PortalInmobiliario |
| Uruguay | Plan de ordenamiento territorial | IVA / ITP | InfoCasas |
| Ecuador | Norma municipal / IRM | IVA | Plusvalía |
| España | Plan General de Ordenación Urbana | IVA 10% vivienda nueva | Idealista |
| Panamá | Norma de zonificación / MIVIOT | ITBMS | Encuentra24 |

> Las cifras finas (tasas efectivas, costos, márgenes) las **verifica el bootstrap** del país; la tabla es el punto de partida, no la verdad final.

---

## 8. Motor financiero y generador de outputs

**Decisión de implementación:** el agente NO re-codifica openpyxl/python-docx a mano en cada corrida (eso introduce variabilidad y riesgo de error en un entregable financiero). En su lugar, el `.md` **embebe el código de un generador determinista** `generar_valuacion.py` que el agente escribe a `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/` en la primera corrida y reutiliza. El agente solo arma un **JSON de inputs** y lo ejecuta. Esto mantiene la distribución de archivo único (el código vive dentro del `.md`) y garantiza outputs consistentes y correctos.

**Stack del generador:** **Python puro** — `openpyxl` (Excel), `python-docx` (Word), y LibreOffice para PDF (con fallback: si no hay LibreOffice, se entregan Word+Excel y se avisa). Se elige `python-docx` sobre `docx-js` por portabilidad (sin dependencia de Node; el template del curso ya estandariza python-docx + openpyxl).

**Cadena de cálculo (idéntica a la validada):**
```
área techada = (1 − área libre) × área terreno × pisos
área vendible = área techada × eficiencia
N° deptos = área vendible / área promedio ponderada
GDV = área vendible × precio blended + estacionamientos + depósitos
residual (terreno) = GDV − construcción − impuesto 1ª venta − soft − marketing − financieros − margen
```
Todas las fórmulas viven en el Excel (celdas, no valores hardcodeados); el `% terreno sobre GDV` es **output**, no input.

**Inputs JSON (esquema):** país, moneda, TC, área terreno, parámetros de edificabilidad (con fuente y confianza por parámetro), eficiencia, mix/áreas/precios por tipología, precios de estacionamiento/depósito, costos (rasante/sótano), %-cost (soft/mkt/fin/margen), tributo (tasa efectiva + base), datos del profesional/logo, comparables (para la hoja de evidencia), fuentes (URLs).

**Tres escenarios:** el JSON lleva tres juegos de supuestos coherentes (Conservador / Central / Agresivo). Definición canónica:
- **Conservador:** menor edificabilidad, precios prudentes, costos/margen mayores.
- **Central (recomendado):** valores modales de las fuentes.
- **Agresivo:** máxima edificabilidad, precios fuertes, costos/margen ajustados.
El generador calcula los tres lado a lado y verifica que el margen sobre costos quede sano en cada uno; si alguno colapsa, lo marca como advertencia.

> **Distinción de "margen" (no confundir en el JSON):** el **margen del desarrollador** es un *input* — el % del GDV que el promotor exige como utilidad (~15–20%), y es uno de los costos que se restan en el residual. El **margen sobre costos** es un *check derivado* que el generador calcula a posteriori (utilidad ÷ costos totales) solo para verificar viabilidad (umbral sano ≈ 20%). Campos: `margen_desarrollador_input` (entra al cálculo) vs `margen_sobre_costos_check` (sale del cálculo, solo validación).

**Sensibilidad:** tres tablas sobre el central — (1) precio × costo de construcción → US$/m², (2) pisos × área libre → US$/m², (3) precio × margen → % del GDV.

**Tributación parametrizada:** el generador recibe `{tasa_efectiva_vivienda, base}` del knowledge del país y aplica el impuesto sobre la porción de vivienda del GDV. Trazabilidad documentada en la hoja Metodología.

---

## 9. Outputs

**Word** (secciones, brandeado con logo del profesional, fuente/colores configurables; default neutro profesional):
1. Portada (logo + valor central + rango).
2. Resumen ejecutivo (tabla 3 escenarios).
3. Objeto y alcance.
4. Predio y marco normativo (parámetros + fuente).
5. **Metodología y estrategia de abordaje** (enfoque, secuencia/trazabilidad, construcción de escenarios, origen de supuestos, tributación, alcance/limitaciones) — nivel tasador.
6. Pasos 1–4 (edificabilidad → vendible → GDV → estructura, escenario central).
7. Los tres escenarios.
8. Sensibilidad (3 tablas).
9. Conclusiones y recomendaciones.
10. Supuestos y limitaciones.
11. Anexo de evidencia (mapa del predio + comparable real; capturas cuando se logren).
12. Fuentes (URLs clicables).
13. Disclaimer de tasador.

**Excel** (fórmulas vivas): hojas Portada · Resumen · Metodología · Escenarios (3 columnas editables) · Sensibilidad · Comparables · Fuentes.

**PDF:** conversión del Word vía LibreOffice.

**Ruta:** `~/inmobiliaria/outputs/valuaciones-terreno/Valuacion_Terreno_<slug-direccion>_<fecha>.{docx,xlsx,pdf}`.

**Idioma y branding:** según config. Disclaimer obligatorio: *"Esta valuación fue generada por IA con método residual y parámetros inferidos/aportados. Es un estimado de apoyo a la decisión, no una tasación reglamentada. Valídala con un tasador certificado y confirma los parámetros con la autoridad municipal antes de usarla para fines oficiales."*

---

## 10. Cobertura de países y portales

Países v1: Perú, México, Argentina, Colombia, Chile, Uruguay, Ecuador, España, Panamá (= los de `agente-valuacion`). Portal default por país según la tabla de §7 (familia Navent comparte quirks; no-Navent se cubre con learnings). Perú llega pre-sembrado; el resto se bootstrapea al primer uso.

---

## 11. Memoria evolutiva (learnings)

Archivo `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano-learnings.md` (patrón del template). Se lee al inicio (silencioso), se agregan entradas al final de la corrida si valen. Registrar: cambios de UI/anti-bot de portales, particularidades de normativa por jurisdicción detectadas en runtime, preferencias persistentes del usuario (ej. siempre reporta en USD aunque opere en CLP). El `knowledge-{pais}.md` guarda **hechos del país**; los learnings guardan **quirks y preferencias** transversales.

---

## 12. Comandos especiales

`reconfigurar` · `ver config` · `ver learnings` · `olvidar learnings` · `ayuda` · **`ver knowledge {país}`** (muestra el knowledge cacheado legible) · **`actualizar knowledge {país}`** (re-bootstrap de ese país).

---

## 13. Manejo de errores y degradación

- **Sin Python:** mostrar el mensaje de instalación del template y abortar (no hay output sin Python).
- **Sin LibreOffice:** entregar Word + Excel; avisar que el PDF requiere LibreOffice y cómo instalarlo.
- **Sin Chrome / sin extensión:** ofrecer modo "Usuario aporta" para parámetros y pedir al usuario precios/comparables manualmente; no bloquear la valuación.
- **Portal con CAPTCHA / anti-bot:** detenerse, avisar, sugerir resolución manual; no más de 1 corrida/hora por portal (quirk del template).
- **País sin knowledge y sin internet:** ofrecer modo "Usuario aporta" con inputs completos.
- **Edificabilidad de baja confianza:** marcarlo explícitamente en el Word y empujar al usuario al Certificado de Parámetros; la sensibilidad y el disclaimer cubren el riesgo.
- **Margen colapsado en un escenario:** advertir que ese estado del mundo no es viable a esos precios/costos.

---

## 14. Reglas críticas del wizard y estilo

Se incluyen al inicio del `.md` tal cual el template (1 pregunta por turno; wizard mínimo + lazy; sin referencias pedagógicas; sin insights espontáneos; no especular sobre el origen de inputs; estilo canónico telegramático; paths como links clicables; siguiente paso accionable). No-negociables.

---

## 15. Fuera de alcance v1 (YAGNI)

- No es tasación reglamentada (REPEV/equivalente nacional).
- Sin Certificado de Parámetros, no garantiza la norma oficial (lo cubre con modos + disclaimer + sensibilidad).
- Solo usos residencial / mixto con comercio en zócalo. **No** valora hotel, oficinas AAA, industrial, ni usos especiales en v1.
- Residual **estático**, no flujo de caja descontado en el tiempo (sin cronograma de ventas ni VAN/TIR). 
- Pre-siembra de knowledge solo para Perú; los demás países se bootstrapean on-demand (no se pre-escriben 9 manuales).
- No integra la API de Valiación ni valiapro (son agentes/skills aparte).

---

## 16. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Edificabilidad inferida incorrecta → valor sesgado | 3 modos (incl. parámetros oficiales del usuario), sensibilidad pisos×área libre, disclaimer, empuje al certificado |
| Tributación mal aplicada por país | Knowledge revisado por fuente oficial + tratamiento conservador + nota de trazabilidad en Metodología |
| Portal cambia UI / bloquea | Quirks del template + learnings + degradación a inputs manuales |
| Bootstrap de país devuelve datos pobres | Marcar confianza baja, pedir validación al usuario, registrar para refinar |
| Variabilidad del output entre corridas | Generador determinista embebido (no re-codificar a mano) |

---

## 17. Artefactos a producir (alcance del plan de implementación)

La implementación v1 produce **dos archivos** (más el generador embebido dentro del primero):

1. `~/.claude/commands/agente-valuador-terreno-urbano.md` — el agente completo: reglas críticas + wizard + flujo por valuación + definición del modelo + **código embebido de `generar_valuacion.py`** + estructura de outputs + comandos especiales + disclaimer.
2. `agente-valuador-terreno-urbano/knowledge-peru.md` (seed) — y, en distribución, copiado a `~/inmobiliaria/agents-config/agente-valuador-terreno-urbano/knowledge-peru.md` en la primera corrida en Perú.

Adicionalmente, para el repo del curso: `commands/agente-valuador-terreno-urbano.md` + entrada en el README + (opcional) `agente-valuador-terreno-urbano-readme.docx` generado.

El bootstrap de otros países y el flujo conversacional son **comportamiento en runtime descrito en el `.md`**, no código separado — por eso el plan es acotado y de un solo entregable principal.

---

## 18. Criterios de aceptación

- Corre end-to-end en Perú (caso Av. Arequipa 1110) y reproduce el resultado validado con los mismos supuestos. Es una **prueba de regresión**: el central ~US$ 4.62M / ~US$3,836/m² / **15% del GDV es el output observado** del modelo con esos inputs, NO un objetivo al que se calibra el generador (el % de terreno siempre sale del residual, nunca se fija).
- Los tres modos de parámetros funcionan y degradan correctamente.
- Genera Word + Excel + PDF brandeados, con Excel de fórmulas vivas y cero errores de fórmula.
- En un segundo país (ej. México) sin knowledge previo, ejecuta el bootstrap y produce una valuación coherente con su tributación/costos.
- Cumple todas las convenciones del template (rutas, comandos, estilo, disclaimer).

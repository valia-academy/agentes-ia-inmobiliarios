# Ideas para construir tus propios agentes

Esta es una lista de **30+ agentes** que puedes construir con `agente-constructor-de-agentes` para extender tu inmobiliaria. Son ejemplos — el meta-agente puede construir cualquier cosa que se te ocurra.

> **Cómo usar esta lista:** cuando termines la clase y quieras tu primer agente propio, ojea las categorías abajo, elige una idea que te resuene, e invoca `agente-constructor-de-agentes` describiéndola. El meta-agente te conducirá por el resto.

---

## Comunicación con clientes

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Mensaje de bienvenida** | Nombre + inmueble visto | Mensaje WhatsApp/email personalizado |
| **Seguimiento post-visita** | Cliente + inmueble visitado + reacción | 3 versiones de mensaje según reacción (le gustó / pensándo / no le gustó) |
| **Invitación formal a visita** | Cliente + dirección + día + hora | Mensaje listo con datos + recordatorio |
| **Respuesta a objeciones frecuentes** | Objeción del cliente | 2-3 versiones de respuesta argumentada |
| **Mensaje de seguimiento de oferta pendiente** | Cliente + inmueble + estado de oferta | Mensaje cordial sin presionar |
| **Felicitación post-cierre** | Cliente + inmueble | Mensaje formal de agradecimiento + invitación a referir |

## Operación interna

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Ficha de cliente nuevo (lead)** | Datos en bruto del cliente | Word estructurado para CRM |
| **Ficha de inmueble brandeada** ⭐ *(ejemplo flagship del meta-agente en clase — texto y presentación primero, fotos al final)* | Carpeta con fotos del inmueble + Word con características | Word editable con tu logo + datos de contacto, listo para revisar y compartir |
| **Planning semanal** | Citas + pendientes | Word con organización + checklist diario |
| **Checklist pre-visita** | Inmueble + cliente | Word con qué preguntar, qué revisar, qué llevar |
| **Resumen semanal de pipeline** | Lista de clientes activos + estado | Word con dashboard del pipeline + acciones recomendadas |
| **Onboarding de inmueble nuevo** | Datos del propietario + inmueble | Carpeta con todos los documentos y datos organizados |

## Marketing y redes

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Post de Instagram/Facebook por inmueble** | Datos + fotos del inmueble | Caption + hashtags + sugerencia de qué fotos usar |
| **Caption corto para Reels/TikTok** | Inmueble + ángulo de venta | Caption optimizado para video corto |
| **Newsletter mensual a base** | Lista de inmuebles destacados | Word listo para enviar |
| **Auditoría de tu propio listing** | URL de tu listing publicado | Análisis crítico + recomendaciones de mejora |
| **Contenido educativo para tu zona** | Tema + audiencia | Post de blog/redes sobre proceso de compra/venta |

## Análisis y reportes

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Reporte mensual a propietario** | Datos de cobranzas + ocupación + mes | PDF brandeado con KPIs + comparativo de mercado |
| **Reporte trimestral a inversionista** | Cartera de inmuebles del inversionista | Reporte con yield + comparativos + recomendaciones |
| **Comparativo de 2-3 inmuebles para cliente** | URLs/datos de los inmuebles | Word con tabla comparativa + pros/cons + recomendación |
| **Análisis de barrio (dossier)** | Zona objetivo + perfil cliente | PDF con colegios, transporte, comercio, precios históricos |
| **Análisis de tu performance del mes** | Tus deals + métricas | Word con review personal + áreas de mejora |

## Captación

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Investigador de prospecto** | Nombre + identificador (email, LinkedIn, empresa) | Brief Word con perfil + capacidad económica + approach sugerido |
| **Mensaje de captación frío para FSBO** | URL de listing FSBO o expirado | Mensaje hiperpersonalizado para el dueño |
| **Análisis crítico de listing ajeno** | URL del listing | "Así sí lo vendería yo" — material para captar al dueño |
| **Ficha pre-llamada con dueño nuevo** | Lo que sabes del dueño + inmueble | Brief con preguntas a hacer + objeciones esperables + plan de cierre |

## Cierre y negociación

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Generador de carta-oferta formal** | Datos básicos de la transacción | Word formal listo para enviar |
| **Presentación de propuesta para cliente** | Inmueble + condiciones de oferta | PDF con sustento de oferta |
| **Análisis de contrapropuesta** | Contrapropuesta recibida | Word con análisis + recomendación de acción |
| **Redacción de adenda contractual** | Cambio que se quiere documentar | Word con redacción legal preliminar |

## Post-venta y referidos

| Idea | Input típico | Output típico |
|------|--------------|---------------|
| **Programa de referidos** | Cliente que cerró | Mensaje de agradecimiento + pedido de referidos + incentivo |
| **Encuesta de satisfacción post-cierre** | Cliente + transacción | Email + Google Form sugerido para feedback |
| **Mantenimiento de relación con propietarios** | Lista de propietarios pasados | Mensaje de check-in cada 6 meses |

---

## Cómo construir tu agente

Invoca al meta-agente y describe la idea:

```
@agente-constructor-de-agentes Quiero crear un agente para [tu idea].
```

El meta-agente te guiará. Si tu idea es muy concreta, va directo a construir; si es vaga, te hace preguntas para entenderla bien antes de diseñar.

---

## Cuándo el meta-agente te sugiere considerar el curso avanzado

Si tu idea requiere alguna de estas cosas, el meta-agente la construirá igual pero te avisará que la versión robusta requiere el **curso avanzado** (que cubre Managed Agents, Agent SDK, MCPs, automatización 24/7):

- **Trigger automático por evento externo** (lead nuevo en WhatsApp, email recibido, formulario de Meta Ads).
- **Funcionamiento 24/7 sin que tú lo invoques** (cron diario, monitoreo continuo).
- **Integración profunda con CRM, WhatsApp Business, Meta Ads, Gmail** vía MCP servers.
- **Coordinación entre múltiples agentes** que comparten estado.
- **Escala** (cientos de operaciones por día, no por demanda manual).

> Para ese tipo de capacidades el curso intro te abre la puerta; el curso avanzado te entrega la implementación robusta lista para producción.

---

## Espacio para tus ideas

Si tienes una idea que no está en esta lista y crees que puede servirle a otros alumnos, **compártela en Skool**. Si funciona bien la incluimos en futuras versiones del repo y le damos crédito a quien la propuso.

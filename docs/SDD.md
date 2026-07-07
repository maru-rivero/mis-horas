# Documento de Diseño de Software (SDD)

## Lumea — Registro de horas y gastos para servicios del hogar

*Lumea: nombre de marca inventado, evocando "lumen/luz" — claridad y transparencia en las cuentas.*

| | |
|---|---|
| **Versión** | 0.2 |
| **Fecha** | 2026-07-07 |
| **Estado** | En revisión |
| **Mercado** | Uruguay (es-UY, UYU, zona horaria America/Montevideo) |

---

## 1. Introducción

### 1.1 Propósito

Este documento define el diseño de una aplicación web (instalable como PWA en el celular) para registrar horas trabajadas y gastos en servicios del hogar: limpieza, cuidado de niños, lavado de autos, jardinería y trabajos similares.

La aplicación conecta a dos tipos de personas:

- **La persona trabajadora**: presta el servicio de forma independiente (no pertenece a una empresa).
- **La persona pagadora** (dueño/a de casa): contrata el servicio y paga por las horas trabajadas y los gastos aprobados.

### 1.2 Principios rectores

Estas son las reglas que gobiernan todas las decisiones de diseño, en orden de prioridad:

1. **Confianza y explicabilidad**: las personas van a confiar en esta aplicación para cobrar y pagar su trabajo. Cada número que muestra la aplicación debe poder explicarse: quién lo registró, cuándo, quién lo aprobó y con qué tarifa se calculó. Nada cambia de estado sin dejar rastro.
2. **Simplicidad para personas no técnicas**: la audiencia no usa herramientas de oficina. Cada pantalla debe entenderse sin manual, en español rioplatense, con acciones grandes y claras desde el celular.
3. **Seguridad de los datos**: los datos son laborales y económicos. Acceso estrictamente limitado a las partes involucradas, cifrado en tránsito y en reposo, y cumplimiento de la Ley N° 18.331 de Protección de Datos Personales de Uruguay.
4. **Costo cero**: gratuita para las personas usuarias y operable con planes gratuitos de infraestructura.

### 1.3 Alcance de la versión 1 (MVP)

**Incluye**: registro de horas, registro de gastos con comprobante, flujo de aprobación bilateral (individual y por lote), cierre mensual automático, reporte mensual, documentos de facturación adjuntos al cierre (PDF o imagen), notificaciones por correo electrónico.

**No incluye (fases posteriores)**: notificaciones push web (fase 2), WhatsApp (fase 3), aplicaciones nativas, pagos dentro de la aplicación, cálculos de aportes a BPS o aguinaldo.

---

## 2. Roles y casos de uso

### 2.1 Roles

Una misma cuenta puede actuar en ambos roles (alguien puede limpiar una casa y a la vez contratar una niñera). El rol no es de la cuenta sino del **acuerdo**.

| Rol | Qué puede hacer |
|---|---|
| Trabajador/a | Registrar horas y gastos, ver estado de aprobación, ver reportes de sus acuerdos. |
| Pagador/a | Registrar horas (en nombre del acuerdo), aprobar o rechazar horas y gastos, ver reportes. |
| Ambos | Crear acuerdos, invitar a la contraparte, ver el historial completo de su acuerdo. |

### 2.2 El acuerdo (concepto central)

Un **acuerdo** vincula a una persona trabajadora con una persona pagadora para un servicio concreto:

- Tipo de servicio (limpieza, niñera, autos, jardín, otro).
- Tarifa por hora en pesos uruguayos, acordada por ambas partes.
- Historial de tarifas: si la tarifa cambia, la nueva rige desde una fecha; las horas anteriores conservan la tarifa vigente en su fecha. Así todo total es reproducible.

Una persona trabajadora puede tener varios acuerdos (varias casas), y una persona pagadora puede tener varios acuerdos (varios servicios). Cada acuerdo tiene sus horas, gastos, cierres y reportes propios.

### 2.3 Casos de uso principales

1. **Registrar horas**: cualquiera de las dos partes registra una jornada: fecha, hora de inicio y fin (o duración), y una nota de la tarea realizada. La entrada queda **pendiente de aprobación por la contraparte** (quien la registró ya la avaló implícitamente al crearla).
2. **Registrar un gasto**: la persona trabajadora carga fecha, monto, descripción y foto del comprobante. Queda pendiente de aprobación de la persona pagadora.
3. **Aprobar / rechazar**: la contraparte aprueba o rechaza cada entrada. El rechazo **exige un motivo**. También puede aprobar **todo lo pendiente en un solo paso** (aprobación por lote). Toda aprobación —individual o por lote— notifica a ambas partes.
4. **Corregir**: una entrada rechazada puede editarse y reenviarse. Una entrada ya aprobada que se edita **vuelve a estado pendiente** y requiere nueva aprobación (queda registrado el cambio).
5. **Cierre de mes**: automático (ver §5.3). Genera el reporte y notifica a ambas partes.
6. **Consultar reporte**: detalle por día de horas y tareas, gastos aprobados, y total a pagar (horas × tarifa + gastos).

---

## 3. Requisitos funcionales

| ID | Requisito |
|---|---|
| RF-01 | Registro de cuenta con correo electrónico y contraseña; verificación de correo obligatoria. |
| RF-02 | Creación de acuerdos con invitación a la contraparte por correo; el acuerdo se activa cuando ambas partes lo aceptan (incluida la tarifa). |
| RF-03 | Registro de horas por día con inicio/fin o duración, tarea y nota opcional, por cualquiera de las dos partes. |
| RF-04 | Registro de gastos con monto, fecha, descripción y foto de comprobante (solo trabajador/a). |
| RF-05 | Aprobación o rechazo (con motivo obligatorio) de cada hora o gasto por la contraparte de quien lo registró. |
| RF-06 | Aprobación por lote de todo lo pendiente de un acuerdo, con notificación a todas las personas involucradas. |
| RF-07 | Edición de entradas: pendientes y rechazadas son editables; una aprobada que se edita vuelve a pendiente; tras el cierre del mes, inmutables. |
| RF-08 | Cierre mensual automático por acuerdo, con recordatorios previos sobre pendientes. |
| RF-09 | Reporte mensual por acuerdo: horas por día, tareas, tarifa aplicada, gastos aprobados, total a pagar; descargable/imprimible. |
| RF-10 | Notificaciones por correo para cada evento relevante (ver §7). |
| RF-11 | Historial visible por entrada y por acuerdo: cada acción con actor, fecha/hora y motivo. |
| RF-12 | Cambio de tarifa con fecha de vigencia, propuesto por una parte y aceptado por la otra. |
| RF-13 | Adjuntar al cierre mensual el documento de facturación o cobro (PDF o imagen: factura, recibo, comprobante de pago), por cualquiera de las dos partes; descargable por ambas en cualquier momento posterior. Cada adjunto queda registrado en la bitácora y notifica a la contraparte. |

## 4. Requisitos no funcionales

| ID | Requisito |
|---|---|
| RNF-01 | **Idioma**: toda la interfaz, correos y reportes en español (es-UY, tratamiento de vos). Formatos: fechas DD/MM/AAAA, moneda $ (UYU). |
| RNF-02 | **Explicabilidad**: todo cambio de estado se registra en una bitácora inmutable (solo agregar, nunca modificar ni borrar). Ningún total se muestra sin poder desglosarse. |
| RNF-03 | **Seguridad**: TLS en todo el tráfico; contraseñas con hash fuerte (bcrypt/argon2, lo gestiona el proveedor de autenticación); aislamiento por fila en base de datos (RLS): cada persona solo ve los acuerdos donde participa; comprobantes en almacenamiento privado con URLs firmadas de corta vida. |
| RNF-04 | **Privacidad**: cumplimiento de la Ley 18.331 (Uruguay): consentimiento informado, finalidad declarada, derecho de acceso, rectificación y supresión de datos. |
| RNF-05 | **Costo**: infraestructura operable en planes gratuitos; sin publicidad ni venta de datos. |
| RNF-06 | **Usabilidad móvil**: diseño primero-móvil; registrar una jornada debe tomar menos de 30 segundos; funciona como PWA instalable. |
| RNF-07 | **Disponibilidad de datos**: respaldos automáticos de la base; los reportes cerrados se conservan como instantánea (no se recalculan). |

---

## 5. Reglas de negocio y máquinas de estado

### 5.1 Estados de una entrada (hora o gasto)

```
                 ┌──────────── editar ────────────┐
                 ▼                                │
 crear ──► PENDIENTE ── aprobar ──► APROBADA ── editar ──► PENDIENTE
                 │                     │
              rechazar              (cierre de mes)
                 ▼                     ▼
             RECHAZADA ── editar ──► CERRADA (inmutable)
                 │
              anular (por quien la creó)
                 ▼
              ANULADA
```

- **Quien crea la entrada la avala; la contraparte aprueba.** Si la registró la persona pagadora, la aprueba la trabajadora, y viceversa. Esto cumple el requisito de que el flujo pueda iniciarlo cualquiera de las dos partes.
- El **rechazo siempre lleva motivo** (texto obligatorio). El motivo se muestra a quien registró la entrada y queda en la bitácora.
- El **monto de una hora** se calcula con la tarifa vigente en la fecha trabajada, no en la fecha de aprobación.
- Al **cerrar el mes**, las entradas aprobadas del período pasan a CERRADA y se congela su importe en el reporte.

### 5.2 Aprobación por lote

- Botón "Aprobar todo lo pendiente" a nivel de acuerdo y mes.
- Internamente genera un evento de aprobación por cada entrada (misma trazabilidad que la aprobación individual) más un evento de lote que los agrupa.
- Notifica a ambas partes: a quien registró ("tus horas fueron aprobadas") y a quien aprobó (confirmación con el resumen del lote).

### 5.3 Cierre mensual automático

| Momento (hora de Montevideo) | Acción |
|---|---|
| Día 1 del mes siguiente, 09:00 | Recordatorio a quien tenga entradas pendientes de aprobar o corregir. |
| Día 3, 09:00 | Último aviso si aún quedan pendientes. |
| Día 5, 23:59 | **Cierre automático del mes anterior.** |

Al cierre:

1. Las entradas **aprobadas** componen el total a pagar y quedan inmutables.
2. Las entradas aún **pendientes o rechazadas** se listan en el reporte en una sección aparte, "No incluidas", con su estado y motivo; **no** suman al total.
3. Si una pendiente se resuelve después del cierre, genera un **reporte complementario (adenda)** del mes, que se notifica a ambas partes. El reporte original nunca se modifica.
4. Se genera el reporte, se guarda como instantánea y se notifica a ambas partes.

*Justificación de la ventana de gracia*: el mes calendario termina, pero las personas necesitan unos días para aprobar lo último. Cinco días equilibra puntualidad del cobro con tiempo real de revisión. Es un parámetro configurable por acuerdo en versiones futuras.*

### 5.4 Inmutabilidad y trazabilidad

- La tabla de **eventos** es de solo-agregar: `quién, cuándo, qué acción, sobre qué entrada, motivo, datos antes/después`.
- Toda pantalla de detalle tiene una sección **"Historial"** que muestra estos eventos en lenguaje claro: *"Ana registró 4 horas el 12/07 · Bruno aprobó el 13/07 · Ana editó la nota el 14/07 (volvió a pendiente) · Bruno aprobó el 14/07"*.
- Los reportes cerrados guardan una copia de los datos al momento del cierre (instantánea JSON + documento), de modo que lo que ambas partes vieron y acordaron es recuperable siempre.

---

## 6. Arquitectura

### 6.1 Stack (todo en planes gratuitos)

| Capa | Tecnología | Por qué |
|---|---|---|
| Frontend | **Next.js (React) + TypeScript**, PWA con service worker | Un solo código para escritorio y celular; instalable; ecosistema maduro. |
| Estilos/UI | Tailwind CSS + componentes accesibles (Radix) | Interfaz consistente y rápida de construir. |
| Backend y base de datos | **Supabase** (PostgreSQL + Auth + Storage + Row Level Security) | Autenticación, permisos por fila y almacenamiento de comprobantes en un solo servicio con plan gratuito sólido. |
| Hosting | **Vercel** (plan gratuito) | Despliegue continuo desde GitHub. |
| Correo transaccional | **Resend** (plan gratuito, 100 correos/día) | Suficiente para el arranque; plantillas en español. |
| Tareas programadas | Vercel Cron / Supabase pg_cron | Recordatorios y cierre mensual automático. |

*Límites aceptados del plan gratuito*: pausa de la base por inactividad prolongada (Supabase), 100 correos/día (Resend). Documentados como riesgos en §10.*

### 6.2 Modelo de datos (esquema lógico)

```
usuarios            (id, nombre, email, telefono?, creado_en)
acuerdos            (id, trabajador_id, pagador_id, servicio, estado, moneda='UYU',
                     dia_cierre=5, creado_en, aceptado_por_trabajador_en, aceptado_por_pagador_en)
tarifas             (id, acuerdo_id, monto_por_hora, vigente_desde,
                     propuesta_por, aceptada_por, aceptada_en)
registros_horas     (id, acuerdo_id, fecha, inicio?, fin?, duracion_min, tarea,
                     nota?, creado_por, estado, tarifa_id_aplicada, creado_en)
gastos              (id, acuerdo_id, fecha, monto, descripcion,
                     comprobante_url, creado_por, estado, creado_en)
eventos             (id, entidad_tipo, entidad_id, acuerdo_id, actor_id, accion,
                     motivo?, datos_antes jsonb, datos_despues jsonb, lote_id?, creado_en)
                     -- SOLO INSERTAR: sin UPDATE ni DELETE (política de BD)
cierres_mensuales   (id, acuerdo_id, anio_mes, cerrado_en, total_horas, total_importe,
                     total_gastos, instantanea jsonb, es_adenda, adenda_de?)
adjuntos_cierre     (id, cierre_id, tipo ('factura'|'recibo'|'comprobante_pago'|'otro'),
                     archivo_url, nombre_archivo, formato ('pdf'|'imagen'),
                     subido_por, creado_en)
                     -- PDF o imagen (JPG/PNG), bucket privado, descargable por ambas partes
notificaciones      (id, usuario_id, canal, evento_tipo, payload jsonb,
                     estado_envio, enviado_en)
```

Reglas de acceso (RLS): una fila de `acuerdos` y todo lo que cuelga de ella solo es visible para `trabajador_id` y `pagador_id`. Los comprobantes viven en un bucket privado; se sirven con URL firmada válida por minutos.

---

## 7. Notificaciones

Canal del MVP: **correo electrónico**. Fase 2: push web (PWA). Fase 3: WhatsApp (requiere API de WhatsApp Business, con costos; se evaluará cuando haya uso real).

| Evento | Destinatario | Contenido |
|---|---|---|
| Entrada registrada (hora/gasto) | La contraparte | Qué se registró y botón para aprobar/rechazar. Se agrupan en un resumen diario si hay varias. |
| Entrada aprobada | Quien la registró | Confirmación con detalle. |
| Entrada rechazada | Quien la registró | Motivo del rechazo y enlace para corregir. |
| **Lote aprobado** | **Ambas partes** | Resumen del lote: cantidad de entradas, horas totales, importe. |
| Recordatorio de pendientes (días 1 y 3) | Quien deba actuar | Lista de pendientes del mes que está por cerrar. |
| **Mes cerrado / reporte listo** | **Ambas partes** | Total del mes y enlace al reporte. |
| Adenda emitida | Ambas partes | Qué se resolvió después del cierre y nuevo total complementario. |
| Documento de facturación adjuntado a un cierre | La contraparte | Tipo de documento (factura/recibo/comprobante), quién lo subió y enlace para descargarlo. |
| Cambio de tarifa propuesto/aceptado | La contraparte / ambas | Tarifa nueva y fecha de vigencia. |

---

## 8. Reporte mensual

Contenido del reporte (visible en la aplicación y descargable como PDF):

1. **Encabezado**: acuerdo, mes, partes, fecha de cierre.
2. **Detalle por día**: fecha, horas (inicio–fin), tarea, tarifa aplicada, importe, quién registró y quién aprobó (con fecha).
3. **Gastos aprobados**: fecha, descripción, monto, enlace al comprobante.
4. **Totales**: horas del mes, importe por horas, importe por gastos, **total a pagar**.
5. **No incluidas**: entradas pendientes o rechazadas al cierre, con estado y motivo.
6. **Documentos adjuntos**: facturas, recibos o comprobantes de pago (PDF o imagen) subidos por cualquiera de las partes después del cierre, listados con quién los subió y cuándo, descargables en todo momento. Los adjuntos se agregan al cierre sin modificar la instantánea del reporte.
7. **Nota de trazabilidad**: identificador del cierre e indicación de que el documento es una instantánea inmutable.

---

## 9. Seguridad y privacidad

- **Autenticación**: correo + contraseña con verificación de correo; opción de enlace mágico. 2FA opcional en fase 2.
- **Autorización**: Row Level Security en PostgreSQL; ninguna consulta puede cruzar acuerdos ajenos, ni siquiera por error de la aplicación.
- **Cifrado**: TLS en tránsito; cifrado en reposo provisto por Supabase; comprobantes en bucket privado.
- **Bitácora**: la tabla `eventos` no admite UPDATE/DELETE (política a nivel de base de datos).
- **Ley 18.331 (Uruguay)**: aviso de privacidad claro en el registro; datos usados solo para el funcionamiento del servicio; derecho de acceso, rectificación y supresión (la supresión de cuenta anonimiza datos personales pero conserva los cierres ya emitidos, porque son registros de la contraparte también — esto se informa en el aviso).
- **Mínimo dato necesario**: no se piden cédula, dirección ni datos bancarios en el MVP.

---

## 10. Riesgos y mitigaciones

| Riesgo | Mitigación |
|---|---|
| Límite de 100 correos/día (Resend gratuito) | Agrupar notificaciones en resúmenes diarios; monitorear volumen; migrar de plan si crece. |
| Pausa de base de datos por inactividad (Supabase gratuito) | Tarea programada de mantenimiento activo; aviso al usuario si tarda el primer acceso. |
| Disputas entre partes | El motivo obligatorio de rechazo + historial completo dan contexto; el diseño evita el borrado silencioso. |
| Usuarios sin correo o que no lo leen | Fase 2 (push) y fase 3 (WhatsApp) atacan esto; mientras tanto, la campana de novedades dentro de la aplicación muestra todo lo no visto. |

---

## 11. Plan de fases

| Fase | Contenido |
|---|---|
| **1 — MVP** | Cuentas, acuerdos e invitaciones, registro de horas y gastos, aprobación individual y por lote, cierre automático, reporte mensual con PDF, notificaciones por correo, historial/bitácora visible. |
| **2** | Notificaciones push (PWA), 2FA, cambio de tarifa asistido, exportación CSV, día de cierre configurable. |
| **3** | WhatsApp, plantillas de jornada ("mismo horario que siempre"), múltiples monedas, modo sin conexión. |

---

## 12. Registro de decisiones

| # | Decisión | Elegido | Motivo |
|---|---|---|---|
| D-01 | Plataforma v1 | Web responsiva + PWA | Un código, sin tiendas de aplicaciones, instalable en el celular. |
| D-02 | Manejo de dinero | Tarifas y totales dentro de la aplicación | El reporte mensual funciona como estado de cuenta real. |
| D-03 | Canales de notificación | Correo (MVP) → push (F2) → WhatsApp (F3) | Correo es gratis y universal; WhatsApp tiene costo de API. |
| D-04 | Mercado | Uruguay | es-UY, UYU, Ley 18.331. |
| D-05 | Granularidad de aprobación | Individual **y** por lote | Pedido explícito; ambas con notificación a todas las partes. |
| D-06 | Cierre de mes | Automático, día 5 del mes siguiente, con recordatorios días 1 y 3 | Pedido explícito de cierre automático; la gracia da tiempo de revisión. |
| D-07 | Infraestructura | Planes gratuitos (Vercel + Supabase + Resend) | Costo cero de operación al arrancar. |
| D-08 | Nombre | **Lumea** | Elegido por la dueña del producto. Nombre de marca corto y memorable, evoca "lumen/luz": claridad en las cuentas. Dominios candidatos: lumea.uy / lumea.app. |
| D-09 | Documentos de facturación | Adjuntos (PDF o imagen) sobre el cierre mensual, subibles por cualquiera de las partes y descargables por ambas | La factura la puede emitir la persona trabajadora y el comprobante de pago la pagadora; el cierre es el lugar natural donde ambas los buscan después. |

### Preguntas abiertas

| # | Pregunta | Supuesto vigente mientras tanto |
|---|---|---|
| P-01 | ~~¿Nombre definitivo de la aplicación?~~ | **Resuelta**: Lumea (ver D-08). |
| P-02 | ¿Tarifas diferenciadas (nocturna, feriados)? | Una sola tarifa por acuerdo en el MVP. |
| P-03 | ¿Se necesita algo relativo a BPS / trabajo doméstico formal (recibos, aguinaldo)? | Fuera de alcance del MVP; el reporte sirve como respaldo informal. |
| P-04 | ¿Idioma del documento y del repositorio? | SDD en español; código y commits en inglés (convención técnica). |

---

## 13. Glosario

- **Acuerdo**: vínculo entre una persona trabajadora y una pagadora para un servicio con una tarifa.
- **Entrada**: un registro de horas o un gasto.
- **Lote**: aprobación de todas las entradas pendientes de un acuerdo en una sola acción.
- **Cierre**: congelamiento mensual de las entradas aprobadas y emisión del reporte.
- **Adenda**: reporte complementario por entradas resueltas después del cierre.
- **Bitácora**: registro inmutable de eventos que hace explicable cada dato de la aplicación.

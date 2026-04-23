# Manual de Usuario SAP — Gestión de Anticipos a Proveedores
## GRUPO VENADO | INDUSTRIAS VENADO S.A. (ERSA)

| Campo | Detalle |
|---|---|
| **Módulo SAP** | Finanzas (FI) — Cuentas por Pagar (FI-AP) |
| **Transacciones** | F-47, F-48, F-53, F-54, FBL1N, FB03, FF67 |
| **Sociedad** | Industrias Venado S.A. — GV01 |
| **Dirigido a** | Usuarios finales — Área de Tesorería / Contabilidad |
| **Versión** | 1.1 — Abril 2026 |

---

## ¿Qué es un Anticipo a Proveedor en SAP?

Un anticipo a proveedor es un pago realizado a un acreedor **ANTES** de recibir la mercancía o el servicio. En SAP, este proceso tiene un tratamiento contable especial: el anticipo **NO** se registra directamente en la cuenta del proveedor (cuenta reconciliación normal), sino en una **cuenta especial de anticipos** (cuenta alternativa de reconciliación), lo que permite mantener visibilidad separada de estos montos hasta que sean compensados con la factura definitiva.

### ¿Por qué SAP trata los anticipos de forma especial?

- **Segregación contable:** El saldo de anticipos se muestra separado del saldo de facturas pendientes del proveedor.
- **Control de saldos:** Permite al área contable identificar cuánto dinero está comprometido en anticipos sin compensar.
- **Cumplimiento normativo:** Las normas contables (NIIF/NIC) exigen que los anticipos se expongan como activos diferenciados, no como cuentas por pagar.
- **Compensación posterior:** Al recibir la factura del proveedor (transacción F-54), el anticipo se compensa automáticamente con la deuda generada.

### Flujo del proceso completo

| Paso | Transacción | Acción | Resultado |
|---|---|---|---|
| 1 | **F-47** | Solicitud de anticipo | Documento tipo KA |
| 2 | **F-48** | Contabilización del anticipo | Documento tipo KZ (Pago anticipo) |
| 3 | **F-53 / FF67** | Salida de pago / Extracto bancario | Documento de pago registrado |
| 4 | **FBL1N** | Verificar saldo del proveedor | Lista de partidas individuales |
| 5 | **FB03** | Visualizar documento contable | Consulta del documento generado |
| 6 | **F-54** | Compensar anticipo con factura | Cierre del ciclo del anticipo |

> ⚠️ F-54 es el paso más crítico del proceso — omitirlo genera doble pago al proveedor.

---

## PROCESO PASO A PASO

---

### PASO 1 — Registrar la Solicitud de Anticipo (F-47)

La transacción **F-47** crea una Solicitud de Anticipo (documento tipo KA). Este documento **NO** genera movimiento contable real; sirve como referencia para el posterior pago del anticipo (F-48). Es una buena práctica registrarla para que el anticipo quede asociado a un pedido de compra.

**Cómo ingresar:** En la barra de comandos SAP escribir `F-47` y presionar Enter.

#### Pantalla: Solicitud de anticipo — Datos cabecera

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Fecha documento** | Fecha del documento de solicitud | 20.02.2026 |
| **Clase doc.** | Tipo de documento SAP | KA (fijo) |
| **Sociedad** | Código de la sociedad contable | GV01 |
| **Fecha contab.** | Período contable en que se registra | 20.02.2026 |
| **Período** | Período contable (mes) | 11 |
| **Moneda/T.C.** | Moneda del documento | BOB |
| **Txt.cab.doc.** | Texto referencial de la cabecera | INFORMAC / libre |
| **Cuenta (Acreedor)** | Número de cuenta del proveedor en SAP | 100120 |

> ℹ️ Al ingresar el número de cuenta del proveedor y presionar Enter, SAP muestra automáticamente el nombre y dirección del acreedor. Verifique que corresponda al proveedor correcto antes de continuar.

#### Pantalla: Añadir Posición de Acreedor

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Importe** | Monto del anticipo solicitado | 1.500,00 BOB |
| **Vence el** | Fecha de vencimiento del anticipo | 21.02.2026 |
| **Doc.compras** | Número del Pedido de Compra (PO) relacionado | 4500036797 |
| **PosPre** | Posición presupuestaria asignada | 104030107 |
| **Bloqueo pago** | Indicador para bloquear pago automático | Vacío = no bloqueado |

> ⚠️ El campo **Doc.compras** es muy importante: vincula el anticipo con la orden de compra correspondiente, facilitando la trazabilidad durante la compensación posterior.

> ✅ La solicitud de anticipo (F-47) genera un documento tipo **KA** con número en el rango 1500xxxxxx. Anote este número — se referenciará en F-48.

---

### PASO 2 — Contabilizar el Anticipo (F-48)

La transacción **F-48** registra el pago efectivo del anticipo. A diferencia de F-47 que solo genera una solicitud, F-48 crea el **asiento contable real**. El documento generado es de tipo KZ.

#### Asiento contable que genera F-48

| Cuenta | Debe | Haber | Descripción |
|---|---|---|---|
| Cta. Anticipos Proveedor (Especial) | 1.500,00 | — | Anticipo pagado al proveedor (KT 39) |
| Banco (Cta. mayor, ej. BMSC) | — | 1.500,00 | Salida de fondos del banco (KT 50) |

#### Campos clave — Datos Cabecera F-48

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Fecha documento** | Fecha del documento contable | 20.02.2026 |
| **Clase doc.** | Tipo de documento de pago | KZ (pago proveedor) |
| **Sociedad** | Sociedad ERSA | GV01 |
| **Moneda/T.C.** | Moneda del documento | BOB |
| **Referencia** | Número de referencia interno/externo | 45 (libre) |
| **Cuenta (Acreedor)** | Proveedor al que se paga el anticipo | 100120 |

> ℹ️ Desde la pantalla de cabecera, hacer clic en **'Solicitudes'** para traer la solicitud de anticipo (F-47) previamente registrada. Esto vincula ambos documentos automáticamente.

#### Posición de Cuenta Mayor — Banco

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Cuenta de mayor** | Cuenta contable del banco de pago | 101041013 (BMSC) |
| **Importe** | Monto a pagar — debe coincidir con el anticipo | 1.500,00 BOB |
| **Posición** | Clave de contabilización Haber banco | 50 (Haber) |
| **Fondo** | Fondo presupuestario asignado | PROPIOS |
| **Área funcional** | Clasificación funcional del gasto | GAF01 |
| **Fecha valor** | Fecha de valoración bancaria | 20.02.2026 |

> ✅ Al grabar, SAP confirma: **'Doc. 1700015274 se contabilizó en sociedad GV01'**. Anote el número de documento KZ para seguimiento.

---

### PASO 3 — Verificar Saldo del Proveedor (FBL1N)

**FBL1N** es la transacción de consulta de partidas individuales de acreedores. Permite visualizar todos los documentos contabilizados para un proveedor.

#### Criterios de selección

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Cuenta de acreedor** | Número del proveedor a consultar | 100120 |
| **Sociedad** | Sociedad contable | GV01 |
| **Partidas abiertas** | Solo documentos pendientes de compensar | Marcar para ver anticipos vigentes |
| **Partidas compensadas** | Documentos ya compensados/pagados | Para historial |
| **Todas las partidas** | Todo el historial del proveedor | Más completo |

#### Indicadores de estado en la lista

| Indicador | Significado |
|---|---|
| 🔴 Círculo rojo | Partida vencida o con alerta |
| 🟢 Círculo verde | Partida vigente / dentro de plazo |
| **KA** | Solicitud de Anticipo (F-47) |
| **KZ** | Pago de Anticipo (F-48) |
| **AB** | Compensación de anticipo con factura |

---

### PASO 4 — Visualizar Documento Contable (FB03)

**FB03** permite consultar cualquier documento contable en SAP. Es de **solo lectura** — no permite modificaciones.

> ℹ️ Para navegar directamente a un documento desde FBL1N, hacer **doble clic** sobre el número de documento. SAP abrirá automáticamente FB03.

---

### PASO 5 — Registrar Salida de Pagos (F-53)

**F-53** se usa para registrar pagos de facturas normales o salidas de pago referenciando partidas abiertas del proveedor.

> ⛔ El balance del documento (D - H) **siempre debe ser 0,00** antes de grabar. Si hay diferencia, SAP no permitirá contabilizar.

---

### PASO 6 — Consulta Global de Anticipos (FBL1N con Filtros)

Para ver todos los anticipos pendientes de la sociedad, ejecutar FBL1N con estos filtros en la sección **Clase**:

- ✅ **Apuntes estadísticos:** Activado — incluye documentos KA/KZ de anticipo
- ☐ **Partidas normales:** Desactivado — filtra solo anticipos
- ☐ **Operaciones CME:** Desactivado — excluye diferencias de cambio

---

## PAGO DE FACTURA Y COMPENSACIÓN — EVITAR DOBLE PAGO

> ⛔ **PROBLEMA REAL:** El doble pago ocurre cuando el operador paga la factura completa **sin compensar** el anticipo previamente registrado. SAP no bloquea automáticamente este error.

### ¿Cómo avisa SAP que hay anticipos pendientes?

| Indicador | Dónde aparece | Significado |
|---|---|---|
| Clase KZ / KA en FBL1N | Lista de partidas | Anticipo abierto — no compensado |
| Importe negativo en BOB | Columna Importe en ML | Dinero ya entregado al proveedor |
| Saldo ≠ 0 en cuenta especial | FBL1N con Apuntes estadísticos | Anticipo pendiente de compensar |
| Aviso en F-53 / F-58 | Durante contabilización de pago | SAP informa si hay anticipos abiertos |

> ⚠️ **REGLA DE ORO:** Antes de registrar cualquier pago (F-53, F-58 o F-110), **SIEMPRE** ejecutar FBL1N para ese proveedor y verificar si existen partidas abiertas de clase KZ o KA.

---

### PASO A — Verificar anticipos en FBL1N antes del pago

Ejecutar FBL1N con:
- **Cuenta acreedor:** Número del proveedor (ej. 100120)
- **Sociedad:** GV01
- **Selección:** Partidas abiertas — fecha clave: hoy
- **Clase — Apuntes estadísticos:** Activado

Si aparecen partidas KZ con importe negativo y sin compensación → **NO proceder con el pago completo**.

---

### PASO B — Compensar el anticipo con la factura (F-54)

#### ¿Qué hace F-54 contablemente?

| Cuenta | Debe | Haber | Descripción |
|---|---|---|---|
| Cta. Proveedor (Reconciliación normal) | 1.500,00 | — | Cancela la deuda de la factura |
| Cta. Anticipos Especial (KT 39) | — | 1.500,00 | Elimina el anticipo de la cuenta especial |

#### Campos clave en F-54

| Campo | Descripción | Ejemplo |
|---|---|---|
| **Fecha documento** | Fecha del documento de compensación | Fecha de la factura recibida |
| **Clase doc.** | Tipo de documento | AB (compensación) |
| **Sociedad** | Sociedad contable | GV01 |
| **Acreedor (Cuenta)** | Número del proveedor | 100120 |
| **Nº factura referencia** | Número del documento MIRO/FB60 | Ej. 1800012345 |
| **Ejercicio factura** | Año del documento de factura | 2026 |

---

### PASO C — Pagar solo la diferencia neta (F-53 o F-58)

| Ejemplo | ❌ SIN compensar | ✅ CON F-54 |
|---|---|---|
| Anticipo pagado (F-48) | 1.500,00 BOB | 1.500,00 BOB |
| Factura recibida (MIRO) | 10.000,00 BOB | 10.000,00 BOB |
| Se ejecuta F-54 | NO — omitido | SÍ — compensado |
| Pago con F-53 | 10.000,00 BOB (factura completa) | 8.500,00 BOB (neto) |
| **Total pagado** | **11.500,00 BOB ← DOBLE PAGO** | **10.000,00 BOB ✅** |

---

### Checklist anti-doble pago

| # | Acción antes de pagar una factura | ☐ |
|---|---|---|
| 1 | Ejecutar FBL1N para el proveedor | ☐ |
| 2 | Verificar partidas abiertas clase KZ | ☐ |
| 3 | Si hay anticipo abierto: ejecutar F-54 | ☐ |
| 4 | Confirmar en FBL1N que el KZ ya no aparece | ☐ |
| 5 | Calcular monto neto: Factura − Anticipo | ☐ |
| 6 | Pagar el monto neto con F-53 o F-58 | ☐ |
| 7 | Verificar saldo proveedor = 0 en FBL1N | ☐ |

---

### Flujo completo del ciclo — Referencia rápida

| # | Transacción | Acción | Clase doc. | FBL1N muestra |
|---|---|---|---|---|
| 1 | F-47 | Solicitud de anticipo | KA | Partida KA abierta |
| 2 | F-48 | Pago del anticipo | KZ | Partida KZ abierta (negativo) |
| 3 | MIRO/FB60 | Registro de factura | RE/KR | Factura abierta (positivo) |
| **4** | **F-54** | **Compensación anticipo↔factura** | **AB** | **KZ desaparece** ✅ |
| 5 | F-53/F-58 | Pago de la diferencia neta | KZ | Saldo proveedor = 0 ✅ |

> ⛔ **El paso 4 (F-54) es el que ERSA ha omitido en casos anteriores, generando el doble pago.** Capacitar al equipo de tesorería para que nunca salte este paso.

---

## Errores Frecuentes

| Error | Causa | Solución |
|---|---|---|
| **'Período no está abierto'** | Fecha de contabilización en período cerrado | Contactar al administrador SAP para abrir el período |
| **'Cuenta XX no existe en PCDA'** | Cuenta especial de anticipos no configurada en el proveedor | Configurar en FK02 — campo 'Cuenta alternativa de reconciliación' |
| **Balance del documento no es cero** | Importes de posiciones no cuadran | Verificar que Debe = Haber en todas las posiciones |

---

## Buenas Prácticas

- Siempre registrar F-47 antes de F-48 para mantener trazabilidad con el pedido de compra.
- Verificar en FBL1N después de cada contabilización que el registro es correcto.
- Documentar el número de documento KZ generado en F-48.
- Compensar anticipos a la brevedad posible al recibir la factura (F-54).
- Para pagos en USD, verificar el tipo de cambio en OB08 antes de contabilizar.

---

## Tablas SAP Relacionadas

| Tabla | Descripción |
|---|---|
| **BKPF** | Cabecera de documentos contables |
| **BSEG** | Posiciones de documentos contables |
| **BSAK** | Partidas compensadas de acreedores |
| **BSIK** | Partidas abiertas de acreedores |
| **LFB1** | Maestro de acreedor por sociedad |
| **T052** | Condiciones de pago |

---

## Transacciones Complementarias

| Transacción | Descripción |
|---|---|
| **F-47** | Solicitud de anticipo a proveedor |
| **F-48** | Contabilización del anticipo |
| **F-53** | Salida de pagos manual |
| **F-54** | Compensar anticipo con factura |
| **F-58** | Pago con impresión de cheque |
| **FBL1N** | Lista de partidas individuales acreedores |
| **FB03** | Visualizar documento contable |
| **FF67** | Extracto de cuenta manual (bancario) |
| **FK02** | Modificar maestro de acreedor |
| **FBZP** | Configurar pagos automáticos y bancos propios |
| **MIRO** | Registro de factura de proveedor (MM) |
| **FB60** | Registro de factura de proveedor (FI) |

---

*Grupo Venado (ERSA) • Módulo FI SAP • Versión 1.1 • Abril 2026*
*Confidencial — Solo uso interno ERSA*

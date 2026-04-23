# VEMASSA | Proceso de Facturación y Logística

## INFORME TÉCNICO — Proceso de Facturación y Logística
### Integración entre Sociedades A y B — VENADO

| Campo | Detalle |
|---|---|
| **Empresa** | VENADO |
| **Alcance** | Sociedad A y Sociedad B |
| **Sistema** | SAP R3 + CRM SGV |
| **Proceso** | Consignación, Facturación por Terceros y Distribución |
| **Versión** | 1.0 |
| **Fecha** | 3 de marzo de 2026 |

---

## 1. Antecedentes y Contexto Organizacional

La empresa VENADO opera con dos sociedades jurídicas independientes que fabrican productos de consumo masivo, ambas integradas en un único sistema SAP R3.

### Sociedad A
- Cuenta con 3 centros productivos y 9 centros de distribución.
- Dispone de equipo propio de ventas y distribución.
- Opera con el CRM SGV para el proceso comercial completo.

### Sociedad B
- Cuenta con 1 centro productivo.
- No dispone de centros de distribución ni equipo de ventas.
- El propietario de Sociedad A es copropietario al 50% de Sociedad B.

### Codificación de Materiales en SAP

Al estar ambas sociedades en una única base de datos SAP, todos los materiales fabricados comparten el esquema de codificación **3XXXXX**. Durante el proceso de traspaso entre sociedades, los materiales de Sociedad B son recodificados con el esquema **6XXXXX** para la Sociedad A, siendo materiales no valorados contablemente.

---

## 2. Objetivo del Proceso

El objetivo principal es distribuir los productos fabricados por la Sociedad B utilizando la infraestructura logística y comercial de la Sociedad A, evitando la doble facturación para minimizar costos al propietario común.

Para lograrlo se implementa un esquema de **facturación por terceros**: la Sociedad A emite facturas propias por sus productos y facturas por cuenta de Sociedad B por los productos de esta última, simulando un proceso de consignación gestionado íntegramente en SAP R3.

---

## 3. Descripción del Proceso de Consignación

### 3.1 Configuración de Centros SAP

Para instrumentar la consignación se crearon centros específicos en cada sociedad:

| Centro | Sociedad | Función |
|---|---|---|
| **GVF1** | Sociedad B | Centro de consignación. Almacena los materiales transferidos y disponibles para Sociedad A con código 3XXXXX. |
| **VEF1** | Sociedad A | Centro receptor. Recibe los materiales recodificados como 6XXXXX y redistribuye a los centros de distribución. |

### 3.2 Flujo de Transferencia de Materiales

**Paso 1 — Producción y Almacenamiento en Sociedad B**

La Sociedad B produce el material y lo almacena en el almacén de Productos Terminados de su planta productiva con código **3XXXXX**.

**Paso 2 — Transferencia al Centro de Consignación (Mov. Z50)**

Mediante la transacción **MIGO** con tipo de movimiento **Z50**, los materiales son transferidos al centro GVF1 dentro de la misma Sociedad B. El material mantiene el código 3XXXXX.

**Paso 3 — Replicación Automática en Sociedad A (JOB cada 30 minutos)**

Un JOB programado en SAP se ejecuta cada 30 minutos. Verifica los materiales transferidos al centro GVF1 con movimiento Z50 y los replica en la Sociedad A mediante MIGO con tipo de movimiento **Z52**. Durante este proceso se efectúa la conversión de código 3XXXXX a 6XXXXX usando una tabla de equivalencias parametrizada en SAP.

**Paso 4 — Distribución a Centros de Distribución**

Los materiales con código 6XXXXX ingresan al centro VEF1 de la Sociedad A, desde donde se redistribuyen a los 9 centros de distribución disponibles. Estos materiales son **no valorados** contablemente para la Sociedad A.

---

## 4. Proceso Comercial y de Ventas

### 4.1 Canales de Venta

La totalidad del proceso comercial se gestiona en el CRM SGV, que está integrado con SAP R3 mediante una interfaz bidireccional. Los canales de venta disponibles son:

- Pedidos de vendedores mediante dispositivos móviles (materiales 3XXXXX y 6XXXXX).
- Ventas directas desde camiones de distribución.
- Ventas en agencias.

### 4.2 Gestión en el CRM SGV

El CRM SGV centraliza todo el proceso comercial, incluyendo:

- Ventas y facturación.
- Cuentas por cobrar y gestión de pagos.
- Devoluciones y notas de crédito/débito.
- Cobranzas.

### 4.3 Interfaz CRM-SAP: Tratamiento Contable por Tipo de Material

| Código Material | Registro Contable | Descripción |
|---|---|---|
| **3XXXXX** | Sociedad A | Ventas, cobranzas, CxC y devoluciones se registran en la contabilidad de la Sociedad A. |
| **6XXXXX** | Sociedad B | Los procesos se ejecutan en la Sociedad B. El stock se descuenta del centro GVF1 mediante conversión 6XXXXX → 3XXXXX. |

---

## 5. Proceso de Salida de Materiales y Réplica

### 5.1 Movimientos de Salida en Sociedad A

Toda salida de materiales con código 6XXXXX de la Sociedad A (movimientos **Z84, Z86, 201, 917, 918** y otros) genera automáticamente su réplica correspondiente en la Sociedad B. Este proceso es gestionado por un JOB programado en SAP denominado **proceso de salida de bandeos**.

### 5.2 Proceso Automático de Réplica (JOB SAP)

- El JOB se ejecuta a intervalos configurables en el sistema SAP.
- Detecta movimientos de salida de materiales 6XXXXX en Sociedad A.
- Genera automáticamente el movimiento equivalente en Sociedad B, descontando el stock del centro GVF1.
- La conversión de código se realiza en sentido inverso: **6XXXXX → 3XXXXX**.
- Cada movimiento genera de manera automática el asiento contable correspondiente en la sociedad respectiva.

---

## 6. Resumen del Esquema de Codificación

| Código | Sociedad | Valorado | Uso |
|---|---|---|---|
| **3XXXXX** | Sociedad B | Sí | Material producido y en consignación (centro GVF1) |
| **6XXXXX** | Sociedad A | No | Material equivalente para ventas y distribución (centro VEF1 y centros de distribución) |

---

## 7. Puntos Clave del Proceso

- La **facturación por terceros** evita la doble facturación y reduce costos operativos para el propietario común.
- El **JOB de replicación** garantiza que el stock en la Sociedad A esté actualizado cada 30 minutos.
- La **tabla de equivalencias 3XXXXX ↔ 6XXXXX** es el elemento central de la integración entre sociedades.
- Los materiales 6XXXXX son intencionalmente **no valorados** en Sociedad A para evitar impacto contable duplicado.
- Todos los movimientos de salida en Sociedad A generan **asientos contables automáticos** en la sociedad que corresponde.
- El **CRM SGV** actúa como sistema comercial único para ambas sociedades, simplificando la operación del equipo de ventas.

---

*VENADO — Grupo Venado (ERSA) | Versión 1.0 | Marzo 2026*

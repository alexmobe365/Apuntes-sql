## Sección 1 · Consultas con JOIN

### Pregunta 1

El equipo de Datos va a construir la tabla de hechos de ventas internacionales. Antes de automatizar la carga, Tomás te pide comprobar a mano que las facturas cruzan correctamente con sus tres dimensiones: cliente, unidad y cuenta.

Muestra las facturas emitidas en el **T3 de 2026** a clientes de **fuera de España** cuyo importe sea de **al menos 80.000 €**.

- Columnas: `factura_id`, `fecha_emision`, `razon_social`, `pais`, `unidad` (nombre de la unidad), `cuenta` (nombre de la cuenta), `importe`.
- Orden: de mayor a menor importe.

**Salida esperada:**

| factura_id | fecha_emision | razon_social | pais | unidad | cuenta | importe |
| --- | --- | --- | --- | --- | --- | --- |
| 56 | 2026-08-10 | Aceros Regiomontanos S.A. de C.V. | MX | Proyectos | Proyectos llave en mano | 128000.00 |
| 51 | 2026-07-16 | Aceros Regiomontanos S.A. de C.V. | MX | Ventas LatAm | Material eléctrico | 111800.00 |
| 64 | 2026-09-08 | Aceros Regiomontanos S.A. de C.V. | MX | Ventas LatAm | Herramienta industrial | 100400.00 |
| 52 | 2026-07-22 | Lusitana de Energia Lda. | PT | Ventas Iberia | Material eléctrico | 88200.00 |
| 57 | 2026-08-12 | Minera Sierra Alta S.A. de C.V. | MX | Ventas LatAm | Herramienta industrial | 88000.00 |

---

## Sección 2 · Subconsultas

### Pregunta 2

Antes de publicar la facturación en la capa de consumo, el pipeline debe señalar los valores atípicos para que alguien los revise.

Con una **subconsulta escalar**, lista las facturas cuyo importe supera **el doble del importe medio** de todas las facturas.

- Columnas: `factura_id`, `razon_social`, `fecha_emision`, `importe`.
- Orden: de mayor a menor importe.

**Salida esperada:**

| factura_id | razon_social | fecha_emision | importe |
| --- | --- | --- | --- |
| 42 | Naviera Atlántida S.A. | 2026-06-15 | 211200.00 |
| 10 | Naviera Atlántida S.A. | 2026-02-10 | 184000.00 |
| 59 | Naviera Atlántida S.A. | 2026-08-25 | 176000.00 |
| 26 | Construcciones Peñalara S.A. | 2026-04-15 | 170000.00 |
| 28 | Naviera Atlántida S.A. | 2026-04-21 | 156800.00 |
| 43 | Minera Sierra Alta S.A. de C.V. | 2026-06-18 | 144600.00 |
| 33 | Hormigones del Sur S.A. | 2026-05-12 | 142400.00 |
| 9 | Minera Sierra Alta S.A. de C.V. | 2026-02-05 | 134000.00 |
| 49 | Hormigones del Sur S.A. | 2026-07-08 | 133600.00 |

### Pregunta 3

Riesgos sospecha que los cobros de la cartera que gestionan los empleados con puesto `Comercial` no están bien conciliados y pide una cifra de control antes de lanzar el proceso de conciliación.

**Sin usar `JOIN`**, con subconsultas anidadas a **tres niveles** (cobros → facturas → clientes → empleados), obtén por método de cobro el número de cobros y el importe total cobrado en facturas de clientes cuyo gestor tiene el puesto `Comercial`.

- Columnas: `metodo`, `cobros`, `importe_cobrado`.
- Orden: de mayor a menor importe cobrado.

**Salida esperada:**

| metodo | cobros | importe_cobrado |
| --- | --- | --- |
| Transferencia | 3 | 228000.00 |
| Domiciliación | 7 | 81000.00 |
| Pagaré | 2 | 66000.00 |

---

## Sección 3 · CTE

### Pregunta 4

El cálculo «importe cobrado por factura» se va a reutilizar en varios procesos del equipo, así que conviene aislarlo como un paso con nombre.

Define una **CTE** que obtenga el importe cobrado de cada factura y úsala para listar las facturas con **cobro parcial**: han recibido algún cobro, pero no están totalmente cobradas.

- Columnas: `factura_id`, `razon_social`, `importe`, `cobrado`, `pendiente`.
- Orden: de mayor a menor saldo pendiente y, en caso de empate, por `factura_id`.

**Salida esperada:**

| factura_id | razon_social | importe | cobrado | pendiente |
| --- | --- | --- | --- | --- |
| 33 | Hormigones del Sur S.A. | 142400.00 | 40000.00 | 102400.00 |
| 42 | Naviera Atlántida S.A. | 211200.00 | 120000.00 | 91200.00 |
| 11 | Construcciones Peñalara S.A. | 108000.00 | 28000.00 | 80000.00 |
| 27 | Minera Sierra Alta S.A. de C.V. | 119000.00 | 50000.00 | 69000.00 |
| 16 | Hormigones del Sur S.A. | 117000.00 | 60000.00 | 57000.00 |
| 51 | Aceros Regiomontanos S.A. de C.V. | 111800.00 | 60000.00 | 51800.00 |
| 48 | Logística Moncayo S.L. | 63000.00 | 30000.00 | 33000.00 |
| 29 | Ferragens do Douro Lda. | 49000.00 | 24000.00 | 25000.00 |
| 41 | Talleres Ebro S.L. | 25200.00 | 12000.00 | 13200.00 |

### Pregunta 5

El equipo comercial quiere saber en qué países y segmentos de cliente se concentró la facturación del T3.

Resuélvelo como un pipeline de **CTE encadenadas**, donde cada una se apoya en la anterior:

1. `facturas_t3`: las facturas emitidas en el T3 de 2026.
2. `enriquecidas`: las facturas anteriores con el país y el segmento de su cliente.
3. `por_segmento`: número de facturas e importe por país y segmento.

La consulta final muestra las combinaciones de país y segmento que facturaron **200.000 € o más**.

- Columnas: `pais`, `segmento`, `facturas`, `importe`.
- Orden: de mayor a menor importe.

**Salida esperada:**

| pais | segmento | facturas | importe |
| --- | --- | --- | --- |
| ES | Gran cuenta | 8 | 528000.00 |
| MX | Gran cuenta | 4 | 428200.00 |
| PT | Distribuidor | 3 | 214600.00 |

### Pregunta 6

Para la dimensión organizativa del modelo analítico hay que aplanar la jerarquía que guarda `dbo.unidades`, de modo que cada unidad lleve su profundidad y su ruta completa.

Con una **CTE recursiva** que parta de la unidad raíz, devuelve todas las unidades con su nivel (la raíz es el nivel 0) y su ruta, separando los nombres con `>`.

- Columnas: `unidad_id`, `nombre`, `tipo`, `nivel`, `ruta`.
- Orden: por `ruta`.

**Salida esperada:**

| unidad_id | nombre | tipo | nivel | ruta |
| --- | --- | --- | --- | --- |
| 1 | Grupo Cierzo | Grupo | 0 | Grupo Cierzo |
| 4 | Corporativo | División | 1 | Grupo Cierzo > Corporativo |
| 9 | Finanzas | Departamento | 2 | Grupo Cierzo > Corporativo > Finanzas |
| 16 | Control de Gestión | Centro de coste | 3 | Grupo Cierzo > Corporativo > Finanzas > Control de Gestión |
| 15 | Riesgos | Centro de coste | 3 | Grupo Cierzo > Corporativo > Finanzas > Riesgos |
| 10 | Tecnología | Departamento | 2 | Grupo Cierzo > Corporativo > Tecnología |
| 17 | Datos | Centro de coste | 3 | Grupo Cierzo > Corporativo > Tecnología > Datos |
| 2 | División Industrial | División | 1 | Grupo Cierzo > División Industrial |
| 5 | Comercial Industrial | Departamento | 2 | Grupo Cierzo > División Industrial > Comercial Industrial |
| 11 | Ventas Iberia | Centro de coste | 3 | Grupo Cierzo > División Industrial > Comercial Industrial > Ventas Iberia |
| 12 | Ventas LatAm | Centro de coste | 3 | Grupo Cierzo > División Industrial > Comercial Industrial > Ventas LatAm |
| 6 | Operaciones y Logística | Departamento | 2 | Grupo Cierzo > División Industrial > Operaciones y Logística |
| 14 | Almacén Monterrey | Centro de coste | 3 | Grupo Cierzo > División Industrial > Operaciones y Logística > Almacén Monterrey |
| 13 | Almacén Zaragoza | Centro de coste | 3 | Grupo Cierzo > División Industrial > Operaciones y Logística > Almacén Zaragoza |
| 3 | División Servicios | División | 1 | Grupo Cierzo > División Servicios |
| 7 | Mantenimiento | Departamento | 2 | Grupo Cierzo > División Servicios > Mantenimiento |
| 8 | Proyectos | Departamento | 2 | Grupo Cierzo > División Servicios > Proyectos |

---

## Sección 4 · Vistas

### Pregunta 7

Hugo Barrios, **Financial Analyst**, monta cada trimestre la cuenta de resultados del grupo en su herramienta de BI. No debe consultar las tablas base ni conocer cómo se cruzan.

Crea la vista `dbo.vw_cuenta_resultados` con una fila por ejercicio, trimestre y cuenta.

- Columnas: `ejercicio`, `trimestre`, `tipo`, `cuenta` (nombre de la cuenta), `importe`.
- La vista reúne los dos tipos de movimiento: las facturas (por `fecha_emision`) y los gastos (por `fecha`).

**Comprobación:** importe total por trimestre y tipo, ordenado por `trimestre` y `tipo`.

**Salida esperada de la comprobación:**

| trimestre | tipo | importe |
| --- | --- | --- |
| 1 | Gasto | 1426490.00 |
| 1 | Ingreso | 1483000.00 |
| 2 | Gasto | 1530590.00 |
| 2 | Ingreso | 1608400.00 |
| 3 | Gasto | 1330600.00 |
| 3 | Ingreso | 1306500.00 |

### Pregunta 8

Clara Mendiola, **Budget Analyst**, hace el seguimiento del presupuesto de gastos y necesita comparar cada línea presupuestada con lo realmente gastado.

Crea la vista `dbo.vw_presupuesto_gastos` con una fila por cada línea de `dbo.presupuestos` cuya cuenta sea de tipo `Gasto`.

- Columnas: `ejercicio`, `trimestre`, `unidad`, `cuenta`, `presupuesto`, `gasto_real`, `desviacion`.
- `gasto_real`: suma de los gastos del mismo trimestre, unidad y cuenta. Si una línea no tiene gastos, vale 0.
- `desviacion` = gasto real − presupuesto.
- Sugerencia: calcula primero, en una CTE, el gasto real por periodo, unidad y cuenta, y únelo después con el presupuesto.

**Comprobación:** líneas del T3 de 2026 cuyo gasto real supera al presupuesto en más de 5.000 €, de mayor a menor desviación.

**Salida esperada de la comprobación:**

| unidad | cuenta | presupuesto | gasto_real | desviacion |
| --- | --- | --- | --- | --- |
| Proyectos | Servicios profesionales | 60000.00 | 89000.00 | 29000.00 |
| Datos | Sueldos y salarios | 18900.00 | 30900.00 | 12000.00 |
| Datos | Licencias de software | 10500.00 | 20400.00 | 9900.00 |
| Ventas LatAm | Sueldos y salarios | 14700.00 | 23400.00 | 8700.00 |
| Almacén Monterrey | Compra de mercaderías | 165500.00 | 174000.00 | 8500.00 |
| Ventas Iberia | Publicidad y marketing | 12000.00 | 18200.00 | 6200.00 |

### Pregunta 9

Iván Calvo, **Risk Analyst**, revisa cada semana la exposición de crédito de los clientes y necesita una única fuente fiable para su informe.

Crea la vista `dbo.vw_riesgo_cliente` con una fila por cliente, **incluidos los que no tienen facturas**.

- Columnas: `cliente_id`, `razon_social`, `rating`, `limite_credito`, `exposicion`, `saldo_vencido`.
- `exposicion`: suma de los saldos pendientes del cliente (0 si no tiene facturas).
- `saldo_vencido`: parte de la exposición que corresponde a facturas con vencimiento anterior a la fecha de corte (0 si no tiene).
- Sugerencia: apóyate en dos CTE, una con el importe cobrado por factura y otra con el saldo pendiente de cada factura.

**Comprobación:** clientes cuya exposición supera su límite de crédito, de mayor a menor exposición.

**Salida esperada de la comprobación:**

| razon_social | rating | limite_credito | exposicion | saldo_vencido |
| --- | --- | --- | --- | --- |
| Construcciones Peñalara S.A. | D | 400000.00 | 468600.00 | 468600.00 |
| Envases Tapatíos S.A. de C.V. | D | 100000.00 | 113800.00 | 113800.00 |

---

## Sección 5 · SELECT INTO

> En SQL Server, el patrón CTAS (`CREATE TABLE ... AS SELECT`) se escribe con `SELECT ... INTO`.
> 

### Pregunta 10

El equipo de BI va a probar un cuadro de mando nuevo y necesita una copia estática de la facturación del T3 para no trabajar sobre `dbo.facturas`.

Con una instrucción **`SELECT ... INTO`**, crea la tabla `dbo.facturas_2026_t3` con las facturas emitidas en el T3 de 2026.

- Columnas: `factura_id`, `cliente_id`, `fecha_emision`, `fecha_vencimiento`, `importe`.
- El script debe poder ejecutarse varias veces sin dar error.

**Comprobación:** número de facturas, importe total y fechas de la primera y la última emisión de la tabla creada.

**Salida esperada de la comprobación:**

| facturas | importe_total | primera_emision | ultima_emision |
| --- | --- | --- | --- |
| 21 | 1306500.00 | 2026-07-06 | 2026-09-30 |

### Pregunta 11

La capa de consumo necesita una tabla de facturas con el cliente ya resuelto y el estado de cobro calculado, para que BI no tenga que cruzar tablas en cada informe.

Con **`SELECT ... INTO`**, crea la tabla `dbo.facturas_enriquecidas` con una fila por factura.

- Columnas: `factura_id`, `razon_social`, `pais`, `fecha_emision`, `importe`, `cobrado`, `pendiente`, `estado_cobro`.
- `estado_cobro` vale `Sin cobro` si la factura no ha recibido ningún cobro, `Cobro parcial` si ha recibido alguno pero queda saldo pendiente y `Cobrada` si no queda saldo.
- El script debe poder ejecutarse varias veces sin dar error.

**Comprobación:** por cada `estado_cobro`, número de facturas, importe facturado y saldo pendiente, de mayor a menor número de facturas.

**Salida esperada de la comprobación:**

| estado_cobro | facturas | importe | pendiente |
| --- | --- | --- | --- |
| Cobrada | 38 | 1784900.00 | 0.00 |
| Sin cobro | 21 | 1666400.00 | 1666400.00 |
| Cobro parcial | 9 | 946600.00 | 522600.00 |

---

## Sección 6 · Tablas temporales

### Pregunta 12

Tesorería pide dos cifras sobre los cobros del T3. Las dos salen del mismo cruce de tablas y no quieres calcularlo dos veces ni dejar ningún objeto nuevo en la base de datos.

Crea la **tabla temporal** `#cobros_t3` con los cobros cuya fecha de cobro cae en el T3 de 2026.

- Columnas: `cobro_id`, `razon_social`, `fecha_cobro`, `importe`, `metodo`.

A partir de la tabla temporal, obtén:

- **Consulta A:** número de cobros e importe cobrado por método, de mayor a menor importe.
- **Consulta B:** los tres clientes con mayor importe cobrado en el trimestre.

**Salida esperada de la consulta A:**

| metodo | cobros | importe_cobrado |
| --- | --- | --- |
| Transferencia | 10 | 483400.00 |
| Confirming | 2 | 276800.00 |
| Domiciliación | 5 | 85900.00 |
| Pagaré | 2 | 64000.00 |

**Salida esperada de la consulta B:**

| razon_social | importe_cobrado |
| --- | --- |
| Naviera Atlántida S.A. | 301800.00 |
| Lusitana de Energia Lda. | 160600.00 |
| Aceros Regiomontanos S.A. de C.V. | 153400.00 |

### Pregunta 13

El banco ha enviado el lote de cobros de principios de octubre, que está en `dbo.stg_cobros` tal como ha llegado. Antes de cargarlo hay que depurarlo. En esta pregunta **no se carga nada**: `dbo.cobros` no debe modificarse.

Resuélvelo con **tablas temporales**, en tres pasos:

1. `#lote_unico`: el lote sin filas repetidas.
2. `#lote_valido`: las filas de `#lote_unico` que cumplen las tres condiciones siguientes:
    - Su `cobro_id` todavía no existe en `dbo.cobros`.
    - Su factura existe en `dbo.facturas`.
    - Su importe es mayor que 0.
3. A partir de las tablas temporales:
    - **Consulta A:** número de filas de `dbo.stg_cobros`, de `#lote_unico` y de `#lote_valido`, con las columnas `etapa` y `filas`, de mayor a menor número de filas.
    - **Consulta B:** el contenido de `#lote_valido`, ordenado por `cobro_id`.

**Salida esperada de la consulta A:**

| etapa | filas |
| --- | --- |
| stg_cobros | 10 |
| #lote_unico | 9 |
| #lote_valido | 5 |

**Salida esperada de la consulta B:**

| cobro_id | factura_id | fecha_cobro | importe | metodo |
| --- | --- | --- | --- | --- |
| 49 | 41 | 2026-10-01 | 13200.00 | Transferencia |
| 50 | 42 | 2026-10-01 | 91200.00 | Confirming |
| 51 | 48 | 2026-10-02 | 16000.00 | Transferencia |
| 54 | 16 | 2026-10-02 | 57000.00 | Pagaré |
| 55 | 63 | 2026-10-02 | 15400.00 | Domiciliación |

---

## Sección 7 · Decisiones de diseño

### Pregunta 14

El equipo de Datos va a restringir el acceso a la información financiera: solo podrán consultarla las personas que dependen, **directa o indirectamente**, del CFO (Álvaro Montes). La consulta que obtiene esa lista se ejecutará cada noche y debe seguir funcionando **sin modificarla** aunque la organización añada o elimine niveles de mando.

Tienes dos técnicas para recorrer la jerarquía de `dbo.empleados`: **subconsultas** o **CTE**. Solo una de las dos cumple el requisito.

1. Indica cuál eliges y explica en dos o tres líneas por qué la otra no es válida.
2. Escribe la consulta que devuelve los empleados que dependen del CFO, sin incluirle a él.
- Columnas: `nombre`, `puesto`, `nivel` (quienes dependen directamente del CFO son el nivel 1).
- Orden: por `nivel` y `nombre`.

**Salida esperada:**

| nombre | puesto | nivel |
| --- | --- | --- |
| Nerea Azcona | Controller | 1 |
| Patricia Soler | Head of Risk | 1 |
| Clara Mendiola | Budget Analyst | 2 |
| Hugo Barrios | Financial Analyst | 2 |
| Iván Calvo | Risk Analyst | 2 |

### Pregunta 15

El lunes se cargará en `dbo.cobros` el lote de cobros de octubre. Antes, Tesorería quiere conservar la **foto de cobros a cierre del T3**: el importe cobrado a cada cliente hasta el 30/09/2026. Los requisitos son tres:

1. La foto **no debe cambiar** cuando se carguen cobros nuevos.
2. Debe **seguir existiendo en enero**, para compararla con el cierre del T4, y cualquier analista debe poder consultarla desde su propia sesión.
3. Debe crearse y cargarse en **una sola instrucción**.

Tienes tres opciones: una **vista**, una tabla creada con **`SELECT ... INTO`** o una **tabla temporal**. Solo una cumple los tres requisitos.

1. Indica cuál eliges y explica por qué las otras dos no son válidas.
2. Impleméntala con el nombre `cierre_cobros_2026_t3`.
- Columnas: `cliente_id`, `razon_social`, `importe_cobrado`, `fecha_foto` (30/09/2026).

**Comprobación:** contenido del objeto creado, de mayor a menor importe cobrado.

**Salida esperada de la comprobación:**

| cliente_id | razon_social | importe_cobrado | fecha_foto |
| --- | --- | --- | --- |
| 4 | Naviera Atlántida S.A. | 535800.00 | 2026-09-30 |
| 9 | Aceros Regiomontanos S.A. de C.V. | 354000.00 | 2026-09-30 |
| 7 | Lusitana de Energia Lda. | 321600.00 | 2026-09-30 |
| 2 | Hormigones del Sur S.A. | 264000.00 | 2026-09-30 |
| 10 | Minera Sierra Alta S.A. de C.V. | 184000.00 | 2026-09-30 |
| 12 | Hospital San Lorenzo | 156800.00 | 2026-09-30 |
| 13 | Logística Moncayo S.L. | 102800.00 | 2026-09-30 |
| 1 | Talleres Ebro S.L. | 70900.00 | 2026-09-30 |
| 5 | Bodegas Valdeolmos S.L. | 69500.00 | 2026-09-30 |
| 8 | Ferragens do Douro Lda. | 66000.00 | 2026-09-30 |
| 3 | Electro Montajes Rioja S.L. | 55500.00 | 2026-09-30 |
| 6 | Construcciones Peñalara S.A. | 28000.00 | 2026-09-30 |

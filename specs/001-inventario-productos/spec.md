# Feature Specification: Catálogo de productos y libro de movimientos de inventario

**Feature Directory**: `specs/001-inventario-productos`

**Created**: 2026-08-19

**Status**: Draft — 5 preguntas de clarificación resueltas; quedan 14 puntos abiertos marcados
con `[NEEDS CLARIFICATION]` (ver el resumen al final)

**Input**: Descripción del usuario:

> El emprendedor tiene una tienda de productos físicos. Hoy sabe qué tiene en bodega porque lo
> mira, o porque lo anota en un cuaderno que nunca cuadra. Necesita saber en cualquier momento
> cuántas unidades tiene de cada producto, y necesita que ese número se mantenga solo a medida
> que compra y vende, sin actualizarlo a mano nunca.
>
> Alcance: catálogo de productos (registrar, consultar, buscar, editar, desactivar); libro de
> movimientos como contrato compartido (hecho inmutable con producto, cantidad con signo, costo
> unitario del momento, fecha, tipo y referencia al documento origen; tipos extensibles: entrada
> por compra, salida por venta, carga inicial y reversión); existencias como suma de movimientos,
> consulta de existencias de todos los productos, kardex por producto con saldo acumulado, y
> carga inicial de existencias.
>
> Fuera de alcance: proveedores/clientes, registro de compras o ventas, categorías/variantes/
> códigos de barras/lotes/vencimientos, múltiples bodegas, ajuste por conteo físico, valorización
> PEPS o promedio ponderado, servicios sin inventario.

---

## Encuadre según la constitución *(obligatorio — añadido a la plantilla)*

> La plantilla `spec-template.md` no contempla estos dos campos, que los principios I y II
> declaran obligatorios. Se añaden aquí (ver Sync Impact Report de `constitution.md`).

### Actividades a las que sirve (Principio II)

| Actividad | ¿Sirve? | Cómo |
|-----------|---------|------|
| **Registrar** | Sí | Alta y edición de productos; registro de movimientos de inventario (carga inicial, reversión y los que otros módulos aporten por contrato). |
| **Consultar** | Sí | Listado y búsqueda del catálogo; existencias actuales de todos los productos; historial de movimientos de un producto con saldo acumulado. |
| **Analizar** | No | Esta feature no produce indicadores ni visualizaciones. Es la base de datos de hechos sobre la que otras features analizarán. |

### Conceptos nuevos que expone al usuario (Principio I)

| Concepto | ¿Indispensable? | Justificación |
|----------|-----------------|---------------|
| **Producto** | Sí | Es el objeto que el emprendedor ya maneja mentalmente; no es vocabulario nuevo. |
| **Existencias** | Sí | Es la pregunta que motiva la feature ("cuántas unidades tengo"). Vocabulario que ya usa. |
| **Movimiento** | Sí | Sin él no se puede explicar por qué las existencias son las que son. Es el único modo de cumplir el principio IV sin exponer jerga contable. |
| **Carga inicial** | Sí | El emprendedor necesita una forma de decirle al sistema lo que ya tiene el primer día. Se presenta como "lo que tengo hoy en bodega", no como un asiento de apertura. |
| **Corrección (reversión)** | Sí | Es la contrapartida obligatoria de la inmutabilidad: si no puede borrar, tiene que poder corregir. |
| **Producto inactivo** | Sí | Contrapartida obligatoria de "nunca se elimina". |
| **Costo de referencia** | A revisar | Es el concepto más cercano a jerga. Ver `[NEEDS CLARIFICATION: C5]` sobre si debe ser obligatorio al dar de alta un producto. |
| ~~Kardex~~ | **No se expone** | "Kardex" es jerga contable y el principio de Restricciones Técnicas la prohíbe en la interfaz. En pantalla se llama **"Movimientos del producto"** o equivalente en español de negocio. "Kardex" queda como nombre interno del equipo. |

---

## Clarifications

### Session 2026-08-19

- Q: ¿Puede el emprendedor registrar a mano una entrada o una salida de inventario dentro de esta feature, o solo puede registrar la carga inicial y las correcciones? (C2) → A: Solo carga inicial y correcciones. No hay pantalla de registro manual de entradas ni salidas; esas llegan por el contrato compartido cuando existan compras y ventas.
- Q: Cuando llegue una salida que dejaría las existencias de un producto por debajo de cero, ¿el sistema debe rechazarla, aceptarla y mostrar el saldo en negativo, o aceptarla avisando? (C6) → A: Rechazarla. El movimiento no se registra y el contrato devuelve un fallo explícito. Las existencias nunca pueden ser negativas.
- Q: Si corregir un movimiento exigiera una salida que dejaría las existencias en negativo, ¿esa corrección también se rechaza? (C12, parcial) → A: No. La corrección es la única excepción: se registra aunque deje el saldo negativo, y el producto queda destacado como pendiente de regularizar.
- Q: ¿Cuántas veces puede el emprendedor cargar existencias iniciales de un mismo producto, y en qué momento lo hace? (C9, parcial) → A: Una sola vez por producto, como acción separada posterior al alta, permitida solo mientras el producto no tenga ningún otro movimiento.
- Q: ¿De dónde sale el costo unitario que queda guardado en cada movimiento, si la valorización por PEPS o promedio está fuera de alcance? (C10 + C9 parcial) → A: Lo aporta el módulo que origina el movimiento cuando lo conoce; si no lo aporta, se copia el costo de referencia vigente del producto. El emprendedor nunca escribe un costo unitario.

---

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Tener el catálogo de lo que vendo (Priority: P1)

El emprendedor abre el sistema por primera vez y registra los productos que vende: nombre,
en qué unidad los mide, a qué precio los vende y cuánto le cuestan. Después puede encontrar
cualquiera de ellos en una lista, buscarlo por nombre, corregir un dato mal escrito y retirar de
circulación uno que ya no vende — sin perderlo, porque tiene historia detrás.

**Why this priority**: sin catálogo no hay nada a lo que asociar movimientos. Es el único
requisito previo duro de todas las demás historias, y por sí sola ya sustituye la hoja de
"lista de productos y precios" que hoy vive en Excel.

**Independent Test**: se prueba dando de alta varios productos, buscándolos, editando uno y
desactivando otro, sin registrar ni un solo movimiento. Entrega valor porque el emprendedor ya
tiene su lista de precios centralizada.

**Acceptance Scenarios**:

1. **Given** el catálogo vacío, **When** el emprendedor registra un producto con nombre, unidad
   de medida, precio de venta y costo de referencia, **Then** el producto aparece en el listado
   y queda activo.
2. **Given** un catálogo con varios productos, **When** el emprendedor escribe parte de un
   nombre en la búsqueda, **Then** el listado muestra solo los productos cuyo nombre coincide.
3. **Given** un producto registrado, **When** el emprendedor corrige su precio de venta,
   **Then** el listado muestra el precio nuevo y los movimientos ya registrados conservan el
   costo unitario con el que se registraron.
4. **Given** un producto que ya no se vende, **When** el emprendedor lo desactiva, **Then** el
   producto deja de ofrecerse para nuevas operaciones pero sigue existiendo y sus movimientos
   siguen siendo consultables.
5. **Given** cualquier producto del catálogo, **When** el emprendedor busca una opción de
   eliminar, **Then** no existe tal opción en ninguna pantalla.

---

### User Story 2 - Saber cuánto tengo hoy, sin contar (Priority: P2)

El emprendedor registra de una vez lo que ya tiene en bodega el día que empieza a usar el
sistema, y a partir de ahí abre una pantalla y ve cuántas unidades tiene de cada producto. Ese
número no lo escribe nunca: sale solo de lo que ha entrado y salido.

**Why this priority**: es la pregunta que motiva la feature. Depende de US1 (necesita productos)
pero no de US3 ni de US4.

**Independent Test**: se prueba cargando las existencias iniciales de varios productos y
abriendo la consulta de existencias; el número mostrado debe coincidir con lo cargado. Entrega
valor porque sustituye el conteo visual y el cuaderno.

**Acceptance Scenarios**:

1. **Given** un producto recién registrado sin movimientos, **When** el emprendedor consulta sus
   existencias, **Then** el sistema muestra cero.
2. **Given** un producto sin movimientos, **When** el emprendedor registra la carga inicial de
   40 unidades, **Then** la consulta de existencias muestra 40.
3. **Given** un producto con 40 unidades cargadas, **When** llega una salida de 15 a través del
   contrato de movimientos (FR-019) — esta feature no ofrece pantalla que la capture, ver FR-032
   —, **Then** la consulta de existencias muestra 25 sin que el emprendedor ejecute ninguna
   acción de recálculo ni de sincronización.
4. **Given** varios productos con movimientos, **When** el emprendedor abre la consulta de
   existencias, **Then** ve todos sus productos con la cantidad de cada uno en una sola pantalla.
5. **Given** cualquier pantalla del sistema, **When** el emprendedor intenta escribir directamente
   sobre la cifra de existencias, **Then** el sistema no lo permite: las existencias no son un
   campo editable.

---

### User Story 3 - Entender por qué el número es el que es (Priority: P3)

El emprendedor ve que un producto marca 25 unidades y no le cuadra con lo que recuerda. Abre los
movimientos de ese producto y ve, en orden, cada entrada y cada salida con su fecha y de dónde
vino, y cuántas unidades quedaban después de cada una. Encuentra la salida que no esperaba y
entiende el número sin preguntarle a nadie.

**Why this priority**: es lo que convierte al sistema en confiable frente al cuaderno. Depende de
US1 y US2 pero puede entregarse después.

**Independent Test**: se prueba registrando una secuencia conocida de movimientos de un producto
y abriendo su historial; el saldo acumulado de la última línea debe coincidir con las existencias
mostradas en la consulta general.

**Acceptance Scenarios**:

1. **Given** un producto con movimientos, **When** el emprendedor abre sus movimientos, **Then**
   ve cada movimiento en orden cronológico con fecha, tipo, cantidad con su signo, costo unitario,
   referencia de origen y el saldo acumulado después de ese movimiento.
2. **Given** el historial de un producto, **When** el emprendedor mira el saldo acumulado de la
   última línea, **Then** ese valor es idéntico a las existencias que muestra la consulta general.
3. **Given** un producto sin movimientos, **When** el emprendedor abre sus movimientos, **Then**
   el sistema muestra una lista vacía y existencias en cero, sin error.

---

### User Story 4 - Corregir un error sin borrar nada (Priority: P4)

El emprendedor descubre que un movimiento se registró mal — 100 unidades donde eran 10. No puede
borrarlo ni editarlo. Registra una corrección que anula el movimiento equivocado, y el historial
queda mostrando las tres cosas: el error, la corrección y el saldo correcto.

**Why this priority**: es la consecuencia práctica de la inmutabilidad del principio IV. Sin
esto, un error tipográfico deja el inventario mal para siempre y el emprendedor pierde la
confianza en el número.

**Independent Test**: se prueba registrando un movimiento erróneo, corrigiéndolo, y verificando
que las existencias vuelven al valor esperado y que ambos movimientos siguen visibles.

**Acceptance Scenarios**:

1. **Given** un movimiento registrado por error, **When** el emprendedor lo corrige, **Then** el
   sistema crea un movimiento nuevo de tipo reversión, con cantidad de signo contrario e igual
   magnitud, que referencia al movimiento original.
2. **Given** un movimiento ya corregido, **When** el emprendedor consulta el historial, **Then**
   siguen apareciendo tanto el movimiento original como su reversión, y ninguno aparece borrado
   ni tachado del cálculo.
3. **Given** cualquier movimiento del historial, **When** el emprendedor busca una opción de
   editarlo o eliminarlo, **Then** no existe tal opción.

---

### Edge Cases

- **Salida mayor que las existencias**: el movimiento se rechaza por completo y no queda rastro
  de él en el libro; el contrato devuelve un fallo con el detalle (FR-033). Las existencias nunca
  quedan en negativo.
- **Corrección que deja el saldo en negativo**: se cargan 40 unidades, se venden 35, y luego se
  descubre que la carga inicial correcta eran 10. La corrección saca 40 sobre un saldo de 5 y se
  registra igualmente (FR-035): el producto queda en −35 y aparece destacado como pendiente de
  regularizar hasta que se registre la entrada que falta.
- **Producto inactivo con existencias**: un producto se desactiva cuando todavía le quedan
  unidades. ¿Sigue apareciendo en la consulta de existencias? Ver `[NEEDS CLARIFICATION: C15]`.
- **Movimiento sobre producto inactivo**: llega un movimiento (o una corrección de un movimiento
  antiguo) de un producto ya desactivado. Ver `[NEEDS CLARIFICATION: C7]`.
- **Dos movimientos con la misma fecha**: el orden entre ellos cambia el saldo acumulado que se
  muestra línea a línea. Ver `[NEEDS CLARIFICATION: C14]`.
- **Carga inicial repetida**: imposible por diseño. Tras la primera carga inicial —o tras
  cualquier otro movimiento— la opción deja de ofrecerse y el sistema rechaza una segunda
  (FR-031a).
- **Carga inicial olvidada**: el emprendedor registra un producto, empieza a operar con él y
  solo entonces cae en que nunca cargó lo que tenía. Como ya hay movimientos, la carga inicial
  está cerrada y la vía es la corrección (FR-035). Ver `[NEEDS CLARIFICATION: C12 (resto)]` sobre
  si esa vía está disponible desde el historial.
- **Corrección de una corrección**: el emprendedor se equivoca al corregir. Ver
  `[NEEDS CLARIFICATION: C12]`.
- **Cantidad cero o negativa en un alta**: se intenta registrar un movimiento con cantidad cero,
  o una entrada con cantidad negativa. El sistema debe rechazarlo (FR-021).
- **Producto sin ningún movimiento**: aparece en existencias con cero, no se omite del listado.
- **Nombre duplicado**: se registra un producto con un nombre que ya existe. Ver
  `[NEEDS CLARIFICATION: C3]`.
- **Edición del precio o del costo de referencia**: no puede alterar retroactivamente el costo
  unitario de ningún movimiento ya registrado (FR-007).
- **Búsqueda sin resultados**: el listado muestra el estado vacío con un mensaje comprensible,
  no una tabla en blanco sin explicación.

## Requirements *(mandatory)*

### Functional Requirements — Catálogo de productos

- **FR-001**: El sistema MUST permitir registrar un producto con nombre, unidad de medida, precio
  de venta y costo de referencia.
- **FR-002**: El sistema MUST permitir consultar el listado completo de productos.
- **FR-003**: El sistema MUST permitir buscar productos dentro del listado.
  `[NEEDS CLARIFICATION: C16 — ¿sobre qué campos busca (solo nombre, o también unidad/código)? ¿la búsqueda es por coincidencia parcial e insensible a mayúsculas y acentos?]`
- **FR-004**: El sistema MUST permitir editar los datos de un producto ya registrado.
  `[NEEDS CLARIFICATION: C4 — ¿todos los campos son editables, incluido el nombre? ¿editar un producto con movimientos históricos tiene alguna restricción adicional?]`
- **FR-005**: El sistema MUST permitir desactivar un producto y MUST NOT ofrecer en ninguna
  pantalla la posibilidad de eliminarlo.
- **FR-006**: Un producto desactivado MUST conservar íntegro su historial de movimientos y este
  MUST seguir siendo consultable.
- **FR-007**: Editar el precio de venta o el costo de referencia de un producto MUST NOT alterar
  el costo unitario ni ningún otro dato de los movimientos ya registrados.
- **FR-008**: El sistema MUST distinguir productos activos de inactivos en el listado.
  `[NEEDS CLARIFICATION: C15 — ¿el listado de productos y la consulta de existencias muestran los inactivos por defecto, los ocultan, o los muestran solo cuando tienen existencias distintas de cero?]`
- **FR-009**: El sistema MUST poder reactivar un producto desactivado, o MUST declarar
  explícitamente que la desactivación es irreversible.
  `[NEEDS CLARIFICATION: C8 — ¿la desactivación es reversible?]`

**Campos del producto — decisiones abiertas**

- **FR-010**: La unidad de medida de un producto MUST registrarse junto al producto.
  `[NEEDS CLARIFICATION: C1 — ¿la unidad de medida es texto libre que el emprendedor escribe, o una lista cerrada predefinida por el sistema (unidad, kg, litro, caja…)? Si es lista cerrada, ¿quién la define y es ampliable por el usuario?]`
- **FR-011**: El sistema MUST definir qué campos son obligatorios al dar de alta un producto.
  `[NEEDS CLARIFICATION: C5 — el criterio de éxito exige que el alta no tenga campos opcionales. ¿Eso significa que los cuatro campos son obligatorios? En particular: ¿un emprendedor que no conoce todavía su costo de referencia puede registrar el producto? Si el costo de referencia no fuera obligatorio, sería un campo opcional y contradiría el criterio de éxito; si lo fuera, bloquea el alta.]`
- **FR-012**: El sistema MUST identificar cada producto de forma inequívoca para el emprendedor.
  `[NEEDS CLARIFICATION: C3 — ¿el nombre debe ser único y el sistema rechaza duplicados, o se permiten nombres repetidos? ¿Existe además un código de producto propio visible para el usuario, o el nombre es el único identificador que él ve? (Códigos de barras están fuera de alcance; esto se refiere a un código interno.)]`

### Functional Requirements — Libro de movimientos (contrato compartido)

- **FR-013**: Cada movimiento de inventario MUST registrar, como mínimo: producto, cantidad con
  signo, costo unitario del momento, fecha de ocurrencia, fecha de registro, tipo de movimiento y
  referencia al documento que lo originó.
- **FR-014**: La cantidad de un movimiento MUST ser positiva para entradas y negativa para
  salidas.
- **FR-015**: Un movimiento registrado MUST NOT poder editarse ni eliminarse por ningún camino:
  ni desde la interfaz, ni desde el contrato que consumen otros módulos.
- **FR-016**: La única forma de corregir un movimiento erróneo MUST ser registrar un movimiento
  nuevo de tipo reversión, de signo contrario, que referencie al movimiento original.
- **FR-017**: El sistema MUST soportar, como mínimo, los tipos de movimiento: **entrada por
  compra**, **salida por venta**, **carga inicial** y **reversión**.
- **FR-018**: El conjunto de tipos de movimiento MUST poder ampliarse con tipos nuevos sin
  modificar la lógica de cálculo de existencias ni la de presentación del historial.
- **FR-019**: El módulo MUST exponer un contrato explícito que permita a otros módulos —compras y
  ventas, aún no especificados— registrar movimientos y consultar existencias, sin acceder a sus
  estructuras internas de datos.
- **FR-020**: Los módulos consumidores MUST NOT poder redefinir ni reinterpretar los tipos de
  movimiento existentes; los usan tal como están definidos aquí.
- **FR-021**: El sistema MUST rechazar el registro de un movimiento con cantidad cero, con
  cantidad de signo incompatible con su tipo, sin producto, o con costo unitario negativo.
- **FR-022**: El costo unitario de un movimiento MUST determinarse así: si el módulo que origina
  el movimiento lo aporta, se registra el valor aportado; si no lo aporta, se copia el costo de
  referencia vigente del producto en ese momento. Una vez registrado MUST NOT recalcularse nunca.
- **FR-022a**: El emprendedor MUST NOT escribir un costo unitario en ninguna pantalla de esta
  feature. En particular, la carga inicial copia el costo de referencia vigente del producto.
- **FR-022b**: Esta regla MUST NOT interpretarse como valorización de inventario: no establece el
  costo de la mercancía vendida ni ningún método PEPS o promedio ponderado, que siguen fuera de
  alcance. Solo fija qué cifra queda guardada como dato histórico en cada movimiento.
- **FR-023**: Todo movimiento MUST llevar una referencia al documento que lo originó.
  `[NEEDS CLARIFICATION: C11 — ¿qué referencia llevan los movimientos que no nacen de un documento externo (carga inicial y reversión)? ¿Es obligatoria en todos los casos? ¿Qué ve el emprendedor en esa columna del historial para una carga inicial?]`
- **FR-024**: El sistema MUST registrar la fecha de ocurrencia y la fecha de registro de cada
  movimiento (exigido por el principio IV).
  `[NEEDS CLARIFICATION: C13 — ¿el emprendedor puede fijar una fecha de ocurrencia distinta de hoy (registro retroactivo)? ¿Se aceptan fechas futuras? ¿La granularidad es día, o fecha y hora?]`

### Functional Requirements — Existencias y consulta

- **FR-025**: Las existencias de un producto MUST calcularse siempre como la suma de las
  cantidades de sus movimientos.
- **FR-026**: Las existencias MUST NOT persistirse como campo almacenado editable, MUST NOT
  exponerse como campo modificable por el usuario y MUST NOT requerir ningún proceso de
  recálculo, sincronización o cierre para estar al día.
- **FR-027**: El sistema MUST permitir consultar en una sola pantalla las existencias actuales de
  todos los productos.
- **FR-028**: El sistema MUST permitir consultar los movimientos de un producto en orden
  cronológico, mostrando en cada línea el saldo acumulado después de ese movimiento.
- **FR-029**: El saldo acumulado de la última línea del historial de un producto MUST coincidir
  siempre con las existencias que muestra la consulta general para ese producto.
- **FR-030**: El sistema MUST definir un orden total y estable de los movimientos de un producto.
  `[NEEDS CLARIFICATION: C14 — cuando dos movimientos de un producto comparten la misma fecha de ocurrencia, ¿qué criterio los desempata (fecha de registro, orden de captura, otro)? El saldo acumulado línea a línea depende de esta decisión.]`
- **FR-031**: El sistema MUST permitir al emprendedor registrar la carga inicial de existencias de
  un producto como una **acción separada posterior al alta**, no como un campo del formulario de
  registro.
- **FR-031a**: Un producto MUST admitir **como máximo una** carga inicial, y solo mientras no
  tenga ningún otro movimiento registrado. En cuanto el producto tenga cualquier movimiento —
  incluida su propia carga inicial— el sistema MUST NOT ofrecer ni aceptar otra.
- **FR-031b**: La carga inicial MUST registrarse como un movimiento del libro igual que cualquier
  otro (tipo carga inicial), sin trato especial en el cálculo de existencias.
- **FR-031c**: La carga inicial MUST registrar como costo unitario el costo de referencia vigente
  del producto, copiado automáticamente (FR-022a). Falta fijar su fecha de ocurrencia.
  `[NEEDS CLARIFICATION: C9 (resto) — ¿qué fecha de ocurrencia lleva la carga inicial: siempre la del día en que se registra, o una fecha que el emprendedor elige (p. ej. el día en que realmente contó la bodega)?]`

  > **Consecuencia aceptada**: combinando FR-031a con FR-032, hasta que exista el módulo de
  > compras las existencias de un producto no pueden aumentar por ningún camino después de su
  > carga inicial. El MVP demostrable de esta feature cubre por tanto: catálogo, carga inicial,
  > consulta, historial y corrección.
- **FR-032**: El emprendedor MUST poder registrar a mano exactamente dos clases de movimiento en
  esta feature: la **carga inicial** de existencias (FR-031) y la **corrección** de un movimiento
  previo (FR-016). El sistema MUST NOT ofrecer ninguna pantalla de registro manual de entradas ni
  de salidas de inventario. Las entradas por compra y las salidas por venta MUST llegar
  exclusivamente a través del contrato de FR-019, cuando existan los módulos que las originan.

  > **Consecuencia aceptada**: hasta que exista el módulo de ventas, la única forma de que las
  > existencias de un producto disminuyan desde la interfaz es la corrección de un movimiento
  > previo. Los escenarios de salida se verifican con pruebas automatizadas contra el contrato
  > (SC-008), no desde la interfaz.
- **FR-033**: El sistema MUST rechazar todo movimiento de salida que dejaría las existencias del
  producto por debajo de cero. El movimiento no se registra en absoluto y el contrato de FR-019
  MUST devolver un fallo explícito que identifique el producto, las existencias disponibles en
  ese momento y la cantidad solicitada, para que el módulo que lo originó pueda explicárselo al
  emprendedor.
- **FR-033a**: **Invariante**: un movimiento que no sea una corrección MUST NOT dejar las
  existencias de un producto por debajo de cero. Es una propiedad verificable del libro de
  movimientos en su conjunto, no solo una validación de entrada. La única excepción son las
  correcciones (FR-035).

  > **Consecuencia aceptada**: una venta puede fallar porque la compra que la abastece todavía no
  > se ha registrado. El módulo de ventas —fuera de alcance aquí— deberá manejar ese fallo y
  > guiar al emprendedor a registrar antes la entrada. Ver también FR-035 sobre el efecto de esta
  > regla al corregir un movimiento.
- **FR-034**: El sistema MUST definir el tratamiento de los movimientos que afectan a un producto
  inactivo.
  `[NEEDS CLARIFICATION: C7 — ¿un producto desactivado admite movimientos nuevos? ¿Y admite específicamente la reversión de un movimiento anterior a su desactivación, que es la única forma de corregir un error en su historial?]`
- **FR-035**: Una corrección (movimiento de tipo reversión) MUST registrarse siempre, aunque deje
  las existencias del producto por debajo de cero. Es la **única** excepción a FR-033 y FR-033a:
  sin ella, un error grande quedaría sin poder corregirse nunca, ya que esta feature no ofrece
  registro manual de entradas (FR-032).
- **FR-035a**: Mientras un producto tenga existencias negativas por efecto de una corrección, la
  consulta de existencias MUST destacarlo como pendiente de regularizar, y su historial MUST
  permitir identificar la corrección que lo dejó así.
- **FR-035b**: El sistema MUST definir el resto del alcance de la operación de corrección.
  `[NEEDS CLARIFICATION: C12 (resto) — ¿desde dónde la origina el emprendedor: directamente desde el historial del producto, o solo desde el módulo que originó el movimiento? ¿Se permite corregir una corrección? ¿Se permite corregir parcialmente, por menos unidades que el original? ¿Se pide un motivo y queda registrado?]`

### Functional Requirements — Transversales (derivados de la constitución)

- **FR-036**: Las cantidades de inventario y los importes MUST representarse con un tipo decimal
  exacto; MUST NOT usarse coma flotante binaria.
- **FR-037**: El vocabulario visible en pantalla MUST usar español de negocio. En particular, el
  historial de movimientos de un producto MUST NOT etiquetarse como "kardex" ni con ninguna otra
  jerga contable.
- **FR-038**: Todas las operaciones de esta feature MUST funcionar con la máquina desconectada de
  la red.
- **FR-039**: La cantidad de un movimiento MUST admitir decimales o restringirse a enteros de
  forma explícita.
  `[NEEDS CLARIFICATION: C17 — ¿se venden productos por peso o volumen (2,5 kg) o siempre en unidades enteras? Si se admiten decimales, ¿cuántos y cómo se redondea? La constitución contempla "cantidades de inventario fraccionadas", pero no las obliga.]`

### Key Entities

- **Producto**: lo que el emprendedor compra y vende. Atributos: nombre, unidad de medida, precio
  de venta, costo de referencia, estado (activo / inactivo). Nunca se elimina.
- **Movimiento de inventario**: hecho inmutable que representa una entrada o una salida de un
  producto. Atributos: producto afectado, cantidad con signo, costo unitario del momento, fecha de
  ocurrencia, fecha de registro, tipo de movimiento, referencia al documento origen y —cuando es
  una reversión— referencia al movimiento que corrige. Una vez creado no se modifica ni se borra.
- **Tipo de movimiento**: clasificación del movimiento (entrada por compra, salida por venta,
  carga inicial, reversión). Conjunto ampliable sin tocar el cálculo de existencias.
- **Referencia de origen**: identificación del documento u operación que provocó el movimiento.
  Permite volver desde el historial hasta la causa.
- **Existencias**: *no es una entidad almacenada*. Es el resultado de sumar las cantidades de los
  movimientos de un producto. Se incluye aquí para dejar explícito que no tiene representación
  propia ni ciclo de vida.
- **Saldo acumulado**: *tampoco es una entidad almacenada*. Es el valor calculado línea a línea al
  presentar el historial de un producto.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Un emprendedor sin formación contable registra un producto nuevo en menos de 60
  segundos, en una sola pantalla y sin ayuda externa.
- **SC-002**: El alta de un producto presenta **cero** campos opcionales: el emprendedor no tiene
  que decidir si rellenar algo o no.
- **SC-003**: En cualquier momento y para cualquier producto, la existencia mostrada coincide
  exactamente con la suma de las cantidades de sus movimientos, sin que el usuario haya ejecutado
  ningún proceso de recálculo, sincronización o cierre.
- **SC-004**: El número de movimientos modificados o eliminados a lo largo de toda la vida del
  sistema es **cero**: el 100% de las correcciones aparece como un movimiento adicional que
  referencia al original.
- **SC-005**: Ante una existencia que no le cuadra, el emprendedor identifica en menos de 2
  minutos qué movimientos la produjeron, usando solo la pantalla de movimientos del producto y
  sin preguntar a otra persona.
- **SC-006**: 9 de cada 10 usuarios de prueba sin formación contable completan, en el primer
  intento y sin asistencia, la secuencia: registrar un producto → cargar sus existencias
  iniciales → consultar cuántas unidades tiene.
- **SC-007**: Con 500 productos y 10.000 movimientos registrados, la consulta de existencias de
  todos los productos y el historial de un producto se muestran en menos de 2 segundos.
- **SC-008**: El 100% de las reglas de cálculo y validación de esta feature (suma de existencias,
  saldo acumulado, signo por tipo de movimiento, inmutabilidad, reversión, rechazo por saldo
  insuficiente y origen del costo unitario) está cubierto por al menos una prueba automatizada
  que se ejecuta sin arrancar la interfaz.
- **SC-009**: Ningún producto presenta existencias negativas, salvo cuando una corrección lo dejó
  así; en ese caso el 100% de esos productos aparece destacado como pendiente de regularizar en
  la consulta de existencias.

## Assumptions

Supuestos adoptados donde la descripción no decidía, **documentados aquí precisamente para que no
sean decisiones silenciosas**. Cualquiera de ellos puede rebatirse en `/speckit-clarify`.

- **Un solo usuario**: el sistema lo opera una única persona en su máquina. No hay roles, permisos
  ni auditoría de quién hizo qué, más allá de la fecha de registro del movimiento.
- **Una sola moneda**, sin conversión ni tipo de cambio. Es la consecuencia de que proveedores y
  clientes estén fuera de alcance.
- **Sin impuestos**: precio de venta y costo de referencia son cifras planas. El tratamiento de
  IVA u otros impuestos pertenece a compras y ventas, fuera de alcance.
- **Volumen esperado**: micronegocio — del orden de cientos de productos y miles de movimientos
  al año. SC-007 fija la cota concreta contra la que se verifica.
- **Todo producto controla existencias**: no hay productos sin inventario en el MVP (lo dice
  explícitamente el alcance).
- **Una sola bodega implícita**: al no haber ubicaciones, las existencias de un producto son un
  único número, sin desglose.
- **El historial de movimientos se muestra completo** por producto, sin filtros por rango de
  fechas ni paginación en el MVP. Si SC-007 no se cumpliera con el volumen esperado, esta decisión
  se revisa en `/speckit-plan`.
- **La reversión no elimina lógicamente**: tanto el movimiento original como su reversión siguen
  contando en la suma de existencias (se anulan entre sí aritméticamente) y ambos siguen visibles
  en el historial. No hay marca de "anulado" que excluya líneas del cálculo.
- **El estado del producto (activo/inactivo) no afecta al cálculo de existencias**: desactivar un
  producto no altera su saldo. Solo afecta a su disponibilidad para nuevas operaciones y,
  posiblemente, a su visibilidad en los listados (ver C15).

## Dependencies

- **Ninguna dependencia hacia otros módulos**: esta feature es la base. No consume contratos de
  nadie.
- **Módulos que dependerán de ella**: compras y ventas (aún no especificados) consumirán el
  contrato de FR-019 para registrar movimientos y consultar existencias. Por el principio III y la
  regla de "contrato primero" del flujo de trabajo, ese contrato debe quedar escrito y acordado
  con su responsable antes de que se implemente cualquier código que lo consuma.
- **Decisiones diferidas a `/speckit-plan`**: framework de interfaz y motor de persistencia. Esta
  spec no los presupone.

## Puntos abiertos — resumen

| ID | Tema | Impacto |
|----|------|---------|
| C1 | Unidad de medida: ¿lista cerrada o texto libre? | Alcance / UX |
| ~~C2~~ | ~~¿Hay registro manual de entradas y salidas en esta feature?~~ | ✅ **Resuelto** — solo carga inicial y corrección (FR-032) |
| C3 | Identificación del producto: ¿nombre único? ¿código propio? | Alcance / datos |
| C4 | ¿Qué campos son editables tras registrar un producto? | UX |
| C5 | ¿Qué campos son obligatorios en el alta (en especial el costo de referencia)? | UX / criterio de éxito |
| ~~C6~~ | ~~Salida que deja existencias negativas~~ | ✅ **Resuelto** — se rechaza; saldo nunca negativo (FR-033) |
| C7 | Movimientos y reversiones sobre productos inactivos | Contrato / datos |
| C8 | ¿La desactivación es reversible? | UX |
| C9 | Carga inicial: fecha de ocurrencia | Datos — ⚠️ **parcial**: resueltos momento, repetibilidad y costo (FR-031, FR-031a, FR-031c); solo queda la fecha |
| ~~C10~~ | ~~Costo unitario a registrar en las salidas~~ | ✅ **Resuelto** — lo aporta el originador; en su defecto, costo de referencia (FR-022) |
| C11 | Referencia de origen en carga inicial y reversión | Datos / UX |
| C12 | Origen, alcance y motivo de la corrección; corrección de una corrección | Alcance / UX — ⚠️ **parcial**: resuelto que la corrección nunca se bloquea por saldo (FR-035); el resto sigue abierto (FR-035b) |
| C13 | Fecha de ocurrencia: retroactiva, futura, granularidad | Datos |
| C14 | Desempate del orden cuando coinciden fechas | **Corrección del saldo** |
| C15 | Visibilidad de los productos inactivos en listados y existencias | UX |
| C16 | Campos y comportamiento de la búsqueda | UX |
| C17 | ¿Cantidades fraccionadas o solo enteras? | Datos |

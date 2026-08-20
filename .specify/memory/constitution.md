<!--
SYNC IMPACT REPORT
Version change: (plantilla sin rellenar) → 1.0.0
Bump rationale: ratificación inicial. El fichero previo era el scaffold con
placeholders, sin versión ni principios definidos.

Principios definidos (7 candidatos → 6 principios):
- I.   Simplicidad como restricción dura            ← candidato 1
- II.  Alcance acotado: Registrar, Consultar, Analizar ← FUSIÓN candidatos 2 + 5
- III. Fronteras explícitas y dirección de dependencias ← candidato 3 (vertical + horizontal)
- IV.  Hechos inmutables, saldos derivados          ← candidato 4
- V.   Local-first                                  ← candidato 6
- VI.  Verificabilidad de las reglas de negocio     ← candidato 7

Fusión aplicada: los candidatos 2 (Registrar/Consultar/Analizar) y 5 (MVP/YAGNI)
respondían la misma pregunta de revisión — "¿esto entra en el alcance?" — el
primero en el eje de producto y el segundo en el eje temporal. Se unifican en un
único principio con checklist común.

Secciones añadidas: Restricciones Técnicas, Flujo de Trabajo y Revisión, Gobernanza.
Secciones eliminadas: ninguna.

Plantillas desalineadas (NO modificadas por este comando):
- ⚠ .specify/templates/plan-template.md
- ⚠ .specify/templates/spec-template.md
- ⚠ .specify/templates/tasks-template.md
- ✅ .specify/templates/checklist-template.md (genérica, compatible)

TODOs diferidos:
- TODO(RATIFICATION_DATE): fecha de ratificación pendiente de confirmar por el equipo.
-->

# ERP Simple para Micronegocios — Constitución

## Core Principles

### I. Simplicidad como restricción dura

Elige siempre la solución que obliga al usuario a entender menos conceptos.

- Entre dos alternativas que satisfacen el mismo requisito, MUST elegirse la que introduce
  menos conceptos nuevos en la experiencia del usuario (menos pantallas, menos campos
  obligatorios, menos vocabulario que deba aprender). La alternativa descartada y el motivo
  MUST quedar registrados en el plan de la feature.
- Toda funcionalidad nueva MUST declarar en su spec qué conceptos nuevos expone al usuario y
  por qué son indispensables. Cero conceptos nuevos es un resultado válido y preferible.
- Los flujos primarios MUST NOT exigir conocimiento contable previo: completar un registro no
  puede requerir que el usuario entienda débito/crédito, asientos, plan de cuentas ni cierres
  de período.
- Un campo o una opción de configuración MUST NOT añadirse cuando un valor por defecto sirve
  al caso mayoritario; en su lugar se fija ese valor por defecto y se documenta.

Racional: evita el modo de falla principal del producto — que el emprendedor abandone el ERP
porque le exige entender contabilidad antes de poder registrar una venta.

### II. Alcance acotado: Registrar, Consultar, Analizar

Implementa solo aquello que sirve a una de las tres actividades y está especificado.

- Toda funcionalidad MUST declarar en su spec a cuál de las tres actividades sirve — Registrar,
  Consultar o Analizar. Si no encaja en ninguna, queda fuera del alcance por definición y
  MUST NOT implementarse.
- MUST NOT existir código sin un requisito que lo respalde en el `spec.md` de la feature: nada
  de parámetros, ganchos de extensión, capas de abstracción ni ramas "por si acaso".
- Todo lo pospuesto MUST registrarse como entrada en `backlog.md` (qué se pospone, por qué,
  qué lo desbloquearía) en lugar de codificarse a medias.
- Un módulo MUST NOT exponer en su contrato operaciones que ninguna historia de usuario del
  MVP consuma.

Racional: evita que el MVP crezca hasta no poder terminarse, y el código muerto que un equipo
de 2-3 personas trabajando en paralelo no puede mantener ni razonar.

### III. Fronteras explícitas y dirección de dependencias (NO NEGOCIABLE)

Mantén las dependencias apuntando siempre hacia el dominio.

- Monolito modular: cada módulo MUST exponer un contrato explícito (operaciones públicas y
  tipos de datos que cruzan la frontera), y un módulo MUST NOT leer ni escribir las tablas,
  ficheros ni estructuras internas de otro módulo. Toda comunicación entre módulos pasa por
  ese contrato.
- Los módulos de dominio MUST NOT importar el framework de interfaz. La dirección de la
  dependencia es interfaz → aplicación → dominio, nunca a la inversa, y ninguna capa importa
  hacia afuera.
- Las vistas MUST NOT ejecutar consultas de persistencia ni contener reglas de negocio
  (cálculos, validaciones, transiciones de estado); solo recoge entrada, invoca un caso de uso
  y presenta el resultado.
- La persistencia MUST accederse únicamente a través de repositorios cuya interfaz se declara
  en la capa de dominio o de aplicación, no en la capa de datos.

Racional: permite que 2-3 personas con agentes de código separados trabajen en paralelo sin
pisarse, y que la interfaz o el motor de datos se reemplacen sin reescribir las reglas de
negocio.

### IV. Hechos inmutables, saldos derivados

Registra los movimientos económicos como hechos que no se editan y calcula todo lo demás.

- Todo movimiento económico (venta, compra, gasto, ingreso, ajuste de inventario) MUST
  persistirse como registro inmutable, con fecha de ocurrencia y fecha de registro.
- Una operación MUST NOT modificar ni borrar un movimiento ya registrado. Corregir un error
  MUST generar un movimiento nuevo — reverso o ajuste — que referencia al original.
- Saldos, existencias e indicadores MUST derivarse por cálculo a partir de los movimientos, y
  MUST NOT existir como campos editables por el usuario.
- Si por rendimiento se introduce un valor precalculado, MUST ser reconstruible en cualquier
  momento a partir de los movimientos y MUST NOT tratarse como fuente de verdad.

Racional: evita el descuadre silencioso — la clase de error propia de Excel donde alguien
ajusta un saldo a mano y ya nadie puede reconstruir de dónde salió la cifra.

### V. Local-first

Haz que la aplicación funcione completa sin conexión y con los datos en la máquina del usuario.

- La aplicación MUST arrancar y permitir Registrar, Consultar y Analizar con la máquina
  desconectada de la red.
- Los flujos primarios MUST NOT depender de un servicio remoto, cuenta en la nube, licencia en
  línea ni autenticación externa.
- Los datos del negocio MUST almacenarse en el sistema de ficheros de la máquina del usuario,
  en una ruta conocida que él pueda copiar y respaldar.
- La aplicación MUST NOT enviar datos del negocio fuera de la máquina. Toda exportación MUST
  ser una acción explícita del usuario hacia un fichero que él elige.

Racional: el usuario objetivo trabaja con conectividad intermitente y desconfía de ceder sus
datos; además elimina de raíz una clase entera de fallos (latencia, cuentas, sincronización)
que el MVP no tiene presupuesto para manejar.

### VI. Verificabilidad de las reglas de negocio

Cubre con pruebas automatizadas toda regla de negocio, y solo ellas.

- Toda regla de negocio — cálculo, validación y transición de estado — MUST tener al menos una
  prueba automatizada que la ejercite sin arrancar la interfaz.
- MUST existir una prueba de arquitectura que falle cuando aparezca un import del framework de
  interfaz fuera de la capa de presentación.
- Las pruebas de dominio MUST NOT requerir el framework de interfaz ni una instancia real del
  motor de persistencia; se apoyan en los contratos de repositorio del principio III.
- La interfaz MUST NOT tener pruebas automatizadas obligatorias en el MVP: su verificación es
  manual y ese es un compromiso consciente, no un descuido.

Racional: convierte la frontera del principio III en algo que comprueba una máquina en lugar
de la disciplina de tres personas, y evita gastar el presupuesto de pruebas del MVP en la capa
más volátil.

## Restricciones Técnicas

- Plataforma: aplicación de escritorio en Python. La versión exacta del intérprete se fija en
  `plan.md`.
- Esta constitución NO fija el framework de interfaz ni el motor de persistencia. Ambos se
  deciden en `/speckit-plan` y quedan registrados allí. Lo que sí fija —y no es negociable— es
  la dirección de las dependencias entre capas del principio III: cualquier elección de
  framework o motor que la viole queda descartada por esa sola razón.
- Los importes monetarios MUST representarse con un tipo decimal exacto. MUST NOT usarse coma
  flotante binaria (`float`) para dinero ni para cantidades de inventario fraccionadas.
- El vocabulario visible en la interfaz MUST estar en español de negocio (venta, gasto,
  existencias) y MUST NOT usar jerga contable ni los nombres técnicos del modelo de datos.

## Flujo de Trabajo y Revisión

- Cada feature recorre `/speckit-specify` → `/speckit-plan` → `/speckit-tasks` →
  `/speckit-implement`. El `plan.md` MUST incluir el Constitution Check con un veredicto
  explícito por cada uno de los seis principios antes de empezar a implementar.
- Cada módulo MUST tener una persona responsable declarada. Un cambio en el contrato público de
  un módulo MUST acordarse con esa persona y escribirse antes de implementar el código que lo
  consume: el contrato primero, la implementación después.
- Antes de integrar, el revisor MUST recorrer las reglas MUST/MUST NOT de los seis principios
  como checklist y dejar constancia del resultado en la revisión.
- Las pruebas de dominio y la prueba de arquitectura del principio VI MUST pasar antes de
  integrar cualquier cambio.

## Governance

- Esta constitución prevalece sobre cualquier otra práctica, spec, plan, tarea o preferencia
  individual. Ante un conflicto, gana la constitución.
- **Versionado semántico** del propio documento:
  - MAJOR: se elimina un principio, se redefine de forma incompatible, o se invierte una regla
    que obligaba a reescribir código ya escrito bajo la regla anterior.
  - MINOR: se añade un principio o una sección nueva, o se amplía materialmente una guía
    existente.
  - PATCH: aclaraciones de redacción, correcciones tipográficas, refinamientos sin cambio de
    significado.
- **Procedimiento de enmienda**: (1) propuesta escrita que nombre el principio afectado, el
  problema concreto que la regla vigente causó y el texto sustituto; (2) acuerdo unánime del
  equipo — al ser 2-3 personas, cualquier objeción bloquea; (3) actualización de este fichero
  con el bump de versión, la fecha de última enmienda y un Sync Impact Report; (4) revisión de
  las plantillas de `.specify/templates/` que la enmienda deje desalineadas.
- **Cuando una spec contradice un principio**: la spec cede. La implementación de ese requisito
  se detiene y se toma una de dos vías: reescribir la spec para cumplir el principio, o —si el
  equipo concluye que el principio es el equivocado— enmendar primero la constitución por el
  procedimiento anterior y solo entonces implementar. MUST NOT implementarse un requisito que
  contradice un principio vigente alegando que es temporal o que se arregla después.
- **Excepciones documentadas**: si el equipo acepta conscientemente una violación, MUST quedar
  registrada en la tabla Complexity Tracking de `plan.md`, nombrando la alternativa más simple
  y por qué se rechazó. Una violación no registrada es un defecto bloqueante en la revisión.
- Los agentes de código y las personas MUST leer esta constitución al inicio de cada feature;
  `contexto-producto.md` aporta el contexto de producto y no sustituye a este documento.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE) | **Last Amended**: 2026-08-19

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

ERP de escritorio en Python para emprendedores y micronegocios que hoy gestionan su operación
con Excel. MVP académico de alcance deliberadamente reducido. El contexto de producto está en
[docs/contexto-producto.md](docs/contexto-producto.md).

Equipo de 2-3 personas trabajando en paralelo con agentes de código separados.

## Lee la constitución antes de escribir código

[.specify/memory/constitution.md](.specify/memory/constitution.md) es el contrato compartido del
equipo y **prevalece sobre cualquier spec, plan o preferencia**. Este fichero apunta a ella; no
duplica sus reglas. Si algo aquí la contradice, gana la constitución.

Índice de los seis principios (solo para orientarte — las reglas MUST/MUST NOT están allí):

1. Simplicidad como restricción dura
2. Alcance acotado: Registrar, Consultar, Analizar
3. Fronteras explícitas y dirección de dependencias (NO NEGOCIABLE)
4. Hechos inmutables, saldos derivados
5. Local-first
6. Verificabilidad de las reglas de negocio

Cuando una spec contradice un principio, la spec cede: se reescribe la spec, o se enmienda
primero la constitución. Las excepciones aceptadas van a la tabla Complexity Tracking del
`plan.md` de la feature; una violación no registrada es un defecto bloqueante.

## Flujo de trabajo (Spec Kit)

`/speckit-specify` → `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`

- Las specs por feature viven en `specs/<###-nombre>/` (spec.md, plan.md, tasks.md).
- `plan.md` debe incluir el Constitution Check con veredicto explícito por cada principio antes
  de implementar.
- Los scripts de [.specify/scripts/powershell/](.specify/scripts/powershell/) los invocan los
  comandos de Spec Kit; rara vez hace falta llamarlos a mano.
- Contrato primero: un cambio en el contrato público de un módulo se acuerda con su responsable
  y se escribe antes que el código que lo consume. Es lo que permite trabajar en paralelo sin
  pisarse.

### Plantillas desalineadas con la constitución

Al generar artefactos, corrige sobre la marcha estos tres puntos (detalle en el Sync Impact
Report al inicio de `constitution.md`):

- `tasks-template.md` declara los tests como opcionales — el principio VI los hace obligatorios
  para toda regla de negocio, y falta la tarea de la prueba de arquitectura.
- `spec-template.md` no tiene campo para declarar a cuál de las tres actividades sirve la
  feature ni qué conceptos nuevos expone al usuario. La feature 001 lo resuelve añadiendo una
  sección **"Encuadre según la constitución"** justo antes de las historias de usuario, con una
  tabla de actividades servidas y otra de conceptos nuevos expuestos. Replica ese patrón.
- `plan-template.md` tiene el Constitution Check genérico, sin los seis gates concretos.

## Decisiones abiertas

- **Framework de interfaz y motor de persistencia: sin decidir.** Se eligen en `/speckit-plan`.
  La constitución fija la dirección de las dependencias entre capas, no la tecnología. No los
  des por supuestos al escribir specs.
- Fecha de ratificación de la constitución: pendiente de confirmar (`TODO(RATIFICATION_DATE)`).

## Comandos

No hay build, test ni lint configurados: no existe `pyproject.toml` ni `requirements.txt`. Al
añadir las primeras pruebas —que el principio VI exige para las reglas de negocio— documenta
aquí cómo ejecutar la suite completa y cómo ejecutar una prueba suelta.

## Estado del repositorio

Mantén esta sección al día y breve. Es una foto de dónde está el proyecto, no un registro de lo
hecho: el historial pertenece a git y a los artefactos de `specs/`.

- **Feature activa**: [specs/001-inventario-productos/](specs/001-inventario-productos/) —
  catálogo de productos, libro de movimientos inmutables y existencias derivadas. `spec.md`
  escrita y clarificada; quedan **14 puntos abiertos** tabulados al final de la spec, de los que
  C14 (desempate cuando dos movimientos comparten fecha) es el único de impacto alto. Aún no hay
  `plan.md`.
- **Sin código de aplicación**: `main.py` sigue vacío.
- **No existe `backlog.md`**, que el principio II señala como destino de todo alcance pospuesto.
  Créalo cuando se posponga lo primero — el "fuera de alcance" de la feature 001 (compras,
  ventas, proveedores, lotes, múltiples bodegas, valorización) todavía no está registrado allí.
- **Repositorio git** sobre la rama `main`. No existe `.specify/extensions.yml`, así que los
  comandos de Spec Kit no crean ramas ni ejecutan hooks: se trabaja directamente sobre `main`.
- La feature activa se localiza por [.specify/feature.json](.specify/feature.json), no por el
  nombre de la rama. Si trabajas otra feature, ese fichero es el que hay que apuntar.

## Idioma

Documentación, specs y vocabulario de la interfaz en español. La interfaz usa español de
negocio (venta, gasto, existencias), nunca jerga contable.

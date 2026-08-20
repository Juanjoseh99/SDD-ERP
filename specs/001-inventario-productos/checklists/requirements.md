# Specification Quality Checklist: Catálogo de productos y libro de movimientos de inventario

**Purpose**: Validar la completitud y calidad de la especificación antes de pasar a planificación
**Created**: 2026-08-19
**Feature**: [spec.md](../spec.md)

## Content Quality

- [x] Sin detalles de implementación (lenguajes, frameworks, APIs)
- [x] Centrada en el valor para el usuario y la necesidad de negocio
- [x] Escrita para interlocutores no técnicos
- [x] Todas las secciones obligatorias completadas (más las dos que exigen los principios I y II
      y que la plantilla no contempla: actividades a las que sirve y conceptos nuevos expuestos)

## Requirement Completeness

- [ ] **No quedan marcadores [NEEDS CLARIFICATION]** — quedan 17 (C1–C17), deliberadamente, por
      instrucción explícita del usuario: "Prefiero diez marcadores a una decisión silenciosa".
      Se resuelven en `/speckit-clarify`.
- [x] Los requisitos son verificables y no ambiguos — salvo los 17 puntos marcados, que están
      señalados como abiertos en lugar de redactados de forma ambigua
- [x] Los criterios de éxito son medibles
- [x] Los criterios de éxito son agnósticos de tecnología
- [x] Los escenarios de aceptación están definidos para las cuatro historias
- [x] Los casos límite están identificados y enlazados a su punto abierto correspondiente
- [x] El alcance está acotado: la sección de entrada delimita explícitamente lo que queda fuera
- [x] Dependencias y supuestos identificados (secciones Assumptions y Dependencies)

## Feature Readiness

- [x] Todos los requisitos funcionales tienen criterio de aceptación claro o un punto abierto
      declarado
- [x] Los escenarios de usuario cubren los flujos primarios (catálogo, existencias, historial,
      corrección)
- [x] La feature satisface los resultados medibles definidos en Success Criteria
- [x] No se filtran detalles de implementación a la especificación

## Alineación con la constitución

- [x] **I. Simplicidad**: la spec declara los conceptos nuevos que expone y justifica cada uno;
      "kardex" se rechaza como etiqueta de interfaz por ser jerga contable (FR-037)
- [x] **II. Alcance acotado**: la spec declara a qué actividades sirve (Registrar y Consultar; no
      Analizar) y respeta el fuera de alcance sin añadir nada
- [x] **III. Fronteras**: FR-019 y FR-020 exigen contrato explícito para compras y ventas, y la
      sección Dependencies recoge la regla de "contrato primero"
- [x] **IV. Hechos inmutables**: FR-015, FR-016, FR-024, FR-025 y FR-026 lo cubren; SC-004 lo mide
- [x] **V. Local-first**: FR-038
- [x] **VI. Verificabilidad**: SC-008 exige cobertura automatizada de todas las reglas de cálculo
      y validación sin arrancar la interfaz

## Notes

- El único ítem incompleto es intencional. La instrucción del usuario al invocar
  `/speckit-specify` fue explícita: marcar todo lo no resuelto en lugar de inventar criterio.
  Los 17 puntos están tabulados al final de `spec.md`, priorizados por impacto.
- **Antes de `/speckit-plan` deben resolverse al menos los de impacto alto**: C2 (alcance del
  registro manual), C6 (existencias negativas), C9 (carga inicial), C10 (costo unitario en
  salidas) y C14 (orden de desempate). Los demás pueden decidirse durante la planificación si el
  equipo lo prefiere, pero deben quedar registrados.
- Ruta recomendada: `/speckit-clarify` (resuelve hasta 5 por pasada) antes de `/speckit-plan`.

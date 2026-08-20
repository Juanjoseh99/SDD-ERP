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

- [ ] **No quedan marcadores [NEEDS CLARIFICATION]** — quedan 14 tras la sesión de clarificación
      del 2026-08-19 (resueltos C2, C6 y C10; parcialmente C9 y C12). Los marcadores son
      deliberados, por instrucción explícita del usuario: "Prefiero diez marcadores a una decisión
      silenciosa".
- [x] Los requisitos son verificables y no ambiguos — salvo los 14 puntos marcados, que están
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
  Los 14 puntos que siguen abiertos están tabulados al final de `spec.md`.
- **Sesión de clarificación 2026-08-19** (5 preguntas, cupo agotado): resueltos por completo C2,
  C6 y C10; resueltos en parte C9 (falta la fecha de ocurrencia de la carga inicial) y C12 (falta
  origen, parcialidad y motivo de la corrección).
- **Sigue pendiente de impacto alto**: C14 (criterio de desempate cuando dos movimientos comparten
  fecha). Afecta al saldo acumulado que se muestra línea a línea, así que debe quedar decidido en
  `/speckit-plan` aunque no se ejecute otra pasada de clarificación.
- Los once restantes (C1, C3, C4, C5, C7, C8, C11, C13, C15, C16, C17) son de impacto medio o
  bajo y pueden decidirse durante la planificación, siempre que queden registrados allí.

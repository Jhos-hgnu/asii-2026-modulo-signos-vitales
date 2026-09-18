# Semana 6: Primer parcial y defensa tecnica

## Objetivo

La Semana 6 corresponde al primer parcial del curso. Su proposito es defender
los contenidos desarrollados para ASII-13 durante las Semanas 1 a 5, sin
introducir funcionalidad nueva.

## Alcance de la defensa

| Semana | Contenidos a defender | Evidencia |
| --- | --- | --- |
| 1 | Actores, alcance, casos de uso y UML. | [Semana 1](../semana-01-actores-alcance-y-casos-de-uso.md) |
| 2 | Requisitos funcionales y no funcionales, criterios de aceptacion y principios SOLID. | [Semana 2](../semana-02-rf-rnf-criterios-aceptacion-solid.md) |
| 3 | Vista arquitectonica, componentes y dependencias. | [Semana 3](../semana-03-vista-arquitectonica.md) |
| 4 | MVC, Repository, separacion por capas e implementacion inicial. | [Semana 4](../semana-04-mvc-repository.md) |
| 5 | Arquitectura cliente-servidor, contrato REST, integracion y analisis monolito modular frente a microservicios. | [Semana 5](../semana-05-contrato-api-e-integracion.md) |

## Estado actual de ASII-13

La implementacion inicial permite registrar signos vitales mediante:

```text
POST /api/v1/vital-signs
```

El flujo actual incluye el controlador HTTP, el servicio de registro, el
repositorio y la evaluacion de rangos para identificar valores anormales. La
decision arquitectonica vigente mantiene ASII-13 en el monolito modular Laravel;
la alternativa de microservicio solo se documenta como evolucion futura.

## Brechas conocidas

Los siguientes elementos no forman parte de la implementacion actual y no se
desarrollan durante la Semana 6:

- UI especifica para signos vitales y alertas.
- Endpoints de historial y detalle de alerta.
- RBAC especifico del modulo.
- Implementacion concreta de `VitalSignRangeProvider`.
- Normalizacion de mediciones.
- Validacion tenant-scoped.
- Consistencia transaccional.

## Criterio de evaluacion

La evidencia de esta semana es la defensa tecnica trazable a la documentacion y
al codigo entregados en las Semanas 1 a 5. No se incorporan endpoints, cambios
funcionales ni decisiones arquitectonicas nuevas en este parcial.

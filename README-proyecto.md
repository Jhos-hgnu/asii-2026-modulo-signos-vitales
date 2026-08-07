# Proyecto Final ASII 2026 — Sistema Hospitalario Integrado

Repositorio base para el **Proyecto Final de Análisis de Sistemas II 2026**. Este repositorio es una base limpia para que cada estudiante implemente un **módulo vertical individual** del Sistema Hospitalario Integrado, usando ramas, worktrees, pull requests, documentación y evidencia de pruebas.

> Este proyecto no consiste en “hacer pantallas”. Cada módulo debe demostrar análisis, diseño, arquitectura, implementación, pruebas, seguridad e integración.

## Estado actual del repositorio

| Área                | Estado real                                                                                                                                          |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stack backend       | Laravel 12, PHP 8.2+, JWT, Spatie Laravel Permission y Stancl Tenancy.                                                                               |
| Stack frontend      | Vue 3, Vite, Pinia, Vue Router y Axios.                                                                                                              |
| Seguridad base      | JWT, middleware de tenant por `X-Tenant-ID`, base RBAC con roles/permisos.                                                                           |
| Modelo clínico      | Existen modelos, factories, seeders y migraciones scaffold para pacientes, médicos, camas, admisiones, EMR, laboratorio, auditoría y notificaciones. |
| API actual          | Solo hay API de autenticación bajo `/api/v1`: registro, login, usuario actual, refresh y logout.                                                     |
| Módulos clínicos    | Pendientes: faltan controllers, endpoints, pantallas, flujos, pruebas funcionales y contratos por módulo.                                            |
| Documentación nueva | Este README, el plan semanal, la guía de worktrees y las plantillas de issues/PR son la base de trabajo ASII.                                        |

## Trabajo individual por módulos verticales

Cada estudiante debe construir una capacidad completa del HIS, no una pieza aislada. Un módulo terminado debe incluir:

- análisis del problema, actores y casos de uso;
- requerimientos funcionales y no funcionales;
- diseño arquitectónico y de componentes;
- contrato API y reglas de permisos;
- implementación backend/frontend mínima funcional;
- pruebas y evidencia de ejecución;
- documentación y evidencia de integración.

## Asignación de módulos

|   # | Estudiante                         | GitHub                | Módulo asignado                                                     |
| --: | ---------------------------------- | --------------------- | ------------------------------------------------------------------- |
|   1 | BRANDON JOSÉ AQUINO ARRIVILLAGA    | `AquinoB2090`         | Gestión de usuarios, tenants y endurecimiento de autenticación      |
|   2 | LUIS DAVID AROCHE CONTRERAS        | `Luis890D`            | RBAC: roles, permisos y protección de rutas                         |
|   3 | GLENDI PATRICIA CAMPOS ORELLANA    | `Glendi20`            | Pacientes: registro, edición, búsqueda y detalle                    |
|   4 | BILLY EDUARDO CARDONA LÓPEZ        | `pendiente-confirmar` | Médicos, especialidades y disponibilidad                            |
|   5 | EDDY ADOLFO CASTRO VÉLIZ           | `EddCastro`           | Citas médicas y agenda                                              |
|   6 | MADELIN JAZMÍN CERÓN MOLINA        | `MadelinCeron`        | Catálogo de salas, wards y camas                                    |
|   7 | OSCAR LEONEL CRUZ PAREDES          | `Leonels16`           | Admisión hospitalaria y asignación de cama                          |
|   8 | ESAÚ ABIMAEL DE LA CRUZ            | `00AbiMendoza`        | Traslados, altas y liberación de camas                              |
|   9 | JAVIER ALEXANDER FAJARDO LÓPEZ     | `Javifa2324`          | Dashboard de ocupación hospitalaria                                 |
|  10 | MARYORI RACHAEL FAJARDO PAREDES    | `maryorifajardo14`    | Expediente médico electrónico base                                  |
|  11 | JOSHUA EDUARDO GARCIA REYES        | `jgarciar73-svg`      | Notas SOAP y diagnósticos                                           |
|  12 | AXEL ELIÚ HERRERA SÁNCHEZ          | `AxelHerrera11`       | Alergias clínicas y alerta visual en expediente                     |
|  13 | JOSUÉ FERNANDO HICHO GARCÍA        | `Jhos-hgnu`           | Signos vitales y alertas por valores anormales                      |
|  14 | MARIA YAMILET LINDO PABLO          | `Yamilet1235`         | Catálogo de medicamentos                                            |
|  15 | MARÍA DE LOS ANGELES LÓPEZ FAJARDO | `Mar-03`              | Prescripciones electrónicas con validación de alergias              |
|  16 | MERCEDES AZUCENA LOPEZ PEREZ       | `Azucena17`           | Órdenes de laboratorio desde EMR                                    |
|  17 | JOSUÉ MANUEL MARTÍNEZ PEDROZA      | `Josuemart555`        | Catálogo de pruebas de laboratorio                                  |
|  18 | ARMANDO CECILIO MORALES SAGASTUME  | `ArmandoMorales`      | Recepción de muestras y códigos identificadores                     |
|  19 | KEILY FABIOLA ORELLANA MARROQUÍN   | `orem6`               | Ingreso de resultados de laboratorio                                |
|  20 | GERSON GIOVANNI ORELLANA VÉLIZ     | `gioore`              | Validación de resultados por bioquímico                             |
|  21 | DULCE MARÍA PRADO VÁSQUEZ          | `Dulce2024`           | Alertas críticas y notificaciones internas                          |
|  22 | BORIS ALEXANDER QUIROA ORELLANA    | `bquiroao`            | Gobernanza y auditoría de movimientos del sistema                   |
|  23 | ERICK ROLANDO RAMAZZINI MURALLES   | `ErickRamazzini`      | Reportes y analytics básicos                                        |
|  24 | ALBINO SEBASTIAN ROSALES RUANO     | `codsebas`            | Contratos API: OpenAPI/Postman y documentación técnica              |
|  25 | CINDY MAYTTÉ RUANO CALDERÓN        | `cindyruano`          | QA, pruebas E2E, CI y guía de despliegue final                      |
|  26 | JOSUÉ DAVID MORALES RAMÍREZ        | `morales-js`          | Dashboard base UI/UX e implementación transversal                   |
|  27 | LIS IVETTE ROSALES COLINDREZ       | `Lis671`              | Reportes operativos exportables y filtros administrativos           |
|  28 | DIDHYER ALEXANDER ORTÍZ GUEVARA    | `DidhyerOrtiz`        | QA funcional, pruebas manuales y matriz de regresión                |
|  29 | HUGO DAVID MOSCOSO CASTRO          | `HugoX2024`           | Investigación y prototipo NativePHP para extensión móvil/escritorio |

Notas de delimitación:

- El módulo de **Boris Alexander Quiroa Orellana** cubre gobernanza para super admin: bitácora/auditoría de movimientos de usuarios en el sistema, tabla `audit_logs` o ajuste documentado del scaffold existente, interfaz protegida y filtros obligatorios por usuario, módulo y rango libre de fechas (`desde` / `hasta`).
- El módulo de **Josué David Morales Ramírez** cubre el dashboard base: layout, navegación, tarjetas/resúmenes iniciales, estados vacíos/carga/error y consistencia UI/UX transversal. No reemplaza los reportes analíticos.
- El módulo de **Lis Ivette Rosales Colindrez** cubre reportes operativos exportables: filtros, tablas, impresión/PDF/CSV y reportes administrativos. Se diferencia del módulo de Erick, que se enfoca en analytics básicos y visualización de indicadores.
- El módulo de **Hugo David Moscoso Castro** es exploratorio-aplicado: evaluar NativePHP como puente Laravel hacia app móvil/escritorio y entregar un prototipo acotado o informe técnico con decisión de viabilidad.

## Flujo obligatorio con worktree

Nunca trabajes directamente sobre `main`. Cada módulo debe desarrollarse en su propio worktree y rama feature desde `origin/develop`.

```bash
git fetch origin
git worktree add ../shi-asii-XX-slug -b feature/asii-XX-slug-usuario origin/develop
cd ../shi-asii-XX-slug
```

Ejemplo de nombre:

```bash
git worktree add ../shi-asii-03-pacientes -b feature/asii-03-pacientes-glendi20 origin/develop
cd ../shi-asii-03-pacientes
```

Reglas estrictas:

- No trabajar en `main`.
- No hacer `force push`.
- No mezclar módulos diferentes en una misma rama.
- No abrir PR sin evidencia de análisis, diseño, implementación y pruebas.
- No modificar archivos de otros módulos sin coordinación previa.
- Todo PR debe apuntar a `develop`, no a `main`.

Guía detallada: [`docs/worktree-guide.md`](docs/worktree-guide.md).

## Instalación rápida Laravel/Vue

Requisitos: PHP 8.2+, Composer 2, Node.js 20+, npm y SQLite o MySQL.

```bash
composer install
cp .env.example .env
php artisan key:generate
php artisan jwt:secret
```

Para SQLite local:

```bash
touch database/database.sqlite
php artisan migrate:fresh --seed
```

Frontend y servidor local:

```bash
npm install
npm run dev
```

En otra terminal:

```bash
php artisan serve
```

Validaciones recomendadas antes de abrir PR:

```bash
php artisan route:list --path=api
npm run build
php artisan test
```

## API base disponible

Todas las rutas actuales están bajo `/api/v1` y requieren `X-Tenant-ID` cuando aplica.

| Método | Ruta             | Autenticación |
| ------ | ---------------- | ------------- |
| POST   | `/auth/register` | No            |
| POST   | `/auth/login`    | No            |
| GET    | `/auth/me`       | Bearer JWT    |
| POST   | `/auth/refresh`  | JWT refresh   |
| POST   | `/auth/logout`   | Bearer JWT    |

Los endpoints clínicos de pacientes, admisiones, EMR, laboratorio, reportes y auditoría están pendientes y se construirán por módulos.

## Plan semanal ASII

El plan completo está en [`docs/weekly-plan.md`](docs/weekly-plan.md).

| Semana | Entrega principal del módulo                         | Evidencia esperada                                                                                                                           |
| -----: | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
|      1 | Diagnóstico, actores y casos de uso del módulo.      | Diagrama UML de casos de uso + narrativa breve.                                                                                              |
|      2 | RF/RNF, criterios de aceptación y diseño inicial.    | Tabla RF/RNF + ejemplo de al menos un principio SOLID aplicado al módulo, usando como fuente `https://mvpcluster.com/diseno-de-software-2/`. |
|      3 | Vista arquitectónica del módulo.                     | Diagrama C4/UML o vista de componentes de alto nivel.                                                                                        |
|      4 | Diseño por capas y responsabilidades.                | UI, API, lógica, persistencia y objetos reutilizables.                                                                                       |
|      5 | Contrato API preliminar y plan de integración.       | Endpoints, payloads, errores, permisos, rama, worktree y PR.                                                                                 |
|      6 | Primera evaluación parcial.                          | Defensa teórica y caso práctico arquitectónico.                                                                                              |
|      7 | Diseño de componentes backend/frontend.              | Diagrama de componentes + propuesta de refactorización.                                                                                      |
|      8 | Flujo UX por rol.                                    | User flow, wireframes iniciales y reglas de interacción.                                                                                     |
|      9 | Evaluación de usabilidad y accesibilidad.            | Checklist, hallazgos y mejoras propuestas.                                                                                                   |
|     10 | Adaptación responsive/móvil.                         | Escenarios móviles y prioridades de pantalla.                                                                                                |
|     11 | Mockup o prototipo navegable.                        | Mockup desktop/móvil en Figma, Canva, Excalidraw o equivalente.                                                                              |
|     12 | Segunda evaluación parcial.                          | Defensa teórica y caso práctico de diseño.                                                                                                   |
|     13 | Plan de revisión técnica formal.                     | Checklist de revisión, responsables y evidencia.                                                                                             |
|     14 | Plan de aseguramiento de calidad.                    | Métricas, riesgos y estrategia SQA del módulo.                                                                                               |
|     15 | Pruebas unitarias, caja blanca y caja negra.         | Casos de prueba + evidencia de ejecución.                                                                                                    |
|     16 | Integración, validación y primera entrega funcional. | Evidencia en staging/on-premise y errores corregidos.                                                                                        |
|     17 | Seguridad y despliegue final.                        | Matriz de amenazas, pruebas de permisos/datos sensibles y bitácora de despliegue.                                                            |
|     18 | Evaluación final y defensa.                          | Demo funcional, PR integrado y evidencia completa.                                                                                           |

## Evaluación

| Componente                 | Puntos |
| -------------------------- | -----: |
| Actividades ASII semanales |     20 |
| Proyecto individual final  |     15 |

Proyecto individual — 15 puntos:

| Criterio                                      | Puntos |
| --------------------------------------------- | -----: |
| Funcionalidad completa del módulo             |      5 |
| Integración con el HIS y flujos existentes    |      3 |
| Calidad del análisis y diseño                 |      3 |
| Pruebas, validaciones y seguridad             |      2 |
| Documentación, demo y evidencia de despliegue |      2 |

## Definition of Done por módulo

Un módulo se considera terminado cuando cumple todo lo siguiente:

- [ ] Issue asignado y documentado.
- [ ] Rama feature creada desde `origin/develop` en worktree propio.
- [ ] RF/RNF y criterios de aceptación documentados.
- [ ] Casos de uso y alcance del módulo claros.
- [ ] Diseño arquitectónico, por capas y de componentes incluido.
- [ ] Contrato API documentado: endpoints, payloads, respuestas, errores y permisos.
- [ ] Backend implementado con controllers delgados, validaciones, servicios/actions cuando aplique y control por rol/tenant.
- [ ] UI mínima funcional en Vue para el flujo principal del módulo.
- [ ] Pruebas ejecutadas y evidencia adjunta.
- [ ] Seguridad revisada: roles, permisos, tenant y datos clínicos sensibles.
- [ ] PR abierto hacia `develop` con checklist completo.
- [ ] Evidencia de integración con otros módulos o con el flujo general del HIS.
- [ ] Demo o capturas incluidas cuando el módulo tenga interfaz.

## Documentos de apoyo

- [`docs/weekly-plan.md`](docs/weekly-plan.md): cronograma detallado de semanas 1 a 18.
- [`docs/worktree-guide.md`](docs/worktree-guide.md): flujo estricto de ramas, worktrees y recuperación.
- [Plantilla de Pull Request](.github/pull_request_template.md): evidencia obligatoria para revisión.
- [Plantilla de issue de módulo](.github/ISSUE_TEMPLATE/module_task.md): estructura para asignar y dar seguimiento a cada módulo.

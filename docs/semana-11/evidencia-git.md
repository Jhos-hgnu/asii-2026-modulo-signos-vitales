# Evidencia Git de ASII-13

## Rama

`feature/asii-13-signos-vitales-alertas-jhos-hgnu`

## Comando utilizado

```bash
git log --oneline --decorate
```

## Commits principales

```text
62ccd14 test(asii13): agregar pruebas de registro y evaluacion de signos vitales
f368dbf feat(asii13): agregar ruta api para registro de signos vitales
84332fe feat(asii13): agregar capa mvc para registro de signos vitales
e56e771 feat(asii13): integrar evaluacion de umbrales en registro
5642ae7 feat(asii13): ampliar repository para trazabilidad de alertas
ca3f97e feat(asii13): agregar proveedor de rangos de signos vitales
d250498 feat(asii13): agregar evaluador de umbrales de signos vitales
296741b feat(asii13): agregar servicio de registro de signos vitales
2e6cd6e feat(asii13): implementar repository eloquent de signos vitales
111984c feat(asii13): agregar contratos repository para signos vitales
5e7b9b4 docs(asii13): agregar especificacion arquitectonica mvc repository
```

Los hashes corresponden al historial de la rama al cierre de la implementacion inicial MVC Repository y sus pruebas unitarias.

## Semana 5: cliente-servidor, contrato API e integracion

### Contexto Git

- Modulo: `ASII-13 - Signos vitales y alertas por valores anormales`.
- Rama: `feature/asii-13-signos-vitales-alertas-jhos-hgnu`.
- Worktree: `../shi-asii-13-signos-vitales-alertas`.
- Base actualizada: `origin/develop`.
- PR objetivo: `develop`.

### Alcance de la entrega

La Semana 5 agrega solamente evidencia de analisis y diseno. No modifica rutas, controladores, servicios, repositorios, modelos, migraciones, frontend, infraestructura ni modulos de otros companeros.

Archivos de esta entrega:

- `docs/asii-13/README.md`: indice aislado de la documentacion del modulo.
- `docs/asii-13/semana-05-contrato-api-e-integracion.md`: contrato REST, seguridad, integracion, brechas y decision monolito modular.
- `docs/asii-13/semana-05-arquitectura-cliente-servidor.puml`: diagrama editable del estado actual.
- `docs/asii-13/semana-05-evolucion-arquitectura.puml`: evolucion futura marcada como no implementada.

### Decisiones y riesgos documentados

- Se mantiene ASII-13 dentro del monolito modular Laravel; no existe justificacion medible para extraer un microservicio.
- La unica ruta registrada es `POST /api/v1/vital-signs`; historial y detalle de alertas son propuestas no implementadas.
- Se documentan como brechas la ausencia de RBAC propio, el desalineamiento de `measurements`, la falta de binding de rangos, validaciones tenant-scoped incompletas y ausencia de transaccion/observabilidad propia.
- Outbox, eventos, colas, notificaciones e `Idempotency-Key` son alternativas futuras no implementadas.

### Validaciones realizadas

```bash
git fetch origin
git merge origin/develop
git status --short
git branch --show-current
git worktree list
php artisan route:list --path=api/v1/vital-signs
php artisan test --filter=VitalSign
```

Se verificaron directamente las rutas, prefijo API, middleware, controlador, Form Request, servicio, repositorio, modelo, migracion, provider de rangos, cliente Axios, permisos existentes y pruebas unitarias. PlantUML CLI no estaba disponible, por lo que los diagramas se entregan como archivos `.puml` editables sin PNG generado.

## Semana 8: diseno de experiencia de usuario

### Contexto Git

- Rama: `feature/asii-13-week-08-ux-jhos-hgnu`.
- Worktree: `../shi-asii-13-week-08`.
- Base: `wip/asii-13-continuidad-jhos-hgnu` en `c258d21`.

### Alcance de la entrega

La Semana 8 agrega exclusivamente diseno UX y evidencia documental. No crea una
interfaz Vue, no modifica el endpoint de ASII-13 ni cambia comportamiento de
backend, rutas, modelos, migraciones o modulos ajenos.

Archivos de esta entrega:

- `docs/asii-13/semana-08-diseno-experiencia-usuario.md`: roles, flujo, estados, reglas y riesgos UX.
- `docs/asii-13/semana-08-user-flow.puml`: flujo de registro y resultados alternativos.
- `docs/asii-13/semana-08-wireframes.puml`: wireframes de baja fidelidad editables.

### Validaciones de referencia

Se revisaron el plan semanal, documentacion ASII-13 de semanas 1 a 7, contrato
de semana 5, ruta API, Form Request, controlador, servicio, modelo y los
patrones Vue existentes de layout, mensajes, dialogos, estados de carga y badges.
No se generaron PNG porque PlantUML CLI no esta disponible en el entorno.

## Semana 9: usabilidad y accesibilidad

### Contexto Git

- Rama: `feature/asii-13-week-09-usabilidad-accesibilidad-jhos-hgnu`.
- Worktree: `../shi-asii-13-week-09`.
- Base de Semana 8: `feature/asii-13-week-08-ux-jhos-hgnu` en `9cfc332`.

### Alcance de la entrega

La Semana 9 evalua el diseno propuesto de Semana 8; no evalua una interfaz
ASII-13 como si estuviera implementada. Se revisan por separado los patrones
reales disponibles en el frontend general y ASII-20. No se modifican Vue,
backend, rutas, modelos, migraciones ni middleware.

Archivos de esta entrega:

- `docs/asii-13/semana-09-usabilidad-y-accesibilidad.md`: checklists, hallazgos y mejoras priorizadas.
- `docs/asii-13/semana-09-wireframes-revisados.puml`: propuesta revisada trazable a los hallazgos.

### Limitacion de integracion

Tras el fetch, `origin/develop` avanzo desde `82dc683` hasta `70fa77b` y su
historial elimina archivos de ASII-13. Por restriccion de Semana 9 no se hizo
merge: la evaluacion conserva como base el resultado terminado de Semana 8.

### Referencia de accesibilidad

Se emplea WCAG 2.1 nivel AA como referencia de diseno para criterios
seleccionados. No se reclama cumplimiento formal: no hubo auditoria completa,
pruebas con tecnologia asistiva ni una UI ASII-13 implementada.

## Semana 10: diseno para movilidad

### Contexto Git

- Rama: `feature/asii-13-week-10-movilidad-responsive-jhos-hgnu`.
- Worktree: `../shi-asii-13-week-10`.
- Base de Semana 9: `feature/asii-13-week-09-usabilidad-accesibilidad-jhos-hgnu` en `186d28f`.

### Alcance de la entrega

La Semana 10 adapta conceptualmente el flujo de registro a tablet y telefono.
No implementa CSS responsive, Vue, Tailwind, PWA, almacenamiento offline ni
cambios de backend. Los wireframes se entregan como PlantUML editable y se
marcan como propuesta no implementada.

Archivos de esta entrega:

- `docs/asii-13/semana-10-diseno-movilidad.md`: escenarios, matriz responsive, reglas tactiles y conectividad.
- `docs/asii-13/semana-10-wireframes-responsive.puml`: wireframes de tablet y telefono.

### Referencia tecnica

El proyecto incluye Tailwind v4, pero los componentes Vue existentes tambien
usan media queries locales y no hay una convencion compartida de breakpoints.
La propuesta se expresa por reflow segun el espacio disponible, sin imponer un
breakpoint global. PlantUML CLI sigue sin estar disponible; no se generaron PNG.

## Semana 11: prototipo navegable

### Contexto Git

- Repositorio grupal: [Sistema Hospitalario Integrado](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026).
- Rama: [`feature/asii-13-week-11-prototipo-navegable-jhos-hgnu`](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026/tree/feature/asii-13-week-11-prototipo-navegable-jhos-hgnu).
- Commit de referencia: [`663c163`](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026/commit/663c16367fea2afc92c5335da2b483bf2837a622).
- Base de Semana 10: `feature/asii-13-week-10-movilidad-responsive-jhos-hgnu` en `ca07d57`.

La rama y el commit de referencia se verificaron con:

```bash
git ls-remote --heads https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026.git "feature/*asii-13*"
```

### Alcance de la entrega

La Semana 11 agrega un prototipo HTML autocontenido y su documentacion. Simula
registro, validacion de interfaz, exito normal, alerta clinica y error de
red/servidor con reintento, usando datos ficticios que no persisten.

No modifica Laravel, Vue, Vite, rutas, controladores, servicios, repositorios,
modelos, migraciones, contratos ni reglas clinicas. No consume
`POST /api/v1/vital-signs`.

Archivos de esta entrega:

- `docs/asii-13/semana-11/prototipo/index.html`: prototipo navegable estatico.
- `docs/asii-13/semana-11/README.md`: alcance, trazabilidad y validaciones.
- En este repositorio personal: `docs/semana-11/README.md` y `docs/semana-11/evidencia-git.md`, que preservan la evidencia y las referencias verificables al trabajo grupal.

### Integracion y base conservada

Despues de `git fetch origin`, `origin/develop` contiene eliminaciones de
`docs/asii-13` que afectan el alcance del modulo. No se hizo merge ni rebase:
se conserva `ca07d57` como base documental completa de Semana 10.

# ADR-001: Arquitectura propuesta para ASII-13

- Estado: Aceptada e implementada parcialmente.
- Fecha: 2026-08-21.
- Alcance: Signos vitales y alertas por valores anormales.

## Contexto

El proyecto usa Laravel 12, Eloquent directo en componentes existentes, JWT con `tymon/jwt-auth`, `TenantMiddleware` basado en `X-Tenant-ID` y roles mediante Spatie. ASII-13 reutiliza `VitalSign` y `vital_signs` y cuenta con una implementacion inicial de Repository, Application, Domain, MVC, endpoint y pruebas unitarias.

La tabla existente conserva el tenant, expediente, admision opcional, usuario registrador, mediciones, `has_alert`, `alert_details` y `measured_at`. Sus indices respaldan el historial por expediente y las alertas por tenant.

## Decision

ASII-13 implementa parcialmente MVC para el limite HTTP y una separacion interna por responsabilidades:

| Capa | Responsabilidad propuesta |
| --- | --- |
| Presentation | `VitalSignController` coordina respuesta HTTP; Form Requests validan el contrato de entrada. |
| Application | Un servicio de aplicacion orquesta el contexto autenticado, persistencia, solicitud de rangos mediante contrato y evaluacion. |
| Domain | Reglas de unidades y `VitalSignRangeEvaluator` clasifican solo mediciones con umbrales aprobados. |
| Infrastructure | Implementaciones Eloquent de repositorios encapsulan consultas y escrituras tenant-scoped. |

La dependencia propuesta es `Controller -> Application Service -> Repository interface` y `Application Service -> Domain evaluator`. Las implementaciones Eloquent dependen de los modelos existentes, no al reves.

## MVC

Laravel conserva el papel de framework MVC:

- Las rutas y controladores son la entrada HTTP.
- Los Form Requests representan validacion de presentacion.
- `VitalSign` permanece como modelo Eloquent de persistencia existente.
- La vista web Vue consumira el contrato API en una fase separada.

El controlador no debe contener umbrales clinicos, filtros de tenant repetidos ni acceso Eloquent disperso.

## Repository en ASII-13

`VitalSignRepositoryInterface` actua como interfaz de aplicacion y `EloquentVitalSignRepository` como implementacion de infraestructura.

El proyecto actualmente usa Eloquent directamente y no tiene repositorios. Se introduce el patron solo para ASII-13 porque este modulo combina escrituras clinicas trazables, varias consultas con aislamiento obligatorio y reglas de dominio que deben probarse sin acoplarse a Eloquent. El repositorio centraliza el predicado obligatorio `tenant_id`, las cargas necesarias del historial y la busqueda segura de registros para alertas.

No debe convertirse en una abstraccion generica para todos los modelos ni duplicar toda la API de Eloquent. La interfaz debe contener exclusivamente operaciones del caso de uso, por ejemplo crear un registro, buscar expediente en un tenant, obtener historial por paciente y obtener una alerta por ID dentro de un tenant.

El binding interfaz-implementacion queda pendiente para uso en tiempo de ejecucion. Si el patron se extiende a otros modulos, debe conservar operaciones especificas por caso de uso en lugar de duplicar la API completa de Eloquent.

## Reutilizacion de VitalSign

Se reutilizan `App\Models\VitalSign` y `vital_signs` porque contienen todos los datos estructurales requeridos: relaciones con expediente, admision y autor; mediciones clinicas; fecha; tenant; y campos de alerta. Crear otra tabla duplicaria mediciones, fragmentaria el historial y perderia los indices existentes `idx_vs_record_time` e `idx_vs_alerts`.

No se modifica el modelo ni la tabla en esta decision. Las unidades se validaran en el contrato de entrada contra las unidades canonicas de las columnas existentes; los valores normalizados continuaran almacenandose en esas columnas.

## Alertas

ASII-13 no utiliza `critical_alerts`. Aunque dicha tabla enumera `signo_vital_anormal`, pertenece a la arquitectura de alertas criticas de laboratorio, no posee referencia a `vital_signs` y no es la fuente definida para este modulo. La alerta de ASII-13 se mantiene en el registro originador mediante `has_alert` y `alert_details`.

`alert_details` debera almacenar una estructura explicable con medicion, valor, unidad, limites, motivo y version o identificador de la regla aplicada. Si falta un umbral, la medicion queda pendiente y no se interpreta como normal.

## HOSPITAL y CENTRAL

Los datos clinicos viven en HOSPITAL y se delimitan por `tenant_id`. En el estado actual, la conexion `central` y la conexion SQLite predeterminada apuntan al mismo archivo de desarrollo; esto no elimina el limite logico entre contextos.

ASII-13 tratara el UUID de `tenant_id` como referencia logica al tenant CENTRAL. No realizara joins para trasladar datos clinicos a CENTRAL ni dependera de informacion clinica centralizada. Las referencias externas a CENTRAL deben resolverse por UUID y los datos clinicos deben persistirse y consultarse siempre bajo el hospital activo.

## Prevencion de fugas entre tenants

1. Las rutas futuras aplicaran `tenant` antes de `jwt.auth`, igual que las rutas protegidas existentes.
2. `TenantMiddleware` resolvera el tenant desde `X-Tenant-ID`; `JwtAuth` rechazara discrepancias con el usuario JWT.
3. El servicio recibira el tenant y usuario del contexto autenticado, no valores de tenant o autor enviados por el cliente.
4. Cada operacion del repositorio recibira `tenantId` y lo aplicara como condicion de consulta o escritura.
5. Antes de crear un registro se verificara que expediente, paciente y admision opcional pertenezcan al mismo tenant y tengan relacion clinica compatible.
6. El historial y detalle de alerta se buscaran por identificador y `tenant_id` simultaneamente; un ID existente de otro tenant no se debe revelar.
7. Los permisos Spatie se verificaran por accion antes de acceder a datos clinicos.

## Consecuencias

- La evaluacion de umbrales sera comprobable con pruebas unitarias sin base de datos.
- El aislamiento por tenant tendra un punto de aplicacion reutilizable en lugar de depender de disciplina en cada controlador.
- Se agregan interfaces y bindings nuevos al proyecto, por lo que su nomenclatura y alcance deben mantenerse limitados al modulo hasta que el equipo adopte una convencion transversal.
- La clasificacion clinica definitiva sigue pendiente de una fuente de umbrales aprobados y versionados.

## Implementacion realizada

- `VitalSignRepositoryInterface` define las operaciones de registro, historial, busqueda tenant-scoped y persistencia de trazabilidad de alerta.
- `EloquentVitalSignRepository` implementa el contrato con `VitalSign`, aplica `tenant_id` en consultas y persiste `has_alert` y `alert_details` cuando una capa superior lo solicita.
- `VitalSignRegistrationService` coordina la creacion del registro, solicita rangos mediante el contrato `VitalSignRangeProvider`, ejecuta la evaluacion y delega la trazabilidad al Repository.
- `VitalSignRangeEvaluator` es un componente puro de Domain que clasifica un valor como `below_min`, `normal` o `above_max`.
- `VitalSignRangeProvider` define el contrato para obtener rangos aprobados por tipo de medicion y unidad; aun no tiene una fuente de almacenamiento o configuracion concreta.
- `StoreVitalSignRequest` valida la estructura HTTP y las unidades permitidas sin consultar ni persistir datos.
- `VitalSignController` obtiene el usuario autenticado y el tenant del contexto de la solicitud, y delega el registro al servicio.
- `POST /api/v1/vital-signs` apunta al controlador y usa los middleware existentes `tenant` y `jwt.auth`.
- Las pruebas unitarias cubren limites inclusivos del evaluador y la orquestacion de registro con dobles para Repository y proveedor de rangos.

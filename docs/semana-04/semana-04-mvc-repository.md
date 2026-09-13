# ASII-13: Implementacion MVC y patron Repository

## Introduccion

La semana 4 materializa parcialmente la arquitectura definida en las semanas anteriores para ASII-13: signos vitales y alertas por valores anormales. La implementacion conserva `VitalSign` y `vital_signs` como persistencia existente y separa el limite HTTP, la orquestacion de aplicacion, las reglas de dominio y el acceso Eloquent.

## Objetivo

La separacion implementada sigue el flujo:

```text
Controller
  -> Service
  -> Repository Interface
  -> Repository Eloquent
  -> Modelo VitalSign
```

El Controller recibe la solicitud, el Service coordina el caso de uso, la interfaz desacopla la persistencia y la implementacion Eloquent opera sobre el modelo existente.

## Arquitectura implementada

### Presentation

- `StoreVitalSignRequest` valida `medical_record_id`, `measured_at`, la estructura de `measurements` y las unidades permitidas.
- `VitalSignController` obtiene usuario y tenant del contexto autenticado, delega en el servicio y retorna JSON.

### Application

- `VitalSignRegistrationService` crea el registro mediante el Repository, solicita rangos, usa el evaluador y persiste la trazabilidad cuando hay una condicion anormal.

### Domain

- `VitalSignRangeEvaluator` compara un valor con limites aprobados y retorna un resultado explicable.
- `VitalSignRangeProvider` define el contrato para obtener rangos por tipo de medicion y unidad.

### Infrastructure

- `EloquentVitalSignRepository` implementa `VitalSignRepositoryInterface` usando Eloquent y operaciones limitadas al caso de uso.

### Persistence

- `VitalSign` permanece como el modelo de persistencia clinica existente para `vital_signs`.

La vista arquitectonica de referencia se encuentra en [../semana-03/diagramas/arquitectura-asii-13.png](../semana-03/diagramas/arquitectura-asii-13.png).

## Aplicacion del patron Repository

Antes, la convencion existente del proyecto permitia que un Controller accediera directamente a Eloquent. En ASII-13, el Service depende de `VitalSignRepositoryInterface`, no de `EloquentVitalSignRepository`.

Beneficios de esta decision:

- Separa las responsabilidades de aplicacion y persistencia.
- Permite pruebas unitarias del Service con dobles del Repository.
- Centraliza las consultas y escrituras restringidas por `tenant_id`.
- Reduce el acoplamiento del caso de uso con Eloquent.

## Flujo implementado

La ruta disponible es `POST /api/v1/vital-signs`.

```text
Request
  -> Controller
  -> Service
  -> Repository
  -> VitalSign
  -> Evaluator
  -> Alert trace
```

`StoreVitalSignRequest` valida la entrada. `VitalSignController` entrega los datos validados, tenant y usuario a `VitalSignRegistrationService`. El Service persiste con el Repository, solicita rangos y evalua las mediciones disponibles. Cuando la evaluacion es anormal, el Repository actualiza `has_alert` y `alert_details` del mismo `VitalSign`.

## Seguridad multi tenant

- JWT autentica al usuario mediante el middleware `jwt.auth`.
- `TenantMiddleware` requiere `X-Tenant-ID`, resuelve el tenant y lo guarda en el contexto de la solicitud.
- `JwtAuth` rechaza solicitudes cuando el tenant del token no coincide con `X-Tenant-ID`.
- El Repository aplica `tenant_id` al consultar historial, buscar registros y actualizar trazabilidad.

## Evaluacion de umbrales

`VitalSignRangeEvaluator` devuelve uno de estos estados:

- `below_min`: valor menor al limite minimo.
- `normal`: valor entre limites, incluidos los valores iguales a minimo o maximo.
- `above_max`: valor mayor al limite maximo.

Si `VitalSignRangeProvider` no retorna un rango aprobado, la medicion no recibe clasificacion y no se genera trazabilidad de alerta.

## Evidencia de pruebas

```text
PASS Tests\Unit\VitalSigns\VitalSignRangeEvaluatorTest
PASS Tests\Unit\VitalSigns\VitalSignRegistrationServiceTest

Tests: 5 passed
Assertions: 27
```

Las pruebas usan mocks para `VitalSignRepositoryInterface` y `VitalSignRangeProvider`. El evaluador se usa como implementacion real en la prueba del Service porque es una clase `final`; el resultado persistido en la traza comprueba su participacion sin base de datos.

## Conclusion

La semana 4 deja una entrada HTTP inicial, una orquestacion de aplicacion desacoplada, un Repository tenant-scoped y un evaluador de dominio comprobado por pruebas unitarias. La decision principal es mantener Controllers delgados y hacer que el Service dependa de contratos.

La implementacion es parcial: `VitalSignRangeProvider` aun no tiene una fuente concreta de rangos aprobados ni binding para uso en tiempo de ejecucion; tambien quedan pendientes permisos especificos de ASII-13, mapeo final del payload clinico y pruebas HTTP de integracion.

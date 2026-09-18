# ASII-13: Semana 5 - Contrato API e integracion

## 1. Objetivo

Esta entrega analiza el modulo ASII-13 desde la perspectiva cliente-servidor: comunicacion HTTP/REST, contrato API, seguridad, integracion con el HIS, propiedad de datos, consistencia, resiliencia, observabilidad y la frontera potencial de un microservicio. Se basa en el codigo presente en esta rama; no crea microservicios ni modifica la logica funcional.

## 2. Estado actual comprobado

El HIS es actualmente un monolito modular Laravel. El cliente web existente usa Vue 3, Vue Router, Pinia y Axios con JavaScript; no se encontro Inertia, TypeScript ni una pantalla especifica de ASII-13. El cliente Axios toma la base desde `VITE_API_URL` o `/api/v1`, agrega `Accept: application/json` y adjunta el JWT y `X-Tenant-ID` desde `localStorage` cuando existen.

La ruta registrada de ASII-13 es `POST /api/v1/vital-signs`. El prefijo `/api/v1` se configura en `bootstrap/app.php`. La ruta aplica los middleware `tenant` y `jwt.auth`, y delega en `VitalSignController`.

El controlador obtiene el tenant resuelto y el usuario autenticado, luego llama a `VitalSignRegistrationService`. El servicio usa `VitalSignRepositoryInterface`, `VitalSignRangeProvider` y `VitalSignRangeEvaluator`; el repositorio Eloquent persiste mediante `VitalSign` y la tabla `vital_signs`.

La tabla clinica contiene `tenant_id`, `medical_record_id`, `admission_id` opcional, `registered_by`, columnas de medicion, `has_alert`, `alert_details` y `measured_at`. La propiedad logica de estos datos es HOSPITAL. En desarrollo, la configuracion puede usar una conexion compartida, pero ASII-13 mantiene el limite logico por `tenant_id`; no debe mover datos clinicos a CENTRAL.

[Diagrama editable de cliente-servidor](semana-05-arquitectura-cliente-servidor.puml)

## 3. Arquitectura cliente-servidor actual

```text
Usuario clinico
  -> cliente Vue 3 / Axios
  -> HTTP JSON /api/v1/vital-signs
  -> TenantMiddleware
  -> JwtAuth
  -> VitalSignController
  -> VitalSignRegistrationService
  -> EloquentVitalSignRepository / VitalSign
  -> base de datos clinica HOSPITAL
```

`TenantMiddleware` exige `X-Tenant-ID`, busca el tenant y lo deja en los atributos de la solicitud. `JwtAuth` autentica el Bearer JWT y rechaza la solicitud si el `tenant_id` del usuario no coincide con la cabecera. El repositorio limita las consultas de historial, busqueda y actualizacion de alerta por `tenant_id`.

El cliente general tiene soporte Axios para enviar las cabeceras, pero no se encontro un componente Vue que consuma especificamente la ruta de signos vitales. Por ello, la ruta existe en backend sin una UI ASII-13 implementada y comprobada en esta rama.

## 4. Comunicacion y dependencias

### Registro y evaluacion actuales

1. El cliente envia `medical_record_id`, `measured_at` y `measurements` al endpoint.
2. `StoreVitalSignRequest` valida que haya una o mas mediciones, fecha valida y unidades permitidas.
3. El controlador toma tenant y usuario del contexto confiable; el cliente no envia `tenant_id` ni `registered_by` para persistencia.
4. El servicio crea inicialmente un `VitalSign` sin alerta, consulta una regla por medicion y evalua los limites mediante `VitalSignRangeEvaluator`.
5. Una evaluacion fuera de rango actualiza el mismo registro con `has_alert = true` y el detalle explicable en `alert_details`.

Las dependencias reales son `MedicalRecord`, `Admission` opcional, `User`, `Tenant` y `VitalSign`. El historial del repositorio relaciona `VitalSign` con `MedicalRecord` para filtrar por paciente y tenant, aunque no existe aun una ruta HTTP de historial.

ASII-13 no usa `critical_alerts`: esa tabla pertenece al modulo ASII-21 y no hay integracion explicita entre ambos modulos en el codigo. Un consumidor futuro podria recibir una notificacion o evento de alerta, pero no existe tal publicacion en esta rama.

## 5. Contrato REST

### Cabeceras comunes

```http
Accept: application/json
Content-Type: application/json
Authorization: Bearer <jwt>
X-Tenant-ID: <uuid-del-hospital>
```

### 5.1 Registrar signos vitales - IMPLEMENTADO COMO RUTA

| Elemento | Contrato actual |
| --- | --- |
| Metodo y URL | `POST /api/v1/vital-signs` |
| Middleware | `tenant`, `jwt.auth` |
| Autenticacion | Bearer JWT obligatoria |
| Permiso especifico | No implementado; no hay middleware `permission` ni autorizacion en el Form Request |
| Body validado | `medical_record_id`, `measured_at`, `measurements` |
| Respuesta del controlador | `201 Created` con `{ "data": <VitalSign> }` |

Ejemplo de solicitud aceptada por la validacion HTTP actual:

```json
{
  "medical_record_id": 123,
  "admission_id": 45,
  "measured_at": "2026-09-10T10:30:00Z",
  "measurements": {
    "temperature": { "value": 38.5, "unit": "C" },
    "heart_rate": { "value": 110, "unit": "lpm" }
  }
}
```

Respuesta prevista por el controlador al completarse la operacion:

```json
{
  "data": {
    "id": 81,
    "tenant_id": "<tenant-del-contexto>",
    "medical_record_id": 123,
    "registered_by": 7,
    "has_alert": true,
    "alert_details": {
      "temperature": {
        "measurement_type": "temperature",
        "value": 38.5,
        "minimum": 36.0,
        "maximum": 37.5,
        "unit": "C",
        "rule_version": "<version>",
        "status": "above_max",
        "is_normal": false
      }
    },
    "measured_at": "2026-09-10T10:30:00Z"
  }
}
```

El ejemplo de respuesta muestra la forma que devuelve el controlador y una alerta posible; no confirma que una regla clinica concreta este configurada. La ruta no es verificable de extremo a extremo en el estado actual por las brechas documentadas en la seccion 10.

### 5.2 Consultar historial - PROPUESTO / NO IMPLEMENTADO

| Elemento | Contrato propuesto |
| --- | --- |
| Metodo y URL | `GET /api/v1/patients/{patient}/vital-signs` |
| Proposito | Listar los signos vitales del paciente, ordenados por `measured_at` descendente y restringidos al tenant activo. |
| Autenticacion | Bearer JWT y `X-Tenant-ID`. |
| Permiso propuesto | `vital-signs.view`. |
| Parametro | `patient`: identificador de paciente. |
| Respuesta propuesta | `200 OK` con una coleccion de `VitalSign`. |

```json
{
  "data": [
    {
      "id": 81,
      "medical_record_id": 123,
      "has_alert": true,
      "measured_at": "2026-09-10T10:30:00Z"
    }
  ]
}
```

### 5.3 Consultar detalle de alerta - PROPUESTO / NO IMPLEMENTADO

| Elemento | Contrato propuesto |
| --- | --- |
| Metodo y URL | `GET /api/v1/vital-signs/{vitalSign}/alert` |
| Proposito | Obtener `alert_details` de un signo vital del tenant activo. |
| Autenticacion | Bearer JWT y `X-Tenant-ID`. |
| Permiso propuesto | `vital-signs.alerts.view`. |
| Parametro | `vitalSign`: identificador del registro. |
| Respuesta propuesta | `200 OK` con el detalle explicable de alerta. |

```json
{
  "data": {
    "vital_sign_id": 81,
    "has_alert": true,
    "alert_details": {
      "temperature": {
        "status": "above_max",
        "value": 38.5,
        "unit": "C"
      }
    }
  }
}
```

### Codigos HTTP

| Codigo | Estado actual o comportamiento propuesto |
| --- | --- |
| `200` | Propuesto para historial y detalle de alerta. |
| `201` | Implementado por el controlador para un registro completado. |
| `400` | Implementado por `TenantMiddleware` si falta `X-Tenant-ID`. |
| `401` | Implementado por `JwtAuth` si el token es invalido o expiro. |
| `403` | Implementado por `JwtAuth` cuando token y cabecera tenant no coinciden; propuesto tambien para permisos ASII-13. |
| `404` | Implementado por `TenantMiddleware` si el tenant no existe; propuesto para paciente, expediente, admision o signo vital no encontrado dentro del tenant. |
| `409` | Propuesto para una clave de idempotencia reutilizada con contenido diferente u otro conflicto de negocio definido. No existe en ASII-13 actual. |
| `422` | Implementado por la validacion de `StoreVitalSignRequest` para entrada invalida. |
| `500` | Posible ante un fallo no controlado. No existe una respuesta de error ASII-13 personalizada. |

## 6. Seguridad y aislamiento multi-tenant

La seguridad actual se apoya en JWT y `X-Tenant-ID`. El usuario autenticado determina `registered_by`; el servicio recibe el ID del tenant y del usuario desde el controlador, no del body. Esto evita que el cliente seleccione directamente esos valores persistidos.

Spatie RBAC esta instalado y se usa en otros modulos, pero ASII-13 no tiene permisos ni middleware de permiso registrados. Los permisos `vital-signs.create`, `vital-signs.view` y `vital-signs.alerts.view` son una propuesta para una etapa de implementacion posterior, no permisos existentes.

Los datos clinicos pertenecen a HOSPITAL y se deben consultar y escribir con el `tenant_id` del hospital activo. CENTRAL puede ser una referencia logica de tenants o identidad global, pero no debe almacenar ni requerir joins remotos para los signos vitales clinicos.

## 7. Manejo de errores esperado

El contrato futuro debe rechazar datos invalidos con `422`, JWT ausente o invalido con `401`, tenant ausente con `400`, tenant desconocido con `404` y discrepancia JWT/tenant con `403`. La ausencia de permiso debe usar `403` cuando los permisos propuestos se implementen.

Antes de crear un registro, una fase posterior debe comprobar que expediente, paciente y admision opcional existen, pertenecen al tenant activo y mantienen una relacion clinica compatible. Un recurso ajeno no debe revelar su existencia: la respuesta debe ser `404` o `403` segun la convencion que adopte el HIS. Los errores internos no deben exponer datos clinicos ni detalles de infraestructura.

## 8. Consistencia, resiliencia e idempotencia

El flujo actual crea el `VitalSign` y, si se detecta anormalidad, actualiza despues `has_alert` y `alert_details`. Ambas escrituras no estan envueltas por una transaccion en ASII-13. Si la segunda escritura falla, puede quedar un registro sin la traza de alerta calculada; por ello, una implementacion posterior debe usar una transaccion local para la creacion, evaluacion y actualizacion de alerta.

El flujo se mantiene local a HOSPITAL y no necesita una llamada remota a CENTRAL para persistir el registro. Esto reduce dependencia de red y favorece la continuidad mientras la base clinica local este disponible.

`Idempotency-Key` no existe actualmente. Como propuesta futura, el cliente podria enviar esa cabecera al registrar signos vitales para asociar una solicitud repetida con una sola operacion. La clave, el hash del payload y la respuesta deberian persistirse de forma tenant-scoped; reutilizar una clave con un payload diferente podria devolver `409 Conflict`.

## 9. Observabilidad

No se encontraron logs, metricas, correlation IDs/request IDs ni auditoria especifica de ASII-13. La traza `alert_details` explica por que una medicion fue anormal y `registered_by` identifica al registrador, pero no reemplazan una observabilidad operativa.

Como mejora futura se recomiendan logs estructurados sin valores clinicos sensibles, un identificador de correlacion propagado desde el cliente, metricas de registros/anomalias/errores y una auditoria de acceso a historial y alertas. Ninguna de estas capacidades se implementa en esta semana.

## 10. Brechas y riesgos comprobados

### Brechas actuales

| Hallazgo | Estado | Riesgo | Recomendacion | Prioridad |
| --- | --- | --- | --- | --- |
| Solo existe la ruta `POST /api/v1/vital-signs`; no hay rutas HTTP para historial ni alerta. | Confirmado | La consulta definida por el modulo no esta expuesta al cliente. | Implementar y probar los endpoints propuestos en una tarea posterior. | Alta |
| No hay autorizacion RBAC especifica: `authorize()` retorna `true` y la ruta no usa `permission`. | Confirmado | Un usuario JWT del tenant podria intentar registrar sin permiso propio de ASII-13. | Definir permisos y aplicarlos en ruta/policy/Form Request. | Alta |
| `StoreVitalSignRequest` acepta `blood_pressure`, pero la tabla usa `systolic_bp` y `diastolic_bp`. | Confirmado | La presion arterial no tiene representacion consistente en el contrato. | Definir payload y normalizacion explicita para ambas columnas. | Alta |
| El servicio elimina `measurements` y no lo normaliza a las columnas del modelo. | Confirmado | Puede persistirse un registro sin las mediciones enviadas y alertarse un dato no persistido. | Normalizar y validar el payload antes de crear el modelo; cubrirlo con prueba HTTP. | Alta |
| `VitalSignRangeProvider` es una interfaz sin binding ni implementacion concreta. | Confirmado | El contenedor no puede resolver el servicio de registro en ejecucion. | Acordar una fuente de rangos aprobados e implementar/bindear el proveedor. | Alta |
| La creacion no comprueba en el servicio/repository que expediente o admision pertenezcan al tenant. | Parcialmente confirmado | Puede crearse una relacion clinica inconsistente si las FK existen pero son de otro tenant. | Validar pertenencia y compatibilidad antes de persistir. | Alta |
| Creacion y traza de alerta son dos escrituras sin transaccion ASII-13. | Confirmado | Existe riesgo de estado parcial ante fallo intermedio. | Usar una transaccion local cuando se implemente el flujo completo. | Media |
| No hay observabilidad especifica del modulo. | Confirmado | Dificulta investigar errores, latencia y volumen de alertas. | Agregar logs, correlacion y metricas sin exponer PHI. | Media |

### Mejoras futuras

| Propuesta | Estado | Beneficio | Prioridad |
| --- | --- | --- | --- |
| `Idempotency-Key` tenant-scoped | Propuesto / no implementado | Evita duplicados por reintentos de red. | Media |
| Outbox y eventos de dominio | Propuesto / no implementado | Desacopla notificaciones de la escritura clinica. | Media |
| Eventos, cola y consumidor de notificaciones | Propuesto / no implementado | Permite tratar alertas sin bloquear el registro. | Baja |

## 11. Monolito modular frente a microservicio

### Decision

**Se recomienda mantener ASII-13 dentro del monolito modular Laravel actualmente.**

El modulo ya comparte autenticacion JWT, `TenantMiddleware`, una infraestructura RBAC comun, `VitalSign`, `MedicalRecord`, `Admission`, `Patient`, `User` y la base clinica HOSPITAL. Tambien requiere consistencia local entre el registro y su alerta. En este contexto, el monolito modular reduce acoplamiento operativo, mantiene transacciones locales y evita duplicar mecanismos ya presentes.

| Opcion | Ventajas actuales | Costos o limites actuales |
| --- | --- | --- |
| Monolito modular | Reutiliza seguridad y modelos existentes; consultas y transacciones locales; despliegue simple; no requiere sincronizar datos clinicos. | Comparte ciclo de despliegue y recursos con el HIS. |
| Microservicio de signos vitales | Podria escalar y desplegarse de forma independiente en el futuro. | Requiere contratos entre servicios, autenticacion y autorizacion distribuidas, red, timeouts, reintentos, manejo de fallos parciales, observabilidad distribuida, sincronizacion de datos y consistencia eventual. |

Un microservicio no exige servidores fisicos distintos: podria ejecutarse en contenedores o infraestructura compartida. Aun asi, introduce limites de red y operacion que no resuelven una necesidad medible en el estado actual. No se encontro evidencia de alto volumen, dispositivos medicos conectados, multiples consumidores externos, despliegues independientes frecuentes ni requisitos de disponibilidad propios que justifiquen esa complejidad.

## 12. Posible extraccion futura de ASII-13

**ANALISIS FUTURO - NO IMPLEMENTADO**

La extraccion solo deberia evaluarse con metricas que demuestren alto volumen sostenido de registros, ingreso continuo desde dispositivos medicos, multiples aplicaciones consumidoras, integraciones externas, necesidad de escalar o desplegar el modulo de forma independiente, requisitos especiales de disponibilidad, procesamiento asincrono intensivo o crecimiento significativo de alertas.

La evolucion razonada seria:

```text
Monolito modular
  -> contratos internos claros y pruebas de integracion
  -> eventos de dominio
  -> Outbox transaccional local
  -> integracion asincrona con colas/consumidores
  -> posible extraccion solo si las metricas lo justifican
```

El segundo diagrama editable muestra esta frontera sin representar componentes futuros como existentes: [semana-05-evolucion-arquitectura.puml](semana-05-evolucion-arquitectura.puml).

## 13. Integracion asincrona futura

**PROPUESTA FUTURA - NO IMPLEMENTADA**

Despues de que una transaccion local persista un registro anormal y su traza, el modulo podria registrar el evento `VitalSignAlertDetected` en una outbox. Un worker o cola publicaria el evento y un consumidor de notificaciones lo procesaria sin bloquear el registro clinico:

```text
Registro de signo vital
  -> deteccion de valor anormal
  -> VitalSignAlertDetected
  -> outbox local
  -> worker / cola
  -> consumidor de notificaciones
```

Este patron no implica usar `critical_alerts` de ASII-21 ni crea una tabla, cola o servicio en esta entrega. Su objetivo es ofrecer una ruta de evolucion que mantenga el registro clinico consistente ante indisponibilidad temporal del consumidor.

## 14. Evidencia y cierre

- Rama: `feature/asii-13-signos-vitales-alertas-jhos-hgnu`.
- Worktree: `../shi-asii-13-signos-vitales-alertas`.
- PR objetivo: `develop`.
- Alcance de esta semana: analisis, contrato, diagramas y riesgos; ningun cambio funcional.
- Validaciones de referencia: inspeccion de rutas, middleware, controlador, request, servicio, repositorio, modelo, migracion, provider, frontend Axios, pruebas unitarias y estado Git.

La evidencia Git detallada de esta entrega se conserva en [evidencia-git.md](evidencia-git.md).

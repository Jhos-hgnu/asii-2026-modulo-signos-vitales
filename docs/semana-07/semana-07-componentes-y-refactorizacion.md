# Semana 7 - Componentes backend/frontend y refactorizacion

## 1. Objetivo

Analizar la implementacion existente de ASII-13, Signos vitales y alertas por
valores anormales, para identificar responsabilidades, dependencias,
acoplamiento, cohesion y oportunidades de refactorizacion. Esta semana es una
actividad de analisis y documentacion tecnica: no modifica el comportamiento
del modulo ni implementa las propuestas descritas.

## 2. Estado actual de ASII-13

La ruta registrada es `POST /api/v1/vital-signs`. Recibe un expediente clinico,
fecha de medicion y un arreglo de mediciones. La ruta esta protegida por
`tenant` y `jwt.auth`; el tenant y el usuario autenticado se toman del contexto
de la solicitud, no del cuerpo JSON.

El backend contiene controlador, Form Request, servicio de registro,
repositorio Eloquent, contrato de repositorio y evaluador de rangos. Tambien
declara el contrato `VitalSignRangeProvider`. No se encontro una implementacion
concreta ni un binding de ese contrato en `AppServiceProvider`, por lo que el
contenedor no puede resolver por si solo el flujo HTTP completo actual.

La aplicacion tiene una SPA Vue 3 general con Axios, Vue Router y Pinia. No hay
componente, pagina, ruta ni llamada Axios especificos de ASII-13.

## 3. Componentes backend existentes

| Componente | Responsabilidad | Dependencias | Observaciones |
| --- | --- | --- | --- |
| `routes/api.php` | Publicar el endpoint y aplicar middleware transversal. | `tenant`, `jwt.auth`, `VitalSignController`. | No aplica middleware `permission` para ASII-13. |
| `TenantMiddleware` | Exigir `X-Tenant-ID`, cargar el tenant y dejarlo en la solicitud. | `Tenant`. | Aisla el contexto HTTP; no valida por si mismo el expediente ni la admision recibidos. |
| `JwtAuth` | Autenticar el Bearer JWT y comprobar coincidencia entre usuario y tenant de la cabecera. | JWTAuth, usuario autenticado. | Proteccion transversal reutilizable. |
| `StoreVitalSignRequest` | Validar estructura, fecha, tipos de medicion y unidades permitidas. | Laravel validation, `Rule`. | `authorize()` retorna `true`; no hay RBAC propio. Sus unidades son una constante local. |
| `VitalSignController` | Obtener tenant y usuario del contexto, delegar el registro y responder `201`. | Form Request, `VitalSignRegistrationService`, autenticacion API. | Es delgado y no contiene reglas clinicas. Asume que middleware resolvio tenant y usuario. |
| `VitalSignRegistrationService` | Orquestar alta, consulta de rangos, evaluacion y traza de alerta. | Repositorio, `VitalSignRangeProvider`, `VitalSignRangeEvaluator`. | Depende de contratos para repositorio y rangos. Crea primero y actualiza la alerta despues. |
| `VitalSignRangeEvaluator` | Clasificar un valor contra minimo y maximo y producir traza explicable. | Ninguna. | Clase final, pura y cohesionada; tiene pruebas unitarias de limites. |
| `VitalSignRangeProvider` | Declarar la consulta de un rango aprobado por tipo y unidad. | Ninguna. | Es solo una interfaz; no existe implementacion concreta ni binding actual. |
| `VitalSignRepositoryInterface` | Definir persistencia tenant-scoped, historial, busqueda y actualizacion de alerta. | `VitalSign`, colecciones Eloquent. | El contrato esta orientado al agregado; combina lectura y escritura, sin justificar aun contratos adicionales. |
| `EloquentVitalSignRepository` | Persistir y consultar `VitalSign` filtrando por tenant. | Eloquent `VitalSign`. | Implementa el contrato y valida que el tenant no sea vacio. No valida que expediente/admission sean del tenant antes de crear. |
| `VitalSign` | Representar el registro clinico y sus relaciones. | `MedicalRecord`, `Admission`, `User`. | Persiste columnas de medicion, `has_alert` y `alert_details`. |
| `Patient`, `MedicalRecord`, `Admission` | Relacionar signo vital con paciente, expediente y admision opcional. | Modelos Eloquent relacionados. | `MedicalRecord` tiene muchos signos; `Patient` los expone mediante `hasManyThrough`. |
| `AppServiceProvider` | Resolver el contrato de repositorio a Eloquent. | Contenedor Laravel, repositorio. | Tiene binding de `VitalSignRepositoryInterface`; falta el de `VitalSignRangeProvider`. |
| Pruebas unitarias existentes | Comprobar evaluacion de limites y orquestacion del servicio con mocks. | PHPUnit, mocks. | No hay prueba HTTP/feature de la ruta ni prueba de integracion de persistencia ASII-13. |

## 4. Flujo actual del registro de signos vitales

El flujo comprobado, en relacion con el contrato de Semana 5, es:

```text
Cliente API o SPA general
  -> POST /api/v1/vital-signs
  -> TenantMiddleware (X-Tenant-ID)
  -> JwtAuth (Bearer JWT y tenant del usuario)
  -> StoreVitalSignRequest (estructura, fecha, unidades)
  -> VitalSignController (tenant y usuario del contexto)
  -> VitalSignRegistrationService
  -> EloquentVitalSignRepository::createForTenant
  -> VitalSign / tabla vital_signs
  -> VitalSignRangeProvider::findApprovedRange por medicion
  -> VitalSignRangeEvaluator::evaluate
  -> EloquentVitalSignRepository::updateAlertTraceForTenant si hay anormalidad
```

El orden real crea inicialmente un registro con `has_alert = false` y despues
actualiza `has_alert` y `alert_details` si el evaluador encuentra un valor fuera
de rango. La consulta al provider ocurre despues de la primera escritura.

`has_alert` se determina cuando alguna evaluacion retorna un estado distinto de
`normal`. La traza guarda valor, limites, unidad, version de regla y estado.
Si no hay rango aprobado, la medicion queda sin clasificacion.

La ruta no queda operativa de extremo a extremo mientras `VitalSignRangeProvider`
carezca de implementacion y binding. Ademas, el request acepta un objeto
`measurements`, pero el servicio lo elimina antes de persistir y no contiene un
mapeo explicito desde ese objeto a las columnas del modelo. En particular,
`blood_pressure` no se normaliza a `systolic_bp` y `diastolic_bp`. Estas son
brechas verificadas, no refactorizaciones aplicadas en esta semana.

El diagrama editable de los componentes reales se encuentra en
[semana-07-componentes.puml](diagrams/semana-07-componentes.puml).

## 5. Evaluacion de separacion por capas

La capa HTTP separa middleware, validacion y controlador. El controlador no
accede directamente a Eloquent ni evalua rangos. El servicio coordina el caso
de uso, mientras que el repositorio concentra los detalles de persistencia y el
evaluador concentra la regla aritmetica de limites.

La separacion es adecuada para el alcance inicial, pero queda incompleta en dos
puntos. Primero, la capa de aplicacion recibe arreglos HTTP sin un normalizador
o DTO que los traduzca a los atributos persistibles. Segundo, el contrato de
rangos pertenece al dominio pero no tiene adaptador de infraestructura ni
registro en el contenedor. No se recomienda introducir mas capas genericas ni
abstracciones adicionales antes de resolver estas brechas concretas.

## 6. Evaluacion SOLID

| Principio | Estado | Evidencia | Mejora propuesta |
| --- | --- | --- | --- |
| Single Responsibility Principle | Cumple parcialmente | Controlador, evaluador y repositorio tienen responsabilidades reconocibles. El servicio mezcla orquestacion con la interpretacion de un arreglo de mediciones sin normalizar. | Extraer una normalizacion de mediciones cuando se implemente la correccion del payload. |
| Open/Closed Principle | Cumple parcialmente | El evaluador no necesita cambios para evaluar otro rango, pero `StoreVitalSignRequest` declara tipos y unidades en una constante local. | Centralizar el catalogo o una politica de unidades cuando se requieran nuevos tipos. |
| Liskov Substitution Principle | No aplica | Solo existe una implementacion de repositorio y ninguna del provider de rangos. | Evaluar sustitucion cuando exista un provider concreto y sus pruebas de contrato. |
| Interface Segregation Principle | Cumple parcialmente | `VitalSignRangeProvider` es pequeno y especifico. El repositorio agrupa alta, historial, busqueda y actualizacion de alerta del mismo agregado. | Mantener el contrato actual; dividirlo solo si aparecen consumidores con necesidades independientes. |
| Dependency Inversion | Cumple parcialmente | El servicio depende de `VitalSignRepositoryInterface` y `VitalSignRangeProvider`; el repositorio se bindea a Eloquent. | Implementar y bindear un adaptador concreto de `VitalSignRangeProvider`. |

## 7. Componentes frontend

### Estado actual: no implementado

No existe una interfaz Vue/Inertia especifica de ASII-13. La aplicacion usa Vue
3 y Axios general; Axios ya adjunta `Authorization` y `X-Tenant-ID` desde
`localStorage`. El router solo declara las rutas generales de inicio y login,
sin pantalla de signos vitales. No se crea frontend en esta semana.

### Propuesta futura

Los siguientes son nombres propuestos, no componentes existentes:

| Componente propuesto | Responsabilidad | Datos e interaccion API | Motivo de separacion |
| --- | --- | --- | --- |
| `VitalSignForm` | Coordinar el registro de un conjunto de mediciones. | Recibiria `medical_record_id`, admision opcional y fecha; enviaria `POST /api/v1/vital-signs`. | Separa el flujo de envio del detalle de cada medicion. |
| `VitalSignMeasurementFields` | Capturar valores y unidades por tipo de medicion. | Recibiria catalogo de tipos y valores iniciales; entregaria el objeto `measurements` al formulario. | Reduce duplicacion y concentra reglas de presentacion por campo. |
| `VitalSignHistory` | Presentar registros historicos cuando exista el endpoint correspondiente. | Recibiria identificador de paciente y coleccion de signos; consumiria un historial futuro. | Separa lectura longitudinal de la captura. |
| `VitalSignAlertBadge` | Mostrar de forma accesible si un registro tiene alerta. | Recibiria `has_alert` y resumen no sensible de `alert_details`. | Evita repetir la semantica visual de alerta en listas y detalle. |
| `VitalSignDetails` | Mostrar un registro y su traza explicable. | Recibiria un `VitalSign`; podria usar un endpoint de detalle futuro. | Mantiene separada la visualizacion clinica del formulario. |

## 8. Oportunidades de refactorizacion

| Hallazgo | Impacto | Refactor propuesto | Prioridad |
| --- | --- | --- | --- |
| No hay implementacion ni binding de `VitalSignRangeProvider`. | El contenedor no puede construir el servicio para la ruta HTTP. | Implementar un provider de rangos aprobados y registrarlo contra su interfaz. | Necesaria |
| El payload `measurements` no se mapea a las columnas de `VitalSign`; `blood_pressure` no se traduce a sistolica/diastolica. | Pueden persistirse registros sin las mediciones recibidas o evaluar datos no persistidos. | Definir un normalizador o DTO de entrada y ajustar contrato, reglas y pruebas juntos. | Necesaria |
| No se comprueba que `medical_record_id` ni `admission_id` pertenezcan al tenant y sean clinicamente compatibles. | Riesgo de relacion entre recursos de tenants distintos o inconsistentes. | Validar pertenencia y relacion antes de persistir, manteniendo la consulta tenant-scoped. | Necesaria |
| Alta y actualizacion de alerta son dos escrituras independientes. | Un fallo intermedio puede dejar un registro sin su traza de alerta. | Envolver el caso de uso completo en una transaccion local al implementar el flujo definitivo. | Necesaria |
| La ruta solo requiere JWT; `StoreVitalSignRequest::authorize()` permite a cualquier usuario autenticado del tenant. | No hay control de permiso especifico para registrar signos vitales. | Definir permiso/policy ASII-13 y aplicarlo en ruta, policy o Form Request. | Recomendable |
| Errores de provider, recursos clinicos o persistencia no tienen traduccion especifica del modulo. | Respuestas de error poco consistentes y menor trazabilidad del fallo. | Definir excepciones de aplicacion y mapeo HTTP al implementar validaciones clinicas. | Recomendable |
| Solo hay pruebas unitarias de servicio y evaluador. | No se comprueba el ensamblaje de middleware, request, bindings y Eloquent. | Agregar pruebas feature e integracion despues de completar los componentes faltantes. | Recomendable |
| No hay historial, detalle de alerta, idempotencia ni eventos. | Limitacion funcional y operativa, no defecto de separacion actual. | Priorizar despues de estabilizar el registro transaccional y tenant-scoped. | Futura/opcional |
| No hay observabilidad especifica. | Dificulta diagnosticar volumen, errores y alertas. | Agregar logs estructurados, correlacion y metricas sin exponer datos clinicos. | Futura/opcional |

## 9. Refactorizaciones que NO se recomienda realizar

- No extraer `VitalSignRangeEvaluator`: es una clase pura, pequena, cohesionada y
  cubierta por pruebas de limites.
- No dividir todavia `VitalSignRepositoryInterface` en multiples repositorios:
  sus operaciones siguen perteneciendo al agregado `VitalSign` y no hay
  consumidores independientes comprobados.
- No introducir un bus de eventos, outbox, cola o microservicio para resolver el
  registro actual. Primero se deben resolver rangos, normalizacion, aislamiento
  tenant-scoped y transaccion local.
- No crear una UI, composables o stores de Vue antes de que los contratos de
  historial, detalle y normalizacion esten definidos e implementados.
- No trasladar reglas de rango al controlador ni al repositorio; el evaluador y
  el contrato de provider ya ofrecen una frontera razonable para esa logica.

## 10. Relacion con el endpoint REST

`POST /api/v1/vital-signs` mantiene el contrato de Semana 5. Intervienen la
ruta protegida, los middleware tenant/JWT, `StoreVitalSignRequest`,
`VitalSignController`, `VitalSignRegistrationService`, el provider de rangos,
con el modelo resultante al completarse la operacion.

La validacion HTTP cubre presencia, forma y unidades permitidas. La seguridad
transversal valida cabecera tenant, JWT y coincidencia tenant-usuario. La
validacion de que expediente y admision pertenezcan al tenant, la normalizacion
de mediciones y la consistencia de las dos escrituras no estan implementadas;
son propuestas documentadas, no comportamientos actuales.

## 11. Riesgos

- La ausencia de provider concreto de rangos impide resolver el servicio desde
  el contenedor en el flujo HTTP real.
- La diferencia entre el contrato `measurements` y las columnas persistidas
  puede producir datos clinicos incompletos o alertas sobre valores no guardados.
- La falta de validacion tenant-scoped de expediente y admision puede afectar el
  aislamiento clinico si se reciben identificadores no compatibles.
- La falta de transaccion puede dejar `has_alert` y `alert_details` sin la
  consistencia esperada ante un fallo entre escrituras.
- La falta de RBAC ASII-13 y de pruebas HTTP deja sin cobertura la autorizacion
  fina y el ensamblaje completo del endpoint.

## 12. Propuesta de evolucion

La secuencia recomendada es:

```text
1. Definir y bindear la fuente de rangos aprobados.
2. Acordar el payload y normalizar measurements a columnas persistibles.
3. Validar expediente/admission dentro del tenant y su relacion clinica.
4. Hacer atomico el registro y la traza de alerta.
5. Agregar RBAC y pruebas feature/integracion.
6. Exponer historial y detalle de alerta cuando el backend sea consistente.
7. Implementar UI especifica sobre contratos estabilizados.
```

Esta evolucion conserva el monolito modular y sus limites actuales. Una posible
extraccion de componentes solo debe reconsiderarse si existen requerimientos
medibles de escala, dispositivos conectados o despliegue independiente.

## 13. Conclusion

ASII-13 ya separa adecuadamente el transporte HTTP, la orquestacion, la
persistencia y la evaluacion aritmetica de rangos. El principal trabajo futuro
no es aumentar abstracciones, sino completar los contratos que ya existen y
cerrar las brechas de normalizacion, tenant, transaccion, autorizacion y
pruebas de extremo a extremo. La Semana 7 deja estas decisiones trazables sin
introducir cambios funcionales.

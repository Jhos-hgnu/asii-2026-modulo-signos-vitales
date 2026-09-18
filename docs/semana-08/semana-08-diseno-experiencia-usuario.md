# Semana 8 - Diseno de experiencia de usuario

## 1. Objetivo

Disenar la experiencia del flujo principal de ASII-13 para registrar signos
vitales, comunicar el resultado y distinguir errores de entrada de una alerta
clinica. Esta entrega es de analisis y diseno: no implementa una pantalla Vue ni
modifica el backend.

## 2. Contexto del modulo

| Aspecto | Estado comprobado |
| --- | --- |
| Registro API | **EXISTENTE:** `POST /api/v1/vital-signs`, protegido por `tenant` y `jwt.auth`. |
| Validacion | **EXISTENTE:** exige `medical_record_id`, `measured_at` y al menos una medicion con su unidad permitida. |
| Resultado | **EXISTENTE EN EL CONTRATO:** el controlador responde `201` con el `VitalSign`, que puede incluir `has_alert` y `alert_details`. |
| Pantalla ASII-13 | **NO IMPLEMENTADA:** no hay pagina, componente, ruta Vue ni llamada Axios especifica. |
| Historial y detalle de alerta | **PROPUESTOS / NO IMPLEMENTADOS:** sus endpoints fueron definidos en Semana 5, pero no existen rutas HTTP. |
| Evaluacion clinica completa | **BRECHA EXISTENTE:** el provider de rangos no tiene implementacion ni binding; el flujo no esta comprobado de extremo a extremo. |

El diseno se limita al registro. La seleccion de paciente y el contexto clinico
son necesarios para iniciar el flujo, pero su fuente de datos no se inventa:
debe integrarse con el modulo propietario de paciente/expediente cuando exista
un contrato estable.

## 3. Roles involucrados

| Rol | Estado | Necesidad UX en este flujo |
| --- | --- | --- |
| Enfermera | **DOCUMENTADO:** actor principal de CU-01. | Identificar al paciente, capturar mediciones, corregir errores y confirmar el resultado. |
| Medico | **DOCUMENTADO:** consulta historial y alertas en CU-02/CU-04. | Revisar posteriormente la condicion anormal y su detalle cuando existan las rutas y UI necesarias. |
| Administrador | **DOCUMENTADO CONDICIONAL:** participa segun permisos. | No se le asigna un flujo propio ni permiso concreto en este diseno. |
| Sistema | **DOCUMENTADO:** evalua valores despues del registro. | Mostrar la clasificacion devuelta sin sustituir el juicio clinico. |

Los permisos `vital-signs.create`, `vital-signs.view` y
`vital-signs.alerts.view` siguen siendo **PROPUESTOS** de Semana 5. Actualmente
ASII-13 no aplica RBAC propio; por ello los estados de falta de autorizacion se

## 4. Necesidades del usuario

| Necesidad | Respuesta de diseno |
| --- | --- |
| Confirmar que se registra para el paciente correcto | Cabecera persistente de contexto con nombre, identificador y expediente; no permitir editar esos datos dentro del formulario. |
| Capturar rapidamente datos disponibles | Campos agrupados por medicion, unidad visible y solo una medicion como minimo, tal como exige la validacion actual. |
| Corregir datos sin perder el trabajo | Errores junto al campo, resumen al inicio y conservacion de los valores no invalidos. |
| Saber si se guardo | Estado de envio, boton bloqueado y mensaje de confirmacion tras `201`. |
| Distinguir una alerta de un fallo | La alerta aparece solo despues de un guardado exitoso y no usa el mismo tratamiento visual ni verbal que un error. |
| Conocer el siguiente paso | Resultado normal permite volver al contexto; resultado anormal pide seguir el protocolo clinico local y ofrece revisar el detalle cuando exista esa capacidad. |

## 5. Flujo UX principal

El diagrama editable es [semana-08-user-flow.puml](semana-08-user-flow.puml).

```text
Inicio / contexto clinico
  -> paciente y expediente identificados
  -> abrir "Registrar signos vitales"
  -> completar fecha/hora y una o mas mediciones
  -> validar
  -> enviar POST /api/v1/vital-signs
  -> 201 Created
  -> resultado normal o resultado con alerta clinica
```

La entrada al formulario y el resumen de paciente son **PROPUESTOS**: no existe
una ruta Vue ASII-13 ni una integracion de seleccion de paciente comprobada. El

## 6. Flujo normal

1. La enfermera llega desde un contexto clinico que ya identifica paciente y expediente.
2. La vista propuesta presenta el contexto y el acceso a "Registrar signos vitales".
3. Se muestra fecha/hora de medicion y grupos de campos opcionales para las mediciones admitidas por el request.
4. La enfermera ingresa al menos una medicion y revisa las unidades fijas mostradas.
5. El sistema valida formato, campos requeridos y unidades antes de enviar; la validacion de servidor sigue siendo la fuente de verdad.
6. Al enviar, se muestra progreso y se deshabilita la accion para evitar reintentos involuntarios.
7. Tras `201 Created`, se muestra una confirmacion con fecha/hora y resultado.
8. Si `has_alert` es falso, el resultado es "Registro guardado sin alerta reportada". Si es verdadero, se presenta una alerta clinica no bloqueante con el detalle recibido.

El texto "sin alerta reportada" no afirma que todos los valores son normales:
el backend puede no clasificar una medicion si no hay rango aprobado.

## 7. Estados alternativos y errores

| Situacion | Clasificacion | Respuesta propuesta | Base |
| --- | --- | --- | --- |
| Campo obligatorio vacio | Error de entrada | Marcar el campo, explicar la correccion y mantener el formulario. No enviar. | `medical_record_id`, `measured_at` y `measurements` son requeridos. |
| Formato, valor o unidad invalida | Error de entrada | Mensaje junto al campo y resumen; conservar los otros datos. No enviar o mostrar `422` del servidor. | Reglas de `StoreVitalSignRequest`. |
| Valor fuera de rango aceptable | Alerta clinica, no error de entrada | Permitir enviar si la estructura es valida; luego mostrar alerta solo si la respuesta registrada trae `has_alert`. | Semana 5 y servicio evaluador. |
| Paciente o contexto no encontrado | Error de contexto | Informar que el contexto ya no esta disponible y ofrecer volver a la busqueda/expediente propietario. No exponer datos de otro tenant. | **PROPUESTO:** validacion tenant-scoped pendiente. |
| Sesion invalida o expirada | Error de autenticacion | Informar que la sesion no es valida y dirigir a acceso; no reintentar automaticamente el POST. | `401` documentado. |
| Falta de autorizacion | Error de autorizacion | Informar acceso no autorizado sin revelar datos clinicos; ofrecer retorno seguro. | `403` existe por tenant; RBAC ASII-13 es propuesto. |
| Error de servidor | Error de operacion | Informar que no se confirmo el guardado y habilitar reintento consciente con los datos conservados. | `500` posible; sin respuesta ASII-13 personalizada. |
| Error de comunicacion | Error de comunicacion | Mostrar que no se pudo confirmar el envio, conservar datos y permitir reintento manual. | **PROPUESTO:** no existe idempotencia. |

No se debe mostrar una alerta clinica como error de formulario ni borrar los
caida de red: sin `Idempotency-Key` existe incertidumbre sobre el resultado del
POST y el usuario debe reintentar de forma consciente.

## 8. Retroalimentacion de valores anormales

| Elemento | Propuesta UX |
| --- | --- |
| Momento | Despues de una respuesta exitosa que incluya `has_alert: true`; no antes ni como bloqueo de entrada. |
| Mensaje | "Registro guardado. Se detectaron valores fuera del rango configurado." |
| Detalle | Mostrar tipo de medicion, valor, unidad, estado y limites solo cuando lleguen en `alert_details`. |
| Diferenciacion | Contenedor de alerta con icono y texto; no usar solo color ni el estilo de error de validacion. |
| Siguiente accion | "Sigue el protocolo clinico local"; el acceso al detalle/historial se marca **PROPUESTO** hasta que existan endpoint, UI y permisos. |
| Limite clinico | No definir rangos numericos en UX. Los valores y rangos provienen de la evaluacion del servidor cuando exista una regla aprobada. |

## 9. Wireframes iniciales

Los wireframes de baja fidelidad editables estan en
[semana-08-wireframes.puml](semana-08-wireframes.puml). Son **PROPUESTAS**, no
pantallas implementadas ni especificacion visual final.

| ID | Vista propuesta | Elementos principales |
| --- | --- | --- |
| W1 | Contexto del paciente | Paciente, expediente, admision opcional y accion de registro. |
| W2 | Formulario | Fecha/hora, mediciones con unidad y boton de guardar. |
| W3 | Errores de validacion | Resumen y mensajes asociados a fecha/hora o medicion. |
| W4 | Exito sin alerta | Confirmacion de `201`, fecha/hora y retorno al contexto. |
| W5 | Exito con alerta | Confirmacion y bloque de alerta clinica con detalle recibido. |
| W6 | Ayuda contextual | Diferencia entre error de entrada y alerta clinica; unidades y siguiente paso. |

## 10. Reglas de interaccion

| Regla | Comportamiento propuesto | Trazabilidad |
| --- | --- | --- |
| Contexto persistente | Mostrar paciente, expediente y admision si existe; no capturarlos como texto libre. | `medical_record_id` requerido; `admission_id` opcional. |
| Campos obligatorios | Solicitar fecha/hora y una o mas mediciones; cada medicion contiene valor y su unidad. | `StoreVitalSignRequest`. |
| Validacion | Validar al abandonar un campo y al enviar; presentar el resultado del servidor como definitivo. | `422` documentado. |
| Errores | Usar mensaje cercano al campo, resumen y foco en el primer error; no usar solo color. | **PROPUESTO UX.** |
| Envio | Habilitar cuando el formulario tiene estructura local valida; durante envio cambiar etiqueta a "Guardando..." y bloquear doble envio. | **PROPUESTO UX.** |
| Reintento | Ante error de red/servidor conservar valores y requerir accion explicita para reintentar. | Sin idempotencia actual. |
| Confirmacion | Mostrar solo tras `201`; incluir resultado y nunca asumir que una alerta equivale a error de guardado. | Controlador y contrato de Semana 5. |
| Alerta clinica | No bloquear por si misma un formulario estructuralmente valido; explicar que se registro y mostrar detalle devuelto. | Servicio marca `has_alert` despues de evaluar. |
| Navegacion | Cancelar o volver conserva contexto de paciente, pero advierte antes de descartar datos editados. | **PROPUESTO UX.** |

## 11. Ayudas al usuario

| Punto de ayuda | Contenido propuesto |
| --- | --- |
| Encabezado de paciente | Explica que el registro se asociara al expediente mostrado y que debe verificarse antes de guardar. |
| Campos de medicion | Presenta la unidad aceptada por el API junto al campo, sin exigir que el usuario la memorice. |
| Mensaje de error | Indica que se debe corregir formato, campo faltante o unidad antes de guardar. |
| Mensaje de alerta | Aclara que el valor se guardo y que la condicion requiere seguir el protocolo clinico local. |
| Error de red | Aclara que no hay confirmacion del guardado y permite reintento manual. |

## 12. Componentes existentes que podrian reutilizarse

| Elemento existente | Uso posible | Limite |
| --- | --- | --- |
| `AppLayout.vue` | Estructura de cabecera, navegacion y contenido. | Aun no enlaza ASII-13. |
| Cliente Axios | Envia JWT y `X-Tenant-ID` desde almacenamiento local. | No tiene llamada ASII-13. |
| `LabResultsPage.vue` | Patron de `aria-busy`, carga, error, retroalimentacion y dialogo de confirmacion. | Pertenece a ASII-20; no es un componente compartido de ASII-13. |
| `ResultStatusBadge.vue` | Referencia de badge textual con etiqueta accesible. | Sus estados son de laboratorio y no deben reutilizarse con semantica clinica sin adaptacion. |

## 13. Elementos propuestos/no implementados

| Elemento | Estado |
| --- | --- |
| Ruta, pagina y menu de signos vitales | **PROPUESTO / NO IMPLEMENTADO.** |
| Selector o contexto de paciente integrado | **PROPUESTO / NO IMPLEMENTADO.** |
| `VitalSignForm`, campos de medicion y badge de alerta | **PROPUESTOS / NO IMPLEMENTADOS.** |
| Historial y detalle navegable de alertas | **PROPUESTOS / NO IMPLEMENTADOS:** dependen de endpoints pendientes. |
| Permisos UI para crear, ver y revisar alertas | **PROPUESTOS / NO IMPLEMENTADOS.** |
| Prevencion robusta de duplicados por idempotencia | **PROPUESTA FUTURA / NO IMPLEMENTADA.** |

## 14. Riesgos de UX

| Riesgo | Impacto UX | Mitigacion de diseno |
| --- | --- | --- |
| `measurements` no se normaliza a las columnas persistidas | La interfaz podria confirmar datos que no se persisten como se espera. | No presentar este diseno como flujo funcional hasta corregir contrato, normalizacion y pruebas. |
| No hay provider de rangos enlazado | Una alerta puede no estar disponible en el flujo real. | Basar el mensaje solo en respuesta confirmada; no anticipar clasificaciones. |
| Falta validacion de expediente/admission por tenant | Riesgo de contexto clinico incorrecto. | Mostrar contexto y requerir integracion tenant-scoped antes de implementar UI. |
| Dos escrituras sin transaccion | Resultado confirmado podria carecer de traza de alerta ante fallo intermedio. | Informar solo el estado recibido y resolver consistencia backend antes de UI productiva. |
| Sin idempotencia | Reintentos pueden duplicar registros. | Bloquear doble clic, conservar datos y pedir reintento manual consciente. |
| Alertas solo por color | Usuarios pueden no distinguir una condicion clinica. | Icono, texto explicito y detalle textual; evaluar accesibilidad en Semana 9. |

## 15. Conclusiones

La Semana 8 entrega un flujo UX trazable para la enfermera, con continuidad para
la revision clinica del medico cuando las capacidades pendientes existan. El

La siguiente implementacion no debe empezar por una pantalla aislada: primero se
7. Por ello esta entrega conserva todos los componentes visuales como propuesta.

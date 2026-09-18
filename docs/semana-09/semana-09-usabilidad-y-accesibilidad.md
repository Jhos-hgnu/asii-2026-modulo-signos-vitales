# Semana 9 - Usabilidad y accesibilidad

## 1. Objetivo

Evaluar el diseno UX propuesto para ASII-13 en Semana 8, identificar problemas
interfaz. Esta entrega es documental; no modifica codigo ni declara una
conformidad formal de accesibilidad.

## 2. Artefactos evaluados

| Artefacto | Tipo de evaluacion | Estado |
| --- | --- | --- |
| `semana-08-diseno-experiencia-usuario.md` | Roles, reglas, errores y ayudas propuestos. | Diseno propuesto. |
| `semana-08-user-flow.puml` | Camino normal y estados alternativos. | Diseno propuesto. |
| `semana-08-wireframes.puml` | W1 a W6: contexto, formulario, errores, exito y ayuda. | Diseno propuesto. |
| `AppLayout.vue` | Navegacion y estructura general. | Componente real, sin entrada ASII-13. |
| `LabResultsPage.vue` y componentes ASII-20 | Referencia real de labels, estados, alertas, dialogos y responsive. | No pertenece a ASII-13. |

No existe pagina, ruta, llamada Axios o componente Vue de ASII-13. Por tanto,
la evaluacion de ASII-13 es del **DISENO PROPUESTO**. Los componentes de
laboratorio solo se observan como patrones reales que podrian orientar una
implementacion futura.

## 3. Metodologia de revision

Se revisaron el plan de Semana 9, los artefactos de Semana 8, el contrato de
registro y los patrones visuales existentes. Se aplicaron heuristicas practicas
accesibilidad se usaron criterios relevantes de WCAG 2.1 AA como referencia de

Estados de la checklist:

- **Cumple:** el artefacto define la condicion de forma suficiente para el diseno.
- **Cumple parcialmente:** la intencion existe, pero falta especificacion verificable.
- **Requiere mejora:** el diseno deja una ambiguedad que puede perjudicar el uso.
- **No aplica:** no corresponde al flujo o depende de una UI inexistente.

## 4. Checklist de usabilidad

| Criterio | Estado | Evidencia y evaluacion |
| --- | --- | --- |
| Proposito claro de la pantalla | Cumple | W1 y W2 identifican ASII-13 y la accion de registrar. |
| Identificacion del paciente y contexto | Cumple parcialmente | W1 propone paciente, ID y expediente, pero no define que hacer ante identificadores largos, homonimos o contexto desactualizado. |
| Agrupacion logica de campos | Cumple parcialmente | W2 lista mediciones, pero no separa visualmente captura, fecha/hora y acciones ni define grupos para facilitar el escaneo. |
| Etiquetas comprensibles | Cumple | Las mediciones y unidades usan terminologia documentada para el rol clinico. |
| Campos obligatorios | Cumple parcialmente | Fecha/hora usa `*` y se indica una medicion minima; falta una leyenda visible que explique el asterisco y la obligatoriedad de valor/unidad por medicion agregada. |
| Prevencion de errores | Cumple parcialmente | Se muestran unidades y se propone validacion previa; faltan restricciones de entrada y ejemplos no clinicos de formato. |
| Mensajes de validacion | Cumple parcialmente | W3 distingue el error y conserva datos; falta definir el foco en el primer error y la asociacion visible entre resumen y campo. |
| Mensaje de exito | Cumple | W4 y W5 confirman que el registro se guardo solo despues de `201`. |
| Error tecnico frente a alerta clinica | Cumple | Semana 8 y W3/W5 los distinguen en texto, momento y consecuencia. |
| Consistencia de botones y acciones | Cumple parcialmente | Existen acciones de cancelar, guardar y volver, pero no hay jerarquia visual ni regla comun para la accion primaria. |
| Cancelar o regresar | Cumple parcialmente | Se propone preservar contexto y advertir datos no guardados, pero W2 no muestra esa advertencia. |
| Retroalimentacion durante envio | Cumple parcialmente | Se propone "Guardando..." y bloqueo; el wireframe no lo representa ni define el estado de boton. |
| Prevencion de doble registro | Cumple parcialmente | Bloquear doble clic reduce el riesgo, pero la ausencia de idempotencia deja incertidumbre ante red interrumpida. |
| Carga cognitiva | Requiere mejora | Ocho mediciones se muestran a la vez sin jerarquia, agrupacion ni indicacion de que solo se capturan las disponibles. |
| Visibilidad del estado | Cumple parcialmente | Hay estado de exito y error, pero no se especifica carga accesible ni el resultado de una medicion sin rango aprobado. |
| Terminologia clinica para el rol | Cumple | Enfermera es el rol primario y los textos no introducen rangos clinicos inventados. |
| Consistencia con sistema existente | Cumple parcialmente | El frontend real usa labels, mensajes y dialogos, pero ASII-13 no tiene un sistema de componentes compartidos definido. |

## 5. Checklist de accesibilidad

WCAG 2.1 AA es una referencia de diseno. Los estados no implican que la
aplicacion cumpla WCAG: para eso harian falta implementacion, pruebas de teclado,
lector de pantalla, contraste medido y auditoria completa.

| Criterio | Referencia | Estado | Evaluacion |
| --- | --- | --- | --- |
| Etiquetas accesibles y asociacion label/input | 1.3.1, 3.3.2 | Requiere mejora | W2 escribe nombres de campos, pero no especifica `label` asociado a cada control. |
| Orden logico de navegacion | 2.4.3 | Cumple parcialmente | El orden visual es razonable; no existe orden de foco ni retorno desde mensajes o dialogos. |
| Navegacion por teclado | 2.1.1 | Requiere mejora | El diseno no define operacion por teclado para acciones, ayuda o retorno. |
| Foco visible | 2.4.7 | Requiere mejora | Ningun wireframe define indicador de foco. |
| Contraste suficiente | 1.4.3 | Requiere mejora | Wireframes sin paleta ni contraste verificable. |
| No depender solo del color | 1.4.1 | Cumple parcialmente | La alerta exige icono y texto, pero no se define tratamiento equivalente para error, exito y foco. |
| Errores asociados al campo | 3.3.1 | Cumple parcialmente | W3 pone texto cerca del campo, pero no define relacion programatica ni anuncio al lector. |
| Texto e instrucciones comprensibles | 3.3.2 | Cumple | Ayuda y mensajes distinguen correccion de entrada frente a alerta clinica. |
| Estructura semantica | 1.3.1 | Requiere mejora | No se especifican encabezados, `form`, `fieldset`/`legend`, resumen ni regiones de estado. |
| Botones con nombre claro | 2.4.6 | Cumple | Guardar, cancelar, volver y registrar otra medicion describen su accion. |
| Alertas perceptibles | 4.1.3 | Requiere mejora | Se pide icono y texto, pero falta una region de estado o alerta anunciable. |
| Tamano tactil razonable | Referencia de diseno | Cumple parcialmente | Aun no hay medidas; debe definirse antes de movilidad en Semana 10. |
| Zoom y reflow | 1.4.10 | No aplica | No existe layout implementado que pueda probarse. |
| Lectores de pantalla | 4.1.2, 4.1.3 | Requiere mejora | Faltan nombres, descripciones, invalidacion y anuncios programaticos propuestos. |
| Ayudas accesibles | 3.3.5 | Cumple parcialmente | W6 tiene contenido claro, pero no define su activacion, foco o relacion con el campo. |

## 6. Evaluacion de formularios

W2 reduce el error al mostrar unidades junto a cada medicion y el contrato exige
una o mas mediciones. Sin embargo, el listado plano de ocho mediciones aumenta
la carga cognitiva y no deja claro que una medicion activa requiere valor y
unidad. La propuesta revisada agrupa la captura, declara el minimo y usa un
grupo semantico por medicion en la futura implementacion.

El `StoreVitalSignRequest` existente es la fuente de verdad para estructura,
antes de recibir una respuesta exitosa.

## 7. Evaluacion de errores

W3 ya comunica que un error de entrada impide enviar y propone conservar datos.
Falta hacer verificable el comportamiento: enfocar el primer campo invalido,
anunciar un resumen y enlazar cada mensaje con su control. Estos cambios se
plasman como **PROPUESTA REVISADA - NO IMPLEMENTADA** en WR1.

Los errores de red y servidor deben comunicar que no se confirmo el guardado,
no que "fallo" con certeza. Como no existe idempotencia, el reintento debe ser
manual y consciente para evitar duplicados.

## 8. Evaluacion de alertas clinicas

W5 tiene la distincion principal correcta: primero confirma el guardado y luego
declara una alerta clinica. Esta alerta no debe impedir el registro si la entrada
es valida ni usar solo rojo para comunicar la condicion. Falta definir una region
anunciable, un titulo persistente y una accion posterior que no prometa un
protocolo clinico local y condiciona cualquier enlace a capacidades futuras.

## 9. Navegacion y teclado

Como referencia real, `RejectResultDialog.vue` de ASII-20 devuelve foco y limita
Tab dentro de su dialogo. ASII-13 no posee dialogos ni interfaz implementada, por
lo que no se atribuye esa capacidad al modulo. Para la futura UI se propone:

- Orden de Tab: contexto, fecha/hora, mediciones activadas, ayuda, cancelar y guardar.
- Indicador `:focus-visible` distinguible del estado de error y alerta.
- Tras error de envio, foco en el resumen; tras validacion local, foco en el primer campo invalido.
- Tras exito o alerta clinica, foco en el titulo del resultado para anunciar el cambio de pantalla.
- Si se usa un dialogo de descarte, foco atrapado y retorno al activador.

## 10. Contraste y comunicacion visual

El diseno no dispone de colores ni medidas, por lo que no se puede medir
contraste. La implementacion futura debe verificar contraste de texto y estados,
combinar etiqueta textual, icono o forma y color.

`LabResultsPage.vue` usa `role="status"`, `role="alert"`, labels visibles y
mensajes de carga; son patrones reales reutilizables como idea, no evidencia de
que ASII-13 ya los implemente.

## 11. Hallazgos

| ID | Categoria | Hallazgo | Evidencia | Impacto | Prioridad |
| --- | --- | --- | --- | --- | --- |
| H1 | Accesibilidad | Falta especificar labels asociados, grupos semanticos y descripciones para los campos. | W2 presenta texto junto a cajas sin semantica definida. | Un lector de pantalla no tendria relaciones verificables entre medicion, unidad, ayuda y error. | Alta |
| H2 | Prevencion de errores | El formulario presenta ocho mediciones planas sin agrupacion ni estrategia de captura progresiva. | W2. | Aumenta carga cognitiva y probabilidad de omitir o interpretar mal campos. | Alta |
| H3 | Accesibilidad | No se define foco visible, orden de teclado ni movimiento de foco tras cambios de estado. | W2-W5 y user flow. | Personas que usan teclado pueden perder contexto al corregir, guardar o recibir una alerta. | Alta |
| H4 | Retroalimentacion | La alerta clinica necesita una region anunciable y una accion posterior limitada a capacidades existentes. | W5 confirma alerta, pero no define anuncio ni limite de accion. | La alerta puede pasar inadvertida o prometer un detalle no implementado. | Alta |
| H5 | Usabilidad | La advertencia por descarte y el estado "Guardando..." se describen, pero no estan representados. | Reglas de Semana 8; W2. | El usuario puede perder datos o volver a enviar por incertidumbre. | Media |
| H6 | Accesibilidad | Error y exito no tienen requisito explicito de texto, icono y contraste, equivalente al de alerta clinica. | W3-W5. | Los estados pueden depender del color o ser ambiguos para parte de la audiencia. | Media |
| H7 | Navegacion | El contexto de paciente no indica tratamiento para homonimia, identificador largo o dato desactualizado. | W1. | Riesgo de que la enfermera no detecte un contexto incorrecto antes de registrar. | Media |
| H8 | Consistencia | No existe especificacion de componentes compartidos ASII-13, aunque hay patrones en frontend general. | `AppLayout.vue`, ASII-20 y Semana 8. | La futura UI puede divergir en botones, mensajes y dialogos. | Baja |

Las prioridades se justifican por el efecto en completar un registro correcto y
entender su resultado, no por severidad clinica. H1-H4 deben resolverse antes de
implementar la interfaz, porque afectan acceso, prevencion de errores y lectura

## 12. Priorizacion

| Grupo | IDs | Criterio |
| --- | --- | --- |
| Necesarias antes de implementar UI | H1, H2, H3, H4 | Definen semantica, carga de captura, navegacion accesible y comprension del resultado. |
| Recomendables | H5, H6, H7 | Refinan confianza, comunicacion visual y verificacion de contexto. |
| Futuras | H8 | Conviene resolverlo al crear los componentes, sin bloquear esta documentacion. |

## 13. Mejoras propuestas

| Prioridad | Mejora | Problema que resuelve | Esfuerzo estimado cualitativo |
| --- | --- | --- | --- |
| Alta | Definir `label`, `fieldset`/`legend`, ayuda y relacion de error por medicion. | H1; permite interpretar controles y errores con tecnologia asistiva. | Medio |
| Alta | Reagrupar formulario: contexto fijo, fecha/hora y mediciones activables con unidades visibles. | H2; reduce carga cognitiva sin inventar rangos clinicos. | Medio |
| Alta | Especificar orden de teclado, foco visible y destino de foco en error, exito y alerta. | H3; mantiene orientacion al usar teclado. | Bajo |
| Alta | Añadir titulo, icono, texto y region de estado para alerta clinica; enlazar detalle solo si existe. | H4; distingue alerta de error sin prometer funcionalidades. | Medio |
| Media | Mostrar estado de envio y confirmacion antes de permitir otro envio; advertir al descartar datos. | H5; reduce doble envio y perdida de captura. | Bajo |
| Media | Definir tokens visuales verificables para error, exito, alerta y foco. | H6; evita dependencia exclusiva de color y prepara revision de contraste. | Medio |
| Media | Mostrar identificador estable y accion de volver a fuente de paciente si el contexto cambia. | H7; mejora verificacion antes de guardar. | Bajo |
| Baja | Documentar convenciones locales para boton primario, mensajes y dialogos al crear ASII-13. | H8; reduce inconsistencias futuras. | Bajo |

El wireframe revisado
[semana-09-wireframes-revisados.puml](semana-09-wireframes-revisados.puml)
traza H1-H7. Es una **PROPUESTA REVISADA - NO IMPLEMENTADA** y no reemplaza los
wireframes originales de Semana 8.

## 14. Limitaciones de la evaluacion

- No existe UI ASII-13 implementada para probar con teclado, zoom, contraste o lector de pantalla.
- WCAG 2.1 AA se uso como referencia de diseno; no se realizo auditoria formal.
- Los rangos clinicos y la evaluacion de alertas no se validaron como flujo completo porque el provider de rangos sigue sin implementacion/binding.
- `origin/develop` posterior a la base de continuidad elimina archivos de ASII-13; por restriccion de esta semana no se hizo merge y la evaluacion conserva `9cfc332`.
- Los patrones de ASII-20 son evidencia de componentes reales externos al modulo, no una implementacion transferida a ASII-13.

## 15. Conclusiones

Semana 8 ya resuelve la distincion conceptual clave: error de entrada no es una
alerta clinica. Semana 9 muestra que esa diferencia necesita detalles
implementables de semantica, foco, anuncio de estado y agrupacion para que sea
comprensible y accesible en la practica.

Antes de crear la UI se deben atender H1-H4 y estabilizar las brechas de backend
checklists, hallazgos y propuestas sin modificar el comportamiento del sistema.

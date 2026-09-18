# Semana 10 - Diseno para movilidad

## 1. Objetivo

Adaptar conceptualmente el flujo de registro de signos vitales de ASII-13 a
tablet y telefono, manteniendo la separacion entre error de entrada y alerta
clinica definida en Semanas 8 y 9. La entrega contiene escenarios, reglas y
wireframes; no implementa una interfaz responsive real.

## 2. Base UX utilizada

| Artefacto | Aporte a movilidad |
| --- | --- |
| Semana 8: user flow y wireframes | Define registro, confirmacion, errores y alerta clinica. |
| Semana 9: H1-H4 | Requiere semantica de campos, captura progresiva, foco y alerta anunciable antes de implementar UI. |
| Semana 9: H5-H7 | Guian estado de envio, comunicacion visual y verificacion del contexto en pantalla pequena. |
| `POST /api/v1/vital-signs` | **EXISTENTE:** unico endpoint de ASII-13 para registrar; requiere tenant y JWT. |
| Pantalla ASII-13 | **NO IMPLEMENTADA:** no hay pagina, ruta o llamada Vue especifica. |

El proyecto tiene Tailwind v4, pero no una convencion documentada de
breakpoints para la SPA: `LabResultsPage.vue` usa reglas locales a 800 px y
otros componentes usan 560 px o 767 px. Esta propuesta evita establecer un
breakpoint global nuevo; describe el reflow por el espacio disponible.

## 3. Restricciones del contexto clinico movil

- La enfermera es el rol documentado que registra signos vitales.
- El paciente, expediente y admision opcional deben conservarse visibles sin permitir su edicion en el formulario.
- Solo se deben solicitar las mediciones disponibles; el request exige una o mas, fecha/hora y unidad aceptada.
- Un valor clinicamente anormal no es un error de entrada y no bloquea el envio si la estructura es valida.
- La alerta solo se comunica despues de una respuesta exitosa que incluya `has_alert`.
- No hay UI ASII-13, historial HTTP, detalle de alerta, idempotencia ni modo offline implementados.

## 4. Escenarios moviles

| Escenario | Contexto, rol y dispositivo | Objetivo y pasos principales | Riesgos | Comportamiento esperado |
| --- | --- | --- | --- | --- |
| A. Tablet junto al paciente | Enfermera, tablet en portrait o landscape junto a cama. | Verificar contexto, abrir registro, capturar mediciones disponibles y guardar. | Paciente equivocado, interrupcion, pantalla compartida. | Contexto persistente, captura en una o dos columnas segun espacio y accion primaria visible. |
| B. Telefono en pantalla reducida | Enfermera, telefono en portrait. | Registrar una o mas mediciones con teclado virtual. | Controles pequenos, scroll largo, teclado cubre accion. | Una columna, grupos activables y barra de accion al final sin ocultar contexto ni errores. |
| C. Registro normal | Enfermera, tablet o telefono. | Enviar una entrada valida y recibir `201` sin alerta reportada. | Confundir exito con una evaluacion clinica completa. | Confirmar solo el guardado y decir "sin alerta reportada", no afirmar que todos los valores son normales. |
| D. Valor anormal registrado | Enfermera, tablet o telefono. | Enviar entrada valida y recibir `has_alert: true`. | Confundir la alerta con error tecnico o no verla tras scroll. | Mostrar confirmacion primero y bloque clinico con icono, texto y detalle recibido; indicar protocolo clinico local. |
| E. Error de validacion | Enfermera, tablet o telefono. | Intentar guardar con campo vacio, dato invalido o unidad no aceptada. | Error fuera del viewport o perdida de datos. | Mantener valores, mostrar resumen, desplazar/focalizar el primer campo invalido y no enviar mientras haya error local. |
| F. Conectividad o servidor | Enfermera, tablet o telefono con red inestable. | Intentar enviar y no recibir confirmacion. | Duplicado por reintento, creer que se guardo. | Conservar datos, comunicar que el guardado no fue confirmado y permitir reintento manual; sin modo offline. |

## 5. Flujo principal en tablet

1. Mostrar una franja fija de contexto con paciente, identificador estable y expediente.
2. Presentar fecha/hora y mediciones en dos columnas solo cuando el ancho permita que cada control y su unidad sigan legibles y operables.
3. Usar grupos activables por medicion para reducir carga cognitiva, segun H2 de Semana 9.
4. Mantener una accion primaria "Guardar signos vitales" visible al final del formulario; no depender de hover.
5. En landscape, conservar el contexto arriba y no convertir la alerta clinica en un panel lateral que pueda pasar inadvertido.
6. En portrait, volver a una columna antes de comprimir etiquetas, unidades o botones.

## 6. Flujo principal en telefono

1. Encabezado compacto con nombre, identificador y expediente; el identificador estable permanece junto al nombre para reducir homonimia.
2. Una sola columna con fecha/hora y mediciones activables, evitando ocho entradas expuestas simultaneamente.
3. Cada medicion activada despliega valor, unidad visible y ayuda; el control de valor solicita teclado numerico cuando sea compatible.
4. Si el teclado virtual cubre la accion principal, se debe permitir desplazamiento hasta el boton sin perder el resumen de errores.
5. Tras error local o del servidor, foco y viewport se dirigen al resumen o primer campo afectado, sin borrar lo ingresado.
6. Tras `201`, reemplazar el formulario por resultado normal o alerta clinica, con foco en su titulo.

## 7. Propuesta responsive

| Elemento | Escritorio | Tablet | Telefono |
| --- | --- | --- | --- |
| Formulario | Puede aprovechar mayor ancho; no se implementa en esta semana. | Una o dos columnas solo con controles completos. | Una columna. |
| Mediciones | Grupos visibles y activables. | Grupos activables; dos columnas solo si no reducen tactilidad. | Una medicion activa por bloque vertical. |
| Contexto paciente | Cabecera persistente. | Franja superior compacta. | Encabezado compacto, identificador estable visible. |
| Navegacion | Menu general. | Acciones directas y retorno visible. | Retorno y ayuda sin depender de menu desplegable. |
| Accion principal | Al final del formulario. | Al final, accesible tras scroll. | Al final del flujo; no debe quedar oculta permanentemente por teclado virtual. |
| Mensajes | Cerca del campo y resumen. | Resumen visible; enlazado al campo. | Resumen arriba y mensaje junto al control; desplazar al primer error. |
| Alerta clinica | Resultado amplio. | Bloque completo posterior a confirmacion. | Bloque de ancho completo; titulo, icono y texto antes del detalle. |
| Ayudas | Texto contextual. | Ayuda expandible sin hover. | Boton etiquetado que expande ayuda y conserva foco. |

No se prescribe una cifra de breakpoint: esa decision debe alinearse con la
convencion que se acuerde al implementar los componentes. La regla propuesta es
pasar a una columna antes de que etiquetas, unidades, foco o controles tactiles
dejen de ser utilizables.

## 8. Wireframes

Los wireframes editables estan en
[semana-10-wireframes-responsive.puml](semana-10-wireframes-responsive.puml).
Todos se identifican como **PROPUESTA RESPONSIVE - NO IMPLEMENTADA**.

| ID | Vista | Trazabilidad |
| --- | --- | --- |
| MR1 | Tablet, formulario de registro. | H1, H2, H3 y H7. |
| MR2 | Telefono, formulario de registro. | H1, H2, H3 y H5. |
| MR3 | Telefono, errores de validacion. | H1, H3, H5 y H6. |
| MR4 | Tablet/telefono, exito sin alerta. | H3, H5 y H6. |
| MR5 | Tablet/telefono, alerta clinica. | H3, H4 y H6. |

## 9. Reglas de interaccion tactil

| Regla | Propuesta |
| --- | --- |
| Controles | Definir objetivos tactiles claramente separados; como referencia de diseno, buscar un minimo aproximado de 44 por 44 CSS px cuando se implemente. |
| Hover | No usar hover como unico acceso a ayuda, errores, acciones o estado. |
| Foco | Mantener `:focus-visible` distinguible de error y alerta; no eliminar el foco para usuarios de teclado externo. |
| Teclado virtual | Solicitar teclado numerico cuando sea compatible con el valor, sin reemplazar validacion del servidor. |
| Error | Mostrar resumen y texto junto al campo; mover foco y viewport al primer error sin ocultar el contexto. |
| Accion primaria | Etiqueta explicita; durante envio usar "Guardando..." y deshabilitar reenvio involuntario. |
| Contexto | Mantener paciente, identificador y expediente visibles o recuperables sin editar los datos. |
| Rotacion | Reordenar sin perder datos, foco ni estado; portrait usa una columna y landscape solo amplifica el espacio. |
| Scroll | Nunca ocultar una alerta clinica o accion principal sin indicador; evitar paneles laterales obligatorios en pantalla pequena. |
| Color | Error, exito, alerta y foco combinan texto, icono o forma y color. |

## 10. Errores y retroalimentacion

| Estado | Mensaje y comportamiento movil propuesto |
| --- | --- |
| Error de formulario | "Corrige los campos indicados antes de guardar." Resumen anunciable, foco en primer error y mensaje junto a cada control. No se envia. |
| Enviando | "Guardando signos vitales..." Estado perceptible y accion deshabilitada para evitar doble envio. |
| Registro exitoso sin alerta | "Registro guardado. No se reporto una alerta en la respuesta." Ofrecer retorno al contexto o nuevo registro. |
| Registro exitoso con alerta | Confirmar guardado y luego mostrar alerta clinica con titulo, icono, texto y detalle recibido. |
| Error de servidor o red | "No se pudo confirmar el guardado." Conservar datos y permitir reintento manual; no afirmar que el registro no existe. |

## 11. Alertas clinicas en pantallas pequenas

La alerta clinica aparece despues de la confirmacion, no dentro del mismo estilo
que el error de entrada. Debe ocupar el ancho disponible, empezar con un titulo
visible y anunciable, incluir icono y texto, y mostrar solo el detalle que llegue
historial o detalle que todavia no existe.

El siguiente paso propuesto es "Siga el protocolo clinico local" y volver al
contexto. Esto mantiene clara la condicion sin convertir la interfaz en un motor

## 12. Accesibilidad movil

- Implementar en el futuro labels asociados, grupos semanticos y ayuda relacionada con cada medicion, conforme a H1.
- Mantener orden de foco vertical y predecible, incluso con teclado externo en tablet.
- Usar regiones de estado para carga, confirmacion y alerta clinica, conforme a H4.
- Permitir zoom y reflow sin cortar etiquetas, unidades, resumen de errores o acciones.
- No depender solo de color, vibracion o una posicion visual para comunicar estados.
- Los dialogs de descarte, si se crean, deben devolver foco al activador; el patron real de ASII-20 es solo referencia, no funcionalidad ASII-13.

WCAG 2.1 AA sigue siendo referencia de diseno, no declaracion de cumplimiento
formal: no existe UI ASII-13 para pruebas de contraste, teclado o lector de
pantalla.

## 13. Consideraciones de conectividad

El cliente existente usa Axios y el endpoint de ASII-13 es HTTP. No hay cache,
cola local, PWA, sincronizacion diferida ni `Idempotency-Key` implementados.
Por ello, ante red inestable se conservan los valores en pantalla solo durante
la sesion de la vista y se solicita reintento manual consciente.

**PROPUESTO / NO IMPLEMENTADO:** evaluar idempotencia tenant-scoped y una
estrategia de continuidad clinica en una etapa posterior. No se asume que el
registro pueda funcionar sin conexion.

## 14. Mejoras futuras

| Mejora | Estado |
| --- | --- |
| Implementar H1-H4 antes de UI ASII-13. | Necesaria antes de implementar. |
| Definir componentes compartidos para campos, resumen, estado y alerta. | Recomendada. |
| Acordar convencion responsive de la SPA. | Recomendada. |
| Probar con tablet real, telefono, teclado externo y lector de pantalla. | Recomendada antes de liberar. |
| Idempotencia y estrategia de conectividad. | Propuesta futura / no implementada. |

## 15. Elementos no implementados

- Pagina, ruta, componentes Vue y CSS responsive de ASII-13.
- Cambios en Tailwind, backend, endpoint, middleware, modelo o reglas clinicas.
- Modo offline, PWA, almacenamiento local, cola de sincronizacion e idempotencia.
- Historial y detalle de alerta navegables.
- Breakpoints globales nuevos para el proyecto.

## 16. Conclusiones

La movilidad no cambia el contrato ni la regla clinica de ASII-13: adapta la
captura para que una enfermera pueda verificar contexto, registrar una o mas
mediciones y entender el resultado desde tablet o telefono. La prioridad es
preservar orientacion, tactilidad, foco y retroalimentacion dentro de una sola
columna cuando el espacio lo requiera.

Esta propuesta incorpora los hallazgos de Semana 9 sin implementar codigo. Una
UI futura debe resolver primero H1-H4 y las brechas backend ya documentadas antes

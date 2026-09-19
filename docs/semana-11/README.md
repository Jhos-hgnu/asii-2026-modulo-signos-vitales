# Semana 11 - Prototipo navegable

> **Trazabilidad del trabajo:** la evidencia implementada del modulo ASII-13 se
> conserva en el repositorio grupal
> [Sistema Hospitalario Integrado](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026),
> rama [`feature/asii-13-week-11-prototipo-navegable-jhos-hgnu`](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026/tree/feature/asii-13-week-11-prototipo-navegable-jhos-hgnu),
> commit [`663c163`](https://github.com/compilations-teams/sistema-hospitalario-integrado-SistenasII-2026/commit/663c16367fea2afc92c5335da2b483bf2837a622).

Este repositorio personal conserva la evidencia individual de la Semana 11;
la referencia anterior permite comprobar su procedencia en el trabajo grupal.

## 1. Objetivo

Materializar el flujo UX de las Semanas 8, 9 y 10 en un prototipo estatico,
interactivo y navegable para demostrar el registro de signos vitales.

## 2. Alcance

`prototipo/index.html` es un unico archivo HTML autocontenido con CSS y
JavaScript vanilla. Usa paciente y mediciones ficticias, no requiere servidor ni
dependencias externas, y conserva los datos solo durante la sesion del navegador.

No usa Laravel, Vue, Vite, API, base de datos ni autenticacion. No consume
`POST /api/v1/vital-signs`, no persiste informacion y no aplica rangos, alertas
ni reglas clinicas reales. El selector **Modo prototipo / simulacion** determina
el resultado de forma explicita; las mediciones no lo determinan.

## 3. Base de diseno

| Semana | Aplicacion                                                                                |
| ------ | ----------------------------------------------------------------------------------------- |
| 8      | Contexto, formulario, errores, exito, alerta y error de comunicacion.                     |
| 9      | Semantica de formulario, labels, foco, mensajes anunciables y separacion de alerta/error. |
| 10     | Reflow, controles tactiles, una columna movil y alerta dentro del flujo vertical.         |
| 5      | Mediciones y unidades como referencia del contrato documentado, sin invocar el endpoint.  |

## 4. Flujo implementado

```text
Contexto ficticio -> Registrar -> fecha, hora y una o mas mediciones
  -> validacion de interfaz -> resultado simulado
     -> exito normal | exito con alerta | error de red/servidor -> reintento
```

Cancelar vuelve al contexto. Los resultados exitosos permiten volver o registrar
otra medicion. El reintento conserva lo ingresado y permite elegir el estado a
demostrar sin recapturar valores.

## 5. Estados simulados

- Validacion: fecha, hora y una medicion son requeridas; se comprueba formato
  numerico no negativo solo como ergonomia de interfaz, no como regla clinica.
- Exito normal: confirma un registro simulado sin alerta reportada.
- Exito con alerta: confirma primero el registro y presenta una alerta separada.
- Error de red/servidor: no confirma el guardado, conserva datos y permite reintento.

## 6. Accesibilidad aplicada

- HTML semantico: encabezados, `main`, `form`, `fieldset` y `legend`.
- Labels asociados, unidades visibles y errores relacionados con `aria-describedby`.
- Resumen de errores enfocable, orden natural de tabulacion, skip link y foco visible.
- `aria-live` para cambios de estado; alerta con encabezado, texto, borde y `role="alert"`, no solo color.
- Botones con nombres de accion comprensibles y sin dependencia de hover.

No se declara conformidad WCAG formal: faltan auditoria y pruebas con usuarios y
tecnologia asistiva real.

## 7. Responsive y movilidad

Escritorio y tablet muestran contextos y mediciones en cuadricula. A 720 px o
menos, los campos y acciones pasan a una columna, conservan una altura minima de
44 px y evitan desplazamiento horizontal. La alerta se muestra en el flujo
vertical y no en un panel lateral.

## 8. Como ejecutar el prototipo

Abrir `prototipo/index.html` en un navegador. Seleccionar **Registrar signos
vitales**, completar los campos requeridos, elegir un resultado simulado y
guardar. No se requiere conexion ni servidor.

## 9. Limitaciones

- No persiste ni consulta expedientes reales.
- No consume `POST /api/v1/vital-signs` ni implementa reintento HTTP, RBAC, idempotencia u offline.
- No determina rangos, alertas ni diagnosticos clinicos.
- No sustituye la futura integracion Vue/Laravel ni corrige brechas backend de Semana 5.

## 10. Evidencias de validacion

| Escenario                    | Resultado esperado                      | Resultado del prototipo                              |
| ---------------------------- | --------------------------------------- | ---------------------------------------------------- |
| Registro valido normal       | Confirmacion sin alerta.                | Selector **Exito normal** muestra registro exitoso.  |
| Campo requerido              | Explicar correccion sin enviar.         | Resumen y error por campo.                           |
| Conservacion tras validacion | Mantener valores validos.               | La validacion no limpia campos.                      |
| Exito con alerta             | Guardado y alerta distintos.            | Registro exitoso y bloque `role="alert"`.            |
| Error de red/servidor        | No confirmar guardado; conservar datos. | Estado "Registro no confirmado".                     |
| Reintento                    | No recapturar valores.                  | Selector y boton Reintentar.                         |
| Teclado/foco                 | Operacion perceptible.                  | Tabulacion natural, skip link y `:focus-visible`.    |
| Movil/escritorio             | Reflow legible.                         | Una columna a 720 px y cuadricula en anchos mayores. |

## 11. Conclusiones

El prototipo permite validar visualmente el flujo de enfermeria y, en especial,
la diferencia entre error de interfaz, confirmacion de registro y alerta clinica,
sin representar funcionalidad clinica o de integracion real.

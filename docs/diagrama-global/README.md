# Diagrama Global de Procesos

Este diagrama integra el modulo de **Signos Vitales y Alertas por Valores Anormales** dentro del flujo principal del Sistema Hospitalario Integrado. Describe el proceso desde el acceso del usuario hasta la trazabilidad final, incluyendo los modulos que proporcionan contexto al registro clinico y los que consumen sus resultados.

## Archivos

- Codigo fuente PlantUML: [`proceso-global.puml`](proceso-global.puml).
- Imagen sugerida para agregar manualmente: `imagenes/proceso-global.png`.

## Evidencia de elaboracion

De acuerdo con los metadatos locales del sistema de archivos, estos documentos fueron creados el 18/08/2026. Se incorporan posteriormente al repositorio para completar la evidencia de las tareas trabajadas. La fecha del commit registra el momento de su publicacion en Git; la fecha local se conserva aqui como referencia documental.

## Como leer el diagrama

1. El proceso inicia validando autenticacion, tenant y permisos. Si el acceso no esta autorizado, el flujo finaliza.
2. El paciente se busca o registra y se vincula a una atencion ambulatoria o hospitalaria. En caso de ingreso, intervienen admision y asignacion de cama.
3. El expediente medico electronico provee el contexto clinico de la atencion.
4. El enfermero registra los signos vitales. El modulo valida los datos, guarda el registro con su trazabilidad y evalua los valores contra rangos de referencia.
5. Si se detecta un valor anormal, el modulo crea una alerta priorizada y la comunica a notificaciones internas. El medico la consulta, registra seguimiento y, cuando corresponde, genera acciones clinicas.
6. Las acciones derivadas pueden integrarse con notas SOAP, diagnosticos, laboratorio, alergias y prescripciones. Finalmente, auditoria, reportes y analitica reciben la trazabilidad del proceso.

## Limites del modulo asignado

La responsabilidad directa del modulo asignado comienza al ingresar las mediciones y comprende la validacion, el almacenamiento, la evaluacion contra rangos, la creacion de alertas y su seguimiento. Los demas bloques representan puntos de integracion con los modulos indicados en `README-proyecto.md`; no definen su implementacion interna.

## Relaciones principales

| Modulo integrado | Relacion con Signos Vitales y Alertas |
| --- | --- |
| Seguridad y Tenant | Autoriza cada operacion y aisla los datos clinicos por tenant. |
| Pacientes | Proporciona el paciente al que pertenece cada medicion. |
| Citas, Admision y Camas | Proporcionan el contexto de la atencion, incluyendo el ingreso cuando aplica. |
| Expediente medico electronico | Centraliza el contexto clinico consultado durante el seguimiento. |
| Notificaciones internas | Recibe las alertas por valores anormales para informar al personal responsable. |
| Notas SOAP, diagnosticos, laboratorio y prescripciones | Reciben las acciones clinicas que el medico decide tras valorar una alerta. |
| Auditoria, reportes y analitica | Consumen los eventos trazables para control e indicadores. |

## Generacion de imagen

Genere la imagen desde `proceso-global.puml` con PlantUML y agreguela manualmente en `docs/diagrama-global/imagenes/proceso-global.png`.

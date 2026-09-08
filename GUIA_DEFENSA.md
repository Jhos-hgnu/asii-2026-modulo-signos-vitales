# Guía para la defensa oral

Esta guía resume los puntos principales que deben poder explicarse durante la defensa individual del módulo **ASII-13: Signos vitales y alertas por valores anormales**. Incluye el contexto trabajado en la Semana 1 y, principalmente, las decisiones de diseño correspondientes a la Semana 2.

## 1. Problema identificado

El Sistema Hospitalario Integrado no cuenta actualmente con un flujo funcional
para registrar, evaluar y dar seguimiento a los signos vitales de los pacientes.

## 2. Proceso de negocio modelado

Registro de signos vitales, evaluación automática de valores, generación de
alertas y seguimiento por parte del personal clínico.

## 3. Actor principal

El enfermero registra las mediciones.

## 4. Actor responsable del seguimiento

El médico consulta y registra el seguimiento de las alertas.

## 5. Decisión principal

El sistema determina si existen valores fuera de los rangos configurados.

## 6. Diferencia entre los diagramas

- Casos de uso: muestra actores y funcionalidades.
- Secuencia: muestra mensajes y orden de interacción.
- Actividades: muestra acciones, decisiones y flujo del proceso.

## 7. Decisión de diseño relevante

La evaluación de los valores se modeló como un proceso interno automático,
porque no representa una acción iniciada directamente por un actor externo.

## 8. Limitación actual

Los rangos de referencia todavía son una propuesta conceptual y deberán
validarse y configurarse de acuerdo con las reglas institucionales del sistema.

# Semana 2 — Aplicación del principio LSP

## 7. Consigna individual

La actividad de Semana 2 requiere aplicar el principio de sustitución de Liskov (**LSP**) al flujo:

**Captura de signos vitales → evaluación de umbrales → generación de alerta.**

Además, deben presentarse:

- Requisitos funcionales y no funcionales.
- Criterios de aceptación.
- Diseño antes y después.
- Justificación de responsabilidades y dependencias.
- Evidencia verificable.
- Artefactos editables.
- Evidencia Git.
- Declaración transparente del uso de IA.

## 8. ¿Qué es LSP?

El principio de sustitución de Liskov establece, en términos de diseño orientado a objetos, que una implementación derivada debe poder utilizarse en lugar de su abstracción base sin provocar un comportamiento incorrecto en el sistema.

En este módulo, LSP se utiliza para que las distintas reglas de evaluación de signos vitales puedan ser intercambiables siempre que respeten el mismo contrato.

### Idea principal para la defensa

> El componente que evalúa los signos vitales no debería necesitar saber qué tipo concreto de regla está utilizando. Debe poder trabajar con cualquier implementación válida del contrato de evaluación.

## 9. Problema del diseño inicial

En el diseño inicial, las reglas de evaluación podrían depender de métodos o parámetros distintos según el signo vital.

Por ejemplo:

- Una regla de temperatura podría evaluar un único valor.
- Una regla de presión arterial podría necesitar presión sistólica y diastólica.
- Una regla de saturación podría utilizar otro conjunto de validaciones.

Si el componente consumidor tiene que preguntar qué tipo de regla recibió o aplicar lógica especial para cada implementación, las reglas dejan de ser realmente sustituibles.

Esto provoca:

- Dependencia de implementaciones concretas.
- Condicionales específicos por tipo de regla.
- Mayor acoplamiento.
- Mayor riesgo de romper el flujo al agregar nuevas reglas.
- Dificultad para sustituir una implementación por otra.

## 10. Diseño propuesto aplicando LSP

Se propone establecer un contrato común para todas las reglas de evaluación.

Conceptualmente:

```text
ReglaEvaluacionVital
    evaluar(registro: VitalSign): ResultadoEvaluacion
```

Las implementaciones concretas pueden ser, por ejemplo:

```text
ReglaTemperatura
ReglaPresionArterial
ReglaSaturacionOxigeno
ReglaGlucosa
ReglaFrecuenciaCardiaca
```

Todas deben respetar el mismo contrato.

El componente consumidor puede trabajar únicamente con la abstracción:

```text
MotorEvaluacionVital
        |
        v
ReglaEvaluacionVital
```

El `MotorEvaluacionVital` no debería necesitar código especial para reconocer cada regla concreta.

## 11. ¿Cómo se demuestra LSP en este diseño?

LSP se cumple conceptualmente cuando una implementación concreta puede sustituirse por otra sin obligar al componente consumidor a cambiar su lógica.

Ejemplo:

Si inicialmente el motor utiliza:

```text
ReglaTemperatura
```

y posteriormente se incorpora:

```text
ReglaSaturacionOxigeno
```

el motor debería poder ejecutar ambas mediante:

```text
evaluar(registro: VitalSign): ResultadoEvaluacion
```

sin agregar condiciones como:

```text
si es temperatura...
si es presión...
si es saturación...
```

El comportamiento específico queda encapsulado dentro de cada implementación.

## 12. Responsabilidades principales del diseño

| Componente                                 | Responsabilidad                                                                                   |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| **VitalSign / registro de signos vitales** | Contener o representar las mediciones registradas.                                                |
| **ReglaEvaluacionVital**                   | Definir el contrato común que deben respetar las reglas.                                          |
| **Reglas concretas**                       | Evaluar un signo vital o condición específica respetando el contrato.                             |
| **MotorEvaluacionVital**                   | Coordinar la ejecución de las reglas sin depender de implementaciones concretas.                  |
| **GestorAlertas**                          | Crear o gestionar la alerta cuando el resultado de la evaluación determine una condición anormal. |

## 13. Dependencia relevante

La dependencia importante del diseño es:

```text
MotorEvaluacionVital
        |
        v
ReglaEvaluacionVital
```

El motor depende de una **abstracción**, no de una clase concreta como `ReglaTemperatura`.

Esto permite agregar o reemplazar implementaciones de reglas sin modificar innecesariamente el motor.

## 14. Flujo después de aplicar el diseño

1. El enfermero registra los signos vitales.
2. El sistema valida el registro.
3. Se almacena el registro de signos vitales.
4. El motor de evaluación recibe el `VitalSign`.
5. El motor ejecuta las reglas disponibles a través del contrato `ReglaEvaluacionVital`.
6. Cada regla devuelve un `ResultadoEvaluacion`.
7. El sistema determina si existe una condición anormal.
8. Si corresponde, se solicita al componente de alertas crear o vincular la alerta.
9. La alerta queda relacionada con el registro que originó la condición anormal.

## 15. Requisitos que deben poder explicarse

### Requisitos funcionales

El estudiante debe poder explicar, como mínimo, que el módulo requiere:

- Registrar signos vitales asociados a un paciente.
- Validar las mediciones antes de almacenarlas.
- Evaluar automáticamente los valores registrados.
- Detectar condiciones fuera de los umbrales definidos.
- Generar una alerta cuando corresponda.
- Permitir la consulta de información al personal autorizado.

### Requisitos no funcionales

Debe poder explicar aspectos como:

- Autenticación.
- Roles y permisos.
- Separación por tenant.
- Trazabilidad.
- Capacidad de modificar o sustituir reglas de evaluación sin alterar innecesariamente el componente consumidor.

## 16. Criterio de aceptación directamente relacionado con LSP

Un criterio relevante para demostrar el principio es:

> Dado un motor de evaluación que depende del contrato común de reglas, cuando una implementación válida sea sustituida por otra que respete el mismo contrato, entonces el motor debe poder utilizarla sin requerir lógica especial para identificar su tipo concreto.

## 17. Diferencia entre el diseño antes y después

| Aspecto                | Antes                                          | Después                                                          |
| ---------------------- | ---------------------------------------------- | ---------------------------------------------------------------- |
| Dependencia            | Puede existir dependencia de reglas concretas. | Se depende de un contrato común.                                 |
| Sustitución            | Puede requerir cambios en el consumidor.       | Las implementaciones pueden sustituirse si respetan el contrato. |
| Condicionales por tipo | Pueden ser necesarios.                         | Deben evitarse en el consumidor.                                 |
| Extensión              | Agregar una regla puede modificar el motor.    | Una nueva regla puede añadirse como otra implementación válida.  |
| Acoplamiento           | Mayor.                                         | Menor respecto de las reglas concretas.                          |
| Principio              | No se garantiza LSP.                           | Se diseña buscando cumplir LSP.                                  |

## 18. Ejemplo de modificación durante la defensa

Si el docente solicita modificar un elemento, una opción sencilla es agregar una nueva regla:

```text
ReglaFrecuenciaRespiratoria
```

Esta nueva clase debe respetar:

```text
evaluar(registro: VitalSign): ResultadoEvaluacion
```

La explicación debe ser:

> La nueva regla puede incorporarse porque respeta el mismo contrato. El MotorEvaluacionVital no necesita una condición específica para saber que está trabajando con una regla de frecuencia respiratoria.

Otro ejemplo sería agregar:

```text
ReglaFrecuenciaCardiaca
```

siguiendo exactamente el mismo criterio.

## 19. Preguntas probables de defensa

### ¿Por qué aplicaste LSP?

Porque la consigna individual exige aplicar LSP al flujo de captura, evaluación y generación de alertas. El punto de diseño seleccionado fue la sustitución de las reglas utilizadas para evaluar los signos vitales.

### ¿Dónde está aplicado LSP?

En la relación entre el contrato `ReglaEvaluacionVital`, sus implementaciones concretas y el componente `MotorEvaluacionVital` que las utiliza.

### ¿Qué significa que una regla sea sustituible?

Significa que el motor puede trabajar con distintas implementaciones válidas mediante el mismo contrato, sin cambiar su lógica por el tipo concreto recibido.

### ¿Qué rompería LSP?

Una implementación que cambie las condiciones esperadas del contrato, devuelva un resultado incompatible o requiera que el consumidor agregue tratamiento especial para poder utilizarla.

### ¿Por qué no basta con crear una interfaz para afirmar que se cumple LSP?

Porque LSP no depende únicamente de que varias clases implementen la misma interfaz. También requiere que las implementaciones respeten el comportamiento esperado por el consumidor y puedan sustituirse sin producir resultados incompatibles.

### ¿Qué ocurre cuando se detecta un valor anormal?

La regla devuelve un resultado que representa la condición detectada y el flujo permite generar o vincular una alerta al registro de signos vitales correspondiente.

### ¿La alerta realiza un diagnóstico médico?

No. La alerta funciona como apoyo para identificar una condición fuera de los parámetros configurados y facilitar su revisión por personal autorizado.

## 20. Evidencia presentada

Para demostrar el cumplimiento de la actividad se incluyen:

- Documento de Semana 2.
- Requisitos funcionales.
- Requisitos no funcionales.
- Criterios de aceptación.
- Diseño antes de aplicar LSP.
- Diseño después de aplicar LSP.
- Diagramas editables en PlantUML.
- Imágenes exportadas de los diagramas.
- Matriz de trazabilidad.
- Evidencia Git.
- `DECLARACION_IA.md`.
- Esta guía para la defensa oral.

## 21. Evidencia Git que debe poder explicarse

El estudiante debe conocer el propósito de los commits realizados durante la Semana 2.

Historial generado:

```text
c47494d docs(semana-02): agregar documento de diseño aplicando LSP
ad263c4 docs(semana-02): agregar diagramas antes y despues de LSP
e378a77 docs(semana-02): completar evidencia y entrega final de LSP
```

Debe poder explicar qué se agregó en cada etapa y mostrar, si se solicita:

```powershell
git log --oneline --decorate
```

También debe conocer:

- La URL de su repositorio.
- La rama utilizada para la actividad.
- El commit o etiqueta utilizada como referencia evaluada.
- La ubicación de los artefactos dentro del repositorio.

## 22. Uso de inteligencia artificial

La utilización de IA se declaró de forma transparente en `DECLARACION_IA.md`.

Durante la defensa se debe poder explicar que la herramienta fue utilizada como apoyo para:

- Revisión de requisitos.
- Análisis de coherencia.
- Propuestas de diseño.
- Borradores de PlantUML.
- Organización documental.
- Revisión de trazabilidad.

Las decisiones finales, modificaciones y validación fueron realizadas por el estudiante.

## 23. Limitaciones actuales

La propuesta continúa siendo principalmente un diseño previo a la implementación completa.

Entre las limitaciones actuales se encuentran:

- Los rangos o umbrales clínicos definitivos todavía deben ser establecidos o validados.
- La implementación deberá comprobar que las reglas concretas realmente respeten el contrato definido.
- El cumplimiento de LSP deberá validarse posteriormente mediante pruebas unitarias y de integración.
- Aún pueden definirse detalles adicionales de repositorios, persistencia, transacciones y notificaciones.

## 24. Resumen corto para explicar el trabajo

> En la Semana 1 se modeló el flujo general del módulo de signos vitales: registro, evaluación automática y generación de alertas. En la Semana 2 se aplicó LSP específicamente al proceso de evaluación. El problema del diseño inicial era que el motor podía terminar dependiendo de reglas concretas o tratamientos especiales. La mejora consiste en definir un contrato común, `ReglaEvaluacionVital`, que permite utilizar diferentes reglas mediante la misma operación de evaluación. Así, el motor puede sustituir una regla válida por otra sin modificar su lógica, siempre que todas respeten el comportamiento esperado. La propuesta se documentó mediante requisitos, criterios de aceptación, diagramas antes y después, matriz de trazabilidad y evidencia Git.

## 25. Punto clave para recordar

> **LSP no significa únicamente utilizar herencia o una interfaz. Significa que las implementaciones deben poder sustituirse respetando el contrato y el comportamiento esperado por el componente que las utiliza.**

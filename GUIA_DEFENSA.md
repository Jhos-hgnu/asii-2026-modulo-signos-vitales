# Guía para la defensa oral

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

| ID    | Requisito o regla                                        | Diagrama     | Elemento trazable                        |
| ----- | -------------------------------------------------------- | ------------ | ---------------------------------------- |
| RQ-01 | El sistema debe permitir registrar signos vitales        | Casos de uso | CU-01 Registrar signos vitales           |
| RQ-01 | El sistema debe permitir registrar signos vitales        | Secuencia    | Mensaje `POST /api/v1/signos-vitales`    |
| RQ-01 | El sistema debe permitir registrar signos vitales        | Actividades  | Actividad “Registrar signos vitales”     |
| RQ-02 | El sistema debe validar los datos antes de almacenarlos  | Secuencia    | Bloque `alt Datos inválidos`             |
| RQ-02 | El sistema debe validar los datos antes de almacenarlos  | Actividades  | Decisión “¿Datos válidos?”               |
| RQ-03 | El sistema debe evaluar los valores registrados          | Casos de uso | CU-04 Evaluar valores registrados        |
| RQ-03 | El sistema debe evaluar los valores registrados          | Secuencia    | Actividad “Evaluar valores registrados”  |
| RQ-03 | El sistema debe evaluar los valores registrados          | Actividades  | Acción “Evaluar los valores registrados” |
| RQ-04 | El sistema debe generar una alerta por valores anormales | Casos de uso | CU-05 Generar alerta                     |
| RQ-04 | El sistema debe generar una alerta por valores anormales | Secuencia    | Bloque `alt Existen valores anormales`   |
| RQ-04 | El sistema debe generar una alerta por valores anormales | Actividades  | Decisión “¿Existen valores anormales?”   |
| RQ-05 | El personal autorizado debe consultar alertas            | Casos de uso | CU-06 Consultar alertas                  |
| RQ-05 | El personal autorizado debe consultar alertas            | Secuencia    | `GET /api/v1/alertas`                    |
| RQ-05 | El personal autorizado debe consultar alertas            | Actividades  | Actividad “Consultar alertas”            |
| RQ-06 | El médico debe registrar seguimiento                     | Casos de uso | CU-08 Registrar seguimiento              |
| RQ-06 | El médico debe registrar seguimiento                     | Secuencia    | `PATCH /api/v1/alertas/{id}`             |
| RQ-06 | El médico debe registrar seguimiento                     | Actividades  | Actividad “Registrar seguimiento”        |

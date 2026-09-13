# Guia de defensa ASII-13

## Por que utilizar Repository?

Permite separar la logica de aplicacion de la persistencia Eloquent y facilita pruebas.

## Por que existe un Service?

Porque coordina el caso de uso sin colocar reglas de negocio en Controller.

## Donde se evaluan los umbrales?

En Domain mediante `VitalSignRangeEvaluator`.

## Como se evita fuga entre hospitales?

Mediante `TenantMiddleware`, JWT y consultas restringidas por `tenant_id`.

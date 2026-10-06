# OpenAPI

Fuente de verdad de las APIs REST expuestas por QUICKPATCH.

- Angular y Flutter no inventan endpoints ni campos.
- Los servicios backend no cambian contratos silenciosamente.
- Un breaking change requiere identificar consumidores y estrategia de migración.

La especificación OpenAPI se agrega de manera incremental al implementar cada capacidad.

## Convenciones

- **Archivo:** `<servicio>.v<versión mayor>.yaml`, OpenAPI 3.0 (lo valida el CI con Spectral).
- **Autenticación:** JWT Bearer emitido por Identity con los claims `sub`, `tenant_id` y `role`. El tenant y el usuario nunca viajan en el cuerpo.
- **Correlación:** cabecera `X-Correlation-Id`; si no llega, el servicio genera una y la devuelve.
- **Errores:** `application/problem+json` (RFC 9457) con `correlationId` y, en los errores de validación, `errors` por campo.

## Catálogo

|API|Versión|Proveedor|Consumidores|Archivo|
|---|---|---|---|---|
|ServiceRequest|1.0.0|ServiceRequest|Mobile (cliente), API Gateway|[service-request.v1.yaml](service-request.v1.yaml)|
|Catalog|1.0.0|Catalog|Mobile, panel Angular|[catalog.v1.yaml](catalog.v1.yaml)|

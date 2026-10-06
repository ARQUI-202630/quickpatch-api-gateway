# Contratos compartidos QUICKPATCH

Este directorio contiene las fronteras versionadas entre aplicaciones y microservicios.

- `openapi/`: contratos REST.
- `events/`: contratos de eventos Kafka.

Los contratos se modifican de forma explícita y el Pull Request debe identificar productores, consumidores y compatibilidad.

## Convenciones OpenAPI

- **Archivo:** `<servicio>.v<versión mayor>.yaml`, OpenAPI 3.0 (lo valida el CI con Spectral).
- **Autenticación:** JWT Bearer emitido por Identity con los claims `sub`, `tenant_id` y `role`. El tenant y el usuario nunca viajan en el cuerpo.
- **Correlación:** cabecera `X-Correlation-Id`; si no llega, el servicio genera una y la devuelve.
- **Errores:** `application/problem+json` (RFC 9457) con `correlationId` y, en los errores de validación, `errors` por campo.

## Catálogo OpenAPI

|API|Versión|Proveedor|Consumidores|Archivo|
|---|---|---|---|---|
|ServiceRequest|1.0.0|ServiceRequest|Mobile (cliente), API Gateway|[service-request.v1.yaml](openapi/service-request.v1.yaml)|
|Catalog|1.0.0|Catalog|Mobile, panel Angular|[catalog.v1.yaml](openapi/catalog.v1.yaml)|

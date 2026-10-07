# quickpatch-api-gateway

API Gateway de QUICKPATCH: la única puerta de entrada HTTP de las aplicaciones (Flutter y Angular) hacia los 8 microservicios, y el dueño de los **contratos REST** (OpenAPI) que expone.

Es uno de los 12 repositorios del multirepo (SCRUM-333). Los demás repositorios de servicio, web y mobile incluyen este repositorio como submódulo en `contracts/api-gateway/` para leer los contratos (`contracts/api-gateway/openapi/...`).

## Estructura

```text
openapi/   Contratos REST, uno por servicio y versión mayor (fuente de verdad)
nginx/     Configuración del gateway: rutas /api/<servicio> → servicio en k3s (SCRUM-338)
```

## Qué hace el gateway

- Expone `/api/` y lo quita antes de reenviar al servicio (los contratos declaran `https://quickpatch.internal/api` como servidor).
- Termina TLS en la entrada y propaga `X-Correlation-Id`.
- No valida el token ni contiene reglas de negocio: cada servicio valida el JWT RS256 con la llave pública de Identity (ADR-018).

## Convenciones OpenAPI

- **Archivo:** `<servicio>.v<versión mayor>.yaml`, OpenAPI 3.0 (lo valida el CI con Spectral).
- **Autenticación:** JWT Bearer emitido por Identity con los claims `sub`, `tenant_id` y `role`. El tenant y el usuario nunca viajan en el cuerpo.
- **Correlación:** cabecera `X-Correlation-Id`; si no llega, el servicio genera una y la devuelve.
- **Errores:** `application/problem+json` (RFC 9457) con `correlationId` y, en los errores de validación, `errors` por campo.
- **Compatibilidad:** el CI compara cada contrato con la rama base (`oasdiff breaking`) y rechaza los cambios incompatibles o la eliminación de un contrato; un cambio incompatible exige una versión mayor nueva (`v2`) publicada en paralelo.
- **Versión del repositorio:** tags `vMAJOR.MINOR.PATCH`; cada consumidor fija una versión en su submódulo.

## Catálogo OpenAPI

|API|Versión|Proveedor|Consumidores|Archivo|
|---|---|---|---|---|
|ServiceRequest|1.0.0|ServiceRequest|Mobile (cliente)|[service-request.v1.yaml](openapi/service-request.v1.yaml)|
|Catalog|1.1.0|Catalog|Mobile, panel Angular|[catalog.v1.yaml](openapi/catalog.v1.yaml)|
|Identity|1.1.0|Identity|Mobile, panel Angular|[identity.v1.yaml](openapi/identity.v1.yaml)|

## Historia

Los contratos REST vivían en `quickpatch-contracts` junto con los de eventos. Por la retroalimentación del profesor (12 repositorios), los contratos REST pasaron a este repositorio y los de eventos a `quickpatch-kafka`; el historial de cada archivo se conserva.

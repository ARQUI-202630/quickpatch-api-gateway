# quickpatch-api-gateway

API Gateway de QUICKPATCH: la única puerta de entrada HTTP de las aplicaciones (Flutter y Angular) hacia los 8 microservicios, y el dueño de los **contratos REST** (OpenAPI) que expone.

Es uno de los 12 repositorios del multirepo (SCRUM-333). Los demás repositorios de servicio, web y mobile incluyen este repositorio como submódulo en `contracts/api-gateway/` para leer los contratos (`contracts/api-gateway/openapi/...`).

## Estructura

```text
openapi/   Contratos REST, uno por servicio y versión mayor (fuente de verdad)
asyncapi/  Contratos de los canales WebSocket (AsyncAPI 3), uno por canal y versión mayor
nginx/     Configuración de Nginx del gateway (VM1) para producción, QA y Grafana; sirve el panel Angular de cada ambiente
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
|ServiceRequest|1.9.0|ServiceRequest|Mobile (cliente y técnico) y panel web (administradores)|[service-request.v1.yaml](openapi/service-request.v1.yaml)|
|Catalog|1.3.0|Catalog|Mobile, panel Angular|[catalog.v1.yaml](openapi/catalog.v1.yaml)|
|Identity|1.8.0|Identity|Mobile, panel Angular|[identity.v1.yaml](openapi/identity.v1.yaml)|
|Matching|1.1.0|Matching|Mobile (técnico)|[matching.v1.yaml](openapi/matching.v1.yaml)|
|Communication (buzón de notificaciones, ADR-024, y chat de la solicitud, ADR-026)|1.3.0|Communication|Mobile|[communication.v1.yaml](openapi/communication.v1.yaml)|
|Payments (pago de la solicitud con la pasarela simulada, ADR-009, y su comprobante)|1.1.0|Payments|Mobile (cliente y empresa) y administrador del tenant|[payments.v1.yaml](openapi/payments.v1.yaml)|

**Contratos internos.** Los llaman otros servicios dentro del clúster; no pasan por el gateway ni tienen ruta en el Ingress (prefijo `/internal`). El servicio que llama reenvía el JWT del usuario.

|API|Versión|Proveedor|Consumidores|Archivo|
|---|---|---|---|---|
|Identity interna (nombre y teléfono por lote)|1.0.0|Identity|ServiceRequest|[identity-internal.v1.yaml](openapi/identity-internal.v1.yaml)|

## Catálogo AsyncAPI

|Canal|Versión|Proveedor|Consumidores|Archivo|
|---|---|---|---|---|
|Estado en tiempo real y mensajes del chat (`/v1/realtime`, ADR-023 y ADR-026)|1.1.0|Communication|Mobile, panel Angular|[realtime.v1.yaml](asyncapi/realtime.v1.yaml)|

El CI todavía no valida `asyncapi/`; mientras tanto se valida con `npx @asyncapi/cli validate asyncapi/realtime.v1.yaml`.

## Historia

Los contratos REST vivían en `quickpatch-contracts` junto con los de eventos. Por la retroalimentación del profesor (12 repositorios), los contratos REST pasaron a este repositorio y los de eventos a `quickpatch-kafka`; el historial de cada archivo se conserva.

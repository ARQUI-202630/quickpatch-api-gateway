# Contratos de eventos Kafka

Fuente de verdad de los eventos compartidos entre microservicios.

Cada evento debe documentar:

- nombre;
- versión;
- productor;
- consumidores;
- esquema;
- campos obligatorios y opcionales;
- semántica;
- compatibilidad;
- idempotencia cuando aplique.

El sobre común debe mantenerse alineado con SDD y DD.

## Convenciones

- **Archivo:** `<eventType>.v<versión>.json`, JSON Schema draft-07 (lo valida el CI con AJV).
- **Sobre común** (DD, sección 8.2.1): `eventId`, `eventType`, `eventVersion`, `occurredAt`, `correlationId`, `tenantId`, `producer` y `data`. Cada esquema lo repite completo para que se valide de forma independiente.
- **Topic:** uno por tipo de evento, con el mismo nombre que `eventType`.
- **Clave del mensaje:** el identificador del agregado (por ejemplo `serviceRequestId`), para conservar el orden por entidad.
- **Publicación:** con Transactional Outbox (ADR-007); `eventId` es el `id` de `outbox_events`.
- **Consumo:** idempotente por `eventId` (`processed_events`, RN-EV1).
- **Compatibilidad:** agregar un campo opcional es compatible; quitar o renombrar un campo, o volverlo obligatorio, es un cambio incompatible y exige una versión nueva del evento (`v2`) publicada en paralelo mientras los consumidores migran.

## Catálogo

|Evento|Versión|Productor|Consumidores|Esquema|
|---|---|---|---|---|
|`service-request.created`|1|ServiceRequest|Matching, Communication|[service-request.created.v1.json](service-request.created.v1.json)|
|`catalog.category-changed`|1|Catalog|ServiceRequest, Matching|[catalog.category-changed.v1.json](catalog.category-changed.v1.json)|

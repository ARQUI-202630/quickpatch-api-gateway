# AGENTS — API Gateway

- `openapi/`: contratos REST/OpenAPI de los 8 servicios (fuente de verdad).
- `nginx/`: enrutamiento del gateway hacia los servicios.
- Todo cambio de contrato identifica proveedor y consumidores, y clasifica compatibilidad.
- Breaking changes requieren una versión mayor nueva.
- No incluir lógica de negocio.

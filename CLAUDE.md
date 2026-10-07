# Claude Code — API Gateway

Este repositorio contiene los contratos REST (`openapi/`) y la configuración del gateway (`nginx/`).

Antes de cambiar un contrato:

1. identificar el servicio proveedor;
2. identificar consumidores (Mobile, panel Angular);
3. clasificar compatibilidad;
4. evitar breaking changes silenciosos;
5. documentar versión e impacto en el README.

El gateway no contiene reglas de negocio ni valida tokens.

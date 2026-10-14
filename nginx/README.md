# Configuración del gateway

`gateway-nginx.conf.j2` es la configuración de Nginx de VM1, el punto único de entrada (ADR-022). Se trasladó desde `quickpatch-infrastructure` (SCRUM-338).

- **Producción** (servidor por defecto): `/api/` quita el prefijo y reenvía al Traefik de VM3; lo demás sirve el panel Angular publicado en VM1.
- **QA:** solo la red del equipo. `/api/` quita el prefijo y reenvía al Traefik de VM2; lo demás también va al Traefik de QA mientras el panel de QA no se publique en VM1.
- **Grafana:** solo la red del equipo, hacia el Grafana de VM1.
- **Canal (`X-Channel-Id`, DD 10.3):** el gateway fija la cabecera en cada petición y descarta la que mande el cliente. Vale `quickpatch-web` si el `Origin` (POST) o el `Referer` (GET) es el panel de producción o de QA, y `quickpatch-mobile` en cualquier otro caso. Cada servicio traduce el canal a tenant con `Channel__Tenants__*`. Límite: un cliente que no sea navegador puede escribir `Origin` y `Referer`; con un solo tenant los dos canales dan el mismo tenant, y con más de uno hace falta DEP-09.
  - **Si se cambia el Nginx:** un `proxy_set_header` dentro de una `location` o un `server` hace que esa ruta deje de heredar los del bloque `http`, incluido `X-Channel-Id`, que volvería a llegar como el cliente lo mande. Y si el panel de `quickpatch-web` agrega `Referrer-Policy: no-referrer`, sus GET pasarían a `quickpatch-mobile` sin dar ningún error.
  - La cabecera también llega a Grafana; no le hace daño.
- En todos: TLS con el certificado autofirmado de VM1 y soporte de WebSocket. Las cabeceras de la petición, como `X-Correlation-Id`, pasan sin cambios, salvo `X-Channel-Id` (arriba).

Los Ingress de cada servicio usan las rutas del contrato sin `/api` (`/v1/auth`, `/v1/catalog`, ...), así que una ruta nueva en `openapi/` no necesita cambios aquí mientras use el prefijo de un servicio que ya tiene Ingress.

## Cómo se despliega

Es una plantilla de Jinja con las IPs y los nombres del inventario de `infrastructure/ansible` del repositorio `quickpatch`, que trae este repositorio como submódulo en `apps/api-gateway`. Desde `infrastructure/ansible`:

```bash
./ap playbooks/deploy-gateway.yml
```

El playbook escribe la configuración solo si `nginx -t` la acepta con los certificados reales de VM1, y después recarga Nginx. Se aplica la versión que fija el submódulo.

# Configuración del gateway

`gateway-nginx.conf.j2` es la configuración de Nginx de VM1, el punto único de entrada (ADR-022). Se trasladó desde `quickpatch-infrastructure` (SCRUM-338).

- **Producción** (servidor por defecto): `/api/` quita el prefijo y reenvía al Traefik de VM3; lo demás sirve el panel Angular publicado en VM1.
- **QA:** solo la red del equipo. `/api/` quita el prefijo y reenvía al Traefik de VM2; lo demás también va al Traefik de QA mientras el panel de QA no se publique en VM1.
- **Grafana:** solo la red del equipo, hacia el Grafana de VM1.
- En todos: TLS con el certificado autofirmado de VM1 y soporte de WebSocket. Las cabeceras de la petición, como `X-Correlation-Id`, pasan sin cambios.

Los Ingress de cada servicio usan las rutas del contrato sin `/api` (`/v1/auth`, `/v1/catalog`, ...), así que una ruta nueva en `openapi/` no necesita cambios aquí mientras use el prefijo de un servicio que ya tiene Ingress.

## Cómo se despliega

Es una plantilla de Jinja con las IPs y los nombres del inventario de `infrastructure/ansible` del repositorio `quickpatch`, que trae este repositorio como submódulo en `apps/api-gateway`. Desde `infrastructure/ansible`:

```bash
./ap playbooks/deploy-gateway.yml
```

El playbook escribe la configuración solo si `nginx -t` la acepta con los certificados reales de VM1, y después recarga Nginx. Se aplica la versión que fija el submódulo.

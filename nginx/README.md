# Configuración del gateway

Hay dos plantillas de Nginx: el gateway de cada ambiente y la entrada TLS de VM1. Se trasladaron desde `quickpatch-infrastructure` (SCRUM-338) y se separaron para que producción y QA tengan cada uno su gateway (SCRUM-573).

```
VPN ─443→ VM1  entrada: certificado, TLS, red del equipo, Grafana
              ├─ quickpatch.internal ────→ VM3:8080  gateway de producción → Traefik :30080
              └─ qa.* / beta.qa.* ───────→ VM2:8080  gateway de QA         → Traefik :30080
```

## `gateway-ambiente-nginx.conf.j2`: el gateway de un ambiente (VM3 y VM2)

Es la misma plantilla para los dos ambientes; `entorno` (`produccion` o `qa`) decide el nombre, el tenant Beta y el panel. Escucha en HTTP por el puerto `gateway_puerto` (8080), al que solo llega VM1 (regla de firewall en `firewall.yml`), y reenvía al Traefik de su propia VM.

- `/api/` quita el prefijo y reenvía al Traefik del ambiente; `/api/v1/realtime` es el WebSocket (75 s, sin buffer).
- Lo demás sirve el panel Angular de la carpeta `panel` de esa VM (`/opt/quickpatch/gateway/panel`), que escribe el job de `quickpatch-web`.
- `/gateway-salud` responde `ok` sin tocar Traefik, para comprobar el gateway desde VM1.
- **Canal (`X-Channel-Id`, DD 10.3):** el gateway fija la cabecera en cada petición y descarta la que mande el cliente (incluida la que ponga VM1). Vale `quickpatch-web` si el `Origin` (POST) o el `Referer` (GET) es el panel del ambiente, y `quickpatch-mobile` en cualquier otro caso. Cada servicio traduce el canal a tenant con `Channel__Tenants__*`. **Excepción, tenant Beta de QA:** quien entra por `beta.qa.quickpatch.internal` recibe `quickpatch-beta` sin importar `Origin` ni `Referer`; ese nombre solo existe en QA y no sirve el panel (SCRUM-62). Límite: un cliente que no sea navegador puede escribir `Origin` y `Referer`; con un solo tenant los dos canales dan el mismo tenant, y con más de uno hace falta DEP-09.
  - **Si se cambia el Nginx:** un `proxy_set_header` dentro de una `location` o un `server` hace que esa ruta deje de heredar los del bloque `http`, incluido `X-Channel-Id`, que volvería a llegar como el cliente lo mande. Y si el panel de `quickpatch-web` agrega `Referrer-Policy: no-referrer`, sus GET pasarían a `quickpatch-mobile` sin dar ningún error.
- No filtra por red de origen: siempre le llega VM1. La lista de redes del equipo se aplica en la entrada.

## `entrada-nginx.conf.j2`: la entrada de VM1

Termina el HTTPS (certificado autofirmado), aplica la lista de redes del equipo a QA, Beta y Grafana, y reenvía por `Host` al gateway del ambiente sin cambiar `Host`, `Origin` ni `Referer`. Producción es el servidor por defecto: entrar por IP o por un nombre desconocido cae ahí. Los dos saltos usan 75 s para el WebSocket; si uno usa el valor por defecto (60 s), corta la conexión inactiva.

- **Marcha atrás por ambiente:** `gateway_propio.qa` y `gateway_propio.produccion` (en `inventory/group_vars/all/main.yml`) valen `false` mientras el ambiente lo atienda VM1 por sí mismo (API a Traefik y panel en VM1, la forma anterior) y `true` cuando lo atienda su gateway. Con los dos en `false` la configuración es la de antes del cambio. Cuando los dos estén en `true` y estables, se borran las ramas locales de la plantilla.
- **Grafana:** solo la red del equipo, hacia el Grafana de VM1.

Los Ingress de cada servicio usan las rutas del contrato sin `/api` (`/v1/auth`, `/v1/catalog`, ...), así que una ruta nueva en `openapi/` no necesita cambios aquí mientras use el prefijo de un servicio que ya tiene Ingress.

## Cómo se despliega

Son plantillas de Jinja con las IPs y los nombres del inventario de `infrastructure/ansible` del repositorio `quickpatch`, que trae este repositorio como submódulo en `apps/api-gateway`. Desde `infrastructure/ansible`:

```bash
./ap playbooks/deploy-gateway.yml
```

El playbook despliega primero el gateway de cada ambiente (VM3 y VM2) y después la entrada de VM1. Cada configuración se escribe solo si `nginx -t` la acepta, y después se recarga Nginx. Se aplica la versión que fija el submódulo. `verificar-gateway-ambiente.yml` comprueba cada gateway desde VM1 antes y después de cambiar `gateway_propio`.

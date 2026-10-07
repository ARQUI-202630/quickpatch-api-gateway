# Configuración del gateway

Aquí vivirá la configuración de Nginx del API Gateway (rutas `/api/<servicio>` hacia cada servicio en k3s, TLS y cabeceras), que hoy está en `quickpatch-infrastructure/ansible/templates/gateway-nginx.conf.j2`.

El traslado y el despliegue a QA y producción son la subtarea SCRUM-338 (DevOps).

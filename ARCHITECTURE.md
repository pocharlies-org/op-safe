# ARCHITECTURE.md — op-safe

Envoltorio de fiabilidad para la CLI de 1Password (`op`) en macOS con integración de la app de escritorio (desbloqueo biométrico): serializa las llamadas y repara el puente cuando la app se queda sorda.

## Clientes y versiones
- Un script de shell `bin/op-safe` (envoltorio de `op`), `bin/1password-bridge-heal.sh` (reparación) y un LaunchAgent `launchagents/com.USERNAME.1password-bridge-watchdog.plist` (vigilante); `install.sh` los instala. Solo macOS. Sin MCP, sin web, sin iOS.
- **Ruta AgentGateway: ninguna.** El acceso a 1Password para agentes va por la ruta `/1password` del gateway y, desde el x86, por la skill `op-via-mac` que ejecuta `op-safe` en el Mac por SSH.

## Dependencias en ambos sentidos
- **Depende de:** la CLI `op` y la app de 1Password 8 (puente local con Touch ID/Apple Watch), `launchd`.
- **Quién depende de él:** la skill `op-via-mac` (llamada remota por SSH al Mac). Ningún manifiesto de `~/k8s`.
- Sin `CONTRACTS.yaml`. Superficie: la línea de órdenes de `op-safe`, que reenvía a `op`.

## Stack
- Bash, `launchd`, 1Password CLI. Sin dependencias de paquetes.

## Componentes compartidos
- Ninguno. No es una librería: es una herramienta de operador.

## Cómo se construye
- Una sola cola de llamadas a `op` (serialización) con tiempos de espera y detección de los tres fallos descritos en el README (bombardeo, app zombi tras auto-actualización, ciclo bloqueo/desbloqueo); la reparación es salir de la app y relanzarla.

## Tests
- `tests/run-tests.sh`.

## CI/CD y despliegue
- Sin workflows. Se instala a mano con `install.sh` en el Mac. Tronco: `main`.

## Decisiones y trampas
- No usar desde pods de Kubernetes ni para claves SSH (el agente SSH se reenvía aparte).
- El nombre del plist lleva `USERNAME` como marcador: `install.sh` lo sustituye; no instalar el fichero a mano.
- Los secretos no pasan por el transcript: el valor sale por la salida de `op`, no se commitea.

## Reutilización
- Para secretos de cargas en clúster: Vault + external-secrets, no `op-safe`. Búsquedas: lectura de `README.md`, `bin/op-safe`, `install.sh`, `tests/run-tests.sh`; la skill `op-via-mac` lo usa.

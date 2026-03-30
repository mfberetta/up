# up
Práctica Docker y Git para la asignatura Virtualización de redes y Servicios de la UP

# 🐳 Limitar recursos en Docker (CPU y Memoria)

## 🧠 ¿Qué muestra `docker stats`?

Cuando ejecutás:

```bash
docker stats
```
Podés ver algo como:

```bash
CONTAINER ID   NAME               CPU %   MEM USAGE / LIMIT     MEM %
....           xxxxx              5.94%   89.25MiB / 1.925GiB   4.53%
```

🔹 CPU % — ¿Qué significa?
El CPU % es relativo al total de CPUs del host, no a un solo core.

📌 Ejemplo:
CPU % = 5.94%
👉 Está usando el 5.94% del total de CPU disponible del servidor

💡 Interpretación:
CPUs del host	Uso real aproximado
2 cores	~0.12 cores
4 cores	~0.24 cores

🔥 Importante
100% = uso completo de todos los CPUs disponibles
200% = uso de 2 cores completos

🔹 MEM LIMIT — ¿Por qué aparece 1.92GiB?
Si no definís límites, Docker muestra:
👉 La memoria total disponible del host (o cgroup padre)

📌 Ejemplo:
MEM LIMIT = 1.925GiB

👉 Significa:
Tu servidor tiene ~2GB de RAM disponibles
El contenedor puede usar toda esa memoria

⚠️ Importante
Si no definís límites:
Un contenedor puede consumir toda la RAM
Puede afectar otros servicios
Puede colgar el servidor

🧠 Limitar memoria en Docker
🔹 Ejemplo básico
docker run -m 512m nginx

👉 Limita el contenedor a 512 MB de RAM

🔹 Opciones útiles
docker run \
  -m 512m \
  --memory-swap 1g \
  nginx

-m → límite de RAM
--memory-swap → RAM + swap total permitido

🔒 Sin swap:
--memory-swap 512m

⚙️ Limitar CPU en Docker
🔹 1. Limitar cantidad de CPUs
docker run --cpus="1.5" nginx
👉 Máximo 1.5 CPUs

🔹 2. Afinidad de CPU (CPU pinning)
docker run --cpuset-cpus="0,1" nginx
👉 Solo usa cores 0 y 1

🔹 3. Cuotas de CPU
docker run \
  --cpu-period=100000 \
  --cpu-quota=50000 \
  nginx
👉 Equivale a 50% de un CPU

🧩 Ejemplo completo (tipo VM)
docker run -d \
  --name mi_app \
  -m 512m \
  --cpus="1" \
  --cpuset-cpus="0" \
  nginx

🐳 Usando Docker Compose

🔹 Modo normal
services:
  web:
    image: nginx
    mem_limit: 512m
    cpus: 1.0

🔹 Docker Swarm
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M

🆚 Docker vs VM
Característica	Docker	VM
Aislamiento	OS (cgroups)	Completo
Overhead	Bajo	Alto
Límites CPU/RAM	Sí	Sí
Boot	Segundos	Minutos

💡 Buenas prácticas
Definir siempre límites de CPU y memoria
Evitar que un contenedor consuma todos los recursos
Usar --cpus + -m como base
Para cargas críticas → usar cpuset
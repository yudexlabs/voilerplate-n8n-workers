# n8n — Arquitectura Profesional con Docker

Configuración lista para producción con Workers, Redis como message broker y PostgreSQL como base de datos. Los servicios están completamente desacoplados: Editor, Webhook y Workers corren de forma independiente.

## Arquitectura

```
Usuarios ──► Editor (puerto 5678)  ──┐
                                      ├──► Redis (cola) ──► Worker 1
Externos ──► Webhook (puerto 5679) ──┘                 ──► Worker 2
                                              │
                                              ▼
                                        PostgreSQL
```

## Requisitos previos

- Docker >= 24.x
- Docker Compose >= 2.x
- `openssl` instalado (para generar la encryption key)

---

## Configuración inicial

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/tu-repo.git
cd tu-repo
```

### 2. Crear el archivo `.env`

```bash
cp .env.example .env
```

Editar `.env` con tus valores:

```env
POSTGRES_DB=n8n
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=CambiaEstoPorAlgoSeguro123!
N8N_ENCRYPTION_KEY=reemplaza_esto_con_clave_generada
```

> **Generar una encryption key segura:**
> ```bash
> openssl rand -hex 32
> ```
> Copia el resultado y pégalo en `N8N_ENCRYPTION_KEY`.

> **Importante:** Nunca subas `.env` a tu repositorio. Ya está incluido en `.gitignore`.

---

## Levantar los servicios

```bash
docker compose up -d
```

Esto levanta en orden:
1. PostgreSQL
2. Redis
3. n8n Editor
4. n8n Webhook
5. Worker 1 y Worker 2

### Verificar que todo corre correctamente

```bash
docker compose ps
```

Todos los servicios deben aparecer con estado `Up`.

```bash
docker compose logs -f n8n-editor
```

Espera a ver `n8n ready on port 5678` antes de abrir el navegador.

---

## Acceso

| Servicio | URL |
|----------|-----|
| Editor (UI) | http://localhost:5678 |
| Webhook receiver | http://localhost:5679 |

---

## Comandos útiles

```bash
# Ver logs de todos los servicios
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f n8n-worker-1

# Detener todos los servicios
docker compose down

# Detener y eliminar volúmenes (⚠️ borra todos los datos)
docker compose down -v

# Reiniciar un servicio específico
docker compose restart n8n-editor

# Escalar workers (agregar más instancias)
docker compose up -d --scale n8n-worker-1=3
```

---

## Estructura del proyecto

```
.
├── docker-compose.yml   # Definición de servicios
├── .env                 # Variables de entorno (no subir a Git)
├── .env.example         # Plantilla de variables
└── README.md
```

---

## Solución de problemas

**Los workers no procesan flujos**
Verifica que `EXECUTIONS_MODE=queue` esté configurado en el `.env` y que Redis esté corriendo:
```bash
docker compose ps redis
```

**Error de conexión a PostgreSQL**
Espera unos segundos al primer inicio. PostgreSQL puede tardar en inicializarse. Revisa los logs:
```bash
docker compose logs postgres
```

**Perdí acceso a mis credenciales**
La `N8N_ENCRYPTION_KEY` no debe cambiar nunca después del primer inicio. Guárdala en un gestor de contraseñas.

---

## Recursos

- [Documentación oficial n8n — Queue Mode](https://docs.n8n.io/hosting/scaling/queue-mode/)
- [Variables de entorno n8n](https://docs.n8n.io/hosting/configuration/environment-variables/)

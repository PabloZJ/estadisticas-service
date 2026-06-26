# estadisticas-service

Microservicio de estadísticas del casino VidalCasino 2.0. API REST en Python/FastAPI. Provee métricas en tiempo real sobre el comportamiento de los jugadores y el rendimiento del casino.

## Stack

- Python 3.12 + FastAPI + Uvicorn
- PostgreSQL 16 (compartida con casino-backend)
- JWT validación (HS256) — el token lo emite casino-backend
- Docker + Amazon ECR
- Kubernetes (Amazon EKS)

## Puerto

| Entorno | Puerto |
|---------|--------|
| Local | 8006 |
| Clúster | ClusterIP :8006 |

## Endpoints

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/estadisticas` | Estadísticas generales del casino |
| GET | `/api/estadisticas/jugador` | Estadísticas del usuario autenticado |
| GET | `/livez` | Liveness probe — 200 si el proceso está vivo |
| GET | `/readyz` | Readiness probe — 200 BD ok / 503 BD caída |
| GET | `/docs` | Documentación Swagger automática |

## Variables de entorno

| Variable | Descripción |
|----------|-------------|
| `JWT_SECRET` | Secreto para validar tokens JWT |
| `DB_HOST` | Host de PostgreSQL |
| `DB_PORT` | Puerto de PostgreSQL (default: 5432) |
| `DB_USER` | Usuario de PostgreSQL |
| `DB_PASSWORD` | Contraseña de PostgreSQL |
| `DB_NAME` | Nombre de la base de datos |
| `CORS_ORIGIN` | Origen permitido para CORS |

## Construcción local

```bash
docker build -t estadisticas-service .
docker run -p 8006:8006 --env-file .env estadisticas-service
```

## Despliegue en EKS

El pipeline CI/CD se dispara automáticamente con push a la rama `deploy`:

```bash
git checkout deploy
git merge dev
git push origin deploy
```

El pipeline hace:
1. Build de la imagen Docker
2. Push a Amazon ECR con tags `latest`, `v1.0.0` y `$GITHUB_SHA`
3. Deploy en EKS con `kubectl apply`

## Comandos útiles

```bash
# Ver pods
kubectl get pods | grep estadisticas

# Ver logs
kubectl logs deployment/estadisticas-service

# Ver HPA
kubectl get hpa estadisticas-service-hpa

# Verificar sondas
curl http://localhost:8006/livez
curl http://localhost:8006/readyz
```

## Troubleshooting

| Problema | Solución |
|----------|----------|
| CrashLoopBackOff | `kubectl logs deployment/estadisticas-service` |
| BD no conecta | Verificar secret `casino-secrets` y pod de postgres |
| Pipeline falla | Actualizar GitHub Secrets con nuevas credenciales del Learner Lab |

## Ramas

| Rama | Uso |
|------|-----|
| `main` | Referencia estable |
| `dev` | Desarrollo diario |
| `deploy` | Dispara el pipeline CI/CD |
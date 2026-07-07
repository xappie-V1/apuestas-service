# apuestas-service

Microservicio de **apuestas deportivas** del casino (FastAPI). Comparte la base de
datos PostgreSQL y el `JWT_SECRET` con `casino-backend` (no tiene login propio:
valida el JWT que emite el backend). Lista eventos con cuotas 1X2, registra
apuestas, **simula** el partido (modelo Poisson) y liquida las apuestas; los
equipos/escudos se siembran desde thesportsdb.

- Prefijo de rutas: `/api/apuestas` · Docs: `/docs`

## Endpoints
| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/apuestas/eventos` | Eventos abiertos con cuotas y escudos |
| POST | `/api/apuestas` | Registrar una apuesta (debita saldo) |
| GET | `/api/apuestas/mis-apuestas` | Apuestas del usuario |
| POST | `/api/apuestas/eventos/{id}/simular` | Simula y liquida el partido |
| POST | `/api/apuestas/reiniciar` | Regenera la cartelera |

## Ejecutar en local
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# variables: copia .env.example a .env y ajústalas
uvicorn app.main:app --reload --port 8005
```
Requiere una PostgreSQL accesible con las tablas compartidas (`usuarios`,
`transacciones`) que crea `casino-backend`.

## Qué debes hacer en este repo (Entrega ET)
Trabaja en tu **fork**, con ramas `dev` (trabajo) y `deploy` (gatilla el pipeline).

1. **Sondas de salud** (ver el `TODO` en `app/main.py`): implementa
   *liveness* (200 simple, sin BD) y *readiness* (verifica la BD → 200/503).
2. **Dockerfile** del servicio (escucha en el puerto **8005**).
3. **Manifiestos de Kubernetes**: `Deployment` + `Service` **ClusterIP**, con
   `livenessProbe`/`readinessProbe` apuntando a tus rutas y la config/secretos
   tomados del Secret compartido `casino-secrets`.
4. **Workflow CI/CD** (`.github/workflows/`) gatillado por la rama `deploy`:
   build → push a Amazon ECR → deploy a EKS. Credenciales por **GitHub Secrets**.
5. **HPA** (autoescalado por CPU) y autorecuperación de pods.

> Este servicio es **interno** (ClusterIP): nunca se expone a Internet.
> No trae pruebas unitarias (la etapa de *test* del pipeline aplica al backend
> Node y al frontend Angular).
> Transversal (una sola vez para toda la plataforma, no por repo):
> **Prometheus + Grafana** en el clúster y el **video** de demostración.

# PACTA — Stack Tecnológico
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Visión General de la Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTES                             │
│                                                             │
│   ┌─────────────────┐    ┌──────────────────────────────┐  │
│   │   Web App (SaaS)│    │  Desktop App (Win + Linux)   │  │
│   │  Flask + Tailwind│   │  Python + PyWebView / Tauri  │  │
│   └────────┬────────┘    └──────────────┬───────────────┘  │
└────────────┼───────────────────────────┼────────────────────┘
             │ HTTPS / GraphQL           │ Local API
             ▼                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND (FastAPI)                         │
│                                                             │
│   ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐  │
│   │  REST Auth  │  │  GraphQL    │  │  Background Jobs │  │
│   │  /auth/*    │  │  /graphql   │  │  (notificaciones)│  │
│   └─────────────┘  └──────┬──────┘  └──────────────────┘  │
│                            │                                 │
│   ┌─────────────────────────────────────────────────┐       │
│   │           SQLAlchemy ORM (async)                │       │
│   └──────────────────────┬──────────────────────────┘       │
└──────────────────────────┼──────────────────────────────────┘
                           │
           ┌───────────────┼────────────────────┐
           ▼               ▼                    ▼
    ┌─────────────┐  ┌──────────┐  ┌────────────────────┐
    │ PostgreSQL  │  │  Redis   │  │   Object Storage   │
    │ (datos)     │  │  (cache) │  │   (documentos)     │
    └─────────────┘  └──────────┘  └────────────────────┘
```

---

## 2. Frontend — Web App (SaaS)

### 2.1 Framework Principal

| Tecnología | Versión | Rol |
|---|---|---|
| **Python** | 3.12+ | Lenguaje del servidor web |
| **Flask** | 3.x | Framework web (servidor de páginas + proxy API) |
| **Jinja2** | 3.x | Motor de plantillas HTML |
| **Tailwind CSS** | 3.x | Framework de estilos utilitarios |
| **Alpine.js** | 3.x | Reactividad mínima en el cliente |
| **HTMX** | 1.x | Interacciones AJAX sin SPA completa |

### 2.2 Justificación del Stack Frontend

Flask + Tailwind + HTMX es la combinación elegida por las siguientes razones:

- **Simplicidad operacional**: Un solo proceso Python sirve el frontend. No hay build step de Node.js en producción.
- **SEO nativo**: Las páginas se renderizan en servidor, accesibles por defecto.
- **HTMX** permite interacciones dinámicas (modales, actualizaciones parciales, búsqueda en vivo) sin necesidad de un SPA completo.
- **Alpine.js** maneja el estado UI local (toggles, dropdowns, validaciones) sin bundle pesado.
- **Tailwind** compila solo las clases usadas — CSS mínimo en producción.

### 2.3 Librerías Frontend Adicionales

| Librería | Propósito |
|---|---|
| Chart.js 4.x | Gráficos en dashboard y reportes |
| Flatpickr | Date pickers |
| Tom Select | Selects con búsqueda (clientes, proveedores) |
| Filepond | Drag & drop upload de documentos |
| Toastify.js | Notificaciones toast |

### 2.4 Build y Assets

```
flask-assets   → Compilación y minificación de CSS/JS
pytailwindcss  → Integración de Tailwind directamente desde Python
```

---

## 3. Backend — API (FastAPI)

### 3.1 Core

| Tecnología | Versión | Rol |
|---|---|---|
| **Python** | 3.12+ | Lenguaje principal |
| **FastAPI** | 0.115+ | Framework de API asíncrono |
| **Uvicorn** | 0.30+ | Servidor ASGI (producción: Gunicorn + Uvicorn workers) |
| **Pydantic v2** | 2.x | Validación de datos y schemas |
| **SQLAlchemy** | 2.x (async) | ORM con soporte async |
| **asyncpg** | 0.29+ | Driver PostgreSQL async |
| **Alembic** | 1.x | Migraciones de base de datos |

### 3.2 GraphQL

| Tecnología | Versión | Rol |
|---|---|---|
| **Strawberry GraphQL** | 0.235+ | Framework GraphQL para Python (type-safe, integración nativa con FastAPI) |
| **DataLoader** | (incluido en Strawberry) | Batching de queries N+1 |

**¿Por qué Strawberry?**

Strawberry usa decoradores Python y type hints nativos — no hay schema SDL separado que mantener. Integración directa con FastAPI como un endpoint (`/graphql`) y soporte para subscriptions via WebSocket.

### 3.3 Autenticación y Seguridad

| Tecnología | Rol |
|---|---|
| **python-jose** | Firma y verificación de JWT |
| **passlib + bcrypt** | Hash de contraseñas |
| **fastapi-limiter** | Rate limiting por IP/usuario |
| **python-multipart** | Manejo de uploads multipart |

### 3.4 Tareas en Background

| Tecnología | Rol |
|---|---|
| **APScheduler** | Scheduler de tareas (verificación diaria de vencimientos) |
| **Redis** | Broker de cache y estado de sesiones |

### 3.5 Almacenamiento de Documentos

El storage de archivos usa la interfaz **S3-compatible** (compatible con MinIO self-hosted o AWS S3 en cloud):

| Tecnología | Rol |
|---|---|
| **boto3** | Cliente S3 |
| **MinIO** (self-hosted) | Object storage para el VPS |

### 3.6 Herramientas de Desarrollo

| Herramienta | Propósito |
|---|---|
| **pytest + pytest-asyncio** | Tests unitarios e integración |
| **httpx** | Cliente HTTP para tests de API |
| **Black** | Formateo de código |
| **Ruff** | Linter rápido |
| **mypy** | Type checking estático |

---

## 4. Base de Datos — PostgreSQL

### 4.1 Configuración

| Aspecto | Detalle |
|---|---|
| **Motor** | PostgreSQL 16+ |
| **Driver** | asyncpg (async nativo) |
| **ORM** | SQLAlchemy 2.x async |
| **Migraciones** | Alembic |
| **Pool de conexiones** | SQLAlchemy async connection pool (min: 5, max: 20) |

### 4.2 Estrategia de Diseño

- **UUID v4** como primary keys (no secuenciales, seguros para multi-tenant futuro).
- **Soft delete** en todas las entidades core (campo `deleted_at`).
- **Auditoría en tabla separada** (`audit_logs`), nunca en las mismas tablas de entidades.
- **Índices** en campos de búsqueda frecuente: `status`, `end_date`, `client_id`, `supplier_id`.
- **Tipos nativos de PostgreSQL**: `TIMESTAMPTZ` para fechas, `NUMERIC(15,2)` para montos.

---

## 5. Capa GraphQL

### 5.1 Operaciones Disponibles

```graphql
# Queries (lectura)
query {
  contracts(filters: ContractFilters, pagination: PaginationInput): ContractConnection
  contract(id: ID!): Contract
  clients(search: String): [Client]
  suppliers(search: String): [Supplier]
  dashboard: DashboardStats
  reports(type: ReportType!, dateRange: DateRangeInput): Report
  auditLogs(entityId: ID, entityType: String): [AuditLog]
}

# Mutations (escritura)
mutation {
  createContract(input: ContractInput!): Contract
  updateContract(id: ID!, input: ContractInput!): Contract
  deleteContract(id: ID!): Boolean
  createSupplement(input: SupplementInput!): Supplement
  updateSupplementStatus(id: ID!, status: SupplementStatus!): Supplement
  uploadDocument(file: Upload!, entityId: ID!, entityType: String!): Document
}

# Subscriptions (tiempo real)
subscription {
  contractStatusChanged(contractId: ID!): Contract
  newNotification: Notification
}
```

### 5.2 Seguridad en GraphQL

- Autenticación via header `Authorization: Bearer <jwt>`.
- Límite de profundidad de query: máximo 7 niveles.
- Límite de complejidad de query: máximo 100 puntos.
- Introspección deshabilitada en producción.
- Rate limiting: 100 requests/minuto por usuario autenticado.

---

## 6. Infraestructura

### 6.1 VPS (Producción)

| Componente | Tecnología |
|---|---|
| **SO** | Ubuntu 24.04 LTS |
| **Proxy reverso** | Caddy (HTTPS automático via Let's Encrypt) |
| **Gestor de procesos** | systemd |
| **Servidor ASGI** | Gunicorn + Uvicorn workers (FastAPI) |
| **Servidor WSGI** | Gunicorn (Flask frontend) |
| **Base de datos** | PostgreSQL 16 (local o managed) |
| **Cache** | Redis 7 |
| **Object Storage** | MinIO |

### 6.2 Diagrama de Servicios en VPS

```
Internet
    │
    ▼
Caddy (puerto 443/80)
    ├── pacta.example.com → Flask (puerto 5000)
    └── api.pacta.example.com → FastAPI (puerto 8000)
                                    │
                              ┌─────┴──────┐
                              │  /graphql  │
                              │  /auth/*   │
                              │  /docs     │
                              └────────────┘
                                    │
                    ┌───────────────┼────────────┐
                    ▼               ▼            ▼
               PostgreSQL         Redis        MinIO
               (5432)             (6379)       (9000)
```

### 6.3 Variables de Entorno

```bash
# FastAPI Backend
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/pacta
REDIS_URL=redis://localhost:6379/0
SECRET_KEY=<256-bit random>
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=7
MINIO_ENDPOINT=localhost:9000
MINIO_ACCESS_KEY=<key>
MINIO_SECRET_KEY=<secret>
MINIO_BUCKET=pacta-documents

# Flask Frontend
FLASK_ENV=production
API_BASE_URL=https://api.pacta.example.com
SECRET_KEY=<256-bit random>
```

---

## 7. App de Escritorio (Fase Final)

La aplicación de escritorio reutiliza el backend FastAPI y agrega una capa de presentación nativa.

### 7.1 Stack Desktop

| Tecnología | Rol |
|---|---|
| **Python** | Lenguaje base |
| **PyWebView** | Renderiza la interfaz web en una ventana nativa |
| **FastAPI (embebido)** | Backend local que corre dentro del proceso de escritorio |
| **SQLite** | Base de datos local para modo offline |
| **PyInstaller** | Empaquetado en ejecutable (.exe / AppImage) |

### 7.2 Estrategia Offline-First (Desktop)

```
App Desktop (PyWebView)
        │
   ┌────▼────────────────────┐
   │   FastAPI Local          │
   │   (localhost:8765)       │
   └────┬────────────────────┘
        │
   ┌────▼──────────────────────────┐
   │   SQLite (local) + Sync Engine│
   │   ↕ Sync cuando hay internet  │
   │   PostgreSQL remoto           │
   └───────────────────────────────┘
```

### 7.3 Distribución Desktop

| Plataforma | Formato |
|---|---|
| Windows 10/11 | `.exe` instalador (NSIS) via PyInstaller |
| Ubuntu 20.04+ | `.AppImage` o `.deb` via PyInstaller |

La compilación se automatiza via **GitHub Actions** (un workflow por plataforma).

---

## 8. Seguridad — Resumen

| Capa | Medida |
|---|---|
| Transporte | HTTPS/TLS 1.3 obligatorio (Caddy) |
| Autenticación | JWT RS256, refresh tokens rotativos |
| Contraseñas | bcrypt, factor de costo 12 |
| API | Rate limiting, CORS configurado, cabeceras de seguridad |
| Base de datos | Queries parametrizados (ORM), sin string interpolation |
| Documentos | URLs pre-firmadas (expiran en 1 hora) |
| Secretos | Variables de entorno, nunca en código |
| Logs | Sin PII en logs de error |

# PACTA — Contexto del Proyecto
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Historia del Proyecto

### 1.1 Origen (v1.0 — 2025)

PACTA comenzó como un proyecto de gestión de contratos construido sobre **Next.js 15 + React 19 + TypeScript**, usando **Supabase** como backend-as-a-service y **Tailwind CSS + shadcn/ui** para la interfaz.

La v1.0 cubrió los módulos fundamentales:
- Contratos, clientes, proveedores, firmantes autorizados
- Suplementos con flujo básico de aprobación
- Dashboard con KPIs y gráficos (Recharts)
- Notificaciones automáticas por vencimiento
- Sistema de auditoría
- Reportes básicos con exportación
- Roles: admin, manager, editor, viewer

El proyecto fue desarrollado en una etapa donde el uso de IA en el flujo de trabajo era aún experimental, lo que resultó en algunas decisiones de arquitectura que hoy se consideran subóptimas:

- **Sin separación clara frontend/backend**: Next.js manejaba tanto el frontend como las API routes, creando acoplamiento.
- **Supabase como dependencia crítica**: La plataforma como servicio limita el control sobre la base de datos y el modelo de costos.
- **Sin GraphQL**: Las queries estaban hardcodeadas como REST calls directos a PostgREST (Supabase), sin flexibilidad para reportes complejos.
- **Sin app desktop en el roadmap**: La arquitectura no contemplaba reutilización fuera del browser.
- **TypeScript overhead**: Para un equipo pequeño, el overhead de TypeScript en todas las capas ralentizó el desarrollo inicial.

### 1.2 Decisión de Reescritura (v2.0 — 2026)

La reescritura completa responde a tres motivaciones principales:

**1. Control total de la infraestructura**: Migrar de Supabase a un backend propio con FastAPI + PostgreSQL elimina la dependencia de un tercero y reduce costos en escala.

**2. Stack unificado en Python**: Con FastAPI en el backend, tiene sentido usar Flask+Jinja2 en el frontend (mismo ecosistema, mismas herramientas, mismo lenguaje). Esto simplifica el equipo necesario y el onboarding.

**3. Visión a largo plazo**: La arquitectura v2.0 está diseñada desde el inicio para soportar una aplicación de escritorio (PyWebView + FastAPI embebido), algo imposible de lograr con el stack Next.js/Supabase anterior.

---

## 2. Contexto del Mercado

### 2.1 Categoría del Producto

PACTA pertenece a la categoría **CLM — Contract Lifecycle Management**. El mercado global de CLM fue valorado en ~$1.7B USD en 2024 y crece al 12% anual.

### 2.2 Competidores de Referencia

| Producto | Fortaleza | Debilidad vs PACTA |
|---|---|---|
| DocuSign CLM | Integración con firma electrónica | Precio elevado, complejo |
| Ironclad | UX excelente, IA integrada | Enterprise-only, muy caro |
| Juro | Moderno, colaborativo | Sin app desktop |
| ContractSafe | Simple, fácil de usar | Limitado en reportes |
| **PACTA** | **Control total, precio SaaS accesible, app desktop** | En construcción |

### 2.3 Diferenciadores Estratégicos de PACTA

1. **SaaS + Desktop**: El único CLM que ofrece una aplicación de escritorio offline-capable para empresas que no pueden depender 100% de internet.
2. **GraphQL nativo**: Consultas flexibles que permiten reportes personalizados sin desarrollar código nuevo.
3. **Precio accesible para medianas empresas**: Sin el overhead de soluciones enterprise.
4. **Self-hosteable**: Empresas con políticas de datos estrictas pueden instalar PACTA en sus propios servidores.

---

## 3. Principios de Diseño del Sistema

### 3.1 Principios Técnicos

- **API-first**: El backend es agnóstico del cliente. Funciona igual para el web app, la app desktop o una integración de terceros.
- **Async-first**: FastAPI + asyncpg + SQLAlchemy async — sin bloqueos en I/O.
- **Fail loudly in development, fail gracefully in production**: Los errores se exponen completamente en desarrollo y se capturan con respuestas amigables en producción.
- **Migraciones como código**: Alembic gestiona todos los cambios de schema. No hay cambios manuales en la base de datos.
- **Soft delete universal**: Nada se elimina físicamente. Todo tiene `deleted_at` y es recuperable.

### 3.2 Principios de UX

- **Claridad sobre densidad**: Interfaces claras con información jerarquizada, no paneles sobrecargados.
- **Feedback inmediato**: Toda acción del usuario debe tener respuesta visual en < 200ms (optimistic UI donde sea posible).
- **Búsqueda primero**: La barra de búsqueda es el elemento más accesible en cada listado.
- **Destructivo siempre confirma**: Cualquier acción irreversible requiere confirmación explícita.

---

## 4. Equipo y Roles

| Rol | Responsabilidad |
|---|---|
| **Lead Developer** | Arquitectura, backend, decisiones técnicas |
| **Frontend Developer** | Flask templates, Tailwind, HTMX, Alpine.js |
| **DevOps** | VPS, Caddy, systemd, CI/CD |
| **QA** | Pruebas funcionales, regresión |

En fase inicial, el Lead Developer cubre todos los roles con apoyo de herramientas de IA.

---

## 5. Herramientas de Desarrollo

| Herramienta | Propósito |
|---|---|
| **GitHub** (mowgliph/pacta) | Control de versiones, CI/CD |
| **GitHub Actions** | Tests automáticos, linting, build de desktop |
| **Claude Code CLI** | Asistente de desarrollo agentic |
| **VS Code / Neovim** | Editor |
| **Postman / Bruno** | Testing manual de API |
| **pgAdmin / DBeaver** | Administración de PostgreSQL |
| **Linear / GitHub Issues** | Gestión de tareas |

---

## 6. Convenciones del Proyecto

### 6.1 Estructura de Ramas Git

```
main          → Producción (solo merge via PR)
develop       → Rama de integración
feature/*     → Nuevas funcionalidades
fix/*         → Correcciones
chore/*       → Mantenimiento, dependencias
```

### 6.2 Commits (Conventional Commits)

```
feat: agregar módulo de suplementos
fix: corregir cálculo de contratos vencidos
docs: actualizar PRD con nuevos casos de uso
chore: actualizar dependencias de Python
refactor: extraer lógica de notificaciones a servicio
test: agregar tests del módulo de contratos
```

### 6.3 Estructura de Directorios

```
pacta/
├── backend/                    # FastAPI API
│   ├── app/
│   │   ├── api/                # Endpoints REST (auth) y GraphQL
│   │   ├── core/               # Config, seguridad, dependencias
│   │   ├── models/             # SQLAlchemy models
│   │   ├── schemas/            # Pydantic schemas
│   │   ├── graphql/            # Strawberry types, queries, mutations
│   │   ├── services/           # Lógica de negocio
│   │   └── repositories/       # Capa de acceso a datos
│   ├── migrations/             # Alembic migrations
│   └── tests/
│
├── frontend/                   # Flask web app
│   ├── app/
│   │   ├── templates/          # Jinja2 templates
│   │   ├── static/             # CSS, JS, assets
│   │   ├── routes/             # Blueprints de Flask
│   │   └── services/           # Llamadas al backend API
│   └── tests/
│
├── desktop/                    # App de escritorio (fase final)
│   ├── main.py                 # Entry point PyWebView
│   ├── backend_runner.py       # FastAPI embebido
│   └── build/                  # Scripts de PyInstaller
│
└── docs/                       # Esta documentación
```

---

## 7. Glosario

| Término | Definición |
|---|---|
| **Contrato** | Acuerdo formal entre un cliente y un proveedor, gestionado en PACTA |
| **Suplemento** | Modificación o adenda a un contrato existente |
| **Firmante Autorizado** | Persona con poder legal para firmar contratos en nombre de su empresa |
| **REU Code** | Código de identificación fiscal/registro único empresarial |
| **Auditoría** | Registro inmutable de todas las operaciones sobre entidades del sistema |
| **Soft Delete** | Eliminación lógica (campo `deleted_at`), no física, de un registro |
| **GraphQL** | Lenguaje de consulta de APIs que permite pedir exactamente los datos necesarios |
| **JWT** | JSON Web Token — mecanismo de autenticación sin estado |
| **CLM** | Contract Lifecycle Management — categoría de software para gestión de contratos |
| **ASGI** | Asynchronous Server Gateway Interface — estándar para servidores Python async |

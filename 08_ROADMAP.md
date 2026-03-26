# PACTA — Roadmap
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Visión de Largo Plazo (3 años)

PACTA evoluciona en tres horizontes:

```
HORIZONTE 1 (0-6 meses):  SaaS MVP — Reemplazar hojas de cálculo
HORIZONTE 2 (6-18 meses): SaaS Completo — Herramienta empresarial completa
HORIZONTE 3 (18-36 meses): Plataforma — Desktop app + integraciones + IA
```

---

## 2. Roadmap por Versiones

### v2.0 — MVP (Semanas 1–8) ← ACTUAL
**Objetivo:** Plataforma funcional en producción lista para primeros usuarios reales

**Módulos:**
- Autenticación completa (JWT, roles, refresh token)
- CRUD de las 5 entidades core (contratos, clientes, proveedores, firmantes, suplementos)
- Upload y descarga de documentos (MinIO)
- Dashboard con KPIs y gráfico de distribución
- Notificaciones automáticas por vencimiento (30 y 7 días)
- Cron jobs: marcado automático de contratos vencidos
- Auditoría completa de todas las operaciones
- Reportes básicos + exportación CSV
- Gestión de usuarios y roles
- Deploy en VPS con Caddy + HTTPS

**KPIs de éxito:**
- Sistema en producción con HTTPS
- Al menos 3 organizaciones piloto usando PACTA
- Cero contratos vencidos sin notificación en las organizaciones piloto

---

### v2.1 — Reportes Avanzados (Semanas 9–12)
**Objetivo:** Convertir PACTA en la fuente de verdad analítica sobre contratos

**Funcionalidades:**
- Exportación de reportes a **PDF** (con charts embebidos)
- Reporte de análisis financiero avanzado (por trimestre, por tipo, por cliente/proveedor)
- Reporte de suplementos (resumen de modificaciones por periodo)
- Filtros de fecha avanzados en todos los reportes (año fiscal, trimestre, custom)
- GraphQL Subscriptions (tiempo real) para dashboard KPIs
- Widgets de reporte embebibles (iframe) — primeros pasos hacia integraciones

---

### v2.2 — Búsqueda y UX Avanzada (Semanas 13–18)
**Objetivo:** Experiencia de usuario de nivel enterprise

**Funcionalidades:**
- **Búsqueda full-text** con PostgreSQL FTS (buscar dentro de descripciones, modificaciones de suplementos)
- **Búsqueda global** en la navbar (Ctrl+K) — busca en contratos, clientes, proveedores simultáneamente
- **Filtros guardados** — El usuario puede guardar combinaciones de filtros frecuentes
- **Vista Kanban de contratos** por estado (alternativa a la tabla)
- **Vistas personalizables** del dashboard (drag-and-drop de widgets)
- **Modo oscuro** (toggle en navbar, persistido en perfil)
- **Atajos de teclado** para acciones frecuentes

---

### v2.3 — Colaboración y Comentarios (Semanas 19–24)
**Objetivo:** Hacer de PACTA una plataforma colaborativa para equipos

**Funcionalidades:**
- **Comentarios en contratos** — Los usuarios pueden dejar notas/comentarios contextuales
- **Menciones** (@usuario en comentarios) con notificación
- **Historial de versiones de documentos** — Track de qué versión de un documento está vigente
- **Asignación de contratos** a usuarios específicos (responsable del contrato)
- **Notificaciones por email** (SMTP) además de las in-app
- **Recordatorios manuales** — Un usuario puede crear alertas custom sobre cualquier contrato

---

### v2.4 — Multi-Organización / Multi-Tenant (Semanas 25–32)
**Objetivo:** Escalar PACTA como SaaS para múltiples organizaciones

**Funcionalidades:**
- **Modelo multi-tenant** — Cada organización tiene sus datos completamente aislados
- **Onboarding flow** — Registro de nuevas organizaciones, setup de la cuenta
- **Gestión de suscripción** — Planes (Basic, Pro, Enterprise), integración con Stripe
- **Límites por plan** (contratos, usuarios, storage)
- **Panel de administración super-admin** (solo PACTA Team)
- **Invitaciones a equipo** por email

---

### v2.5 — Self-Host Edition (Semanas 33–40)
**Objetivo:** Empresas con políticas de datos estrictas pueden usar PACTA on-premise

**Funcionalidades:**
- **Docker Compose** all-in-one (FastAPI + Flask + PostgreSQL + MinIO + Redis)
- **Instalador automático** (script de bash) para Ubuntu Server
- **Documentación de self-host** completa
- **Licencia self-host** (anual, one-time)
- **Panel de actualización** — Detecta nuevas versiones y aplica con un clic
- **Backup automático** incluido en el stack self-host

---

### v3.0 — Desktop App (Meses 10–18)
**Objetivo:** PACTA disponible como aplicación de escritorio para Windows y Linux

**Funcionalidades:**
- **App de escritorio** (PyWebView + FastAPI embebido)
- **Modo offline** con SQLite local + sincronización al volver a conectar
- **Resolución de conflictos de sincronización** (merge strategy)
- **Notificaciones nativas del sistema operativo** (Windows toast, Linux libnotify)
- **Atajo global** (Ctrl+Shift+P) para abrir PACTA desde cualquier lugar
- **Integración con sistema de archivos** — Abrir documentos directamente en la app local

**Distribución:**
- Windows: `.exe` (NSIS installer) via PyInstaller + GitHub Actions
- Linux: `.AppImage` y `.deb` via PyInstaller + GitHub Actions
- Auto-update integrado (Sparkle/WinSparkle)

---

### v3.1 — IA y Automatización (Meses 18–24)
**Objetivo:** PACTA usa IA para reducir trabajo manual y detectar riesgos

**Funcionalidades:**
- **Extracción automática de datos** de contratos escaneados (OCR + LLM)
- **Resumen automático** de contratos largos
- **Detección de riesgos** — IA identifica cláusulas problemáticas o inusuales
- **Generación de alertas inteligentes** basadas en patrones históricos
- **Asistente de búsqueda en lenguaje natural** ("muéstrame contratos con clientes tech que vencen este trimestre")

---

### v3.2 — Integraciones (Meses 24–36)
**Objetivo:** PACTA como hub central que se conecta con el ecosistema empresarial

**Funcionalidades:**
- **Webhook API** — Notificar sistemas externos sobre eventos de PACTA
- **REST API pública** — Para integraciones custom
- **Integración con firma electrónica** (DocuSign, FirmaVirtual)
- **Integración con Google Drive / OneDrive** — Subir documentos directo desde cloud storage
- **Integración con Slack** — Notificaciones de vencimiento en Slack
- **Zapier / Make connector** — Automatizaciones no-code

---

## 3. Tabla Resumen del Roadmap

| Versión | Periodo | Foco Principal | Estado |
|---|---|---|---|
| v2.0 MVP | Semanas 1-8 | Plataforma funcional base | 🟡 En desarrollo |
| v2.1 | Semanas 9-12 | Reportes avanzados + PDF | ⏳ Planificado |
| v2.2 | Semanas 13-18 | Búsqueda full-text + UX | ⏳ Planificado |
| v2.3 | Semanas 19-24 | Colaboración + comentarios | ⏳ Planificado |
| v2.4 | Semanas 25-32 | Multi-tenant + SaaS billing | ⏳ Planificado |
| v2.5 | Semanas 33-40 | Self-host Docker edition | ⏳ Planificado |
| v3.0 | Meses 10-18 | Desktop App (Win + Linux) | ⏳ Planificado |
| v3.1 | Meses 18-24 | IA y automatización | 💡 Idea |
| v3.2 | Meses 24-36 | Integraciones ecosystem | 💡 Idea |

---

## 4. Deuda Técnica Anticipada

Estas son decisiones del MVP que se sabe que necesitarán revisión:

| Área | Deuda | Versión de resolución |
|---|---|---|
| Storage de documentos | MinIO en mismo VPS (riesgo de pérdida de datos) | v2.4 → mover a storage externo redundante |
| GraphQL sin persisted queries | Potencial abuso de queries complejas | v2.2 → implementar persisted queries + límites de complejidad |
| Notificaciones solo in-app | Baja visibilidad si el usuario no abre PACTA | v2.3 → agregar notificaciones por email |
| Sin tests E2E | Regresiones posibles al agregar funcionalidades | v2.1 → agregar Playwright tests |
| SQLite en desktop (v3.0) | Sync de datos es complejo | v3.0 → diseñar desde inicio con CRDT o timestamps |

---

## 5. Criterios de Avance entre Versiones

Antes de comenzar cada versión, la anterior debe cumplir:

- [ ] Todos los criterios de aceptación del QA checklist de la versión anterior
- [ ] Cero bugs críticos abiertos en GitHub Issues
- [ ] Al menos un usuario real usando la versión en producción
- [ ] Documentación actualizada (PRD, App Flow, Casos de Uso)
- [ ] Tests automáticos pasan en CI (GitHub Actions)

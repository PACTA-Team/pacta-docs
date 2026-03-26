# PACTA — MVP Completo
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Definición del MVP

El MVP de PACTA v2.0 es la versión mínima que puede ser usada en producción por organizaciones reales para reemplazar hojas de cálculo y carpetas compartidas como sistema de gestión de contratos.

El MVP no incluye todas las funcionalidades finales del producto, pero sí entrega **valor real e inmediato** al usuario desde el primer día.

---

## 2. Criterio de MVP

Un MVP exitoso cumple:
- Una organización puede gestionar su cartera completa de contratos sin necesidad de sistemas externos.
- Los contratos vencidos se detectan automáticamente.
- Existe trazabilidad completa de quién hizo qué y cuándo.
- Los reportes básicos están disponibles sin trabajo manual.

---

## 3. Alcance del MVP

### 3.1 Incluido en el MVP

| Módulo | Funcionalidades |
|---|---|
| **Autenticación** | Login, logout, JWT, roles |
| **Contratos** | CRUD completo, filtros, búsqueda, cambio de estado |
| **Clientes** | CRUD completo |
| **Proveedores** | CRUD completo |
| **Firmantes** | CRUD completo, asociación a empresa |
| **Suplementos** | Crear, flujo Borrador → Aprobado → Activo |
| **Documentos** | Upload y descarga de archivos adjuntos |
| **Dashboard** | KPIs en tiempo real, gráfico de distribución, vencimientos próximos |
| **Notificaciones** | Alertas automáticas por vencimiento (30 y 7 días) |
| **Reportes** | Distribución por estado, vencimientos, exportación CSV |
| **Auditoría** | Log completo de todas las operaciones |
| **Usuarios** | Gestión de cuentas y roles (Admin) |

### 3.2 Fuera del MVP (Post-MVP)

| Funcionalidad | Versión Planificada |
|---|---|
| Exportación de reportes a PDF | v2.1 |
| Subscriptions GraphQL (tiempo real) | v2.1 |
| Búsqueda full-text avanzada (PostgreSQL FTS) | v2.2 |
| App de escritorio (Windows + Linux) | v3.0 |
| Modo self-host con instalador | v2.5 |
| Integración de firma electrónica | v3.0 |

---

## 4. Entregables Técnicos del MVP

### 4.1 Backend (FastAPI)

**Módulo de autenticación:**
- `POST /auth/login` — Retorna JWT + refresh token
- `POST /auth/refresh` — Renueva el JWT con el refresh token
- `POST /auth/logout` — Invalida el refresh token
- `GET /auth/me` — Información del usuario actual

**GraphQL endpoint (`/graphql`):**

```
Queries:
- contracts(filters, pagination) → ContractConnection
- contract(id) → Contract
- clients(search) → [Client]
- client(id) → Client
- suppliers(search) → [Supplier]
- supplier(id) → Supplier
- authorizedSigners(companyId, companyType) → [AuthorizedSigner]
- supplements(contractId) → [Supplement]
- documents(entityId, entityType) → [Document]
- dashboard → DashboardStats
- notifications(unreadOnly) → [Notification]
- auditLogs(filters) → AuditLogConnection
- users → [User]  (solo Admin)
- reports(type, dateRange, filters) → Report

Mutations:
- createContract(input) → Contract
- updateContract(id, input) → Contract
- updateContractStatus(id, status) → Contract
- deleteContract(id) → Boolean
- createClient(input) → Client
- updateClient(id, input) → Client
- deleteClient(id) → Boolean
- createSupplier(input) → Supplier
- updateSupplier(id, input) → Supplier
- deleteSupplier(id) → Boolean
- createAuthorizedSigner(input) → AuthorizedSigner
- updateAuthorizedSigner(id, input) → AuthorizedSigner
- deleteAuthorizedSigner(id) → Boolean
- createSupplement(input) → Supplement
- updateSupplementStatus(id, status) → Supplement
- uploadDocument(file, entityId, entityType) → Document
- deleteDocument(id) → Boolean
- markNotificationRead(id) → Notification
- markAllNotificationsRead → Boolean
- createUser(input) → User  (solo Admin)
- updateUser(id, input) → User  (solo Admin)
- deactivateUser(id) → User  (solo Admin)
```

**Jobs en background:**
- Cron diario a las 8:00am: verifica contratos próximos a vencer y genera notificaciones
- Cron diario a medianoche: marca como "Vencido" los contratos cuya `end_date` pasó

**Upload de documentos:**
- `POST /upload` — Sube archivo al MinIO, retorna URL y metadata
- `GET /documents/{id}/url` — Genera URL pre-firmada temporal (1 hora)

### 4.2 Frontend (Flask)

**Páginas a implementar:**

| Ruta | Template | Descripción |
|---|---|---|
| `/login` | `auth/login.html` | Formulario de autenticación |
| `/dashboard` | `dashboard/index.html` | Panel con KPIs y gráficos |
| `/contracts` | `contracts/list.html` | Lista con filtros y búsqueda |
| `/contracts/new` | `contracts/form.html` | Formulario de creación |
| `/contracts/<id>` | `contracts/detail.html` | Detalle completo |
| `/contracts/<id>/edit` | `contracts/form.html` | Formulario de edición |
| `/clients` | `clients/list.html` | Lista de clientes |
| `/clients/new` | `clients/form.html` | Creación de cliente |
| `/clients/<id>` | `clients/detail.html` | Detalle de cliente |
| `/suppliers` | `suppliers/list.html` | Lista de proveedores |
| `/suppliers/new` | `suppliers/form.html` | Creación de proveedor |
| `/suppliers/<id>` | `suppliers/detail.html` | Detalle de proveedor |
| `/authorized-signers` | `signers/list.html` | Lista de firmantes |
| `/supplements` | `supplements/list.html` | Lista global de suplementos |
| `/documents` | `documents/list.html` | Repositorio de documentos |
| `/reports` | `reports/index.html` | Centro de reportes |
| `/notifications` | `notifications/list.html` | Lista de notificaciones |
| `/audit-log` | `audit/list.html` | Log de auditoría (Manager+) |
| `/users` | `users/list.html` | Gestión de usuarios (Admin) |
| `/profile` | `users/profile.html` | Perfil del usuario actual |

**Componentes HTMX reusables:**
- `_contract_row.html` — Fila de contrato para tabla (usada en búsqueda en vivo)
- `_client_option.html` — Opción de cliente para selects dinámicos
- `_supplier_option.html` — Opción de proveedor para selects dinámicos
- `_notification_item.html` — Item de notificación para el panel
- `_pagination.html` — Componente de paginación

### 4.3 Base de Datos (PostgreSQL)

**Tablas principales:**

```sql
-- Usuarios
users (id, name, email, hashed_password, role, status, last_access, created_at, updated_at, deleted_at)

-- Clientes
clients (id, name, address, reu_code, contacts, created_by, created_at, updated_at, deleted_at)

-- Proveedores
suppliers (id, name, address, reu_code, contacts, created_by, created_at, updated_at, deleted_at)

-- Firmantes autorizados
authorized_signers (id, company_id, company_type, first_name, last_name, position, phone, email, created_by, created_at, updated_at, deleted_at)

-- Contratos
contracts (id, contract_number, title, client_id, supplier_id, client_signer_id, supplier_signer_id, start_date, end_date, amount, type, status, description, created_by, created_at, updated_at, deleted_at)

-- Suplementos
supplements (id, contract_id, supplement_number, description, effective_date, modifications, status, client_signer_id, supplier_signer_id, created_by, created_at, updated_at)

-- Documentos
documents (id, entity_id, entity_type, filename, storage_key, mime_type, size_bytes, uploaded_by, created_at)

-- Notificaciones
notifications (id, user_id, type, title, message, entity_id, entity_type, read_at, created_at)

-- Audit log
audit_logs (id, user_id, action, entity_type, entity_id, previous_state, new_state, ip_address, created_at)

-- Refresh tokens
refresh_tokens (id, user_id, token_hash, expires_at, revoked_at, created_at)
```

---

## 5. Criterios de Aceptación del MVP

### QA — Contratos
- [ ] Se puede crear un contrato con todos los campos requeridos
- [ ] Se puede editar un contrato existente
- [ ] El contrato muestra correctamente el estado calculado (vencido si `end_date` pasó)
- [ ] Se puede filtrar por estado, tipo y rango de fechas
- [ ] La búsqueda en tiempo real funciona correctamente
- [ ] Solo Admin y Manager pueden eliminar
- [ ] La eliminación requiere confirmación con número de contrato

### QA — Notificaciones
- [ ] Se generan notificaciones para contratos que vencen en ≤ 30 días
- [ ] Se generan notificaciones urgentes para contratos que vencen en ≤ 7 días
- [ ] El badge en la navbar muestra el conteo correcto de no leídas
- [ ] Marcar como leída actualiza el badge

### QA — Seguridad
- [ ] Un Viewer no puede acceder a rutas de edición
- [ ] Un Editor no puede aprobar suplementos
- [ ] El JWT expirado resulta en redirect a login
- [ ] Tres intentos fallidos bloquean temporalmente el login
- [ ] Las URLs de documentos son pre-firmadas y expiran

### QA — Rendimiento
- [ ] La lista de contratos carga en < 2 segundos con 500 contratos
- [ ] La búsqueda en tiempo real responde en < 300ms
- [ ] El dashboard carga en < 3 segundos

---

## 6. Plan de Desarrollo del MVP

### Fase 1: Fundación (Semanas 1–2)
- Setup del repositorio y estructura de directorios
- Configuración de PostgreSQL + Alembic
- Configuración de FastAPI + Strawberry GraphQL
- Autenticación: login, JWT, refresh token
- Modelos de datos y primeras migraciones
- Setup de Flask + Tailwind + HTMX
- Layout base, navbar y sidebar

### Fase 2: CRUD Core (Semanas 3–5)
- CRUD de usuarios, clientes, proveedores, firmantes
- CRUD de contratos (el más complejo)
- CRUD de suplementos con flujo de estados
- Upload de documentos (MinIO integration)

### Fase 3: Dashboard y Reportes (Semana 6)
- Dashboard con KPIs (queries GraphQL)
- Gráfico de distribución por estado
- Tabla de vencimientos próximos
- Módulo de reportes básico
- Exportación CSV

### Fase 4: Notificaciones y Auditoría (Semana 7)
- Job scheduler (APScheduler)
- Sistema de notificaciones
- UI de notificaciones (panel + página)
- UI del audit log

### Fase 5: QA y Despliegue (Semana 8)
- Tests unitarios e integración
- Corrección de bugs
- Configuración de Caddy + systemd en VPS
- Deploy a producción
- Smoke testing en producción

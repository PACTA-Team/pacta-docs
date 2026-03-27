# 🗄️ Estructura de Datos — Backend PACTA v2.0

**Versión:** 1.0  
**Fecha:** 2026-03-27  
**Motor:** PostgreSQL 16+ con SQLAlchemy 2.x async  
**Estrategia:** UUID v4 como primary keys, soft delete, auditoría separada

---

## 📋 Índice de Tablas

1. **users** — Usuarios del sistema
2. **clients** — Empresas cliente
3. **suppliers** — Empresas proveedoras
4. **signatories** — Personas autorizadas para firmar
5. **contracts** — Contratos (core)
6. **supplements** — Adendas/modificaciones contractuales
7. **documents** — Archivos adjuntos
8. **notifications** — Notificaciones de usuarios
9. **audit_logs** — Registro de todas las operaciones
10. **contract_status_history** — Historial de cambios de estado (opcional)

---

## 📊 Esquema Detallado por Tabla

### 1️⃣ **users** (Usuarios del Sistema)

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role ENUM('admin', 'manager', 'editor', 'viewer') NOT NULL DEFAULT 'viewer',
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP WITH TIME ZONE NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_role ON users(role) WHERE deleted_at IS NULL;
CREATE INDEX idx_users_active ON users(is_active) WHERE deleted_at IS NULL;
```

**Roles:**
- `admin` — Acceso total, gestión de usuarios
- `manager` — CRUD contratos, aprobaciones, reportes
- `editor` — Crear/editar contratos, no eliminar
- `viewer` — Solo lectura, exportación

---

### 2️⃣ **clients** (Empresas Cliente)

```sql
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    address VARCHAR(500),
    city VARCHAR(100),
    country VARCHAR(100),
    fiscal_code VARCHAR(50) UNIQUE NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(255),
    contact_person VARCHAR(255),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$' OR email IS NULL)
);

CREATE INDEX idx_clients_name ON clients(name) WHERE deleted_at IS NULL;
CREATE INDEX idx_clients_fiscal_code ON clients(fiscal_code) WHERE deleted_at IS NULL;
CREATE INDEX idx_clients_active ON clients(is_active) WHERE deleted_at IS NULL;
```

---

### 3️⃣ **suppliers** (Empresas Proveedor)

```sql
CREATE TABLE suppliers (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    address VARCHAR(500),
    city VARCHAR(100),
    country VARCHAR(100),
    fiscal_code VARCHAR(50) UNIQUE NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(255),
    contact_person VARCHAR(255),
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$' OR email IS NULL)
);

CREATE INDEX idx_suppliers_name ON suppliers(name) WHERE deleted_at IS NULL;
CREATE INDEX idx_suppliers_fiscal_code ON suppliers(fiscal_code) WHERE deleted_at IS NULL;
CREATE INDEX idx_suppliers_active ON suppliers(is_active) WHERE deleted_at IS NULL;
```

---

### 4️⃣ **signatories** (Firmantes Autorizados)

```sql
CREATE TABLE signatories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    entity_type ENUM('client', 'supplier') NOT NULL,
    entity_id UUID NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    title VARCHAR(100),
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    identity_document VARCHAR(50) NOT NULL,
    identity_type ENUM('dni', 'passport', 'rut', 'other') DEFAULT 'dni',
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    CONSTRAINT fk_signatory_client FOREIGN KEY (entity_id, entity_type) 
        REFERENCES clients(id) ON DELETE CASCADE 
        WHERE entity_type = 'client',
    
    CONSTRAINT fk_signatory_supplier FOREIGN KEY (entity_id, entity_type) 
        REFERENCES suppliers(id) ON DELETE CASCADE 
        WHERE entity_type = 'supplier',
    
    CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')
);

CREATE INDEX idx_signatories_entity ON signatories(entity_type, entity_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_signatories_email ON signatories(email) WHERE deleted_at IS NULL;
CREATE INDEX idx_signatories_active ON signatories(is_active) WHERE deleted_at IS NULL;
```

---

### 5️⃣ **contracts** (Contratos — CORE)

```sql
CREATE TABLE contracts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_number VARCHAR(50) UNIQUE NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    
    -- Relaciones con empresas
    client_id UUID NOT NULL REFERENCES clients(id) ON DELETE RESTRICT,
    supplier_id UUID NOT NULL REFERENCES suppliers(id) ON DELETE RESTRICT,
    
    -- Firmantes
    client_signatory_id UUID NOT NULL REFERENCES signatories(id) ON DELETE RESTRICT,
    supplier_signatory_id UUID NOT NULL REFERENCES signatories(id) ON DELETE RESTRICT,
    
    -- Fechas y montos
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    amount NUMERIC(15, 2) NOT NULL DEFAULT 0.00,
    
    -- Clasificación
    contract_type ENUM(
        'service', 'supply', 'license', 'leasing', 'maintenance', 
        'consulting', 'partnership', 'other'
    ) NOT NULL DEFAULT 'service',
    
    status ENUM(
        'draft', 'pending', 'active', 'expired', 'cancelled', 'terminated'
    ) NOT NULL DEFAULT 'draft',
    
    -- Metadata
    created_by UUID NOT NULL REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    CHECK (end_date >= start_date),
    CHECK (amount >= 0)
);

CREATE INDEX idx_contracts_number ON contracts(contract_number) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_status ON contracts(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_client_id ON contracts(client_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_supplier_id ON contracts(supplier_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_end_date ON contracts(end_date) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_type ON contracts(contract_type) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_created_by ON contracts(created_by) WHERE deleted_at IS NULL;
```

**Lógica:**
- Cambio automático a `expired` si `end_date < TODAY` (implementado en QueryRepository o trigger)
- No se puede activar si ya está vencido
- No se puede eliminar si está activo (solo soft delete a "cancelled")

---

### 6️⃣ **supplements** (Suplementos/Adendas)

```sql
CREATE TABLE supplements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    contract_id UUID NOT NULL REFERENCES contracts(id) ON DELETE CASCADE,
    supplement_number VARCHAR(50) NOT NULL,  -- SUP-001, SUP-002, etc. (único por contract)
    
    description TEXT NOT NULL,
    modifications_detail JSONB,  -- {modificaciones: [...]}
    effective_date DATE NOT NULL,
    
    status ENUM('draft', 'approved', 'active', 'cancelled') NOT NULL DEFAULT 'draft',
    
    -- Firmantes
    client_signatory_id UUID NOT NULL REFERENCES signatories(id) ON DELETE RESTRICT,
    supplier_signatory_id UUID NOT NULL REFERENCES signatories(id) ON DELETE RESTRICT,
    
    -- Aprobación
    approved_by UUID REFERENCES users(id) ON DELETE SET NULL,
    approved_at TIMESTAMP WITH TIME ZONE NULL,
    
    created_by UUID NOT NULL REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP WITH TIME ZONE NULL,
    
    UNIQUE(contract_id, supplement_number),
    CHECK (effective_date >= CURRENT_DATE)
);

CREATE INDEX idx_supplements_contract_id ON supplements(contract_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_supplements_status ON supplements(status) WHERE deleted_at IS NULL;
CREATE INDEX idx_supplements_effective_date ON supplements(effective_date) WHERE deleted_at IS NULL;
```

---

### 7️⃣ **documents** (Documentos/Archivos Adjuntos)

```sql
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    -- Polimórfica: asociada a múltiples entidades
    entity_type ENUM('contract', 'client', 'supplier', 'supplement') NOT NULL,
    entity_id UUID NOT NULL,
    
    file_name VARCHAR(255) NOT NULL,
    file_size BIGINT NOT NULL,  -- bytes
    mime_type VARCHAR(100),
    
    -- S3/MinIO storage
    s3_key VARCHAR(500) NOT NULL,  -- ruta en object storage
    
    -- Metadata
    uploaded_by UUID NOT NULL REFERENCES users(id) ON DELETE SET NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP WITH TIME ZONE NULL,  -- presigned URL expiry
    
    CHECK (file_size > 0 AND file_size <= 52428800)  -- max 50MB
);

CREATE INDEX idx_documents_entity ON documents(entity_type, entity_id);
CREATE INDEX idx_documents_s3_key ON documents(s3_key);
CREATE INDEX idx_documents_uploaded_by ON documents(uploaded_by);
```

**Tipos MIME permitidos:**
```
application/pdf
application/vnd.openxmlformats-officedocument.wordprocessingml.document (docx)
application/vnd.openxmlformats-officedocument.spreadsheetml.sheet (xlsx)
application/msword (doc)
image/jpeg
image/png
```

---

### 8️⃣ **notifications** (Notificaciones)

```sql
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    
    type ENUM(
        'contract_expiring', 
        'supplement_pending', 
        'status_changed',
        'document_uploaded'
    ) NOT NULL,
    
    -- Referencia a entidad que genera la notificación
    entity_type VARCHAR(50),  -- 'contract', 'supplement', etc.
    entity_id UUID,
    
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    
    is_read BOOLEAN DEFAULT false,
    read_at TIMESTAMP WITH TIME ZONE NULL,
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CHECK ((is_read = false AND read_at IS NULL) OR (is_read = true AND read_at IS NOT NULL))
);

CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(is_read, user_id);
CREATE INDEX idx_notifications_created_at ON notifications(created_at DESC);
```

---

### 9️⃣ **audit_logs** (Auditoría — Registro Completo)

```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
    action ENUM('CREATE', 'UPDATE', 'DELETE') NOT NULL,
    
    entity_type VARCHAR(50) NOT NULL,  -- 'contract', 'client', 'supplement', etc.
    entity_id UUID NOT NULL,
    
    -- Estado anterior y nuevo en JSON (para visualizar cambios)
    old_values JSONB,
    new_values JSONB,
    
    description TEXT,
    
    ip_address INET,
    user_agent VARCHAR(500),
    
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    CHECK (
        (action = 'CREATE' AND old_values IS NULL) OR
        (action IN ('UPDATE', 'DELETE') AND old_values IS NOT NULL)
    )
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_action ON audit_logs(action);
CREATE INDEX idx_audit_logs_created_at ON audit_logs(created_at DESC);
```

**Ejemplo de old_values/new_values:**
```json
{
  "status": {"old": "draft", "new": "active"},
  "updated_at": {"old": "2026-03-27T10:00:00Z", "new": "2026-03-27T11:00:00Z"}
}
```

---

## 📈 Diagrama E-R Simplificado

```
┌──────────┐
│  users   │◄─────────────────┐
└──────────┘                  │
     ▲                        │
     │                        │
     ├─ created_by            │
     └─ approved_by           │
                              │
┌──────────────┐         ┌────────────┐
│   clients    │◄────────│ signatories│
└──────────────┘         └────────────┘
     ▲                        ▲
     │                        │
   client_id               client_signatory_id
     │                        │
┌────────────────────────────────────────┐
│           contracts                     │
│  (core central, relaciona todo)         │
└────────────────────────────────────────┘
     ▲                        ▲
     │                        │
 supplier_id          supplier_signatory_id
     │                        │
┌──────────────┐         ┌────────────┐
│  suppliers   │◄────────│ signatories│
└──────────────┘         └────────────┘


contracts ←──one-to-many──► supplements
contracts ←──one-to-many──► documents
supplements ←──one-to-many──► documents

contracts ←──many───► audit_logs
supplements ←──many───► audit_logs
clients ←──many───► audit_logs

contracts ←──one-to-many──► notifications
```

---

## 🔑 Decisiones de Diseño

### 1. UUID v4 como Primary Keys
✅ **Ventajas:**
- Seguros para future multi-tenancy
- No secuenciales (privacidad)
- Distribuidos (no hot spots en BD)

### 2. Soft Delete (deleted_at)
✅ **Ventajas:**
- Preservar auditoría histórica
- Reversible (restore)
- Cumplimiento regulatorio

✅ **Implementación:**
```sql
-- Todos los índices incluyen: WHERE deleted_at IS NULL
-- Queries por default filtran deleted_at IS NULL
```

### 3. Auditoría en Tabla Separada
✅ **Ventajas:**
- No contamina tablas de negocio
- Queries de auditoría independientes
- Fácil retención/archivado

### 4. JSONB para Datos Flexibles
✅ **Casos de uso:**
- `old_values/new_values` en audit_logs
- `modifications_detail` en supplements
- Permitir evolución sin migrations

### 5. Polimorfismo Restringido
✅ **En signatories y documents:**
```sql
entity_type ENUM('client', 'supplier', 'contract', 'supplement')
entity_id UUID
-- Base de datos restringe qué tipos son válidos
```

---

## ⚡ Índices Principales

| Tabla | Campo(s) | Tipo | Razón |
|-------|----------|------|-------|
| **contracts** | end_date | BTREE | Búsquedas de vencimiento |
| **contracts** | status | BTREE | Filtros por estado |
| **contracts** | client_id, supplier_id | BTREE | Joins frecuentes |
| **audit_logs** | entity_type, entity_id | BTREE | Rastrear cambios de entidad |
| **audit_logs** | created_at | BTREE DESC | Logs recientes primero |
| **notifications** | user_id, is_read | BTREE | Notificaciones por usuario |
| **clients/suppliers** | fiscal_code | UNIQUE | Búsquedas rápidas |

---

## 🔒 Constraints & Validaciones

### En Base de Datos
- ✅ UNIQUEs en: email, fiscal_code, contract_number
- ✅ FKs con DELETE CASCADE en documentos
- ✅ FKs con DELETE RESTRICT en contratos (no eliminar dependencias)
- ✅ CHECK constraints en fechas, montos, emails

### En ORM (SQLAlchemy)
- ✅ Pydantic schemas con validaciones
- ✅ Custom validators en servicios
- ✅ Transacciones ACID

### En API
- ✅ Rate limiting en login
- ✅ Validación de roles en endpoints
- ✅ Paginación obligatoria

---

## 📊 Volúmenes Estimados (6 meses)

| Tabla | Registros |
|-------|-----------|
| users | 50-100 |
| clients | 100-200 |
| suppliers | 100-200 |
| signatories | 500-1,000 |
| contracts | 10,000+ |
| supplements | 2,000-5,000 |
| documents | 20,000+ |
| notifications | 100,000+ |
| audit_logs | 500,000+ |

**Storage estimado:** 2-5 GB (sin archivos adjuntos)  
**Archivos adjuntos:** 100+ GB (MinIO/S3)

---

## 🔄 Migraciones Alembic (Fase 1)

```bash
# Inicial
alembic revision --autogenerate -m "Initial schema with users, clients, suppliers"

# Fase 2
alembic revision --autogenerate -m "Add contracts, signatories, supplements"

# Fase 3
alembic revision --autogenerate -m "Add documents, notifications, audit_logs"
```

Cada migración es reversible (down).

---

## 🚀 Próximos Pasos

1. **Revisar schema** con stakeholders
2. **Crear modelos SQLAlchemy** basados en este schema
3. **Generar migraciones Alembic** automáticas
4. **Setup BD local** con PostgreSQL (docker-compose)
5. **Tests de schema** (constraints, índices)


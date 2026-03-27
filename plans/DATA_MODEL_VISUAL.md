# 📐 Modelo de Datos Visual — PACTA Backend

**Versión:** 1.0  
**Fecha:** 2026-03-27

---

## 🎯 10 Tablas Principales

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ARQUITECTURA DE DATOS                              │
└─────────────────────────────────────────────────────────────────────────────┘

                                    ┌──────────┐
                                    │  users   │ (50-100)
                                    │ -------- │
                                    │ id (PK)  │
                                    │ email    │ UNIQUE
                                    │ password │
                                    │ role     │ enum(admin, manager, editor, viewer)
                                    │ active   │
                                    └──────────┘
                                         ▲
                        ┌────────────────┼────────────────┐
                        │                │                │
                   created_by        approved_by     read_by
                        │                │                │
                        │                │                │
        ┌───────────────┴────────────┐  │  ┌──────────────────────────────┐
        │                            │  │  │                              │
        │                            ▼  ▼  ▼                              │
        │         ┌──────────────────────────────────┐  ┌───────────────┐ │
        │         │      contracts (10,000+)         │  │ supplements   │ │
        │         │ ─────────────────────────────────│  │ (2,000-5,000) │ │
        │         │ id (PK)                          │  │ ───────────── │ │
        │         │ contract_number  UNIQUE          │  │ id (PK)       │ │
        │         │ title                            │  │ contract_id   │ │───┐
        │         │ description                      │  │ supplement_# │ │   │
        │         │ amount           NUMERIC         │  │ status        │ │   │
        │         │ contract_type    ENUM            │  │ modifications │ │   │
        │         │ status           ENUM            │  │ effective_dt  │ │   │
        │         │ start_date       ──────────────────────────────────┐ │   │
        │         │ end_date         ◄─── INDEX FOR EXPIRY ALERTS     │ │   │
        │         │                                    │ approved_by   │ │   │
        │         │ client_id        ─────────────────┐│ approved_at   │ │   │
        │         │ supplier_id      ─────────────┐  ││ created_by    │ │   │
        │         │                              │  ││               │ │   │
        │         │ client_signatory_id      ┐   │  ││ FK: contract  │─┼───┼──→ "belongs_to"
        │         │ supplier_signatory_id    │   │  │└───────────────┘ │   │
        │         │                          │   │  │                  │   │
        │         │ created_by (FK users)    │   │  └──────────────────┘   │
        │         │ created_at               │   │                          │
        │         │ updated_at               │   │   ┌────────────────────┐ │
        │         │ deleted_at SOFT DELETE   │   │   │   documents        │ │
        │         │                          │   │   │  (20,000+)         │ │
        │         └──────────────────────────┘   │   │ ──────────────────│ │
        │                  ▲                      │   │ id (PK)           │ │
        │                  │                      │   │ entity_type ENUM  │─┼─ polimórfica
        │           KEY INDEX:                    │   │ entity_id         │ │
        │           - status                      │   │ file_name         │ │
        │           - end_date                    │   │ mime_type         │ │
        │           - client_id                   │   │ s3_key  (MinIO)   │ │
        │           - supplier_id                 │   │ uploaded_by (FK)  │ │
        │           - created_by                  │   │ created_at        │ │
        │                                         │   │ expires_at        │ │
        │                                         │   └────────────────────┘
        │                                         │
        │                                         └──────────────┐
        │                                                        │
        │              ┌──────────────────────────────────────────┘
        │              │
        │              ▼
        │    ┌──────────────────┐
        │    │  signatories     │
        │    │  (500-1,000)     │
        │    │ ──────────────── │
        │    │ id (PK)          │
        │    │ entity_type ENUM │─ Polimórfica (client | supplier)
        │    │ entity_id  (FK)  │
        │    │ first_name       │
        │    │ last_name        │
        │    │ title/position   │
        │    │ email            │
        │    │ phone            │
        │    │ identity_doc     │
        │    │ is_active        │
        │    └──────────────────┘
        │           ▲
        │           │
        │    ┌──────┴─────┐
        │    │            │
        └────┼────────────┼──────────────────────────────────┐
             │            │                                  │
         client_id    supplier_id                            │
             │            │                                  │
             ▼            ▼                                  │
    ┌──────────────┐ ┌──────────────┐                       │
    │   clients    │ │  suppliers   │                       │
    │ (100-200)    │ │ (100-200)    │                       │
    │ ──────────── │ │ ──────────── │                       │
    │ id (PK)      │ │ id (PK)      │                       │
    │ name         │ │ name         │                       │
    │ address      │ │ address      │                       │
    │ city         │ │ city         │                       │
    │ fiscal_code  │ │ fiscal_code  │  UNIQUE              │
    │ phone        │ │ phone        │                       │
    │ email        │ │ email        │                       │
    │ contact_pers │ │ contact_pers │                       │
    │ is_active    │ │ is_active    │                       │
    │ created_at   │ │ created_at   │                       │
    │ deleted_at   │ │ deleted_at   │                       │
    │              │ │              │                       │
    └──────────────┘ └──────────────┘                       │
          ▲                ▲                                 │
          │                │                                 │
          └────────┬───────┘                                 │
                   │                                         │
                   │ entities                                │
                   │ (clients or suppliers)                  │
                   │                                         │
                   ▼                                         │
             ┌──────────────┐                               │
             │notifications │ FK: user_id ◄─────────────────┘
             │(100,000+)    │
             │──────────────│
             │ id (PK)      │
             │ user_id (FK) │
             │ type ENUM    │ contract_expiring, supplement_pending, etc.
             │ entity_type  │ contract, supplement, etc.
             │ entity_id    │
             │ title        │
             │ message      │
             │ is_read      │
             │ read_at      │
             │ created_at   │ INDEX: created_at DESC
             └──────────────┘


┌──────────────────────────────────────────────────────────────────────────┐
│                     audit_logs (500,000+)                                │
│ ──────────────────────────────────────────────────────────────────────── │
│ id (PK)                                                                  │
│ user_id (FK users)  ────────────────► quién hizo el cambio              │
│ action ENUM         (CREATE, UPDATE, DELETE)                             │
│ entity_type         (contract, client, supplement, etc.)                 │
│ entity_id           ────────────────► qué se cambió                      │
│ old_values JSONB    ────────────────► estado anterior                    │
│ new_values JSONB    ────────────────► estado nuevo                       │
│ description         (descripción del cambio)                             │
│ ip_address INET     (para seguridad)                                     │
│ user_agent          (navegador/cliente)                                  │
│ created_at          INDEX: created_at DESC                               │
│                                                                          │
│ INDEXES:                                                                 │
│   - (entity_type, entity_id) ─► rastrear todos los cambios de una entidad│
│   - created_at DESC ─► obtener logs recientes primero                    │
│   - action ─► filtrar por tipo de operación                              │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Relaciones Principales

### 1. **Users → Everything** (Auditoría)
```
users.id (PK)
    ├─→ contracts.created_by (FK)
    ├─→ supplements.created_by (FK)
    ├─→ supplements.approved_by (FK)
    ├─→ documents.uploaded_by (FK)
    ├─→ notifications.user_id (FK)
    └─→ audit_logs.user_id (FK)
```

### 2. **Clients & Suppliers** (Entidades Base)
```
clients.id (PK) ◄─── Linked to:
    ├─→ signatories.entity_id (cuando entity_type='client')
    ├─→ contracts.client_id (FK)
    └─→ documents.entity_id (cuando entity_type='client')

suppliers.id (PK) ◄─── Linked to:
    ├─→ signatories.entity_id (cuando entity_type='supplier')
    ├─→ contracts.supplier_id (FK)
    └─→ documents.entity_id (cuando entity_type='supplier')
```

### 3. **Contracts** (Centro del Sistema)
```
contracts.id (PK) ◄─── Linked to:
    ├─→ supplements.contract_id (1:N)
    ├─→ documents.entity_id (1:N, cuando entity_type='contract')
    ├─→ notifications (generan notificaciones)
    └─→ audit_logs (cada cambio se registra)

    Requiere:
    ├─→ contracts.client_id → clients.id (FK RESTRICT)
    ├─→ contracts.supplier_id → suppliers.id (FK RESTRICT)
    ├─→ contracts.client_signatory_id → signatories.id (FK RESTRICT)
    └─→ contracts.supplier_signatory_id → signatories.id (FK RESTRICT)
```

### 4. **Supplements** (Modificaciones)
```
supplements.id (PK) ◄─── Linked to:
    ├─→ contracts.id (N:1, FK CASCADE)
    ├─→ documents.entity_id (1:N, cuando entity_type='supplement')
    ├─→ signatories (client & supplier signatories)
    └─→ audit_logs (cambios en workflow)
```

### 5. **Documents** (Polimórficos)
```
documents.id (PK) ◄─── Puede asociarse a:
    ├─→ contracts (entity_type='contract', entity_id=contract.id)
    ├─→ clients (entity_type='client', entity_id=client.id)
    ├─→ suppliers (entity_type='supplier', entity_id=supplier.id)
    └─→ supplements (entity_type='supplement', entity_id=supplement.id)

    Storage: MinIO/S3
    ├─→ s3_key es la ruta en object storage
    ├─→ presigned URLs expiran en 1 hora
    └─→ archivos se almacenan en bucket 'pacta-documents'
```

### 6. **Notifications** (Tiempo Real)
```
notifications.id (PK) ◄─── Creadas por:
    ├─→ Task: contract_expiry_checker (diaria 8am)
    │   └─→ Busca contratos con end_date <= 30 días
    │   └─→ Crea notificación para managers/editors
    │
    ├─→ Task: supplement_approval_reminder (diaria 9am)
    │   └─→ Busca supplements en estado 'draft' > 7 días
    │   └─→ Crea notificación para managers
    │
    └─→ Eventos manuales:
        └─→ Cambio de estado de contrato
        └─→ Carga de documento

    Cada notificación:
    ├─→ Referencia a user.id (para quién es)
    ├─→ Referencia a entidad (contract, supplement, etc.)
    ├─→ Marca is_read / read_at
    └─→ Se lista en UI con badge de no-leídas
```

### 7. **Audit Logs** (Compliance & Security)
```
audit_logs.id (PK) ◄─── Se crean automáticamente para:
    ├─→ users: create, update, delete
    ├─→ clients: create, update, delete
    ├─→ suppliers: create, update, delete
    ├─→ signatories: create, update, delete
    ├─→ contracts: create, update, delete, status_change
    ├─→ supplements: create, update, approve, activate
    ├─→ documents: create, delete
    └─→ notifications: mark_read

    Formato: ACID, JSON diffs
    ├─→ old_values / new_values = cambios específicos
    ├─→ user_id = quién lo hizo
    ├─→ ip_address / user_agent = contexto
    └─→ created_at = cuándo
```

---

## 🔐 Constraints Críticos

### Foreign Key Constraints

| Origen | Destino | Action | Razón |
|--------|---------|--------|-------|
| contracts.client_id | clients.id | **RESTRICT** | No borrar cliente si tiene contratos |
| contracts.supplier_id | suppliers.id | **RESTRICT** | No borrar proveedor si tiene contratos |
| contracts.client_signatory_id | signatories.id | **RESTRICT** | No borrar signatory si está en contrato activo |
| contracts.supplier_signatory_id | signatories.id | **RESTRICT** | No borrar signatory si está en contrato activo |
| supplements.contract_id | contracts.id | **CASCADE** | Borrar supplements si contrato se borra |
| documents.* | contracts, clients, etc. | **CASCADE** | Borrar documentos con entidad padre |
| notifications.user_id | users.id | **CASCADE** | Borrar notificaciones si usuario se borra |

### Unique Constraints

| Tabla | Campo(s) | Razón |
|-------|----------|-------|
| users | email | No pueden haber dos usuarios con el mismo email |
| clients | fiscal_code | RUT/CUIT/NIF único por empresa |
| suppliers | fiscal_code | RUT/CUIT/NIF único por empresa |
| contracts | contract_number | Identificador único de contrato |
| supplements | (contract_id, supplement_number) | SUP-001, SUP-002, etc. secuencial por contrato |

### Check Constraints

```sql
-- Validaciones a nivel BD
contracts: CHECK (end_date >= start_date)  -- Fecha coherente
contracts: CHECK (amount >= 0)              -- Monto positivo
documents: CHECK (file_size > 0 AND file_size <= 52428800)  -- 0-50MB
notifications: CHECK ((is_read=false AND read_at IS NULL) OR (is_read=true AND read_at IS NOT NULL))
users: CHECK (email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$')  -- Email válido
```

---

## 📈 Índices para Performance

```sql
-- Búsquedas de vencimientos (crítico)
CREATE INDEX idx_contracts_end_date 
    ON contracts(end_date) WHERE deleted_at IS NULL;

-- Filtros por estado (común)
CREATE INDEX idx_contracts_status 
    ON contracts(status) WHERE deleted_at IS NULL;

-- Joins cliente/proveedor (muy frecuente)
CREATE INDEX idx_contracts_client_id 
    ON contracts(client_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_contracts_supplier_id 
    ON contracts(supplier_id) WHERE deleted_at IS NULL;

-- Búsqueda por nombre (common)
CREATE INDEX idx_clients_name 
    ON clients(name) WHERE deleted_at IS NULL;
CREATE INDEX idx_suppliers_name 
    ON suppliers(name) WHERE deleted_at IS NULL;

-- Auditoría eficiente
CREATE INDEX idx_audit_logs_entity 
    ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_created_at 
    ON audit_logs(created_at DESC);

-- Notificaciones sin leer (muy usado)
CREATE INDEX idx_notifications_user_id_unread 
    ON notifications(user_id, is_read) WHERE is_read = false;

-- Búsquedas de fiscal_code (única)
CREATE UNIQUE INDEX idx_clients_fiscal_code 
    ON clients(fiscal_code);
CREATE UNIQUE INDEX idx_suppliers_fiscal_code 
    ON suppliers(fiscal_code);
```

---

## 🗂️ Soft Delete Strategy

Todas las tablas core tienen `deleted_at TIMESTAMP WITH TIME ZONE NULL`:

```sql
-- Cuando se "elimina" un contrato:
UPDATE contracts SET deleted_at = NOW() WHERE id = '...'

-- Por defecto, todas las queries incluyen:
WHERE deleted_at IS NULL

-- Para recuperar datos eliminados (solo admins):
SELECT * FROM contracts WHERE deleted_at IS NOT NULL ORDER BY deleted_at DESC

-- Para purgar datos antiguos (GDPR cleanup, opcional):
DELETE FROM contracts WHERE deleted_at < NOW() - INTERVAL '1 year'
```

**Beneficios:**
- ✅ Auditoría completa (no se pierden datos)
- ✅ Recuperación ante errores
- ✅ Cumplimiento regulatorio
- ✅ Reversible

---

## 📊 Data Growth Example (6 Months)

```
Month 1:
  - users: 10
  - clients: 20
  - suppliers: 15
  - contracts: 100
  - audit_logs: 5,000
  
Month 3:
  - users: 35
  - clients: 80
  - suppliers: 60
  - contracts: 3,500
  - audit_logs: 150,000
  
Month 6:
  - users: 75
  - clients: 150
  - suppliers: 140
  - contracts: 10,000+
  - signatories: 800
  - supplements: 3,000
  - documents: 20,000
  - notifications: 100,000
  - audit_logs: 500,000+
  
  Total BD size: ~2-5 GB
  S3/Documents: ~100+ GB
```

---

## 🚀 Evolución Schema

### v1.0 (MVP)
- Users, Clients, Suppliers, Signatories, Contracts, Supplements
- Basic auditing

### v2.0 (Current)
- + Documents, Notifications, Audit Logs completos
- + GraphQL

### v3.0 (Future)
- + Electronic signature integration
- + Multi-tenant support (org_id en todas las tablas)
- + Advanced reporting (materialized views)
- + Workflow customization tables

---

## 💡 Conclusión

El modelo de datos está diseñado para:
✅ **Escalabilidad** — UUIDs, índices, soft delete  
✅ **Auditoría** — Tabla separada con JSON diffs  
✅ **Flexibilidad** — JSONB para datos variables  
✅ **Seguridad** — Constraints en BD, RESTRICT/CASCADE correcto  
✅ **Performance** — Índices estratégicos, queries optimizadas  
✅ **Compliance** — Soft delete, immutable logs, PII protegido


# 📚 Índice de Documentación — Backend PACTA

**Fecha:** 2026-03-27  
**Versión:** 1.0

---

## 📖 Documentos en la Carpeta `/pacta-docs/plans/`

### 1. **BACKEND_IMPLEMENTATION_PLAN.md** (864 líneas)
**Propósito:** Plan completo de implementación del backend en 4 fases.

**Contenido:**
- 📐 Visión general de la arquitectura
- 📂 Estructura de directorios propuesta (src/, tests/, models/, services/, etc.)
- 🔄 Dependencias entre módulos
- 📋 **Fase 1:** Fundaciones & Cores Arquitectónicos (15-18 días)
  - Inicialización proyecto, BD & ORM, Autenticación JWT, Usuarios, Repository pattern, Exception handling, CI/CD, OpenAPI
- 📋 **Fase 2:** Módulos Core (14-17 días)
  - Clients, Suppliers, Signatories, Contracts, Supplements
- 📋 **Fase 3:** Módulos Secundarios (18-21 días)
  - Documents, Audit Logs, Notifications, Background Jobs, Reports, GraphQL completo
- 📋 **Fase 4:** Testing & Production (18-22 días)
  - Testing exhaustivo, Performance, Security, Documentation, Deployment, Migration, QA
- 📊 Timeline total: **65-78 días (~10-11 semanas)**
- 🎯 Criterios de éxito por fase
- 🔐 Consideraciones de seguridad
- 📦 Stack tecnológico resumen

**Uso:** Referencia para el plan de trabajo, seguimiento de sprints, estimaciones.

---

### 2. **DATABASE_SCHEMA_SUMMARY.md** (534 líneas)
**Propósito:** Especificación SQL y design decisions de la base de datos.

**Contenido:**
- 📋 **10 Tablas Principales:**
  1. `users` — Usuarios (roles: admin, manager, editor, viewer)
  2. `clients` — Empresas cliente
  3. `suppliers` — Empresas proveedor
  4. `signatories` — Firmantes autorizados (polimórficos)
  5. `contracts` — Contratos (core central)
  6. `supplements` — Suplementos/Adendas contractuales
  7. `documents` — Archivos adjuntos (polimórficos, MinIO/S3)
  8. `notifications` — Notificaciones de usuarios
  9. `audit_logs` — Auditoría completa (JSON diffs)
  10. `contract_status_history` — Historial (opcional)

- 🔑 **Decisiones de Diseño:**
  - UUID v4 como primary keys (no secuenciales)
  - Soft delete con `deleted_at` (preservar auditoría)
  - Auditoría en tabla separada (no contamina datos)
  - JSONB para datos flexibles (old_values/new_values)
  - Polimorfismo restringido en signatories & documents

- ⚡ **Índices Principales** (end_date, status, client_id, etc.)
- 🔒 **Constraints & Validaciones** (UNIQUE, FK, CHECK)
- 📊 **Volúmenes Estimados** (6 meses)
- 🚀 **Migraciones Alembic**

**Uso:** Implementación de modelos SQLAlchemy, creación de migraciones BD, referencia para queries.

---

### 3. **DATA_MODEL_VISUAL.md** (534 líneas)
**Propósito:** Diagramas visuales y relaciones entre entidades.

**Contenido:**
- 🎯 **Arquitectura Visual ASCII** mostrando todas las relaciones
- 📊 **Relaciones Principales:**
  - Users → Everything (auditoría)
  - Clients & Suppliers (entidades base)
  - Contracts (centro del sistema, requiere múltiples FK)
  - Supplements (modificaciones contractuales)
  - Documents (polimórficos: contract, client, supplier, supplement)
  - Notifications (generadas por tasks y eventos)
  - Audit Logs (registro de todas las operaciones)

- 🔐 **Constraints Críticos:**
  - FK con RESTRICT (no eliminar si tiene dependencias)
  - FK con CASCADE (eliminar con padre)
  - UNIQUE: email, fiscal_code, contract_number
  - CHECK: fechas coherentes, montos positivos

- 📈 **Índices para Performance** (consultas críticas)
- 🗂️ **Soft Delete Strategy** (beneficios y uso)
- 📊 **Data Growth Example** (proyecciones 6 meses)
- 🚀 **Evolución Schema** (v1.0, v2.0, v3.0 future)

**Uso:** Documentación visual para stakeholders, entendimiento de relaciones, diagrama ERD de referencia.

---

## 🔗 Cómo Usar Esta Documentación

### Para **Implementadores** (Developers)
1. 📖 Leer **BACKEND_IMPLEMENTATION_PLAN.md** para entender las fases
2. 📖 Leer **DATABASE_SCHEMA_SUMMARY.md** para SQL/ORM models
3. 📖 Consultar **DATA_MODEL_VISUAL.md** cuando tengas dudas de relaciones

### Para **Arquitectos**
1. 📖 Revisar **DATA_MODEL_VISUAL.md** para visión completa
2. 📖 Consultar decisiones de diseño en **DATABASE_SCHEMA_SUMMARY.md**
3. 📖 Revisar timeline y dependencias en **BACKEND_IMPLEMENTATION_PLAN.md**

### Para **Project Managers**
1. 📖 Usar **BACKEND_IMPLEMENTATION_PLAN.md** para sprint planning
2. 📖 Seguimiento de fases (F1, F2, F3, F4)
3. 📖 Timeline: 65-78 días (~10-11 semanas)

### Para **QA / Testing**
1. 📖 Leer **DATABASE_SCHEMA_SUMMARY.md** para constraints & validaciones
2. 📖 Usar **BACKEND_IMPLEMENTATION_PLAN.md** sección F4 (Testing)
3. 📖 Crear test cases basados en esquema y relaciones

### Para **DevOps**
1. 📖 Consultar **BACKEND_IMPLEMENTATION_PLAN.md** sección F4 (Deployment)
2. 📖 Revisar requerimientos de BD (PostgreSQL 16+)
3. 📖 Configurar backups, migraciones, monitoring

---

## 📊 Resumen Ejecutivo de la Estructura de Datos

### 🎯 10 Entidades Principales

| Entidad | Registros (6mo) | Propósito | Criticidad |
|---------|-----------------|----------|-----------|
| **users** | 50-100 | Usuarios y control de acceso | 🔴 CRÍTICA |
| **clients** | 100-200 | Empresas cliente | 🔴 CRÍTICA |
| **suppliers** | 100-200 | Empresas proveedoras | 🔴 CRÍTICA |
| **signatories** | 500-1,000 | Firmantes autorizados | 🔴 CRÍTICA |
| **contracts** | 10,000+ | Contratos (core negocio) | 🔴 CRÍTICA |
| **supplements** | 2,000-5,000 | Modificaciones contractuales | 🟠 ALTA |
| **documents** | 20,000+ | Archivos adjuntos | 🟠 ALTA |
| **notifications** | 100,000+ | Alertas y notificaciones | 🟡 MEDIA |
| **audit_logs** | 500,000+ | Registro de auditoría | 🟡 MEDIA |
| **contract_status_history** | - | Historial de estados (opcional) | 🟡 MEDIA |

---

### 🔑 Características Clave

✅ **UUID v4 Primary Keys**
- No secuenciales (privacidad)
- Seguros para multi-tenancy futuro

✅ **Soft Delete**
- Preserva auditoría histórica
- Recuperación ante errores
- Cumplimiento regulatorio

✅ **Auditoría Completa**
- Tabla separada (500K+ logs)
- JSON diffs (old_values/new_values)
- Usuario, IP, user agent, timestamp

✅ **Relaciones Polimórficas**
- Documents: contract, client, supplier, supplement
- Signatories: client, supplier

✅ **Índices Estratégicos**
- end_date (búsqueda de vencimientos)
- status (filtros)
- client_id, supplier_id (joins)

✅ **Constraints Estrictos**
- UNIQUE: email, fiscal_code, contract_number
- FK RESTRICT: proteger integridad
- CHECK: validaciones en BD

---

### 📈 Capacidad Estimada

| Métrica | Valor |
|---------|-------|
| Usuarios activos | 50-100 |
| Clientes | 100-200 |
| Proveedores | 100-200 |
| Contratos activos | 10,000+ |
| Notificaciones/mes | 50,000+ |
| Audit logs/mes | 100,000+ |
| **BD Total Size** | 2-5 GB |
| **Documentos/S3** | 100+ GB |

---

## 🚀 Próximos Pasos Después de Revisar Docs

1. ✅ **Revisar documentos** con equipo de desarrollo
2. ✅ **Validar schema** con DBA/Arquitecto
3. ✅ **Crear modelos SQLAlchemy** basados en schema
4. ✅ **Generar migraciones Alembic** automáticas
5. ✅ **Setup BD local** (PostgreSQL en docker-compose)
6. ✅ **Iniciar Fase 1** (F1.1: Inicialización de proyecto)

---

## 📞 Cambios Futuros al Schema

**Si necesitas cambios al schema:**
1. Actualizar el `.md` correspondiente
2. Crear migración Alembic (`alembic revision --autogenerate`)
3. Versionar en git
4. Documentar reasoning en ADR (Architecture Decision Record)

**Versionado:**
- v1.0 — Inicial (esta versión)
- v2.0 — (Future) Multi-tenant support
- v3.0 — (Future) Advanced reporting, workflows

---

## 📎 Referencias Cruzadas

| Documento | Sección | Para |
|-----------|---------|------|
| BACKEND_IMPLEMENTATION_PLAN.md | Fase 1.2 | Crear BD & ORM |
| DATABASE_SCHEMA_SUMMARY.md | Tabla `contracts` | SQL models |
| DATA_MODEL_VISUAL.md | Contracts (Centro) | Entender relaciones |
| BACKEND_IMPLEMENTATION_PLAN.md | Fase 3.2 | Auditoría |
| DATABASE_SCHEMA_SUMMARY.md | audit_logs | Implementar logs |

---

**Generado:** 2026-03-27  
**Versión Docs:** 1.0  
**Estado:** ✅ Ready for Implementation

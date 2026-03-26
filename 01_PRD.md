# PACTA — Product Requirements Document (PRD)
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  
**Autor:** PACTA Team  

---

## 1. Resumen Ejecutivo

**PACTA** es una plataforma SaaS de gestión del ciclo de vida de contratos (CLM — *Contract Lifecycle Management*) diseñada para organizaciones que necesitan controlar, auditar y automatizar sus acuerdos comerciales. La plataforma centraliza la relación entre clientes, proveedores, firmantes autorizados y documentos legales, ofreciendo trazabilidad completa, alertas inteligentes y reportes avanzados.

La versión 2.0 representa una reescritura completa desde cero con una arquitectura moderna, separación clara entre frontend y backend, y soporte para una futura aplicación de escritorio (Windows + Linux).

---

## 2. Problema que Resuelve

Las organizaciones de mediano tamaño gestionan contratos en hojas de cálculo, correos electrónicos y carpetas compartidas. Esto genera:

- **Contratos vencidos sin detectar**, con consecuencias legales y económicas.
- **Falta de trazabilidad** sobre quién modificó qué y cuándo.
- **Búsqueda lenta y difícil** de documentos y cláusulas específicas.
- **Sin control de firmantes autorizados**, exponiendo a la empresa a firmas inválidas.
- **Reportes manuales** que consumen tiempo y tienen errores.

---

## 3. Propuesta de Valor

| Característica | Impacto |
|---|---|
| Alerta automática de vencimientos | Cero contratos vencidos sin atención |
| Trazabilidad completa con auditoría | Cumplimiento legal y corporativo |
| Gestión de firmantes autorizados | Validez jurídica garantizada |
| Suplementos con flujo de aprobación | Modificaciones controladas y rastreadas |
| GraphQL para consultas complejas | Reportes rápidos y flexibles |
| App desktop (fase final) | Acceso offline y uso enterprise |

---

## 4. Usuarios Objetivo

### 4.1 Segmento Primario
- **Empresas medianas** con 20–500 empleados
- Sectores: servicios profesionales, logística, manufactura, construcción, tecnología
- Volumen: 50–2,000 contratos activos

### 4.2 Roles de Usuario

| Rol | Descripción | Permisos |
|---|---|---|
| **Admin** | Administrador del sistema | Acceso total, gestión de usuarios, configuración |
| **Manager** | Gerente / Responsable legal | CRUD completo de contratos, reportes, aprobaciones |
| **Editor** | Asistente legal / Operador | Crear y editar, sin eliminar |
| **Viewer** | Consultor / Directivo | Solo lectura, exportación |

---

## 5. Alcance del Producto

### 5.1 Módulos Core

1. **Contratos** — CRUD completo, filtros avanzados, historial de cambios
2. **Clientes** — Gestión de empresas cliente con contactos y documentos
3. **Proveedores** — Gestión de empresas proveedoras con contactos y documentos
4. **Firmantes Autorizados** — Personas habilitadas para firmar por cada empresa
5. **Suplementos** — Adendas y modificaciones contractuales con workflow
6. **Documentos** — Repositorio de archivos adjuntos por contrato
7. **Reportes** — Análisis financiero, distribución de estado, vencimientos
8. **Notificaciones** — Alertas automáticas por vencimiento y eventos clave
9. **Usuarios** — Gestión de cuentas y roles
10. **Auditoría** — Log de todas las operaciones con usuario, fecha y delta

### 5.2 Fuera de Alcance (v2.0)
- Firma electrónica integrada (planificado v3.0)
- OCR de contratos escaneados (planificado v2.5)
- Integración con ERP/CRM externos (planificado v3.0)
- App móvil nativa (no planificada)

---

## 6. Requerimientos Funcionales

### RF-01: Gestión de Contratos
- El sistema debe permitir crear, editar, ver y eliminar contratos.
- Cada contrato debe tener: número único, título, cliente, proveedor, firmantes (uno por empresa), fechas de inicio y fin, monto, tipo, estado y descripción.
- El sistema debe calcular automáticamente si un contrato está vencido al cruzar la fecha de fin.
- Los contratos deben poder filtrarse por estado, tipo, cliente, proveedor y rango de fechas.
- Los contratos eliminados deben pasar a estado "cancelado" con registro en auditoría (soft delete).

### RF-02: Gestión de Clientes y Proveedores
- CRUD completo para clientes y proveedores.
- Cada entidad tiene: nombre, dirección, código REU/fiscal, contactos, documentos adjuntos.
- No se puede eliminar un cliente/proveedor con contratos activos.

### RF-03: Firmantes Autorizados
- Cada firmante está asociado a una empresa (cliente o proveedor).
- Datos requeridos: nombre, apellido, cargo, teléfono, email, documento de identidad.
- Un contrato requiere exactamente un firmante por parte (cliente + proveedor).

### RF-04: Suplementos
- Los suplementos son modificaciones formales a contratos existentes.
- Flujo: Borrador → Aprobado → Activo.
- Cada suplemento tiene número secuencial por contrato, descripción de modificaciones, fecha de vigencia y firmantes.

### RF-05: Reportes
- Reporte de distribución de contratos por estado (gráfico y tabla).
- Reporte de vencimientos próximos (configurable: 30, 60, 90 días).
- Reporte financiero: valor total, promedio, por cliente/proveedor.
- Exportación en CSV y PDF.

### RF-06: Notificaciones
- Alertas automáticas cuando un contrato vence en los próximos 30 días.
- Notificaciones por suplementos pendientes de aprobación.
- Sistema de marcado de leídas.

### RF-07: Auditoría
- Todo cambio (create, update, delete) genera un registro con: usuario, timestamp, entidad, acción, estado anterior, estado nuevo.
- Los logs son de solo lectura para todos los roles.

### RF-08: Autenticación y Autorización
- Login con email y contraseña.
- JWT con refresh token.
- Sesiones con expiración configurable.
- Cada endpoint de la API valida rol y permiso.

---

## 7. Requerimientos No Funcionales

| Categoría | Requerimiento |
|---|---|
| **Rendimiento** | Respuesta de API < 300ms para el 95% de las peticiones |
| **Disponibilidad** | 99.5% uptime (SaaS) |
| **Seguridad** | HTTPS obligatorio, JWT firmado, contraseñas con bcrypt, SQL paramétrico |
| **Escalabilidad** | Arquitectura stateless, lista para contenedores |
| **Internacionalización** | Soporte español e inglés en la UI |
| **Accesibilidad** | WCAG 2.1 nivel AA |
| **Compatibilidad** | Navegadores modernos (Chrome, Firefox, Edge, Safari últimas 2 versiones) |
| **App Desktop** | Soporte Windows 10+ y Ubuntu 20.04+ |

---

## 8. Métricas de Éxito

| Métrica | Objetivo (6 meses) |
|---|---|
| Usuarios activos mensuales | 500+ |
| Contratos gestionados | 10,000+ |
| Tiempo promedio de creación de contrato | < 3 minutos |
| Tasa de retención mensual | > 85% |
| NPS | > 40 |
| Tiempo de respuesta API (P95) | < 300ms |

---

## 9. Dependencias y Riesgos

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Migración de datos del sistema anterior | Media | Alto | Script de migración + validación manual |
| Adopción del nuevo stack (GraphQL) | Baja | Medio | Documentación interna + REST fallback para operaciones simples |
| Seguridad de documentos adjuntos | Media | Alto | Almacenamiento en objeto con acceso firmado temporalmente |
| Performance con volúmenes altos | Baja | Alto | Índices en BD, paginación obligatoria, caché en queries frecuentes |

---

## 10. Criterios de Aceptación del MVP

- [ ] Login y control de roles funcional
- [ ] CRUD completo de las 5 entidades core (contratos, clientes, proveedores, firmantes, suplementos)
- [ ] Dashboard con KPIs en tiempo real
- [ ] Sistema de notificaciones de vencimiento operativo
- [ ] Exportación de reportes en CSV
- [ ] Registro de auditoría activo en todas las operaciones
- [ ] Desplegado en producción con HTTPS

# PACTA — App Flow
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## 1. Visión General del Flujo

```
Usuario → Autenticación → Dashboard → [Módulos] → Acciones → Auditoría
```

La aplicación sigue un flujo centrado en el contrato. Todas las entidades (clientes, proveedores, firmantes, suplementos, documentos) orbitan alrededor del contrato como entidad principal.

---

## 2. Flujo de Autenticación

```
┌─────────────────────────────────────────┐
│           Pantalla de Login             │
│   Email + Contraseña → [Ingresar]       │
└───────────────┬─────────────────────────┘
                │
        ┌───────▼────────┐
        │  Validación    │
        │  API (POST     │
        │  /auth/login)  │
        └───────┬────────┘
                │
        ┌───────▼────────┐      ┌──────────────────┐
        │ ¿Credenciales  │─ NO →│ Mensaje de error  │
        │  válidas?      │      │ (max 3 intentos)  │
        └───────┬────────┘      └──────────────────┘
               SÍ
                │
        ┌───────▼────────┐
        │  JWT emitido   │
        │  + Refresh     │
        │  Token         │
        └───────┬────────┘
                │
        ┌───────▼────────┐
        │  Redirect →    │
        │  /dashboard    │
        └────────────────┘
```

**Notas:**
- El JWT expira en 15 minutos. El refresh token en 7 días.
- El frontend renueva el JWT automáticamente antes de la expiración.
- Si el refresh token expira, redirige a login.
- Máximo 3 intentos fallidos → bloqueo temporal de 15 minutos.

---

## 3. Arquitectura de Navegación

```
/                       → Redirect a /dashboard (si autenticado) o /login
/login                  → Pantalla de autenticación
/dashboard              → Panel principal con KPIs
/contracts              → Lista de contratos
/contracts/[id]         → Detalle de contrato
/clients                → Lista de clientes
/suppliers              → Lista de proveedores
/authorized-signers     → Lista de firmantes autorizados
/supplements            → Lista de suplementos
/documents              → Repositorio de documentos
/reports                → Centro de reportes
/notifications          → Centro de notificaciones
/users                  → Gestión de usuarios (solo Admin)
/profile                → Perfil del usuario actual
/audit-log              → Registro de auditoría (Manager+)
```

---

## 4. Flujo del Dashboard

```
DASHBOARD
├── KPI Cards (tiempo real via GraphQL subscription)
│   ├── Contratos Activos (count)
│   ├── Vencen en 30 días (count + lista)
│   ├── Suplementos Pendientes (count)
│   └── Valor Total Activos ($)
│
├── Gráfico de distribución por estado (Pie/Donut)
│
├── Tabla: Contratos próximos a vencer (top 5)
│   └── [Ver todos] → /contracts?filter=expiring
│
├── Actividad reciente (últimos 5 cambios del audit log)
│
└── Acciones rápidas
    ├── [+ Nuevo Contrato]
    ├── [+ Nuevo Cliente]
    └── [Ver Reportes]
```

---

## 5. Flujo de Gestión de Contratos

### 5.1 Lista de Contratos
```
/contracts
├── Barra de búsqueda (número, título, cliente, proveedor)
├── Filtros
│   ├── Estado (Activo / Vencido / Pendiente / Cancelado)
│   ├── Tipo (Servicio / Compra / Arrendamiento / Asociación / Empleo / Otro)
│   ├── Cliente
│   ├── Proveedor
│   └── Rango de fechas
├── Tabla paginada (20 por página)
│   └── Columnas: N°, Título, Cliente, Proveedor, Estado, Fecha Fin, Monto
│       └── Acciones: [Ver] [Editar] [Eliminar*]
└── [+ Nuevo Contrato]

*Solo Admin y Manager pueden eliminar
```

### 5.2 Crear / Editar Contrato
```
FORMULARIO DE CONTRATO
├── Información básica
│   ├── Número de contrato (auto-generado, editable)
│   ├── Título
│   └── Descripción
├── Partes
│   ├── Cliente (select con búsqueda)
│   │   └── Firmante del cliente (select filtrado por cliente)
│   ├── Proveedor (select con búsqueda)
│   │   └── Firmante del proveedor (select filtrado por proveedor)
├── Fechas y montos
│   ├── Fecha de inicio
│   ├── Fecha de fin
│   └── Monto ($)
├── Clasificación
│   ├── Tipo de contrato
│   └── Estado inicial
└── [Guardar] → Validación → API → Redirect a /contracts/[id]
```

### 5.3 Detalle de Contrato
```
/contracts/[id]
├── Header: Número, Título, Estado (badge con color)
├── Información completa (read-only o inline-edit)
├── Sección: Suplementos
│   └── Lista + [+ Agregar Suplemento]
├── Sección: Documentos
│   └── Lista de archivos + [+ Subir Documento]
├── Sección: Historial de Auditoría
│   └── Timeline de cambios
└── Acciones: [Editar] [Cancelar Contrato] [Exportar PDF]
```

---

## 6. Flujo de Suplementos

```
Contrato Existente
       │
       └── [+ Agregar Suplemento]
              │
              ▼
       FORMULARIO DE SUPLEMENTO
       ├── N° de Suplemento (auto: SUP-001, SUP-002...)
       ├── Descripción de la modificación
       ├── Fecha de vigencia
       ├── Detalles de las modificaciones (rich text)
       ├── Firmante del cliente
       ├── Firmante del proveedor
       └── Documento adjunto (opcional)
              │
              ▼
         Estado: BORRADOR
              │
         [Aprobar] (Manager+)
              │
              ▼
         Estado: APROBADO
              │
         [Activar] (la fecha de vigencia llegó o es manual)
              │
              ▼
         Estado: ACTIVO
```

---

## 7. Flujo de Documentos

```
SUBIDA DE DOCUMENTO
├── Drag & drop o selección de archivo
├── Validación: tipo (PDF, DOCX, XLSX, JPG, PNG) y tamaño (< 50MB)
├── Asociación a entidad (Contrato / Cliente / Proveedor / Firmante / Suplemento)
├── Upload a storage (S3-compatible)
└── Registro en BD con URL firmada temporal

ACCESO A DOCUMENTO
├── El frontend solicita URL firmada al backend (válida por 1 hora)
├── El usuario descarga directamente desde el storage
└── No hay exposición de URLs permanentes
```

---

## 8. Flujo de Reportes

```
/reports
├── Selector de tipo de reporte
│   ├── Distribución por Estado
│   ├── Vencimientos Próximos
│   ├── Análisis Financiero
│   ├── Por Cliente / Proveedor
│   └── Resumen de Suplementos
├── Filtros de periodo (rango de fechas)
├── Vista previa del reporte (gráficos + tabla)
└── Exportar
    ├── [Descargar CSV]
    └── [Descargar PDF]
```

---

## 9. Flujo de Notificaciones

```
SISTEMA DE NOTIFICACIONES (Automático, ejecuta en background)

Disparadores:
├── Contrato vence en 30 días → Notificación a Manager + Editor del contrato
├── Contrato vence en 7 días → Notificación URGENTE
├── Contrato vencido (día 0) → Notificación + badge rojo en navbar
├── Suplemento en estado BORRADOR por más de 7 días → Recordatorio
└── Usuario creado → Notificación de bienvenida

UI de Notificaciones:
├── Icono en navbar con badge (count no leídas)
├── Panel desplegable (últimas 5)
│   └── [Ver todas] → /notifications
└── /notifications
    ├── Lista completa (paginada)
    ├── Filtro: Leídas / No leídas
    └── [Marcar todas como leídas]
```

---

## 10. Flujo de Gestión de Usuarios (Solo Admin)

```
/users
├── Lista de usuarios (nombre, email, rol, estado, último acceso)
├── [+ Nuevo Usuario]
│   └── Formulario: nombre, email, contraseña temporal, rol
├── [Editar usuario]
│   └── Puede cambiar: nombre, rol, estado (activo/inactivo)
└── [Desactivar] (no elimina, deshabilita el acceso)

Nota: El Admin no puede desactivarse a sí mismo.
```

---

## 11. Flujo de Auditoría

```
Toda acción de escritura genera automáticamente:
{
  user_id: string,
  action: "CREATE" | "UPDATE" | "DELETE",
  entity: "contract" | "client" | "supplier" | ...,
  entity_id: string,
  timestamp: datetime,
  previous_state: JSON | null,
  new_state: JSON | null,
  ip_address: string
}

/audit-log (Manager+)
├── Timeline cronológico inverso
├── Filtros: usuario, entidad, tipo de acción, fecha
└── Exportar CSV del log
```

---

## 12. Estados y Transiciones del Contrato

```
                  ┌──────────┐
         ┌────────│ PENDIENTE│
         │        └────┬─────┘
    [Cancelar]         │ [Activar]
         │             ▼
         │        ┌──────────┐
         │        │  ACTIVO  │─────[Fecha fin]────▶ VENCIDO
         │        └────┬─────┘
         │        [Cancelar]
         │             │
         ▼             ▼
    ┌───────────────────────┐
    │       CANCELADO       │
    └───────────────────────┘

VENCIDO puede volver a ACTIVO solo si se crea un suplemento que extiende la fecha.
```

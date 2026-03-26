# PACTA Docs

Documentación oficial de **PACTA** — Plataforma CLM (Contract Lifecycle Management).

> **Contratos bajo control.**

---

## 📋 Índice de Documentación

| # | Documento | Descripción |
|---|-----------|-------------|
| 01 | [PRD.md](01_PRD.md) | Product Requirements Document — Visión, funcionalidades, requerimientos |
| 02 | [APP_FLOW.md](02_APP_FLOW.md) | Flujos de aplicación, navegación y autenticación |
| 03 | [TECNOLOGIA.md](03_TECNOLOGIA.md) | Stack tecnológico: FastAPI, Flask, GraphQL, PostgreSQL |
| 04 | [CONTEXTO_PROYECTO.md](04_CONTEXTO_PROYECTO.md) | Contexto, historia, principios de diseño |
| 05 | [CASOS_DE_USO.md](05_CASOS_DE_USO.md) | 12 casos de uso detallados |
| 06 | [MVP.md](06_MVP.md) | Alcance del MVP, criterios de aceptación |
| 07 | [BRANDING_DISENO.md](07_BRANDING_DISENO.md) | Identidad de marca y sistema de diseño |
| 08 | [ROADMAP.md](08_ROADMAP.md) | Roadmap v2.0 → v3.2 (3 años) |

---

## 🎯 Propósito

**PACTA** es una plataforma SaaS de gestión del ciclo de vida de contratos diseñada para organizaciones de mediano tamaño (20–500 empleados) que necesitan controlar, auditar y automatizar sus acuerdos comerciales.

### Problemas que resuelve

| Problema | Solución PACTA |
|----------|----------------|
| ❌ Contratos vencidos sin detectar | Alertas automáticas a 30 y 7 días |
| ❌ Sin trazabilidad de cambios | Auditoría completa con usuario, timestamp y delta |
| ❌ Búsqueda lenta de documentos | Búsqueda full-text + filtros avanzados |
| ❌ Firmantes no autorizados | Gestión de firmantes por empresa |
| ❌ Reportes manuales con errores | Reportes automáticos exportables (CSV/PDF) |

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTES                             │
│   ┌─────────────────┐    ┌──────────────────────────────┐  │
│   │   Web App (SaaS)│    │  Desktop App (Win + Linux)   │  │
│   │  Flask + Tailwind│   │  Python + PyWebView          │  │
│   └────────┬────────┘    └──────────────┬───────────────┘  │
└────────────┼───────────────────────────┼────────────────────┘
             │ HTTPS / GraphQL           │ Local API
             ▼                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND (FastAPI)                         │
│   ┌─────────────┐  ┌─────────────┐  ┌──────────────────┐  │
│   │  REST Auth  │  │  GraphQL    │  │  Background Jobs │  │
│   │  /auth/*    │  │  /graphql   │  │  (notificaciones)│  │
│   └─────────────┘  └──────┬──────┘  └──────────────────┘  │
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

## 🚀 Quick Start

### Para nuevos colaboradores

1. Lee el [04_CONTEXTO_PROYECTO.md](04_CONTEXTO_PROYECTO.md) para entender la historia y arquitectura
2. Revisa el [03_TECNOLOGIA.md](03_TECNOLOGIA.md) para conocer el stack completo
3. Estudia el [02_APP_FLOW.md](02_APP_FLOW.md) para comprender los flujos de usuario
4. Consulta el [06_MVP.md](06_MVP.md) para el alcance actual y criterios de aceptación

### Para actualizar documentación

```bash
# Crear rama
git checkout -b docs/tu-cambio

# Editar archivos
# ...

# Commit y push
git add .
git commit -m "docs: descripción del cambio"
git push
```

---

## 📁 Estructura del Repositorio

```
pacta-docs/
├── README.md                 # Este archivo
├── LICENSE                   # Licencia MIT
├── SECURITY.md               # Política de seguridad
├── 01_PRD.md                 # Requisitos del producto
├── 02_APP_FLOW.md            # Flujos de aplicación
├── 03_TECNOLOGIA.md          # Stack tecnológico
├── 04_CONTEXTO_PROYECTO.md   # Contexto y arquitectura
├── 05_CASOS_DE_USO.md        # Casos de uso
├── 06_MVP.md                 # Alcance MVP
├── 07_BRANDING_DISENO.md     # Branding y diseño
└── 08_ROADMAP.md             # Roadmap 3 años
```

---

## 🔗 Repositorios Relacionados

| Repositorio | Descripción | Estado |
|-------------|-------------|--------|
| [pacta_appweb](https://github.com/PACTA-Team/pacta_appweb) | Frontend Flutter (app móvil POS) | ✅ En uso |
| [pacta-backend](https://github.com/PACTA-Team/pacta-backend) | Backend API FastAPI + GraphQL | 🟡 En desarrollo |

---

## 📊 Roadmap Resumen

| Versión | Periodo | Foco | Estado |
|---------|---------|------|--------|
| v2.0 MVP | Semanas 1-8 | Plataforma funcional base | 🟡 En desarrollo |
| v2.1 | Semanas 9-12 | Reportes avanzados + PDF | ⏳ Planificado |
| v2.2 | Semanas 13-18 | Búsqueda full-text + UX | ⏳ Planificado |
| v2.3 | Semanas 19-24 | Colaboración + comentarios | ⏳ Planificado |
| v2.4 | Semanas 25-32 | Multi-tenant + SaaS billing | ⏳ Planificado |
| v2.5 | Semanas 33-40 | Self-host Docker edition | ⏳ Planificado |
| v3.0 | Meses 10-18 | Desktop App (Win + Linux) | ⏳ Planificado |

---

## 📄 Licencia

MIT License — ver [LICENSE](LICENSE) para detalles.

---

**PACTA Team** © 2026 | *Contratos bajo control.*

# Pacta Docs

Documentación oficial de **Pacta** — Sistema POS offline-first para pequeños comercios.

---

## 📋 Índice de Documentación

| # | Documento | Descripción |
|---|-----------|-------------|
| 01 | [PRD.md](01_PRD.md) | Product Requirements Document — Visión, problema, funcionalidades |
| 02 | [APP_FLOW.md](02_APP_FLOW.md) | Flujos de la aplicación y navegación |
| 03 | [TECNOLOGIA.md](03_TECNOLOGIA.md) | Stack tecnológico y dependencias |
| 04 | [CONTEXTO_PROYECTO.md](04_CONTEXTO_PROYECTO.md) | Contexto, arquitectura hexagonal, principios |
| 05 | [CASOS_DE_USO.md](05_CASOS_DE_USO.md) | Casos de uso y escenarios |
| 06 | [MVP.md](06_MVP.md) | Alcance del MVP y métricas de éxito |
| 07 | [BRANDING_DISENO.md](07_BRANDING_DISENO.md) | Identidad de marca y diseño de pantallas |
| 08 | [ROADMAP.md](08_ROADMAP.md) | Hoja de ruta del proyecto |

---

## 🎯 Propósito

Pacta es un sistema de punto de venta (POS) diseñado para negocios minoristas pequeños que operan en entornos con conectividad limitada o nula. Convierte un smartphone Android en una caja registradora completa usando:

- ✅ **100% offline** — Sin dependencia de internet
- 📱 **Escaneo de códigos** — Cámara del dispositivo
- 🖨️ **Impresión Bluetooth** — Impresoras térmicas de bajo costo
- 💾 **Almacenamiento local** — Hive database

---

## 🏗️ Arquitectura

El proyecto sigue **Arquitectura Hexagonal/Clean** con las siguientes capas:

```
┌─────────────────────────────────────┐
│         Presentación (Flutter)       │
│              BLoC / UI              │
├─────────────────────────────────────┤
│         Aplicación (Use Cases)       │
├─────────────────────────────────────┤
│            Dominio (Puro)            │
│     Entidades / Value Objects       │
├─────────────────────────────────────┤
│      Infraestructura (Adaptadores)   │
│   Hive / Bluetooth / Cámara / etc   │
└─────────────────────────────────────┘
```

---

## 🚀 Quick Start

### Para nuevos colaboradores

1. Lee el [04_CONTEXTO_PROYECTO.md](04_CONTEXTO_PROYECTO.md) para entender la arquitectura
2. Revisa el [03_TECNOLOGIA.md](03_TECNOLOGIA.md) para conocer el stack
3. Estudia el [02_APP_FLOW.md](02_APP_FLOW.md) para comprender los flujos
4. Consulta el [06_MVP.md](06_MVP.md) para el alcance actual

### Para actualizar documentación

1. Crea una rama: `git checkout -b docs/tu-cambio`
2. Edita los archivos correspondientes
3. Commit: `git commit -m "docs: descripción del cambio"`
4. Push y abre Pull Request

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
└── 08_ROADMAP.md             # Roadmap
```

---

## 🔗 Repositorios Relacionados

| Repositorio | Descripción |
|-------------|-------------|
| [pacta_appweb](https://github.com/PACTA-Team/pacta_appweb) | Aplicación Flutter (frontend móvil) |
| [pacta-backend](https://github.com/PACTA-Team/pacta-backend) | Backend API (en desarrollo) |

---

## 📄 Licencia

MIT License — ver [LICENSE](LICENSE) para detalles.

---

**PACTA Team** © 2026

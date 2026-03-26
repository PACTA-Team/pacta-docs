# Security Policy

## Reportando una Vulnerabilidad

Si descubres una vulnerabilidad de seguridad en Pacta, por favor repórtala de manera responsable.

### Cómo Reportar

- **GitHub**: Usa la sección [Security](../../security/advisories/new) de este repositorio
- **Email**: Contacta al equipo PACTA directamente

### Qué Incluir

Para ayudarnos a investigar rápidamente, por favor incluye:

1. Descripción de la vulnerabilidad
2. Pasos para reproducir
3. Impacto potencial
4. Versión afectada (si aplica)

### Qué Esperar

- **Acknowledgment**: Respuesta dentro de 48 horas
- **Updates**: Actualizaciones cada 7 días sobre el progreso
- **Resolution**: Timeline basado en la severidad del issue

## Buenas Prácticas de Seguridad

Al contribuir a Pacta:

1. **Nunca cometas datos sensibles** — API keys, credenciales, tokens
2. **Usa variables de entorno** para configuración
3. **Mantén dependencias actualizadas**
4. **Sigue los principios de seguridad offline-first** — los datos financieros son sensibles

## Versiones Soportadas

| Versión | Soportada |
| ------- | --------- |
| latest  | ✅ Sí     |
| < latest| ❌ No     |

## Manejo de Datos

Pacta es una aplicación **offline-first**:

- ✅ Sin telemetría
- ✅ Sin envío de datos a servidores externos (MVP)
- ✅ Almacenamiento local con Hive
- ✅ Permisos mínimos (CÁMARA, BLUETOOTH)

---

**PACTA Team** — Comprometidos con la seguridad y transparencia

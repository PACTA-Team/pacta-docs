# Security Policy

## Reportando una Vulnerabilidad

Si descubres una vulnerabilidad de seguridad en PACTA, por favor repórtala de manera responsable.

### Cómo Reportar

- **GitHub**: Usa la sección [Security](../../security/advisories/new) de este repositorio
- **Email**: Contacta al equipo PACTA directamente

### Qué Incluir

Para ayudarnos a investigar rápidamente, por favor incluye:

1. **Descripción de la vulnerabilidad** — Tipo, ubicación, impacto potencial
2. **Pasos para reproducir** — Instrucciones claras y reproducibles
3. **Impacto potencial** — Qué podría hacer un atacante
4. **Versión afectada** — Si aplica

### Qué Esperar

| Etapa | Timeline |
|-------|----------|
| **Acknowledgment** | Respuesta dentro de 48 horas |
| **Updates** | Actualizaciones cada 7 días sobre el progreso |
| **Resolution** | Timeline basado en la severidad del issue |

---

## Buenas Prácticas de Seguridad

Al contribuir a PACTA:

### 1. Datos Sensibles
- ✅ **Nunca cometas** API keys, credenciales, tokens o secretos
- ✅ **Usa variables de entorno** para configuración sensible
- ✅ **Secretos en producción**: Usa un secrets manager (Vault, AWS Secrets Manager)

### 2. Autenticación y Autorización
- ✅ **JWT firmado** con algoritmo seguro (RS256 o HS256 con clave fuerte)
- ✅ **Refresh tokens rotativos** — invalida el anterior al usar uno nuevo
- ✅ **Rate limiting** en endpoints de autenticación (previene brute force)
- ✅ **Roles y permisos** validados en cada endpoint del backend

### 3. Base de Datos
- ✅ **Queries parametrizadas** — El ORM (SQLAlchemy) previene SQL injection
- ✅ **Sin string interpolation** en queries raw
- ✅ **Índices en campos de búsqueda** — Previene scans lentos en volúmenes altos

### 4. Documentos Adjuntos
- ✅ **URLs pre-firmadas** — Expiran en 1 hora máximo
- ✅ **Sin exposición de URLs permanentes** del object storage
- ✅ **Validación de tipo y tamaño** — Solo PDF, DOCX, XLSX, JPG, PNG (< 50MB)

### 5. Logs y Auditoría
- ✅ **Sin PII en logs** — No loguear emails, contraseñas, tokens
- ✅ **Audit log inmutable** — Solo escritura, nunca edición o delete
- ✅ **IP address registrada** — Para trazabilidad de accesos

---

## Arquitectura de Seguridad

### Transporte
| Capa | Medida |
|------|--------|
| HTTPS/TLS | Obligatorio (Caddy con Let's Encrypt) |
| TLS 1.3 | Configurado en producción |
| HSTS | Habilitado |

### Autenticación
| Capa | Medida |
|------|--------|
| Contraseñas | bcrypt, factor de costo 12 |
| JWT | RS256, expiración 15 min |
| Refresh Token | 7 días, rotativo, revocable |
| Rate Limiting | 10 requests/min en /auth/login |
| Bloqueo | 15 min tras 3 intentos fallidos |

### API
| Capa | Medida |
|------|--------|
| CORS | Configurado por origen permitido |
| GraphQL | Profundidad máx 7, complejidad máx 100 |
| Introspección | Deshabilitada en producción |
| Rate Limiting | 100 requests/min por usuario |

### Base de Datos
| Capa | Medida |
|------|--------|
| ORM | SQLAlchemy con queries parametrizadas |
| Soft Delete | Todas las entidades tienen `deleted_at` |
| Auditoría | Tabla `audit_logs` separada |

### Documentos
| Capa | Medida |
|------|--------|
| Storage | S3-compatible (MinIO self-hosted o AWS S3) |
| Acceso | URLs pre-firmadas (expiran en 1 hora) |
| Bucket | Privado, sin acceso público |

---

## Versiones Soportadas

| Versión | Soportada | Notas |
| ------- | --------- |-------|
| v2.0+   | ✅ Sí     | Versión actual en desarrollo |
| v1.x    | ❌ No     | Legacy (Next.js + Supabase) |

---

## Manejo de Datos Personales

PACTA gestiona datos de clientes, proveedores y firmantes autorizados:

- ✅ **Datos en reposo**: PostgreSQL con acceso restringido
- ✅ **Datos en tránsito**: TLS 1.3 obligatorio
- ✅ **Acceso por rol**: Viewer, Editor, Manager, Admin
- ✅ **Derecho al olvido**: Soft delete con opción de purge manual
- ✅ **Exportación de datos**: CSV export disponible para todos los módulos

---

## Contacto de Seguridad

Para preguntas o reportes de seguridad:

- **GitHub Security Advisories**: [pacta-docs/security](../../security/advisories/new)
- **Email**: security@pacta.example.com (configurar en producción)

---

**PACTA Team** — Comprometidos con la seguridad y transparencia

*Última actualización: 2026-03-26*

# PACTA — Casos de Uso
**Versión:** 2.0  
**Estado:** Producción  
**Fecha:** 2026-03-26  

---

## Convención de Documentación

- **Actor**: Quien ejecuta la acción
- **Precondición**: Estado del sistema antes de la acción
- **Flujo principal**: Secuencia feliz
- **Flujos alternativos**: Variaciones o errores manejados
- **Postcondición**: Estado del sistema tras la acción exitosa

---

## CU-01: Iniciar Sesión

**Actor:** Cualquier usuario registrado  
**Precondición:** El usuario tiene una cuenta activa en el sistema  

**Flujo principal:**
1. El usuario accede a `/login`
2. Ingresa email y contraseña
3. El sistema valida las credenciales contra la base de datos
4. El sistema genera un JWT (15 min) y un refresh token (7 días)
5. El sistema redirige al usuario a `/dashboard`

**Flujos alternativos:**

*FA-01A: Credenciales incorrectas*
- El sistema muestra mensaje de error genérico ("Credenciales inválidas")
- Si es el 3° intento fallido, bloquea la cuenta por 15 minutos

*FA-01B: Cuenta inactiva*
- El sistema informa que la cuenta está deshabilitada y sugiere contactar al administrador

**Postcondición:** El usuario tiene una sesión activa con JWT válido

---

## CU-02: Crear Contrato

**Actor:** Editor, Manager, Admin  
**Precondición:** El usuario está autenticado. Existen al menos un cliente y un proveedor en el sistema  

**Flujo principal:**
1. El usuario navega a `/contracts` y presiona [+ Nuevo Contrato]
2. El sistema muestra el formulario de creación
3. El usuario completa: número de contrato, título, cliente, firmante del cliente, proveedor, firmante del proveedor, fecha inicio, fecha fin, monto, tipo y descripción
4. El usuario presiona [Guardar]
5. El sistema valida todos los campos requeridos
6. El sistema crea el contrato con estado "Pendiente" (por defecto) o "Activo"
7. El sistema genera un registro en `audit_logs` (acción: CREATE)
8. El sistema redirige al usuario a `/contracts/[id]`

**Flujos alternativos:**

*FA-02A: Número de contrato duplicado*
- El sistema muestra error en el campo: "Este número de contrato ya existe"
- El usuario debe corregir antes de guardar

*FA-02B: Fecha de fin anterior a fecha de inicio*
- El sistema muestra error de validación en el campo de fecha fin

*FA-02C: Cliente o proveedor no tiene firmantes registrados*
- El selector de firmante muestra "Sin firmantes disponibles"
- El usuario debe primero crear un firmante para esa empresa

**Postcondición:** El contrato existe en la base de datos. Aparece en la lista de contratos. El audit log tiene el registro de creación.

---

## CU-03: Ver Detalle de Contrato

**Actor:** Cualquier usuario autenticado  
**Precondición:** El contrato existe en el sistema  

**Flujo principal:**
1. El usuario navega a `/contracts/[id]` (via lista o búsqueda)
2. El sistema muestra toda la información del contrato
3. El sistema muestra los suplementos asociados (si existen)
4. El sistema muestra los documentos adjuntos (si existen)
5. El sistema muestra el historial de cambios de auditoría del contrato

**Flujos alternativos:**

*FA-03A: Contrato no encontrado*
- El sistema muestra página 404 con opción de volver al listado

*FA-03B: Viewer intenta acceder a acciones de edición*
- Los botones [Editar] y [Eliminar] no son visibles para el rol Viewer

**Postcondición:** El usuario ve la información completa del contrato

---

## CU-04: Actualizar Estado de Contrato

**Actor:** Manager, Admin  
**Precondición:** El contrato existe. El usuario tiene rol Manager o Admin  

**Flujo principal:**
1. Desde el detalle del contrato, el usuario selecciona un nuevo estado en el selector de estado
2. El sistema solicita confirmación si el cambio es a "Cancelado"
3. El usuario confirma
4. El sistema actualiza el estado del contrato
5. El sistema registra el cambio en audit_logs (estado anterior → nuevo estado)
6. El sistema verifica si debe generar notificaciones (ej: contrato activado)

**Flujos alternativos:**

*FA-04A: Intento de activar un contrato con fecha ya vencida*
- El sistema advierte: "La fecha de fin ya pasó. El contrato pasará a Vencido automáticamente"
- El usuario puede continuar o cancelar

**Postcondición:** El contrato tiene el nuevo estado. El audit log refleja el cambio.

---

## CU-05: Eliminar Contrato (Soft Delete)

**Actor:** Manager, Admin  
**Precondición:** El contrato existe y su estado es "Cancelado" o "Pendiente"  

**Flujo principal:**
1. El usuario presiona [Eliminar] en la lista o el detalle del contrato
2. El sistema muestra un diálogo de confirmación con el nombre del contrato
3. El usuario escribe el número de contrato para confirmar (doble verificación)
4. El sistema marca el contrato como eliminado (`deleted_at = now()`, `status = cancelled`)
5. El sistema registra en audit_logs (acción: DELETE)
6. El sistema redirige a `/contracts`

**Flujos alternativos:**

*FA-05A: Intento de eliminar contrato activo*
- El sistema rechaza la operación: "No es posible eliminar un contrato activo. Primero cancélelo."

**Postcondición:** El contrato no aparece en listados normales. Es visible solo en audit log.

---

## CU-06: Gestionar Suplemento

**Actor:** Editor, Manager, Admin  
**Precondición:** Existe un contrato al que asociar el suplemento  

**Flujo principal (Crear):**
1. Desde `/contracts/[id]`, el usuario presiona [+ Agregar Suplemento]
2. El sistema pre-rellena el número de suplemento (SUP-001, SUP-002...)
3. El usuario completa: descripción, fecha de vigencia, modificaciones detalladas, firmantes y opcionalmente un documento
4. El usuario guarda. El suplemento inicia en estado "Borrador"

**Flujo de aprobación:**
5. Un Manager presiona [Aprobar] → estado cambia a "Aprobado"
6. Al llegar la fecha de vigencia (o manualmente), [Activar] → estado "Activo"

**Flujos alternativos:**

*FA-06A: Editor intenta aprobar un suplemento*
- El botón [Aprobar] no está disponible para el rol Editor

**Postcondición:** El suplemento existe asociado al contrato. El contrato refleja la modificación en su historial.

---

## CU-07: Subir Documento

**Actor:** Editor, Manager, Admin  
**Precondición:** El usuario está en el detalle de un contrato, cliente, proveedor o suplemento  

**Flujo principal:**
1. El usuario arrastra un archivo o presiona [+ Subir Documento]
2. El sistema valida: tipo de archivo (PDF, DOCX, XLSX, JPG, PNG) y tamaño (< 50MB)
3. El sistema sube el archivo al object storage (MinIO)
4. El sistema guarda la referencia en la base de datos (nombre, URL, entidad asociada, usuario, fecha)
5. El documento aparece en la lista de la entidad con opción de descarga

**Flujos alternativos:**

*FA-07A: Archivo muy grande (> 50MB)*
- El sistema rechaza con mensaje: "El archivo excede el tamaño máximo de 50MB"

*FA-07B: Tipo de archivo no permitido*
- El sistema rechaza con mensaje indicando los tipos permitidos

**Postcondición:** El documento está almacenado y accesible desde la entidad asociada.

---

## CU-08: Generar Reporte

**Actor:** Viewer, Editor, Manager, Admin  
**Precondición:** Hay datos suficientes en el sistema para el tipo de reporte solicitado  

**Flujo principal:**
1. El usuario navega a `/reports`
2. Selecciona el tipo de reporte (distribución, vencimientos, financiero, etc.)
3. Configura los filtros (rango de fechas, cliente, proveedor)
4. El sistema ejecuta la query GraphQL correspondiente
5. El sistema renderiza gráficos y tablas con los resultados
6. El usuario puede exportar en CSV o PDF

**Postcondición:** El usuario tiene acceso a los datos del reporte. Si exportó, tiene el archivo descargado.

---

## CU-09: Gestionar Usuarios (Solo Admin)

**Actor:** Admin  
**Precondición:** El usuario tiene rol Admin  

**Flujo principal (Crear usuario):**
1. El Admin navega a `/users` y presiona [+ Nuevo Usuario]
2. Completa: nombre, email, contraseña temporal, rol
3. El sistema crea el usuario
4. El sistema envía notificación de bienvenida al nuevo usuario
5. El nuevo usuario deberá cambiar la contraseña en el primer login

**Flujo alternativo:**

*FA-09A: Email duplicado*
- El sistema informa que ya existe un usuario con ese email

**Postcondición:** El nuevo usuario puede acceder al sistema con sus credenciales.

---

## CU-10: Recibir y Ver Notificaciones

**Actor:** Cualquier usuario autenticado  
**Precondición:** Existen notificaciones generadas para el usuario  

**Flujo principal:**
1. El sistema verifica diariamente (cron a las 8:00am) contratos próximos a vencer
2. Para cada contrato que vence en ≤ 30 días, crea una notificación para managers y editores del sistema
3. El usuario ve un badge con número en el ícono de campana en la navbar
4. El usuario presiona el ícono → ve panel con las últimas 5 notificaciones
5. El usuario presiona [Ver todas] → navega a `/notifications`
6. El usuario puede marcar notificaciones como leídas individualmente o todas a la vez

**Postcondición:** Las notificaciones leídas dejan de contar en el badge.

---

## CU-11: Buscar y Filtrar Contratos

**Actor:** Cualquier usuario autenticado  
**Precondición:** Existen contratos en el sistema  

**Flujo principal:**
1. El usuario navega a `/contracts`
2. Escribe en la barra de búsqueda (búsqueda en tiempo real con debounce 300ms)
3. La búsqueda aplica sobre: número de contrato, título, nombre de cliente, nombre de proveedor
4. El usuario puede combinar búsqueda con filtros de estado, tipo y rango de fechas
5. Los resultados se actualizan sin recargar la página (HTMX)
6. La URL se actualiza con los parámetros de filtro (para compartir/bookmark)

**Postcondición:** El usuario ve la lista filtrada. La URL refleja los filtros activos.

---

## CU-12: Ver Audit Log

**Actor:** Manager, Admin  
**Precondición:** El usuario tiene rol Manager o Admin  

**Flujo principal:**
1. El usuario navega a `/audit-log`
2. El sistema muestra el log cronológico inverso (más reciente primero)
3. El usuario puede filtrar por: usuario que realizó la acción, tipo de entidad, tipo de acción, rango de fechas
4. Cada entrada muestra: usuario, acción, entidad, descripción del cambio, timestamp
5. El usuario puede expandir una entrada para ver el estado anterior y nuevo (JSON diff)
6. El usuario puede exportar el log filtrado a CSV

**Postcondición:** El usuario tiene visibilidad completa de los cambios en el sistema.

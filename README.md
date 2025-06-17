
# 📘 Base de Datos: Sistema de Reportes Ciudadanos

Este proyecto implementa una base de datos PostgreSQL diseñada para gestionar reportes ciudadanos sobre situaciones de interés público (fallas, quejas, incidentes), con enfoque en **control de acceso, trazabilidad y auditoría**.

## 🧱 Estructura General

La base de datos está dividida en **tres grandes bloques**:

---

### 1. 🔢 Tablas de Catálogo y Normalización

Estas tablas definen valores fijos que se referencian en otras partes del sistema:

- **`roles`**: define los distintos roles de usuario (admin, moderador, ciudadano, etc.).
- **`estados_reporte`**: representa el estado de un reporte (pendiente, en proceso, resuelto).
- **`categorias_reporte`**: clasifica los reportes por tipo (infraestructura, seguridad, salud, etc.).

Cada una incluye campos de auditoría como `activo`, `creado_en`, `actualizado_en`.

---

### 2. 👥 Tablas Principales

Son las entidades centrales del sistema:

- **`usuarios`**: almacena datos del usuario, incluyendo su rol, contacto y credenciales.
- **`reportes`**: contiene los reportes realizados por usuarios, incluyendo ubicación geográfica, estado y categoría.
- **`comentarios`**: permite a los usuarios comentar en reportes (útil para seguimiento o debate).

Incluyen control de auditoría completo: marcas de tiempo, `actualizado_por_id`, `activo`, etc.

---

### 3. 🛡️ Seguridad y Auditoría

Diseñado para trazabilidad total:

- **`historial_estados_reporte`**: registra cada cambio de estado de un reporte con quién lo hizo y cuándo.
- **`api_keys`**: gestión de llaves API por usuario, con expiración y trazabilidad.
- **`tokens_revocados`**: mantiene un log de tokens JWT revocados (para invalidación segura).
- **`logs_api`**: almacena peticiones hechas a la API, con datos como IP, método, usuario, y respuesta.

---

## ⚙️ Automatización con Triggers

Se definió una **función PL/pgSQL** (`fn_actualizar_timestamp`) que actualiza automáticamente el campo `actualizado_en` en cada modificación. Esta función está conectada a las tablas mediante **triggers** específicos.

```sql
-- Ejemplo
CREATE TRIGGER tr_actualizar_timestamp_roles
BEFORE UPDATE ON roles
FOR EACH ROW EXECUTE FUNCTION fn_actualizar_timestamp();
```

---

## 🧪 Consideraciones Técnicas

- Se usa el tipo `SERIAL` o `BIGSERIAL` para las llaves primarias.
- Claves foráneas entre tablas garantizan integridad referencial.
- Todos los registros importantes cuentan con campo `activo` para permitir desactivación lógica en lugar de eliminación física.

---

## 📤 ¿Cómo restaurar esta base de datos?

1. Crear una base de datos vacía:
   ```bash
   createdb nombre_base
   ```

2. Ejecutar el script:
   ```bash
   psql -U tu_usuario -d nombre_base -f script.sql
   ```

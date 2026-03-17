# 👑 Riesgo 8 — Exceso de Privilegios en Bases de Datos

**Ámbito:** Backend, base de datos (PostgreSQL, MySQL, Supabase)

---

## El problema

Para facilitar la configuración inicial, la IA suele conectar la aplicación a la base de datos usando el usuario administrador o `root`. Este usuario tiene acceso total: puede crear tablas, borrar datos, modificar permisos y ejecutar cualquier comando estructural.

Si la aplicación sufre una vulnerabilidad de inyección SQL — o si el servidor es comprometido — el atacante hereda todos esos privilegios. Con acceso `root`, puede borrar toda la base de datos, exfiltrar todos los registros o modificar datos de pago.

**Principio de Mínimo Privilegio (PoLP):** cada proceso debe tener únicamente los permisos que necesita para su función, nada más.

---

## ❌ Código inseguro

```javascript
// MAL: conectando con el usuario administrador de PostgreSQL
import { Pool } from 'pg'

const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  user: 'postgres',     // ← usuario root/admin con acceso total
  password: process.env.DB_PASSWORD,
  port: 5432,
})

// Si esta conexión es explotada, el atacante puede:
// DROP TABLE users;
// DELETE FROM orders WHERE 1=1;
// ALTER USER postgres PASSWORD 'atacante123';
```

```sql
-- MAL: un solo usuario para toda la aplicación con todos los permisos
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;
```

---

## ✅ Código seguro

**Paso 1: Crear roles separados en PostgreSQL**

```sql
-- Ejecutar una vez al configurar la base de datos

-- Rol de solo lectura (para reportes, dashboards)
CREATE ROLE app_reader;
GRANT CONNECT ON DATABASE miapp TO app_reader;
GRANT USAGE ON SCHEMA public TO app_reader;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_reader;

-- Rol de escritura (para la API principal)
CREATE ROLE app_writer;
GRANT CONNECT ON DATABASE miapp TO app_writer;
GRANT USAGE ON SCHEMA public TO app_writer;
GRANT SELECT, INSERT, UPDATE ON TABLE users, orders, products TO app_writer;
-- NO se otorga DELETE masivo ni comandos estructurales

-- Rol con borrado controlado (solo tablas específicas)
CREATE ROLE app_cleaner;
GRANT DELETE ON TABLE sessions, temp_uploads TO app_cleaner;
-- NO puede borrar users, orders ni datos críticos

-- Crear usuarios que hereden los roles
CREATE USER api_service WITH PASSWORD 'contraseña_fuerte_generada';
GRANT app_writer TO api_service;

CREATE USER reports_service WITH PASSWORD 'otra_contraseña_fuerte';
GRANT app_reader TO reports_service;
```

**Paso 2: Conectar con el usuario adecuado según el servicio**

```javascript
// Servicio de API principal — puede leer y escribir, no puede eliminar en masa
const apiPool = new Pool({
  connectionString: process.env.DATABASE_URL_API,
  // DATABASE_URL_API usa el usuario api_service (rol app_writer)
})

// Servicio de reportes — solo lectura
const reportsPool = new Pool({
  connectionString: process.env.DATABASE_URL_REPORTS,
  // DATABASE_URL_REPORTS usa el usuario reports_service (rol app_reader)
})
```

**Con Supabase (Row Level Security):**

```sql
-- Habilitar RLS en todas las tablas
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

-- Los usuarios solo pueden ver sus propios datos
CREATE POLICY "users_own_data" ON users
  FOR ALL USING (auth.uid() = id);

-- Los usuarios solo pueden ver y modificar sus propias órdenes
CREATE POLICY "users_own_orders" ON orders
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "users_insert_orders" ON orders
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- NO política de DELETE para órdenes desde el cliente
-- El borrado de órdenes solo pasa por funciones de servidor con rol service_role
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — PRINCIPIO DE MÍNIMO PRIVILEGIO:
Aplica el Principio de Mínimo Privilegio (PoLP) en la configuración de la
base de datos. Crea roles separados según la función: un rol de solo lectura
para consultas, un rol de lectura/escritura para la API principal, y roles
específicos con DELETE solo para las tablas donde es necesario.
Nunca uses el usuario 'postgres', 'root' o el usuario administrador para
la conexión de la aplicación en producción. Si el stack es Supabase, habilita
Row Level Security (RLS) en todas las tablas y define políticas que restrinjan
el acceso por usuario autenticado. Muéstrame el SQL para crear los roles
y las variables de entorno separadas por nivel de acceso.
```

---

## Verificación rápida

```bash
# Busca conexiones usando el usuario root/postgres explícitamente
grep -rn "user.*postgres\|user.*root\|username.*admin" src/ .env

# En Supabase: verifica que RLS está habilitado
# Ejecuta en el SQL Editor de Supabase:
# SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public';
# Todas las tablas deben tener rowsecurity = true
```

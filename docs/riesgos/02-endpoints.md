# 🔓 Riesgo 2 — Endpoints Desprotegidos

**Ámbito:** Backend y Microservicios (Node.js, Express, Fastify, Supabase)

---

## El problema

Cuando le pides a la IA que genere un endpoint de API, el resultado típico es una ruta funcional pero sin ninguna capa de autenticación. Cualquier persona que conozca la URL puede hacer peticiones GET, POST, DELETE directamente contra tu base de datos, sin necesidad de estar logueada.

Esto es especialmente común en:
- Rutas generadas para pruebas que luego llegan a producción
- Endpoints de administración sin protección
- Webhooks internos accesibles desde internet

---

## ❌ Código inseguro

```javascript
// Express — MAL: cualquiera puede acceder a los datos de usuarios
app.get('/api/users', async (req, res) => {
  const users = await db.query('SELECT * FROM users')
  res.json(users)  // ← sin ninguna validación de quién hace la petición
})

app.delete('/api/users/:id', async (req, res) => {
  await db.query('DELETE FROM users WHERE id = $1', [req.params.id])
  res.json({ success: true })  // ← cualquiera puede borrar registros
})
```

---

## ✅ Código seguro

**Con JWT (Node.js/Express):**

```javascript
// middleware/auth.js
import jwt from 'jsonwebtoken'

export function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1]

  if (!token) {
    return res.status(401).json({ error: 'Token requerido' })
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)
    req.user = payload
    next()
  } catch {
    return res.status(401).json({ error: 'Token inválido o expirado' })
  }
}

// routes/users.js — aplicar el middleware antes de la lógica
app.get('/api/users', requireAuth, async (req, res) => {
  const users = await db.query('SELECT id, name, email FROM users')
  res.json(users)
})

app.delete('/api/users/:id', requireAuth, requireRole('admin'), async (req, res) => {
  await db.query('DELETE FROM users WHERE id = $1', [req.params.id])
  res.json({ success: true })
})
```

**Con Supabase (Next.js App Router):**

```typescript
// app/api/users/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'

export async function GET() {
  const supabase = createRouteHandlerClient({ cookies })

  // Verificar sesión antes de cualquier otra cosa
  const { data: { session } } = await supabase.auth.getSession()
  if (!session) {
    return Response.json({ error: 'No autorizado' }, { status: 401 })
  }

  const { data: users, error } = await supabase
    .from('users')
    .select('id, name, email')  // solo los campos necesarios

  if (error) return Response.json({ error: 'Error interno' }, { status: 500 })
  return Response.json(users)
}
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — SEGURIDAD DE ENDPOINTS:
Todo endpoint que generes para este backend debe requerir autenticación
obligatoria antes de ejecutar cualquier lógica o acceder a la base de datos.
Implementa un middleware de validación de tokens (JWT o Supabase session)
que se aplique antes de cada handler. Las rutas deben devolver HTTP 401 si
el token está ausente, y HTTP 403 si el usuario autenticado no tiene
permisos para esa operación. Si necesitas crear una ruta pública (sin auth),
avísame explícitamente con el motivo antes de generarla.
```

---

## Verificación rápida

```bash
# Busca rutas sin middleware de autenticación
grep -n "app.get\|app.post\|app.put\|app.delete" routes/*.js | grep -v "requireAuth\|authMiddleware\|verifyToken"
```

Toda línea que aparezca en ese resultado es una ruta potencialmente desprotegida.

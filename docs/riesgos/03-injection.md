# 💉 Riesgo 3 — Inyección de Código (Falta de Input Validation)

**Ámbito:** Fullstack — desde el formulario en frontend hasta la recepción en backend

---

## El problema

Confiar en lo que el usuario escribe es un error fatal. Si los datos del formulario se usan directamente en una consulta SQL, un comando del sistema o un template de HTML, un atacante puede inyectar código malicioso que se ejecuta con los privilegios de tu aplicación.

La IA suele omitir esta capa porque hace el ejemplo más corto y fácil de entender. En producción, eso se traduce en vulnerabilidades explotables.

---

## ❌ Código inseguro

```javascript
// MAL: concatenación directa de input del usuario en SQL
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body
  // Si email = "admin@test.com' OR '1'='1", devuelve todos los usuarios
  const user = await db.query(
    `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`
  )
  res.json(user.rows[0])
})

// MAL: sin validación de formato ni tipo
app.post('/api/orders', async (req, res) => {
  const { userId, amount, productId } = req.body
  // amount podría ser -1000, "DROP TABLE orders", o cualquier cosa
  await db.query(`INSERT INTO orders VALUES (${userId}, ${amount}, ${productId})`)
})
```

---

## ✅ Código seguro

**Con Zod (TypeScript/Node.js):**

```typescript
import { z } from 'zod'

// Define el esquema de validación
const LoginSchema = z.object({
  email: z.string().email('Email inválido').max(255),
  password: z.string().min(8, 'Mínimo 8 caracteres').max(128)
})

const OrderSchema = z.object({
  userId: z.string().uuid('ID inválido'),
  amount: z.number().positive('El monto debe ser positivo').max(100000),
  productId: z.string().uuid('ID de producto inválido')
})

app.post('/api/login', async (req, res) => {
  // Validar y parsear — lanza error si no cumple el esquema
  const result = LoginSchema.safeParse(req.body)
  if (!result.success) {
    return res.status(400).json({ error: 'Datos inválidos', details: result.error.issues })
  }

  const { email, password } = result.data

  // Usar consultas parametrizadas — NUNCA concatenación
  const user = await db.query(
    'SELECT * FROM users WHERE email = $1',
    [email]  // ← el valor va separado, el driver lo escapa automáticamente
  )

  if (!user.rows[0] || !await bcrypt.compare(password, user.rows[0].password_hash)) {
    return res.status(401).json({ error: 'Credenciales incorrectas' })
  }

  res.json({ token: generateJWT(user.rows[0]) })
})
```

**Con Supabase (consultas parametrizadas automáticas):**

```typescript
// Supabase usa consultas parametrizadas internamente — solo valida el input
const OrderSchema = z.object({
  amount: z.number().positive().max(100000),
  productId: z.string().uuid()
})

export async function POST(request: Request) {
  const body = await request.json()
  const result = OrderSchema.safeParse(body)

  if (!result.success) {
    return Response.json({ error: 'Datos inválidos' }, { status: 400 })
  }

  const { data: order, error } = await supabase
    .from('orders')
    .insert(result.data)  // ← datos ya validados y tipados

  if (error) return Response.json({ error: 'Error interno' }, { status: 500 })
  return Response.json(order)
}
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — VALIDACIÓN DE INPUTS:
Implementa validación estricta y sanitización para cada dato que ingrese a
la aplicación, tanto en frontend como en backend. Usa Zod (TypeScript) o
joi/yup (JavaScript) para definir esquemas que rechacen formatos, tipos o
longitudes inesperadas. Nunca construyas consultas SQL concatenando strings:
usa siempre consultas parametrizadas o un ORM. Ningún dato del usuario debe
interactuar con la base de datos sin ser validado y sanitizado previamente
contra inyecciones SQL, NoSQL y XSS.
```

---

## Verificación rápida

```bash
# Busca concatenación directa de variables en strings SQL (señal de alerta)
grep -rn "query\s*[`'\"].*\${" src/
grep -rn "query\s*[`'\"].*\+" src/

# Busca inputs sin validación en handlers de Express
grep -n "req.body\." routes/ | grep -v "Schema\|validate\|parse\|sanitize"
```

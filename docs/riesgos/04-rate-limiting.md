# ⚡ Riesgo 4 — Caídas del Servidor y Abuso de Costos (Rate Limiting)

**Ámbito:** Backend, APIs y Automatizaciones (n8n, Node.js, Edge Functions)

---

## El problema

Sin límites de peticiones, cualquier persona puede atacar tu API con miles de requests por segundo. Esto provoca:

- **Caída del servidor**: el backend no puede responder a usuarios legítimos
- **Costos explosivos**: si la ruta consume APIs de pago (OpenAI, Stripe, SMS), el atacante genera miles de dólares en facturación tuya
- **Fuerza bruta**: un atacante puede probar millones de combinaciones de contraseñas en minutos

La IA omite el rate limiting porque no es parte de la lógica de negocio del ejemplo que le pediste.

---

## ❌ Código inseguro

```javascript
// MAL: el endpoint de login acepta peticiones ilimitadas
// → un atacante puede probar 1M de contraseñas por hora
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body
  const user = await authenticateUser(email, password)
  res.json({ token: user.token })
})

// MAL: el endpoint de IA acepta peticiones ilimitadas
// → un atacante puede generar $10,000 en costos de OpenAI en horas
app.post('/api/generate', async (req, res) => {
  const response = await openai.chat.completions.create({ ... })
  res.json(response)
})
```

---

## ✅ Código seguro

**Con express-rate-limit (Node.js/Express):**

```javascript
import rateLimit from 'express-rate-limit'

// Límite estricto para rutas sensibles
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // ventana de 15 minutos
  max: 10,                    // máximo 10 intentos por IP en esa ventana
  message: { error: 'Demasiados intentos. Intenta en 15 minutos.' },
  standardHeaders: true,      // incluye headers RateLimit-*
  legacyHeaders: false,
  handler: (req, res, next, options) => {
    console.warn(`Rate limit alcanzado: ${req.ip} en ${req.path}`)
    res.status(429).json(options.message)
  }
})

// Límite general para la API
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,  // ventana de 1 minuto
  max: 60,              // 60 peticiones por minuto por IP
  message: { error: 'Límite de peticiones alcanzado. Intenta en un momento.' },
  standardHeaders: true,
  legacyHeaders: false,
})

// Límite específico para rutas con costo (IA, email, SMS)
const expensiveLimiter = rateLimit({
  windowMs: 60 * 60 * 1000,  // ventana de 1 hora
  max: 20,                   // 20 peticiones por hora por usuario autenticado
  keyGenerator: (req) => req.user?.id || req.ip,  // limitar por usuario, no solo IP
  message: { error: 'Límite de uso alcanzado. Disponible en 1 hora.' },
})

// Aplicar a las rutas
app.post('/api/login', loginLimiter, loginHandler)
app.use('/api/', apiLimiter)
app.post('/api/generate', requireAuth, expensiveLimiter, generateHandler)
```

**Con Supabase Edge Functions (Deno):**

```typescript
// Implementación simple con un Map en memoria (para single-instance)
const requestCounts = new Map<string, { count: number; resetAt: number }>()

function checkRateLimit(identifier: string, maxRequests = 60, windowMs = 60000): boolean {
  const now = Date.now()
  const record = requestCounts.get(identifier)

  if (!record || now > record.resetAt) {
    requestCounts.set(identifier, { count: 1, resetAt: now + windowMs })
    return true
  }

  if (record.count >= maxRequests) return false

  record.count++
  return true
}

Deno.serve(async (req) => {
  const clientIP = req.headers.get('x-forwarded-for') || 'unknown'

  if (!checkRateLimit(clientIP)) {
    return new Response(
      JSON.stringify({ error: 'Límite de peticiones alcanzado' }),
      { status: 429, headers: { 'Content-Type': 'application/json' } }
    )
  }

  // lógica del endpoint...
})
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — RATE LIMITING:
Añade obligatoriamente un middleware de Rate Limiting a todas las rutas de
la API. Configura límites diferenciados según la sensibilidad de cada ruta:
- Rutas de autenticación (login, registro, recuperar contraseña): máximo
  10 intentos por IP en ventanas de 15 minutos
- API general: 60 peticiones por minuto por IP
- Rutas con costo externo (OpenAI, SMS, email): 20 peticiones por hora
  por usuario autenticado
Devuelve HTTP 429 con un mensaje genérico cuando se exceda el límite.
Registra en logs los intentos de abuso para monitoreo.
```

---

## Verificación rápida

```bash
# Busca rutas sin rate limiter explícito
grep -n "app.post\|app.get\|app.put\|app.delete" routes/*.js | grep -v "Limiter\|rateLimit"

# Verifica que el paquete está instalado
cat package.json | grep "express-rate-limit\|rate-limit\|upstash"
```

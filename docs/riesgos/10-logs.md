# 📋 Riesgo 10 — Fuga de Datos Sensibles en Logs

**Ámbito:** Backend y Aplicaciones Móviles

---

## El problema

Cuando le pides a la IA que te ayude a depurar un error, su solución más común es agregar `console.log` con el estado completo del objeto. Si ese objeto contiene contraseñas, emails, tokens de sesión o datos de tarjetas, esa información queda guardada en:

- Los archivos de log del servidor (accesibles para cualquiera con acceso al servidor)
- Los servicios de logging en la nube (Datadog, Logtail, CloudWatch) donde múltiples personas pueden verlos
- El historial de la terminal del desarrollador en su máquina local

Los logs son frecuentemente el primer lugar que revisa un auditor de seguridad — y el último lugar que el equipo de desarrollo piensa en limpiar.

---

## ❌ Código inseguro

```javascript
// MAL: logueando el objeto completo del usuario (incluye password hash, tokens)
app.post('/api/login', async (req, res) => {
  const user = await findUser(req.body.email)
  console.log('Usuario encontrado:', user)
  // → { id: '123', email: 'user@example.com', password_hash: '$2b$10$...',
  //     session_token: 'eyJhbGci...', phone: '+1234567890' }

  const { password } = req.body
  console.log('Procesando login para:', req.body)
  // → { email: 'user@example.com', password: 'miContraseña123' }
})

// MAL: logueando headers con tokens de autorización
app.use((req, res, next) => {
  console.log('Request recibido:', req.headers)
  // → { authorization: 'Bearer eyJhbGci...TOKEN_COMPLETO...' }
  next()
})
```

```dart
// MAL en Flutter: print con datos sensibles
print('Response del servidor: ${response.body}')
// Si body incluye tokens o datos personales, quedan en los logs del dispositivo
```

---

## ✅ Código seguro

**Funciones de enmascaramiento:**

```typescript
// utils/logger.ts

// Enmascara un string dejando solo los últimos N caracteres visibles
function maskString(value: string, visibleChars = 4): string {
  if (!value || value.length <= visibleChars) return '****'
  return `****${value.slice(-visibleChars)}`
}

// Enmascara un email mostrando solo las primeras 2 letras y el dominio
function maskEmail(email: string): string {
  const [local, domain] = email.split('@')
  if (!domain) return '****'
  return `${local.slice(0, 2)}****@${domain}`
}

// Sanitiza un objeto removiendo campos sensibles antes de loguearlo
function sanitizeForLog(obj: Record<string, unknown>): Record<string, unknown> {
  const sensitiveFields = [
    'password', 'password_hash', 'passwordHash',
    'token', 'accessToken', 'refreshToken', 'sessionToken',
    'secret', 'apiKey', 'api_key',
    'creditCard', 'cardNumber', 'cvv',
    'ssn', 'sin', 'taxId',
    'authorization',
  ]

  const sanitized: Record<string, unknown> = {}

  for (const [key, value] of Object.entries(obj)) {
    const isSecret = sensitiveFields.some(field =>
      key.toLowerCase().includes(field.toLowerCase())
    )

    if (isSecret) {
      sanitized[key] = '[REDACTED]'
    } else if (key === 'email' && typeof value === 'string') {
      sanitized[key] = maskEmail(value)
    } else if (typeof value === 'object' && value !== null) {
      sanitized[key] = sanitizeForLog(value as Record<string, unknown>)
    } else {
      sanitized[key] = value
    }
  }

  return sanitized
}

// Logger seguro que reemplaza console.log en producción
export const logger = {
  info: (message: string, data?: Record<string, unknown>) => {
    const safe = data ? sanitizeForLog(data) : undefined
    console.log(`[INFO] ${message}`, safe ?? '')
  },
  error: (message: string, error?: Error, data?: Record<string, unknown>) => {
    const safe = data ? sanitizeForLog(data) : undefined
    // En producción: nunca loguear el stack trace completo hacia el cliente
    console.error(`[ERROR] ${message}`, {
      message: error?.message,
      // stack solo en desarrollo
      ...(process.env.NODE_ENV === 'development' && { stack: error?.stack }),
      ...safe,
    })
  },
  warn: (message: string, data?: Record<string, unknown>) => {
    const safe = data ? sanitizeForLog(data) : undefined
    console.warn(`[WARN] ${message}`, safe ?? '')
  },
}
```

**Uso correcto:**

```typescript
// BIEN: usar el logger seguro en lugar de console.log
app.post('/api/login', async (req, res) => {
  try {
    const user = await findUser(req.body.email)
    // logger.info sanitiza automáticamente los campos sensibles
    logger.info('Intento de login', { userId: user.id, email: user.email })
    // → [INFO] Intento de login { userId: '123', email: 'us****@example.com' }

    // Lógica de autenticación...

  } catch (error) {
    logger.error('Error en login', error as Error, { email: req.body.email })
    res.status(500).json({ error: 'Error interno' })
  }
})

// BIEN: loguear headers sin el token completo
app.use((req, res, next) => {
  const authHeader = req.headers.authorization
  logger.info('Request recibido', {
    method: req.method,
    path: req.path,
    hasAuth: !!authHeader,
    tokenPreview: authHeader ? maskString(authHeader, 6) : 'none'
  })
  next()
})
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — SEGURIDAD EN LOGS:
Queda estrictamente prohibido registrar en consola o archivos cualquier tipo
de Información Personal Identificable (PII), contraseñas, tokens de sesión,
hashes de contraseñas o datos financieros. Crea una función de logger seguro
que sanitice automáticamente los campos sensibles antes de registrarlos.
Los campos como 'password', 'token', 'accessToken', 'refreshToken',
'secret', 'apiKey', 'creditCard', 'ssn' deben aparecer siempre como
'[REDACTED]' en los logs. Los emails deben mostrarse parcialmente enmascarados.
Los stack traces solo deben ser visibles en entorno de desarrollo,
nunca en producción ni en respuestas al cliente.
```

---

## Auditoría de logs existentes

```bash
# Busca console.log con objetos de usuario o respuestas completas
grep -rn "console\.log" src/ | grep -E "user|password|token|response|body|headers"

# Busca prints en Flutter con datos potencialmente sensibles
grep -rn "print(" lib/ | grep -E "user|token|password|response|email"

# Listar todos los console.log en el proyecto para revisión manual
grep -rn "console\.\(log\|error\|warn\|debug\)" src/ --include="*.ts" --include="*.js"
```

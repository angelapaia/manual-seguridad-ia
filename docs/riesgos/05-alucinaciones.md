# 🧠 Riesgo 5 — Alucinaciones de Código y Errores Silenciosos

**Ámbito:** Arquitectura general — cualquier módulo generado por IA

---

## El problema

Los modelos de IA cometen dos tipos de errores que son peligrosos en producción:

1. **Alucinaciones**: inventan métodos de librerías que no existen, usan APIs deprecadas o asumen comportamientos que el SDK no tiene. El código parece correcto pero falla en tiempo de ejecución.

2. **Happy Path**: asumen que todo proceso asíncrono siempre tendrá éxito. No incluyen manejo de errores para el caso en que la API externa falle, la base de datos tarde demasiado, o el formato de respuesta cambie.

El resultado es una aplicación que se rompe silenciosamente en producción sin dar información útil para diagnosticar el problema.

---

## ❌ Código inseguro

```javascript
// MAL: sin manejo de errores — si OpenAI falla, el servidor se cae
app.post('/api/generate', async (req, res) => {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: req.body.messages
  })
  // Si openai lanza una excepción, nada la captura y el proceso muere
  res.json({ text: response.choices[0].message.content })
})

// MAL: función de librería que no existe (alucinación típica)
import { sanitizeAndValidate } from 'zod/utils'  // ← este import no existe
const clean = sanitizeAndValidate(userInput)     // ← falla silenciosamente
```

---

## ✅ Código seguro

```typescript
// BIEN: manejo exhaustivo de errores con información útil para debugging

app.post('/api/generate', requireAuth, expensiveLimiter, async (req, res) => {
  try {
    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: req.body.messages,
      max_tokens: 1000,
    })

    if (!response.choices?.[0]?.message?.content) {
      // La API respondió pero sin el formato esperado
      console.error('Respuesta inesperada de OpenAI:', JSON.stringify(response))
      return res.status(502).json({ error: 'Respuesta inválida del servicio externo' })
    }

    res.json({ text: response.choices[0].message.content })

  } catch (error) {
    // Distinguir tipos de error para responder apropiadamente
    if (error instanceof OpenAI.APIError) {
      if (error.status === 429) {
        return res.status(503).json({ error: 'Servicio temporalmente no disponible' })
      }
      if (error.status === 401) {
        // Error de configuración interna — no exponer al cliente
        console.error('Error de autenticación con OpenAI — revisar API key')
        return res.status(500).json({ error: 'Error interno de configuración' })
      }
    }

    // Error genérico — logear internamente, respuesta genérica al cliente
    console.error('Error en /api/generate:', error)
    return res.status(500).json({ error: 'Error interno. Intenta más tarde.' })
    // ← NUNCA: res.status(500).json({ error: error.message, stack: error.stack })
  }
})
```

**Patrón para verificar que la librería existe antes de usarla:**

```bash
# Antes de usar cualquier import sugerido por la IA, verifica que existe
npm info zod/utils 2>/dev/null || echo "Este paquete/export no existe"

# O en el código, valida el import
import { z } from 'zod'
console.log(Object.keys(z))  // Verifica qué exports realmente existen
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — MANEJO DE ERRORES Y CÓDIGO DEFENSIVO:
Programa a la defensiva en cada bloque que interactúe con APIs externas,
bases de datos o procesos asíncronos. Incluye try/catch exhaustivo que
maneje explícitamente los casos de error más comunes (timeout, 401, 429,
502, formato inesperado de respuesta). Nunca asumas el "Happy Path".
Captura los errores de forma limpia: loguea la información técnica
internamente, pero devuelve mensajes genéricos al cliente sin revelar
el stack trace, mensajes de error del sistema o estructura interna.
Antes de usar métodos de librerías, verifica que existan en la versión
indicada en package.json.
```

---

## Lista de verificación post-generación

Después de que la IA genere código, verifica manualmente:

- [ ] ¿Cada `await` tiene un `try/catch` o manejo de error?
- [ ] ¿Los métodos de librerías existen? Búscalos en la documentación oficial
- [ ] ¿El código maneja el caso en que la API externa devuelva un formato inesperado?
- [ ] ¿Los errores internos no se exponen al cliente?
- [ ] ¿Los timeouts están configurados en llamadas externas?

```typescript
// Siempre configura timeout en llamadas externas
const controller = new AbortController()
const timeoutId = setTimeout(() => controller.abort(), 10000)  // 10 segundos

try {
  const response = await fetch(url, { signal: controller.signal })
} catch (error) {
  if (error.name === 'AbortError') {
    return res.status(504).json({ error: 'El servicio externo tardó demasiado' })
  }
} finally {
  clearTimeout(timeoutId)
}
```

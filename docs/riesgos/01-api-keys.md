# 🔑 Riesgo 1 — Exposición de API Keys y Credenciales

**Ámbito:** Frontend (Web) y Aplicaciones Móviles (React, Flutter, Next.js)

---

## El problema

La IA tiende a escribir el código más corto que funcione. En un componente de React que consume una API externa, eso significa dejar la clave directamente en el archivo `.js` o `.tsx`. Si ese archivo llega al navegador — y lo hace siempre — cualquier persona puede extraerla con las DevTools en segundos.

Con una API Key de OpenAI o Stripe expuesta, un atacante puede:
- Consumir tu cuota generando cargos masivos
- Acceder a datos de tus usuarios
- Hacer peticiones en tu nombre a terceros

---

## ❌ Código inseguro (lo que la IA genera por defecto)

```javascript
// React — MAL: la key queda expuesta en el bundle del cliente
const response = await fetch('https://api.openai.com/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer sk-proj-abc123xyz...',  // ← expuesta
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ model: 'gpt-4o', messages })
})
```

```dart
// Flutter — MAL: la key queda en el binario de la app
final response = await http.post(
  Uri.parse('https://api.openai.com/v1/chat/completions'),
  headers: {'Authorization': 'Bearer sk-proj-abc123xyz...'},  // ← expuesta
);
```

---

## ✅ Código seguro

**Opción A: Variables de entorno en Next.js (recomendado)**

```javascript
// .env.local — nunca va al repositorio
OPENAI_API_KEY=sk-proj-abc123xyz...

// app/api/chat/route.ts — la key vive en el servidor
import OpenAI from 'openai'

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY  // ← solo accesible en el servidor
})

export async function POST(request: Request) {
  const { messages } = await request.json()
  const response = await client.chat.completions.create({
    model: 'gpt-4o',
    messages
  })
  return Response.json(response)
}

// components/Chat.tsx — el cliente llama a tu propio endpoint
const response = await fetch('/api/chat', {  // ← sin keys
  method: 'POST',
  body: JSON.stringify({ messages })
})
```

**Opción B: Edge Function en Supabase**

```typescript
// supabase/functions/chat/index.ts
const OPENAI_KEY = Deno.env.get('OPENAI_API_KEY')  // ← variable de entorno segura

Deno.serve(async (req) => {
  const { messages } = await req.json()
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    headers: { 'Authorization': `Bearer ${OPENAI_KEY}` },
    body: JSON.stringify({ model: 'gpt-4o', messages })
  })
  return new Response(await response.text())
})
```

---

## 📋 Prompt Preventivo

Copia esto y pégalo antes de tu petición cuando vayas a integrar cualquier API externa:

```
CRÍTICO — SEGURIDAD DE CREDENCIALES:
Actúa como un experto en ciberseguridad. Bajo ninguna circunstancia debes
hardcodear API keys, tokens, contraseñas o credenciales en el código del
cliente (Frontend/Móvil). Toda llave debe ser consumida a través de variables
de entorno (.env) accesibles únicamente desde el servidor. Cualquier llamada
a servicios de terceros debe triangularse a través de un backend propio o
Edge Functions. Si el stack es Next.js, usa Route Handlers en /app/api/.
Si es Flutter, usa Supabase Edge Functions. Reescribe el código asegurando
esta separación y avísame si alguna parte requiere exponer una credencial.
```

---

## Verificación rápida

Antes de hacer commit, ejecuta:

```bash
# Busca patrones de keys en el código del cliente
grep -r "sk-" src/
grep -r "Bearer " src/
grep -r "apiKey:" src/
grep -rE "([A-Za-z0-9]{32,})" src/components/  # strings largos sospechosos
```

Si encuentras algo, muévelo a `.env` y accédelo desde el servidor.

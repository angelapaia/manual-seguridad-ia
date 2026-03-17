# 🕷️ Riesgo 7 — Cross-Site Scripting (XSS)

**Ámbito:** Frontend Web (React, Next.js, Vue) y aplicaciones que renderizan contenido dinámico

---

## El problema

XSS ocurre cuando una aplicación muestra en pantalla contenido que contiene código JavaScript malicioso. El script se ejecuta en el navegador del usuario víctima con acceso completo a sus cookies, tokens de sesión y datos de la página.

En aplicaciones que integran IA, el riesgo es doble: el contenido generado por el modelo podría contener scripts maliciosos (si el modelo fue manipulado), y el contenido guardado por usuarios puede ser inyectado en la vista de otros.

---

## ❌ Código inseguro

```jsx
// MAL: renderizar HTML directamente sin sanitizar
function ChatMessage({ content }) {
  return (
    // Si content = "<img src=x onerror='document.cookie'>"
    // → el script se ejecuta en el navegador del usuario
    <div dangerouslySetInnerHTML={{ __html: content }} />
  )
}

// MAL: construir HTML con concatenación
const messageHTML = `<p>${userGeneratedText}</p>`
document.getElementById('chat').innerHTML = messageHTML
```

```dart
// MAL en Flutter WebView: ejecutar contenido sin sanitizar
WebViewController()
  .loadHtmlString('<html><body>${userContent}</body></html>')
```

---

## ✅ Código seguro

**React — renderizar texto plano (la opción más segura):**

```jsx
// BIEN: React escapa el contenido automáticamente cuando usas JSX normal
function ChatMessage({ content }) {
  return (
    <div className="message">
      {content}  {/* ← React escapa los caracteres especiales automáticamente */}
    </div>
  )
}

// Si necesitas saltos de línea:
function ChatMessage({ content }) {
  return (
    <div className="message">
      {content.split('\n').map((line, i) => (
        <span key={i}>{line}<br /></span>
      ))}
    </div>
  )
}
```

**Si necesitas renderizar Markdown o HTML (con sanitización):**

```bash
npm install dompurify marked
npm install -D @types/dompurify
```

```tsx
import DOMPurify from 'dompurify'
import { marked } from 'marked'

interface SafeMarkdownProps {
  content: string
}

function SafeMarkdown({ content }: SafeMarkdownProps) {
  const rawHtml = marked(content) as string

  // DOMPurify elimina cualquier script, event handler o atributo peligroso
  const cleanHtml = DOMPurify.sanitize(rawHtml, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'code', 'pre', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'blockquote'],
    ALLOWED_ATTR: ['class'],
    FORBID_TAGS: ['script', 'style', 'iframe', 'form', 'input'],
    FORBID_ATTR: ['onclick', 'onload', 'onerror', 'onmouseover', 'href', 'src'],
  })

  return (
    <div
      className="prose"
      dangerouslySetInnerHTML={{ __html: cleanHtml }}
    />
  )
}
```

**Configurar Content Security Policy en Next.js:**

```typescript
// next.config.ts — bloquea scripts externos no autorizados
const nextConfig = {
  headers: async () => [
    {
      source: '/(.*)',
      headers: [
        {
          key: 'Content-Security-Policy',
          value: [
            "default-src 'self'",
            "script-src 'self' 'unsafe-inline'",  // ajusta según necesites
            "style-src 'self' 'unsafe-inline'",
            "img-src 'self' data: https:",
            "connect-src 'self' https://api.openai.com https://*.supabase.co",
            "frame-ancestors 'none'",
          ].join('; ')
        },
        {
          key: 'X-Content-Type-Options',
          value: 'nosniff'
        },
        {
          key: 'X-Frame-Options',
          value: 'DENY'
        }
      ]
    }
  ]
}
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — PREVENCIÓN DE XSS:
Escapa y sanitiza SIEMPRE cualquier contenido dinámico, generado por el
usuario o por IA, antes de renderizarlo en la interfaz. En React, usa
JSX normal (que escapa automáticamente) en lugar de dangerouslySetInnerHTML.
Si debes renderizar Markdown o HTML, usa DOMPurify con una allowlist
estricta de tags y atributos permitidos antes de insertar en el DOM.
Configura un Content Security Policy (CSP) en los headers HTTP para
limitar la ejecución de scripts no autorizados. Nunca construyas HTML
con concatenación de strings que incluya variables del usuario.
```

---

## Verificación rápida

```bash
# Busca usos de dangerouslySetInnerHTML sin sanitización
grep -rn "dangerouslySetInnerHTML" src/ | grep -v "DOMPurify\|sanitize"

# Busca innerHTML sin sanitizar
grep -rn "\.innerHTML\s*=" src/ | grep -v "sanitize\|DOMPurify\|textContent"

# Verifica que DOMPurify está instalado si se usa renderizado de HTML
cat package.json | grep "dompurify"
```

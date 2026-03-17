# 🤝 Cómo Contribuir

¡Gracias por querer mejorar este manual! Las contribuciones de la comunidad son la razón por la que este recurso crece.

---

## ¿Qué tipo de contribuciones se aceptan?

- **Nuevos riesgos**: ¿Hay una vulnerabilidad común en desarrollo con IA que no está cubierta?
- **Mejoras a prompts existentes**: ¿Encontraste una formulación más efectiva?
- **Ejemplos de código**: Más stacks, más frameworks, más casos reales
- **Errores**: Código incorrecto, links rotos, información desactualizada
- **Traducciones**: Llevar el manual a otros idiomas

---

## Proceso para contribuir

### 1. Abre un Issue primero

Antes de escribir código o modificar archivos, abre un [Issue](https://github.com/angelapaia/manual-seguridad-ia/issues) describiendo:

- ¿Qué quieres agregar o cambiar?
- ¿Por qué es útil para la comunidad?
- ¿Tienes un ejemplo real del problema?

Esto evita que trabajes en algo que ya está siendo desarrollado.

### 2. Haz un Fork y crea una rama

```bash
git clone https://github.com/tu-usuario/manual-seguridad-ia.git
cd manual-seguridad-ia
git checkout -b feat/nuevo-riesgo-csrf
```

Convención de nombres para ramas:
- `feat/` — nuevo contenido
- `fix/` — corrección de error
- `update/` — mejora de contenido existente
- `translate/` — traducción

### 3. Escribe tu contribución

Si agregas un nuevo riesgo, sigue la estructura de los archivos existentes en `docs/riesgos/`:

```markdown
# [Emoji] Riesgo N — Nombre del Riesgo

**Ámbito:** Área técnica relevante

## El problema
[Descripción del problema]

## ❌ Código inseguro
[Ejemplo real de lo que la IA genera y NO deberías usar]

## ✅ Código seguro
[Solución correcta con código funcional]

## 📋 Prompt Preventivo
[El prompt listo para copiar y pegar]

## Verificación rápida
[Comandos para detectar el problema en código existente]
```

### 4. Envía tu Pull Request

```bash
git add .
git commit -m "feat: agrega riesgo 11 - CSRF en formularios"
git push origin feat/nuevo-riesgo-csrf
```

Luego abre un Pull Request en GitHub describiendo qué cambiaste y por qué.

---

## Estándares de calidad

Para que tu PR sea aceptado:

- [ ] El código de ejemplo funciona (está probado, no es pseudocódigo)
- [ ] El prompt preventivo es específico y accionable
- [ ] El riesgo es relevante para desarrollo asistido por IA (no solo seguridad general)
- [ ] Los comandos de verificación rápida son correctos
- [ ] No hay información personal ni credenciales reales en los ejemplos

---

## Código de conducta

- Las contribuciones deben ser constructivas y orientadas a educar
- No se acepta contenido que facilite ataques contra sistemas reales
- El objetivo es la defensa y la educación, no la ofensiva

---

## Mantenedores

- [@angelapaia](https://github.com/angelapaia) — mantenedor principal

Si tienes dudas, abre un Issue o inicia una discusión en el repositorio.

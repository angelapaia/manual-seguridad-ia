# 🔄 Flujo de Trabajo Diario

Este es el proceso obligatorio de **3 pasos** que todo desarrollador debe seguir al usar un asistente de IA en su día a día. Está diseñado para que la seguridad no sea un obstáculo, sino una parte natural del flujo.

---

## Paso 1 — Preparación (en segundo plano)

Al abrir tu editor, activa los hooks de seguridad. Estos se ejecutan silenciosamente mientras escribes, sin interrumpir tu trabajo.

```bash
# Escáner de seguridad automático tras cada edición
npx claude-code-templates@latest --hook=security/security-scanner

# Verificador de dependencias riesgosas
npx claude-code-templates@latest --hook=security/dependency-checker

# Protección de archivos críticos contra modificaciones accidentales
npx claude-code-templates@latest --hook=security/file-protection
```

> Solo necesitas ejecutar estos comandos una vez por proyecto. Quedan configurados en el entorno local.

---

## Paso 2 — Codificación (prevención activa)

Si vas a pedirle a la IA que genere un módulo crítico — un login, conexión a base de datos, integración de pagos, endpoint de API — **no lo hagas sin un prompt preventivo**.

### ¿Cómo usarlo?

1. Identifica el tipo de módulo que vas a generar
2. Ve a la tabla del [README](../README.md#-los-10-riesgos-críticos) y encuentra el riesgo correspondiente
3. Abre el archivo de ese riesgo y copia el **Prompt Preventivo**
4. Pégalo al inicio de tu petición a la IA

### Ejemplo

```
[Prompt Preventivo - Riesgo 1 pegado aquí]

Ahora, crea un componente de React que consuma la API de OpenAI
para generar resúmenes de texto.
```

La IA tomará las restricciones como parte del contexto y generará código seguro desde el primer intento, en lugar de tener que corregirlo después.

---

## Paso 3 — Auditoría (cierre del ciclo)

**Nunca hagas un commit sin auditar.** Ejecuta la skill maestra antes de subir el código:

```bash
npx github:angelapaia/skill_auditor_seguridad
```

La herramienta asume el rol de un Ingeniero de Seguridad, analiza los archivos modificados y emite un reporte estructurado. Si el reporte indica alertas rojas, corrígelas antes de continuar.

### ¿Qué analiza?

- Credenciales hardcodeadas o expuestas en el frontend
- Endpoints sin middleware de autenticación
- Inputs sin validación o sanitización
- Dependencias con vulnerabilidades conocidas
- Logs con información sensible

---

## Resumen visual

```
Al abrir el editor          Al codificar                Antes del commit
       │                         │                            │
       ▼                         ▼                            ▼
  Activa hooks  ──────►  Pega el prompt preventivo  ──► Ejecuta auditoría
  de seguridad            según el módulo que vayas       y corrige alertas
  (una sola vez)          a generar                        rojas
```

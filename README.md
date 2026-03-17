# 🛡️ Manual de Seguridad para Desarrollo con IA

[![Licencia: MIT](https://img.shields.io/badge/Licencia-MIT-yellow.svg)](LICENSE)
[![Contribuciones bienvenidas](https://img.shields.io/badge/contribuciones-bienvenidas-brightgreen.svg)](CONTRIBUTING.md)
[![Español](https://img.shields.io/badge/idioma-Español-blue.svg)]()
[![Mantenido](https://img.shields.io/badge/mantenido-sí-success.svg)]()

> Estándar de seguridad innegociable para equipos que usan asistentes de IA (Claude Code, Cursor, GitHub Copilot, etc.) en producción.

La IA acelera el desarrollo, pero por defecto prioriza que el código **funcione rápido**, no que sea **seguro**. Este manual desglosa los 10 riesgos más críticos del desarrollo asistido por IA y entrega los prompts exactos para blindar el código desde el primer intento.

---

## 📋 Tabla de Contenidos

- [¿Por qué existe este manual?](#-por-qué-existe-este-manual)
- [Flujo de trabajo diario](#-flujo-de-trabajo-diario)
- [Checklist de pre-despliegue](#-checklist-de-pre-despliegue)
- [Protocolo de emergencia](#-protocolo-de-emergencia)
- [Los 10 Riesgos Críticos](#-los-10-riesgos-críticos)
- [Ecosistema de herramientas](#-ecosistema-de-herramientas)
- [Prompts preventivos (referencia rápida)](#-prompts-preventivos)
- [Contribuir](#-contribuir)

---

## ❓ ¿Por qué existe este manual?

Cuando le pides a una IA que genere un endpoint, un login o una integración de pagos, el resultado funciona — pero raramente cumple con los estándares mínimos de seguridad empresarial. El modelo no sabe si estás en un hackathon o manejando datos de clientes reales.

Este repositorio actúa como una **biblioteca de prompts preventivos**: antes de pedirle a la IA que genere un módulo crítico, copias el prompt correspondiente y lo incluyes en tu petición. La IA escribe código seguro desde el primer intento.

---

## 🔄 Flujo de Trabajo Diario

Ver detalle completo → [`docs/flujo-diario.md`](docs/flujo-diario.md)

```
┌─────────────────────────────────────────────────────────────────┐
│  1. PREPARACIÓN    │  2. CODIFICACIÓN    │  3. AUDITORÍA        │
│                    │                     │                      │
│  Activa los hooks  │  Pega el prompt     │  Ejecuta             │
│  de seguridad en   │  preventivo antes   │  @auditar-seguridad  │
│  segundo plano     │  de cada módulo     │  antes del commit    │
│                    │  crítico            │                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ Checklist de Pre-Despliegue

Ver detalle completo → [`docs/checklist.md`](docs/checklist.md)

Antes de hacer merge o deploy a producción, verifica:

- [ ] Auditoría IA ejecutada → reporte indica **0 Riesgos Críticos**
- [ ] Cero credenciales en el frontend (API Keys, tokens, URIs de BD)
- [ ] Endpoints protegidos con autenticación (JWT / Supabase)
- [ ] Inputs validados y rutas con Rate Limiting
- [ ] Base de datos sin acceso con usuario root

---

## 🚨 Protocolo de Emergencia

Ver detalle completo → [`docs/emergencias.md`](docs/emergencias.md)

**Si se expone una credencial o se vulnera el sistema:**

1. **Revocar** la API Key comprometida en el panel del proveedor
2. **Avisar** al líder técnico de inmediato
3. **Rotar** contraseñas de todos los servicios conectados
4. **Auditar** logs de las últimas horas
5. **Parchear** y redesplegar con nuevas credenciales

---

## 📚 Los 10 Riesgos Críticos

Cada riesgo tiene su propio documento con: descripción del problema, ejemplo de código inseguro ❌, solución correcta ✅ y prompt preventivo listo para copiar.

| # | Riesgo | Ámbito | Archivo |
|---|--------|--------|---------|
| 1 | Exposición de API Keys y Credenciales | Frontend / Móvil | [`docs/riesgos/01-api-keys.md`](docs/riesgos/01-api-keys.md) |
| 2 | Endpoints Desprotegidos | Backend | [`docs/riesgos/02-endpoints.md`](docs/riesgos/02-endpoints.md) |
| 3 | Inyección de Código (Input Validation) | Fullstack | [`docs/riesgos/03-injection.md`](docs/riesgos/03-injection.md) |
| 4 | Caídas por Abuso de API (Rate Limiting) | Backend / APIs | [`docs/riesgos/04-rate-limiting.md`](docs/riesgos/04-rate-limiting.md) |
| 5 | Alucinaciones y Errores Silenciosos | Arquitectura | [`docs/riesgos/05-alucinaciones.md`](docs/riesgos/05-alucinaciones.md) |
| 6 | Inyección de Prompts | Integraciones IA | [`docs/riesgos/06-prompt-injection.md`](docs/riesgos/06-prompt-injection.md) |
| 7 | Cross-Site Scripting (XSS) | Frontend | [`docs/riesgos/07-xss.md`](docs/riesgos/07-xss.md) |
| 8 | Exceso de Privilegios en BD | Backend / BD | [`docs/riesgos/08-privilegios.md`](docs/riesgos/08-privilegios.md) |
| 9 | Dependencias Comprometidas | Entorno | [`docs/riesgos/09-dependencias.md`](docs/riesgos/09-dependencias.md) |
| 10 | Fuga de Datos en Logs | Backend / Móvil | [`docs/riesgos/10-logs.md`](docs/riesgos/10-logs.md) |

---

## 🧰 Ecosistema de Herramientas

### Herramienta principal del equipo

```bash
# Auditoría de seguridad completa antes de cada commit
npx github:JefferCB1/skill_auditor_seguridad
```

### Hooks automáticos (aitmpl.com)

```bash
# Escáner silencioso en tiempo de escritura
npx claude-code-templates@latest --hook=security/security-scanner

# Verificador de dependencias riesgosas
npx claude-code-templates@latest --hook=security/dependency-checker

# Protección de archivos críticos (.env, configs)
npx claude-code-templates@latest --hook=security/file-protection
```

### Agentes de auditoría

```bash
# Auditor de seguridad arquitectónico
npx claude-code-templates@latest --agent=security/security-auditor

# Agente para diseño de prompts seguros
npx claude-code-templates@latest --agent=development/prompt-engineer
```

---

## 📋 Prompts Preventivos

Referencia rápida de todos los prompts → [`prompts/preventivos.md`](prompts/preventivos.md)

---

## 🤝 Contribuir

¿Encontraste un riesgo que no está cubierto? ¿Tienes un mejor prompt? Lee [`CONTRIBUTING.md`](CONTRIBUTING.md).

---

## 📄 Licencia

MIT © [angelapaia](https://github.com/angelapaia)

> Documento original elaborado por Jefferson CB como parte del estándar de seguridad del equipo. Mantenido y extendido por la comunidad.

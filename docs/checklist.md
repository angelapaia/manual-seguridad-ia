# ✅ Checklist de Pre-Despliegue

Copia este checklist en el ticket de tu tarea (Notion, Linear, Jira) y marca cada casilla antes de pasar el código a revisión o producción.

**Ningún PR debe llegar a `main` sin este checklist completado al 100%.**

---

## Checklist

```
SEGURIDAD — Release de Producción
Fecha: ___________   Desarrollador: ___________   PR/Tarea: ___________

AUDITORÍA IA
[ ] Ejecuté @auditar-seguridad en todos los archivos modificados
[ ] El reporte indica "0 Riesgos Críticos"
[ ] Corregí todas las alertas rojas antes de continuar

CREDENCIALES Y SECRETOS
[ ] No hay API Keys, tokens o URIs de BD en el código cliente (React, Flutter, etc.)
[ ] Las variables de entorno están en .env y .env está en .gitignore
[ ] Las llamadas a servicios externos pasan por el backend o Edge Functions
[ ] Verifiqué con `git diff` que no sube ningún .env al repositorio

ENDPOINTS Y AUTENTICACIÓN
[ ] Todas las rutas nuevas requieren token de autenticación (JWT / Supabase)
[ ] No hay rutas públicas accidentales que expongan datos
[ ] El middleware de autenticación se aplica antes de cualquier lógica de negocio

VALIDACIÓN DE ENTRADAS
[ ] Todos los inputs del usuario están siendo validados (formato, tipo, longitud)
[ ] Los datos se sanitizan antes de interactuar con la base de datos
[ ] Los parámetros de URL/query también se validan

RATE LIMITING Y ESTABILIDAD
[ ] Las rutas nuevas tienen Rate Limiting configurado
[ ] Los errores se capturan con try/catch y no exponen el stack trace al cliente
[ ] Se devuelven mensajes de error genéricos al usuario final

BASE DE DATOS
[ ] La conexión usa un rol con permisos mínimos necesarios (no root/admin)
[ ] No hay consultas que construyan SQL con concatenación de strings
[ ] Las migraciones han sido revisadas por otro desarrollador

DEPENDENCIAS
[ ] Las librerías nuevas son conocidas, mantenidas y están en versión estable
[ ] Ejecuté `npm audit` o equivalente y no hay vulnerabilidades críticas

LOGS
[ ] No hay console.log con datos de usuario, tokens o información sensible
[ ] Los logs de producción enmascaran o excluyen PII (emails, contraseñas, tarjetas)

REVISIÓN FINAL
[ ] Otro desarrollador revisó el código antes del merge
[ ] El ambiente de staging fue probado y está estable
```

---

## Cómo usar este checklist en Notion

1. Crea una base de datos de tareas con una propiedad tipo **Checklist** o usa un bloque de texto con los items arriba
2. Asigna la tarea al desarrollador responsable antes del deploy
3. El líder técnico valida que todas las casillas estén marcadas antes de aprobar el merge

---

## Severidades de alerta

| Nivel | Descripción | Acción requerida |
|-------|-------------|------------------|
| 🔴 Crítico | Credencial expuesta, endpoint público sin auth | Bloquea el merge. Corregir inmediatamente |
| 🟠 Alto | Sin rate limiting, logs con PII | Corregir antes del deploy |
| 🟡 Medio | Validación incompleta, permisos amplios | Corregir en el sprint actual |
| 🟢 Bajo | Mejoras de buenas prácticas | Registrar como deuda técnica |

# 📦 Riesgo 9 — Riesgos en la Cadena de Suministro (Dependencias)

**Ámbito:** Configuración de entorno (package.json, pubspec.yaml, requirements.txt)

---

## El problema

La IA puede recomendar librerías que:

1. **Están abandonadas**: sin actualizaciones en años, con vulnerabilidades conocidas sin parchear
2. **Han sido comprometidas**: ataques de supply chain donde un paquete legítimo fue infectado con código malicioso (esto ocurre realmente — ver [event-stream 2018](https://snyk.io/blog/malicious-code-found-in-npm-package-event-stream/))
3. **Son typosquatting**: paquetes con nombres casi idénticos a los populares, creados por atacantes (ej. `lodahs` en lugar de `lodash`)
4. **Tienen conocidas CVEs**: vulnerabilidades públicamente documentadas que el atacante puede explotar

Instalar un paquete malicioso puede comprometer credenciales, instalar backdoors o exfiltrar datos durante el proceso de build.

---

## ❌ Prácticas inseguras

```bash
# MAL: instalar sin verificar nada
npm install some-random-package

# MAL: instalar versiones sin fijar (permite actualizaciones automáticas maliciosas)
# package.json
{
  "dependencies": {
    "lodash": "*",          # ← cualquier versión, incluidas comprometidas
    "express": "latest"     # ← se actualiza automáticamente sin revisión
  }
}

# MAL: ignorar los warnings de npm audit
npm install --legacy-peer-deps  # suprimir warnings
```

---

## ✅ Prácticas seguras

**Antes de instalar cualquier paquete:**

```bash
# 1. Verificar en npm que es el paquete correcto (descargas, autor, fecha)
npm info nombre-del-paquete

# Busca:
# - Descargas semanales (más de 100k = popular y probablemente legítimo)
# - Última actualización (reciente = mantenido)
# - Repositorio de GitHub (¿existe? ¿tiene actividad?)
# - Número de versiones (historial largo = proyecto maduro)

# 2. Auditar antes de instalar
npm install nombre-del-paquete
npm audit  # ejecutar SIEMPRE después de instalar

# 3. Fijar versiones exactas en package.json
npm install nombre-del-paquete --save-exact
# Genera: "nombre-del-paquete": "2.3.1" en lugar de "^2.3.1"
```

**Configurar auditoría automática:**

```json
// package.json — bloquear install si hay vulnerabilidades críticas
{
  "scripts": {
    "preinstall": "npm audit --audit-level=high",
    "postinstall": "npm audit --audit-level=moderate"
  }
}
```

**Para proyectos Flutter:**

```bash
# Verificar el paquete en pub.dev antes de agregarlo
# Busca: puntuación de likes, verificado por el equipo de Dart, última actualización

flutter pub outdated          # ver paquetes desactualizados
dart pub audit                # (si está disponible en tu versión)
```

**Lockfiles — nunca ignorarlos:**

```bash
# SIEMPRE commitear el lockfile al repositorio
git add package-lock.json  # npm
git add yarn.lock          # yarn
git add pnpm-lock.yaml     # pnpm

# El lockfile garantiza que todos instalan exactamente las mismas versiones
# Sin lockfile, un desarrollador nuevo puede instalar una versión comprometida
```

**Revisar permisos de paquetes sospechosos:**

```bash
# Ver qué hace un paquete en sus scripts de instalación
npm pack nombre-del-paquete --dry-run
cat node_modules/nombre-del-paquete/package.json | grep -A5 '"scripts"'

# Los scripts postinstall pueden ejecutar código arbitrario al instalar
# Si ves scripts que hacen curl, wget o acceden a la red → alerta roja
```

---

## 📋 Prompt Preventivo

```
CRÍTICO — SEGURIDAD DE DEPENDENCIAS:
Si recomiendas instalar dependencias o librerías externas, asegúrate de:
1. Recomendar únicamente paquetes con más de 50k descargas semanales en npm,
   con historial de mantenimiento activo (última actualización reciente) y
   repositorio de GitHub con actividad verificable
2. Especificar la versión exacta a instalar (sin ^ ni ~) para evitar
   actualizaciones automáticas no revisadas
3. Incluir el comando de auditoría a ejecutar después de la instalación:
   'npm audit --audit-level=moderate' para Node.js
4. Advertirme explícitamente si el paquete tiene dependencias transitivas
   con CVEs conocidas o si el paquete tiene scripts de postinstall que
   acceden a la red
```

---

## Herramientas de auditoría

```bash
# npm (incluido con Node.js)
npm audit
npm audit fix        # corregir automáticamente vulnerabilidades de bajo riesgo
npm audit fix --force  # (cuidado: puede introducir breaking changes)

# Snyk (más detallado)
npx snyk test
npx snyk monitor     # monitoreo continuo

# Socket.dev (detecta supply chain attacks)
npx @socket/cli check nombre-del-paquete

# retire.js (para librerías de frontend)
npx retire --path src/
```

## Verificación rápida

```bash
# Ejecutar después de cualquier npm install
npm audit --audit-level=high

# Verificar que no hay paquetes con typosquatting
# Lista de nombres similares a populares que deberías tener
echo "Verifica manualmente: lodash vs lodahs, express vs expres, react vs reakt"
```

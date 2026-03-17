# 🚨 Protocolo de Emergencia

Si ocurre un incidente de seguridad — credencial expuesta, brecha de datos, código malicioso en producción — cada minuto cuenta. Sigue estos pasos en orden estricto.

---

## Los 5 pasos

### Paso 1 — Revocar accesos (primero esto, siempre)

Entra al panel del proveedor afectado y **elimina o revoca la credencial comprometida**. No intentes arreglar el código primero.

| Proveedor | Dónde revocar |
|-----------|---------------|
| OpenAI | platform.openai.com → API Keys |
| Supabase | app.supabase.com → Project Settings → API |
| Stripe | dashboard.stripe.com → Developers → API Keys |
| AWS | console.aws.amazon.com → IAM → Access Keys |
| GitHub | github.com → Settings → Developer Settings → Tokens |
| Google Cloud | console.cloud.google.com → Credentials |

> El objetivo es que la llave robada deje de funcionar. Aunque el atacante la tenga, no podrá usarla.

---

### Paso 2 — Avisar al equipo

Notifica al líder técnico con este mensaje (cópialo y complétalo):

```
🚨 INCIDENTE DE SEGURIDAD — [fecha y hora]

Tipo: [API Key expuesta / Brecha de BD / Endpoint vulnerado / otro]
Servicio afectado: [OpenAI / Supabase / Stripe / etc.]
Descubierto por: [tu nombre]
Cómo ocurrió: [descripción breve]
Estado actual: [Key revocada ✅ / Pendiente ⏳]
Próximo paso: [rotación de credenciales / auditoría de logs / etc.]
```

La transparencia rápida reduce el impacto. Ocultar el incidente siempre lo empeora.

---

### Paso 3 — Rotación masiva

Si se expuso la base de datos o un servicio central:

- Cambia las contraseñas de **todos** los servicios conectados a esa misma infraestructura
- Genera nuevas API Keys para todos los servicios afectados
- Actualiza los archivos `.env` en todos los entornos (desarrollo, staging, producción)
- Invalida todas las sesiones activas de usuarios si aplica

---

### Paso 4 — Auditoría de daños

Revisa qué ocurrió mientras la credencial estuvo expuesta:

```bash
# Revisa logs de acceso recientes en tu plataforma
# Busca patrones anómalos: IPs desconocidas, volumen inusual, horarios extraños

# En Supabase: Database → Logs
# En AWS: CloudWatch → Log Groups
# En Vercel: Deployments → Functions → Logs
```

Preguntas clave:
- ¿Hubo peticiones desde IPs desconocidas?
- ¿Se ejecutaron consultas inusuales en la base de datos?
- ¿Aumentó el costo de facturación de algún servicio?
- ¿Se crearon usuarios o registros no autorizados?

---

### Paso 5 — Parchear y redesplegar

1. Corrige el código que causó la exposición
2. Genera nuevas credenciales y actualiza `.env` en producción
3. Despliega el parche verificando que los hooks de seguridad no emitan alertas
4. Ejecuta `@auditar-seguridad` sobre los archivos modificados
5. Documenta el incidente para que no se repita

---

## Escenarios comunes

### Subí un `.env` a GitHub por accidente

```bash
# 1. Revocar todas las keys del .env inmediatamente (ver Paso 1)
# 2. Eliminar el archivo del historial de git
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env" \
  --prune-empty --tag-name-filter cat -- --all

# 3. Forzar push (coordinar con el equipo primero)
git push origin --force --all

# 4. Agregar .env a .gitignore si no estaba
echo ".env" >> .gitignore
echo ".env.local" >> .gitignore
echo ".env.production" >> .gitignore
```

> Revocar las keys es más importante que borrar el historial. Hazlo primero.

### La IA generó código con credenciales hardcodeadas

1. No ejecutar ni hacer commit del código
2. Eliminar las credenciales del archivo
3. Mover los valores a `.env`
4. Agregar el prompt preventivo correspondiente (Riesgo 1) y regenerar

### Un endpoint fue accedido sin autorización

1. Desactivar temporalmente el endpoint si es posible
2. Revisar logs para estimar el alcance
3. Agregar middleware de autenticación
4. Auditar otros endpoints del mismo servicio

---

## Contactos de emergencia por proveedor

Guarda estos enlaces en favoritos:

- **OpenAI incidentes**: status.openai.com
- **Supabase status**: status.supabase.com
- **Stripe radar (fraude)**: dashboard.stripe.com/radar
- **GitHub security advisories**: github.com/security

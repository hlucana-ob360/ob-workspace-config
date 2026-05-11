# OB Big Agency — Libro Maestro
_Actualizado: 2026-05-11_

## Reglas de trabajo
- Claude Chat = diseñar, pensar, crear ideas
- Claude Code = ejecutar y construir
- Hans NO es ingeniero de sistemas — nunca pedirle que pegue código o revise archivos técnicos

## Agencias — directorio completo

| Agencia | Carpeta local | Puerto | URL Cloud Run | Estado | Git remote |
|---|---|---|---|---|---|
| OB Sire Agent | `~/OB-Sire-Agent` | 3002 (prod) / 3001 (.env) | https://ob-sire-agent-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.sireodoo`) | https://github.com/hlucana-ob360/ob-sire-agent |
| OB Finance Report | `~/OB-Finance-Report` | 3001 (prod) / 3000 (.env) | https://ob-finance-report-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.financereport`) | https://github.com/hlucana-ob360/ob-finance-report |
| OB Prospection Agent | `~/OB-ProspectionAgent` | 3004 | https://ob-prospection-agent-923114664136.europe-west1.run.app | ✅ corriendo · **v7.0** (Brief opcional + Discovery como fallback) | https://github.com/hlucana-ob360/ob-prospection-agent |
| OB Executive Board | `~/OB-ExecutiveBoard` | 3005 | https://ob-executive-board-923114664136.europe-west1.run.app _(convención — verificar despliegue)_ | ✅ corriendo · **v7.2** (cache inteligente + trigger automático de inicio) | https://github.com/hlucana-ob360/ob-executive-board |
| OB CRM Agent | `~/OB-CRM-Agent` | 3007 | https://ob-crm-agent-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.crmagent`) | https://github.com/hlucana-ob360/ob-crm-agent |
| OB Content Agent | `~/OB-Big-Agency/ob-content-agent` | 3008 | https://ob-content-agent-923114664136.europe-west1.run.app | ✅ **v1.0 CERRADO 2026-05-10** · pipeline E2E HTML → PDF (Montserrat base64 + `document.fonts.ready`) → GCS `ob-content-agent-pickup` (signed URLs V4) → Canva URL Import. Tokens Canva persistidos en Firestore (`canva_tokens/default`). 8 plantillas HTML branded en `templates/canva/`. Revisión `ob-content-agent-00012-slv`. | https://github.com/hlucana-ob360/ob-content-agent |
| OB Atención Agent (legacy WhatsApp) | `~/OB-Atencion-Agent` | 3002 | https://ob-atencion-agent-923114664136.europe-west1.run.app | ⛔ no corriendo localmente | https://github.com/hlucana-ob360/ob-atencion-agent |
| OB Atención Agent (Telegram v2) | `~/OB-Atencion-Agent-Telegram` | 3002 | _(no desplegado aún — servicio `ob-atencion-agent-telegram`)_ | ⛔ no corriendo localmente | https://github.com/hlucana-ob360/ob-atencion-agent-telegram |
| OB Builder Agent | `~/OB-BuilderAgent` | 3006 (referenciado) | _(solo local — `http://localhost:3006`)_ | ⛔ carpeta vacía / sin código | (sin git) |
| OB SIRE (utilitario descarga) | `~/OB-SIRE` | n/a | n/a | script suelto `descargar-sire.js` (no es agencia) | sin remote |

> ⚠️ Conflicto de puerto: OB-Sire-Agent, OB-Atencion-Agent y OB-Atencion-Agent-Telegram declaran 3002. Producción local lo asigna al Sire Agent vía LaunchAgent.

## Comandos de arranque por agencia
```bash
cd ~/OB-Sire-Agent && claude
cd ~/OB-Finance-Report && claude
cd ~/OB-ProspectionAgent && claude
cd ~/OB-ExecutiveBoard && claude
cd ~/OB-CRM-Agent && claude
cd ~/OB-Big-Agency/ob-content-agent && claude
cd ~/OB-Atencion-Agent && claude
cd ~/OB-Atencion-Agent-Telegram && claude
cd ~/OB-BuilderAgent && claude
```

## Puertos asignados (producción local)
| Puerto | Agencia |
|---|---|
| 3001 | OB Finance Report |
| 3002 | OB Sire Agent (en producción local) — también declarado por las dos Atención Agent (no activos) |
| 3004 | OB Prospection Agent |
| 3005 | OB Executive Board |
| 3006 | OB Builder Agent (reservado, sin código) |
| 3007 | OB CRM Agent |
| 3008 | OB Content Agent |

## Google Cloud
- Proyecto GCP: **ob-360-agents** (project number `923114664136`)
- Region: **europe-west1**
- Registry: `europe-west1-docker.pkg.dev/ob-360-agents/cloud-run-source-deploy/`
- **Runtime SA estándar (post 2026-05-11)**: `923114664136-compute@developer.gserviceaccount.com` (compute default, Client ID `108210977057783116466`). **7/7 agencias** autentican vía ADC contra esta SA.
- **IAM bindings de la compute SA (project-level + resource-level)**:
  - `roles/datastore.user`, `roles/logging.logWriter`, `roles/logging.viewer`, `roles/storage.admin` (project)
  - `roles/iam.serviceAccountTokenCreator` self→self (resource-level sobre sí misma — necesario para DWD vía `iamcredentials.signJwt`)
  - `roles/secretmanager.secretAccessor` por cada uno de los 7 secrets (resource-level)
- **DWD en Workspace Admin**: Client ID `108210977057783116466` autorizado con scope `https://www.googleapis.com/auth/gmail.send` — usado por `ob-executive-board/tools/gmail.js` para impersonar `hlucana@ob-360.com`.
- **SA legacy `ob-finance-report@ob-360-agents.iam.gserviceaccount.com`**: deprecada. Ya no la usa ningún servicio en producción. Su key id `702fcd6b73a383182c2f1abcbd7e47161ef61a6b` fue rotada el 2026-05-11 (incidente de fuga). Mantenida por si hace falta rollback rápido — eliminable en próxima limpieza.

### Estrategia de costos Cloud Run — aplicada 2026-05-10
- **Todas las agencias**: `min-instances=0` (scale-to-zero, cold-start aceptado en pre-prod)
- **ob-atencion-agent**: `cpu-throttling=true` añadido (estaba en `always-allocated` por error → ~$48 USD/mes desperdiciados)
- Ahorro estimado total: ~$48 USD/mes → <$2 USD/mes

### Patrón ADC (Application Default Credentials) — aplicado 2026-05-10 · completado cross-agency 2026-05-11
- **Regla:** NUNCA usar `ob-finance-report-key.json` ni cualquier otro JSON key en Cloud Run
- Clientes GCP se instancian sin parámetros: `new Storage()`, `new BigQuery()`, `new google.auth.GoogleAuth({ scopes })` — **sin** `keyFilename` / `credentials` / `keyFile`
- El runtime SA del servicio Cloud Run hereda automáticamente las credenciales
- Para signed URLs V4: el runtime SA necesita `roles/iam.serviceAccountTokenCreator` self→self
- En local, **no usar** `gcloud auth application-default login` — la policy de Workspace bloquea los scopes sensibles (Drive/Sheets). En local, seguir con SA key via `GOOGLE_SERVICE_ACCOUNT_KEY` apuntando a path; el código en cada agencia detecta JSON vs path vs unset → ADC.
- **Estado por agencia (2026-05-11):**
  - ✅ ob-crm-agent — ADC (compute SA), refactor con detección JSON/path/ADC
  - ✅ ob-prospection-agent — ADC (compute SA), código ya estaba ADC-ready
  - ✅ ob-content-agent — ADC (compute SA), refactor con detección JSON/path/ADC
  - ✅ ob-finance-report — ADC (compute SA), código ya estaba ADC-ready; key file removida de imagen
  - ✅ ob-sire-agent — ADC (compute SA), código ya ADC-ready. Migración a Secret Manager completada 2026-05-11.
  - ✅ ob-atencion-agent — limpieza de env vars (no usa Google APIs en código actual)
  - ✅ ob-executive-board — DWD migrado 2026-05-11 (commit `b2b3d69`). `tools/gmail.js` usa ADC + `iamcredentials.signJwt` + token exchange OAuth2. Verificado end-to-end (message_id `19e18a6707ae0864`).
- **DWD (Domain-Wide Delegation):**
  - ✅ `OB-ExecutiveBoard/tools/gmail.js` — migrado a ADC + signJwt 2026-05-11. **Único consumer activo de DWD en el ecosistema**.
  - ~~`OB-CRM-Agent/tools/calendar.js` y `tools/gmail.js`~~ — deprecados 2026-05-11 (commit `c9ba267`). Stubs que lanzan Error explicativo. `tools/google-auth-key.js` queda orphan en el repo (sin callers).

### Patrón DWD + ADC (referencia técnica)
Para impersonar un usuario humano vía DWD sin keyFile (caso `ob-executive-board/tools/gmail.js`):

1. `GoogleAuth({ scopes: ['cloud-platform'] })` → autentica vía metadata server.
2. `iamcredentials.projects.serviceAccounts.signJwt({ name: 'projects/-/serviceAccounts/<SA_EMAIL>', requestBody: { payload: <JWT con sub=USUARIO_IMPERSONADO> } })` → Google firma con la private key de la SA, sin que el código la vea.
3. POST a `https://oauth2.googleapis.com/token` con `grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer` y `assertion=<JWT firmado>` → access_token impersonado.
4. Llamar la API de destino con `Authorization: Bearer <access_token>`.

**Requisitos**: la SA tiene `roles/iam.serviceAccountTokenCreator` self→self + el Client ID de la SA autorizado en Workspace Admin con los scopes que necesita.

### Secret Manager — migración 2026-05-11
- **7 secrets en producción** (project `ob-360-agents`):
  `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `HUBSPOT_ACCESS_TOKEN`, `TELEGRAM_BOT_TOKEN`, `APOLLO_API_KEY`, `APIFY_API_TOKEN`, `CANVA_CLIENT_SECRET`. Versión 1 cargada desde los env vars previos.
- IAM: `roles/secretmanager.secretAccessor` para `923114664136-compute@developer.gserviceaccount.com` en los 7 secrets.
- **Consumo por servicio (vía `--update-secrets` en Cloud Run):**
  - ob-crm-agent: ANTHROPIC, GEMINI, HUBSPOT
  - ob-atencion-agent: ANTHROPIC, TELEGRAM
  - ob-prospection-agent: ANTHROPIC, APOLLO, APIFY, HUBSPOT, GEMINI
  - ob-content-agent: ANTHROPIC, GEMINI, HUBSPOT, CANVA
  - ob-finance-report: ANTHROPIC
  - ob-sire-agent: ANTHROPIC
  - ob-executive-board: ANTHROPIC, GEMINI, HUBSPOT
- **Regla:** nuevos secretos NUNCA como env var plaintext en Cloud Run. `gcloud secrets create <NAME>` + IAM binding al compute SA + `gcloud run services update --update-secrets=<NAME>=<NAME>:latest`.

### Incidente 2026-05-11 — fuga de SA private key en respuestas HTTP de error
- **Servicio**: ob-crm-agent, revisión `00006-mbj` (deployada con el fix funcional de eliminar contacto/campaña).
- **Bug**: `tools/sheets.js` pasaba `GOOGLE_SERVICE_ACCOUNT_KEY` (env var con JSON inline en Cloud Run) a `keyFilename` de googleapis → `ENOENT` al intentar abrir como path → `err.message` contenía el JSON completo (incluida private_key) → `server.js` `errorResponse` devolvía `err.message` crudo en 500s.
- **Filtración**: cualquier endpoint que tocara Sheets y fallara devolvía la SA key en la respuesta HTTP. También loggeada en Cloud Run stderr.
- **Remediación**:
  1. Rotación inmediata de SA key id `702fcd6b73a383182c2f1abcbd7e47161ef61a6b` de `ob-finance-report@ob-360-agents.iam.gserviceaccount.com`.
  2. Acceso público quitado durante el fix; restaurado tras verificación.
  3. Refactor de `sheets.js` para detección JSON-vs-path-vs-ADC.
  4. `errorResponse` en `server.js` sanitiza 5xx (sólo `"Error interno"`, nunca `err.message` crudo). `NOT_FOUND` tagueado para 404 explícito.
  5. Migración a ADC con compute runtime SA (extensión a 4 servicios más cross-agency).
  6. Borrado de las 7 copias locales de `ob-finance-report-key.json` (todas eran la key revocada).
- **Lección**: nunca devolver `err.message` crudo de librerías externas en respuestas HTTP. Sanitizar en el handler.

### Revisiones activas Cloud Run (2026-05-11)
| Servicio | Revisión | Cambio respecto a 2026-05-10 |
|---|---|---|
| ob-atencion-agent | `00016-9rx` | Limpieza env vars Google leftover + ANTHROPIC/TELEGRAM via Secret Manager |
| ob-content-agent | `00019-7pq` | ADC migration (Drive/Calendar refactor) + 4 secrets via SM |
| ob-crm-agent | `00008-hmn` | Fix fuga SA key (sheets.js + errorResponse sanitización) + delete contacto/campaña + ADC + 3 secrets via SM |
| ob-executive-board | `00024-k4n` | DWD migration (ADC + signJwt) + 3 secrets via SM + key file removida de imagen |
| ob-finance-report | `00011-fz2` | ADC migration (key file removida de imagen) + ANTHROPIC via SM |
| ob-prospection-agent | `00010-6rh` | ADC migration + 5 secrets via SM |
| ob-sire-agent | `00005-mgf` | ADC migration (key file removida de imagen vía .dockerignore + .gcloudignore) + ANTHROPIC via SM. Sheet 1ZxTwsZRY8yp8E6wVL9PMbMT1nigj8ah7hMlEko_P13s upgraded a Editor para compute SA; folder 1fgnKaMNtKg37DwIX2rB-syKZKFkCLQZA compartido. |

## Google Drive — IDs de carpetas

### Workspace estratégico
| Carpeta | ID |
|---|---|
| OB_Workspace_Estrategia (raíz) | `1aZbJ1Ctun1CSmtwSvaU95DFXJzu3KNT` |
| Identidad | `1AaieXZ4U6YGVrMkNi3uTHzb9hrWAfy3z` |
| Operativa | `1hz8z0-JcBZPiFCml7TQswp2y5jcPlBw5` |
| Briefs | `1pR0y8MTe5c4XJKlidnaHz5tD4S6kSA5Q` |

### OB Big Agency (raíz operativa de agencias) — renombrado de `Agencias/` el 2026-05-10
| Carpeta | ID |
|---|---|
| OB_Big_Agency (raíz) | `1hAU9QIPJJN5GsMgjsJR10HTascdw-Kpo` |
| OB_Attention_Agent _(typo `OB_Atention_Agent` corregido el 2026-05-10)_ | `1QrS5O_TcFje4_fpi_FF-vllc_PECuIAP` |
| OB_Content_Agent (raíz) | `1F-oNE_nVLWGHZtqx4EzXqDMTscJeojas` |
| OB_Content_Agent / Estrategias | `1NoSwhhlNNpxknYxwxnAhd1r_gTYH0EYG` |
| OB_Content_Agent / Grillas | `1GFPWpZ7cPiA6jeNay2taEOW3kybT6yLJ` |
| OB_Content_Agent / Piezas | `1Fs9gh80j5iap9j6zuz0JNRqRAtyNDuYt` |
| OB_Content_Agent / Reportes | `1_yYRZsiAn5Xf5MjY4olHTsJqC6y4wHyZ` |

> ⚠️ ID descontinuado el 2026-05-10: `1XiBwHzqxPWIK08YwDobpwblW3mZuPnMy` (carpeta antigua duplicada `OB_CRM_Agent`). No referenciar.

> ⚠️ Las carpetas de OB_Content_Agent están actualmente en *My Drive* — la Service Account no tiene cuota propia y no puede escribir directamente. Mover a Shared Drive o habilitar OAuth delegation antes de producción. Mientras tanto la agencia degrada graceful a `sessions/` local.

## Variables de entorno por agencia
_(solo nombres — los valores viven en cada `.env` local y en Cloud Run secrets)_

### OB Sire Agent (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`
**Plaintext (config)**: `SHEETS_SPREADSHEET_ID`, `GOOGLE_DRIVE_FOLDER_ID`, `RUC`, `NOMBRE_EMPRESA`, `NODE_ENV`
**Local-only (no en Cloud Run)**: `ODOO_PASSWORD`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`, `PORT`

### OB Finance Report (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`
**Plaintext (config)**: `PORT`, `GOOGLE_DRIVE_FOLDER_ID`, `SHEETS_SIRE_ID`, `GOOGLE_DRIVE_REPORTS_FOLDER_ID`, `NODE_ENV`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY_FILE`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER` _(ya no aplica)_

### OB Prospection Agent (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`, `APOLLO_API_KEY`, `APIFY_API_TOKEN`, `HUBSPOT_ACCESS_TOKEN`, `GEMINI_API_KEY`
**Plaintext (config)**: `GOOGLE_CALENDAR_ID`, `WHATSAPP_ESPANA`, `WHATSAPP_PERU`, `PORT`, `NODE_ENV`, `MAX_PROSPECTOS_SESION`, `MAX_CONTACTOS_BD_PROPIA`, `GEMINI_MODEL`, `AI_PROVIDER`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY` _(ahora ADC)_

### OB Executive Board (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `HUBSPOT_ACCESS_TOKEN`
**Plaintext (config)**: `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CALENDAR_ID`, `GMAIL_USER`, `URL_FINANCE_REPORT`, `URL_SIRE_AGENT`, `URL_ATENCION_AGENT`, `URL_PROSPECTION_AGENT`, `URL_BUILDER_AGENT`, `URL_CRM_AGENT`, `NODE_ENV`, `NOMBRE_EMPRESA`, `EMAIL_INFORME`, `GEMINI_MODEL`, `AI_PROVIDER`, `DRIVE_ESTRATEGIA_FOLDER_ID`, `DRIVE_IDENTIDAD_FOLDER_ID`, `DRIVE_OPERATIVA_FOLDER_ID`, `DRIVE_BRIEFS_FOLDER_ID`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY` _(ahora ADC + DWD signJwt)_, `DIA_INFORME`/`HORA_INFORME` _(no se validan en el código actual — pendientes de hardening de idempotencia)_

### OB CRM Agent (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `HUBSPOT_ACCESS_TOKEN`
**Plaintext (config)**: `ENTORNO`, `GEMINI_MODEL`, `AI_PROVIDER`, `GOOGLE_SHEET_ID_DEV`, `GOOGLE_SHEET_ID_PROD`, `GOOGLE_CALENDAR_ID`, `GMAIL_FROM`, `PORT`, `NODE_ENV`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY` _(ahora ADC)_

### OB Content Agent (post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `HUBSPOT_ACCESS_TOKEN`, `CANVA_CLIENT_SECRET`
**Plaintext (config)**: `GEMINI_MODEL`, `AI_PROVIDER`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CALENDAR_ID`, `DRIVE_IDENTIDAD_FOLDER_ID`, `DRIVE_OPERATIVA_FOLDER_ID`, `DRIVE_BRIEFS_FOLDER_ID`, `DRIVE_CONTENT_AGENT_FOLDER_ID`, `DRIVE_CONTENT_ESTRATEGIAS_ID`, `DRIVE_CONTENT_GRILLAS_ID`, `DRIVE_CONTENT_PIEZAS_ID`, `DRIVE_CONTENT_REPORTES_ID`, `URL_EXECUTIVE_BOARD`, `NOMBRE_AGENCIA`, `NOMBRE_EMPRESA`, `EMAIL_HANS`, `CANVA_CLIENT_ID`, `CANVA_REDIRECT_URI`, `PUBLIC_BASE_URL`, `GCS_PICKUP_BUCKET`, `PORT`, `NODE_ENV`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY` _(ahora ADC)_

**Endpoints orquestados estándar** (modo Executive Board):
- `POST /api/recibir-instruccion` — registra directriz del Board
- `POST /api/proponer-plan` — devuelve plan de trabajo (objetivos, KPIs, calendario)
- `POST /api/ejecutar-plan-aprobado` — arranca ejecución tras aprobación de Hans
- `GET  /api/reporte-ejecucion` — KPIs, actividad reciente, estado actual

### OB Atención Agent (legacy)
`WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_VERIFY_TOKEN`, `ANTHROPIC_API_KEY`, `PORT`

### OB Atención Agent (Telegram v2 — desplegado en Cloud Run, post 2026-05-11)
**Via Secret Manager**: `ANTHROPIC_API_KEY`, `TELEGRAM_BOT_TOKEN`
**Plaintext (config)**: `GOOGLE_CALENDAR_ID`, `GMAIL_USER`, `PORT`
**Removidos**: `GOOGLE_SERVICE_ACCOUNT_KEY` _(leftover de versión anterior, el código local actual no usa Google APIs)_

### OB Builder Agent
_(carpeta vacía — sin `.env` ni código)_

## Pendientes activos
- ~~🚨 URGENTE: Executive Board no ejecuta `leer_documentos_estrategicos()` al inicio~~ ✅ **resuelto 2026-05-09**
- ~~OB Content Agent: diseño pendiente~~ ✅ **CERRADO 2026-05-10** (v1.0 deployado, revisión `00019-7pq` tras migración ADC del 2026-05-11)
- ~~ADC migration cross-agency~~ ✅ **resuelto 2026-05-11** (7/7 servicios).
- ~~ob-sire-agent migración~~ ✅ **completado 2026-05-11** (revisión `00005-mgf`).
- ~~CRM `tools/gmail.js` y `tools/calendar.js`~~ ✅ **deprecados 2026-05-11** (commit `c9ba267`).
- ~~ob-executive-board Gmail DWD~~ ✅ **completado 2026-05-11** (revisión `00024-k4n`, commit `b2b3d69`). Patrón ADC + signJwt validado end-to-end.
- ⏸️ **Hardening Executive Board informe (low-pri)**:
  - Guard 409 si ya se envió un informe el mismo día (evitar duplicados como los del 2026-05-11 antes del fix del scheduler).
  - Cálculo de número de semana ISO 8601 en código, no via LLM (hoy genera "Semana 19" / "Semana 20" / "Semana 20, 12 al 18 de mayo" inconsistentemente).
  - Log persistente de envíos (Sheet "Envios_Informes" o Firestore) — auditoría sin depender del filesystem efímero.
- OB Treasury Agent: alcance pendiente
- 🎯 **Comunicación inter-agencias** — arquitectura master pendiente
- Persistencia real del Brief en Cloud Run — usar Shared Drive o Firestore
- OB Builder Agent: carpeta vacía — falta scaffolding completo
- OB Atención Agent: definir cuál versión queda canónica (legacy WhatsApp vs Telegram v2)

## Decisiones arquitectónicas tomadas
- **Brief como memoria persistente** — estándar todas las agencias Arquitectura B
- **HubSpot = base de datos CRM**, Claude vía MCP = inteligencia encima
- Las agencias leen al **OB CRM Agent** directamente, no a HubSpot
- **Cloud Run = producción real** / **Local = Modo Demo**
- **WhatsApp = canal real de negocio**
- **Trello eliminado** → reemplazado por Google Tasks
- **Clay pausado** hasta volumen comercial constante

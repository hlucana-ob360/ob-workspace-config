# OB Big Agency — Libro Maestro
_Actualizado: 2026-05-10_

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
- Service Account: `ob-finance-report@ob-360-agents.iam.gserviceaccount.com`

### Estrategia de costos Cloud Run — aplicada 2026-05-10
- **Todas las agencias**: `min-instances=0` (scale-to-zero, cold-start aceptado en pre-prod)
- **ob-atencion-agent**: `cpu-throttling=true` añadido (estaba en `always-allocated` por error → ~$48 USD/mes desperdiciados)
- Ahorro estimado total: ~$48 USD/mes → <$2 USD/mes

### Patrón ADC (Application Default Credentials) — aplicado 2026-05-10
- **Regla:** NUNCA usar `ob-finance-report-key.json` ni cualquier otro JSON key en Cloud Run
- Clientes GCP se instancian sin parámetros: `new Storage()`, `new BigQuery()`, `new google.auth.GoogleAuth({ scopes })` — **sin** `keyFilename` / `credentials` / `keyFile`
- El runtime SA del servicio Cloud Run hereda automáticamente las credenciales
- Para signed URLs V4: el runtime SA necesita `roles/iam.serviceAccountTokenCreator` self→self
- En local, usar `gcloud auth application-default login` para que ADC funcione fuera de Cloud Run
- **Pendiente DWD (Domain-Wide Delegation):** los siguientes archivos siguen usando JWT con keyFile porque hacen impersonation de usuario; marcados con `TODO(ADC)` en el código a la espera de validar el patrón DWD+ADC con el runtime SA:
  - `OB-ExecutiveBoard/tools/gmail.js` (caso piloto)
  - `OB-CRM-Agent/tools/calendar.js`
  - `OB-CRM-Agent/tools/gmail.js`

### Revisiones activas Cloud Run (2026-05-10)
| Servicio | Revisión |
|---|---|
| ob-atencion-agent | `00014-xjv` |
| ob-content-agent | `00012-slv` |
| ob-crm-agent | `00005-kx7` |
| ob-executive-board | `00023-htd` |
| ob-finance-report | `00010-hsb` |
| ob-prospection-agent | `00008-qfm` |
| ob-sire-agent | `00004-gxg` |

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

### OB Sire Agent
`ANTHROPIC_API_KEY`, `ODOO_PASSWORD`, `PORT`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`, `SHEETS_SPREADSHEET_ID`

### OB Finance Report
`ANTHROPIC_API_KEY`, `PORT`, `GOOGLE_DRIVE_FOLDER_ID`, `GOOGLE_SERVICE_ACCOUNT_KEY_FILE`, `SHEETS_SIRE_ID`, `GOOGLE_DRIVE_REPORTS_FOLDER_ID`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`

### OB Prospection Agent
`ANTHROPIC_API_KEY`, `APOLLO_API_KEY`, `APIFY_API_TOKEN`, `HUBSPOT_ACCESS_TOKEN`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GOOGLE_CALENDAR_ID`, `WHATSAPP_ESPANA`, `WHATSAPP_PERU`, `PORT`, `NODE_ENV`, `MAX_PROSPECTOS_SESION`, `MAX_CONTACTOS_BD_PROPIA`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`

### OB Executive Board
`ANTHROPIC_API_KEY`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CALENDAR_ID`, `GMAIL_USER`, `HUBSPOT_ACCESS_TOKEN`, `URL_FINANCE_REPORT`, `URL_SIRE_AGENT`, `URL_ATENCION_AGENT`, `URL_PROSPECTION_AGENT`, `URL_BUILDER_AGENT`, `URL_CRM_AGENT`, `NODE_ENV`, `PORT`, `NOMBRE_EMPRESA`, `EMAIL_INFORME`, `DIA_INFORME`, `HORA_INFORME`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`, `DRIVE_ESTRATEGIA_FOLDER_ID`, `DRIVE_IDENTIDAD_FOLDER_ID`, `DRIVE_OPERATIVA_FOLDER_ID`, `DRIVE_BRIEFS_FOLDER_ID`

### OB CRM Agent
`ENTORNO`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GOOGLE_SHEET_ID_DEV`, `GOOGLE_SHEET_ID_PROD`, `GOOGLE_CALENDAR_ID`, `GMAIL_FROM`, `HUBSPOT_ACCESS_TOKEN`, `PORT`, `NODE_ENV`

### OB Content Agent
`ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `GEMINI_MODEL`, `AI_PROVIDER`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CALENDAR_ID`, `DRIVE_IDENTIDAD_FOLDER_ID`, `DRIVE_OPERATIVA_FOLDER_ID`, `DRIVE_BRIEFS_FOLDER_ID`, `DRIVE_CONTENT_AGENT_FOLDER_ID`, `DRIVE_CONTENT_ESTRATEGIAS_ID`, `DRIVE_CONTENT_GRILLAS_ID`, `DRIVE_CONTENT_PIEZAS_ID`, `DRIVE_CONTENT_REPORTES_ID`, `HUBSPOT_ACCESS_TOKEN`, `URL_EXECUTIVE_BOARD`, `NOMBRE_AGENCIA`, `NOMBRE_EMPRESA`, `EMAIL_HANS`, `PORT`, `NODE_ENV`

**Endpoints orquestados estándar** (modo Executive Board):
- `POST /api/recibir-instruccion` — registra directriz del Board
- `POST /api/proponer-plan` — devuelve plan de trabajo (objetivos, KPIs, calendario)
- `POST /api/ejecutar-plan-aprobado` — arranca ejecución tras aprobación de Hans
- `GET  /api/reporte-ejecucion` — KPIs, actividad reciente, estado actual

### OB Atención Agent (legacy)
`WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_VERIFY_TOKEN`, `ANTHROPIC_API_KEY`, `PORT`

### OB Atención Agent (Telegram v2)
`TELEGRAM_BOT_TOKEN`, `ANTHROPIC_API_KEY`, `GOOGLE_CALENDAR_ID`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GMAIL_USER`, `PORT`

### OB Builder Agent
_(carpeta vacía — sin `.env` ni código)_

## Pendientes activos
- ~~🚨 URGENTE: Executive Board no ejecuta `leer_documentos_estrategicos()` al inicio~~ ✅ **resuelto 2026-05-09** (Executive Board v7.2 — cache inteligente + trigger automático de inicio)
- ~~OB Content Agent: diseño pendiente~~ ✅ **CERRADO 2026-05-10** (v1.0 deployado a Cloud Run, revisión `ob-content-agent-00012-slv`)
- OB Treasury Agent: alcance pendiente
- 🎯 **Comunicación inter-agencias** — arquitectura master pendiente: bajo mando de OB Executive Board y Hans. Diseño y construcción no iniciados.
- ~~GitHub remote: OB CRM Agent y todas las demás agencias sin remote configurado~~ ✅ **resuelto 2026-05-09** (los 7 repos privados creados en https://github.com/hlucana-ob360)
- ~~OB Prospection Agent: actualizar a estándar v7.0~~ ✅ **resuelto 2026-05-09** (Brief opcional + Discovery como fallback, revision `ob-prospection-agent-00006-kmd`)
- Persistencia real del Brief en Cloud Run — el filesystem es efímero, hoy se pierde tras cold-start o redeploy. Usar Shared Drive (carpeta `Briefs` movida a Drive de equipo) o Firestore.
- OB Builder Agent: carpeta `~/OB-BuilderAgent` vacía — falta scaffolding completo
- OB Atención Agent: definir cuál versión queda canónica (legacy WhatsApp vs Telegram v2 — ambas reclaman puerto 3002)

## Decisiones arquitectónicas tomadas
- **Brief como memoria persistente** — estándar todas las agencias Arquitectura B
- **HubSpot = base de datos CRM**, Claude vía MCP = inteligencia encima
- Las agencias leen al **OB CRM Agent** directamente, no a HubSpot
- **Cloud Run = producción real** / **Local = Modo Demo**
- **WhatsApp = canal real de negocio**
- **Trello eliminado** → reemplazado por Google Tasks
- **Clay pausado** hasta volumen comercial constante

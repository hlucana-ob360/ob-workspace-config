# OB Big Agency — Libro Maestro
_Actualizado: 2026-05-09_

## Reglas de trabajo
- Claude Chat = diseñar, pensar, crear ideas
- Claude Code = ejecutar y construir
- Hans NO es ingeniero de sistemas — nunca pedirle que pegue código o revise archivos técnicos

## Agencias — directorio completo

| Agencia | Carpeta local | Puerto | URL Cloud Run | Estado | Git remote |
|---|---|---|---|---|---|
| OB Sire Agent | `~/OB-Sire-Agent` | 3002 (prod) / 3001 (.env) | https://ob-sire-agent-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.sireodoo`) | https://github.com/hlucana-ob360/ob-sire-agent |
| OB Finance Report | `~/OB-Finance-Report` | 3001 (prod) / 3000 (.env) | https://ob-finance-report-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.financereport`) | https://github.com/hlucana-ob360/ob-finance-report |
| OB Prospection Agent | `~/OB-ProspectionAgent` | 3004 | https://ob-prospection-agent-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.prospectionagent`) | https://github.com/hlucana-ob360/ob-prospection-agent |
| OB Executive Board | `~/OB-ExecutiveBoard` | 3005 | https://ob-executive-board-923114664136.europe-west1.run.app _(convención — verificar despliegue)_ | ✅ corriendo (LaunchAgent `com.ob360.executiveboard`) | https://github.com/hlucana-ob360/ob-executive-board |
| OB CRM Agent | `~/OB-CRM-Agent` | 3007 | https://ob-crm-agent-923114664136.europe-west1.run.app | ✅ corriendo (LaunchAgent `com.ob360.crmagent`) | https://github.com/hlucana-ob360/ob-crm-agent |
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

## Google Cloud
- Proyecto GCP: **ob-360-agents** (project number `923114664136`)
- Region: **europe-west1**
- Registry: `europe-west1-docker.pkg.dev/ob-360-agents/cloud-run-source-deploy/`
- Service Account: `ob-finance-report@ob-360-agents.iam.gserviceaccount.com`

## Google Drive — IDs de carpetas
| Carpeta | ID |
|---|---|
| OB_Workspace_Estrategia (raíz) | `1aZbJ1Ctun1CSmtwSvaU95DFXJzu3KNT` |
| Identidad | `1AaieXZ4U6YGVrMkNi3uTHzb9hrWAfy3z` |
| Operativa | `1hz8z0-JcBZPiFCml7TQswp2y5jcPlBw5` |
| Briefs | `1pR0y8MTe5c4XJKlidnaHz5tD4S6kSA5Q` |

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

### OB Atención Agent (legacy)
`WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_VERIFY_TOKEN`, `ANTHROPIC_API_KEY`, `PORT`

### OB Atención Agent (Telegram v2)
`TELEGRAM_BOT_TOKEN`, `ANTHROPIC_API_KEY`, `GOOGLE_CALENDAR_ID`, `GOOGLE_SERVICE_ACCOUNT_KEY`, `GMAIL_USER`, `PORT`

### OB Builder Agent
_(carpeta vacía — sin `.env` ni código)_

## Pendientes activos
- 🚨 URGENTE: Executive Board no ejecuta `leer_documentos_estrategicos()` al inicio — fix diseñado, pendiente ejecutar
- OB Content Agent: diseño pendiente
- OB Treasury Agent: alcance pendiente
- Comunicación inter-agencias: arquitectura pendiente
- ~~GitHub remote: OB CRM Agent y todas las demás agencias sin remote configurado~~ ✅ **resuelto 2026-05-09** (los 7 repos privados creados en https://github.com/hlucana-ob360)
- OB Prospection Agent: actualizar a estándar v7.0
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

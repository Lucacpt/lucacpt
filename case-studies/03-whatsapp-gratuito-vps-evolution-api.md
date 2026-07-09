# 📱 Notifiche WhatsApp Gratuite — VPS + Evolution API

> **Case Study: Infrastruttura Self-Hosted**
> Stack: Node.js Worker + Evolution API + Redis + Docker · Infra: VPS Aruba (Ubuntu)

---

## Il Problema

Un'attività commerciale necessitava di inviare **reminder automatici via WhatsApp** ai propri clienti (es. appuntamenti del giorno successivo). Le soluzioni SaaS di messaggistica (Twilio, MessageBird, ecc.) hanno costi ricorrenti per messaggio che diventano significativi su volumi anche modesti. Serviva un'alternativa a **costo zero per messaggio**.

## La Soluzione

Un'architettura a 3 livelli completamente **self-hosted** su una VPS economica, che sfrutta le API di WhatsApp Web tramite Evolution API (open-source) per inviare messaggi senza costi per singolo invio.

> ⚠️ Questo progetto non è stato portato a termine per motivi esterni. Viene documentato come case study architetturale per le competenze infrastrutturali acquisite.

---

## Architettura

```
┌──────────────────────────────────────────────────────┐
│               VPS ARUBA (Ubuntu 24.04)                │
│               1 vCPU · 2GB RAM                        │
│                                                        │
│  ┌──────────────────────────────────────────────────┐ │
│  │              DOCKER COMPOSE                       │ │
│  │  ┌────────────────┐    ┌───────────────────────┐ │ │
│  │  │ Evolution API  │◄──►│   Redis 7 (Alpine)    │ │ │
│  │  │  (WhatsApp)    │    │   AOF + LRU 128MB     │ │ │
│  │  │  Max 512MB RAM │    └───────────────────────┘ │ │
│  │  │  Porta 8080    │                               │ │
│  │  │  (solo local)  │                               │ │
│  │  └───────▲────────┘                               │ │
│  └──────────┼────────────────────────────────────────┘ │
│             │                                          │
│  ┌──────────┴────────┐                                 │
│  │  Worker Node.js   │──────► Supabase (Cloud)         │
│  │  gestito da PM2   │        Query appuntamenti       │
│  │  Polling 30 min   │        Aggiorna stati           │
│  └───────────────────┘                                 │
│                                                        │
│  🔒 UFW: porta 8080 bloccata dall'esterno              │
└──────────────────────────────────────────────────────┘
```

---

## I 3 Livelli

### 1. Evolution API (Container Docker)
Gateway open-source che espone REST API per WhatsApp Web. Gestisce la sessione WhatsApp (pairing via QR Code) e l'invio/ricezione messaggi.

| Parametro | Valore |
|-----------|--------|
| Immagine | `atendai/evolution-api:latest` |
| RAM limit | 512MB |
| Porta | 8080 (solo localhost) |
| Autenticazione | API Key generata con `openssl rand -hex 32` |

### 2. Redis (Cache & Sessioni)
Supporto per Evolution API: caching sessioni WhatsApp, persistence AOF per sopravvivere ai riavvii.

| Parametro | Valore |
|-----------|--------|
| Immagine | `redis:7-alpine` |
| Memoria max | 128MB |
| Politica eviction | `allkeys-lru` |
| Healthcheck | `redis-cli ping` ogni 10s |

### 3. Worker Node.js (PM2)
Processo long-running che orchestra il ciclo di notifiche:

```
┌─────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────┐
│  POLL   │────►│  QUERY       │────►│  COMPONI      │────►│  INVIA   │
│ (30min) │     │  Supabase    │     │  Messaggio    │     │  WhatsApp│
└─────────┘     │  Appuntamenti│     │  Personaliz.  │     │  via API │
                │  24-28h      │     └───────────────┘     └─────┬────┘
                └──────────────┘                                  │
                                                                  ▼
                                                         ┌──────────────┐
                                                         │  AGGIORNA    │
                                                         │  Stato DB    │
                                                         │  ✅ inviato  │
                                                         │  ❌ errore   │
                                                         └──────────────┘
```

**Logica di Polling:**
- Finestra temporale: appuntamenti tra 24 e 28 ore da "adesso"
- Filtra solo stati `in_attesa` o `errore` (retry automatico)
- Rate limiting: 2 secondi di pausa tra ogni messaggio (anti-ban WhatsApp)
- Stato tracciato su Supabase: `in_attesa → inviato | errore`

**Messaggio generato:**
```
✨ Promemoria [Nome Attività] ✨

Ciao [Nome]! 👋

Ti ricordiamo il tuo appuntamento:

📅 Martedì 15 luglio
🕐 Ore 10:30
💆 [Trattamento] (~45 min)

Ti aspettiamo! 💕
Per disdire o spostare, rispondi a questo messaggio.
```

---

## Sicurezza

| Misura | Implementazione |
|--------|----------------|
| Evolution API non esposta | UFW blocca porta 8080 dall'esterno |
| Accesso Worker | Solo `localhost:8080` sulla stessa VPS |
| Credenziali Supabase | Service Role Key (bypassa RLS), solo nel Worker |
| API Key Evolution | Generata con entropia crittografica (`openssl rand`) |
| Pairing WhatsApp | QR Code accessibile solo da browser locale/VPN |

---

## Costi

| Voce | Costo |
|------|-------|
| VPS Aruba (1 vCPU, 2GB RAM) | ~€3-5/mese |
| Messaggi WhatsApp | **€0** (nessun costo per invio) |
| Supabase | Free tier sufficiente |
| **Totale** | **~€5/mese** vs €50-200/mese per SaaS equivalenti |

---

## Competenze Acquisite

- Setup e gestione VPS Linux con Docker in produzione
- Self-hosting di Evolution API per WhatsApp Business senza costi
- Architettura Worker con polling, gestione stati asincroni e retry
- Docker Compose con healthcheck, resource limits e network isolation
- Sicurezza: firewall UFW, isolamento rete, gestione secrets
- PM2 per process management Node.js in produzione

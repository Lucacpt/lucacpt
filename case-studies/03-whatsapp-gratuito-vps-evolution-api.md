# Notifiche WhatsApp self-hosted: VPS ed Evolution API

| | |
|---|---|
| Tipo | Promemoria appuntamenti per un centro estetico |
| Stack | Node.js, Evolution API, Redis 7, Docker Compose, PM2, Supabase |
| Infrastruttura | VPS Aruba, Ubuntu 24.04, 1 vCPU, 2 GB RAM |
| Stato | Prototipo, sospeso per motivi esterni al progetto |

Fa parte del gestionale del centro estetico (SvelteKit, Supabase, ruoli admin e cliente). Lo documento per la parte infrastrutturale.

---

## Problema

Il centro voleva mandare ai clienti un promemoria WhatsApp il giorno prima dell'appuntamento. I servizi di messaggistica a pagamento (Twilio, MessageBird e simili) costano per messaggio, e anche su volumi piccoli la spesa mensile non era giustificata.

## Soluzione

Tre componenti su una VPS economica: un gateway WhatsApp open source in container, Redis per le sessioni, e un worker Node.js che legge gli appuntamenti da Supabase e invia i messaggi. Nessuna porta esposta oltre SSH: il worker lavora in pull.

---

## Architettura

```
┌────────────────────────────────────────────────────┐
│ VPS Aruba (Ubuntu 24.04, 1 vCPU, 2 GB)             │
│                                                    │
│  ┌──────────── Docker Compose ──────────────────┐  │
│  │  Evolution API (max 512 MB)  ◄──►  Redis 7   │  │
│  │  porta 8080 solo localhost        AOF, LRU   │  │
│  └──────────────▲───────────────────────────────┘  │
│                 │                                  │
│  ┌──────────────┴─────┐                            │
│  │ Worker Node.js     │ ────► Supabase (cloud)     │
│  │ PM2, ogni 30 min   │       legge appuntamenti,  │
│  └────────────────────┘       aggiorna lo stato    │
│                                                    │
│  UFW: 8080 chiusa dall'esterno                     │
└────────────────────────────────────────────────────┘
```

---

## Componenti

**Evolution API**

| Parametro | Valore |
|-----------|--------|
| Ruolo | Gateway REST verso WhatsApp Web, abbinamento con QR code |
| Limite RAM | 512 MB |
| Rete | Porta 8080 solo su localhost |
| Autenticazione | API key generata con `openssl rand -hex 32` |

**Redis**

| Parametro | Valore |
|-----------|--------|
| Immagine | `redis:7-alpine` |
| Memoria | 128 MB, eviction `allkeys-lru` |
| Persistenza | AOF, per non perdere la sessione al riavvio |
| Healthcheck | `redis-cli ping` ogni 10 s |

**Worker**

```
ogni 30 min ──► appuntamenti tra 24 e 28 ore ──► compone il messaggio ──► invia ──► stato: inviato | errore
```

- Prende solo righe in stato `in_attesa` o `errore`, quindi i fallimenti vengono ritentati al giro successivo
- 2 secondi di pausa tra un invio e l'altro, per non far segnalare il numero
- PM2 con riavvio oltre 64 MB di memoria

---

## Sicurezza

| Rischio | Misura |
|---------|--------|
| Gateway raggiungibile da internet | UFW blocca la 8080, il worker chiama `localhost` |
| Chiave Supabase con privilegi elevati | Presente solo nel worker sulla VPS |
| Abbinamento WhatsApp | QR accessibile solo da tunnel locale |

---

## Costi

| Voce | Costo mensile |
|------|---------------|
| VPS | 3-5 euro |
| Messaggi | 0 |
| Supabase | Free tier |
| Totale | circa 5 euro, contro 50-200 euro di un servizio equivalente |

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| Gateway | Evolution API su sessione WhatsApp Web | Costo zero per messaggio, adatto ai volumi di una singola attività |
| Modello di invio | Worker in pull | Nessuna porta in ingresso da esporre sulla VPS |
| Risorse | Limiti di memoria su container e processi | Tutto lo stack sta in 2 GB di RAM |

---

## Evoluzioni previste

Da un'analisi successiva al prototipo sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Un solo percorso di invio, il worker in pull, al posto del doppio canale worker e cron | Nessun rischio di invii duplicati, gateway mai esposto |
| Immagine Evolution API fissata a una versione precisa | Deploy riproducibili, aggiornamenti controllati |
| WhatsApp Cloud API ufficiale oltre una certa soglia di volumi | Servizio scalabile e conforme per più clienti |

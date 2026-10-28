# Gestionale ristorante: PWA mobile-first

| | |
|---|---|
| Tipo | Gestionale operativo per un ristorante di poke |
| Stack | Vite, React, Tailwind, Supabase (Postgres, Auth, Realtime, Storage, Edge Functions) |
| Hosting | Vercel (PWA, funzioni serverless, cron) e Raspberry Pi Zero 2W in sede |
| Periodo | Febbraio - luglio 2026, 157 commit |
| Stato | In produzione |
| Link | [poke.cucinadasandro.it](https://poke.cucinadasandro.it) |

---

## Problema

Ordini, tavoli, prenotazioni e magazzino erano gestiti su carta e WhatsApp. Il personale doveva lavorare da smartphone e tablet propri, senza installare app dagli store e senza PC in sala. L'infrastruttura doveva costare quasi zero.

## Soluzione

Una Progressive Web App installabile sulla home dei dispositivi. Copre l'intero ciclo: composizione della bowl da parte del cliente, sala, cucina, prenotazioni, magazzino, statistiche e un modulo separato per le feste di paese.

---

## Architettura

```
┌──────────────────────────────────────────────────────────┐
│                     VERCEL                                │
│  PWA (React)   Funzioni API (SMS, email, punti)   Cron    │
└───────┬──────────────────┬──────────────────────┬────────┘
        │                  │                      │
        ▼                  ▼                      ▼
┌──────────────────────────────────────────────────────────┐
│                     SUPABASE                              │
│  Postgres   Auth (ruoli)   Realtime   Storage   Edge Fn   │
└──────────────────────────┬───────────────────────────────┘
                           │ Realtime
              ┌────────────┴─────────────┐
              ▼                          ▼
   ┌─────────────────────┐   ┌─────────────────────────┐
   │ Raspberry Pi Zero 2W│   │ PC Windows (feste)      │
   │ print server Node.js│   │ festa-printer.exe       │
   └──────────┬──────────┘   └───────────┬─────────────┘
              ▼                          ▼
        Stampante termica ESC/POS (USB o rete)
```

| Integrazione | Uso |
|--------------|-----|
| SMTP | Conferme prenotazione, recap giornaliero, alert magazzino |
| SMS (Aruba) | Notifiche urgenti |
| TheFork | Webhook e client OAuth come Supabase Edge Function |

---

## Moduli

**Builder (cliente)**
Configuratore della bowl in più passaggi, con limiti sugli ingredienti e prezzo calcolato in tempo reale. Bowl condivisibili via link, bowl casuale pesata, inviti con codice referral.

**Sala**
- Stato dei tavoli e delle comande sincronizzato tra dispositivi con Supabase Realtime
- Ciclo della comanda: `pending → preparing → completed → served`
- Conto con sconti, coperti, note e chiusura
- Mappa della sala con drag and drop, alternabile alla vista elenco

**Amministrazione (12 sezioni)**
Ordini, consegnati, menu bowl, menu ristorante e cantina, prodotti, clienti, impostazioni, staff, statistiche, prenotazioni, magazzino, archivio. Include export CSV, KPI (incasso, coperti, scontrino medio, occupazione) e programma punti fedeltà.

**Festa**
Modulo per eventi ad alto volume: cassa rapida, display pubblico con sintesi vocale e QR, tracking dell'ordine via token, coupon stampati, gestione omaggi e allergeni.

---

## Numeri

| Voce | Valore |
|------|--------|
| Route dell'applicazione | 30 |
| Migration SQL | 39 |
| Tabelle (da migration) | circa 33 |
| Funzioni API | 6 |
| Test unitari (Vitest) | 81 in 19 file |
| Suite E2E (Playwright) | 3 |

---

## Sicurezza dei dati

- Row Level Security su Postgres con policy per ruolo: admin, sala, cucina, cliente
- Autenticazione Supabase con ruoli assegnati dal pannello staff
- Nessun dato personale nella cache del dispositivo (vedi sotto)
- Tabelle dedicate per log di sicurezza, errori e consensi cookie
- Security review periodica su database e print server

---

## PWA e cache

Configurata con `vite-plugin-pwa`, aggiornamento automatico.

| Risorsa | Strategia | Motivo |
|---------|-----------|--------|
| Navigazione | NetworkFirst, timeout 5 s, fallback `offline.html` | Il personale vede sempre dati aggiornati |
| Immagini e font | CacheFirst, 30 giorni | Riduce traffico su rete mobile |
| `/api/*` e Supabase | NetworkOnly | Nessun dato personale nella cache del dispositivo |

---

## Stampa

Il print server gira su un Raspberry Pi Zero 2W (circa 25 euro tra scheda, alimentatore e SD). Il servizio Node.js ascolta i nuovi ordini via Realtime e li stampa in formato ESC/POS sulla termica già presente.

Per le feste, dove si usa un PC Windows al banco, c'è un secondo print server (Express e `node-thermal-printer`) distribuito come eseguibile singolo, così non serve installare Node sul posto.

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| App nativa o PWA | PWA | Nessuno store, aggiornamenti immediati, gira su qualsiasi dispositivo |
| Pagamenti | Solo stato aperto/chiuso | L'incasso resta sulla cassa fiscale esistente |
| Stampa | Raspberry Pi in sede | Meno di 30 euro una tantum contro un abbonamento SaaS, latenza locale |
| Menu ristorante | Tabella `menu_items` separata | Evita di piegare lo schema del builder a un uso diverso |
| Automazioni | Cron Vercel | Recap giornaliero e controllo magazzino senza un server dedicato |

---

## Evoluzioni previste

Da un'analisi successiva al rilascio sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Schedulare i promemoria delle prenotazioni, già implementati | Meno tavoli vuoti per mancata presentazione |
| Attivare l'integrazione TheFork in produzione | Prenotazioni esterne nello stesso flusso di quelle interne |
| Calcolare prezzi e sconti in funzioni Postgres | Il client invia solo la composizione dell'ordine, una sola fonte di verità per gli importi |
| Portare la copertura dei test verso la soglia dell'85% già configurata, con E2E sui flussi per ruolo | Rilasci più rapidi con meno verifiche manuali |
| Coda degli ordini offline per la sala | La sala continua a lavorare anche con rete instabile |

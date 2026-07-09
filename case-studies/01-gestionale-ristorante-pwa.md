# 🍣 Gestionale Ristorante PWA — Mobile-First

> **Case Study: System Design & Architettura**
> Stack: Vite + React + Supabase + Tailwind · Deploy: Vercel (PWA) + Raspberry Pi Zero 2W
>
> 🔗 **Live:** [poke.cucinadasandro.it](https://poke.cucinadasandro.it)

---

## Il Problema

Un ristorante pokè in crescita gestiva tutto con carta, WhatsApp e memoria umana: ordini, tavoli, prenotazioni, inventario. Serviva una soluzione digitale completa che lo staff potesse usare **dai propri smartphone e tablet**, senza installare app da store, senza PC fissi in sala e con costi infrastrutturali vicini allo zero.

## La Soluzione

Una **Progressive Web App (PWA)** installabile direttamente sulla home screen dei dispositivi dello staff. L'applicazione copre l'intero ciclo operativo del ristorante: dalla composizione dell'ordine lato cliente, alla gestione di sala, cucina, prenotazioni, inventario e statistiche avanzate.

Ogni interfaccia è progettata **mobile-first**: i camerieri gestiscono i tavoli dal tablet in sala, la cucina riceve le comande in tempo reale, il titolare monitora le statistiche dal telefono.

---

## Architettura

```
┌─────────────────────────────────────────────────────────────┐
│                    VERCEL (Edge Network)                     │
│  ┌──────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │ PWA/SPA  │  │  Serverless  │  │    Cron Functions      │ │
│  │ (React)  │  │  Functions   │  │  (Reminder/Notifiche)  │ │
│  └────┬─────┘  └──────┬───────┘  └───────────┬────────────┘ │
└───────┼───────────────┼───────────────────────┼─────────────┘
        │               │                       │
        ▼               ▼                       ▼
┌─────────────────────────────────────────────────────────────┐
│                 SUPABASE (Cloud BaaS)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────────┐  │
│  │ Postgres │  │   Auth   │  │ Realtime │  │  Storage   │  │
│  │  + RLS   │  │ (Ruoli)  │  │  (Live)  │  │  (Media)   │  │
│  └──────────┘  └──────────┘  └─────┬────┘  └────────────┘  │
└────────────────────────────────────┼────────────────────────┘
                                     │
                                     ▼
                        ┌────────────────────────┐
                        │  RASPBERRY PI ZERO 2W  │
                        │  ┌──────────────────┐  │
                        │  │  Print Server    │  │
                        │  │  (Node.js)       │  │
                        │  └────────┬─────────┘  │
                        └───────────┼────────────┘
                                    ▼
                        ┌────────────────────────┐
                        │  STAMPANTE TERMICA      │
                        │  ESC/POS (USB/Network)  │
                        └────────────────────────┘
```

### Comunicazioni Esterne
| Servizio | Scopo | Provider |
|----------|-------|----------|
| Email | Conferme prenotazione, reminder, alert inventario | SMTP proprietario |
| SMS | Notifiche urgenti | API Aruba |

---

## I 4 Macro-Moduli

Il sistema è organizzato con una **feature-based architecture** in 4 moduli indipendenti:

### 1. Builder — Configuratore Pokè
Configuratore multi-step con validazione ingredienti, massimali e calcolo prezzo dinamico. Interfaccia touch-friendly pensata per il cliente che ordina da asporto dal proprio smartphone.

### 2. Waiter (Sala) — Gestione Tavoli Mobile
Il cuore operativo per i camerieri che usano il **tablet in sala**:
- Gestione tavoli in tempo reale con aggiornamenti via Supabase Realtime
- Comande con pipeline di stati: `pending → preparing → completed → served`
- Gestione conto: sconti (% o fisso), coperti, note, chiusura
- **Mappa sala interattiva** con drag & drop nativo per riposizionare i tavoli
- Toggle tra vista lista e vista mappa
- Quick order modal per ordini rapidi

### 3. Admin — Dashboard Completa (22 Pannelli)
- CRUD menù ristorante, cantina vini, menù fissi, categorie con ordinamento e icone
- Prenotazioni web + manuali (telefono) con reminder automatici
- Staff con ruoli e permessi
- Inventario con alert sotto-scorta e email automatica a fine giornata
- Statistiche: incassi giornalieri/settimanali/mensili, piatti top, fasce orarie
- KPI: revenue, covers, avg ticket, occupancy rate
- Storico comande con filtri e **export CSV**
- Programma fedeltà a punti

### 4. Festa — Modulo Eventi
Modulo dedicato per sagre e feste:
- Dashboard ordini rapidi con flusso semplificato
- Display ordini per il pubblico (monitor esterno)
- Stampa ticket con formattazione termica
- Configurazione evento (prodotti, prezzi, sconti, omaggi)

---

## Database Design

**~15 tabelle** con schema relazionale su PostgreSQL (Supabase):

```
tables ──────────┐
table_sessions ──┤
orders ──────────┼── Core Sala & Comande
order_items ─────┤
                 │
menu_items ──────┤
menu_categories ─┼── Menù & Cantina
wine_list ───────┤
fixed_menus ─────┘

reservations ────┐
reservation_     ├── Prenotazioni
  settings ──────┘

inventory ───────── Magazzino

customers ───────── Anagrafica & Fedeltà
```

### Row Level Security (RLS)
Ogni ruolo (Admin, Waiter, Kitchen, Customer) ha policy SQL granulari che isolano i dati. Un cameriere vede solo i tavoli e le comande della propria sessione; un cliente vede solo i propri ordini e il proprio profilo fedeltà.

### Realtime Subscriptions
Le comande e gli stati dei tavoli si sincronizzano in tempo reale tra tutti i dispositivi dello staff tramite i canali Realtime di Supabase. Quando la cucina segna un piatto come "completato", il tablet del cameriere in sala si aggiorna istantaneamente.

---

## Print Server Hardware

Il problema della stampa comande è risolto con un approccio **ibrido cloud + hardware locale** a costo quasi zero:

| Componente | Costo |
|-----------|-------|
| Raspberry Pi Zero 2W | ~€15 |
| Alimentatore + SD Card | ~€10 |
| Stampante termica ESC/POS | Già in dotazione |

Il servizio Node.js sul RPi si connette a Supabase Realtime, ascolta i nuovi ordini e li formatta secondo il protocollo ESC/POS per la stampante termica. Per gli eventi su postazioni Windows è disponibile anche un eseguibile compilato.

---

## Testing & Quality Assurance

| Tipo | Tool | Copertura |
|------|------|-----------|
| **Unit** | Vitest + React Testing Library | Utilities logiche, hooks critici, render dinamico menù |
| **E2E** | Playwright | Login per ruolo, ordine da tablet, evasione cucina, checkout asporto |
| **Target** | Coverage | 85% sul codice critico |

---

## Fasi di Sviluppo

| # | Fase | Stato |
|---|------|-------|
| 1 | Menù Digitale Completo + Cantina Vini | ✅ |
| 2 | Gestione Tavoli, Comande & Conto | ✅ |
| 3 | Prenotazione Tavoli + Reminder | ✅ |
| 4 | Mappa Sala Interattiva | ✅ |
| 5 | Magazzino & Inventario | ✅ |
| 6 | Statistiche Avanzate & Storico | ✅ |
| 7 | Integrazione TheFork | 🚧 In attesa credenziali API |
| 8 | Testing & QA | ✅ Infrastruttura + test implementati |

---

## Decisioni Architetturali Chiave

| Decisione | Scelta | Motivazione |
|-----------|--------|-------------|
| App nativa vs PWA | **PWA** | Aggiornamenti immediati, no app store, installabile su qualsiasi dispositivo |
| Database separato per menù | **Tabella `menu_items` dedicata** | Non estendere la tabella `signature_items` del builder pokè |
| Pagamenti | **Solo tracking stato** | La cassa fisica gestisce il pagamento, il gestionale traccia solo aperto/chiuso |
| Print server | **RPi Zero 2W locale** | Costo <€30 vs SaaS a pagamento, latenza zero |
| Reminder prenotazioni | **Funzionanti per tutte le fonti** | Web + manuali (telefono) + futuro TheFork |

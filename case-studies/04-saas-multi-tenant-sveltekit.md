# SaaS multi-tenant per sagre e feste

| | |
|---|---|
| Tipo | Preordini e cassa per eventi gastronomici, più eventi sulla stessa piattaforma |
| Stack | SvelteKit 2, Svelte 5, Supabase (Postgres, Realtime), Tailwind, Vercel Edge |
| Periodo | Maggio 2026 |
| Stato | Prototipo |

---

## Problema

Le sagre ricevono centinaia di ordini in poche ore. Ogni evento ha menu, prezzi e casse propri, ma rifare il software per ogni sagra non ha senso. Serviva una piattaforma unica in cui ogni evento avesse uno spazio separato.

## Soluzione

Multi-tenancy basata sullo slug nell'URL (`/nome-sagra`). Ogni richiesta risolve lo slug nel tenant e carica solo i suoi dati. Il pannello di gestione mostra gli ordini in tempo reale e le statistiche calcolate dal database.

---

## Architettura

```
/sagra-del-pesce              /festa-della-birra
        │                             │
        ▼                             ▼
┌────────────────────────────────────────────────┐
│ SvelteKit (runtime edge)                        │
│                                                 │
│  /[sagra_slug]   slug → tenant, prodotti        │
│  /admin          ordini live, statistiche       │
└───────────────────────┬────────────────────────┘
                        ▼
┌────────────────────────────────────────────────┐
│ Supabase                                        │
│  sagre, prodotti, ordini, elementi_ordine       │
│  vista v_statistiche_sagra                      │
│  canale Realtime su ordini                      │
└────────────────────────────────────────────────┘
```

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| Isolamento tenant | Slug nell'URL e filtro per tenant su ogni query | Un solo deploy per tutti gli eventi |
| Statistiche | Vista SQL (`v_statistiche_sagra`) | Incasso, carico cucina, attesa media e più venduti calcolati nel database |
| Cucina | Canale Realtime sugli ordini | Nessun polling durante i picchi |
| Runtime | Edge per route | Latenza bassa su rete mobile congestionata |

---

## Stato

Prototipo sviluppato nelle prime quattro fasi: infrastruttura, vetrina per tenant, ordini, pannello con statistiche in tempo reale. La gestione delle feste è oggi coperta dal modulo Festa del [gestionale ristorante](01-gestionale-ristorante-pwa.md), che è in produzione.

---

## Evoluzioni previste

Da un'analisi successiva al prototipo sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Policy RLS e schema versionati come migration | Isolamento tra tenant garantito dal database e replicabile |
| Autenticazione e ruoli sul pannello di gestione | Pronto per più organizzatori sulla stessa piattaforma |
| CI con test sul flusso d'ordine | Affidabilità nei picchi di carico degli eventi |

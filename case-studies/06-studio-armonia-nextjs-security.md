# Studio arMonia: migrazione da WordPress con sicurezza in CI

| | |
|---|---|
| Tipo | Sito e piattaforma corsi per uno studio di neuropsicologia clinica |
| Stack | Next.js 15 (App Router), React 19, TypeScript, Tailwind 4, Radix UI, Supabase, Zod, nodemailer |
| Hosting | Vercel, Supabase |
| Periodo | Settembre - ottobre 2026, 43 commit |
| Stato | Sito in produzione. Area corsi e pagamenti pronti, disattivati su richiesta del cliente |
| Link | [studioarmonia.org](https://www.studioarmonia.org) |

---

## Problema

Lo studio aveva un sito WordPress da sostituire. Oltre al sito servivano un blog gestito internamente e, in prospettiva, una piattaforma di corsi con iscrizioni. Il settore è sanitario: i dati raccolti dal form contatti sono sensibili e il sito deve reggere un'analisi di sicurezza esterna.

## Soluzione

Un'applicazione Next.js divisa in gruppi di route: pagine pubbliche, autenticazione, area riservata e amministrazione. La piattaforma corsi è già sviluppata ma resta spenta dietro un flag. Il lavoro principale è la pipeline: ogni modifica passa da controlli di qualità e di sicurezza prima di arrivare in produzione.

---

## Architettura

```
┌──────────────────────────────────────────────────────────┐
│ Next.js su Vercel                                         │
│                                                           │
│  (public)     servizi, blog, team, contatti, prenotazione │
│  (auth)       login, registrazione, recupero password     │
│  (dashboard)  area iscritti            ┐                  │
│  admin        blog, corsi              ├ 404 se           │
│                                        ┘ PLATFORM_ENABLED │
│  middleware   sessione, fail-closed sulle route protette  │
└───────────────────────┬──────────────────────────────────┘
                        ▼
┌──────────────────────────────────────────────────────────┐
│ Supabase                                                  │
│  13 tabelle, RLS su tutte, 35 policy                      │
│  check_rate_limit (RPC atomica, solo service_role)        │
│  pg_cron: cancellazione dati personali scaduti            │
└──────────────────────────────────────────────────────────┘
```

---

## Form contatti

```
richiesta ──► honeypot ──► validazione Zod ──► rate limit (5 ogni 10 min per IP hashato)
         ──► salvataggio ──► audit log ──► email allo studio
```

Se l'invio email fallisce, la richiesta resta salvata con stato `notify_failed`: nessun contatto si perde.

---

## Pipeline

**CI su PR e su `main`**

| Job | Cosa verifica |
|-----|---------------|
| quality | Lint, typecheck, test unitari |
| migrations-guard | Script custom: blocca migration con tabelle senza RLS, policy di scrittura aperte ad `anon` o `public`, modifiche alla baseline |
| secrets | gitleaks sull'intera history |
| sast | Semgrep, regole Next.js, TypeScript, React e secrets |
| deps | `npm audit` (produzione, livello alto) e commento sulla PR con le licenze cambiate |
| build ed e2e | Playwright sull'artefatto buildato |
| actionlint | Correttezza dei workflow |

Tutte le action sono pinnate per SHA.

**CD su `main`**

```
build prebuilt ──► deploy Vercel produzione ──► attesa /api/health
               ──► fallisce se /api/contatti o /blog rispondono 500
               ──► controllo header di sicurezza (non bloccante)
```

Dependabot settimanale con aggiornamenti minor e patch raggruppati.

---

## Sicurezza applicativa

| Area | Implementazione |
|------|-----------------|
| Header | CSP statica (le pagine restano prerenderizzate), HSTS preload, X-Frame-Options DENY, Permissions-Policy |
| Accesso | RBAC admin e staff con funzioni SQL dedicate, RLS su tutte le tabelle |
| Input | Zod, honeypot, sanitizzazione HTML con allowlist per il blog, redirect solo verso destinazioni sicure |
| Errori | Risposte generiche con un `errorId` per la correlazione nei log |
| Health check | `/api/health` visibile solo agli admin, 404 per tutti gli altri |
| Analytics | Caricati solo dopo il consenso |
| Migrazione | Redirect 301 dai vecchi URL WordPress |

Ad ottobre 2026 ho chiuso un piano di remediation in quattro fasi a seguito di un'analisi di sicurezza. Report e brief sono in `docs/security/`.

---

## Test

| Tipo | Quantità |
|------|----------|
| Unitari | 15 (rate limiter, sanitizzazione, redirect, gestione errori) |
| Script di CI | 8 (guard delle migration, diff delle dipendenze) |
| Policy RLS | 12 controlli, eseguiti su Supabase locale o staging |
| E2E Playwright | 14 (prenotazione, consenso, form contatti, pagine) |

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| Piattaforma corsi | Flag che restituisce 404 | Codice già in produzione ma irraggiungibile finché il cliente non decide |
| Rate limiting | Tabella Postgres e RPC atomica | Funziona su serverless senza un Redis in più |
| IP | Hash con salt | Serve a limitare gli abusi, non a identificare le persone |
| Retention | `pg_cron` nel database | La pulizia non dipende dal deploy dell'applicazione |

---

## Evoluzioni previste

Da un'analisi successiva al rilascio sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Attivazione della piattaforma corsi | Nuovo canale di ricavo per lo studio, già sviluppato |
| Pagamenti Stripe con webhook idempotenti (tabella `processed_webhook_events` già predisposta) | Nessun doppio addebito o doppia iscrizione |
| Progetto Supabase di staging collegato alla CI | Test delle policy RLS a ogni PR, non solo in locale |
| CSP basata su nonce | Policy più restrittiva mantenendo le pagine dinamiche dove servono |

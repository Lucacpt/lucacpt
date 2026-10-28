# Portale studio dentistico: raccolta contatti e blog

| | |
|---|---|
| Tipo | Sito istituzionale con raccolta contatti per uno studio odontoiatrico |
| Stack | Next.js 15 (App Router), React 19, TypeScript, Tailwind 4, Supabase, Tiptap, nodemailer |
| Hosting | Vercel, Supabase |
| Stato | In produzione |
| Link | [studioodontoiatricofrancescobruno.it](https://www.studioodontoiatricofrancescobruno.it/) |

---

## Problema

Lo studio voleva un sito curato che portasse richieste di contatto, non prenotazioni online con calendario. Il blog doveva essere gestito dalla segreteria senza passare dallo sviluppatore. Trattandosi di dati sanitari, i contatti non dovevano restare in archivio più del necessario.

## Soluzione

Un sito Next.js con pagine dei servizi generate staticamente, un form contatti che salva il lead e avvisa sia lo studio sia il paziente, e un pannello di amministrazione minimo per blog e richieste.

---

## Architettura

```
┌───────────────────────────────────────────────┐
│                    VERCEL                      │
│  Pagine pubbliche   Route API   Cron giornaliero│
└───────┬─────────────────┬──────────────┬───────┘
        │                 │              │
        ▼                 ▼              ▼
┌───────────────┐   ┌───────────┐   ┌─────────────────┐
│   SUPABASE    │   │   SMTP    │   │ Pulizia lead    │
│ leads         │   │ studio +  │   │ oltre 30 giorni │
│ blog_posts    │   │ paziente  │   └─────────────────┘
│ Storage       │   └───────────┘
└───────────────┘
```

---

## Flusso contatti

```
Form sul sito ──► /api/booking ──► insert in "leads" ──► email allo studio ──► email di cortesia al paziente
```

Dati raccolti: nome, email, telefono, trattamento di interesse, messaggio. Nessuna scelta di data: lo studio richiama e fissa l'appuntamento.

Dal pannello la segreteria vede le richieste e le segna come gestite.

---

## Blog e contenuti

| Contenuto | Gestione |
|-----------|----------|
| Articoli | Editor Tiptap nel pannello `/admin/blog`, salvati su Supabase, immagini su Supabase Storage |
| Servizi (18) | Dati statici nel codice, pagine generate con `generateStaticParams` |
| Team (15 schede) | Dati statici nel codice |
| Statistiche | Report Looker Studio incorporato nel pannello |

---

## Privacy e operatività

| Tema | Implementazione |
|------|-----------------|
| Retention | Cron Vercel giornaliero che cancella i lead più vecchi di 30 giorni |
| Consenso | Banner cookie; Google Tag Manager si carica solo dopo il consenso |
| Free tier Supabase | Cron ogni 3 giorni che interroga il database per evitare la sospensione per inattività |
| Misure | Vercel Analytics e Speed Insights |

---

## Pagine

| Route | Rendering |
|-------|-----------|
| `/` | Statico |
| `/servizi/[slug]` | Statico, 18 pagine generate in build |
| `/blog`, `/blog/[slug]` | Dinamico, contenuti sempre aggiornati dal pannello |
| `/contatti` | Form contatti |
| `/admin/*` | Riservato, 5 pagine |

In totale 14 pagine, 5 route API e 2 cron.

---

## Scelte principali

| Decisione | Scelta | Motivo |
|-----------|--------|--------|
| CMS | Pannello custom con Tiptap | Bastavano gli articoli: un CMS completo avrebbe aggiunto complessità senza uso reale |
| Servizi e team | Dati nel codice | Cambiano raramente e così le pagine restano statiche e veloci |
| Lead | Cancellazione dopo 30 giorni | Minimizzazione dei dati sanitari: il lead serve solo fino al primo contatto |
| Statistiche | Looker Studio incorporato | Zero codice da mantenere per i report |

---

## Evoluzioni previste

Da un'analisi successiva al rilascio sono emerse queste implementazioni per la prossima versione:

| Evoluzione | Beneficio |
|------------|-----------|
| Schema Supabase versionato in migration | Ambiente riproducibile e modifiche tracciate |
| Pipeline GitHub Actions con lint, build e test E2E del form contatti | Ogni modifica verificata prima del deploy |
| Statistiche dei contatti per trattamento nel pannello | Lo studio vede quali servizi generano più richieste |

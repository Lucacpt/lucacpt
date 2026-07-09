# 🦷 Portale Studio Dentistico — Lead Generation Premium

> **Case Study: System Design & Architettura**
> Stack: Next.js 15 (App Router) + React 19 + TypeScript + Supabase · Deploy: Vercel (EU)
>
> 🔗 **Live:** [studioodontoiatricofrancescobruno.it](https://www.studioodontoiatricofrancescobruno.it/)

---

## Il Problema

Una clinica odontoiatrica di fascia alta necessitava di un portale web che trasmettesse esclusività ed eccellenza medica, con focus sulla **Lead Generation** (cattura contatti) piuttosto che sulle prenotazioni online dirette. Serviva inoltre un CMS per la gestione autonoma dei contenuti (blog, casi clinici Prima/Dopo, trattamenti) e un'attenzione particolare alla compliance GDPR data la sensibilità del settore sanitario.

## La Soluzione

Un portale **Next.js con App Router** che combina performance estreme (Core Web Vitals eccellenti grazie ai Server Components), un CMS headless integrato (Payload CMS) e una pipeline di lead generation automatizzata con notifica email bidirezionale.

---

## Architettura

```
┌──────────────────────────────────────────────────────┐
│              VERCEL (Regione EU — GDPR)               │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ Next.js App  │  │   Server     │  │  Vercel    │ │
│  │  (SSR/SSG)   │  │  Actions     │  │   Blob     │ │
│  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
└─────────┼─────────────────┼────────────────┼────────┘
          │                 │                │
          │                 ▼                │
          │    ┌─────────────────────┐       │
          │    │   SMTP Serverplan   │       │
          │    │    (Email SMTP)     │       │
          │    └─────────────────────┘       │
          ▼                                  │
┌──────────────────────────────┐             │
│     SUPABASE (EU Region)     │             │
│  ┌──────────┐  ┌──────────┐ │             │
│  │ Postgres │  │ Storage  │◄┼─────────────┘
│  │ (Lead +  │  │ (File    │ │  Media marketing
│  │  CMS DB) │  │ interni) │ │  su Vercel Blob,
│  └──────────┘  └──────────┘ │  file interni
└──────────────────────────────┘  su Supabase Storage
```

---

## Strategia Media (Ibrida)

La gestione dei media è stata separata in due canali per ottimizzare latenza e sicurezza:

| Tipo Media | Storage | Motivazione |
|-----------|---------|-------------|
| Immagini marketing (galleria Prima/Dopo, copertine blog, UI) | **Vercel Blob** | Latenza zero via CDN, ottimizzato con `next/image` + Sharp |
| File interni, documenti, foto staff, backup | **Supabase Storage** | Protetto da RLS, dati in EU, non pubblico |

Payload CMS è configurato con **due Upload Collections** distinte, ciascuna con il proprio adattatore di storage.

---

## Lead Generation Pipeline

```
Cliente compila     Next.js Server     Salvataggio        Email notifica      Email cortesia
form contatti  ──►  Action server  ──► su Supabase   ──► alla segreteria ──► al paziente
                    side (no JS          (tabella           (SMTP              (template
                     client)              leads)          Serverplan)         formattato)
```

**Dati catturati:** Nome, Email, Telefono, Messaggio, Trattamento di interesse.

L'intera pipeline è server-side (Server Actions): nessuna chiave API o credenziale è mai esposta al browser.

---

## CMS — Payload CMS Integrato

Payload CMS gestisce autonomamente i contenuti senza intervento dello sviluppatore:

| Collezione | Descrizione |
|-----------|-------------|
| `BlogPosts` | Articoli blog con editor TipTap (immagini, link, allineamento) |
| `CasiClinici` | Galleria Prima/Dopo con campi strutturati |
| `Trattamenti` | Catalogo servizi offerti |

### Dashboard Analytics Personalizzata
Un widget custom dentro la dashboard Payload interroga le **Google Analytics Data API (v1)** tramite Service Account per mostrare:
- Volume: visitatori unici mensili
- Acquisizione: sorgenti traffico (Organic, Social, Direct)
- Comportamento: pagine/trattamenti più visitati
- **Geolocalizzazione**: mappa visite per Città/Regione — fondamentale per evidenziare il "turismo dentale" (pazienti fuori zona)

---

## Compliance GDPR

| Requisito | Implementazione |
|-----------|----------------|
| Localizzazione dati | Vercel EU (Francoforte/Parigi) + Supabase EU |
| Dati sensibili | Mai su storage pubblico (Vercel Blob), solo su Supabase Storage con RLS |
| Credenziali | Solo server-side, mai esposte al client |
| Informativa | Pagine Cookie Policy e Privacy Policy dedicate |
| Immagini pazienti | Ottimizzate con Sharp server-side, servite via `next/image` con `remotePatterns` configurati |

---

## Pagine & Struttura

| Route | Tipo | Descrizione |
|-------|------|-------------|
| `/` | SSG/SSR | Homepage premium |
| `/servizi` | SSG | Catalogo trattamenti |
| `/chi-siamo` | SSG | Presentazione team e studio |
| `/contatti` | SSR | Form lead generation con Server Action |
| `/blog` | SSG/ISR | Articoli dal CMS |
| `/admin` | Protetta | Dashboard Payload CMS |
| `/cookie-policy` | SSG | Informativa cookie |
| `/privacy-policy` | SSG | Informativa privacy |

SEO automatizzata: `sitemap.ts` dinamica, `robots.ts`, meta tags per pagina.

---

## Decisioni Architetturali Chiave

| Decisione | Scelta | Motivazione |
|-----------|--------|-------------|
| Framework | **Next.js App Router** | Server Components per performance, Server Actions per sicurezza |
| CMS | **Payload CMS integrato** | Nessun servizio esterno, tutto nel monorepo Next.js |
| Storage duale | **Vercel Blob + Supabase Storage** | Performance CDN per pubblico, sicurezza RLS per interno |
| Email | **SMTP dedicato Serverplan** | Deliverability garantita, dominio proprietario |
| Hosting EU | **Vercel Francoforte** | GDPR compliance nativa |

# 🎪 SaaS Multi-Tenant — SvelteKit per Sagre e Feste

> **Case Study: Architettura Multi-Tenant**
> Stack: SvelteKit 2 + Svelte 5 + Supabase + Tailwind · Deploy: Vercel

---

## Il Problema

Sagre, feste paesane e eventi gastronomici locali hanno bisogno di gestire ordini, menù e flussi di cassa durante eventi ad alta intensità (centinaia di ordini in poche ore). Serviva una piattaforma **multi-tenant** dove ogni evento avesse il proprio spazio isolato, con un pannello admin centralizzato per la gestione.

## La Soluzione

Un'architettura **slug-based multi-tenancy** con SvelteKit, dove ogni sagra è accessibile tramite il proprio URL (`/nome-sagra`) e i dati sono completamente isolati a livello di database tramite RLS.

> ⚠️ Questo progetto non è stato portato a termine. Viene documentato come case study per l'architettura multi-tenant e la gestione di volumi elevati di sviluppo in tempi compressi.

---

## Architettura Multi-Tenant

```
URL: /sagra-del-pesce           URL: /festa-della-birra
         │                               │
         ▼                               ▼
┌─────────────────────────────────────────────────┐
│            SvelteKit (App Router)                 │
│                                                   │
│   /(tenant)/[sagra_slug]/+page.server.ts          │
│         │                                         │
│         ├── Risolve slug → tenant_id              │
│         ├── Carica dati filtrati per tenant        │
│         └── Renderizza server-side                │
│                                                   │
│   /admin/+page.server.ts                          │
│         └── Gestione centralizzata tutte le sagre │
└───────────────────────┬─────────────────────────┘
                        │
                        ▼
┌───────────────────────────────────────────────────┐
│              SUPABASE (PostgreSQL + RLS)            │
│                                                     │
│   ┌─────────────────────────────────────────────┐  │
│   │  RLS Policy: tenant_id = current_tenant()    │  │
│   │  Ogni query è automaticamente filtrata       │  │
│   └─────────────────────────────────────────────┘  │
│                                                     │
│   Auth SSR (@supabase/ssr + hooks.server.ts)       │
└───────────────────────────────────────────────────┘
```

### Pattern di Routing SvelteKit

| Route | Scopo |
|-------|-------|
| `/(tenant)/[sagra_slug]` | Pagina pubblica dell'evento, risolta dinamicamente |
| `/admin` | Pannello centralizzato per tutte le sagre |
| `/api/*` | Endpoint interni per operazioni CRUD protette |

### Isolamento Dati
- Ogni tenant (sagra) è identificato da uno **slug univoco**
- Le query al database sono **sempre filtrate** per tenant ID
- **RLS Supabase** garantisce l'isolamento a livello di riga — anche in caso di bug applicativo, un tenant non può mai accedere ai dati di un altro
- Auth gestito server-side con `@supabase/ssr` e `hooks.server.ts`

---

## Competenze Acquisite

- Pattern **multi-tenant** con SvelteKit (route groups + dynamic params)
- Gestione di **volumi di sviluppo elevati** in tempi molto compressi
- Architettura SaaS con isolamento dati per tenant via RLS
- SvelteKit SSR con data loading server-side (`+page.server.ts`)
- Auth SSR con Supabase (cookies httpOnly, hooks server)

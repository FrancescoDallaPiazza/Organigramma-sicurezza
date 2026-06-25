# Organigramma Sicurezza

Repository di lavoro per la gestione della formazione e dell'organigramma della
sicurezza sul lavoro (D.Lgs. 81/08 + Accordo Stato-Regioni 17/04/2025) di
Overall Group S.r.l.

## Contenuto

### `checkupformazione81/` — skill
Skill per generare il **CHECKUP FORMAZIONE 81** (analisi sintetica dello stato di
adempimento agli obblighi formativi di un cliente), con i riferimenti normativi
(accordi storici e ASR 59/2025, DM antincendio/primo soccorso, ecc.) e gli asset
di branding.

### `organigramma-app/` — applicazione
App web **standalone single-file** (offline, dati in `localStorage`) per costruire
e gestire l'organigramma della sicurezza aziendale: figure e ruoli, formazione,
scadenze e fascicolo documentale. Calcola il **livello di rischio** dal codice
ATECO (Allegato IV ASR 2025) e permette di **aggiornare il catalogo corsi** da un
Excel. Dettagli in `organigramma-app/LEGGIMI-app.md` e specifica completa in
`organigramma-app/docs/progetto-organigramma-sicurezza-81.md`.

Uso rapido: apri `organigramma-app/index.html` in un browser moderno
(Chrome / Edge / Firefox).

## Proprieta
(c) Overall Group S.r.l. - strumento a uso interno. Tutti i diritti riservati.

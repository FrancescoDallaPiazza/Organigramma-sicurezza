# App "Organigramma Sicurezza 81" — v2.8

Contenuto di questa sottocartella:
- `index.html` ............. l'applicazione (single-file, offline, localStorage)
- `docs/progetto-organigramma-sicurezza-81.md` ... specifica e logica del progetto
- `data/ASR26-TabellaCorsi.xlsx` .......... catalogo corsi ASR uniformato (default app)
- `data/ASR26-TabellaCorsi-originale.xlsx` ... sorgente originale (tracciabilita)

Apri `index.html` nel browser per usare l'app. Dopo un aggiornamento del file,
fai un ricarica forzato della pagina (i dati restano in localStorage del browser).

## Novita v2.8
- Font del bottone "Scegli file" uniformato al resto dell'app.
- Esonero consolidato a livello di figura: un solo checkbox; se attivo sparisce il
  box della formazione iniziale e resta solo l'esonero con la sua evidenza dedicata,
  piu la gestione dell'aggiornamento.

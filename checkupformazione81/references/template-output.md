# Template output — CHECKUP FORMAZIONE 81 (Executive Summary)

Riferimento per la skill `checkupformazione81`. Definisce la struttura standard
del documento DOCX di output (Executive Summary) che la skill deve produrre per
ciascun cliente di Overall Group Srl.

**Lunghezza target: 2-3 pagine.** Il documento è pensato per essere consegnato
direttamente al cliente finale, quindi deve essere sintetico, visivo,
orientato all'azione.

---

## Specifiche generali del documento

### Formato

- File `.docx`
- Formato pagina: **A4**, margini 2 cm top/bottom, 2,5 cm destra/sinistra
- Font principale: **Gill Sans MT** (corpo 11pt, titolo sezione 13pt, titolo
  documento 20pt)
- Font di fallback (se Gill Sans non disponibile): **Calibri**
- Interlinea: 1.2
- Allineamento prevalente: giustificato (prose) o sinistra (titoli)

### Branding Overall Group

**Logo**: il file è in `assets/logo_overall.jpg`. Va inserito:

1. **Nell'header di ogni pagina** — piccolo (altezza ≈ 1 cm / 40 px), allineato
   a sinistra. A destra, sulla stessa riga: "Executive Summary — [Ragione
   Sociale Cliente]" in blu scuro. Sotto, una riga orizzontale blu scuro.
2. **Nel frontespizio** (prima pagina, in alto) — il logo può essere replicato
   in dimensione maggiore (altezza ≈ 1,5-2 cm) sopra il titolo principale,
   oppure solo nell'header se si vuole mantenere il design più pulito.

**Colori principali:**
- Blu scuro Overall: `#003366` — titoli, intestazioni tabelle, bordi
  principali, numeri di evidenza
- Blu chiaro: `#D9E6F2` — sfondo di box informativi secondari
- Rosso criticità: `#C00000` — per box di alert e semafori critici
- Verde conforme: `#548235` — per stato positivo
- Giallo-ambra: `#BF8F00` — per criticità minori
- Grigio: `#555555` per testo di footer e note secondarie

**Footer ogni pagina:**
- Testo: "Overall Group Srl — Via Alessandro Volta 36/1, Dossobuono (VR).
  Educazione Sicurezza®" allineato a sinistra, in grigio, corsivo, 8pt
- Numero pagina "Pag. X di Y" allineato a destra, 8pt
- Riga orizzontale blu scuro sopra il footer

### Marchi visivi / semafori

Per le valutazioni rapide di conformità usare sempre gli emoji:
- 🟢 = conforme
- 🟡 = criticità formale o minore
- 🔴 = criticità sostanziale o scadenza superata

---

## Struttura standard dell'Executive Summary

L'ordine delle sezioni è vincolante. Le sezioni 1-6 stanno tipicamente in 2-3
pagine totali, in base al numero di lavoratori.

### 1. Intestazione di documento (parte alta pagina 1)

- Piccola etichetta "CHECKUP FORMAZIONE 81" (grigia, maiuscolo)
- Titolo grande: **"Sintesi per il cliente"** (blu scuro, 20pt)
- Bordo inferiore blu per separare l'intestazione dal resto
- Sotto il titolo: riga di identificazione cliente

  > **RAGIONE SOCIALE** Forma giuridica (es. "S.a.s. di Cognome Nome & C.")
  > Via, CAP Città (PR) — P.IVA 12345678901
  > ATECO XX.XX.XX - descrizione — N addetti — Classe rischio [BASSO/MEDIO/ALTO]
  > Data del checkup: GG/MM/AAAA

### 2. Box "Stato complessivo" (in evidenza, pagina 1)

Un box rettangolare con bordo colorato secondo lo stato:
- 🟢 + "Stato complessivo: CONFORME" (bordo/testo verde, sfondo `#EAF5E6`) —
  nessuna criticità sostanziale
- 🟡 + "Stato complessivo: ATTENZIONE" (bordo/testo giallo, sfondo `#FFF4D9`) —
  criticità minori o scadenze nei prossimi 12 mesi
- 🔴 + "Stato complessivo: CRITICO" (bordo/testo rosso, sfondo `#FCE4E4`) —
  criticità sostanziali o scadenze superate

Dentro il box:
- Emoji grande del semaforo + etichetta
- Paragrafo di 3-4 righe che spiega in modo comprensibile al cliente
  perché lo stato è quello indicato e quali sono le direttrici d'intervento
  principali. Tono neutro-professionale, non allarmistico.

### 3. "Il quadro in un colpo d'occhio" (pagina 1)

4 box numerici affiancati (una riga di una tabella invisibile a 4 colonne),
ciascuno con:
- Numero grande in alto (colorato in base al tipo)
- Etichetta breve sotto (2 righe max)

Box tipici (in ordine):
1. Numero totale di attestati analizzati (blu) / oppure numero totale di
   lavoratori nel piano (se Ramo A senza attestati)
2. Numero conformi 🟢 (verde)
3. Numero criticità minori 🟡 (giallo)
4. Numero criticità sostanziali 🔴 (rosso)

Per il **Ramo A (cliente senza attestati)**, i 4 box possono invece mostrare:
addetti totali, figure SSL già coperte, figure da designare, corsi da
programmare.

### 4. "Le 5 azioni più urgenti" (pagina 1-2)

Tabella con 4 colonne:

| # | Azione | Perché | Durata |
|---|---|---|---|
| 1 | [Azione principale] | [Motivo sintetico: scadenza/urgenza] | Nh |
| 2 | ... | ... | ... |
| 3 | ... | ... | ... |
| 4 | ... | ... | ... |
| 5 | ... | ... | ... |

- Prima colonna: numero in rosso su sfondo rosso chiaro (`#FCE4E4`), grande e
  bold, come a "contrassegnare" l'urgenza
- Seconda colonna: titolo azione in bold, breve (max 8-10 parole)
- Terza colonna: motivo sintetico (max 15-20 parole)
- Quarta colonna: durata del corso (es. "4h", "12h", "4h + 4h", "—" se non
  applicabile)

Scegliere le 5 azioni in ordine di priorità temporale (scadenze superate >
figure obbligatorie mancanti > scadenze imminenti). Se ci sono meno di 5
azioni urgenti, scendere a 3-4 righe. Se l'azienda è pienamente in regola
(Ramo A pianificazione ex novo), la tabella può essere sostituita da un
elenco dei principali corsi da programmare.

### 5. "Dove sta l'azienda, lavoratore per lavoratore" (pagina 2)

Tabella con 4 colonne:

| Lavoratore | Ruolo SSL | Stato | Cosa manca / cosa fare |
|---|---|---|---|
| COGNOME Nome | Datore di Lavoro, RSPP, ... | 🔴 | Frase sintetica (20-30 parole) |
| COGNOME Nome | Lavoratore, RLS | 🟡 | ... |
| COGNOME Nome | Lavoratore | 🟢 | Formazione completa e aggiornata. |

- Intestazione colonne con sfondo blu scuro `#003366` e testo bianco
- Colonna "Stato" centrata, con solo l'emoji del semaforo (nessun testo)
- Ultima colonna: frase discorsiva breve, professionale, che il cliente capisce

Ordine suggerito: prima il Datore di Lavoro (DL), poi altre figure di
vertice (RSPP, Preposto, RLS), poi gli altri lavoratori. Ordine alfabetico
all'interno di ciascun gruppo.

### 6. "Il piano in numeri" (pagina 2-3)

3 box numerici (riga di tabella invisibile a 3 colonne):

1. Interventi ALTA priorità (rosso) — "entro 60 gg"
2. Interventi MEDIA priorità (giallo) — "entro 3-6 mesi"
3. Interventi BASSA priorità (verde) — "entro 12 mesi"

Ciascun box ha il numero grande in alto e sotto l'etichetta.

### 7. "Come Overall Group può aiutare" (pagina 3)

Titolo della sezione in blu.

Sotto, 4-5 punti elenco (indentati, con simbolo ▸ in blu scuro), che
descrivono i servizi offerti rispetto al caso specifico. Punti tipici:

- Progettare un calendario formativo coordinato (raggruppamento corsi per
  ridurre fermo aziendale)
- Erogare direttamente i corsi citati con formatori qualificati ex DI
  06/03/2013, con attestati già conformi all'ASR 17/04/2025
- Ricostruire il fascicolo formativo pregresso (richiesta attestati a
  precedenti datori di lavoro)
- Predisporre le lettere di designazione formali (Preposto, Addetti PS/AI,
  RLS) e i verbali di nomina
- Programmare la revisione annuale del CHECKUP per monitoraggio continuo

Ogni punto va personalizzato sul caso del cliente (non copia-incolla sterile).

### 8. Box "Prossimo passo" (chiusura, pagina 3)

Box rettangolare con bordo blu scuro e sfondo blu chiaro (`#D9E6F2`):

- Titolo "Prossimo passo" in blu scuro, bold
- Paragrafo di 2-3 righe che invita il cliente a un incontro (di persona o
  in videocall) per discutere le priorità e definire il calendario operativo
- Alla fine: **Riferimenti:** Via A. Volta 36/1, Dossobuono (VR) — [eventuale
  email/telefono/sito]

---

## Regole redazionali

- **Tono:** professionale e diretto, orientato al cliente finale. Evitare
  gergo tecnico non spiegato. Il documento deve essere comprensibile a un
  imprenditore non specialista della sicurezza.
- **Persona:** terza persona impersonale ("si rileva", "si segnala") o
  formule neutre ("l'azienda presenta", "mancano"). Non "noi" né "voi",
  salvo nel box finale dove si parla esplicitamente di Overall Group.
- **Acronimi:** la prima volta esplicitati (es. "Rappresentante dei
  Lavoratori per la Sicurezza (RLS)"), poi solo la sigla.
- **Citazioni normative:** minime e compatte — l'Executive Summary non è il
  luogo per citazioni estese. Usare solo i riferimenti strettamente
  necessari (es. "art. 19 D.Lgs. 81/08"). Se servono approfondimenti,
  rimandare all'analisi integrale (non parte di questo deliverable).
- **Date:** sempre in formato GG/MM/AAAA.
- **Durate corsi:** forma "Nh" (es. "4h", "12h").
- **Numeri cliente-friendly:** riferimenti ai lavoratori con COGNOME Nome
  (cognome in maiuscolo), senza CF o dati personali sensibili.
- **Anglicismi:** evitare (niente "executive summary" nel corpo, sostituire
  con "sintesi" dove possibile; il termine può comparire nell'header come
  identificazione del deliverable).

---

## Note tecniche per la skill

- La skill deve:
  1. Creare il file in `/mnt/user-data/outputs/`
  2. Nominarlo: `EXECUTIVE_SUMMARY_CHECKUP_81_[RAGIONE_SOCIALE]_[ANNO].docx`
     (senza spazi, ragione sociale in maiuscolo separata da underscore)
  3. Validare il file con
     `python /mnt/skills/public/docx/scripts/office/validate.py <file>`
  4. Usare `present_files` per renderlo scaricabile

- **Lunghezza target:** 2-3 pagine. Se supera le 3 pagine, comprimere (tabelle
  più dense, testo più conciso). Se non raggiunge le 2 pagine, aggiungere
  contenuto sostanziale (es. dettagliare meglio i "cosa manca / cosa fare"
  della tabella lavoratori) senza inserire riempitivo.

- **Inserimento logo con docx-js:**
  ```javascript
  const { ImageRun } = require('docx');
  const fs = require('fs');
  const logoBuffer = fs.readFileSync(
    '/mnt/skills/user/checkupformazione81/assets/logo_overall.jpg'
  );
  // Dentro un Paragraph:
  new ImageRun({
    data: logoBuffer,
    transformation: { width: 80, height: 32 } // pixel
  })
  ```
  Per l'header, mettere logo + testo in una tabella invisibile a 2 colonne
  (logo a sinistra, testo a destra). Per il frontespizio, si può mettere il
  logo dentro un Paragraph centrato con dimensione maggiore (es. 150 x 60 px).

- **Testo da mostrare in chat dopo la generazione:**
  "L'Executive Summary del CHECKUP FORMAZIONE 81 è pronto. Stato complessivo:
  [🟢/🟡/🔴]. Sono state rilevate [N] criticità sostanziali e pianificate [N]
  azioni prioritarie. Il documento è scaricabile qui sopra ed è pronto per
  essere consegnato al cliente."

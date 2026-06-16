# Mappatura ATECO → classe di rischio formativo (ASR 17/04/2025, Allegato)

Il livello di rischio determina le ore di **formazione specifica** dei lavoratori:
- **BASSO**: 4h specifica → totale 8h (4 generale + 4 specifica)
- **MEDIO**: 8h specifica → totale 12h (4 + 8)
- **ALTO**: 12h specifica → totale 16h (4 + 12)

## Regola di lettura del codice ATECO
- Le prime **2 cifre** individuano il settore principale.
- In caso di dubbio tra due classi, applicare il principio di precauzione (scegliere la più alta).

## Esempi rapidi
- 46.xx.x → BASSO (commercio all'ingrosso)
- 47.xx.x → BASSO (commercio al dettaglio)
- 41-43 → ALTO (costruzioni)
- 10.xx.x → ALTO (industria alimentare)
- 85.xx.x → MEDIO (istruzione)
- 86.xx.x → ALTO (sanità residenziale)
- 01.xx.x → MEDIO/ALTO (agricoltura: verificare attività)

> Per la classificazione puntuale ai fini formativi si rinvia all'Allegato dell'ASR 17/04/2025
> e alla skill `verifica-rischio-ateco`.

## Moduli tecnici-integrativi per DL-RSPP (art. 34) — solo per i settori indicati

| Settore | Codici ATECO 2007 | Durata |
|---|---|---|
| Agricoltura – Silvicoltura – Zootecnia | A 01-02 | 16 ore |
| Pesca | A 03 | 12 ore |
| Costruzioni | F (41-43) | 16 ore |
| Chimico-Petrolchimico | C 19-20 | 16 ore |

Per tutti gli altri settori il modulo integrativo **non si applica** (corso DL 16h + Modulo Comune 8h).
Fonte: `references/accordi/ASR_59_2025_nuovo_accordo_RAW.txt`, Parte II §4.

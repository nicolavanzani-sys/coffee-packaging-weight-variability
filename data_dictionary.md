# Data Dictionary

## Dataset 1 — `pesature_clean.csv` (Measure — baseline)

| Colonna | Descrizione | Tipo | Unità | Valori ammessi | Regola di qualità |
|---|---|---|---|---|---|
| `date` | Data di produzione | Date | — | date valide | Formato uniformato in fase di cleaning (parsing a due formati) |
| `shift` | Turno di produzione | Categoria | — | Mattina, Pomeriggio, Notte | Normalizzato (case, spazi) in fase di cleaning |
| `batch_id` | Identificativo lotto | Testo | — | un batch per turno/giorno | — |
| `operator_id` | Operatore responsabile | Testo | — | codice operatore | Verificata corrispondenza 1:1 con il turno |
| `weight_grams` | Peso della confezione | Numerico | grammi | 200–300 (range di plausibilità fisica) | Missing imputati con mediana per turno; valori impossibili (0, >2000) corretti; duplicati esatti rimossi |

**Righe:** 420 (dopo pulizia, da 424 grezze)

## Dataset 2 — `pilot_clean.csv` (Improve — pilot test contromisura)

| Colonna | Descrizione | Tipo | Unità | Valori ammessi | Regola di qualità |
|---|---|---|---|---|---|
| `date` | Data di produzione | Date | — | date valide | — |
| `shift` | Turno | Categoria | — | sempre "Notte" (test mirato) | — |
| `machine` | Macchina di test | Categoria | — | Macchina_1_Contromisura, Macchina_2_Riferimento | — |
| `batch_id` | Identificativo lotto | Testo | — | — | — |
| `weight_grams` | Peso della confezione | Numerico | grammi | 200–300 | Missing imputati con mediana **per macchina** (non globale, per non attenuare la differenza tra gruppi oggetto del test); duplicati rimossi |

**Righe:** 278 (dopo pulizia, da 282 grezze) — 138 Macchina_1, 140 Macchina_2

## Costanti di specifica (CTQ)

| Parametro | Valore |
|---|---:|
| Target | 250 g |
| USL | 255 g |
| LSL | 245 g |
| Cpk target (cliente) | 1,33 |

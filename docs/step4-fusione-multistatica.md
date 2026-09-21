# Step 4 — Fusione multistatica (ACCANTONATO — stretch goal futuro, dipende da Step 3)

> ⚠️ **Stato aggiornato: come lo Step 3 da cui dipende, questo step è accantonato per budget.** Il progetto attuale procede con singolo elemento + rotazione meccanica (Step 2). Questo documento resta come riferimento tecnico/algoritmico se in futuro si riprende l'estensione multistatica.

Nota preliminare: è lo step dove il progetto passa dall'ingegneria elettronica pura all'algoritmica di elaborazione segnale — se ripreso, richiede pochissimo hardware aggiuntivo oltre allo Step 3.

---

## 1. Cosa cambia concettualmente rispetto allo Step 3

Dataset multistatico: tutte le combinazioni TX/RX tra i 4 elementi (4×4=16 combinazioni, incluse le 4 diagonali già pulse-echo). Ogni combinazione (TX=i, RX=j) ha un percorso acustico diverso (elemento i → target → elemento j).

---

## 2. Hardware necessario (se ripreso)

Con la switching matrix a relè dello Step 3, serve la possibilità di selezionare **indipendentemente** l'elemento TX e l'elemento RX (non necessariamente lo stesso) — verificare che il cablaggio della matrice Step 3 lo permetta. Se sì, **nessun acquisto hardware aggiuntivo**, solo firmware.

---

## 3. Firmware — sequenza di acquisizione

1. Per ogni coppia (TX=i, RX=j), i,j da 1 a 4: seleziona, trigger pulse-echo, salva linea con etichetta (i,j)
2. 16 combinazioni totali, o 10 uniche sfruttando la reciprocità acustica (da verificare sperimentalmente)
3. Ripetere per ogni angolo di rotazione solo dopo aver validato la fusione multistatica a rotazione fissa

---

## 4. Algoritmo chiave: Delay-and-Sum (DAS) beamforming

Per ogni pixel dell'immagine: calcola il tempo di volo teorico elemento_i → pixel → elemento_j per ogni combinazione, "pesca" il valore del segnale a quel tempo, somma tutti i contributi. Riflettore vero = somma costruttiva; rumore = cancellazione parziale.

Passi implementativi (Python):
1. Griglia pixel (x,y)
2. Per ogni pixel e ogni (i,j): distanza geometrica elemento_i → pixel → elemento_j (posizioni note dal design meccanico)
3. Tempo di volo = distanza / velocità suono (**senza dividere per 2** — percorso già completo TX→target→RX)
4. Interpolazione lineare del segnale (i,j) a quell'istante
5. Somma contributi di tutte le combinazioni per quel pixel

Standard di base; variante successiva: **DMAS** (Delay-Multiply-and-Sum) per contrasto migliore.

---

## 5. Costo computazionale

(pixel) × (combinazioni TX/RX) operazioni. Griglia 200×200 + 16 combinazioni ≈ 640.000 operazioni/immagine — fattibile con NumPy vettorizzato, lentissimo con loop Python ingenui.

---

## 6. Checklist (se ripreso)

- [ ] Dataset multistatico completo acquisito (16 o 10 combinazioni)
- [ ] DAS implementato e vettorizzato in NumPy
- [ ] Immagine DAS confrontata con quella Step 2/3
- [ ] Verifica su target noto a due riflettori distinguibili

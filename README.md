# ProgettoQuantum
## 🎯 Obiettivo del Progetto
L'obiettivo principale è prevedere se uno studente supererà o meno l'esame finale di matematica, utilizzando un numero molto limitato di feature (2, 3 e 4). 
Il progetto valuta come le performance del VQC varino all'aumentare dei qubit, cambiando gli ottimizzatori classici (COBYLA vs COBYQA) e introducendo modelli di rumore (simulazione ideale vs. simulazione rumorosa).

## 📊 Dataset e Formulazione del Problema
È stato utilizzato lo **Student Performance Dataset** (studenti portoghesi di matematica). 
Il target continuo originale (`G3`, voto finale da 0 a 20) è stato binarizzato:
*   **Classe 1 (Promosso):** `G3 >= 10`
*   **Classe 0 (Non Promosso):** `G3 < 10`

Per mantenere i circuiti quantistici simulabili e analizzare l'impatto dell'aggiunta di qubit, i voti intermedi (`G1`, `G2`) sono stati volutamente esclusi. Sono stati creati tre set di feature progressivi:
1.  **2 Qubit (2 feature):** `failures`, `absences`
2.  **3 Qubit (3 feature):** `failures`, `absences`, `studytime`
3.  **4 Qubit (4 feature):** `failures`, `absences`, `studytime`, `higher`

## 🧠 Architettura dei Modelli

### Baseline Classica
Per ottenere un riferimento sulle performance, i dati sono stati addestrati (previa applicazione di SMOTE/SMOTENC per bilanciare le classi nel training set) su due modelli classici:
*   **Decision Tree** (max depth = 5)
*   **Random Forest** (200 estimatori, max depth = 5)

### Modello Quantistico Ibrido (VQC)
Il classificatore quantistico è stato costruito utilizzando la seguente architettura ibrida:
1.  **Codifica dei dati (Data Encoding):** `ZFeatureMap`. Le feature classiche sono state normalizzate nel range $[0, \pi]$ per mappare i dati in angoli di rotazione senza incorrere in ambiguità periodiche.
2.  **Circuito Parametrico (Ansatz):** `EfficientSU2` (con parametro `reps=2`). Scelto perché "hardware-efficient", combina rotazioni a singolo qubit e porte di entanglement per catturare le relazioni tra le feature.
3.  **Misurazione e Interpretazione:** Misurazione di tutti i qubit e assegnazione della classe tramite la **parità del bitstring** (numero pari di '1' $\rightarrow$ Classe 0; numero dispari di '1' $\rightarrow$ Classe 1).
4.  **Ottimizzazione (Classica):** Minimizzazione della *Log-Loss* tramite ottimizzatori derivative-free: **COBYLA** (approssimazione lineare) e **COBYQA** (approssimazione quadratica).

## 🧪 Esperimenti e Risultati

Gli esperimenti sul VQC sono stati condotti addestrando il modello su 80 esempi e testandolo su 40 (dovuto ai costi computazionali delle simulazioni quantistiche).

### Risultati Chiave:
*   **Baseline Classica:** La Random Forest si è dimostrata il modello più stabile, raggiungendo un'accuracy del **68.1%** (con 2 feature) e del **65.5%** (con 4 feature).
*   **VQC in Simulazione Ideale:** Il VQC ha mostrato risultati competitivi. Con 4 feature e ottimizzatore COBYLA, ha raggiunto un'accuracy del **70.0%** (F1-score 0.806). 
*   **Confronto Ottimizzatori:** Non c'è un vincitore assoluto. COBYQA ha sovraperformato COBYLA nella configurazione a 3 feature (accuracy 67.5% vs 55.0%), dimostrando che il VQC è altamente sensibile alla forma della loss e al metodo di ottimizzazione.
*   **Impatto del Rumore (Addestramento Rumoroso):** In un esperimento avanzato a 4 feature, il modello di rumore (depolarizzante) è stato iniettato *durante* l'ottimizzazione. Questo ha portato il modello a collassare sulla classe maggioritaria (predicendo tutti gli studenti come "promossi"). 

## 💡 Conclusioni e Takeaways
1.  **Metriche ingannevoli:** Il progetto dimostra l'importanza di non fermarsi all'Accuracy o all'F1-score della classe positiva in dataset sbilanciati. L'analisi delle **matrici di confusione** e del **Macro F1** è stata cruciale per rivelare il collasso del modello durante il training rumoroso.
2.  **Profondità del circuito:** Aumentare le ripetizioni dell'ansatz (`reps=3`) non porta necessariamente a risultati migliori rispetto a `reps=2`. Un circuito troppo profondo diventa più difficile da ottimizzare (Barren Plateaus / complessità dello spazio dei parametri).
3.  **Classico vs Quantistico:** Attualmente, sui dati tabellari analizzati, i modelli classici risultano più stabili e semplici da calibrare. Tuttavia, il VQC si è dimostrato capace di apprendere relazioni complesse, confermandosi un eccellente proof-of-concept per il Quantum Machine Learning.

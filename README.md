# Sistema Wearable di Biomedical Stream Analytics per il Monitoraggio e Contrasto delle Apnee Notturne (OSAS)

Un sistema IoT/Edge-AI a ciclo chiuso (*closed-loop*) per il monitoraggio in tempo reale del sonno, il rilevamento precoce del russamento e dei cali di saturazione d'ossigeno (SpO2), e la stimolazione tattile non invasiva per favorire il cambio di postura notturna senza provocare il risveglio cosciente.

---

## 🎯 Obiettivi del Progetto

La sindrome delle apnee ostruttive del sonno (**OSAS**) è un disturbo respiratorio diffuso che comporta rischi cardiovascolari e sonnolenza diurna. Questo progetto mira a:

1. **Monitoraggio non invasivo:** acquisire segnali fotopletismografici (PPG) per la saturazione arteriosa (SpO2) e la frequenza cardiaca esclusivamente tramite un **polsino all-in-one** (eliminando ingombranti sensori a clip o anelli al dito).
2. **Edge AI Audio Processing:** analizzare l'audio notturno in locale tramite modelli di Machine Learning (es. YAMNet quantizzato con TensorFlow Lite) per riconoscere il pattern acustico del russamento in tempo reale a latenza zero.
3. **Attivazione a ciclo chiuso (Bio-Feedback):** inviare un segnale di vibrazione calibrata (1–2 s) al polsino in caso di desaturazione o russamento persistente, stimolando il soggetto a cambiare posizione senza interrompere il sonno.
4. **Data Analytics & Time-Series:** archiviare lo storico notturno in un database NoSQL per serie temporali, calcolare parametri di sintesi (come l'**ODI** - *Oxygen Desaturation Index*) ed eseguire analisi esplorative, clustering e modelli predittivi con dashboard interattive.
5. **Privacy & Security by Design:** garantire che nessun flusso audio grezzo venga salvato o trasmesso all'esterno dell'ambiente locale, con cifratura delle comunicazioni e piena conformità ai principi del GDPR e dell'AI Act.

---

## 🛠️ Architettura del Sistema

Il sistema adotta un'architettura ibrida (stile Lambda) suddivisa tra elaborazione real-time e analisi storica:

```
┌──────────────────────────────────────────────┐
│            POLSINO WEARABLE (EDGE)           │
│  • Microcontrollore ESP32-C3 (BLE nativo)    │
│  • Sensore ottico MAX30102 (SpO2 / HR)       │
│  • Accelerometro IMU (filtraggio movimento)  │
│  • Mini attuatore a vibrazione (3V coin ERM) │
│  • Batteria LiPo ricaricabile                │
└──────────────────────┬───────────────────────┘
                       │
       [ Streaming BLE │ Feedback Vibrazione ]
                       ▼
┌──────────────────────────────────────────────┐
│           SPEED LAYER (SMARTPHONE)           │
│  • App Android (Kotlin / background service) │
│  • Microfono locale (niente cloud)           │
│  • Classificatore audio TFLite (YAMNet)      │
│  • Logica decisionale e trigger immediato    │
└──────────────────────┬───────────────────────┘
                       │
         [ Sincronizzazione LAN / Wi-Fi ]
                       ▼
┌──────────────────────────────────────────────┐
│             BATCH LAYER & ANALYTICS          │
│  • Storage NoSQL per Time-Series             │
│  • Job di elaborazione batch aggregata       │
│  • Machine Learning & Statistica predittiva  │
│  • Dashboard interattiva di visualizzazione  │
└──────────────────────────────────────────────┘
```

---

## 🔬 Specifiche Tecniche e Metodologia

### 1. Hardware e Acquisizione Segnali
* **Unità centrale:** Seeed XIAO ESP32-C3 (architettura RISC-V, Bluetooth Low Energy 5.0, Wi-Fi 802.11 b/g/n, consumi ultra-ridotti in sleep mode).
* **Sensore biometrico:** MAX30102 via interfaccia I2C (campionamento fotopletismografico a luce rossa e infrarossa).
* **Gestione degli artefatti:** integrazione con accelerometro per escludere finestre temporali disturbate dal movimento involontario del polso e calcolo del calo relativo di SpO2 ($\ge 3\%$) rispetto alla baseline individuale.
* **Attuazione tattile:** motore vibrazionale ERM piatto pilotato tramite transistor e diodo di protezione.

### 2. Audio Processing & Intelligenza Artificiale
* **Modello audio:** inferenza acustica on-device basata su **YAMNet** (pre-addestrato su AudioSet) con fine-tuning specifico sulla classe *Snoring* e quantizzazione a 8-bit (int8) per TensorFlow Lite.
* **Elaborazione in-memory:** il campionamento microfonico avviene esclusivamente nei buffer temporanei di memoria volatile; nessun file audio grezzo viene persistito su disco.

### 3. Pipeline Dati e Modellazione
* **Modellazione analitica:** schema concettuale basato su *Dimensional Fact Modeling (DFM)* per la gestione degli eventi notturni.
* **Archiviazione:** database orientato ai documenti / serie temporali per la memorizzazione dei campioni a 1 Hz e degli eventi discreti.
* **Analytics & Visualization:** pipeline di Machine Learning per l'identificazione dei pattern notturni (clustering PCA/k-means e modelli di classificazione) e dashboard per l'interpretazione visiva del trend respiratorio.

---

## ⚖️ Conformità Normativa e Protezione dei Dati

La progettazione tecnica è intrinsecamente integrata con i requisiti del quadro giuridico europeo ed internazionale:
* **GDPR (Reg. UE 2016/679):** trattamento di categorie particolari di dati sanitari (ex art. 9). Adozione rigorosa dei principi di *Privacy by Design e by Default (art. 25)*, minimizzazione, cifratura dei dati in transito (BLE con Secure Connections, TLS) e a riposo (art. 32).
* **AI Act (Reg. UE 2024/1689):** inquadramento del modello di intelligenza artificiale, requisiti di trasparenza verso l'utente, robustezza e supervisione umana.
* **Inquadramento MDR (Reg. UE 2017/745):** chiara delimitazione tra dispositivo *lifestyle/wellness* non invasivo e software medico di classe IIa (Regola 11), corredata da opportuni disclaimer operativi.

---

## 📁 Documentazione

* Per le specifiche architetturali e i requisiti completi, consultare il documento: [analisi.md](analisi.md).

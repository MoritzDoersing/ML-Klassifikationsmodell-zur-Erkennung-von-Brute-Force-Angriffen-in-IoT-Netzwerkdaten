# Netzwerkdaten-Analyse & Klassifikation von Angriffen

Binäre Klassifikation von schädlichem Netzwerkverkehr auf Basis flowbasierter Features aus dem CIC IoT-DIAD 2024 Datensatz.  
Fokus: Erkennung von Brute-Force-Angriffen in IoT-Umgebungen.

> **Autoren:** Doersing, Huke  
> **Kontext:** Universitätsprojekt — Masterstudiengang Data Science & KI

---

## Überblick

Dieses Projekt entwickelt eine Machine-Learning-Pipeline zur Klassifikation von Netzwerkflows als benigne oder anomal (Brute-Force-Angriffe). Der Datensatz stammt vom Canadian Institute for Cybersecurity und umfasst simulierten Datenverkehr von 105 IoT-Geräten mit 33 Angriffsszenarien.

**Zentrale Designentscheidungen:**
- Flowbasierte Feature-Auswahl (keine Payload-Inspektion) → anwendbar auf unbekannte und Zero-Day-Muster
- Unausgeglichener Datensatz bewusst beibehalten (~95,7 % benigne) — realistischere Abbildung echter Netzwerkbedingungen
- F1-Score (macro) als primäre Bewertungsmetrik aufgrund der Klassenungleichgewichte

---

## Projektstruktur

```
├── Doersing_Huke_KI_Model.ipynb   # Haupt-Notebook
├── requirements.txt
├── README.md
└── data/                          # Nicht im Repository — siehe Datensatz-Abschnitt
    ├── BenignTraffic.pcap_Flow.csv
    ├── BenignTraffic2.pcap_Flow.csv
    └── DictionaryBruteForce.pcap_Flow.csv
```

---

## Datensatz

**CIC IoT-DIAD 2024** — Canadian Institute for Cybersecurity  
Download: [https://www.unb.ca/cic/datasets/iot-diad-2024.html](https://www.unb.ca/cic/datasets/iot-diad-2024.html)

Die folgenden Dateien müssen vor der Ausführung in einem `data/`-Ordner im Projektstammverzeichnis abgelegt werden:

| Datei | Zugewiesenes Label |
|---|---|
| `BenignTraffic.pcap_Flow.csv` | benign |
| `BenignTraffic2.pcap_Flow.csv` | benign |
| `DictionaryBruteForce.pcap_Flow.csv` | anomaly |

---

## Features

Folgende flowbasierte Features wurden für das Training ausgewählt:

| Feature | Beschreibung |
|---|---|
| Flow Duration | Gesamtdauer der Verbindung |
| Total Fwd Packet | Paketanzahl in Vorwärtsrichtung |
| Total Length of Fwd Packet | Gesamtbytes in Vorwärtsrichtung |
| Total Length of Bwd Packet | Gesamtbytes in Rückwärtsrichtung |
| Flow IAT Mean | Mittlere Zwischenankunftszeit zwischen Paketen |
| Flow Bytes/s | Durchschnittlicher Durchsatz in Bytes pro Sekunde |
| SYN Flag Count | TCP-Verbindungsaufbau-Flags |
| ACK Flag Count | TCP-Bestätigungs-Flags |
| FIN Flag Count | TCP-Verbindungsabbau-Flags |

---

## Methodik

### 1 — Explorative Datenanalyse (EDA)
- Korrelationsmatrix (Pearson) zur Identifikation von Feature-Beziehungen
- Boxplots und KDE-Plots je Klasse zur Darstellung von Verteilungsunterschieden
- Zentrales Ergebnis: Anomale Flows konzentrieren sich auf niedrige Werte in fast allen Features — typisch für kurze, automatisierte Angriffsmuster

### 2 — Vorverarbeitung
- Baseline: `SimpleImputer (median)` + `StandardScaler`
- Erweitertes Grid: 5 Skalierer × 3 Imputing-Strategien × `SimpleImputer` / `KNNImputer`
- Beste Konfiguration für LightGBM: `KNNImputer` + `RobustScaler`

### 3 — Modellvergleich (Baseline)

| Modell | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| **Random Forest** | 0,9894 | 0,9382 | 0,9318 | **0,9350** |
| Decision Tree | 0,9869 | 0,9222 | 0,9166 | 0,9194 |
| LightGBM | 0,9774 | 0,8296 | 0,9730 | 0,8868 |
| Gradient Boosting | 0,9794 | 0,8934 | 0,8401 | 0,8646 |

### 4 — Hyperparameter-Optimierung (BayesSearchCV)
Random Forest und LightGBM wurden für die Optimierung ausgewählt. Bayesianische Optimierung mit 50 Iterationen, 4-facher Kreuzvalidierung, Scoring auf `f1_macro`.

Beste LightGBM-Parameter:
- `n_estimators`: 500, `learning_rate`: 0,033, `max_depth`: 15, `num_leaves`: 150, `min_child_samples`: 5, `subsample`: 1,0, `colsample_bytree`: 0,936

### 5 — Abschlussergebnisse

| Modell | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| **RandomForestClassifier** | **0,9896** | **0,9390** | **0,9332** | **0,9361** |
| LGBMClassifier | 0,9857 | 0,8912 | 0,9516 | 0,9190 |

**Gewähltes Modell: RandomForestClassifier** — höchster F1-Score, bestes Gleichgewicht aus Precision und Recall, niedrigste Fehlalarmrate.

---

## Installation & Ausführung

**Voraussetzung:** Python 3.11+

```bash
pip install -r requirements.txt
jupyter notebook Doersing_Huke_KI_Model.ipynb
```

> **Hinweis zu pandas 3.x:** Die `requirements.txt` verwendet die aktuellsten Paketversionen. pandas 3.0 enthält Breaking Changes gegenüber 2.x (u. a. neuer Standard-String-Datentyp). Sollten Kompatibilitätsprobleme auftreten, kann alternativ `pandas==2.2.3` installiert werden.

---

## Limitierungen & Ausblick

- Datensatz ist synthetisch (Laborumgebung) — reale Verkehrsverteilungen können abweichen
- Nur Brute-Force-Angriffe berücksichtigt; Erweiterung auf alle 33 Angriffsszenarien als nächster Schritt denkbar
- Sequenzielle oder graphbasierte Modelle (RNN, GCN) könnten die Erkennungsrate weiter verbessern
- Hybride regelbasierte + lernende Systeme für Produktionsumgebungen prüfenswert

---

## Lizenz

Datensatz: [CIC IoT-DIAD 2024](https://www.unb.ca/cic/datasets/iot-diad-2024.html) — Canadian Institute for Cybersecurity (nicht im Repository enthalten).  
Code: MIT License.

# 🌐 SDN Network Slicer — NCIS Project

> **Network & Cloud Infrastructures** — Progetto d'Esame  
> Implementazione di **Topology Slicing** con **Dynamic Slicing** e **classificazione ML del traffico** su rete SDN emulata con Mininet e Ryu.

---

## 📡 Architettura

La rete SDN è composta da **4 host** e **6 switch OpenFlow (OVS)** organizzati in due percorsi paralleli:

```
        H1 --+                              +-- H3
             S1 -- S2 -- S4 -- S6  (10 Mbps | Upper Slice - Video)
             S1 -- S3 -- S5 -- S6   (1 Mbps | Lower Slice - Dati)
        H2 --+                              +-- H4

        Controller: Ryu (OpenFlow 1.3) -- Dynamic Slicing attivo
```

| Slice | Hosts | Percorso | Banda |
|---|---|---|---|
| **Upper Slice** | H1 <-> H3 | S1->S2->S4->S6 | 10 Mbps |
| **Lower Slice** | H2 <-> H4 | S1->S3->S5->S6 | 1 Mbps |

---

## 🚀 Estensioni Implementate

| Estensione | Descrizione | Stato |
|---|---|---|
| **1. Dashboard** | Monitoraggio live + analisi ML in Streamlit | OK |
| **2. Classificazione Avanzata** | L1 (MAC/Port-based) + L2 (K-Means su packet statistics) | OK |
| **3. Dynamic Slicing** | Lower Slice usa Upper (10 Mbps) quando il video e' inattivo | OK |

---

## Requisiti

### Sulla VM (Mininet)
```bash
# Mininet + Open vSwitch
sudo apt install mininet openvswitch-switch

# Ryu controller
pip install ryu

# Dashboard dependencies
pip install streamlit pandas scikit-learn
```

### Opzionale: D-ITG (traffico piu' realistico)
```bash
sudo apt install d-itg
```

---

## Come avviare il progetto

### 1. Avvia il controller Ryu
```bash
ryu-manager main_controller.py
```

### 2. Avvia la topologia Mininet (in un altro terminale)
```bash
sudo python3 slicing_topo.py
```

### 3. Genera traffico (dalla CLI di Mininet)
```bash
# Traffico video - Upper Slice (H1 -> H3)
source video_traffic.sh

# Traffico dati - Lower Slice (H2 -> H4)
source normal_traffic.sh
```

### 4. Avvia la dashboard
```bash
streamlit run app.py
```
La dashboard sara' accessibile su `http://localhost:8501`

---

## Dashboard — Moduli

### Live Monitor
- Metriche in tempo reale per le due slice (banda stimata, volume cumulativo)
- Banner Dynamic Slicing con stato corrente (Fast Lane ON/OFF)
- Area chart per Upper e Lower Slice

### Analisi ML e Classificazione
- **Classificazione L1** (deterministica): identifica i flussi tramite MAC src/dst
- **Classificazione L2** (K-Means): usa solo byte_count e packet_count, senza conoscere MAC o IP
- **Confronto L1 vs L2**: percentuale di coerenza tra i due classificatori

### Topologia di Rete
- Visualizzazione SVG della rete con colori adattativi
- Tabella dettaglio switch
- Stato corrente Dynamic Slicing

---

## Struttura del Progetto

```
Progetto/
+-- app.py                          # Entry point Streamlit
+-- main_controller.py              # Controller Ryu
+-- slicing_topo.py                 # Topologia Mininet (6 switch)
+-- video_traffic.sh                # Genera traffico video
+-- normal_traffic.sh               # Genera traffico dati
+-- traffic_data.csv                # Dati raccolti dal controller
+-- dashboard/
    +-- utils.py                    # Costanti, stile e funzioni di utilita' condivise
    +-- pages/
        +-- live_monitor.py         # Pagina monitoraggio live
        +-- ml_analysis.py          # Pagina analisi ML
        +-- topology.py             # Pagina topologia SVG
```

---

## Dynamic Slicing — Logica

Il controller monitora la banda del flusso video ogni 10 secondi:

```
bandwidth_video = (delta_byte * 8) / (10s * 1_000_000)  ->  Mbps

if bandwidth_video < 0.5 Mbps:
    Sposta Lower Slice su Upper (10 Mbps) -- Fast Lane attiva
else:
    Riporta Lower Slice su Lower (1 Mbps) -- priorita' al video
```

---

## Classificazione a Due Livelli

```
Livello 1 - Port/MAC-based (deterministico)
  Il controller usa eth_src/eth_dst per instradare i flussi.
  Corrisponde al matching per porta nelle flow-table OpenFlow.
  Fornisce il ground truth per validare il modello ML.

Livello 2 - K-Means su packet statistics (non supervisionato)
  Input: solo byte_count + packet_count normalizzati (StandardScaler).
  Nessuna conoscenza di MAC, IP o porte.
  Scopre autonomamente le 2 categorie di traffico.
  La coerenza con L1 viene calcolata e mostrata nella dashboard.
```

---

## Note

- Il file `traffic_data.csv` viene generato automaticamente dal controller.
- Per abilitare la classificazione D-ITG (port-based a livello OpenFlow),
  cercare i commenti `[D-ITG]` in `main_controller.py` e seguire le istruzioni.

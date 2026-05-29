# MQTT QoS Analysis in an Edge-AI Surveillance System
### Networking Aspects of IoT — Telecommunication Engineering

---

## Project overview

This project provides an experimental analysis of MQTT broker scalability and QoS performance in a multi-camera edge-AI surveillance scenario.

A **Raspberry Pi** connected to a USB camera acts as the edge inference node: it runs **YOLOv26** locally and publishes structured JSON detection events to **HiveMQ Cloud** via MQTT. To simulate a multi-camera deployment, N additional publisher threads run on the Raspberry Pi, each generating synthetic events with a distinct `camera_id`, simulating independent physical cameras.

A **laptop** acts as the backend subscriber: it receives all events, logs them with timestamps, and captures WiFi traffic with Wireshark for offline analysis.

All traffic traverses the real WiFi link to HiveMQ Cloud and back, making Wireshark captures representative of actual network load.

---

## Research question

> *"How do overhead, packet loss, and end-to-end delay evolve as the number of concurrent publishers grows and as the QoS level changes?"*

---

## Network metrics

| Metric | Tool | What it reveals |
|---|---|---|
| **Additional overhead** | Wireshark / tshark | Bytes-on-wire vs useful payload bytes, across QoS levels and N |
| **Packet loss** | Publisher log vs subscriber log | Delivery rate vs N and QoS. Loss emerges organically at saturation |
| **End-to-end delay** | NTP-synced timestamps | p50 / p95 / p99 delay vs N and QoS. p99 is the key finding |

---

## QoS rationale

| Traffic type | Payload | QoS | Rationale |
|---|---|---|---|
| Raw video frame | ~20 KB | 0 | Loss of one frame is acceptable |
| Object detection event (YOLOv26) | ~300 B | 1 | At-least-once delivery required |
| Pose estimation / behaviour tracking | ~1 KB | 2 | Exactly-once delivery required |

---

## Architecture

```
[Raspberry Pi]                          [HiveMQ Cloud]
  USB Camera                                 |
  YOLOv26 inference                          |
  Thread 0: real publisher   --MQTT/TLS-->   |  --MQTT/TLS-->  [Laptop]
  Thread 1: synthetic cam-02 --MQTT/TLS-->   |                 subscriber
  Thread N: synthetic cam-N  --MQTT/TLS-->   |                 logger
                                             |                 Wireshark
```

---

## Experimental plan

| ID | Name | Independent variable | Key output |
|---|---|---|---|
| E1 | Baseline | N=1, QoS 1, 300 B | Reference overhead, loss, delay |
| E2 | QoS comparison | QoS {0,1,2} x payload type | Validates QoS selection rationale |
| E3 | Publisher scaling | N in {1,2,5,10,20}, QoS 1 | Broker saturation point |
| E4 | QoS under load | QoS {0,1,2} x N {1,5,10} | Reliability vs scalability trade-off |
| E5 | Payload scaling | Payload {300B,1KB,20KB} x QoS | Overhead vs payload size |

Each experiment is repeated 5 times. Results reported with mean and standard deviation (error bars on all plots).

---

## Repository structure

```
├── common.py           # Shared event schema and credentials loader
├── publisher.py        # Runs on Raspberry Pi (or laptop for testing)
├── subscriber.py       # Runs on laptop
├── analysis/
│   └── analyze.py      # Parses logs + pcap, computes metrics, plots
├── docs/
│   └── proposal.docx   # Project proposal submitted to the professor
├── logs/               # CSV logs — not tracked by Git
├── results/            # Output plots — not tracked by Git
├── .env.example        # Credential template (copy to .env and fill in)
├── requirements.txt
└── README.md
```

---

## Setup

**1. Clone the repository**
```bash
git clone https://github.com/giugiu39/Networking-Aspects-of-IoT-MQTT-QoS-Analysis-in-an-Edge-AI-Surveillance-System.git
cd Networking-Aspects-of-IoT-MQTT-QoS-Analysis-in-an-Edge-AI-Surveillance-System
```

**2. Create virtual environment and install dependencies**
```bash
# Windows
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt

# Raspberry Pi / Linux
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**3. Configure credentials**
```bash
cp .env.example .env
# Edit .env with your HiveMQ credentials
```

---

## Usage

**Start the subscriber on the laptop first:**
```bash
python subscriber.py --out logs/sub_run01.csv
```

**Then start the publisher (on Raspberry Pi or laptop for testing):**
```bash
# Baseline — 1 thread, QoS 1, 60 seconds
python publisher.py --n-threads 1 --rate 5 --duration 60 --qos 1

# Scaling test — 5 threads, QoS 0
python publisher.py --n-threads 5 --rate 5 --duration 60 --qos 0

# QoS comparison — 1 thread, QoS 2, medium payload
python publisher.py --n-threads 1 --rate 5 --duration 60 --qos 2 --payload-size medium
```

**Analyse results:**
```bash
python analysis/analyze.py --pub-log logs/pub_cam-01_qos1.csv \
                            --sub-log logs/sub_run01.csv \
                            --pcap logs/capture.pcap
```

---

## Hardware

- Raspberry Pi 4 + USB camera (edge inference node)
- Laptop / PC (subscriber + Wireshark + analysis)
- HiveMQ Cloud (MQTT broker)
- WiFi network (shared communication medium)

---

## Author

**Gianluca Perrotta**
Telecommunication Engineering — 1st year Master's
Exam: Networking Aspects of IoT
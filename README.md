# MQTT QoS Analysis in an Edge-AI Surveillance System
### Networking Aspects of IoT — Telecommunication Engineering

## Project overview
Experimental analysis of MQTT broker scalability and QoS performance
in a multi-camera edge-AI surveillance scenario.

A Raspberry Pi runs YOLOv26 inference and publishes detection events
to HiveMQ Cloud via MQTT. N synthetic publisher threads simulate
additional independent camera feeds. A laptop subscriber logs all
received events. Wireshark captures the WiFi traffic.

The three network metrics measured are:
- **Additional overhead** — bytes-on-wire vs payload bytes
- **Packet loss** — delivery rate vs number of publishers N
- **End-to-end delay** — p50/p95/p99 vs N and QoS level

## Repository structure
- `publisher.py` — runs on Raspberry Pi (or laptop for testing)
- `subscriber.py` — runs on laptop
- `common.py` — shared event schema and helpers
- `analysis/analyze.py` — parses logs and generates plots

## Setup
```bash
python -m venv venv
venv\Scripts\activate        # Windows
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill in your HiveMQ credentials.

## Usage
```bash
# Start subscriber (laptop)
python subscriber.py --out logs/sub_run01.csv

# Start publisher (1 thread, QoS 1, 60 seconds)
python publisher.py --n-threads 1 --rate 5 --duration 60 --qos 1

# Start publisher (5 threads, QoS 0)
python publisher.py --n-threads 5 --rate 5 --duration 60 --qos 0
```

## Author
Gianluca Perrotta — Telecommunication Engineering
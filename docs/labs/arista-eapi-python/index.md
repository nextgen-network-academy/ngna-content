# Arista eAPI with Python — Lab

> `// containerlab · ceos-lab · python · clone and run`

---

!!! abstract "Lab Repository"
    **[arista-eapi-python](https://github.com/nextgen-network-academy/arista-eapi-python)** ↗
    Containerlab topology, startup config, working code, starter skeleton, and challenge brief.

    ```bash
    git clone https://github.com/nextgen-network-academy/arista-eapi-python
    ```

---

## Repo structure

```
arista-eapi-python/
├── README.md
├── topo/
│   ├── topology.clab.yml        ← single cEOS node, eAPI pre-enabled
│   └── configs/
│       └── eos1.cfg             ← startup config baked into the device
├── reference/
│   └── eapi_show_version.py     ← complete working solution
├── starter/
│   └── eapi_starter.py          ← skeleton with TODO blocks
├── examples/
│   └── sample_output.json       ← captured eAPI response (offline use)
├── docs/
│   ├── instructor-guide.md      ← walkthrough notes
│   ├── student-challenge.md     ← challenge brief and rubric
│   └── instructor-notes.md      ← answer key and failure modes
├── requirements.txt
├── .env.example
└── .gitignore
```

---

## Prerequisites

| Tool | Purpose |
|---|---|
| Docker | Container runtime |
| Containerlab | Topology orchestrator — `containerlab.dev` |
| cEOS-lab image | Free download from `arista.com` (account required), imported as `ceos:4.32.0F` |
| Python 3.9+ | The script |

```bash
# One-time: import cEOS-lab after downloading from arista.com
docker import cEOS-lab-4.32.0F.tar ceos:4.32.0F
docker images | grep ceos

# Per-project: Python deps
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## Run it

```bash
# 1. Bring up the lab
sudo containerlab deploy -t topo/topology.clab.yml

# 2. Configure (defaults already match the lab — no edits needed)
cp .env.example .env

# 3. Run the reference script
python reference/eapi_show_version.py
```

You should see two GitHub-flavored markdown tables — one of switch facts
(`show version`) and one of interface status — printed straight to your
terminal.

(macOS / Windows-WSL2: drop the `sudo` if your Docker setup doesn't need it.)

---

## Tear down when you're done

```bash
sudo containerlab destroy -t topo/topology.clab.yml --cleanup
```

---

## Default lab credentials

| User | Password |
|---|---|
| admin | admin |

Baked into `topo/configs/eos1.cfg` for lab convenience. **Never reuse these
credentials anywhere a real network can reach.**

---

## Related guide

The conceptual walkthrough — REST vs JSON-RPC, anatomy of an eAPI call, and
the challenge brief — lives in the Learn section:

→ **[Learn: Arista eAPI with Python](../../learn/netops/arista-eapi-python/index.md)**

---

!!! warning "Lab credentials only"
    The reference script disables TLS verification (`verify=False`) so the
    cEOS-lab self-signed cert works out of the box. Never ship that setting
    against production switches — it makes Basic Auth credentials trivially
    interceptable.

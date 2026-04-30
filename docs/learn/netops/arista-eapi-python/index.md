# Arista eAPI with Python

> `// python · eapi · json-rpc · containerlab · day-2`

**Level:** ▓▓▓░░ Intermediate

Programmatic Day-2 operations on EOS. Pull device facts using Python and
eAPI, running against a containerlab-orchestrated cEOS node on your own
laptop. No physical switch required. The script you write here is the
inventory-collection foundation that scales out to fleet-wide automation.

---

!!! abstract "Companion lab"
    Working code, starter skeleton, sample data, and the containerlab topology:
    **[arista-eapi-python](https://github.com/nextgen-network-academy/arista-eapi-python)** ↗
    See the **[Lab page](../../../labs/arista-eapi-python/index.md)** for the clone command and repo structure.

---

## What you'll learn

- The difference between REST and JSON-RPC — and why it matters when reading docs
- How a startup config enables eAPI (and why pre-staging it in Git is the right pattern)
- How to call eAPI from Python with the `requests` library
- How to parse JSON responses and present them as readable tables
- How to handle authentication, TLS, and HTTP errors like a professional

---

## Prerequisites

```yaml
python:        3.9 or newer
docker:        20.10 or newer
containerlab:  0.50+ (containerlab.dev)
ceos-lab:      imported as ceos:4.32.0F (free download from arista.com)
skills:        basic Python (variables, dicts, functions)
```

The cEOS-lab image takes a one-time setup step:

```bash
# After downloading from arista.com:
docker import cEOS-lab-4.32.0F.tar ceos:4.32.0F
docker images | grep ceos
```

---

## A word on terminology

You will hear engineers call eAPI a "REST API." Strictly speaking, it isn't.
**eAPI is JSON-RPC 2.0 over HTTPS — always via HTTP `POST`.** The read-only
equivalent of a REST `GET` is a `show` command, which is idempotent and
doesn't change device state.

Knowing this saves you hours when you start reading Arista's docs side-by-side
with REST API tutorials and they don't quite line up. Arista's true REST
surface lives in CloudVision; that's a separate guide.

---

## REST vs JSON-RPC

|  | REST | JSON-RPC |
|---|---|---|
| Style | Resource-oriented (nouns) | Action-oriented (verbs) |
| HTTP verbs | GET / POST / PUT / DELETE | Almost always POST |
| URL shape | `/interfaces/Ethernet1` | `/command-api` (one endpoint) |
| Request body | Often empty for GET | Always a JSON envelope |
| Read example | `GET /version` | `POST {"method":"runCmds","params":{"cmds":["show version"]}}` |

Same wire transport (HTTPS + JSON). Different design philosophy.

---

## Anatomy of an eAPI call

Every eAPI request looks like this on the wire:

```http
POST /command-api HTTPS/1.1
Host: localhost:8443
Authorization: Basic YWRtaW46YWRtaW4=
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "method":  "runCmds",
  "params":  { "version": 1, "cmds": ["show version"], "format": "json" },
  "id":      1
}
```

Three things to call out:

- **HTTPS** — credentials in the `Authorization` header are base64, not
  encrypted. TLS is what keeps them safe in flight.
- **Basic Auth** — username and password joined by a colon and base64-encoded.
  Anyone who captures this header in plaintext owns your switch.
- **JSON-RPC envelope** — `method` says *what* (`runCmds`), `params.cmds`
  says *which* commands, `id` is just an echo so you can match responses
  to requests in async code.

---

## Spin up the lab

Bring up the cEOS node:

```bash
git clone https://github.com/nextgen-network-academy/arista-eapi-python
cd arista-eapi-python
sudo containerlab deploy -t topo/topology.clab.yml
```

(macOS / Windows-WSL2: drop the `sudo` if your Docker setup doesn't need it.)

You should see containerlab pull up a single node named
`clab-arista-eapi-python-eos1` and report it healthy. The eAPI is now
reachable at `https://localhost:8443`.

---

## What the startup config does

Open `topo/configs/eos1.cfg` and read the eAPI block:

```text
management api http-commands
   no shutdown
   protocol https
   no protocol http
```

That's the *exact same configuration* you'd type into a physical switch's
CLI. Pre-staging it as text in Git is the GitOps mindset in miniature —
configuration as code, version-controlled, reproducible, reviewable. You'll
see this pattern again in the [GitOps track](../../gitops/index.md).

The config also creates a lab admin user (`admin / admin`), a couple of
VLANs, and four interfaces with descriptions — that's what makes
`show interfaces status` return non-trivial data when you query it.

---

## Warm-up: same call with curl

Before reaching for Python, prove the API works with `curl`. This rules out
network, eAPI, and credential issues so you can debug just the Python layer
later.

```bash
curl -k -u admin:admin \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"runCmds","params":{"version":1,"cmds":["show version"],"format":"json"},"id":1}' \
  https://localhost:8443/command-api | python3 -m json.tool
```

If that returns a JSON blob with a `result` field, you're ready for Python.

---

## The Python script

The reference solution — fully commented and ready to run — lives in the
companion repo at `reference/eapi_show_version.py`. Read it after you've
made an honest first attempt at the challenge below.

The core call collapses to roughly this:

```python
import json, requests
from requests.auth import HTTPBasicAuth

payload = {
    "jsonrpc": "2.0",
    "method":  "runCmds",
    "params":  {"version": 1, "cmds": ["show version"], "format": "json"},
    "id":      1,
}

response = requests.post(
    f"https://{host}:{port}/command-api",
    data=json.dumps(payload),
    headers={"Content-Type": "application/json"},
    auth=HTTPBasicAuth(username, password),
    verify=False,        # lab only
    timeout=10,
)
response.raise_for_status()
data = response.json()["result"][0]
```

Three things students consistently miss the first time:

1. **`response.raise_for_status()`** — without it, a 401 or 500 is silently
   absorbed and you debug a `KeyError` instead of an auth problem.
2. **`response.json()["result"]` is a list** — one entry per command, in
   the order you sent them. `[0]` is `show version`, `[1]` is the next.
3. **eAPI errors are JSON-RPC errors, not HTTP errors.** A typo'd command
   returns HTTP 200 with an `error` block in the body. Check for that.

---

## The challenge

Build your own version of the script. Take the `starter/eapi_starter.py`
skeleton from the [companion repo](https://github.com/nextgen-network-academy/arista-eapi-python)
and complete it so it:

1. Connects to the lab's eAPI over HTTPS
2. Authenticates from a `.env` file (no hard-coded secrets)
3. Sends `show version` and `show interfaces status` in a single call
4. Parses the JSON response
5. Renders the data as a clean table — not a raw JSON dump
6. Handles bad credentials, unreachable host, and malformed commands gracefully

The full brief and grading rubric live in `docs/student-challenge.md` in
the repo.

---

## Tear down when you're done

```bash
sudo containerlab destroy -t topo/topology.clab.yml --cleanup
```

---

!!! tip "Stretch goals"
    Once the basic script works, push further:

    - Loop over multiple switches from an `inventory.yaml`
    - Add `argparse` so any `show` command can be passed at runtime
    - Output to CSV with a `--csv` flag
    - Re-implement using the official [`pyeapi`](https://github.com/arista-eosplus/pyeapi) library and reflect on the differences

---

## Related

- **[Lab: Arista eAPI with Python](../../../labs/arista-eapi-python/index.md)** — the cloneable code package
- **[NetOps Track](../index.md)** — other Day-2 ops guides

---

!!! warning "Production safety"
    The reference code uses `verify=False` and Basic Auth so it works against
    self-signed cEOS-lab certs. Production should use a CA-signed switch
    certificate, certificate pinning, and ideally token-based auth via
    CloudVision Service Accounts.

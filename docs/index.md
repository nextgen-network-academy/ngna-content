# AVD + CloudVision Campus Tags

> `// gitops · avd · cvaas · ansible · yaml`

**Difficulty:** ▓▓▓░░ Intermediate  
**Time:** ~45 minutes  
**Published:** April 2026

---

## Overview

Most teams treat Arista AVD and CloudVision Studios like they live in separate worlds. AVD owns Day-1 automation. CloudVision Studios is where operators live for Day-2. Without a clear contract between them, you end up with two teams stepping on each other.

CloudVision Campus Tags are that contract.

When AVD generates and applies Campus Tags during fabric deployment, CloudVision can:

- Render accurate campus topology views automatically
- Build Studio container hierarchies by tag query — no manual drag-and-drop
- Apply configlets to whole containers, not individual devices
- Enable Quick Actions so operators can safely make Day-2 port changes

This guide walks through the full workflow — from AVD fabric variables to a working CloudVision Static Studio manifest.

---

## Prerequisites

| Requirement | Version |
|-------------|---------|
| Arista AVD collection | `4.x+` |
| Ansible | `2.15+` |
| CloudVision as a Service (CVaaS) | Current |
| Python | `3.10+` |

```bash
# Install AVD collection
ansible-galaxy collection install arista.avd

# Verify
ansible-galaxy collection list | grep arista.avd
```

---

## The Problem in Plain Terms

Here's what happens without Campus Tags:

AVD deploys your campus fabric. Configs are pushed. Everything looks correct in EOS. But then an operator logs into CloudVision to make a Day-2 port change — and Studios doesn't know which devices belong to which campus, which building, or which IDF.

They have two options: edit the YAML source files (dangerous if they don't understand AVD), or try to manually assign devices in Studio containers (which AVD will overwrite on the next deploy).

Campus Tags eliminate both problems.

---

## Architecture

```
AVD fabric variables
  ↓ generate_cv_tags: true
AVD generates 4 tags per device
  ↓ cv_deploy playbook
Tags applied to devices in CloudVision
  ↓ tag queries
Studio container hierarchy built automatically
  ↓
Operators manage Day-2 changes safely in CloudVision UI
```

---

## Step 1 — Enable Campus Tag Generation

In your AVD fabric variables file, add the `generate_cv_tags` block:

```yaml title="group_vars/CAMPUS_FABRIC.yml"
# Campus fabric topology
campus_fabric_name: CAMPUS_FABRIC

# Enable CloudVision Campus Tag generation
generate_cv_tags:
  topology_hints: true
  campus_fabric: true
```

!!! tip "What these flags do"
    `topology_hints: true` generates device role tags (`cv_tags_topology_type`).
    `campus_fabric: true` generates campus hierarchy tags (`campus`, `campus_pod`, `campus_access_pod`).

---

## Step 2 — Understand the Four Tags

AVD stamps four tags on every device in the fabric:

| Tag | Example Value | Applied To |
|-----|---------------|------------|
| `campus` | `MAIN_CAMPUS` | All devices |
| `campus_pod` | `BUILDING_A` | All devices |
| `campus_access_pod` | `IDF_1A` | Access switches only |
| `cv_tags_topology_type` | `spine` / `leaf` / `member-leaf` | All devices |

!!! note "Access pod tag"
    `campus_access_pod` is intentionally not applied to spine switches — only to leaf and member-leaf devices that represent physical IDFs.

---

## Step 3 — Define the Static Studio Manifest

The `cv_static_config_manifest` tells CloudVision how to build your Studio container hierarchy using those tags as queries.

```yaml title="group_vars/CAMPUS_FABRIC.yml"
cv_static_config_manifest:
  name: "CAMPUS_FABRIC_MANIFEST"
  description: "AVD-generated campus fabric manifest"
  containers:
    - name: "{{ campus_fabric_name }}"
      tag_query: "campus:{{ campus_name }}"
      containers:
        - name: "{{ campus_pod }}"
          tag_query: "campus_pod:{{ campus_pod_name }}"
          containers:
            - name: "{{ campus_access_pod }}"
              tag_query: "campus_access_pod:{{ campus_access_pod_name }}"
```

!!! warning "Container naming"
    Container names in the manifest must match exactly what AVD generates as tag values. Mismatches result in devices not being placed — check your `group_vars` topology definitions carefully.

---

## Step 4 — Run the Deployment

```bash
# Build configs and generate tags
ansible-playbook playbooks/build.yml

# Deploy to CloudVision — this pushes configs AND applies tags
ansible-playbook playbooks/deploy_campus.yml
```

Expected output:

```
PLAY [Deploy AVD Campus Fabric + CloudVision Tags] ************************

TASK [arista.avd.eos_designs] ............................................. ok
TASK [arista.avd.eos_config_deploy_cvp] ................................... ok
TASK [arista.avd.cv_deploy] ............................................... ok

PLAY RECAP ****************************************************************
SPINE1   : ok=3  changed=1  unreachable=0  failed=0
IDF1     : ok=3  changed=1  unreachable=0  failed=0
IDF2     : ok=3  changed=1  unreachable=0  failed=0
IDF3     : ok=3  changed=1  unreachable=0  failed=0
```

---

## Step 5 — Validate in CloudVision

After deployment, log into CVaaS and verify:

**Campus Topology view:**
Navigate to **Network → Campus** — you should see your campus rendered with correct spine/leaf hierarchy.

**Studio containers:**
Navigate to **Studios → Static Config** — containers should be populated with devices placed by tag query, not manually assigned.

**Device tags:**
Select any device → **Tags** tab — confirm all four Campus tags are present with correct values.

---

## The Day-2 Result

With Campus Tags in place, operators can:

1. Open CloudVision → Campus topology
2. Navigate to their building → IDF → specific switch
3. See a front-panel view of the switch
4. Select a port → apply a port profile via Quick Actions
5. Execute the change

AVD remains the source of truth for fabric-level config. Operators stay in CloudVision for port-level Day-2 changes. Neither team steps on the other.

---

## Working Lab Repo

The full working lab with `group_vars`, `inventory`, and `playbooks` is in the lab repo:

```bash
git clone https://github.com/nextgen-network-academy/avd-campus-tags
cd avd-campus-tags
```

Repo structure:

```
avd-campus-tags/
├── README.md
├── group_vars/
│   ├── CAMPUS_FABRIC.yml      ← fabric variables + generate_cv_tags
│   └── CAMPUS_SWITCHES.yml    ← device-level variables
├── inventory/
│   └── inventory.yml          ← device inventory
├── playbooks/
│   ├── build.yml              ← generate EOS configs
│   └── deploy_campus.yml      ← push to CVaaS + apply tags
└── docs/
    └── topology.png           ← campus topology diagram
```

---

## Related Reading

- [Day-2 Ops with CloudVision Quick Actions](../netops/index.md) ← coming soon
- [Static Studio Manifests — zero to deployed](avd-campus-tags.md) ← planned
- [Arista AVD Documentation](https://avd.arista.com) ↗
- [Original newsletter article](https://arista-southwest-region.github.io/Newsletter/) ↗

---

!!! note "Questions or issues?"
    Found a bug in the lab? Have a question about your specific environment?
    Open an issue on [GitHub](https://github.com/nextgen-network-academy/avd-campus-tags/issues)
    or connect on [LinkedIn](https://www.linkedin.com/in/nicholas-dambrosio-0381871a/).

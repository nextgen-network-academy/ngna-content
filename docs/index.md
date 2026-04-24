# NextGen Network Academy

> `// netops · gitops · aiops · all-the-ops`

Real-world Arista AVD workflows, working lab configs, and step-by-step guides.  
Built by a practitioner, for engineers who actually ship things.

← [Back to nextgennetworkacademy.com](https://nextgennetworkacademy.com)

---

## What you'll find here

**[Learn](learn/index.md)** — Conceptual guides, how-tos, and deep dives. Understand the *why* behind the workflow before you run anything.

**[Labs](labs/index.md)** — Working repos with cloneable code. YAML, playbooks, configlets — all tested in a real lab environment.

**[About](about/index.md)** — Who built this and why.

---

## Latest drops

| Guide | Track | Status |
|-------|-------|--------|
| [AVD + CloudVision Campus Tags](learn/gitops/avd-campus-tags/index.md) | GitOps | ✅ Live |
| Day-2 Ops with CloudVision Quick Actions | NetOps | 🔧 Coming soon |
| Static Studio Manifests — zero to deployed | GitOps | 📋 Planned |
| AVD CI/CD pipeline with GitHub Actions | GitOps | 📋 Planned |

---

## Learning tracks

| Track | Focus | Level |
|-------|-------|-------|
| [📡 Foundations](learn/foundations/index.md) | EOS basics, CloudVision intro, AVD 101 | ▓░░░░ Starter |
| [🔧 NetOps](learn/netops/index.md) | EOS config, VLAN ops, Day-2 CloudVision | ▓▓▓░░ Intermediate |
| [⚙️ GitOps](learn/gitops/index.md) | AVD, Ansible, Git workflows, CI/CD | ▓▓▓▓░ Advanced |
| [🤖 AIOps](learn/aiops/index.md) | Ask AVA, NetDL, AI-driven telemetry | ▓▓░░░ Beginner |

---

## Quick start

```bash
# Clone the AVD campus tags working lab
git clone https://github.com/nextgen-network-academy/avd-cloudvision-campus-tags

# Install Arista AVD collection
ansible-galaxy collection install arista.avd

# Deploy campus fabric + CloudVision tags
ansible-playbook playbooks/deploy_campus.yml
```

---

!!! tip "New here?"
    Start with **[Foundations](learn/foundations/index.md)** if you're new to Arista AVD.  
    Already deploying EOS? Jump straight to **[GitOps](learn/gitops/index.md)**.

!!! note "About this site"
    All content is tested in a working lab environment using Arista cEOS and CVaaS.  
    Nothing here is theoretical — if it's published, it ran.
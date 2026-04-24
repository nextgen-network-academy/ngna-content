# NextGen Network Academy

> `// netops · gitops · aiops · all-the-ops`

Real-world Arista AVD workflows, working lab configs, and step-by-step guides.
Built by a practitioner, for engineers who actually ship things.

← [Back to nextgennetworkacademy.com](https://nextgennetworkacademy.com)

---

## What you'll find here

**[Learn](learn/index.md)** — Conceptual guides, how-tos, and deep dives. Understand the *why* behind the workflow.

**[Labs](labs/index.md)** — Working repos with cloneable code. YAML, playbooks, configlets — tested in a real lab.

**[About](about/index.md)** — Who built this and why.

---

## Latest drops

| Guide | Track | Status |
|-------|-------|--------|
| [AVD + CloudVision Campus Tags](avd-campus-tags/index.md) | GitOps | ✅ Live |
| Day-2 Ops with CloudVision Quick Actions | NetOps | 🔧 Coming soon |
| Static Studio Manifests — zero to deployed | GitOps | 📋 Planned |

---

## Quick start

```bash
# Clone the AVD campus tags working lab
git clone https://github.com/nextgen-network-academy/avd-campus-tags

# Install Arista AVD collection
ansible-galaxy collection install arista.avd

# Deploy campus fabric + CloudVision tags
ansible-playbook playbooks/deploy_campus.yml
```

---

!!! tip "New here?"
    Start with **[Foundations](learn/foundations/index.md)** if you're new to Arista AVD,
    or jump straight to **[Labs](labs/index.md)** if you want working code to clone.
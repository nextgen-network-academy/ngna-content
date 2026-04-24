# AVD + CloudVision Campus Tags — Lab

> `// working code · clone and run · real lab tested`

---

!!! abstract "Lab Repository"
    **[avd-cloudvision-campus-tags](https://github.com/nextgen-network-academy/avd-cloudvision-campus-tags)**  
    Working lab files — group_vars, inventory, playbooks, configlets.

    ```bash
    git clone https://github.com/nextgen-network-academy/avd-cloudvision-campus-tags
    ```

---

## Repo structure

```
avd-cloudvision-campus-tags/
├── README.md
├── group_vars/
│   ├── CAMPUS_FABRIC.yml      ← fabric variables + generate_cv_tags
│   └── CAMPUS_SWITCHES.yml    ← device-level variables
├── inventory/
│   └── inventory.yml          ← device inventory
├── playbooks/
│   ├── build.yml              ← generate EOS configs
│   └── deploy_campus.yml      ← push to CVaaS + apply tags
└── configlets/
    └── Building_A_Banner.cfg  ← example configlet
```

---

## Prerequisites

```bash
# Install Arista AVD collection
ansible-galaxy collection install arista.avd

# Verify
ansible-galaxy collection list | grep arista.avd
```

---

## Related guide

The full conceptual walkthrough — architecture, tag variables, Studio manifest,
and Day-2 operations — is in the Learn section:

→ **[Learn: AVD + CloudVision Campus Tags](../../learn/gitops/avd-campus-tags/index.md)**

---

!!! warning "Before you run anything"
    Always validate configs in a lab environment before deploying to production.
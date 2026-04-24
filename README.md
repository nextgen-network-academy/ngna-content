# ngna-content

> MkDocs content site for [learn.nextgennetworkacademy.com](https://learn.nextgennetworkacademy.com)

## What this repo is

This is the **content layer** for NextGen Network Academy — all guides, lab walkthroughs, how-tos, and learning track overviews. Built with MkDocs Material and a custom retro theme matching the storefront.

## Repo structure

```
ngna-content/
├── mkdocs.yml                        ← MkDocs config
├── docs/
│   ├── index.md                      ← Home page
│   ├── stylesheets/
│   │   └── extra.css                 ← Retro theme CSS
│   ├── learn/
│   │   ├── index.md
│   │   ├── foundations/index.md
│   │   ├── netops/index.md
│   │   ├── gitops/
│   │   │   ├── index.md
│   │   │   └── avd-campus-tags.md
│   │   └── aiops/index.md
│   ├── labs/
│   │   ├── index.md
│   │   └── avd-campus-tags/index.md  ← Main lab guide
│   └── about/index.md
└── .github/
    └── workflows/
        └── deploy.yml                ← Auto-deploy on push to prod
```

## Branch strategy

```
dev   ← daily work, default branch
qa    ← validation
prod  ← auto-deploys to learn.nextgennetworkacademy.com
```

## Local preview

```bash
pip install mkdocs-material
mkdocs serve
# → http://127.0.0.1:8000
```

## Related repos

| Repo | Purpose |
|------|---------|
| [nextgennetworkacademy.github.io](https://github.com/nextgen-network-academy/nextgennetworkacademy.github.io) | Storefront |
| [avd-campus-tags](https://github.com/nextgen-network-academy/avd-campus-tags) | Working lab repo |

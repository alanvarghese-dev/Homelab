# 📚 Homelab Documentation (`docs/`)

This directory is designated for deep-dive architectural documents, network topologies, runbooks, and disaster recovery procedures for the homelab.

---

## 📑 Core Documentation Links

- **[Main Homelab Overview](../README.md)**: Architecture, quickstart, cluster setup, and operational cheatsheet.
- **[Applications Guide](../apps/README.md)**: Kustomize base + overlay pattern and application onboarding.
  - **[Linkding App Guide](../apps/base/linkding/README.md)**: Bookmark manager specs, setup, and admin commands.
  - **[Mealie App Guide](../apps/base/mealie/README.md)**: Recipe manager specs, credentials, and feature overview.
- **[GitOps & Clusters Guide](../clusters/README.md)**: Flux CD v2 structure, reconciliation flow, and bootstrap instructions.

---

## 🔮 Planned Documentation

- [ ] Ingress & DNS routing design (Traefik / NGINX / Pi-hole / Cloudflare Tunnel)
- [ ] TLS certificate automation runbook (`cert-manager` with Let's Encrypt / ACME)
- [ ] Secrets management policy (SOPS with Age or HashiCorp Vault)
- [ ] Backup & restoration strategy for PVCs (`local-path-provisioner` snapshots / Velero)

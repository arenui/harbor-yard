# Harbor Yard

**Where home infrastructure is staged, maintained, and deployed.**

Harbor Yard is my personal home-operations repository. It documents how I define, deploy, and maintain services across a very small fleet of machines using a pull-based, Git-defined workflow.

This repo is intentionally **public and sanitized**. It shows _how_ the system is structured, not the live state of my home network.

---

## What this repository demonstrates

- Pull-based configuration management with **Ansible (`ansible-pull`)**
- Declarative service definitions using **Docker Compose**
- Host-specific configuration via inventory and variables
- A clean separation between **public infrastructure patterns** and **private operational data**
- Practical, low-ceremony “home GitOps” patterns

---

## How it works (high level)

- Each host runs a systemd timer that periodically pulls a Git repository
- Ansible converges the host to its desired state:
  - baseline system configuration
  - Docker installation and configuration
  - deployment of assigned Compose stacks
- Services self-heal back to the declared state over time

This keeps infrastructure reproducible, auditable, and easy to evolve.

---

## Repository structure

```text
inventory.example.yml     # Example inventory (sanitized)
host_vars.example/        # Example per-host configuration
roles/                    # Ansible roles (common, docker, stacks, etc.)
stacks/                   # Docker Compose stack definitions
site.yml                  # Main Ansible entrypoint
```

Secrets, real hostnames, IPs, and environment-specific values are not stored here.

---

## Secrets handling

Real deployments use environment files or encrypted secrets managed outside this public repository.

This repo may include:

- .env.example files
- optional dummy SOPS-encrypted examples to demonstrate workflow

No real credentials are present.

---

## Why the public / private split?

The live system is driven by a private repository that mirrors a sanitized subset of its contents into this public one.

This allows:

- safe public documentation of infrastructure patterns
- realistic structure without leaking sensitive data

---

## Status

This is an actively evolving system.

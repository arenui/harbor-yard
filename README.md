# Harbor Yard

Declarative infrastructure management using Ansible to deploy and maintain Docker-based services across multiple hosts.

## What It Does

Harbor Yard configures Linux hosts and deploys containerized services using a simple, reproducible approach:

1. **System setup** - Installs Docker, configures basic system settings
2. **Stack deployment** - Deploys Docker Compose applications from the `stacks/` directory
3. **Automated updates** - Optional pull-based mode where hosts automatically sync and apply changes

## Quick Start

### Prerequisites

- Ansible installed locally (`pip install ansible`)
- SSH access to target hosts
- Sudo privileges on target hosts

### Setup

1. **Install dependencies**:

   ```bash
   ansible-galaxy install -r requirements.yml
   ```

2. **Configure your infrastructure**:

   ```bash
   # Copy example files
   cp inventory.example.yml inventory.yml
   cp -r group_vars.example group_vars
   cp -r host_vars.example host_vars

   # Edit inventory with your hosts
   vim inventory.yml

   # Configure which stacks to deploy per host
   vim host_vars/your-host.yml
   ```

3. **Run the playbook**:

   ```bash
   # Check what would change
   ansible-playbook site.yml --check

   # Apply configuration
   ansible-playbook site.yml
   ```

## Configuration

### Inventory

Define your hosts in `inventory.yml`:

```yaml
all:
  hosts:
    server1:
      ansible_host: 192.168.1.100
      ansible_user: admin
```

### Group Variables

Set common configuration in `group_vars/all.yml`:

```yaml
common_timezone: "America/Los_Angeles"
docker_users:
  - your-username
```

### Host Variables

Specify which stacks to deploy per host in `host_vars/<hostname>.yml`:

```yaml
stacks:
  - homeassistant
  - zigbee2mqtt
```

## Stacks

Docker Compose applications live in `stacks/`. Each stack is a directory containing at minimum a `docker-compose.yml` or `compose.yml` file.

### Adding a Stack

1. Create `stacks/myapp/docker-compose.yml`
2. Add `myapp` to a host's `stacks` list in `host_vars/`
3. Run the playbook

### Environment Variables

Stacks can reference environment files for secrets:

```yaml
# In host_vars/server.yml
stacks_config:
  myapp:
    env_file: "/etc/secrets/myapp.env"
```

The env file should exist on the target host at the specified path.

## Pull-Based Mode

Instead of running Ansible from your laptop, hosts can pull and apply changes automatically using `ansible-pull`.

Enable by setting in `group_vars/all.yml` or per-host:

```yaml
harbor_yard_enable_timer: true
harbor_yard_repo_url: "https://github.com/you/harbor-yard.git"
harbor_yard_branch: "main"
```

This installs a systemd timer that periodically runs `ansible-pull` to sync from git and apply changes.

## Roles

- **common** - Basic system configuration (timezone, packages)
- **docker** - Installs and configures Docker
- **stacks** - Deploys Docker Compose applications
- **pull_timer** - Sets up ansible-pull automation
- **nvidia** - NVIDIA GPU support for containers
- **observability_agent** - Monitoring/logging agents
- **observability_core** - Central monitoring stack
- **backup_restic** - Backup configuration with Restic

Roles are applied in order as defined in [site.yml](site.yml).

## Secrets Management

Secrets should never be committed to the repository. Options:

1. **Manual deployment** - Place env files on hosts before running Ansible
2. **Ansible Vault** - Encrypt secrets in the repo (add to `.gitignore`)
3. **External secret managers** - Extend roles to fetch from Vault, SOPS, etc.

## Directory Structure

```
.
├── site.yml                 # Main playbook entrypoint
├── inventory.yml            # Host definitions (not included)
├── requirements.yml         # Ansible Galaxy dependencies
├── group_vars/              # Variables for all hosts
├── host_vars/               # Per-host configuration
├── roles/                   # Ansible roles
│   ├── common/
│   ├── docker/
│   ├── stacks/
│   └── ...
└── stacks/                  # Docker Compose applications
    ├── homeassistant/
    │   └── docker-compose.yml
    └── zigbee2mqtt/
        └── docker-compose.yml
```

## Workflow

### Push-Based (Default)

1. Make changes locally
2. Run `ansible-playbook site.yml` to apply
3. Changes happen immediately

### Pull-Based (Automated)

1. Make changes and push to git
2. Hosts pull changes on a schedule (every 15 minutes by default)
3. Changes apply automatically

## Testing

```bash
# Syntax check
ansible-playbook site.yml --syntax-check

# Dry run on specific host
ansible-playbook site.yml -l hostname --check

# See what would change
ansible-playbook site.yml --check --diff
```

## License

MIT

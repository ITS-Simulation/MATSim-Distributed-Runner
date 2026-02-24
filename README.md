[![Generate Workers](https://github.com/ITS-Simulation/MATSim_Distributed_Runner/actions/workflows/sync-config.yml/badge.svg)](https://github.com/ITS-Simulation/MATSim_Distributed_Runner/actions/workflows/sync-config.yml)

# MATSim Distributed Runner

[🇻🇳 Tiếng Việt](./README_VI.md)

This repository hosts the runner configurations for the MATSim Distributed simulation environment. It acts as a deployment hub that automatically manages multiple worker configurations across different hardware profiles.

## 🚀 Automation & Branching Strategy

This repository uses a unique branching model driven entirely by automation. **Please do not manually edit the runner branches.**

*   **`main`**: The source of truth. Contains the central `config.yaml`, the base `Dockerfile`, and the `docker-compose.yaml` template.
*   **Runner Branches** (e.g., `i7`, `i7-high`, `i5`): Automatically generated to represent specific hardware and worker configurations.

### How It Works
1.  **Configuration**: Hardware limits and worker counts are defined in the `config.yaml` file.
2.  **Sync**: Whenever the `main` branch is updated, a GitHub Action (`sync-config.yml`) is triggered to automatically:
    *   Create or update the branches for each defined configuration.
    *   Inject the specific CPU/Memory limits and worker counts into the `docker-compose.yaml` of each branch.
    *   Propagate the latest `Dockerfile` from the `main` branch.
    *   Generate a bilingual `README.md` and `README_VI.md` for each runner branch, detailing its hardware specs, version, and scenario information.
    *   Prune any orphan branches that have been removed from the `config.yaml`.

## ⚙️ Configuration (`config.yaml`)

Runner profiles are configured in the `config.yaml` file on the `main` branch. An example configuration:

```yaml
ip: "192.168.1.1"  # The IP address of the central server

runner:
  i7:              # Profile name (can be any valid git branch name)
    hw:
      cpu: 26.0    # The CPU limit for Docker
      memory: "10G" # The RAM limit for Docker
    workers:
      high: 10     # Creates a branch named 'i7-high' with 10 workers
      normal: 8    # Creates a branch named 'i7' with 8 workers (this acts as the base branch)
      mid: 6       # Creates a branch named 'i7-mid' with 6 workers
      low: 4       # Creates a branch named 'i7-low' with 4 workers
  i5:
    hw:
      cpu: 18.0
      memory: "5G"
    workers:
      high: 6
      normal: 4
```

### Naming Profiles and Tiers
*   **Flexibility**: Profile names (like `i7` or `i5`) and tier names (like `high` or `low`) are entirely unrestricted. Any naming scheme can be used as long as it fits the deployment environment (e.g., CPU lineups, office locations).
*   **The Base Branch**: The tier names `normal` and `base` are reserved as special markers. Using either of these will create a **base branch** named exactly after the profile (for example, branch `i7`). Any other tier name will create a branch in the format `<profile>-<tier>` (like `i7-high`). If neither a `normal` nor `base` tier is defined, no base branch will be created for that profile.
*   **Git Constraints**: As these names translate directly to git branches, they must follow standard git branch naming rules. This means no spaces, and no special characters such as `~`, `^`, `:`, `?`, `*`, `[`, or `\`. They also cannot start or end with a `.` or `/`, contain consecutive `..`, or end with `.lock`.

## 🛠️ Deployment

To deploy a specific runner configuration, simply pull the corresponding branch:

```bash
# Deploy the i7 high-performance configuration
git clone https://github.com/ITS-Simulation/MATSim_Distributed_Runner.git
git checkout i7-high
docker compose up -d --build
```

## 📦 Update Process

Updates are triggered automatically from the [`MATSim-Bus-Optimizer`](https://github.com/ITS-Simulation/MATSim-Bus-Optimizer) repository:
1.  New release in `MATSim-Bus-Optimizer` → Updates `Dockerfile` on `main` (version, checksum).
2.  `sync-config` workflow triggers → Updates all runner branches.
3.  Runners simply need to pull and restart.

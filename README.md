<h1 align="center">📦 Ansible Role: Packer</h1>

<p align="center">
  <em>Install <a href="https://www.packer.io">HashiCorp Packer</a> on Linux — fast, idempotent, and checksum-verified.</em>
</p>

<p align="center">
  <a href="https://github.com/tblakex01/ansible-role-packer/actions/workflows/ci.yml"><img src="https://github.com/tblakex01/ansible-role-packer/actions/workflows/ci.yml/badge.svg" alt="CI Status"></a>
  <a href="https://galaxy.ansible.com/ui/standalone/roles/geerlingguy/packer/"><img src="https://img.shields.io/badge/galaxy-geerlingguy.packer-660198?logo=ansible&logoColor=white" alt="Ansible Galaxy"></a>
  <a href="https://github.com/tblakex01/ansible-role-packer/blob/master/LICENSE"><img src="https://img.shields.io/github/license/tblakex01/ansible-role-packer?color=blue" alt="License"></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Packer-1.15.4-02A8EF?logo=packer&logoColor=white" alt="Packer 1.15.4">
  <img src="https://img.shields.io/badge/ansible--core-%E2%89%A5%202.15-EE0000?logo=ansible&logoColor=white" alt="ansible-core >= 2.15">
  <img src="https://img.shields.io/badge/tested%20with-Molecule-blueviolet?logo=ansible&logoColor=white" alt="Tested with Molecule">
  <img src="https://img.shields.io/badge/platforms-Ubuntu%20%7C%20Debian%20%7C%20EL-orange?logo=linux&logoColor=white" alt="Platforms">
</p>

---

## ✨ Features

- 🚀 **Latest Packer** — installs Packer **1.15.4** by default (any version is configurable).
- 🔐 **Checksum-verified downloads** — every archive is validated against HashiCorp's published `SHA256SUMS` before installation.
- 🧠 **Architecture auto-detection** — `amd64`, `arm64`, `arm`, `386`, and `ppc64le` resolved automatically from host facts.
- ♻️ **Idempotent** — safe to run repeatedly; the binary is only re-downloaded when the version changes.
- 🧩 **Modern Ansible** — fully-qualified collection names (FQCN), `argument_specs` validation, and a clean `ansible-lint` (production profile).
- ✅ **CI-tested** — linted with `yamllint` + `ansible-lint` and verified end-to-end with Molecule across Ubuntu, Debian, and Enterprise Linux.

## 📑 Table of Contents

- [Requirements](#-requirements)
- [Installation](#-installation)
- [Role Variables](#-role-variables)
- [Example Playbooks](#-example-playbooks)
- [How It Works](#-how-it-works)
- [Testing](#-testing)
- [Compatibility](#-compatibility)
- [License](#-license)
- [Author Information](#-author-information)

## 📋 Requirements

- **ansible-core** `>= 2.15`
- A Linux target with internet access to `releases.hashicorp.com`

No external collections or role dependencies are required.

## 📥 Installation

Install the role from Ansible Galaxy:

```bash
ansible-galaxy role install geerlingguy.packer
```

Or pin it via a `requirements.yml`:

```yaml
---
roles:
  - name: geerlingguy.packer
    # src: https://github.com/tblakex01/ansible-role-packer  # to use this fork
```

```bash
ansible-galaxy role install -r requirements.yml
```

## ⚙️ Role Variables

All variables are defined in [`defaults/main.yml`](defaults/main.yml) and validated by [`meta/argument_specs.yml`](meta/argument_specs.yml).

| Variable | Default | Description |
| --- | --- | --- |
| `packer_version` | `"1.15.4"` | Version of Packer to install. |
| `packer_arch` | _auto-detected_ | Release architecture. Derived from `ansible_facts['architecture']`; override to force a value. |
| `packer_arch_map` | _see below_ | Maps the host architecture fact to a Packer release architecture string. |
| `packer_bin_path` | `/usr/local/bin` | Install directory for the `packer` binary (must be on `$PATH`). |
| `packer_verify_checksum` | `true` | Verify the download against HashiCorp's `SHA256SUMS`. |
| `packer_download_url` | _derived_ | Full URL of the release archive. |
| `packer_checksum_url` | _derived_ | URL of the `SHA256SUMS` file used for verification. |
| `packer_download_dest` | _derived_ | Local path the archive is downloaded to. |

<details>
<summary><strong>Architecture mapping (<code>packer_arch_map</code>)</strong></summary>

```yaml
packer_arch: "{{ packer_arch_map[ansible_facts['architecture']] | default('amd64') }}"
packer_arch_map:
  x86_64: amd64
  aarch64: arm64
  armv7l: arm
  armv6l: arm
  i386: "386"
  ppc64le: ppc64le
```

Extend the map if you need to support additional architectures.
</details>

## 🚀 Example Playbooks

**Minimal:**

```yaml
- hosts: servers
  become: true
  roles:
    - role: geerlingguy.packer
```

**Pin a version and install directory:**

```yaml
- hosts: build_agents
  become: true
  roles:
    - role: geerlingguy.packer
      vars:
        packer_version: "1.15.4"
        packer_bin_path: /usr/local/bin
```

**Air-gapped / skip checksum verification (not recommended):**

```yaml
- hosts: servers
  become: true
  roles:
    - role: geerlingguy.packer
      vars:
        packer_verify_checksum: false
```

## 🔍 How It Works

1. 🧰 Ensures `unzip` is installed via the system package manager.
2. ⬇️ Downloads `packer_<version>_linux_<arch>.zip` with `ansible.builtin.get_url`, passing the `SHA256SUMS` URL as the `checksum` so Ansible verifies integrity by filename.
3. 📦 Unarchives the binary into `packer_bin_path`, using `creates:` so the step is skipped when Packer is already present.

## 🧪 Testing

This role is tested with [Molecule](https://ansible.readthedocs.io/projects/molecule/) and Docker.

```bash
pip3 install ansible molecule "molecule-plugins[docker]" docker ansible-lint yamllint

# Lint
yamllint .
ansible-lint

# Full converge → idempotence → verify cycle
molecule test
```

Target a specific distribution with `MOLECULE_DISTRO`:

```bash
MOLECULE_DISTRO=debian12 molecule test
```

The CI matrix runs against `ubuntu2404`, `debian12`, and `rockylinux9`.

## 🖥️ Compatibility

| Family | Versions |
| --- | --- |
| 🟠 Ubuntu | 22.04 (Jammy), 24.04 (Noble) |
| 🔴 Debian | 11 (Bullseye), 12 (Bookworm) |
| 🔵 Enterprise Linux | 8, 9 |

## 📄 License

Released under the **MIT / BSD** license — see [`LICENSE`](LICENSE).

## 👤 Author Information

Originally created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [*Ansible for DevOps*](https://www.ansiblefordevops.com/), and modernized for current Packer and Ansible releases.

# Ansible Role: Packer

[![CI](https://github.com/tblakex01/ansible-role-packer/actions/workflows/ci.yml/badge.svg)](https://github.com/tblakex01/ansible-role-packer/actions/workflows/ci.yml)

Installs [Packer](https://www.packer.io), a Go-based image and box builder, on Linux. The downloaded release archive is checksum-verified against HashiCorp's published `SHA256SUMS` before installation.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
packer_version: "1.15.4"
```

The Packer version to install.

```yaml
packer_arch: "{{ packer_arch_map[ansible_facts['architecture']] | default('amd64') }}"
```

The release architecture to install. By default this is auto-detected from the host's architecture fact (e.g. `x86_64` → `amd64`, `aarch64` → `arm64`). Override it to force a specific value.

```yaml
packer_arch_map:
  x86_64: amd64
  aarch64: arm64
  armv7l: arm
  armv6l: arm
  i386: "386"
  ppc64le: ppc64le
```

The mapping used to translate `ansible_architecture` into a Packer release architecture. Extend it if you need to support additional architectures.

```yaml
packer_bin_path: /usr/local/bin
```

The location where the Packer binary will be installed (should be in the system `$PATH`).

```yaml
packer_verify_checksum: true
```

Whether to verify the downloaded archive against HashiCorp's published `SHA256SUMS`. Set to `false` to skip verification (not recommended).

## Dependencies

None.

## Example Playbook

```yaml
- hosts: servers
  roles:
    - role: geerlingguy.packer
      vars:
        packer_version: "1.15.4"
```

## Testing

This role is tested with [Molecule](https://ansible.readthedocs.io/projects/molecule/) and Docker, and is exercised on every push and pull request by the [CI workflow](.github/workflows/ci.yml).

### What the tests cover

The CI pipeline runs two stages:

1. **Lint** — `yamllint` and `ansible-lint` over the role.
2. **Molecule** — a matrix of supported distributions (Ubuntu 24.04/22.04, Debian 12/11, Rocky Linux 9/8) that each run the full `create → converge → idempotence → verify` sequence. This both installs Packer via the role and proves the run is idempotent.

The Molecule `verify` step performs an **end-to-end check** of the installed binary: it asserts the expected version, then runs a real `packer init`, `packer validate`, and `packer build` against a `null`-builder template with a `shell-local` provisioner (no cloud credentials or target machine required), confirming Packer is fully functional after installation.

### Running the tests locally

```bash
pip3 install -r requirements.txt
molecule test
```

Set `MOLECULE_DISTRO` to test against a specific distribution image (e.g. `MOLECULE_DISTRO=debian12 molecule test`).

## License

MIT / BSD

## Author Information

This role was originally created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/), and modernized for current Packer and Ansible releases.

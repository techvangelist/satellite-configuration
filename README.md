# Satellite Configuration

Ansible automation to configure a blank Red Hat Satellite 6.19 server from scratch. Designed to run from Ansible Automation Platform (AAP) 2.7 using the `redhat.satellite` collection.

## What It Configures

- **Organization & Locations** — Renames the default org, creates Birmingham / Dubai / Johannesburg locations
- **Lifecycle Environments** — Library → Development → QA-Test → Production
- **Red Hat Repositories** — RHEL 8/9/10 BaseOS, AppStream, Kickstart, Satellite Client, Fast Datapath, AAP, OCP (x86_64 + aarch64)
- **Custom Repositories** — EPEL 8/9/10, container registries (Quay + local), Fedora Flatpak remote
- **Content Views** — 15 x86_64 + 7 aarch64 standard CVs
- **Composite Content Views** — 11 CCVs combining base + tooling CVs for server, edge, AAP, and OCP use cases
- **Activation Keys** — Per lifecycle environment per host type (21 total)
- **Provisioning** — OS entries, domain, subnet, PXE templates (PXELinux BIOS + Grub2 UEFI), bootc kickstart template
- **Host Groups** — 7 parent groups (RHEL 8/9/10, Edge, aarch64 variants) with Dev/QA/Prod children each

## Requirements

- Red Hat Satellite 6.19+ with a valid manifest attached
- AAP 2.7 with the `ee-supported-rhel9` execution environment (includes `redhat.satellite` 5.11.0)
- SSH access from AAP to the Satellite server (or `ansible_connection: local` if running on Satellite itself)
- No custom execution environment needed

## Repository Structure

```
├── ansible.cfg
├── collections/
│   └── requirements.yml          # redhat.satellite >=5.10.0
├── inventory/
│   └── hosts.yml                 # satellite.techv.lan
├── playbooks/
│   └── configure_satellite.yml   # Main entry point
└── roles/
    └── satellite_config/
        ├── defaults/main.yml     # Connection defaults (url, user, org)
        ├── files/
        │   └── kickstart_bootc.erb   # ERB provisioning template
        ├── tasks/
        │   ├── main.yml              # Orchestrator with tags
        │   ├── organization.yml
        │   ├── locations.yml
        │   ├── lifecycle_envs.yml
        │   ├── sync_plans.yml
        │   ├── redhat_repos.yml
        │   ├── custom_repos.yml
        │   ├── sync.yml
        │   ├── content_views.yml
        │   ├── ccvs.yml
        │   ├── publish_promote.yml
        │   ├── activation_keys.yml
        │   ├── provisioning.yml
        │   └── hostgroups.yml
        └── vars/main.yml            # All component definitions
```

## Usage

### From AAP

1. Create a Project pointing to this repository
2. Create a **Red Hat Satellite 6** credential with the Satellite URL, username, and password
3. Create a Job Template using `playbooks/configure_satellite.yml`
4. Optionally limit scope with tags (see below)

### From the CLI

```bash
ansible-navigator run playbooks/configure_satellite.yml \
  --mode stdout \
  --eei registry.redhat.io/ansible-automation-platform-27/ee-supported-rhel9:latest \
  --pp never \
  -i inventory/hosts.yml \
  --extra-vars 'satellite_server_url=https://satellite.example.com satellite_username=admin satellite_password=changeme satellite_validate_certs=false' \
  --senv ANSIBLE_COLLECTIONS_PATH=/usr/share/ansible/collections
```

### Tags

Each phase can be run independently:

```
organization, locations, lifecycle_envs, sync_plans,
redhat_repos, custom_repos, repos (both repo types),
sync, content_views, ccvs, cvs (both CV types),
publish, promote, activation_keys,
provisioning, pxe, hostgroups
```

Example — only create content views and composites:

```bash
ansible-navigator run playbooks/configure_satellite.yml --tags cvs ...
```

## Customization

All Satellite component definitions live in `roles/satellite_config/vars/main.yml`. Edit this file to change:

- Repository selections and release versions
- Content view composition
- Host group hierarchy, PXE loaders, and OS assignments
- Subnet addressing and DHCP ranges
- Activation key mappings

Connection defaults (server URL, org name, domain) are in `roles/satellite_config/defaults/main.yml` and can be overridden via extra vars or AAP credential injection.

## License

[MIT](LICENSE)

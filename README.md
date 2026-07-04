# Brocade Switch Inventory

Ansible project for AAP, providing SAN fabric switch inventory via the
`brocade.fos` collection. Designed to be added as an AAP **Project**
pointed at this git repo.

## Setup

1. Install the collection: `ansible-galaxy collection install -r requirements.yml`
2. For each real switch, copy `ansible_vault_vars/ansible_vault_brocade_EXAMPLE.yml.example`
   to `ansible_vault_vars/ansible_vault_brocade_<identifier>.yml`, fill in
   real values, then encrypt it:
   ```
   ansible-vault encrypt ansible_vault_vars/ansible_vault_brocade_<identifier>.yml
   ```
   `<identifier>` is whatever string you'll pass as the `brocade_switch`
   extra_var when launching the job — it's how the playbook finds the
   right credentials without AAP extra_vars ever carrying a
   password directly.
3. In AAP, create a Job Template pointed at
   `playbooks/brocade_switch_inventory.yml`, with the Vault password
   available via an AAP Credential.
4. Launch with extra_var `brocade_switch: <identifier>`.

## Current state — diagnostic phase

The playbook currently dumps its full combined output to
`/tmp/{{ brocade_switch }}_raw_dump.json` on the AAP execution node
instead of POSTing. This is intentional — run it against a real
switch first and inspect the nameserver (`fibrechannel-name-server`)
section to confirm the actual field names FOS returns (`port-name`,
`port-index`, etc. are assumed but unverified). Once confirmed, uncomment
the final `uri` task to POST `/api/switch_facts` endpoint —
the same push pattern used by the existing host Gather Facts playbook.

## Files

- `playbooks/brocade_switch_inventory.yml` — the inventory playbook
- `ansible_vault_vars/` — per-switch encrypted credentials (never committed unencrypted)
- `requirements.yml` — the `brocade.fos` collection dependency

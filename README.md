# HiSMOG — Brocade Switch Inventory

Ansible project for AAP, providing SAN fabric switch inventory via the
`brocade.fos` collection. Designed to be added as an AAP **Project**
pointed at this git repo.

## Setup

1. Install the collection: `ansible-galaxy collection install -r requirements.yml`
2. Copy `playbooks/group_vars/all/vault.yml.example` to
   `playbooks/group_vars/all/vault.yml`, add every switch's credentials
   under `vault_brocade_switches`, keyed by identifier, then encrypt it:
   ```
   ansible-vault encrypt playbooks/group_vars/all/vault.yml
   ```
   The identifier is whatever string you'll pass as the `brocade_switch`
   extra_var when launching the job.

   Note: `group_vars/` must live directly alongside the playbook file
   (`playbooks/group_vars/`) — Ansible auto-loads it relative to the
   playbook's own directory, not the repo root.
3. In AAP, create a Job Template pointed at
   `playbooks/brocade_switch_inventory.yml`, with the Vault password
   available via an AAP Credential.
4. Launch with extra_var `brocade_switch: <identifier>` (must match a key
   under `vault_brocade_switches` in the vault).

## Current state — diagnostic phase

The playbook currently dumps its full combined output to
`/tmp/{{ brocade_switch }}_raw_dump.json` on the AAP execution node
instead of POSTing to HiSMOG. This is intentional — run it against a real
switch first and inspect the nameserver (`fibrechannel-name-server`)
section to confirm the actual field names FOS returns (`port-name`,
`port-index`, etc. are assumed but unverified). Once confirmed, uncomment
the final `uri` task to POST to HiSMOG's `/api/switch_facts` endpoint —
the same push pattern used by the existing host Gather Facts playbook.

## Files

- `playbooks/brocade_switch_inventory.yml` — the inventory playbook
- `playbooks/group_vars/all/vault.yml` — all switch credentials, one
  encrypted vault, keyed by identifier (never committed unencrypted)
- `requirements.yml` — the `brocade.fos` collection dependency

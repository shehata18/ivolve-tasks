## Day 1 — Ansible Labs

This folder contains example inventories, playbooks and roles used for four hands-on labs demonstrating basic Ansible operations: ad-hoc commands, playbooks, roles, and Vault. The README below explains the files, assumptions, and provides step-by-step commands to complete each lab.

### What is included in this folder
- `inventory` — Ansible inventory for the managed host(s).
- `webserver.yml` — Playbook to install and configure a simple Nginx web page (Lab 2).
- `playbook.yml` — MySQL setup playbook that uses `vault.yml` for sensitive variables (Lab 4).
- `site.yml` — Playbook that runs three roles: `docker`, `kubectl`, and `jenkins` (Lab 3).
- `vault.yml` — Encrypted vault file (present in repo as an example). Do not store plaintext credentials here.
- `roles/` — Directory containing three roles:
	- `roles/docker/` — Tasks to install Docker on a target (uses `yum`).
	- `roles/kubectl/` — Tasks to download `kubectl` binary and set permissions.
	- `roles/jenkins/` — Tasks to install and start Jenkins (uses `yum`).

Files were provided as examples and may need small edits depending on the managed node OS/distribution.

Assumptions
- You have a control node (your machine) and a managed node reachable by SSH.
## Day 1 — Ansible Labs (simplified)

This folder contains inventories, playbooks and roles used for four hands-on labs demonstrating basic Ansible operations: ad-hoc commands, playbooks, roles, and Vault.

Included files
- `inventory` — Managed host(s) inventory
- `webserver.yml` — Install and configure Nginx (Lab 2)
- `site.yml` — Runs roles: `docker`, `kubectl`, `jenkins` (Lab 3)
- `playbook.yml` — MySQL setup that uses `vault.yml` for DB credentials (Lab 4)
- `vault.yml` — (encrypted) holds DB user/password for Lab 4

Assumptions
- Control node (your machine) can SSH to managed node(s).
- Inventory already contains a host in group `servers` (see `inventory`).
- Some tasks in roles use `yum`; adjust package manager if using Debian/Ubuntu.

Common command flags used below
- `-i inventory` — use the repository inventory file
- `--ask-become-pass` — prompt for sudo password when required
- `--ask-vault-pass` — prompt for Ansible Vault password (used for MySQL playbook)

Lab 1 — Initial Ansible + ad-hoc check
1) Install Ansible on the control node (OS package manager or pip)
2) Create SSH key and copy to the managed node:
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_ansible -N ""
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub user@<managed-ip>
```
3) Run an ad-hoc disk check:
```bash
ansible -i inventory servers -u <user> -m command -a "df -h" --ask-become-pass
```

Lab 2 — Web server playbook (Nginx)
Run the playbook:
```bash
ansible-playbook -i inventory webserver.yml --ask-become-pass
```
Verify: SSH to the managed node and check the web server or fetch the page:
```bash
ssh user@<managed-ip>
sudo systemctl status nginx    # check service status
curl -s http://localhost        # view served page
```

Lab 3 — Roles: Docker, kubectl, Jenkins
Run the roles:
```bash
ansible-playbook -i inventory site.yml --ask-become-pass
```
Verify: SSH to the managed node and run checks for each tool:
```bash
ssh user@<managed-ip>
docker --version                # Docker version
/usr/local/bin/kubectl version --client  # kubectl version
sudo systemctl status jenkins   # Jenkins service status
```

Lab 4 — MySQL with Ansible Vault
Create or ensure `vault.yml` contains encrypted `db_user` and `db_pass` values (use `ansible-vault create` to make it).
Install required collection and run playbook:
```bash
ansible-galaxy collection install community.mysql
ansible-playbook -i inventory playbook.yml --ask-become-pass --ask-vault-pass
```
Verify: SSH to the managed node and connect using the created DB user to list databases:
```bash
ssh user@<managed-ip>
mysql -u <db_user> -p
# then: SHOW DATABASES;
```

Troubleshooting
- SSH issues: verify user, key and `~/.ssh/authorized_keys` on the managed node.
- Package/install failures: check package manager and network access on the managed node.
- Vault: use `--ask-vault-pass` or `--vault-password-file <file>` (keep that file out of version control).

If you want, I can now:
- Make playbooks cross-distro (use `package` module and `when` with `ansible_facts['os_family']`).
- Add a small `setup.sh` to automate SSH key generation/copy and optional vault creation.

Choose one and I'll implement it.
3) Verify the web page from the control node (replace IP with managed node IP if not localhost):


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
- The `inventory` file already contains a host entry. Example:
	```ini
	[servers]
	mohamed-vm-node ansible_host=192.168.77.131 ansible_user=mohamed
	```
- SSH access: control node can SSH to the managed node as the specified user.
- Packages installation requires network access to package repositories.
- Some playbooks/roles use `yum` (RHEL/CentOS) and one playbook (`webserver.yml`) uses `apt` — adjust if your managed node uses a different package manager.

General tips
- Use `-u <user>` to specify the remote user (if not present in `inventory`) and `-b` to run with become (sudo) where needed.
- If you use Ansible Vault, either use `--ask-vault-pass` or `--vault-password-file <file>` when running `ansible-playbook`.

## Lab 1: Initial Ansible Configuration and Ad-Hoc Execution
Goal: Install/configure Ansible on the control node, create an SSH key, copy it to the managed node, create an inventory and run an ad-hoc command to check disk space.

Step-by-step commands (control node)

1) Install Ansible (choose one depending on OS):

Debian/Ubuntu (apt):
```bash
sudo apt update
sudo apt install -y ansible sshpass
```

RHEL/CentOS (yum/dnf):
```bash
sudo yum install -y epel-release
sudo yum install -y ansible sshpass
```

Alternatively install latest with pip:
```bash
python3 -m pip install --user ansible
```

2) Generate a new SSH key for Ansible (optional: give it a distinct filename)
```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_ansible -N ""
```

3) Copy the public key to the managed node (replace IP/user as needed). If you used a non-default key, reference it with `-i`:
```bash
ssh-copy-id -i ~/.ssh/id_rsa_ansible.pub mohamed@192.168.77.131
```
If `ssh-copy-id` is not available, use:
```bash
ssh -i ~/.ssh/id_rsa_ansible mohamed@192.168.77.131 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys" < ~/.ssh/id_rsa_ansible.pub
```

4) Verify SSH connectivity from control node:
```bash
ssh -i ~/.ssh/id_rsa_ansible mohamed@192.168.77.131 hostname && ssh -i ~/.ssh/id_rsa_ansible mohamed@192.168.77.131 id
```

5) Confirm inventory (this repository already contains `inventory` with a `servers` group). Example content:
```ini
[servers]
mohamed-vm-node ansible_host=192.168.77.131 ansible_user=mohamed
```

6) Run an ad-hoc command to check disk space:
```bash
ansible -i inventory servers -u mohamed -m command -a "df -h" -b
```
or, if using the generated key:
```bash
ANSIBLE_PRIVATE_KEY_FILE=~/.ssh/id_rsa_ansible ansible -i inventory servers -u mohamed -m command -a "df -h" -b
```

Expected result: output showing filesystem disk usage from the managed node.

## Lab 2: Automated Web Server Configuration Using Ansible Playbooks
Goal: Use `webserver.yml` to install Nginx, deploy a simple page and verify it is served.

1) Preview the playbook: `webserver.yml` (it installs `nginx`, copies an `index.html` and verifies via `uri`).

2) Run the playbook:
```bash
ansible-playbook -i inventory webserver.yml -u mohamed -b
```

3) Verify the web page from the control node (replace IP with managed node IP if not localhost):
```bash
curl -s http://192.168.77.131 | head -n 20
```
Or run an Ansible check to request `http://localhost` on the managed node:
```bash
ansible -i inventory servers -u mohamed -b -m uri -a "url=http://localhost" -m debug
```

Notes: `webserver.yml` uses `apt` — ensure the managed node is Debian/Ubuntu. If it's RHEL/CentOS, change the package task to use `yum` and the webroot path if different.

## Lab 3: Structured Configuration Management with Ansible Roles
Goal: Use `site.yml` to execute the three roles (`docker`, `kubectl`, `jenkins`) and verify installations.

1) Inspect `site.yml` which includes:
```yaml
- name: Configure managed node with Docker, kubectl, and Jenkins
	hosts: servers
	become: yes
	roles:
		- docker
		- kubectl
		- jenkins
```

2) Run the site playbook:
```bash
ansible-playbook -i inventory site.yml -u mohamed -b
```

3) Verify each component on the managed node(s):
- Docker:
	```bash
	ansible -i inventory servers -u mohamed -b -m command -a "docker --version"
	```
- kubectl:
	```bash
	ansible -i inventory servers -u mohamed -b -m command -a "/usr/local/bin/kubectl version --client"
	```
- Jenkins:
	```bash
	ansible -i inventory servers -u mohamed -b -m systemd -a "name=jenkins state=started"
	# or check port
	ansible -i inventory servers -u mohamed -b -m command -a "ss -tunlp | grep 8080 || true"
	```

Notes: the included `roles/docker` and `roles/jenkins` tasks use `yum`. If your managed node is Debian/Ubuntu, change package management accordingly or run only the roles that match the OS.

## Lab 4: Securing Sensitive Data with Ansible Vault
Goal: Install MySQL, create a database `iVolve`, create a DB user and secure the DB password using Ansible Vault. Validate by connecting with the created user.

1) Create an encrypted Vault file to hold `db_user` and `db_pass` (if you don't want to use the provided `vault.yml`):
```bash
ansible-vault create vault.yml
# Inside the editor, add something like:
# db_user: ivolve_user
# db_pass: verysecurepassword
```

Alternatively create a plain file and encrypt it:
```bash
cat > vault_plain.yml <<'YAML'
db_user: ivolve_user
db_pass: verysecurepassword
YAML
ansible-vault encrypt --output vault.yml vault_plain.yml
shred -u vault_plain.yml
```

2) Run the MySQL playbook that references `vault.yml` (the provided `playbook.yml` uses the `community.mysql` modules):
```bash
ansible-galaxy collection install community.mysql
ansible-playbook -i inventory playbook.yml -u mohamed -b --ask-vault-pass
```
If you created a `--vault-password-file` instead:
```bash
ansible-playbook -i inventory playbook.yml -u mohamed -b --vault-password-file .vault_pass.txt
```

3) Validate the database from the managed node (check DB list using the new user):
```bash
ansible -i inventory servers -u mohamed -b -m shell -a "mysql -u ivolve_user -p'<<PASSWORD>>' -e 'SHOW DATABASES;'" \
	# Replace <<PASSWORD>> with the vault password or use vault to provide the command safely
```
Or get an interactive shell on the managed node and test from there:
```bash
ssh -i ~/.ssh/id_rsa_ansible mohamed@192.168.77.131
mysql -u ivolve_user -p
# then: SHOW DATABASES;
```

Notes:
- `playbook.yml` installs `python3-PyMySQL` (RHEL package) — ensure the managed node has the MySQL client packages and Python MySQL bindings the `community.mysql` modules require.
- If `community.mysql` modules are missing, run `ansible-galaxy collection install community.mysql` on the control node first.

## Troubleshooting
- If SSH fails, verify the right user, key and that `~/.ssh/authorized_keys` contains the public key.
- If package tasks fail, check the managed node's package manager and repo access.
- When using Vault, `--ask-vault-pass` will prompt for the password; `--vault-password-file` reads it from a file (protect this file with proper permissions).

## Quick checklist mapping to requested deliverables
- Lab 1 (Ansible + SSH + ad-hoc): explained and commands provided — Done
- Lab 2 (Nginx playbook): `webserver.yml` run and verification commands — Done
- Lab 3 (Roles + site.yml): instructions to run `site.yml` plus verification commands — Done
- Lab 4 (MySQL + Vault): how to create vault, run `playbook.yml`, and validate DB access — Done

Added helper files
------------------
I created a few small helper files in this folder to make running the labs easier:

- `ansible.cfg` — sets defaults for this Day 1 lab (inventory path, default private key, disables host key checking for labs, and sets the `roles_path`). This allows you to run `ansible` and `ansible-playbook` without specifying `-i inventory` every time.

- `.vault_pass.txt.example` — a template showing how to store a vault password in a file. Do NOT commit a real `.vault_pass.txt`; copy this file to `.vault_pass.txt`, replace the placeholder password, and set strict permissions (`chmod 600 .vault_pass.txt`).

- `.gitignore` — ignores `.vault_pass.txt` and other common local artifacts so secrets aren't committed by accident.

Using the helper files (examples)
--------------------------------
With `ansible.cfg` present, many commands can be simplified. Examples below assume you either set your private key to `~/.ssh/id_rsa_ansible` or adjust `ansible.cfg` accordingly.

Run the webserver playbook (Lab 2):
```bash
ansible-playbook webserver.yml -u mohamed -b
```

Run all roles (Lab 3):
```bash
ansible-playbook site.yml -u mohamed -b
```

Run the MySQL playbook using a vault password file (Lab 4):
```bash
# copy the example and edit the password, then secure the file
cp .vault_pass.txt.example .vault_pass.txt
# edit .vault_pass.txt to set a strong password, then:
chmod 600 .vault_pass.txt

ansible-playbook playbook.yml -u mohamed -b --vault-password-file .vault_pass.txt
```

If you prefer not to use a password file, use `--ask-vault-pass` instead.

Next steps I can take for you
----------------------------
- Make the playbooks cross-distro by adding `when` checks and using the `package` module where possible.
- Convert role package installation tasks to support both Debian and RedHat families (small edits to each role).
- Add a small `setup` script that creates the SSH key, copies it to the managed node (using `ssh-copy-id` or `ssh`), and optionally creates an encrypted `vault.yml` from plaintext.

Pick any of the above and I'll implement it.


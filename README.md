# Ansible Monthly Server Patching (Ubuntu and Amazon Linux)

Patches Ubuntu and Amazon Linux 2/2023 servers in 5 steps: pre-check, patch, reboot (only if needed), post-check, report.

## Files

| File | Purpose |
|------|---------|
| `inventory` | List of servers (group `servers`) |
| `monthly-patching.yml` | The playbook |
| `group_vars/all.yml` | Settings: SSH user, service name, disk limit, reboot timeout |

## Before you start

1. Install Ansible on your control machine: `sudo apt install ansible`
2. Edit `inventory` with your real server names or IPs.
3. Edit `group_vars/all.yml` (`ansible_user`, `important_service`). The defaults are `ec2-user` and `sshd` for Amazon Linux. For Ubuntu, set these to `ubuntu` and `ssh`.
4. In WSL, ensure the configured private key exists at `/mnt/c/Users/Acer Nitro/Downloads/rituraj(1).pem`. SSH key login must work, and the user needs sudo: `ansible servers -i inventory -m ping`

## Run

    ansible-playbook -i inventory monthly-patching.yml --syntax-check   # check syntax
    ansible-playbook -i inventory monthly-patching.yml --check          # dry run
    ansible-playbook -i inventory monthly-patching.yml                  # real run
    ansible-playbook -i inventory monthly-patching.yml --limit server1 # one server only

For passwordless SSH, install the matching **public** key in `~ec2-user/.ssh/authorized_keys` on each server, and keep the **private** key on the Ansible control machine (or load it into `ssh-agent`). Never share or copy the private key to the servers. Verify access with `ansible servers -i inventory -m ping`. If WSL/OpenSSH rejects permissions on the Windows-mounted key, copy it into WSL's `~/.ssh` directory, run `chmod 600` on that copy, and update `ansible_ssh_private_key_file` accordingly. If sudo asks for a password, add `--ask-become-pass`.

## What each step does

1. **Pre-check**: ping, disk usage (stops if `/` is 90% or more full), kernel version, service status.
2. **Patch**: Ubuntu uses APT; Amazon Linux 2 uses Yum and Amazon Linux 2023 uses DNF.
3. **Reboot**: Ubuntu reboots when `/var/run/reboot-required` exists. Amazon Linux reboots after successful package updates. Waits for the server to return.
4. **Post-check**: ping, service running, kernel version, disk usage.
5. **Report**: example output:

        Server: server01
        Patching: SUCCESS
        Reboot: YES
        Health Check: PASS

## Settings (`group_vars/all.yml`)

| Variable | Meaning |
|----------|---------|
| `ansible_user` | SSH user (default `ec2-user`) |
| `important_service` | Service checked before and after (default `sshd`) |
| `disk_limit_percent` | Max allowed disk usage (default 90) |
| `reboot_timeout` | Seconds to wait after reboot (default 600) |

## Monthly schedule (optional, cron)

    0 2 1 * * cd /home/ec2-user/ansible-monthly-patching && ansible-playbook -i inventory monthly-patching.yml >> patching.log 2>&1
# monthly-patch-ansible
# monthly-patch-ansible
# monthly-patch-ansible

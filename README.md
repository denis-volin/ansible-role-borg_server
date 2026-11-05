# Ansible Role: borg_server

Configure a Borg backup server with SSH access restricted to each client's repository directory.

The role installs a standalone Borg binary, creates the `borg` user and repository directories, and enables a daily backup-age check.
It does not initialize repositories or create backups.

## Requirements

- Debian 12 (bookworm) or 13 (trixie), with systemd.
- Ansible 2.15 or later and root privileges through `become`.
- The `ansible.posix` collection: `ansible-galaxy collection install ansible.posix`.
- Linux x86_64 with glibc 2.35 or later for the default binary.

## Example playbook

```yaml
- name: Configure Borg server
  hosts: backup_servers
  become: true
  roles:
    - role: denis-volin.borg_server
      vars:
        borg_server_repos:
          - host: web.example.com
            key: "ssh-ed25519 AAAA... root@web.example.com"
        borg_server_encryption_passphrase: "{{ vault_borg_passphrase }}"
```

Each entry creates `/srv/borg/repos/<host>` and adds the SSH key with a restricted `borg serve` command.
Initialize the repository from the client before the first backup.
Use `/srv/borg/repos/<host>/repo` as the repository path for the backup-age check.
Store passphrases with Ansible Vault.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `borg_server_version` | `1.4.5` | Borg release to install. |
| `borg_server_binary_url` | GitHub release URL | Standalone binary URL derived from the version. |
| `borg_server_home` | `/srv/borg` | Home directory for the `borg` user. |
| `borg_server_repos` | `[]` | Client entries with `host` and `key` fields. |
| `borg_server_check_path` | `/usr/local/bin/borg-check` | Backup-age check script path. |
| `borg_server_check_timer` | `*-*-* 12:00:00` | Systemd schedule, daily at noon. |
| `borg_server_encryption_passphrase` | `""` | Passphrase used by the backup-age check. |

## Backup-age check

The `borg-check.timer` runs the check daily at noon by default.
The script checks `<borg_server_home>/repos/<host>/repo` and reports backups older than 48 hours.
It does not check repository integrity.

## License

MIT

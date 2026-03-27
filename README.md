# role_mariadb

Ansible role to install, configure, and manage MariaDB server. Supports fresh installation with database/user creation and restoring existing databases from backup archives.

## Requirements

- Ansible >= 2.9
- `community.mysql` collection >= 3.12.0

## Supported Platforms

- AlmaLinux 9, 10
- RockyLinux 9, 10
- Debian 11 (Bullseye), 12 (Bookworm)
- Ubuntu 22.04 (Jammy), 24.04 (Noble)

## Role Variables

### MariaDB Version and Service

| Variable | Default | Description |
|---|---|---|
| `mariadb_version` | `"11.4"` | MariaDB version to install |
| `mariadb_service_name` | `mariadb` | Systemd service name |
| `mariadb_service_enabled` | `true` | Enable service on boot |

### Configuration

| Variable | Default | Description |
|---|---|---|
| `mariadb_port` | `3306` | Listen port |
| `mariadb_bind_address` | `127.0.0.1` | Bind address |
| `mariadb_datadir` | `/var/lib/mysql` | Data directory |
| `mariadb_max_connections` | `150` | Max connections |
| `mariadb_innodb_buffer_pool_size` | `256M` | InnoDB buffer pool |
| `mariadb_character_set_server` | `utf8mb4` | Default charset |
| `mariadb_collation_server` | `utf8mb4_unicode_520_ci` | Default collation |

### Credentials

| Variable | Default | Description |
|---|---|---|
| `mariadb_root_password` | `changeme` | Root password (use vault!) |

### Databases and Users

```yaml
mariadb_databases:
  - name: wordpress_db
    encoding: utf8mb4
    collation: utf8mb4_unicode_520_ci

mariadb_users:
  - name: wp_user
    password: "securepassword"
    host: "localhost"
    priv: "wordpress_db.*:ALL"
```

### Restore from Backup

| Variable | Default | Description |
|---|---|---|
| `mariadb_restore_enabled` | `false` | Enable database restore |
| `mariadb_restore_source` | `""` | Path to backup file (.sql, .sql.gz, .tar.gz, .zip) |
| `mariadb_restore_database` | `""` | Target database name for restore |

The restore is idempotent — it only imports if the target database has no tables.

### Credential Saving

| Variable | Default | Description |
|---|---|---|
| `mariadb_credentials_save_enabled` | `true` | Save credentials to controller |
| `mariadb_credentials_save_path` | `/opt/ansible/credentials/mariadb` | Path on Ansible controller |

## Example Playbook

```yaml
- hosts: database
  become: true
  roles:
    - role: role_mariadb
      vars:
        mariadb_root_password: "{{ vault_mariadb_root_password }}"
        mariadb_databases:
          - name: myapp_db
        mariadb_users:
          - name: myapp_user
            password: "{{ vault_mariadb_myapp_password }}"
            host: "localhost"
            priv: "myapp_db.*:ALL"
```

### Restore Example

```yaml
- hosts: database
  become: true
  roles:
    - role: role_mariadb
      vars:
        mariadb_root_password: "{{ vault_mariadb_root_password }}"
        mariadb_databases:
          - name: wordpress_db
        mariadb_restore_enabled: true
        mariadb_restore_source: "backup/dymer.com.tr/db/database-1774563723.tar.gz"
        mariadb_restore_database: wordpress_db
```

## License

GPL-2.0-or-later

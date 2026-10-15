postgres
========

Роль устанавливает PostgreSQL из стандартных репозиториев Ubuntu, инициализирует кластер в своём каталоге данных, настраивает `postgresql.conf` и `pg_hba.conf`, запускает СУБД как systemd-службу, создаёт пользователей и базы, ставит postgres-exporter.

Requirements
------------

- Коллекция `community.postgresql` на контроллере (`ansible-galaxy collection install -r requirements.yml`).
- Пароли передаются через `ansible-vault`.

Role Variables
--------------

| Переменная | По умолчанию | Описание |
|---|---|---|
| `postgres__data_dir` | `/data/postgres/data` | каталог данных (`$PGDATA`) |
| `postgres__opts` | `listen_addresses`, `port`, `max_connections`, `shared_buffers` | словарь параметров `postgresql.conf`; строковые значения в одинарных кавычках |
| `postgres__hba` | local peer, host scram-sha-256 для 127.0.0.1 и ::1 | список правил `pg_hba.conf`: `type`, `database`, `user`, `address` (необязательно), `method` |
| `postgres__users` | `[]` | список пользователей: `name`, `password`, `role_attr_flags`, `state` |
| `postgres__databases` | `[]` | список баз: `name`, `owner`, `encoding`, `state` |
| `postgres__exporter_version` | `0.20.1` | версия postgres-exporter |
| `postgres__exporter_port` | `9187` | порт метрик экспортера |
| `postgres__exporter_user` | `postgres_exporter` | пользователь БД для экспортера (получает `pg_monitor`) |
| `postgres__exporter_password` | — | **обязательная**, пароль пользователя экспортера |

Внутренние переменные (`vars/main.yml`): `__pg_version`, `__pg_bin_dir`, `__root_dir`, `__service_name`, `__exporter_dir`, `__exporter_url`.

Службы: `postgres-server.service`, `postgres-exporter.service`.

Example Playbook
----------------

```yaml
- name: Install PostgreSQL on vm1
  hosts: vm1
  roles:
    - postgres
```

Переменные инвентаря (`group_vars/all/`):

```yaml
postgres__users:
  - name: appuser
    password: "{{ vault_appuser_password }}"

postgres__databases:
  - name: appdb
    owner: appuser

postgres__exporter_password: "{{ vault_exporter_password }}"
```

Секреты шифруются так:

```bash
ansible-vault encrypt_string 'S3cr3t' --name 'vault_appuser_password'
```
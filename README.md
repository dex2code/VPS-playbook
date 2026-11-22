# VPS-playbook

Ansible-плейбуки для обслуживания парка VPS под **Debian**.

## Плейбуки

| Файл | Назначение |
|---|---|
| `playbooks/ping.yml` | Диагностика: связь, ОС/ядро, hostname, uptime. |
| `playbooks/apt-upgrade.yml` | Плановое обслуживание: `apt` update + dist-upgrade, очистка, перезагрузка. Хосты обрабатываются по одному (`serial: 1`). |
| `playbooks/xray-upgrade.yml` | Обновление xray-core: последний релиз с GitHub (SHA256), замена файлов, config-тест, перезапуск сервиса. При ошибке — откат к `*.old` (только если бэкапы созданы в этом же прогоне). |

## Требования

- Python 3 с `venv`
- SSH-доступ до хостов по ключу

## Настройка

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp inventory/hosts.ini.example inventory/hosts.ini   # затем заполнить реальные хосты

pip install pre-commit && pre-commit install          # опционально: ansible-lint на каждый коммит
```

`inventory/hosts.ini` не коммитится (в `.gitignore`) — там реальные адреса и пути к ключам.
Ролевые группы в инвентори — с суффиксом `_grp` (`[xray_grp]`); встроенная группа `all` (в ней определяются все хосты) и `ungrouped` зарезервированы и суффикс не получают.
Линт также гоняется в CI (`.github/workflows/lint.yml`) на каждый push/PR.

## Запуск

```bash
source .venv/bin/activate                            # обязательно перед любой командой
ansible-playbook playbooks/ping.yml                  # диагностика всех хостов
ansible-playbook playbooks/apt-upgrade.yml           # обновление + reboot, по одному хосту
ansible-playbook playbooks/xray-upgrade.yml          # обновление xray-core (группа [xray_grp])
ansible-playbook playbooks/ping.yml --limit vortex   # один хост
```

`apt-upgrade.yml` перезагружает **каждый** хост при каждом запуске — это намеренное поведение.

Правила работы для агентов — в [AGENTS.md](AGENTS.md).

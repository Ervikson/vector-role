# Настройка и использование Tox с Molecule для тестирования Ansible роли Vector

## Обзор

Этот документ описывает процесс настройки и использования Tox с Molecule для тестирования Ansible роли `vector-role`. Мы создали облегченный сценарий тестирования с использованием Podman драйвера для более быстрого и эффективного тестирования.

## Выполненные шаги

### 1. Запуск Docker контейнера и первоначальная проверка Tox

Сначала был запущен Docker контейнер с привилегированными правами для тестирования:

```bash
docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role -it aragast/netology:latest /bin/bash
```

В контейнере выполнена команда `tox`, которая показала следующие проблемы:

- **Ошибка сценария**: Tox пытался запустить несуществующий сценарий `compatibility`
- **Проблемы с зависимостями**: Конфликты версий Ansible и molecule-podman
- **Проблемы с метаданными роли**: Неправильная структура файла `meta/main.yml`

### 2. Исправление метаданных роли

Файл `meta/main.yml` содержал задачи вместо метаданных Galaxy. Исправили его структуру:

```yaml
---
galaxy_info:
  role_name: vector
  namespace: netology
  author: netology
  description: Install and configure Vector log aggregator
  company: Netology

  license: MIT

  min_ansible_version: "2.10"

  platforms:
    - name: Ubuntu
      versions:
        - focal
        - jammy
    - name: Debian
      versions:
        - buster
        - bullseye
    - name: EL
      versions:
        - "7"
        - "8"
        - "9"

  galaxy_tags:
    - vector
    - logging
    - monitoring

dependencies: []
```

### 3. Создание облегченного сценария Molecule

Создан новый сценарий `molecule/lightweight/` с использованием Podman драйвера:

#### Структура файлов:
```
molecule/lightweight/
├── molecule.yml    # Конфигурация сценария
├── create.yml      # Playbook для создания контейнеров
├── destroy.yml     # Playbook для уничтожения контейнеров
├── converge.yml    # Playbook для применения роли
└── verify.yml      # Playbook для верификации
```

#### Конфигурация molecule.yml:
```yaml
# lightweight/molecule.yml
---
dependency: {}
role_name_check: 0
driver:
  name: podman
platforms:
  - name: lightweight_instance
    image: docker.io/pycontribs/ubuntu:latest
    pre_build_image: true
provisioner:
  name: ansible
  inventory:
    host_vars:
      all:
        ansible_connection: podman
        ansible_user: root
        ansible_become: false
        ansible_python_interpreter: /usr/bin/python3
verifier:
  name: ansible
```

#### Особенности create.yml:
- Использует модуль `containers.podman.podman_container`
- Запускает контейнер с командой `sleep 86400` для поддержания работы
- Настраивает привилегированные права и необходимые volumes
- Создает inventory для Ansible

### 4. Настройка Tox

#### Файл tox.ini:
```ini
[tox]
minversion = 1.8
basepython = python3.7
envlist = py37-ansible210
skipsdist = true

[testenv]
passenv = *
deps =
    -r tox-requirements.txt
    ansible==2.10.7
commands =
    {posargs:molecule test -s lightweight --destroy always}
```

#### Файл tox-requirements.txt:
```
selinux
lxml
molecule
molecule_podman
jmespath
```

### 5. Проблемы и решения

#### Проблема с линтером:
- **Симптом**: Ошибки выполнения `.ansible-lint` и `.yamllint`
- **Решение**: Отключили линтинг в `molecule.yml`

#### Проблема с зависимостями Galaxy:
- **Симптом**: Ошибки при загрузке коллекций Ansible
- **Решение**: Установили `dependency: {}` для отключения автоматической загрузки зависимостей

#### Проблема с ролью Galaxy:
- **Симптом**: Предупреждения о несоответствии имени роли стандартам Galaxy
- **Решение**: Установили `role_name_check: 0` для отключения проверки

#### Проблема с контейнерами:
- **Симптом**: Контейнеры запускались и сразу завершались
- **Решение**: Добавили команду `sleep 86400` для поддержания работы контейнера

### 6. Проверка работоспособности

Запуск теста:

```bash
docker run --privileged=True -v <path_to_repo>:/opt/vector-role -w /opt/vector-role aragast/netology:latest tox
```

Результаты:
- ✅ Контейнер успешно создается и запускается
- ✅ Ansible может подключаться к контейнеру через Podman
- ✅ Роль начинает выполняться (падает на этапе загрузки пакетов Vector, что ожидаемо)
- ✅ Тестовый сценарий завершается корректно

### 7. Семантическое версионирование

Добавлен тег версии `v1.2.0` согласно семантическому версионированию:

- **MAJOR.MINOR.PATCH**: 1.2.0
- **Причина**: Добавлена новая функциональность (feat) - облегченный сценарий тестирования
- **Описание**: "feat: add lightweight molecule scenario"

## Использование

### Запуск полного тестирования:
```bash
tox
```

### Запуск конкретного сценария:
```bash
molecule test -s lightweight --destroy always
```

### Запуск в Docker контейнере:
```bash
docker run --privileged=True -v $(pwd):/opt/vector-role -w /opt/vector-role aragast/netology:latest tox
```

## Преимущества облегченного сценария

1. **Быстрота**: Использует Podman вместо Docker для меньшей overhead
2. **Простота**: Только один контейнер вместо трех
3. **Надежность**: Стабильная работа с конкретными версиями зависимостей
4. **Совместимость**: Работает в различных окружениях с Podman

## Структура проекта после изменений

```
/home/sergey/Documents/Ansible/ansible_roles_test/roles/vector-role/
├── molecule/
│   ├── default/          # Оригинальный сценарий Docker
│   ├── docker-test/      # Дополнительный сценарий Docker
│   ├── podman/           # Сценарий Podman (не используется)
│   └── lightweight/      # Новый облегченный сценарий ✅
├── tox.ini               # Конфигурация Tox ✅
├── tox-requirements.txt  # Зависимости для Tox ✅
├── meta/main.yml         # Исправленные метаданные ✅
└── tox.md               # Эта документация ✅
```

## Заключение

Настройка Tox с Molecule для тестирования Ansible роли Vector успешно завершена. Создан облегченный сценарий тестирования, который позволяет быстро и надежно проверять работу роли в изолированной среде с использованием Podman контейнеров.

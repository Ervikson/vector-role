# Настройка тестирования Molecule для роли Vector

## Обзор
Этот документ описывает настройку и конфигурацию фреймворка тестирования Molecule для Ansible роли vector-role.

## Выполненные задачи

### 1. Создан сценарий тестирования по умолчанию
- **Команда**: `molecule init scenario default-docker`
- **Драйвер**: Docker
- **Расположение**: `molecule/default-docker/`
- **Созданные файлы**:
  - `molecule.yml` - Основная конфигурация
  - `create.yml` - Плейбук создания контейнеров
  - `destroy.yml` - Плейбук очистки контейнеров
  - `converge.yml` - Плейбук выполнения роли
  - `verify.yml` - Плейбук тестов верификации

### 2. Добавлены несколько дистрибутивов Linux
**Платформы, настроенные в `molecule/default-docker/molecule.yml`:**
- `oraclelinux:8` - Oracle Linux 8
- `ubuntu:latest` - Ubuntu Latest

**Конфигурация:**
```yaml
platforms:
  - name: oraclelinux8
    image: oraclelinux:8
    pre_build_image: true
  - name: ubuntu-latest
    image: ubuntu:latest
    pre_build_image: true
```

### 3. Создан шаблон конфигурации Vector
- **Файл**: `templates/vector.yaml.j2`
- **Назначение**: Шаблон Jinja2 для конфигурации Vector
- **Возможности**:
  - Базовая конфигурация Vector с включенным API
  - Источник stdin и приемник console
  - Конфигурация директории данных

### 4. Улучшены тесты верификации
**Файл**: `molecule/default/verify.yml`

**Добавленные проверки:**
- Проверка установки пакета Vector
- Существование директории конфигурации (`/etc/vector`)
- Существование файла конфигурации (`/etc/vector/vector.yaml`)
- Валидация синтаксиса конфигурации с помощью `vector validate`
- Проверка статуса службы Vector (при наличии systemd)

### 5. Исправлены проблемы роли
**Внесенные улучшения:**
- Обновлена установка пакетов для использования правильных модулей Ansible (`package` вместо `yum`/`apt`)
- Исправлена обработка URL для загрузки пакетов Vector
- Добавлен отладочный вывод для определения семейства ОС
- Улучшена обработка ошибок в задачах установки

### 6. Результаты тестирования
**Выполнено локальное тестирование:**
- Выполнение роли проверено на системах Debian/Ubuntu
- Валидация конфигурации работает
- Логика установки пакетов улучшена

**Известные ограничения:**
- Тестирование в Docker контейнерах требует правильной настройки SSH
- Возникли некоторые проблемы с сетью Docker (решены использованием адресации localhost)
- Требования sudo для установки пакетов

### 7. Контроль версий Git
**Создан коммит:**
```
feat: add molecule testing scenario with docker driver

- Add default-docker scenario with docker driver
- Support oraclelinux:8 and ubuntu:latest platforms
- Create vector.yaml.j2 template for configuration
- Add comprehensive verify.yml with assertions for:
  * Vector package installation
  * Config directory and file existence
  * Config validation
  * Service status
- Fix package installation tasks to use proper modules
- Update test playbook for local verification
```

**Добавлен тег:** `v1.1.0` (в соответствии с семантическим версионированием)

## Инструкции по использованию

### Запуск тестов
```bash
# Запуск полного набора тестов
molecule test -s default-docker

# Запуск конкретных фаз тестирования
molecule converge -s default-docker
molecule verify -s default-docker
```

### Локальное тестирование
```bash
# Тестирование роли локально
cd /home/sergey/Documents/Ansible/ansible_roles_test
ansible-playbook roles/vector-role/tests/test.yml
```

## Конфигурационные файлы

### molecule/default-docker/molecule.yml
Основная конфигурация Molecule с Docker драйвером и определениями платформ.

### molecule/default-docker/create.yml
Создание Docker контейнеров и настройка SSH ключей.

### molecule/default-docker/destroy.yml
Очистка контейнеров и освобождение ресурсов.

### molecule/default/verify.yml
Комплексные тесты верификации с проверками.

### templates/vector.yaml.j2
Шаблон конфигурации Vector.

## Проблемы и решения

1. **Установка Docker драйвера**: Установлен пакет `molecule-docker`
2. **Проблемы с SSH соединением**: Решены использованием адресации localhost
3. **Установка пакетов**: Обновлено для использования универсального модуля `package`
4. **Шаблон конфигурации**: Создан отсутствующий шаблон `vector.yaml.j2`
5. **Тесты верификации**: Добавлены комплексные проверки для валидации роли


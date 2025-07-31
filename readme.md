# Timeshift-AutoBackup-systemd: Automated Linux Snapshots

![Systemd](https://img.shields.io/badge/systemd-%23E44D30.svg?logo=systemd&logoColor=white)
![Timeshift](https://img.shields.io/badge/Timeshift-1793D1?logo=timeshift&logoColor=white)
![Shell](https://img.shields.io/badge/Shell-Script-%234EAA25)

## 📌 Описание

Автоматизированное создание снимков системы через **Timeshift** с использованием **systemd timers**. Проект предоставляет:

✔ Готовые конфигурации systemd (сервис + таймер)  
✔ Оптимизированные исключения (кэши, временные файлы, Docker)
✔ Сохранение снимков на другие разделы дисков
✔ Гибкое расписание (ежедневно/еженедельно/ежемесячно) снимков
✔ Интеграция с systemd-journald для централизованного логирования  


## ⚙️ Установка и настройка

### Требования
- Установленный Timeshift (`sudo apt install timeshift`)
- Systemd (включен по умолчанию в современных дистрибутивах)

### Настройка
1. Клонируйте репозиторий в домашнюю директорию: 
```bash
git clone https://github.com/yourusername/timeshift-autobackup-systemd.git
cd timeshift-autobackup-systemd
```

2. Настройте целевой диск для бэкапов:
> **Важно**: Перед использованием замените UUID в строке backup_device_uuid в `timeshift.json` на актуальные значения (`sudo blkid`)
```bash
sudo blkid  # Скопируйте UUID нужного раздела
vim etc/timeshift/timeshift.json
```

3. Установите сервис:
```bash
sudo cp -v ~/timeshift-autobackup-systemd/timeshift.* /etc/systemd/system/
sudo cp -v ~/timeshift-autobackup-systemd/timeshift.json /etc/timeshift/
```

- `etc/systemd/system/timeshift.service` - unit для создания снимков
- `etc/systemd/system/timeshift.timer` - unit расписание (по умолчанию: ежедневно в 18:00)
- `etc/timeshift/timeshift.json` - настройки Timeshift (опционально)

Пример изменения расписания:
```ini
# В timeshift.timer
OnCalendar=*-*-* 04:00:00  # Ежедневно в 4:00
```

4. Верификация установки:
```bash
systemd-analyze verify /etc/systemd/system/timeshift.*
```
Если ошибок нет, команда не выведет ничего или выведет только предупреждения.

5. Перезагрузка конфигурации systemd для unit-файлов (.service или .timer):
```bash
sudo systemctl daemon-reload
```
> **Важно**: Если вы изменяли unit-файлы, не забудьте перезагрузить systemd.

6. Запуск unit-файлов, настраиваем автозапуск при загрузке системы:
```bash
sudo systemctl start timeshift.timer
sudo systemctl enable timeshift.timer
```

7. Проверяем что фоновый процесс запустился:
```bash
systemctl status timeshift.timer
```
Должно показать в строке Loaded: enabled, в Active: active (waiting)

## 🔄 Управление снимками

### Ручное создание снимка
```bash
sudo timeshift --create --comments "Ручной снимок"
```
После создания появится директория где хранятся бэкапы на указанном диске: /timeshift. 
Если папки нет, TimeShift создаст её при первом бэкапе.
в директории: /timeshift/snapshots/2025-07-28_17-56-16/exclude.list
содержится файл "exclude.list" где перечисляется какие разделы не включены в снимок.


### Просмотр существующих снимков
```bash
sudo timeshift --list
# Пример вывода:
# Device: /dev/sdb1 (ext4, 500GB)
# Snapshot | Tag    | Date                | Size
# -------- | ------ | ------------------- | -----
# 1        | D      | 2025-08-01 18:00:01 | 15GB
```

### Очистка старых снимков
```bash
# Удалить конкретный снимок
sudo timeshift --delete --snapshot '2025-07-30_14-34-03'

# Автоматическая очистка (по политике хранения)
sudo timeshift --cleanup
```


## 📊 Мониторинг

Проверка состояния:
Логи только таймера (запуски по расписанию)
```bash
sudo journalctl -u timeshift.timer
```

Логи только сервиса (процесс создания бэкапа)
```bash
sudo journalctl -u timeshift.service
```

## 📂 Структура проекта
```
.
├── etc/
│ ├── systemd/
│ │ └── system/
│ │ ├── timeshift.service
│ │ └── timeshift.timer
│ └── timeshift/
│ └── timeshift.json
├── timeshift/snapshots/ # Автоматически создаваемое хранилище снимков на диске
└── readme.md
```

## Контакты
Если у вас есть вопросы или предложения, пожалуйста, свяжитесь со мной по адресу: rusov63@yahoo.com


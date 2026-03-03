# Anno OCR

<p align="center">
  Инструмент для разметки и распознавания текста на изображениях рукописей
</p>

<p align="center">
  <img alt="Python 3.10+" src="https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white">
  <img alt="PySide6 Qt6" src="https://img.shields.io/badge/PySide6-Qt%206-41CD52?logo=qt&logoColor=white">
  <img alt="ONNX Runtime" src="https://img.shields.io/badge/OCR-ONNX_Runtime-005CED?logo=onnx&logoColor=white">
  <img alt="License MIT" src="https://img.shields.io/badge/License-MIT-F4C430">
</p>

Anno OCR - десктопное приложение для аннотаторов и исследователей, работающих с рукописными и печатными документами. Приложение позволяет размечать слова, строки и блоки текста, запускать OCR и экспортировать результаты в стандартных форматах.

## Содержание

- [Возможности](#возможности)
- [Скриншоты](#скриншоты)
- [Быстрый старт](#быстрый-старт)
- [Конфигурация](#конфигурация)
- [Горячие клавиши](#горячие-клавиши)
- [Сборка](#сборка)
- [Архитектура](#архитектура)
- [Технологии и лицензии зависимостей](#технологии-и-лицензии-зависимостей)
- [Лицензия](#лицензия)
- [Контрибуции](#контрибуции)

## Возможности

### Разметка

- Открытие папки с изображениями: `PNG`, `JPG`, `BMP`, `TIFF`, `WebP`
- Создание, перемещение и изменение размера боксов (слово = один бокс)
- Группировка слов в строки и строк в блоки
- Drag and drop реорганизация структуры
- Вырезать/вставить слов и строк: `Ctrl+X` / `Ctrl+V`
- Множественное выделение: `Ctrl+клик`, рамка, кисть
- Сортировка по строке, блоку и всей странице
- Очистка пустых и лишних объектов

### OCR

- API-режим: подключение к серверу Manuscript-OCR (`detect -> recognize -> correct -> organize`)
- Локальный режим: ONNX-модели (EAST + TRBA + CharLM), работает без интернета
- Пакетная обработка с индикатором прогресса
- Отмена длительных операций

### Импорт и экспорт

- PDF: импорт страниц и экспорт размеченных результатов
- JSON: экспорт/импорт датасета в собственном формате
- COCO JSON: экспорт/импорт для совместимости с внешними инструментами

### Интерфейс

- Автоопределение светлой/темной темы
- Масштабирование и панорамирование изображения
- Превью распознанного текста
- Клавиатурная навигация по страницам
- Индикатор квоты API

## Скриншоты

Добавьте изображения в папку `screenshots/` и обновите пути:

```md
![Главное окно - светлая тема](screenshots/light.png)
![Главное окно - темная тема](screenshots/dark.png)
```

## Быстрый старт

### Запуск из исходников

```bash
git clone https://github.com/<your-username>/anno-ocr.git
cd anno-ocr

python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
python run.py
```

### Готовые сборки

Скачайте `OCR_Tool.exe` (Windows), `OCR_Tool_Linux` или `OCR_Tool_macOS` из раздела `Releases`:

`https://github.com/<your-username>/anno-ocr/releases`

## Конфигурация

При первом запуске рядом с программой создается `ocr_tool_config.json`:

```json
{
  "api_base_url": "http://localhost:8000/api/v1",
  "api_key": "",
  "ocr_mode": "local"
}
```

| Параметр | Описание |
| --- | --- |
| `api_base_url` | Адрес сервера Manuscript-OCR |
| `api_key` | Ключ авторизации (`X-API-Key`) |
| `ocr_mode` | `"local"` - локальные модели, `"api"` - серверный режим |

Настройки также доступны в меню `Файл -> Настройки`.

## Горячие клавиши

| Клавиша | Действие |
| --- | --- |
| `Ctrl+O` | Открыть папку |
| `Left / Right` | Предыдущее / следующее изображение |
| `Page Up / Page Down` | Навигация по страницам |
| `Ctrl+G` | Перейти к странице |
| `Delete` | Удалить выделенные боксы |
| `Ctrl+A` | Выделить все боксы |
| `Ctrl+X` / `Ctrl+V` | Вырезать / вставить слова и строки |
| `Ctrl+C` | Копировать текст |
| `+` / `-` | Масштаб |
| `Ctrl+0` | Сбросить масштаб |
| `E` | Редактировать текст бокса |
| `R` | Поворот изображения на 90 градусов |

## Сборка

### Windows

```bash
build.bat
# Результат: dist/OCR_Tool.exe
```

### Linux / macOS

```bash
bash build_linux.sh
# или
bash build_mac.sh
```

GitHub Actions-сборки для всех платформ находятся в `.github/workflows/build.yml` и запускаются по тегу `v*` или вручную.

## Архитектура

```text
ocr_tool/
├── app.py              # Главное окно, меню, клавиатура
├── controllers/        # Бизнес-логика (5 mixin-контроллеров)
│   ├── navigation.py   # Навигация, загрузка изображений
│   ├── structure.py    # Панель структуры, выделение, drag and drop
│   ├── ocr_ops.py      # OCR-операции, сортировка, пайплайн
│   ├── io_ops.py       # Импорт/экспорт JSON, COCO, PDF
│   └── batch_ops.py    # Пакетная обработка
├── models/
│   ├── data.py         # Box, Line, Block
│   ├── database.py     # SQLite ORM
│   └── geometry.py     # YOLO <-> пиксели
├── services/
│   ├── config.py       # Конфигурация
│   ├── ocr_api.py      # REST-клиент Manuscript-OCR
│   ├── ocr_local.py    # Локальный OCR (ONNX)
│   ├── export.py       # Сериализация в JSON/COCO
│   └── pdf.py          # Импорт/экспорт PDF
├── widgets/
│   ├── viewer.py       # Канвас изображения
│   ├── panels.py       # Панели структуры
│   ├── containers.py   # LineWidget, BlockWidget
│   ├── chips.py        # WordChip
│   └── dialogs.py      # Диалоги
└── theme.py            # QSS-стили, автоопределение темы
```

Координаты боксов хранятся в нормализованном YOLO-формате: `x_center`, `y_center`, `width`, `height` (0..1). Конвертация в пиксели выполняется только на этапе отображения.

## Технологии и лицензии зависимостей

| Библиотека | Назначение | Лицензия |
| --- | --- | --- |
| [PySide6](https://doc.qt.io/qtforpython-6/) | GUI-фреймворк (Qt 6) | LGPLv3 |
| [ONNX Runtime](https://onnxruntime.ai/) | Локальный запуск нейросетей | MIT |
| [manuscript-ocr](https://github.com/manuscript-ocr) | OCR-модели (EAST, TRBA, CharLM) | MIT |
| [reportlab](https://www.reportlab.com/) | Генерация PDF | BSD |
| [Pillow](https://pillow.readthedocs.io/) | Обработка изображений | HPND (MIT-подобная) |
| [requests](https://docs.python-requests.org/) | HTTP-клиент для API | Apache 2.0 |
| [SQLite](https://sqlite.org/) | Локальная база аннотаций | Public Domain |
| [PyInstaller](https://pyinstaller.org/) | Сборка исполняемых файлов | GPLv2 (только этап сборки) |

Все зависимости совместимы с коммерческим использованием.

## Лицензия

MIT

## Контрибуции

Pull Request приветствуются. Для крупных изменений сначала создайте `Issue`, чтобы согласовать подход.

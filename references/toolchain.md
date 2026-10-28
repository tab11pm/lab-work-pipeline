# Инструменты отчёта в этой среде

Локальная среда: Linux, Wayland (ShojiWM). Пути ниже — примеры, подставляй
абсолютные пути.

`LAB_DIR` — текущий рабочий каталог пользователя (`pwd`), а не каталог
установки скилла. `OUT_DIR` — `LAB_DIR/outputs`; создай его командой
`mkdir -p "$OUT_DIR"`, если его нет. Не создавай `outputs` внутри
`~/.agents/skills/`, `~/.claude/skills/` или `~/.codex/skills/`.

## Что установлено

- `soffice` / `libreoffice` — конвертация DOCX ↔ PDF и другие форматы.
- `pdftotext`, `pdfinfo`, `pdftoppm` — текст, метаданные и постраничные
  изображения PDF.
- `grim`, `slurp`, `magick` — скриншоты Wayland, выделение области, обрезка.
- `zip`, `unzip` — проверка и сборка архивов.

Не предполагай наличие `python-docx`, `reportlab`, `PyMuPDF`, `pandoc` или
npm-пакета `docx`: в этой среде их нет. Если без них не обойтись, запроси у
пользователя разрешение на установку через оркестратора.

## Выбор инструмента

1. Если хост предоставляет скилл `docx`/`pdf` или `docx-official`/
   `pdf-official`, работай по нему.
2. Иначе используй LibreOffice и OOXML: DOCX можно собрать как zip с
   `word/document.xml` и стилями либо получить из подготовленного HTML/ODT
   через `soffice`.
3. PDF всегда получай из финального DOCX, а не собирай отдельно.

## DOCX

- Шрифт указывай `Times New Roman`; здесь он подставляется метрически
  совместимым `Liberation Serif`. Если преподаватель проверяет именно имя
  шрифта в PDF, установка `ttf-mscorefonts-installer` требует согласия
  пользователя.
- Поля, интервалы, отступы, нумерацию и колонтитулы задавай до заполнения
  текста.
- Не используй пустые абзацы и пробелы для вёрстки.
- После сборки проверь архив: `unzip -t "$OUT_DIR/report.docx"`.

## PDF

```bash
soffice --headless --convert-to pdf --outdir "$OUT_DIR" "$OUT_DIR/report.docx"
```

Проверь, что PDF создан из последней версии DOCX, и что оба файла относятся к
одной версии отчёта.

## Проверка результата

```bash
pdfinfo "$OUT_DIR/report.pdf"
pdftotext "$OUT_DIR/report.pdf" - | head -n 40
pdftoppm -jpeg -r 100 "$OUT_DIR/report.pdf" "$OUT_DIR/.preview/page"
```

Просмотри полученные изображения страниц. Проверь титульный лист, разрывы
страниц, подписи рисунков и таблиц, нумерацию и отсутствие пустых страниц.

## Скриншоты

```bash
mkdir -p "$LAB_DIR/screenshots"
grim "$LAB_DIR/screenshots/full.png"
grim -g "$(slurp)" "$LAB_DIR/screenshots/region.png"
magick "$LAB_DIR/screenshots/region.png" -trim "$LAB_DIR/screenshots/crop.png"
```

Для терминала используй снимок окна терминала или `grim` с выделением области.
Проверь, что на изображении виден именно фактический результат работы
программы. Если GUI недоступен (`$WAYLAND_DISPLAY`/`$DISPLAY` отсутствует),
это блокер: не рисуй и не подделывай скриншот.

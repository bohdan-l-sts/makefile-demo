# Сценарий демо: Makefile в Laravel

Шпаргалка докладчика к «Makefile в Laravel-проектах: от „у меня всё работает" до идеального DX».
Что где лежит, см. [README.md](README.md).

---

## Подготовка перед лекцией (за день / за час)

Интернет на площадке может подвести. Всё тяжёлое скачиваем заранее:

```bash
cd app               # из корня репозитория makefile-demo
git checkout main
make init          # скачает образы php/mysql/node и vendor/, поднимет стек
make assets        # скачает node_modules и соберёт фронт
make -j3 check     # прогреет PHPStan/Pint/тесты
make down
```

Перед самим выступлением:

```bash
cd app               # из корня репозитория makefile-demo
make down
git checkout step-0
```

Шрифт в терминале: крупный (18–20pt). Отключите уведомления.

---

## Сценарий live demo

Порядок: слайды 1–4 (боль) → демо step-0 и step-1 → слайды 5–8 (make, ловушки, план шагов) → демо step-2…step-4 вперемешку со слайдами 9–15 → слайды 16–18 (границы, итоги, вопросы).
Шаги лежат в ветках `step-0` … `step-4` (`main` = `step-4`). Все команды ниже выполняются в `app/`.
Слайды, сниппеты и этот файл есть в каждой ветке, так что после `checkout` ничего не пропадает.
Между шагами: `git checkout step-N`. Если контейнеры уже подняты от другого шага, сделайте `docker compose down` (Dockerfile между шагами отличается).

### step-0: «Музей боли» (2–3 мин)

```bash
git checkout step-0
cat README.md                         # README приложения (app/README.md), не этот сценарий: 11 шагов, PHP и Node на хост, «спросите Васю»
docker compose up -d --build
docker compose exec app php artisan make:model Post
ls -l app/Models/Post.php             # root root → IDE не даст сохранить
docker compose exec app rm app/Models/Post.php
docker compose down
```

Тезис: Docker есть, но пользоваться им неудобно, поэтому все тащат PHP на хост.

### step-1: «Makefile как свалка алиасов» (3 мин)

```bash
git checkout step-1
cat Makefile
make -n test                          # -n: dry run, показывает команды, ничего не запускает
mkdir test && make test               # make: 'test' is up to date.  ← нет .PHONY
rmdir test
echo "hello" >> storage/logs/laravel.log
make logs                             # tail: cannot open 'laravel.log' ← `cd` потерялся
```

Тезис: алиасы уже лучше истории bash, но у make свои правила, и их надо знать.
Здесь переключаемся на слайды «ловушки» и показываем `snippets/01-tabs` (TAB vs пробелы).

### step-2: «Фундамент» (5 мин)

```bash
docker compose down -v                # -v: удалить и том с базой, чтобы init мигрировал с нуля
git checkout step-2
git clean -fdxq -e vendor -e node_modules  # «свежий клон», но без повторного скачивания
make                                  # авто-справка из комментариев ##
make init                             # .env, контейнеры, composer, ключ, миграции, сиды
make init                             # второй раз: APP_KEY не меняется, сидер не падает
make artisan C="make:model Post -m"
ls -l app/Models/Post.php             # вы, а не root: UID/GID проброшены в образ
rm app/Models/Post.php database/migrations/*_create_posts_table.php
make test C="--filter=UserTest"
```

Ключевые моменты: файловая цель `.env`, `-T` для CI, UID/GID, идемпотентный `init`.

### step-3: «Продвинутое» (5–7 мин)

```bash
git checkout step-3
make init                             # composer.lock изменился при checkout (добавили larastan) → install сам
make -n init                          # второй раз composer install не нужен: vendor свежее composer.lock
touch composer.lock && make -n init   # а теперь нужен: make сравнил время файлов
time make check                       # последовательно, ≈3 с
time make -j3 check                   # параллельно ≈1.3 с, вывод не перемешан (--output-sync)
echo n | make db-fresh                # защита от дурака
make db-dump && ls -lh storage/dump.sql
make doctor
make dev                              # Vite в контейнере на :5174, Ctrl+C
```

### step-4: «Идеал» (3–5 мин)

```bash
git checkout step-4
make artisan migrate:status           # позиционные аргументы (двоеточие не ломает make)
make test UserTest                    # = --filter=UserTest
cat .github/workflows/ci.yml          # CI зовёт `make ci`: те же цели, что и локально
cp Makefile.local.example Makefile.local   # личные алиасы, в git не попадают
```

Военная история (реальная, из нашего проекта): первая версия позиционных аргументов объявляла их
в `.PHONY`, и `make artisan migrate:status` падал с `multiple target patterns. Stop.`, потому что
make прочитал двоеточие как «цель: зависимость». Фикс: catch-all правило `%: ; @:`.

### Мини-примеры (`snippets/`, без Docker, мгновенно)

```bash
cd ../snippets                        # из app/
make -C 01-tabs                       # missing separator: пробелы вместо TAB
cd 02-phony && mkdir test && make test; rmdir test; cd ..
make -C 03-one-shell-per-line broken fixed
make -C 04-file-targets; make -C 04-file-targets; touch 04-file-targets/data.csv; make -C 04-file-targets
make -C 05-include-env show           # почему `include .env` опасен
time make -C 06-parallel all; time make -C 06-parallel -j3 all
```

---

## Что поправлено относительно черновика из Gemini

| Было | Проблема | Стало |
|---|---|---|
| `.env:` → `php artisan key:generate` | запускает PHP на хосте, а смысл доклада как раз в том, чтобы его там не было | ключ генерируется в контейнере, и только если его ещё нет |
| `init` всегда делает `key:generate` | повторный `make init` меняет APP_KEY → слетают сессии и шифрованные данные | `grep -q '^APP_KEY=base64' .env \|\| …` |
| `ifneq (",$(wildcard .env)")` | синтаксическая ошибка: кавычка часть строки, условие всегда истинно | `ifneq (,$(wildcard .env))`, а лучше вообще не `include .env` (см. `snippets/05`) |
| `db-dump` берёт пароль из `.env` через `include` | кавычки и `$` в значениях ломаются, пароль светится в `ps` | креды из окружения самого контейнера БД, `MYSQL_PWD` |
| `read -p "…"` | в `/bin/sh` (dash в Ubuntu) нет `-p` | `printf …; read ans`, либо `SHELL := bash` |
| `echo "\033[…"` | `echo` по-разному понимает escape-коды в sh/bash | `printf` |
| `$(MAKE) -j3 lint phpstan test` | вывод трёх процессов перемешивается | `check: lint stan test` + `make -j3` + `--output-sync=target` |
| `docker compose exec -u $(id -u)` | у UID нет пользователя в контейнере: без HOME, у composer проблемы с кэшем | пользователь создаётся в образе с нужным UID/GID (build args) |
| `touch vendor/autoload.php` от `composer.lock` | ок, но забыли `composer.json` | зависимость от обоих файлов |

## Идеи сверху (добавлено в демо)

- `make doctor`: проверка хоста (docker, compose v2, свободные порты).
- `TTY := $(shell [ -t 0 ] || echo -T)`: один Makefile работает и в терминале, и в CI.
- Node в контейнере (`make dev`, `make assets`, `make npm …`): nvm больше не нужен.
- `node_modules/.package-lock.json` как «бесплатный» маркер для инкрементального `npm install`.
- Строгий режим: `SHELL := bash`, `.SHELLFLAGS := -eu -o pipefail -c`, `.DELETE_ON_ERROR:`,
  `--warn-undefined-variables`.
- `-include Makefile.local`: личные настройки без конфликтов в git.
- `make ci` = то, что делает GitHub Actions. Единая правда для людей и CI.
- `make -n` (dry run), `make -p` (всё, что make знает), `make --trace` (почему цель пересобирается).

## Ответы на вопросы из зала

- **Windows?** Работаем в WSL2 (там make есть), проект лежит в файловой системе Linux, а не в `/mnt/c`, иначе всё медленно.
- **macOS?** Там GNU make 3.81 (2006 г.): нет `--output-sync`, нет `.ONESHELL`… Поэтому в демо стоит проверка `$(.FEATURES)`. Либо `brew install make` → `gmake`.
- **Почему не bash-скрипт `./run.sh`?** Нет графа зависимостей, нет инкрементальности, нет `-j`, всё это пришлось бы писать руками.
- **Почему не composer scripts / npm scripts?** Для их запуска уже нужен PHP/Node на хосте.
- **А Sail?** Sail это тоже обёртка, но только для Laravel и без зависимостей между целями. Makefile может звать Sail внутри.
- **Just / Task (go-task)?** Хорошие альтернативы без легаси make, но их надо ставить. make уже есть везде.
- **Когда Makefile плох?** Сложная логика (циклы, JSON, ветвления) → выносите в `scripts/*.sh` и зовите из make.

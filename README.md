# Makefile в Laravel: демо-материалы к лекции

Всё для доклада «Makefile в Laravel-проектах: от „у меня всё работает" до идеального DX».

| Что | Где |
|---|---|
| Сценарий live demo, подготовка, ответы на вопросы | [`DEMO_SCRIPT.md`](DEMO_SCRIPT.md) |
| Слайды (офлайн, один HTML-файл) | `slides/index.html`: открыть в браузере, `→`/`←` листать, `S` заметки спикера, `F` полный экран |
| Демо-проект (Laravel 13, PHP 8.4, MySQL 8.4, Node 22, всё в Docker) | `app/`, по шагам в ветках `step-0` … `step-4` (`main` = `step-4`) |
| Мини-примеры без Docker (для «ловушек») | `snippets/01…06` |

На хосте нужны только **Docker (Compose v2)**, **make** и **git**. Ни PHP, ни Composer, ни Node, ни nvm.

```bash
git clone git@github.com:bohdan-l-sts/makefile-demo.git
cd makefile-demo/app
git checkout step-0   # … step-4, по сценарию из DEMO_SCRIPT.md
```

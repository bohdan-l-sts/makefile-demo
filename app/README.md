# MakeDemo

> ⚠️ Step 0 — "museum of pain". This is what the project README looks like *before* we have a Makefile.
> Everything below is intentionally bad. See `../DEMO_SCRIPT.md` (lecturer guide) for the demo script.

## Как запустить проект (последнее обновление: полгода назад)

1. Поставьте на хост **PHP 8.4** с расширениями `pdo_mysql`, `intl`, `zip`, `pcntl`
   (на Ubuntu 22.04 нужен PPA ondrej/php, на macOS — `brew install php@8.4`, на Windows — удачи).
2. Поставьте **Composer 2** глобально.
3. Поставьте **nvm** и сделайте `nvm install 22 && nvm use 22`
   (не 18 и не 20! на 20 Vite падает, но это не точно).
4. Скопируйте env: `cp .env.example .env`
5. `composer install`
6. `php artisan key:generate`
7. Поднимите базу: `docker compose up -d db`
8. В `.env` поменяйте `DB_HOST=db` на `DB_HOST=127.0.0.1` и `DB_PORT` на `33066`,
   **но не коммитьте** (в контейнере нужно `db`, на хосте — `127.0.0.1`).
9. `php artisan migrate --seed`
10. `npm install && npm run dev`
11. Если что-то не работает — спросите Васю. Вася уволился.

## Полезные команды (скопируйте себе куда-нибудь)

```bash
docker compose up -d --build
docker compose exec app php artisan migrate:fresh --seed
docker compose exec app php artisan test --filter=UserTest
docker compose exec app ./vendor/bin/pint --test
docker compose exec app php artisan make:model Post -m
docker compose exec app composer require spatie/laravel-data
sudo chown -R $USER:$USER .   # после любой команды выше 🙃
sudo chmod -R 777 storage bootstrap/cache
```

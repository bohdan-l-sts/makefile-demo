# Step 1 — "just aliases". Works... until it doesn't.
# Intentional mistakes (see lecturer guide): no .PHONY, `cd` on its own line,
# still runs as root, and every developer still needs .env/key/migrations by hand.

up:
	docker compose up -d --build

down:
	docker compose down

migrate:
	docker compose exec app php artisan migrate

test:
	docker compose exec app php artisan test

logs:
	cd storage/logs
	tail -n 20 laravel.log

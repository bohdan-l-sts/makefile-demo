# `include .env` looks handy, but .env is not Makefile syntax:
#   quotes stay part of the value, `$` starts a make variable, ${VAR} is expanded by make, not dotenv.
# Also watch the guard: Gemini's `ifneq (",$(wildcard .env)")` is a bug, the quote becomes part of the string.
# $(info) prints exactly what make stored, without a shell in between.

ifneq (,$(wildcard .env))
include .env
endif

show:
	$(info APP_NAME    = [$(APP_NAME)])
	$(info DB_PASSWORD = [$(DB_PASSWORD)])
	$(info GREETING    = [$(GREETING)])
	@:

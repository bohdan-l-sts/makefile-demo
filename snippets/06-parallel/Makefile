# time make all         -> ~3 s (one after another)
# time make -j3 all     -> ~1 s (in parallel)
# make -j3 --output-sync=target all   -> each job's output stays together

all: lint stan test

lint stan test:
	@echo "$@: start"
	@sleep 1
	@echo "$@: done"

.PHONY: all lint stan test

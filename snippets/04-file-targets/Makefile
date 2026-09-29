# make is a build tool: a target is rebuilt only when a prerequisite is newer.
#   make          -> builds report.txt
#   make          -> "is up to date" (nothing to do)
#   touch data.csv && make   -> rebuilds

report.txt: data.csv
	@echo "building $@ from $<"
	@sleep 1
	wc -l $< > $@

data.csv:
	printf "id,name\n1,Ann\n2,Bob\n" > $@

clean:
	rm -f report.txt data.csv
.PHONY: clean

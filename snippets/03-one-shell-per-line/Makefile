# Every recipe line runs in its own shell: `cd` (and variables) do not carry over.

broken:
	cd /tmp
	pwd

fixed:
	cd /tmp && pwd

# Or make the whole recipe one shell script (GNU make 3.82+):
# .ONESHELL:

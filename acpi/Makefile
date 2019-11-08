all: patch compile

compile: dsdt.aml

.PHONY: extract extract_ssdt
extract: dsdt.dat

dsdt.dat:
	cat /sys/firmware/acpi/tables/DSDT > dsdt.dat

extract_ssdt:
	for i in /sys/firmware/acpi/tables/SSDT*; do \
		name=$$(basename $$i); \
		sudo cat $$i > $${name,,}.dat; \
	done

patch: dsdt.dsl
	patch -p1 < x1t3.patch

acpi_override: dsdt.aml
	mkdir -p kernel/firmware/acpi
	cp dsdt.aml kernel/firmware/acpi
	find kernel | bsdcpio -H newc --create > acpi_override

.PHONY: install
install: acpi_override
	cp acpi_override /boot

%.dsl: %.dat
	iasl  -d $<

%.aml: %.dsl
	iasl -tc $<

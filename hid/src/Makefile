obj-m += hid-lenovo-tp1gen3.o
obj-m += hid-multitouch-tp1gen3.o

KERNELRELEASE ?= `uname -r`
KBASE ?=  /lib/modules/$(KERNELRELEASE)
KBUILD_DIR ?= $(KBASE)/build
MDEST_DIR ?= $(KBASE)/extramodules

.PHONY: all clean

all:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd`

clean:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd` clean

%.xz: %
	xz -9kf $<

.PHONY: install run run_multitouch

install: hid-lenovo-tp1gen3.ko.xz hid-multitouch-tp1gen3.ko.xz
	for i in $^; do install -D -m 755 "$$i" $(MDEST_DIR)/"$$i"; done

run:
	rmmod hid-multitouch || true
	rmmod hid-multitouch-tp1gen3 || true
	rmmod hid-lenovo-tp1gen3 || true
	insmod hid-lenovo-tp1gen3.ko
	insmod hid-multitouch-tp1gen3.ko
	sleep 10
	rmmod hid-lenovo-tp1gen3
	rmmod hid-multitouch-tp1gen3

run_lenovo:
	rmmod hid-lenovo-tp1gen3 || true
	insmod hid-lenovo-tp1gen3.ko
	sleep 40
	rmmod hid-lenovo-tp1gen3

run_multitouch:
	rmmod hid-multitouch || true
	rmmod hid-multitouch-tp1gen3 || true
	insmod hid-multitouch-tp1gen3.ko
	sleep 5
	rmmod hid-multitouch-tp1gen3
	modprobe hid-multitouch

obj-m += hid-lenovo-tp1gen3.o

KERNELRELEASE ?= `uname -r`
KBASE ?=  /lib/modules/$(KERNELRELEASE)
KBUILD_DIR ?= $(KBASE)/build
MDEST_DIR ?= $(KBASE)/extra

.PHONY: all clean

all:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd`

clean:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd` clean

%.xz: %
	xz -9kf $<

.PHONY: install run
install: hid-lenovo-tp1gen3.ko.xz
	install -D -m 755 $< $(MDEST_DIR)/$<

run:
	rmmod hid-lenovo-tp1gen3 || true
	insmod hid-lenovo-tp1gen3.ko
	sleep 40
	rmmod hid-lenovo-tp1gen3

obj-m += hid-multitouch.o

KERNELRELEASE ?= `uname -r`
KBASE ?=  /lib/modules/$(KERNELRELEASE)
KBUILD_DIR ?= $(KBASE)/build
MDEST_DIR ?= $(KBASE)/extra

.PHONY: all clean

all:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd`

clean:
	$(MAKE) -C $(KBUILD_DIR) M=`pwd` clean

run:
	rmmod hid-multitouch || true
	insmod hid-multitouch.ko
	sleep 20
	rmmod hid-multitouch
	modprobe hid-multitouch

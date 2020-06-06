ifneq ($(KERNELRELEASE),)
EXTRA_CFLAGS += $(XDEFS)
obj-m += kaodv.o
kaodv-objs := kaodv-mod.o kaodv-debug.o kaodv-netlink.o kaodv-queue.o kaodv-ipenc.o kaodv-expl.o
else
KERNEL_DIR=/usr/src/kernel-headers-5.4.38_1
#/root/flexbuild/build/rfs/rootfs_ubuntu_bionic_arm64/lib/modules/4.14.47/build

.PHONY: clean indent all arm

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(shell pwd) modules
	cp kaodv.ko ../

arm:
	$(MAKE) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -C $(KERNEL_DIR) M=$(shell pwd) modules
	cp kaodv.ko ../

clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(shell pwd) clean
	rm -rf *.o *~

TAGS:
	etags *.c *.h
indent:
	indent -kr -i8 -ts8 -sob -l80 -ss -ncs *.c *.h 
endif


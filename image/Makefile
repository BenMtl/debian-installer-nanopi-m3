distribution = $(shell sed -ne 's/^ID=//p' </etc/os-release)
include ../$(distribution).conf

img = $(distribution)-installer-nanopi.img

all: ../ubootimg/boot.img ../ubootenv/ubootenv
	$(MAKE) clean
	@echo "== extracting kernel from package"
	mkdir tmp
	dpkg-deb -x ../base/linux-image-*.deb tmp
	@echo "== creating $(img)"
	dd if=/dev/zero of=$(img) bs=1M count=$(IMAGE_SIZE) status=none
	@echo "== creating fat partition"
	echo ",,6" | /sbin/sfdisk -q $(img)
	@echo "== losetup partition on /dev/loop0"
	sudo losetup -o 1M /dev/loop0 $(img)
	@echo "== creating fat filesystem on partitition"
	sudo mkfs.fat /dev/loop0
	mkdir mnt
	@echo "== mounting $(img) on mnt"
	sudo mount -t vfat /dev/loop0 mnt
	@echo "== copying files"
	sudo cp -v tmp/boot/vmlinuz-* mnt/vmlinuz
	sudo cp -v tmp/boot/nanopim3-*.dtb mnt/nanopim3.dtb
	sudo cp -v ../installer/upstream/build/dest/nanopim3/$(INITRD_PATH) mnt
	@df mnt
	@echo "== unmounting $(img)"
	sudo umount mnt
	@echo "== undoing losetup"
	sudo losetup -d /dev/loop0
	@echo "== embedding boot.img on $(img)"
	dd if=../ubootimg/boot.img of=$(img) seek=1 conv=notrunc status=none
	@echo "== embedding environment for u-boot"
	../ubootenv/ubootenv <uboot-env.txt | dd of=$(img) bs=1k seek=512 conv=notrunc status=none
	@echo "== done =="
	@echo
	@ls -l $(img)
	@echo

clean:
	rm -rf tmp mnt $(img)

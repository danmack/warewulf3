all: ipmitool

top_srcdir = @top_srcdir@

IPMITOOL_VERSION = 1.8.18
IPMITOOL_SOURCE = $(top_srcdir)/3rd_party/BSD/ipmitool-$(IPMITOOL_VERSION).tar.bz2

ipmitool:
	@ if [ ! -d "_work/ipmitool/ipmitool-$(IPMITOOL_VERSION)" ]; then \
	    echo "Preparing to build ipmitool" ;\
	    mkdir -p _work/ipmitool;\
	    tar xjf $(srcdir)/$(IPMITOOL_SOURCE) -C _work/ipmitool/ ;\
	fi
	@ if [ ! -f "_work/ipmitool/ipmitool-$(IPMITOOL_VERSION)/src/ipmitool" ]; then \
	    echo "Building ipmitool" ;\
		if [ -f "/usr/lib/rpm/config.guess" ]; then \
			cp /usr/lib/rpm/config.guess _work/ipmitool/ipmitool-$(IPMITOOL_VERSION) ;\
		fi; \
		if [ -f "/usr/lib/rpm/config.sub" ]; then \
			cp /usr/lib/rpm/config.sub _work/ipmitool/ipmitool-$(IPMITOOL_VERSION) ;\
		fi; \
	    cd _work/ipmitool/ipmitool-$(IPMITOOL_VERSION) ;\
	    ./configure --enable-static --disable-ipmishell ;\
	    make LDFLAGS=-static ;\
	fi
	@ if [ ! -f "unionfs" ]; then \
	    cp -a _work/ipmitool/ipmitool-$(IPMITOOL_VERSION)/src/ipmitool ipmitool ;\
	fi


rootfs: ipmitool
	rm -rf rootfs
	mkdir rootfs
	mkdir rootfs/sbin
	mkdir rootfs/lib
	mkdir rootfs/lib64
	mkdir -p rootfs/warewulf/provision/
	chmod -R u+w rootfs/
	cp -a ipmitool rootfs/sbin/
	find rootfs -type f -perm -o+x  -print | xargs ldd | grep "=>" | awk '{print $$3}' | grep "^/" | sort | uniq | while read i; do cp -L --parents $$i rootfs/; done
	chmod -R +w rootfs/
	find rootfs -type f -perm -o+x -print | xargs strip -g
	install -m 755 $(srcdir)/10-ipmiconfig rootfs/warewulf/provision/10-ipmiconfig

capability.cpio: rootfs
	cd rootfs/; find . | cpio -o -H newc -F ../capability.cpio

install-data-local: capability.cpio
	install -d -m 755 $(DESTDIR)/$(WAREWULF_STATEDIR)/warewulf/initramfs/capabilities
	install -d -m 755 $(DESTDIR)/$(WAREWULF_LIBEXECDIR)/warewulf
	install -m 644 capability.cpio $(DESTDIR)/$(WAREWULF_STATEDIR)/warewulf/initramfs/capabilities/setup-ipmi
	install -m 755 ipmitool $(DESTDIR)/$(WAREWULF_LIBEXECDIR)/warewulf

uninstall-local:
	rm -rf $(DESTDIR)/$(WAREWULF_STATEDIR)/warewulf/initramfs/capabilities/setup-ipmi
	rm -rf $(DESTDIR)/$(WAREWULF_LIBEXECDIR)/warewulf/ipmitool

clean-local:
	rm -rf _work rootfs ipmitool capability.cpio


EXTRA_DIST = 10-ipmiconfig

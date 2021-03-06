include ../common.mk

PACKAGE_D ?= gkrellmd

SINSTALLDIR ?= $(INSTALLROOT)/bin

CFGDIR ?= $(INSTALLROOT)/etc
CFGDIRMODE ?= 755
CFGMODE ?= 644

SMANDIR ?= $(INSTALLROOT)/share/man/man1

EXTRAOBJS =

ifeq ($(without-libsensors),yes)
	CONFIGURE_ARGS += --without-libsensors
endif
ifeq ($(without-libsensors),1)
	CONFIGURE_ARGS += --without-libsensors
endif

DUMMY_VAR := $(shell ./configure $(CONFIGURE_ARGS))

HAVE_LIBSENSORS = $(shell grep -c HAVE_LIBSENSORS configure.h)
ifeq ($(HAVE_LIBSENSORS),1)
    SENSORS_LIBS ?= -lsensors
endif

GKRELLMD_INCLUDES = gkrellmd.h $(SHARED_PATH)/log.h

PKG_INCLUDE := $(shell $(PKG_CONFIG) --cflags glib-2.0 gmodule-2.0 gthread-2.0)
PKG_LIB := $(shell $(PKG_CONFIG) --libs glib-2.0 gmodule-2.0 gthread-2.0)

FLAGS = -O2 $(PKG_INCLUDE)

FLAGS += $(GTOP_INCLUDE) $(PTHREAD_INC) -I.. -I$(SHARED_PATH) -DGKRELLM_SERVER

LIBS = $(PKG_LIB)
LIBS += $(GTOP_LIBS_D) $(SYS_LIBS) $(SENSORS_LIBS)

ifeq ($(debug),1)
    FLAGS += -g
endif
ifeq ($(debug),yes)
    FLAGS += -g
endif

ifeq ($(profile),1)
    FLAGS += -g -pg
endif
ifeq ($(profile),yes)
    FLAGS += -g -pg
endif

ifeq ($(enable_nls),1)
    FLAGS += -DENABLE_NLS -DLOCALEDIR=\"$(LOCALEDIR)\"
endif
ifeq ($(enable_nls),yes)
    FLAGS += -DENABLE_NLS -DLOCALEDIR=\"$(LOCALEDIR)\"
endif

ifneq ($(PACKAGE_D),gkrellmd)
    FLAGS += -DPACKAGE_D=\"$(PACKAGE_D)\"
endif

ifeq ($(HAVE_GETADDRINFO),1)
    FLAGS += -DHAVE_GETADDRINFO
endif

OS_RELEASE=$(shell uname -r)

OBJS = main.o monitor.o mail.o plugins.o utils.o sysdeps-unix.o log.o

all:	gkrellmd$(BINEXT)

gkrellmd$(BINEXT): check_env $(OBJS) $(EXTRAOBJS)
	$(CC) $(OBJS) $(EXTRAOBJS) -o $@ \
		$(LDFLAGS) $(LIBS) $(LINK_FLAGS)

static: check_env $(OBJS) $(EXTRAOBJS)
	$(CC) $(OBJS) $(EXTRAOBJS) -o gkrellmd.static -static \
		$(LDFLAGS) $(LIBS) $(LINK_FLAGS)

freebsd2:
	$(MAKE) EXTRAOBJS= SYS_LIBS="-lkvm -lmd" gkrellmd

freebsd3 freebsd:
	$(MAKE) EXTRAOBJS= SYS_LIBS="-lkvm -ldevstat -lmd" gkrellmd

# Darwin target for systems that still have libkvm
darwin: 
	$(MAKE) STRIP= HAVE_GETADDRINFO=1 \
		EXTRAOBJS= \
		SYS_LIBS="-framework CoreFoundation -framework IOKit -lkvm" \
		LINK_FLAGS="-Wl,-bind_at_load" \
		gkrellmd

# Darwin 9.x and newer (OS X > 10.4) do not have libkvm, build without it
darwin9: 
	$(MAKE) STRIP= HAVE_GETADDRINFO=1 \
		EXTRAOBJS= \
		SYS_LIBS="-framework CoreFoundation -framework IOKit" \
		LINK_FLAGS="-Wl,-bind_at_load" \
		gkrellmd

# Same as darwin9
macosx: darwin9

netbsd1:
	$(MAKE) EXTRAOBJS= SYS_LIBS="-lkvm" gkrellmd

netbsd2:
	$(MAKE) EXTRAOBJS= SYS_LIBS="-lkvm -pthread" gkrellmd

openbsd:
	$(MAKE) SYS_LIBS="-lkvm -pthread" gkrellmd

solaris:
ifeq ($(OS_RELEASE),5.8)
	$(MAKE) CFLAGS="-Wno-implicit-int" \
		SYS_LIBS="-lkstat -lkvm -ldevinfo -lsocket -lnsl -lintl" \
		LINK_FLAGS="" gkrellmd
else
	$(MAKE) CFLAGS="-Wno-implicit-int" \
		SYS_LIBS="-lkstat -lkvm -ldevinfo -lsocket -lnsl" \
		LINK_FLAGS="" gkrellmd
endif

windows: libgkrellmd.a
	$(MAKE) \
		BINEXT=".exe" \
		CFLAGS="${CFLAGS} -D_WIN32_WINNT=0x0501 -DWINVER=0x0501 -DWIN32_LEAN_AND_MEAN" \
		LINK_FLAGS="-mconsole -Wl,--dynamicbase -Wl,--nxcompat" \
		EXTRAOBJS="${EXTRAOBJS} win32-resource.o win32-plugin.o" \
		SYS_LIBS="-lws2_32 -lwtsapi32 -lpdh -lnetapi32 -liphlpapi -lintl" \
		all

install: install_bin install_inc install_man

install_bin:
	$(INSTALL) -d -m $(INSTALLDIRMODE) $(SINSTALLDIR)
	$(INSTALL) -c $(STRIP) -m $(BINMODE) $(PACKAGE_D)$(BINEXT) $(SINSTALLDIR)/$(PACKAGE_D)$(BINEXT)

install_inc:
	$(INSTALL) -d -m $(INCLUDEDIRMODE) $(INCLUDEDIR)/gkrellm2
	$(INSTALL) -c -m $(INCLUDEMODE) $(GKRELLMD_INCLUDES) $(INCLUDEDIR)/gkrellm2

install_man:
	$(INSTALL) -d -m $(MANDIRMODE) $(SMANDIR)
	$(INSTALL) -c -m $(MANMODE) ../gkrellmd.1 $(SMANDIR)/$(PACKAGE_D).1

install_cfg:
	$(INSTALL) -d -m $(CFGDIRMODE) $(CFGDIR)
	$(INSTALL) -c -m $(CFGMODE) gkrellmd.conf $(CFGDIR)/gkrellmd.conf

uninstall:
	rm -f $(SINSTALLDIR)/$(PACKAGE_D)
	rm -f $(SMANDIR)/$(PACKAGE_D).1

install_darwin install_darwin9 install_macosx:
	$(MAKE) install STRIP=

install_freebsd:
	$(MAKE) install
	chgrp kmem $(SINSTALLDIR)/$(PACKAGE_D)
	chmod g+s $(SINSTALLDIR)/$(PACKAGE_D)

install_netbsd:
	$(MAKE) SMANDIR="$(INSTALLROOT)/man/man1" install

install_openbsd:
	$(MAKE) install
	chgrp kmem $(SINSTALLDIR)/$(PACKAGE_D)
	chmod g+sx $(SINSTALLDIR)/$(PACKAGE_D)

install_solaris:
	$(MAKE) install INSTALL=/usr/ucb/install
	chgrp sys $(SINSTALLDIR)/$(PACKAGE_D)
	chmod g+s $(SINSTALLDIR)/$(PACKAGE_D)  

install_windows: install_inc install_cfg
	$(MAKE) BINEXT=".exe" install_bin
	$(INSTALL) -d -m $(LIBDIRMODE) $(LIBDIR)
	$(INSTALL) -c -m $(BINMODE) libgkrellmd.a $(LIBDIR)

clean:
	$(RM) *.o *~ *.bak configure.h configure.log gkrellmd gkrellmd.exe \
		libgkrellmd.a core

SYSDEPS = ../src/sysdeps/bsd-common.c ../src/sysdeps/bsd-net-open.c \
	../src/sysdeps/darwin.c \
	../src/sysdeps/freebsd.c ../src/sysdeps/gtop.c \
	../src/sysdeps/linux.c ../src/sysdeps/netbsd.c ../src/sysdeps/dragonfly.c \
	../src/sysdeps/openbsd.c ../src/sysdeps/sensors-common.c \
	../src/sysdeps/solaris.c ../src/sysdeps/win32.c 

GKRELLMD_H = gkrellmd.h gkrellmd-private.h

main.o:		main.c $(GKRELLMD_H)
monitor.o:	monitor.c $(GKRELLMD_H) 
mail.o:		mail.c $(GKRELLMD_H)
plugins.o:	plugins.c $(GKRELLMD_H) 
utils.o:	utils.c $(GKRELLMD_H)
sysdeps-unix.o: sysdeps-unix.c ../src/gkrellm-sysdeps.h $(SYSDEPS) $(GKRELLMD_H)
log.o: $(SHARED_PATH)/log.c $(SHARED_PATH)/log.h $(GKRELLMD_H)
win32-gui.o:	win32-gui.c
win32-plugin.o:	win32-plugin.c win32-plugin.h $(GKRELLMD_H)
win32-resource.o:	win32-resource.rc win32-resource.h
	$(WINDRES) $< -o $@
win32-libgkrellmd.o:	win32-libgkrellmd.c win32-plugin.h $(GKRELLMD_H)
libgkrellmd.a:	win32-libgkrellmd.o
	$(AR) -cr libgkrellmd.a win32-libgkrellmd.o

# Checks if the build environment is ok
check_env:
	$(PKG_CONFIG) --atleast-version=2.32 glib-2.0

CC=gcc
INC=-Ilibwps -I.
prefix=/usr/local
exec_prefix=${prefix}
CONFDIR=${prefix}/etc/reaver
CFLAGS=-DCONF_DIR='"$(CONFDIR)"' -DREAVER_DATABASE='"$(CONFDIR)/reaver.db"' -Wall -g -O2 -Wno-unused-function

WPS_OBJS=wps/wps_attr_build.o wps/wps_attr_parse.o wps/wps_attr_process.o \
wps/wps.o wps/wps_common.o wps/wps_dev_attr.o wps/wps_enrollee.o \
wps/wps_registrar.o wps/wps_ufd.o


LIB_OBJS=libwps/libwps.o $(WPS_OBJS) utils/libutils.a \
	crypto/libcrypto.a tls/libtls.a

LDFLAGS=-lm -lpcap -lsqlite3 
UNAME := $(shell uname)

ifeq ($(UNAME), Darwin)
  LIBIWNAME=
else
  LIB_OBJS+=lwe/libiw.a
  LIBIWNAME=libiw
endif

WASH_OBJS=globule.o init.o misc.o 80211.o iface.o \
 builder.o crc.o
REAVER_OBJS=argsparser.o globule.o init.o sigint.o iface.o sigalrm.o \
 misc.o cracker.o sql.o session.o send.o pins.o 80211.o exchange.o builder.o \
 keys.o crc.o

all: wash reaver

$(WPS_OBJS): CFLAGS+=-I. -Iutils

wash: $(LIBIWNAME) $(WASH_OBJS) $(LIB_OBJS)
	$(CC) $(CFLAGS) $(INC) wpsmon.c $(WASH_OBJS) $(LIB_OBJS) $(LDFLAGS) -o wash

reaver: $(LIBIWNAME) $(REAVER_OBJS) $(LIB_OBJS)
	$(CC) $(CFLAGS) $(INC) wpscrack.c $(REAVER_OBJS) $(LIB_OBJS) $(LDFLAGS) -o reaver

utils/libutils.a:
	(cd utils && $(MAKE))

crypto/libcrypto.a: tls/libtls.a
	(cd crypto && $(MAKE))

tls/libtls.a:
	(cd tls && $(MAKE))

libiw:
	(cd lwe && $(MAKE) BUILD_STATIC=y libiw.a)

init.o:
	$(CC) $(CFLAGS) init.c -c

sql.o:
	$(CC) $(CFLAGS) $(INC) sql.c -c

crc.o:
	$(CC) $(CFLAGS) crc.c -c

keys.o:
	$(CC) $(CFLAGS) keys.c -c

argsparser.o: globule.o
	$(CC) $(CFLAGS) $(INC) argsparser.c -c

sigint.o: globule.o
	$(CC) $(CFLAGS) $(INC) sigint.c -c

exchange.o: globule.o send.o sigalrm.o 80211.o
	$(CC) $(CFLAGS) $(INC) exchange.c -c

send.o: globule.o builder.o sigalrm.o
	$(CC) $(CFLAGS) send.c -c

session.o: globule.o
	$(CC) $(CFLAGS) $(INC) session.c -c

80211.o: globule.o sql.o builder.o crc.o
	$(CC) $(CFLAGS) $(INC) 80211.c -c

iface.o: globule.o
	$(CC) $(CFLAGS) iface.c -c

sigalrm.o: globule.o
	$(CC) $(CFLAGS) sigalrm.c -c

misc.o: globule.o 
	$(CC) $(CFLAGS) misc.c -c

builder.o: globule.o
	$(CC) $(CFLAGS) builder.c -c

pins.o: globule.o keys.o
	$(CC) $(CFLAGS) pins.c -c

cracker.o: globule.o init.o pins.o iface.o exchange.o session.o 80211.o
	$(CC) $(CFLAGS) $(INC) cracker.c -c

globule.o:
	$(CC) $(CFLAGS) globule.c -c 

install: cleanbin
	if [ ! -d $(CONFDIR) ]; then mkdir -p $(CONFDIR); fi
	if [ ! -e $(CONFDIR)/reaver.db ]; then cp reaver.db $(CONFDIR)/reaver.db && chmod -R a+rw $(CONFDIR); fi
	if [ -e wash ]; then cp wash ${exec_prefix}/bin/wash; fi
	if [ -e reaver ]; then cp reaver ${exec_prefix}/bin/reaver; fi

clean:
	rm -f *~ *.o reaver wash
	rm -rf *.cache config.status config.log
	rm -f $(LIB_OBJS)
	(cd utils && $(MAKE) clean)
	(cd tls && $(MAKE) clean)
	(cd crypto && $(MAKE) clean)
	(cd lwe && $(MAKE) realclean)
	rm -f Makefile config.h

cleanbin: 
	rm -f ${exec_prefix}/bin/reaver ${exec_prefix}/bin/wash ${exec_prefix}/bin/walsh

distclean: cleanbin clean
	rm -rf $(CONFDIR)

.PHONY: libiw clean cleanbin distclean install

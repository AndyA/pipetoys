DIST_SUBDIRS = m4 t

EXTRA_DIST = 

SUBDIRS = . t

ACLOCAL_AMFLAGS = -I m4

noinst_LTLIBRARIES = libpipetoys.la

bin_PROGRAMS = fatcat tailpipe spliff

libpipetoys_la_SOURCES = utils.h utils.c buffer.h buffer.c
libpipetoys_la_LDFLAGS = -avoid-version -static

fatcat_SOURCES = fatcat.c
fatcat_LDADD = libpipetoys.la

tailpipe_SOURCES = tailpipe.c
tailpipe_LDADD = libpipetoys.la

spliff_SOURCES = spliff.c
spliff_LDADD = libpipetoys.la

test: all
	cd t && $(MAKE) test


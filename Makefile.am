###############################################################################
## Makefile for apertium-fra
###############################################################################

RELEASE=1.1
VERSION=1.1.0
LANG1=fra
BASENAME=apertium-$(LANG1)

TARGETS_COMMON = $(LANG1).automorf.bin \
	$(LANG1).autogen.bin $(LANG1).autopgen.bin \
	$(LANG1).automorf.att.gz $(LANG1).autogen.att.gz \
	$(LANG1).rlx.bin

# Use this goal for creating .deps, otherwise make -j2 will give problems:
@ap_include@

###############################################################################
## French transducer
###############################################################################

$(LANG1).xsl: $(BASENAME).$(LANG1).metadix buscaPar.xsl
	xsltproc buscaPar.xsl $< |uniq >$@.tmp
	mv $@.tmp $@

.deps/$(LANG1).dix: $(LANG1).xsl $(BASENAME).$(LANG1).metadix .deps/.d
	xsltproc $< $(BASENAME).$(LANG1).metadix > $@.tmp
	mv $@.tmp $@

$(LANG1).autogen.bin: .deps/$(LANG1).dix $(BASENAME).$(LANG1).acx
	apertium-validate-dictionary $<
	lt-comp rl $< $@ $(BASENAME).$(LANG1).acx

$(LANG1).automorf.bin: .deps/$(LANG1).dix $(BASENAME).$(LANG1).acx
	apertium-validate-dictionary $<
	lt-comp lr $< $@ $(BASENAME).$(LANG1).acx

$(LANG1).autogen.att.gz: $(LANG1).autogen.bin
	lt-print $< | gzip -9 -c -n > $@

$(LANG1).automorf.att.gz: $(LANG1).automorf.bin
	lt-print $< | gzip -9 -c -n > $@

$(LANG1).autopgen.bin: $(BASENAME).post-$(LANG1).dix
	lt-comp lr $< $@

###############################################################################
## Disambiguation rules
###############################################################################

$(LANG1).rlx.bin: $(BASENAME).$(LANG1).rlx $(CGCOMP)
	$(CGCOMP) $< $@

###############################################################################
## Distribution
###############################################################################
EXTRA_DIST=$(BASENAME).$(LANG1).metadix \
		$(BASENAME).$(LANG1).acx \
		$(BASENAME).$(LANG1).rlx \
		$(BASENAME).post-$(LANG1).dix \
		$(LANG1).prob \
		buscaPar.xsl \
		principal.xsl \
		modes.xml

###############################################################################
## Installation stuff
###############################################################################
#
#   apertium_fra_dir: This is where the compiled binaries go
#   apertium_fra_srcdir: This is where the source files go

apertium_fradir=$(prefix)/share/apertium/$(BASENAME)/
apertium_fra_srcdir=$(prefix)/share/apertium/$(BASENAME)/

apertium_fra_DATA=$(TARGETS_COMMON) $(LANG1).prob

pkgconfigdir = $(prefix)/share/pkgconfig
pkgconfig_DATA = $(BASENAME).pc

# Modes are not installed since they're all just "debug modes". All
# modes are created by this goal.
modes/$(LANG1)-morph.mode: modes.xml
	apertium-validate-modes modes.xml
	apertium-gen-modes modes.xml

noinst_DATA=modes/$(LANG1)-morph.mode

install-data-local: install-modes
	test -d $(DESTDIR)$(apertium_fra_srcdir) || mkdir -p $(DESTDIR)$(apertium_fra_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).metadix $(DESTDIR)$(apertium_fra_srcdir)
	$(INSTALL_DATA) $(BASENAME).$(LANG1).rlx $(DESTDIR)$(apertium_fra_srcdir)
	$(INSTALL_DATA) $(BASENAME).post-$(LANG1).dix $(DESTDIR)$(apertium_fra_srcdir)

CLEANFILES = $(TARGETS_COMMON)
clean-local:
	-rm -rf .deps modes

###############################################################################
## Test
###############################################################################

test: all
	echo "TODO test sentence" | apertium -d . fra-morph | tee .test-mt
	@echo "TODO test analysis results" > .test-ref
	@diff .test-ref .test-mt
	@rm .test-ref .test-mt

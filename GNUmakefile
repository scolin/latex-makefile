
# Copyright 2008 Samuel Colin
# Copyright 2004 Chris Monson (monpublic@gmail.com)

# This file is based on Chris Monson's Makefile:
# http://www.bouncingchairs.net/oss
# As such, parts derived directly from it are licenced under the 
# GPLv2 (http://www.gnu.org/copyleft/gpl.html)
# under the following terms, copied verbatim from the original Makefile
# 
# Parts I added are licenced GPLv2 as well

TMPDIR=._d

define find_dvi
grep '^[%]\+[ \t]*scolin-latex[ \t]*:[ \t]*dvi[ \t]*$$' $(FILE).tex >/dev/null 2>&1
endef

ifdef FILE
EXT ?= $(shell $(find_dvi) && echo dvi || echo pdf)
#EXT ?= pdf
endif

ifeq ($(EXT),pdf)
  BARELATEX = pdflatex
else
  BARELATEX = latex
endif

HEVEAFLAGS ?= -fix
BIBFLAGS ?= -min-crossrefs=1
#VERBOSE := y

# PROGRAMS:
# Unix utilities with particular parameters
#SORT		:= LC_ALL=C sort
SED		:= sed
# Compatibility with FreeBSD
CP		:= cp -RpP
# Avoiding builtin echo
ECHO		:= /bin/echo
# == LaTeX (tetex-provided) ==
# TODO: TeXlive now ?
BIBTEX          := bibtex $(BIBFLAGS)
DVIPS		:= dvips
LATEX		:= $(BARELATEX) -recorder -interaction=nonstopmode $(TEXFLAGS)
KPSEWHICH	:= kpsewhich
PS2PDF_NORMAL	:= ps2pdf
PS2PDF_EMBED	:= ps2pdf13
HEVEA           := hevea $(HEVEAFLAGS)
MAKEINDEX       := makeindex -q
# Glosstex, also ?
# = OPTIONAL PROGRAMS =
# == Makefile Color Output ==
TPUT		:= tput

# Characters that are hard to specify in certain places
space		:= $(empty) $(empty)

# Turn command echoing back on with VERBOSE=something
ifndef VERBOSE
QUIET	:= @
TODEVNULL := > /dev/null
else
SHELL	+= -x
endif

# Get the name of this makefile (always right in 3.80, often right in
# 3.79) This is only really used for documentation, so it isn't too
# serious.
ifdef MAKEFILE_LIST
this_file	:= $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
else
this_file	:= $(wildcard GNUmakefile makefile Makefile)
endif

# Terminal color definitions
black	:= $(shell $(TPUT) setaf 0)
red	:= $(shell $(TPUT) setaf 1)
green	:= $(shell $(TPUT) setaf 2)
yellow	:= $(shell $(TPUT) setaf 3)
blue	:= $(shell $(TPUT) setaf 4)
magenta	:= $(shell $(TPUT) setaf 5)
cyan	:= $(shell $(TPUT) setaf 6)
white	:= $(shell $(TPUT) setaf 7)
bold	:= $(shell $(TPUT) bold)
uline	:= $(shell $(TPUT) smul)
reset	:= $(shell $(TPUT) sgr0)

#
# User-settable definitions
#
LATEX_COLOR_WARNING	?= red
LATEX_COLOR_ERROR	?= red bold
LATEX_COLOR_INFO	?= bold

# Gets the real color from a simple textual definition like those above
# $(call get-color,ALL_CAPS_COLOR_NAME)
# e.g., $(call get-color,WARNING)
get-color	= $(subst $(space),,$(foreach c,$(LATEX_COLOR_$1),$($c)))

#
# STANDARD COLORS
#
C_WARNING	:= $(call get-color,WARNING)
C_ERROR		:= $(call get-color,ERROR)
C_INFO		:= $(call get-color,INFO)
C_RESET		:= $(reset)

# Check that clean targets are not combined with other targets (weird
# things happen, and it's not easy to fix them)
hascleangoals	:= $(if $(sort $(filter clean purge,$(MAKECMDGOALS))),1)
hasbuildgoals	:= $(if $(sort $(filter-out clean purge,$(MAKECMDGOALS))),1)
ifneq "$(hasbuildgoals)" ""
ifneq "$(hascleangoals)" ""
$(error $(C_ERROR)Clean and build targets specified together$(C_RESET)))
endif
endif


# MORECLEAN is specified by the user, if he wants to remove additional
# files when cleaning
LATEXCLEAN = $(FILE).log $(FILE).out $(FILE).aux $(FILE).fls
LATEXCLEAN+= $(FILE).blg
LATEXCLEAN+= $(FILE).idx $(FILE).ilg $(FILE).ist 
LATEXCLEAN+= $(FILE).glo $(FILE).glg
LATEXCLEAN+= $(FILE).toc $(FILE).lof $(FILE).lot
LATEXCLEAN+= $(FILE).maf
LATEXCLEAN+= $(FILE).ptc* $(FILE).plf* $(FILE).plt*
LATEXCLEAN+= $(FILE).mtc* $(FILE).mlf* $(FILE).mlt*
LATEXCLEAN+= $(FILE).stc* $(FILE).slf* $(FILE).slt*
LATEXCLEAN+= $(FILE).nav $(FILE).snm  $(FILE).vrb 
LATEXCLEAN+= *.aux *.blg
LATEXCLEAN+= $(FILE).haux $(FILE).htoc

LATEXCLEAN+= $(MORECLEAN)

# MOREPURGE: see MORECLEAN, but for the purge target
LATEXPURGE = $(FILE).ps $(FILE).dvi $(FILE).pdf 
LATEXPURGE+= $(FILE).bbl $(FILE).ind $(FILE).gls
LATEXPURGE+= bu*.bbl
LATEXPURGE+= $(FILE).html

LATEXPURGE+= $(MOREPURGE)


############################################################################

#
# Utility Functions and Definitions
#

# test-different		= ! diff -q '$1' '$2' &>/dev/null

# replace-if-different-and-remove	= \
# 	$(call test-different,$1,$2) && mv -f '$1' '$2' || rm -f '$1'

# $(call replace-if-different-and-remove,<source>,<target>)
define replace-if-different-and-remove
if [ ! -f "$1" ]; then $(ECHO) "$1" should exist; exit 1; fi; \
if [ ! -f "$2" ]; then mv -f "$1" "$2"; \
else diff -q "$1" "$2" $(TODEVNULL); \
     if [ $$? -ne 0 ]; then mv -f "$1" "$2"; \
     else rm -f "$1"; fi ; \
fi
endef

# Outputs all source dependencies to stdout.  The first argument is
# the file to be parsed, the second is a list of files that will show
# up as dependencies in the new .deps file created here.
#
# NOTE: BSD sed does not understand \|, so we have to do something
# more clunky to extract suitable extensions.
#
# $(call get-inputs,<stem>,<target files>)
define get-inputs
if [ ! -f $(TMPDIR)/get-inputs.sed ]; then touch $(TMPDIR)/get-inputs.sed; $(call make-get-inputs); fi ; \
$(SED) -f $(TMPDIR)/get-inputs.sed $(TMPDIR)/$1.fls | \
$(SED) -e 's/^.*$$/$2: &/' | \
sort | uniq >$(TMPDIR)/$1.deps
endef

define echo-get-inputs
$(ECHO) >>$(TMPDIR)/get-inputs.sed
endef


define make-get-inputs
$(echo-get-inputs) '/^INPUT/!d' ; \
$(echo-get-inputs) 's!^INPUT !!' ; \
$(echo-get-inputs) '/^\/.*$$/d' ; \
$(echo-get-inputs) '/^[a-zA-Z]:/d' ; \
$(echo-get-inputs) '/\.aux$$/d' ; \
$(echo-get-inputs) '/\.bbl$$/d' ; \
$(echo-get-inputs) '/\.ind$$/d' ; \
$(echo-get-inputs) '/\.gls$$/d' ; \
$(echo-get-inputs) '/\.nav$$/d' ; \
$(echo-get-inputs) '/\.toc$$/d' ; \
$(echo-get-inputs) '/\.ptc[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.mtc[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.stc[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.lof$$/d' ; \
$(echo-get-inputs) '/\.plf[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.mlf[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.slf[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.lot$$/d' ; \
$(echo-get-inputs) '/\.plt[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.mlt[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.slt[0-9]*$$/d' ; \
$(echo-get-inputs) '/\.out$$/d'
endef


# $(call update_file,<filename>)
define update_file
if [ -f $1 ]; then cat $1 >>$1.cookie; fi ; \
sort $1.cookie | uniq >$1; \
rm $1.cookie
endef

# In purge : dvi,ps,pdf
# In clean : maf,ilg,glg,blg,out,log
#            lot,lof,toc,nav
#            plt*,plf*,ptc*,mlt*,mlf*,mtc*,slt*,slf*,stc*,
#            glo,gls,idx,ind,ist,aux

# This function will get the generated files and add them to the
# cleaning lists 
# $(call update_clean_file,<stem>)
# 
define update_clean_file
egrep '^OUTPUT' $(TMPDIR)/$1.fls | \
$(SED) -e 's/^OUTPUT //' | \
egrep '\.maf$$|\.ilg$$|\.glg$$|\.blg$$|\.out$$|\.log$$|\.lot$$|\.lof$$|\.toc$$|\.plt[0-9]*$$|\.plf[0-9]*$$|\.ptc[0-9]*$$|\.mlt[0-9]*$$|\.mlf[0-9]*$$|\.mtc[0-9]*$$|\.slt[0-9]*$$|\.slf[0-9]*$$|\.stc[0-9]*$$|\.nav$$|\.snm$$|\.glo$$|\.gls$$|\.idx$$|\.ind$$|\.ist$$|\.aux$$' \
 >$(TMPDIR)/$1.clean.cookie; \
$(call update_file,$(TMPDIR)/$1.clean)
endef

# This function will get the generated files and add them to the
# cleaning lists 
# $(call update_clean_file,<stem>)
# 
define update_purge_file
egrep '^OUTPUT' $(TMPDIR)/$1.fls | \
$(SED) -e 's/^OUTPUT //' | \
egrep '\.pdf$$|\.ps$$|\.dvi$$' \
 >$(TMPDIR)/$1.purge.cookie; \
$(call update_file,$(TMPDIR)/$1.purge)
endef

# $(call trim,<file>) removes duplicate lines from this files
define trim
mv $1 $1.cookie ;\
sort $1.cookie | uniq >$1 ;\
rm $1.cookie
endef


# Note: nested "call" works here because the parameters are known
# statically

# $(call get-bbl-deps,<stem>,,<targets>) 
# We exploit the fact that a bbl appearing in the .fls is not "No
# file" in the .log, and vice-versa
#
define get-bbl-deps
bblstems1=`$(SED) -e '/^No file \(.*\.bbl\)\./!d' -e 's/No file \(.*\)\.bbl\./\1/g' $(TMPDIR)/$1.log | sort | uniq`; \
bblstems2=`$(SED) -e '/^INPUT.*\.bbl$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\)\.bbl$$!\2!' $(TMPDIR)/$1.fls | sort | uniq`; \
bblstems="$$bblstems1 $$bblstems2"; \
for i in $$bblstems; \
do \
  $(ECHO) $2: $$i.bbl >>$(TMPDIR)/$1.deps; \
  $(SED) \
  -e '/^\\bibdata/!d' \
  -e 's/\\bibdata{\([^}]*\)}/\1,/' \
  -e 's/,\{2,\}/,/g' \
  -e 's/,/.bib /g' \
  -e 's/ \{1,\}$$//' \
  $$i.aux | xargs $(KPSEWHICH) - | \
  $(SED) -e "s/^/$$i.bbl: /" | \
  sort | uniq >>$(TMPDIR)/$1.deps; \
  $(ECHO) $$i.aux >>$(TMPDIR)/$1.clean ;\
  $(ECHO) $$i.blg >>$(TMPDIR)/$1.clean ;\
  $(ECHO) $$i.bbl >>$(TMPDIR)/$1.purge ;\
  $(ECHO) $(TMPDIR)/$$i.auxbbl >>$(TMPDIR)/$1.purge ;\
done; \
$(call trim,$(TMPDIR)/$1.clean) ;\
$(call trim,$(TMPDIR)/$1.purge)
endef

# Compute index and glossary dependencies
# $(call make-inds-deps,<stem>,<targets>)
#
define make-inds-deps
indstems1=`$(SED) -e '/^No file \(.*\.ind\)\./!d' -e 's/No file \(.*\.ind\)\./\1/g' $(TMPDIR)/$1.log | sort | uniq`; \
indstems2=`$(SED) -e '/^INPUT.*\.ind$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.ind\)$$!\2!' $(TMPDIR)/$1.fls | sort | uniq`; \
glsstems1=`$(SED) -e '/^No file \(.*\.gls\)\./!d' -e 's/No file \(.*\.gls\)\./\1/g' $(TMPDIR)/$1.log | sort | uniq`; \
glsstems2=`$(SED) -e '/^INPUT.*\.gls$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.gls\)$$!\2!' $(TMPDIR)/$1.fls | sort | uniq`; \
indstems="$$indstems1 $$indstems2"; \
glsstems="$$glsstems1 $$glsstems2"; \
if [ "x$$indstems" != "x " ]; \
then \
  $(ECHO) $2: $$indstems >>$(TMPDIR)/$1.deps; \
  $(ECHO) $$indstems >>$(TMPDIR)/$1.purge; \
  $(ECHO) $$indstems | $(SED) -e 's/\.ind/.ilg/g' >>$(TMPDIR)/$1.clean ;\
fi; \
if [ "x$$glsstems" != "x " ]; \
then \
  $(ECHO) $2: $$glsstems >>$(TMPDIR)/$1.deps; \
  $(ECHO) $$glsstems >>$(TMPDIR)/$1.purge; \
  $(ECHO) $$glsstems | $(SED) -e 's/\.gls/.glg/g' >>$(TMPDIR)/$1.clean ;\
fi ;\
$(call trim,$(TMPDIR)/$1.clean) ;\
$(call trim,$(TMPDIR)/$1.purge)
endef


define echo-flataux
$(ECHO) >>$(TMPDIR)/flatten-aux.sed
endef

define make-flatten-aux
$(echo-flataux) '/\\@input{\(.*\)}/{'; \
$(echo-flataux) '  s//\1/'; \
$(echo-flataux) '  h'; \
$(echo-flataux) '  s!.*!\\:\\@input{&}:{!'; \
$(echo-flataux) '  p'; \
$(echo-flataux) '  x'; \
$(echo-flataux) '  s/.*/r &/p'; \
$(echo-flataux) '  s/.*/d/p'; \
$(echo-flataux) '  s/.*/}/p'; \
$(echo-flataux) '  d'; \
$(echo-flataux) '}'; \
$(echo-flataux) 'd'
endef


# Get all important .aux files from the top-level .aux file and merges
# them all into a single file, which it outputs to stdout.
#
# $(call flatten-aux,<toplevel aux>,<output file>)
define flatten-aux
if [ ! -f $(TMPDIR)/flatten-aux.sed ]; then touch $(TMPDIR)/flatten-aux.sed; $(call make-flatten-aux); fi; \
$(SED) -f $(TMPDIR)/flatten-aux.sed '$1' > "$(TMPDIR)/$1.$$$$.sed.make"; \
$(SED) -f "$(TMPDIR)/$1.$$$$.sed.make" '$1' > "$(TMPDIR)/$1.$$$$.make"; \
$(SED) \
-e '/^\\relax/d' \
-e '/^\\bibcite/d' \
-e 's/^\(\\newlabel{[^}]\{1,\}}\).*/\1/' \
"$(TMPDIR)/$1.$$$$.make" | sort > '$2'; \
rm -f $(TMPDIR)/$1.$$$$.{sed.,}make
endef

# Makes an aux file that only has stuff relevant to the bbl in it
# $(call make-auxbbl-file,<aux file>,<new-aux>)
# -e '/^\\bibcite/p' is useless, looks like
define make-auxbbl-file
$(SED) \
-e '/^\\bibstyle/p' \
-e '/^\\citation/p' \
-e '/^\\bibdata/p' \
-e 'd' \
$1 | sort | uniq > $2
endef


define echo-texlog
$(ECHO) >>$(TMPDIR)/color_tex.sed
endef

# Colorize LaTeX output.
define make-color_tex
$(echo-texlog) '/^[[:space:]]*Output written/{'; \
$(echo-texlog) '  s/.*(\([^)]\{1,\}\)).*/Success!  Wrote \1/'; \
$(echo-texlog) '  s/[[:digit:]]\{1,\}/$(C_INFO)&$(C_RESET)/g'; \
$(echo-texlog) '  s/Success!/$(C_INFO)&$(C_RESET)/g'; \
$(echo-texlog) '  p'; \
$(echo-texlog) '  b end'; \
$(echo-texlog) '}'; \
$(echo-texlog) '/([^ (]*\.tex[^(]*([^ (]*\.tex/{'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)\([^(]*\)\(([^ (]*\.tex\)/$(C_INFO)\1$(C_RESET)\'; \
$(echo-texlog) '\3/'; \
$(echo-texlog) '  P'; \
$(echo-texlog) '  D'; \
$(echo-texlog) '}'; \
$(echo-texlog) '/([^ (]*\.tex/{'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)$$/$(C_INFO)\1$(C_RESET)\'; \
$(echo-texlog) '/'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)[ )]/$(C_INFO)\1$(C_RESET)\'; \
$(echo-texlog) '/'; \
$(echo-texlog) '  P'; \
$(echo-texlog) '  D'; \
$(echo-texlog) '}'; \
$(echo-texlog) 's/^! *LaTeX Error:.*/$(C_ERROR)&$(C_RESET)/p'; \
$(echo-texlog) 't'; \
$(echo-texlog) '/^LaTeX Warning:.*/b warningloop'; \
$(echo-texlog) '/^Underfull.*/b hbox'; \
$(echo-texlog) '/^Overfull.*/b hbox'; \
$(echo-texlog) '/^\#\#\#.*/b warningloop'; \
$(echo-texlog) 'b end'; \
$(echo-texlog) ''; \
$(echo-texlog) ': hbox'; \
$(echo-texlog) '  s/.*/$(C_WARNING)&$(C_RESET)/p'; \
$(echo-texlog) '  b end'; \
$(echo-texlog) ''; \
$(echo-texlog) ': warningloop'; \
$(echo-texlog) '  N'; \
$(echo-texlog) '  s/\(.*\n\)$$/&/'; \
$(echo-texlog) '  t warningdone'; \
$(echo-texlog) '  b warningloop'; \
$(echo-texlog) ': warningdone'; \
$(echo-texlog) '  s/.*/$(C_WARNING)&$(C_RESET)/p'; \
$(echo-texlog) ''; \
$(echo-texlog) ': end'; \
$(if $(VERBOSE),true,$(echo-texlog) 'd')
endef


# $(call latex-color-log,<LaTeX stem>)
define latex-color-log
if [ ! -f $(TMPDIR)/color_tex.sed ]; then touch $(TMPDIR)/color_tex.sed; $(call make-color_tex); fi; \
$(SED) -n -f $(TMPDIR)/color_tex.sed $1.log
endef

define echo-texerr
$(ECHO) >>$(TMPDIR)/latex-errors.sed
endef

define make-latex-errors
$(echo-texerr) '/^! LaTeX Error: File/d'; \
$(echo-texerr) '/^! LaTeX Error: Cannot determine size/d'; \
$(echo-texerr) '/[^ (]*\.tex/{'; \
$(echo-texerr) '  s/[^ (]*\.tex/$(C_ERROR)&$(C_RESET)/g'; \
$(echo-texerr) '  p'; \
$(echo-texerr) '}'; \
$(echo-texerr) '/^! /,/^$$/{'; \
$(echo-texerr) '  H'; \
$(echo-texerr) '  /^$$/{'; \
$(echo-texerr) '    x'; \
$(echo-texerr) '    s/^.*$$/$(C_ERROR)&$(C_RESET)/'; \
$(echo-texerr) '    p'; \
$(echo-texerr) '  }'; \
$(echo-texerr) '}'; \
$(echo-texerr) 'd'
endef


# Colorizes real, honest-to-goodness LaTeX errors that can't be
# overcome with recompilation.
#
# $(call latex-error-log,<LaTeX stem>)
define latex-error-log
if [ ! -f $(TMPDIR)/latex-errors.sed ]; then touch $(TMPDIR)/latex-errors.sed; $(call make-latex-errors); fi; \
$(SED) -f $(TMPDIR)/latex-errors.sed $1.log
endef

define echo-bibcol
$(ECHO) >>$(TMPDIR)/bibtex-color.sed
endef

# Colorize BibTeX output.
define make-bibtex-color
$(echo-bibcol) 's/^Warning--.*/$(C_WARNING)&$(C_RESET)/'; \
$(echo-bibcol) 't'; \
$(echo-bibcol) '/---/,/^.[^:]/{'; \
$(echo-bibcol) '  H'; \
$(echo-bibcol) '  /^.[^:]/{'; \
$(echo-bibcol) '    x'; \
$(echo-bibcol) '    s/\n\(.*\)/$(C_ERROR)\1$(C_RESET)/'; \
$(echo-bibcol) '    p'; \
$(echo-bibcol) '    s/.*//'; \
$(echo-bibcol) '    h'; \
$(echo-bibcol) '    d'; \
$(echo-bibcol) '  }'; \
$(echo-bibcol) '  d'; \
$(echo-bibcol) '}'; \
$(echo-bibcol) '/(.*error.*)/s//$(C_ERROR)&$(C_RESET)/'; \
$(if $(VERBOSE),true,$(echo-bibcol) 'd')
endef

# $(call bibtex-color-log,<LaTeX stem>)
define bibtex-color-log
if [ ! -f $(TMPDIR)/bibtex-color.sed ]; then touch $(TMPDIR)/bibtex-color.sed; $(call make-bibtex-color); fi; \
$(SED) -f $(TMPDIR)/bibtex-color.sed $1.blg
endef

# LaTeX invocations
#
# $(call run-latex,<stem>,<expected extension>)
define run-latex
$(ECHO) Running LaTeX; \
$(LATEX) $1 $(TODEVNULL); \
latexrunerror="$$?"; \
if [ -r pdflatex.fls ]; \
then mv -f pdflatex.fls "$1".fls; \
else if [ -r latex.fls ]; \
then mv -f latex.fls "$1".fls; \
fi; fi; \
if [ ! -d $(TMPDIR) ]; then mkdir $(TMPDIR); fi; \
mv "$1".fls $(TMPDIR); \
$(CP) "$1".log $(TMPDIR); \
$(call update_clean_file,$1) ;\
$(call update_purge_file,$1) ;\
if [ "$$latexrunerror" -ne 0 ]; then rm -f $2; $(call latex-error-log,$1); exit 1; fi
endef

# BibTeX invocations
#
# $(call run-bibtex,<tex stem>)
define run-bibtex 
$(ECHO) Running BibTeX on $1; \
$(BIBTEX) $1 $(TODEVNULL)
endef

# $(call test-run-again,<source stem>)
test-run-again	= egrep -q '^(.*Rerun .*|No file $1\.[^.]+\.|No file [^ ]+\.bbl\.|LaTeX Warning: There were undefined references\.)$$' $(TMPDIR)/$1.log

# $(call rerun,<source stem>,<produced dvi/ps/pdf file>,<step LaTeX compilation>)
define rerun
$(MAKE) -s LATEXSTEP=$3 FILE=$1 $2
endef

# $(call possibly-rerun,<source stem>,<produced dvi/ps/pdf file>,<step LaTeX compilation>)
define possibly-rerun
$(call test-run-again,$1); \
if [ "$$?" -eq 0 ]; then rm -f $2; $(call rerun,$1,$2,$3); \
else $(ECHO) \*\*\* LaTeX warnings and errors below \*\*\* ; \
$(call latex-color-log,$1); \
fi
endef

# Will create the stem.deps dependencies file
# $(call make-deps,<stem>)
define make-deps
  $(call get-inputs,$1,$1.$(EXT)); \
  $(call get-bbl-deps,$1,$1.$(EXT)); \
  $(call make-inds-deps,$1,$1.$(EXT))
endef

############################################################################

# Include if we're not cleaning
#ifeq "$(hascleangoals)" ""
-include $(TMPDIR)/$(FILE).deps
#endif

# Cancelling the dvi implicit rule
%.dvi: %.tex

.SECONDARY:

.PHONY: all FORCE help clean purge unsafe-purge

help:
	$(help_text)


ifndef FILE

FILES=$(shell egrep -l '[^%]*\\documentclass' *.tex)

all:
	$(QUIET)for f in $(FILES); \
	do \
	  echo \#\#\#\#\#\# Now compiling `basename $$f .tex`; \
	  $(MAKE) -s FILE=`basename $$f .tex` all; \
	done

clean:
	$(QUIET)for f in $(FILES); \
	do \
	  echo \#\#\#\#\#\# Now cleaning `basename $$f .tex`; \
	  $(MAKE) -s FILE=`basename $$f .tex` clean; \
	done

purge:
	$(QUIET)for f in $(FILES); \
	do \
	  echo \#\#\#\#\#\# Now purging `basename $$f .tex`; \
	  $(MAKE) -s FILE=`basename $$f .tex` purge; \
	done; \
	rm -f $(TMPDIR)/*; rmdir --ignore-fail-on-non-empty $(TMPDIR)

#TODO: ideally this last rm step should not be necessary

unsafe-purge:
	$(QUIET)for f in $(FILES); \
	do \
	  echo \#\#\#\#\#\# Now \(unsafely\) purging `basename $$f .tex`; \
	  echo \#\#\#\#\#\# You know what you do and/or you have backups ; \
	  $(MAKE) -s FILE=`basename $$f .tex` unsafe-purge; \
	done

else

all: $(FILE).$(EXT)

clean:
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).clean ]; \
	then cat $(TMPDIR)/$(FILE).clean | xargs rm -f ; \
	fi ; \
	rm -rf $(MORECLEAN)

purge: clean
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).purge ]; \
	then cat $(TMPDIR)/$(FILE).purge | xargs rm -f ;\
	fi ; \
	rmdir --ignore-fail-on-non-empty $(TMPDIR) ;\
	rm -rf $(MOREPURGE)

# This target has the potential of removing user files
# (use with caution)
unsafe-purge: purge
	$(QUIET)rm -f $(LATEXCLEAN) ;\
	rm -f $(LATEXPURGE) ;\
	rm -rf $(TMPDIR)

endif


.SUFFIXES: 
.SUFFIXES: .tex .dvi .ps .pdf .html .aux .deps .vars \
.idx .ind .gls .glo .ist .bbl \
.auxbbl .auxbbl.cookie \
.auxidx .auxidx.cookie \
.auxist .auxist.cookie \
.auxglo .auxglo.cookie

$(TMPDIR)/$(FILE).aux.flat: $(FILE).aux
	$(QUIET)$(call flatten-aux,$<,$@)

$(TMPDIR)/%.auxbbl: $(TMPDIR)/%.auxbbl.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

$(TMPDIR)/$(FILE).auxbbl.cookie: $(TMPDIR)/$(FILE).aux.flat
	$(QUIET)$(call make-auxbbl-file,$<,$@)

$(TMPDIR)/%.auxbbl.cookie: %.aux
	$(QUIET)$(call make-auxbbl-file,$<,$@)



$(TMPDIR)/%.auxidx.cookie: %.idx
	$(QUIET)$(CP) $< $@

$(TMPDIR)/%.auxidx: $(TMPDIR)/%.auxidx.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

$(TMPDIR)/%.auxglo.cookie: %.glo
	$(QUIET)$(CP) $< $@

$(TMPDIR)/%.auxist.cookie: %.ist
	$(QUIET)$(CP) $< $@

$(TMPDIR)/%.auxglo: $(TMPDIR)/%.auxglo.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

$(TMPDIR)/%.auxist: $(TMPDIR)/%.auxist.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

# LATEXSTEP tells us which step is _done_ (not about to be done)
# Steps: latex_init, latex_index, latex_refs, last step (compilation
# loop for cross-refs)

ifndef LATEXSTEP

%.bbl: %.aux
	$(QUIET)true

# We need to build the .deps only at the first step
# SC: not exactly...

# We force the re-generation : the user expects something to happen
# when typing "make", at the moment forcing a rebuild is the safe bet,
# dependency tracking in LaTeX being somewhat difficult with all the
# packages that rely on/build intermediate files.
$(FILE).$(EXT): FORCE
	$(QUIET)$(call run-latex,$*,$@); \
	$(call make-deps,$*); \
	$(ECHO) \#\#\#\#\#\# Was step: initial ; \
	$(call rerun,$*,$@,latex_init)

endif

ifeq ($(LATEXSTEP),latex_init)
# Index and glossaries should be done here
%.ind: $(TMPDIR)/%.auxidx
	$(QUIET)$(ECHO) Running makeindex for $@; \
	$(MAKEINDEX) $* ;\
	$(ECHO) $@ >>$(TMPDIR)/$(FILE).index-done

%.gls: $(TMPDIR)/%.auxglo $(TMPDIR)/%.auxist
	$(QUIET)$(ECHO) Running makeindex for glossary $@; \
	$(MAKEINDEX) -t $*.glg -o $@ -s $*.ist $*.glo ; \
	$(ECHO) $@ >>$(TMPDIR)/$(FILE).index-done

%.bbl: %.aux
	$(QUIET)true

# We loop here because page number for the glossary might change after
# the compilation, hence we have to generate again the .gls

$(FILE).$(EXT): FORCE
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).index-done ]; \
	then \
	  egrep '\\cite\>' `cat $(TMPDIR)/$(FILE).index-done` $(TODEVNULL); \
	  if [ $$? -eq 0 ]; then $(call run-latex,$*,$@); fi; \
	  rm $(TMPDIR)/$(FILE).index-done; \
	  $(ECHO) \#\#\#\#\#\# Was step: index, glossaries; \
	  $(call rerun,$(FILE),$@,latex_init); \
	else \
	  $(call rerun,$(FILE),$@,latex_index); \
	fi; 

endif

ifeq ($(LATEXSTEP),latex_index)

# Bibtex should be done here
%.bbl: $(TMPDIR)/%.auxbbl
	$(QUIET)$(call run-bibtex,$*); \
	if [ "$$?" -ne 0 ]; then $(call bibtex-color-log,$*); exit 1; fi; \
	$(call bibtex-color-log,$*); \
	touch $(TMPDIR)/$(FILE).bib-done

$(FILE).$(EXT): FORCE
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).bib-done ]; \
	then \
	  rm $(TMPDIR)/$(FILE).bib-done; \
	  $(call run-latex,$(FILE),$@); \
	  $(ECHO) \#\#\#\#\#\# Was step: bibliography; \
	fi; \
	$(call possibly-rerun,$(FILE),$@,latex_refs)


endif

ifeq ($(LATEXSTEP),latex_refs)

$(FILE).$(EXT): FORCE
	$(QUIET)cref_counter=0; \
	run_again=1; \
	while [ $$cref_counter -lt 4 -a $$run_again -eq 1 ]; \
	do \
	  $(call run-latex,$(FILE),$@); \
	  $(call test-run-again,$(FILE)); \
	  if [ "$$?" -ne 0 ]; then run_again=0; fi; \
	  cref_counter=`expr $$cref_counter \+ 1`; \
	done; \
	$(ECHO) \#\#\#\#\#\# Was step: cross-references; \
	if [ $$cref_counter -gt 2 ]; \
	then \
	  $(ECHO) \#\#\#\#\#\# $$cref_counter compilations needed just for cross-references; \
	  $(ECHO) \#\#\#\#\#\# This seems a lot... ; \
	  if [ $$run_again -eq 1 ]; \
	  then $(ECHO) \#\#\#\#\#\# Plus it seems to need to be run again ; \
	  fi ; \
	  $(ECHO) \#\#\#\#\#\# Either this document is a pathological case for cross-references, ; \
	  $(ECHO) \#\#\#\#\#\# Or you use a badly-programmed crossrefs-wise style file ; \
	  $(ECHO) \#\#\#\#\#\# Or simplier, there are undefined references; \
	fi; \
	$(call latex-color-log,$(FILE))

endif

FORCE:


%.html: %.$(EXT)
	$(HEVEA) $*.hva $*



define help_text
# This is a short usage description of this Makefile for LaTeX
# It should work under the following OSes:
# - Linux
# - MacOSX
# - Cygwin with LaTeX installed at the Windows level
# - Cygwin with LateX installed inside it (untested)
# - FreeBSD (with gmake)
# This Makefile can be included (untested).
#
# TARGETS
#
# help:
#   This help
#
# all:
#   It will compile every compilable document in the directory where
#   (g)make is launched, with automatic bibtex, index, glossaries
#   generation and as many times as necessary to solve references.
#   In a nutshell it will automatically do all for you.
#   By default, compilation is made with pdflatex (hence it produces a
#   pdf file), unless you put the following line into the main file
#   (the file where you declare \documentclass{...}:
#     % scolin-latex: dvi
#   Then it will use latex rather than pdflatex.
#   At the end of the compilation errors will be highlighted, in color
#   if your terminal emulator supports it. If there was an error, it
#   is stopped and will highlight errors as well.
#
# clean:
#   It will remove log files, intermediate files, etc. It will not
#   remove the compiled document (pdf or dvi), the generated
#   bibliographies (bbl), the generated index or glossary files, the
#   files in $(TMPDIR) needed for automation of the process
#
# purge:
#   It will "clean" then remove the unremoved files mentioned in the
#   clean target. This target should not remove important files.
#
# unsafe-purge:
#   It "purge"s everything and then tries to remove more, such as some
#   obscure intermediate files it had not found initially.
#   WARNING: this target is especially aggressive, use it only if:
#   - You have backups (or you have commited your work)
#   - You know what you are doing
#   And even then, this target might miss some useless files too
#
# <file>.html: (experimental)
#   Attempts an HeVeA compilation of the document to produce an html
#   file (HeVeA must be installed, of course)
#
# VARIABLES
#   You can change the following variables to suit certain constraints
#
# TMPDIR: the directory where automation files are stored
# MORECLEAN: mention additional files for the "clean" target
# MOREPURGE: same as above, for the "purge" target.
#   If "purge" or "clean" forgets some files you want to get rid of,
#   you should use those variables
#
# BUGS
#   - Many specific LaTeX packages not supported
#   - Log analysis still too verbose in case of used system-wide TeX
#   files (such as Tikz of the pgf suite)
#   - ...
# Report bugs at: <>
endef

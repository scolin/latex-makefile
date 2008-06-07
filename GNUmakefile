
# Copyright 2008 Samuel Colin
# Copyright 2004 Chris Monson (monpublic@gmail.com)

# This file is based on Chris Monson's Makefile:
# http://www.bouncingchairs.net/oss
# As such, parts derived directly from it are licenced under the 
# GPLv2 (http://www.gnu.org/copyleft/gpl.html)
# under the following terms, copied verbatim from the original Makefile
# 
# Parts I added are licenced GPLv2 as well

ifndef FILE
  FILE=$(basename $(firstword $(shell grep -l documentclass *.tex)))
  documentclass=$(shell grep documentclass $(FILE).tex)
else
  documentclass=$(shell grep documentclass $(FILE).tex)
endif

HEVEAFLAGS ?= -fix
BARELATEX ?= pdflatex
BIBFLAGS ?= -min-crossrefs=1
#VERBOSE := y

# PROGRAMS:
# Unix utilities with particular parameters
#SORT		:= LC_ALL=C sort
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
LATEXCLEAN = $(FILE).log $(FILE).out *.aux $(FILE).fls
LATEXCLEAN+= $(FILE).ps.tmp $(FILE).dvi.tmp $(FILE).pdf.tmp
LATEXCLEAN+= $(FILE).toc $(FILE).lof $(FILE).lot
LATEXCLEAN+= $(FILE).bbl $(FILE).bbl $(FILE).blg
LATEXCLEAN+= $(FILE).idx $(FILE).ind $(FILE).ilg
LATEXCLEAN+= $(FILE).glo $(FILE).gls $(FILE).ist $(FILE).glg
LATEXCLEAN+= $(FILE).nav $(FILE).snm  $(FILE).vrb 
LATEXCLEAN+= $(FILE).mtc* $(FILE).mlf* $(FILE).mlt* $(FILE).maf

LATEXCLEAN+= bu*.bbl bu*.aux bu*.blg
LATEXCLEAN+= *~
LATEXCLEAN+= $(MORECLEAN)

# MOREPURGE: see MORECLEAN, but for the purge target
LATEXPURGE = $(FILE).ps $(FILE).dvi $(FILE).pdf $(FILE).d
LATEXPURGE+= *.aux.make *.auxbbl.make *.auxdvi.make
LATEXPURGE+= $(MOREPURGE)


############################################################################

#
# Utility Functions and Definitions
#

test-different		= ! diff -q '$1' '$2' &>/dev/null

replace-if-different-and-remove	= \
	$(call test-different,$1,$2) && mv -f '$1' '$2' || rm -f '$1'

# Outputs all source dependencies to stdout.  The first argument is the file to
# be parsed, the second is a list of files that will show up as dependencies in
# the new .d file created here.
#
# NOTE: BSD sed does not understand \|, so we have to do something more
# clunky to extract suitable extensions.
#
# $(call get-inputs,<parsed file>,<target files>)
define get-inputs
sed \
-e '/^INPUT/!d' \
-e 's!^INPUT \(\./\)\{0,1\}!$2: !' \
-e '/\.tex$$/p' \
-e '/\.cls$$/p' \
-e '/\.sty$$/p' \
-e '/\.png$$/p' \
-e '/\.jpg$$/p' \
-e 'd' \
$1 | sort | uniq
endef

# $(call get-bbl-deps,<stem>,,<targets>,<out file>)
# We exploit the fact that a bbl appearing in the .fls is not "No file" in the .log, and vice-versa
define get-bbl-deps
bblstems1=`sed -e '/^No file \(.*\.bbl\)\./!d' -e 's/No file \(.*\)\.bbl\./\1/g' $1.log | sort | uniq`; \
bblstems2=`sed -e '/^INPUT.*\.bbl$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\)\.bbl$$!\2!' $1.fls | sort | uniq`; \
bblstems="$$bblstems1 $$bblstems2"; \
if [ ! -f $3 ]; then touch $3; fi; \
for i in $$bblstems; \
do \
  echo $2: $$i.bbl >>$3; \
  sed \
  -e '/^\\bibdata/!d' \
  -e 's/\\bibdata{\([^}]*\)}/\1,/' \
  -e 's/,\{2,\}/,/g' \
  -e 's/,/.bib /g' \
  -e 's/ \{1,\}$$//' \
  $$i.aux | xargs $(KPSEWHICH) - | \
  sed -e "s/^/$$i.bbl: /" | \
  sort | uniq >>$3; \
done
endef

# Compute index and glossary dependencies
# $(call make-inds-deps,<stem>,<targets>,<out file>)
define make-inds-deps
indstems1=`sed -e '/^No file \(.*\.ind\)\./!d' -e 's/No file \(.*\.ind\)\./\1/g' $1.log | sort | uniq`; \
indstems2=`sed -e '/^INPUT.*\.ind$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.ind\)$$!\2!' $1.fls | sort | uniq`; \
glsstems1=`sed -e '/^No file \(.*\.gls\)\./!d' -e 's/No file \(.*\.gls\)\./\1/g' $1.log | sort | uniq`; \
glsstems2=`sed -e '/^INPUT.*\.gls$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.gls\)$$!\2!' $1.fls | sort | uniq`; \
indstems="$$indstems1 $$indstems2"; \
glsstems="$$glsstems1 $$glsstems2"; \
if [ ! -f $3 ]; then touch $3; fi; \
if [ "x$$indstems" != "x " ]; then echo $2: $$indstems >>$3; fi; \
if [ "x$$glsstems" != "x " ]; then echo $2: $$glsstems >>$3; fi
endef

# Makes a an aux file that only has stuff relevant to the dvi in it
# $(call make-auxdvi-file,<flattened-aux>,<new-aux>)
define make-auxdvi-file
sed \
-e '/^\\newlabel/!d' \
$1 > $2
endef

# Makes an aux file that only has stuff relevant to the bbl in it
# $(call make-auxbbl-file,<flattened-aux>,<new-aux>)
define make-auxbbl-file
sed \
-e '/^\\newlabel/d' \
$1 > $2
endef


# Colorizes real, honest-to-goodness LaTeX errors that can't be
# overcome with recompilation.
#
# $(call colorize-latex-errors,<log file>)
define colorize-latex-errors
sed \
-e '/^! LaTeX Error: File/d' \
-e '/^! LaTeX Error: Cannot determine size/d' \
-e '/^! /,/^$$/{' \
-e '  H' \
-e '  /^$$/{' \
-e '    x' \
-e '    s/^.*$$/$(C_ERROR)&$(C_RESET)/' \
-e '    p' \
-e '  }' \
-e '}' \
-e 'd' \
$1
endef


# Get all important .aux files from the top-level .aux file and merges
# them all into a single file, which it outputs to stdout.
#
# $(call flatten-aux,<toplevel aux>,<output file>)
define flatten-aux
sed \
-e '/\\@input{\(.*\)}/{' \
-e     's//\1/' \
-e     'h' \
-e     's!.*!\\:\\@input{&}:{!' \
-e     'p' \
-e     'x' \
-e     's/.*/r &/p' \
-e     's/.*/d/p' \
-e     's/.*/}/p' \
-e     'd' \
-e '}' \
-e 'd' \
'$1' > "$1.$$$$.sed.make"; \
sed -f "$1.$$$$.sed.make" '$1' > "$1.$$$$.make"; \
sed \
-e '/^\\relax/d' \
-e '/^\\bibcite/d' \
-e 's/^\(\\newlabel{[^}]\{1,\}}\).*/\1/' \
"$1.$$$$.make" | sort > '$2'; \
rm -f $1.$$$$.{sed.,}make
endef

# Colorize LaTeX output.
color_tex	:= \
	sed \
	-e '/^[[:space:]]*Output written/{' \
	-e ' s/.*(\([^)]\{1,\}\)).*/Success!  Wrote \1/' \
	-e ' s/[[:digit:]]\{1,\}/$(C_INFO)&$(C_RESET)/g' \
	-e ' s/Success!/$(C_INFO)&$(C_RESET)/g' \
	-e '}' \
	-e 's/^! *LaTeX Error:.*/$(C_ERROR)&$(C_RESET)/' -e 't' \
	-e 's/^LaTeX Warning:.*/$(C_WARNING)&$(C_RESET)/' -e 't' \
	-e 's/^Underfull.*/$(C_WARNING)&$(C_RESET)/' -e 't' \
	-e 's/^Overfull.*/$(C_WARNING)&$(C_RESET)/' -e 't' \
	$(if $(VERBOSE),,-e 'd')

# Colorize BibTeX output.
color_bib	:= \
	sed \
	-e 's/^Warning--.*/$(C_WARNING)&$(C_RESET)/' -e 't' \
	-e '/---/,/^.[^:]/{' \
	-e '  H' \
	-e '  /^.[^:]/{' \
	-e '    x' \
	-e '    s/\n\(.*\)/$(C_ERROR)\1$(C_RESET)/' \
	-e '	p' \
	-e '    s/.*//' \
	-e '    h' \
	-e '    d' \
	-e '  }' \
	-e '  d' \
	-e '}' \
	-e '/(.*error.*)/s//$(C_ERROR)&$(C_RESET)/' \
	$(if $(VERBOSE),,-e 'd')


# $(call latex-color-log,<LaTeX stem>)
latex-color-log	= $(color_tex) $1.log

# $(call latex-error-log,<LaTeX stem>)
latex-error-log	= $(call colorize-latex-errors,$1.log)

# $(call bibtex-color-log,<LaTeX stem>)
bibtex-color-log	= $(color_bib) $1.blg

# LaTeX invocations
#
# $(call latex,<tex file>)
run-latex	= $(LATEX) $1 > /dev/null

# BibTeX invocations
#
# $(call run-bibtex,<tex stem>)
run-bibtex = $(BIBTEX) $1 > /dev/null

# $(call test-run-again,<source stem>)
test-run-again	= egrep -q '^(.*Rerun .*|No file $1\.[^.]+\.)$$' $1.log

# $(call possibly-rerun,<source stem>,<produced dvi/ps/pdf file>,<step LaTeX compilation>)
define possibly-rerun
$(call test-run-again,$1); \
if [ "$$?" -eq 0 ]; then mv $(2) $(2).tmp; $(MAKE) LATEXSTEP=$(3) $(2); \
else if [ -f $(2).tmp ]; then rm $(2).tmp; fi; $(call latex-color-log,$1); fi
endef

# Will create the stem.d dependencies file
# $(call make-deps,<stem>)
define make-deps
  $(call get-inputs,$1.fls,$(addprefix $1.,fls aux pdf)) > $1.d.cookie; \
  $(call get-bbl-deps,$1,$(addprefix $1.,pdf),$1.d.cookie); \
  $(call make-inds-deps,$1,$(addprefix $1.,pdf),$1.d.cookie); \
  $(call replace-if-different-and-remove,$1.d.cookie,$1.d) && touch $1.d
endef

############################################################################

# Cancelling the dvi implicit rule
%.dvi: %.tex

# .SECONDARY: $(FILE).dvi
# .SECONDARY: $(FILE).fls $(FILE).aux
# .SECONDARY: $(FILE).aux.make
# .SECONDARY: $(FILE).auxdvi.make
# .SECONDARY: $(FILE).auxbbl.make
.SECONDARY:

.PHONY: all
all: $(FILE).pdf

# Include if we're not cleaning
ifeq "$(hascleangoals)" ""
-include $(FILE).d
endif

.PHONY: clean
clean:
	$(QUIET)rm -f $(LATEXCLEAN)

.PHONY: purge
purge: clean
	$(QUIET)rm -f $(LATEXPURGE)

.SUFFIXES:
.SUFFIXES: .done .tex .dvi .ps .pdf .html .aux .d .stamp .idx .ind .toc .lof .lot .bbl

%.aux %.fls: %.tex
	$(QUIET)$(call run-latex,$*)

%.aux.make: %.aux.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

%.auxdvi.make: %.auxdvi.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)

%.auxbbl.make: %.auxbbl.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@)


%.aux.make.cookie: %.aux
	$(QUIET)$(call flatten-aux,$<,$@)

%.auxdvi.make.cookie: %.aux.make
	$(QUIET)$(call make-auxdvi-file,$<,$@)

%.auxbbl.make.cookie: %.aux.make
	$(QUIET)$(call make-auxbbl-file,$<,$@)

# LATEXSTEP tells us which step is _done_ (not about to be done)
# Steps: latex_init, latex_index, latex_refs, latex_links => 5 steps

ifndef LATEXSTEP

# We need to build the .d only at the first step

%.bbl: %.auxbbl.make
	$(QUIET)true

%.pdf:
	$(QUIET)echo Step: initial
	$(QUIET)$(call run-latex,$*); \
	if [ "$$?" -ne 0 ]; then $(call latex-error-log,$*); exit 1; \
	else $(call make-deps,$*); $(call test-run-again,$*); \
	if [ "$$?" -eq 0 ]; then rm $@; $(MAKE) LATEXSTEP=latex_init FILE=$* $@; else $(call latex-color-log,$*); fi; \
	fi

endif

ifeq ($(LATEXSTEP),latex_init)
# Index and glossaries should be done here
%.ind: %.idx
	$(QUIET)$(MAKEINDEX) $*

%.gls: %.glo %.ist
	$(QUIET)$(MAKEINDEX) -t $*.glg -o $@ -s $*.ist $<

%.bbl: %.auxbbl.make
	$(QUIET)true

%.pdf:
	$(QUIET)echo Step: index, glossaries
	$(QUIET)$(call run-latex,$*); \
	if [ "$$?" -ne 0 ]; then $(call latex-error-log,$*); exit 1; \
	else $(call test-run-again,$*); \
	if [ "$$?" -eq 0 ]; then rm $@; $(MAKE) LATEXSTEP=latex_index FILE=$* $@; else $(call latex-color-log,$*); fi; \
	fi

endif

ifeq ($(LATEXSTEP),latex_index)

# Bibtex should be done here
%.bbl: %.auxbbl.make
	$(QUIET)$(call run-bibtex,$*); \
	if [ "$$?" -ne 0 ]; then $(call bibtex-color-log,$*); exit 1; fi

%.pdf:
	$(QUIET)echo Step: bibliography
	$(QUIET)$(call run-latex,$*); \
	if [ "$$?" -ne 0 ]; then $(call latex-error-log,$*); exit 1; \
	else $(call test-run-again,$*); \
	if [ "$$?" -eq 0 ]; then rm $@; $(MAKE) LATEXSTEP=latex_refs FILE=$* $@; else $(call latex-color-log,$*); fi; \
	fi

endif

ifeq ($(LATEXSTEP),latex_refs)

%.pdf:
	$(QUIET)echo Step: cross-references
	$(QUIET)$(call run-latex,$*); \
	if [ "$$?" -ne 0 ]; then $(call latex-error-log,$*); exit 1; \
	else $(call test-run-again,$*); \
	if [ "$$?" -eq 0 ]; then rm $@; $(MAKE) LATEXSTEP=latex_links FILE=$* $@; else $(call latex-color-log,$*); fi; \
	fi

endif

ifeq ($(LATEXSTEP),latex_links)

%.pdf:
	$(QUIET)echo Step: last chance to solve undefined references
	$(QUIET)$(call run-latex,$*); \
	if [ "$$?" -ne 0 ]; then $(call latex-error-log,$*); exit 1; else $(call latex-color-log,$*); fi

endif


# Dependencies of %.fls and %.aux will actually go into %.d



%.html: %.pdf
	$(HEVEA) $*.hva $<


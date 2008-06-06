
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

HEVEAFLAGS ?= -fixpoint
BARELATEX ?= pdflatex
BIBFLAGS ?= -min-crossrefs=1
#VERBOSE := y

# EXTERNAL PROGRAMS:
# = ESSENTIAL PROGRAMS =
# == Basic Shell Utilities ==
CAT		:= cat
CP		:= cp -f
DIFF		:= diff
ECHO		:= echo
EGREP		:= egrep
ENV		:= env
MV		:= mv -f
RM		:= rm -f
SED		:= sed
SORT		:= sort
TOUCH		:= touch
UNIQ		:= uniq
WHICH		:= which
XARGS		:= xargs
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
# == Beamer Enlarged Output ==
PSNUP		?= psnup
# == Viewing Stuff ==
VIEW_POSTSCRIPT	?= gv
VIEW_PDF	?= xpdf
VIEW_GRAPHICS	?= display

# Characters that are hard to specify in certain places
space		:= $(empty) $(empty)
colon		:= \:
comma		:= ,

# Useful shell definitions
# TODO: bashisms !!!
sh_true		:= :
sh_false	:= ! :

# Turn command echoing back on with VERBOSE=1
ifndef VERBOSE
QUIET	:= @
endif

# Turn on shell debugging with SHELL_DEBUG=1
# (EVERYTHING is echoed, even $(shell ...) invocations)
ifdef SHELL_DEBUG
SHELL	+= -x
endif

# Get the name of this makefile (always right in 3.80, often right in 3.79)
# This is only really used for documentation, so it isn't too serious.
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

# Check that clean targets are not combined with other targets (weird things
# happen, and it's not easy to fix them)
hascleangoals	:= $(if $(sort $(filter clean purge,$(MAKECMDGOALS))),1)
hasbuildgoals	:= $(if $(sort $(filter-out clean purge,$(MAKECMDGOALS))),1)
ifneq "$(hasbuildgoals)" ""
ifneq "$(hascleangoals)" ""
$(error $(C_ERROR)Clean and build targets specified together$(C_RESET)))
endif
endif

# Extensions generated by LaTeX invocation that can be removed when complete
rm_ext		:= log,aux,dvi,ps,pdf,blg,bbl,out,nav,snm,toc,lof,lot,pfg,fls,vrb
backup_patterns	:= *.bak

# MORECLEAN is specified by the user, if he wants to remove additional
# files when cleaning
LATEXCLEAN = $(FILE).log $(FILE).out *.aux $(FILE).fls
LATEXCLEAN+= $(FILE).toc $(FILE).lof $(FILE).lot
LATEXCLEAN+= $(FILE).bbl $(FILE).bbl $(FILE).blg
LATEXCLEAN+= $(FILE).idx $(FILE).ind $(FILE).ilg
LATEXCLEAN+= $(FILE).glo $(FILE).gls $(FILE).ist $(FILE).glg
LATEXCLEAN+= $(FILE).mtc* $(FILE).mlf* $(FILE).mlt* $(FILE).maf

LATEXCLEAN+= bu*.bbl bu*.aux bu*.blg
LATEXCLEAN+= $(MORECLEAN)

# MOREPURGE: see MORECLEAN, but for the purge target
LATEXPURGE = $(FILE).ps $(FILE).dvi $(FILE).pdf $(FILE).d
LATEXPURGE+= *.aux.make *.auxbbl.make *.auxdvi.make
LATEXPURGE+= $(MOREPURGE)


############################################################################

#
# Utility Functions and Definitions
#

test-exists		= [ -e '$1' ]
test-different		= ! $(DIFF) -q '$1' '$2' &>/dev/null

copy-if-different	= $(call test-different,$1,$2) && $(CP) '$1' '$2'
copy-if-exists		= $(call test-exists,$1) && $(CP) '$1' '$2'
move-if-different	= $(call test-different,$1,$2) && $(MV) '$1' '$2'
replace-if-different-and-remove	= \
	$(call test-different,$1,$2) && $(MV) '$1' '$2' || $(RM) '$1'

# Note that $(DIFF) returns success when the files are the SAME....
# $(call test-different,sfile,dfile)
test-exists-and-different	= \
	$(call test-exists,$2) && $(call test-different,$1,$2)

get-default	= $(if $1,$1,$2)

# Gives a reassuring message about the failure to find include files
# $(call include-message,<list of include files>)
define include-message
$(strip \
$(if $(filter-out $(wildcard $1),$1),\
	$(shell $(ECHO) \
	"$(C_INFO)NOTE: You may ignore warnings about the"\
	"following files:" >&2;\
	$(ECHO) >&2; \
	$(foreach s,$(filter-out $(wildcard $1),$1),$(ECHO) '     $s' >&2;)\
	$(ECHO) "$(C_RESET)" >&2)
))
endef

# Outputs all source dependencies to stdout.  The first argument is the file to
# be parsed, the second is a list of files that will show up as dependencies in
# the new .d file created here.
#
# NOTE: BSD sed does not understand \|, so we have to do something more
# clunky to extract suitable extensions.
#
# $(call get-inputs,<parsed file>,<target files>)
define get-inputs
$(SED) \
-e '/^INPUT/!d' \
-e 's!^INPUT \(\./\)\{0,1\}!$2: !' \
-e '/\.tex$$/p' \
-e '/\.cls$$/p' \
-e '/\.sty$$/p' \
-e 'd' \
$1 | $(SORT) | $(UNIQ)
endef

# $(call get-bbls,<stem>,<targets>)
define get-bbls
$(SED) -e '/^No file \(.*\.bbl\)\./!d' -e 's/No file \(.*\.bbl\)\./\1/g' $1.log | $(SORT) | $(UNIQ) | \
$(SED) -e 's/^/$2: /g'
endef


# $(call get-bbl-deps,<stem>,,<targets>,<out file>)
# We exploit the fact that a bbl appearing in the .fls is not "No file" in the .log, and vice-versa
define get-bbl-deps
bblstems1=`$(SED) -e '/^No file \(.*\.bbl\)\./!d' -e 's/No file \(.*\)\.bbl\./\1/g' $1.log | $(SORT) | $(UNIQ)`; \
bblstems2=`$(SED) -e '/^INPUT.*\.bbl$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\)\.bbl$$!\2!' $1.fls | $(SORT) | $(UNIQ)`; \
bblstems="$$bblstems1 $$bblstems2"; \
if [ ! -f $3 ]; then touch $3; fi; \
for i in $$bblstems; \
do \
  echo $2: $$i.bbl >>$3; \
  $(SED) \
  -e '/^\\bibdata/!d' \
  -e 's/\\bibdata{\([^}]*\)}/\1,/' \
  -e 's/,\{2,\}/,/g' \
  -e 's/,/.bib /g' \
  -e 's/ \{1,\}$$//' \
  $$i.aux | $(XARGS) $(KPSEWHICH) - | \
  $(SED) -e "s/^/$$i.bbl: /" | \
  $(SORT) | $(UNIQ) >>$3; \
done
endef

# TODO: analyse the .fls too...
# $(call get-inds,<stem>,<targets>)
define get-inds
$(SED) -e '/^No file \(.*\.ind\)\./!d' -e 's/No file \(.*\.ind\)\./\1/g' $1.log | $(SORT) | $(UNIQ) | \
$(SED) -e 's/^/$2: /g'
endef

# Compute index and glossary dependencies
# $(call make-inds-deps,<stem>,<targets>,<out file>)
define make-inds-deps
indstems1=`$(SED) -e '/^No file \(.*\.ind\)\./!d' -e 's/No file \(.*\.ind\)\./\1/g' $1.log | $(SORT) | $(UNIQ)`; \
indstems2=`$(SED) -e '/^INPUT.*\.ind$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.ind\)$$!\2!' $1.fls | $(SORT) | $(UNIQ)`; \
glsstems1=`$(SED) -e '/^No file \(.*\.gls\)\./!d' -e 's/No file \(.*\.gls\)\./\1/g' $1.log | $(SORT) | $(UNIQ)`; \
glsstems2=`$(SED) -e '/^INPUT.*\.gls$$/!d' -e 's!^INPUT \(\./\)\{0,1\}\(.*\.gls\)$$!\2!' $1.fls | $(SORT) | $(UNIQ)`; \
indstems="$$indstems1 $$indstems2"; \
glsstems="$$glsstems1 $$glsstems2"; \
if [ ! -f $3 ]; then touch $3; fi; \
if [ "x$$indstems" != "x " ]; then echo $2: $$indstems >>$3; fi; \
if [ "x$$glsstems" != "x " ]; then echo $2: $$glsstems >>$3; fi
endef

# Makes a an aux file that only has stuff relevant to the dvi in it
# $(call make-auxdvi-file,<flattened-aux>,<new-aux>)
define make-auxdvi-file
$(SED) \
-e '/^\\newlabel/!d' \
$1 > $2
endef

# Makes an aux file that only has stuff relevant to the bbl in it
# $(call make-auxbbl-file,<flattened-aux>,<new-aux>)
define make-auxbbl-file
$(SED) \
-e '/^\\newlabel/d' \
$1 > $2
endef

# Colorizes real, honest-to-goodness LaTeX errors that can't be overcome with
# recompilation.
#
# $(call colorize-latex-errors,<log file>)
define colorize-latex-errors
$(SED) \
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

# Get all important .aux files from the top-level .aux file and merges them all
# into a single file, which it outputs to stdout.
#
# $(call flatten-aux,<toplevel aux>,<output file>)
define flatten-aux
$(SED) \
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
$(SED) -f "$1.$$$$.sed.make" '$1' > "$1.$$$$.make"; \
$(SED) \
-e '/^\\relax/d' \
-e '/^\\bibcite/d' \
-e 's/^\(\\newlabel{[^}]\{1,\}}\).*/\1/' \
"$1.$$$$.make" | $(SORT) > '$2'; \
$(RM) $1.$$$$.{sed.,}make
endef

# Generate pdf from postscript
ps2pdf_normal	:= $(PS2PDF_NORMAL)
ps2pdf_embedded	:= \
	$(PS2PDF_EMBED) \
	-dPDFSETTINGS=/printer \
	-dEmbedAllFonts=true \
	-dSubsetFonts=true \
	-dMaxSubsetPct=100

# Colorize LaTeX output.
color_tex	:= \
	$(SED) \
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
	$(SED) \
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

# Make beamer output big enough to print on a full page.  Landscape doesn't
# seem to work correctly.
enlarge_beamer	= $(PSNUP) -l -1 -W128mm -H96mm -pletter

# $(call test-run-again,<source stem>)
test-run-again	= $(EGREP) -q '^(.*Rerun .*|No file $1\.[^.]+\.)$$' $1.log

# This tests whether the dvi target should be run at all, from viewing the log
# file.
# $(call test-log-for-need-to-run,<source stem>)
define test-log-for-need-to-run
$(SED) \
-e '/^No file $(subst .,\\.,$1)\.aux\./d' \
$1.log \
| $(EGREP) -q '^(.*Rerun .*|No file $1\.[^.]+\.|LaTeX Warning: File.*)$$'
endef

STILLNOT = grep "LaTeX Warning: \(Label(s) may have changed. Rerun to get cross-references right\|There were undefined references\)." 

ONEMORETIME = $(STILLNOT) $(1); if [ "$$?" -eq 0 ]; then mv $(2) $(2).tmp; $(MAKE) LATEXSTEP=$(3); else if [ -f $(2).tmp ]; then rm $(2).tmp; fi; fi

# $(call possibly-rerun,<source stem>,<produced dvi/ps/pdf file>,<step LaTeX compilation>)
define possibly-rerun
$(call test-run-again,$1); \
if [ "$$?" -eq 0 ]; then mv $(2) $(2).tmp; $(MAKE) LATEXSTEP=$(3) $(2); \
else if [ -f $(2).tmp ]; then rm $(2).tmp; fi; $(call latex-color-log,$1); fi
endef


# LaTeX invocations
#
# $(call latex,<tex file>)
run-latex	= $(LATEX) $1 > /dev/null

# BibTeX invocations
#
# $(call run-bibtex,<tex stem>)
run-bibtex	= $(BIBTEX) $1 | $(color_bib)


# Convert DVI to Postscript
# $(call make-ps,<dvi file>,<ps file>,<log file>,[<paper size>])
make-ps		= \
	$(DVIPS) -o '$2' $(if $(filter-out BEAMER,$4),-t$(firstword $4),) '$1' \
		$(if $(filter BEAMER,$4),| $(enlarge_beamer)) &> $3

# Convert Postscript to PDF
# $(call make-pdf,<ps file>,<pdf file>,<log file>,<embed file>)
make-pdf	= \
	$(if $(filter 1,$(shell $(CAT) '$4')),\
		$(ps2pdf_embedded),\
		$(ps2pdf_normal)) '$1' '$2' &> $3

# Display information about what is being done
# $(call echo-build,<output file>,[<run number>])
echo-build	= $(ECHO) "$(C_INFO)= $1 --> $2$(if $3, ($3),) =$(C_RESET)"
echo-dep	= $(ECHO) "$(C_INFO)= $1 --> $2 =$(C_RESET)"

# Display a list of something
# $(call echo-list,<values>)
echo-list	= for x in $1; do $(ECHO) "$$x"; done

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
-include $(FILE).d $(call include-message,$(FILE).d)
endif

.PHONY: clean
clean:
	$(QUIET)$(RM) $(LATEXCLEAN)

.PHONY: purge
purge: clean
	$(QUIET)$(RM) $(LATEXPURGE)

.SUFFIXES:
.SUFFIXES: .done .tex .dvi .ps .pdf .html .aux .d .stamp .idx .ind .toc .lof .lot .bbl

%.aux %.fls: %.tex
	$(QUIET)$(call run-latex,$*)

%.aux.make: %.aux.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@) && touch $@

%.auxdvi.make: %.auxdvi.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@) && touch $@

%.auxbbl.make: %.auxbbl.make.cookie
	$(QUIET)$(call replace-if-different-and-remove,$<,$@) && touch $@


%.aux.make.cookie: %.aux
	$(QUIET)$(call flatten-aux,$<,$@)

%.auxdvi.make.cookie: %.aux.make
	$(QUIET)$(call make-auxdvi-file,$<,$@)

%.auxbbl.make.cookie: %.aux.make
	$(QUIET)$(call make-auxbbl-file,$<,$@)

# LATEXSTEP tells us which step _is done_ (not about to be done)
# Steps: latex_init, latex_refs, latex_links => 4 steps

ifndef LATEXSTEP

# We need to build the .d only at the first step

%.pdf %.d: %.fls %.aux
	$(QUIET)echo Step 1; \
	$(call get-inputs,$*.fls,$(addprefix $*.,fls aux pdf)) > $*.d.cookie; \
	$(call get-bbl-deps,$*,$(addprefix $*.,pdf),$*.d.cookie); \
	$(call make-inds-deps,$*,$(addprefix $*.,pdf),$*.d.cookie); \
	$(call replace-if-different-and-remove,$*.d.cookie,$*.d) && touch $*.d; \
	$(call possibly-rerun,$*,$*.pdf,latex_init)

endif

ifeq ($(LATEXSTEP),latex_init)
# Index and glossaries should be done here
%.ind: %.idx
	$(QUIET)$(MAKEINDEX) $*

%.gls: %.glo %.ist
	$(QUIET)$(MAKEINDEX) -t $*.glg -o $@ -s $*.ist $<

%.pdf:
	$(QUIET)echo Step 2; \
	$(call run-latex,$*); \
	touch $*.d; \
	$(call possibly-rerun,$*,$@,latex_index)


endif

ifeq ($(LATEXSTEP),latex_index)

# Bibtex should be done here
%.bbl: %.auxbbl.make
	$(QUIET)$(call run-bibtex,$*)

%.pdf:
	$(QUIET)echo Step 3; \
	$(call run-latex,$*); \
	touch $*.d; \
	$(call possibly-rerun,$*,$@,latex_refs)


endif

ifeq ($(LATEXSTEP),latex_refs)

%.pdf:
	$(QUIET)echo Step 4; \
	$(call run-latex,$*); \
	touch $*.d; \
	$(call possibly-rerun,$*,$@,latex_links)


endif

ifeq ($(LATEXSTEP),latex_links)

%.pdf:
	$(QUIET)echo Step 5; \
	$(call run-latex,$*); \
	touch $*.d; \
	rm -f $@.tmp

endif


# Dependencies of %.fls and %.aux will actually go into %.d



%.html: %.pdf
	$(HEVEA) $<

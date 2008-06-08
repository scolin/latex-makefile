
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
  FILES=$(shell egrep -l '[^%]*\\documentclass' *.tex)
  FILE=$(basename $(firstword $(FILES)))
  documentclass=$(shell grep documentclass $(FILE).tex)
else
  documentclass=$(shell grep documentclass $(FILE).tex)
endif

TMPDIR=._d

HEVEAFLAGS ?= -fix
BARELATEX ?= pdflatex
BIBFLAGS ?= -min-crossrefs=1
#VERBOSE := y

# PROGRAMS:
# Unix utilities with particular parameters
#SORT		:= LC_ALL=C sort
SED		:= sed
CP		:= cp -RpP
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
LATEXCLEAN+= $(FILE).mtc* $(FILE).mlf* $(FILE).mlt* $(FILE).maf
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
if [ ! -f "$1" ]; then echo "$1" should exist; exit 1; fi; \
if [ ! -f "$2" ]; then mv -f "$1" "$2"; \
else diff -q "$1" "$2" &>/dev/null; \
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
if [ ! -f $(TMPDIR)/get-inputs.sed ]; then $(call make-get-inputs,$(TMPDIR)/get-inputs.sed); fi ; \
$(SED) -f $(TMPDIR)/get-inputs.sed $(TMPDIR)/$1.fls | \
$(SED) -e 's/^.*$$/$2: &/' | \
sort | uniq >$(TMPDIR)/$1.deps
endef

define make-get-inputs
echo '/^INPUT/!d' >$1 ; \
echo 's!^INPUT !!' >>$1 ; \
echo '/^\/.*$$/d' >>$1 ; \
echo '/^[a-zA-Z]:/d' >>$1 ; \
echo '/\.aux$$/d' >>$1 ; \
echo '/\.bbl$$/d' >>$1 ; \
echo '/\.ind$$/d' >>$1 ; \
echo '/\.gls$$/d' >>$1 ; \
echo '/\.nav$$/d' >>$1 ; \
echo '/\.toc$$/d' >>$1 ; \
echo '/\.mtc[0-9]*$$/d' >>$1 ; \
echo '/\.lof$$/d' >>$1 ; \
echo '/\.mlf[0-9]*$$/d' >>$1 ; \
echo '/\.lot$$/d' >>$1 ; \
echo '/\.mlt[0-9]*$$/d' >>$1 ; \
echo '/\.out$$/d' >>$1
endef


# $(call update_file,<filename>)
define update_file
if [ -f $1 ]; then cat $1 >>$1.cookie; fi ; \
sort $1.cookie | uniq >$1; \
rm $1.cookie
endef

# In purge : dvi,ps,pdf
# In clean : maf,ilg,glg,blg,out,log
#            lot,lof,toc,mlt*,mlf*,mtc*,nav
#            glo,gls,idx,ind,ist,aux

# This function will get the generated files and add them to the
# cleaning lists 
# $(call update_clean_file,<stem>)
# 
define update_clean_file
egrep '^OUTPUT' $(TMPDIR)/$1.fls | \
$(SED) -e 's/^OUTPUT //' | \
egrep '\.maf$$|\.ilg$$|\.glg$$|\.blg$$|\.out$$|\.log$$|\.lot$$|\.lof$$|\.toc$$|\.mlt[0-9]*$$|\.mlf[0-9]*$$|\.mtc[0-9]*$$|\.nav$$|\.snm$$|\.glo$$|\.gls$$|\.idx$$|\.ind$$|\.ist$$|\.aux$$' \
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
  echo $2: $$i.bbl >>$(TMPDIR)/$1.deps; \
  $(SED) \
  -e '/^\\bibdata/!d' \
  -e 's/\\bibdata{\([^}]*\)}/\1,/' \
  -e 's/,\{2,\}/,/g' \
  -e 's/,/.bib /g' \
  -e 's/ \{1,\}$$//' \
  $$i.aux | xargs $(KPSEWHICH) - | \
  $(SED) -e "s/^/$$i.bbl: /" | \
  sort | uniq >>$(TMPDIR)/$1.deps; \
  echo $$i.aux >>$(TMPDIR)/$1.clean ;\
  echo $$i.blg >>$(TMPDIR)/$1.clean ;\
  echo $$i.bbl >>$(TMPDIR)/$1.purge ;\
  echo $(TMPDIR)/$$i.auxbbl >>$(TMPDIR)/$1.purge ;\
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
  echo $2: $$indstems >>$(TMPDIR)/$1.deps; \
  echo $$indstems >>$(TMPDIR)/$1.purge; \
  echo $$indstems | $(SED) -e 's/\.ind\( \|$$\)/.ilg\1/g' >>$(TMPDIR)/$1.clean ;\
fi; \
if [ "x$$glsstems" != "x " ]; \
then \
  echo $2: $$glsstems >>$(TMPDIR)/$1.deps; \
  echo $$glsstems >>$(TMPDIR)/$1.purge; \
  echo $$glsstems | $(SED) -e 's/\.gls\( \|$$\)/.glg\1/g' >>$(TMPDIR)/$1.clean ;\
fi ;\
$(call trim,$(TMPDIR)/$1.clean) ;\
$(call trim,$(TMPDIR)/$1.purge)
endef


define echo-flataux
echo >>$(TMPDIR)/flatten-aux.sed
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
echo >>$(TMPDIR)/color_tex.sed
endef

# Colorize LaTeX output.
define make-color_tex
$(echo-texlog) '/^[[:space:]]*Output written/{'; \
$(echo-texlog) '  s/.*(\([^)]\{1,\}\)).*/Success!  Wrote \1/'; \
$(echo-texlog) '  s/[[:digit:]]\{1,\}/$(C_INFO)&$(C_RESET)/g'; \
$(echo-texlog) '  s/Success!/$(C_INFO)&$(C_RESET)/g'; \
$(echo-texlog) '  p'; \
$(echo-texlog) '}'; \
$(echo-texlog) '/[^ (]*\.tex[^(]*([^ (]*\.tex/{'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)\([^(]*\)\(([^ (]*\.tex\)/$(C_INFO)\1$(C_RESET)\n\3/'; \
$(echo-texlog) '  P'; \
$(echo-texlog) '  D'; \
$(echo-texlog) '}'; \
$(echo-texlog) '/[^ (]*\.tex/{'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)$$/$(C_INFO)\1$(C_RESET)\n/'; \
$(echo-texlog) '  s/.*(\([^ (]*\.tex\)[ )]/$(C_INFO)\1$(C_RESET)\n/'; \
$(echo-texlog) '  P'; \
$(echo-texlog) '  D'; \
$(echo-texlog) '}'; \
$(echo-texlog) 's/^! *LaTeX Error:.*/$(C_ERROR)&$(C_RESET)/'; \
$(echo-texlog) 't'; \
$(echo-texlog) 's/^LaTeX Warning:.*/$(C_WARNING)&$(C_RESET)/'; \
$(echo-texlog) 't'; \
$(echo-texlog) 's/^Underfull.*/$(C_WARNING)&$(C_RESET)/'; \
$(echo-texlog) 't'; \
$(echo-texlog) 's/^Overfull.*/$(C_WARNING)&$(C_RESET)/'; \
$(echo-texlog) 't'; \
$(echo-texlog) 's/^\#\#\#.*/$(C_WARNING)&$(C_RESET)/'; \
$(echo-texlog) 't'; \
$(if $(VERBOSE),true,$(echo-texlog) 'd')
endef

# $(call latex-color-log,<LaTeX stem>)
define latex-color-log
if [ ! -f $(TMPDIR)/color_tex.sed ]; then touch $(TMPDIR)/color_tex.sed; $(call make-color_tex); fi; \
$(SED) -n -f $(TMPDIR)/color_tex.sed $1.log
endef

define make-latex-errors
echo '/^! LaTeX Error: File/d ;\
/^! LaTeX Error: Cannot determine size/d ;\
/[^ (]*\.tex/{s/[^ (]*\.tex/$(C_ERROR)&$(C_RESET)/g; p} ;\
/^! /,/^$$/{ \
  H ;\
  /^$$/{ \
    x ;\
    s/^.*$$/$(C_ERROR)&$(C_RESET)/ ;\
    p ;\
  } ;\
} ;\
d' \
>> $(TMPDIR)/latex-errors.sed
endef


# Colorizes real, honest-to-goodness LaTeX errors that can't be
# overcome with recompilation.
#
# $(call latex-error-log,<LaTeX stem>)
define latex-error-log
if [ ! -f $(TMPDIR)/latex-errors.sed ]; then touch $(TMPDIR)/latex-errors.sed; $(call make-latex-errors); fi; \
$(SED) -f $(TMPDIR)/latex-errors.sed $1.log
endef

# Colorize BibTeX output.
define make-bibtex-color
echo '\
s/^Warning--.*/$(C_WARNING)&$(C_RESET)/ ;\
t ;\
/---/,/^.[^:]/{ \
  H ;\
  /^.[^:]/{ \
    x ;\
    s/\n\(.*\)/$(C_ERROR)\1$(C_RESET)/ ;\
    p ;\
    s/.*// ;\
    h ;\
    d ;\
  } ;\
  d ;\
} ;\
/(.*error.*)/s//$(C_ERROR)&$(C_RESET)/'>>$(TMPDIR)/bibtex-color.sed; \
$(if $(VERBOSE),true,echo 'd' >>$(TMPDIR)/bibtex-color.sed)
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
echo Running LaTeX; \
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
echo Running BibTeX on $1; \
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
else echo \*\*\* LaTeX warnings and errors below \*\*\* ; \
$(call latex-color-log,$1); \
fi
endef

# Will create the stem.deps dependencies file
# $(call make-deps,<stem>)
define make-deps
  $(call get-inputs,$1,$1.pdf); \
  $(call get-bbl-deps,$1,$1.pdf); \
  $(call make-inds-deps,$1,$1.pdf)
endef

############################################################################

# Cancelling the dvi implicit rule
%.dvi: %.tex

.SECONDARY:

.PHONY: all FORCE

all: $(FILE).pdf

# Include if we're not cleaning
#ifeq "$(hascleangoals)" ""
-include $(TMPDIR)/$(FILE).deps
#endif

.PHONY: clean
clean:
	$(QUIET)cat $(TMPDIR)/$(FILE).clean | xargs rm -f ;\
	rm -rf $(MORECLEAN)

.PHONY: purge
purge: clean
	$(QUIET)cat $(TMPDIR)/$(FILE).purge | xargs rm -f ;\
	rm -rf $(TMPDIR) ;\
	rm -rf $(MOREPURGE)

# This target has the potential of removing user files
# (use with caution)
.PHONY: unsafe-purge
unsafe-purge: purge
	$(QUIET)rm -f $(LATEXCLEAN) ;\
	rm -f $(LATEXPURGE) ;\
	rm -rf $(TMPDIR)

.SUFFIXES: 
.SUFFIXES: .tex .dvi .ps .pdf .html .aux .deps .vars \
.idx .ind .gls .glo .ist .bbl \
.auxbbl .auxbbl.cookie \
.auxglo .auxglo.cookie \
.auxist .auxist.cookie \
.auxidx .auxidx.cookie

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
# Steps: latex_init, latex_index, latex_refs, latex_links => 5 steps

ifndef LATEXSTEP

%.bbl: %.aux
	$(QUIET)true

# We need to build the .deps only at the first step
# SC: not exactly...

# We force the re-generation : the user expects something to happen
# when typing "make", at the moment forcing a rebuild is the safe bet,
# dependency tracking in LaTeX being somewhat difficult with all the
# packages that rely on/build intermediate files.
%.pdf: FORCE
	$(QUIET)$(call run-latex,$*,$@); \
	$(call make-deps,$*); \
	echo \#\#\#\#\#\# Was step: initial ; \
	$(call rerun,$*,$@,latex_init)

endif

ifeq ($(LATEXSTEP),latex_init)
# Index and glossaries should be done here
%.ind: $(TMPDIR)/%.auxidx
	$(QUIET)echo Running makeindex for $@; \
	$(MAKEINDEX) $* ;\
	echo $@ >>$(TMPDIR)/$(FILE).index-done

%.gls: $(TMPDIR)/%.auxglo $(TMPDIR)/%.auxist
	$(QUIET)echo Running makeindex for glossary $@; \
	$(MAKEINDEX) -t $*.glg -o $@ -s $*.ist $*.glo ; \
	echo $@ >>$(TMPDIR)/$(FILE).index-done

%.bbl: %.aux
	$(QUIET)true

# We loop here because page number for the glossary might change after
# the compilation, hence we have to generate again the .gls

$(FILE).pdf: FORCE
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).index-done ]; \
	then \
	  egrep '\\cite\>' `cat $(TMPDIR)/$(FILE).index-done` $(TODEVNULL); \
	  if [ $$? -eq 0 ]; then $(call run-latex,$*,$@); fi; \
	  rm $(TMPDIR)/$(FILE).index-done; \
	  echo \#\#\#\#\#\# Was step: index, glossaries; \
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

$(FILE).pdf: FORCE
	$(QUIET)if [ -f $(TMPDIR)/$(FILE).bib-done ]; \
	then \
	  rm $(TMPDIR)/$(FILE).bib-done; \
	  $(call run-latex,$(FILE),$@); \
	  echo \#\#\#\#\#\# Was step: bibliography; \
	fi; \
	$(call possibly-rerun,$(FILE),$@,latex_refs)


endif

ifeq ($(LATEXSTEP),latex_refs)

$(FILE).pdf: FORCE
	$(QUIET)$(call run-latex,$(FILE),$@); \
	echo \#\#\#\#\#\# Was step: cross-references; \
	$(call possibly-rerun,$(FILE),$@,latex_links)


endif

ifeq ($(LATEXSTEP),latex_links)

$(FILE).pdf: FORCE
	$(QUIET)$(call run-latex,$(FILE),$@); \
	echo \#\#\#\#\#\# Was step: last chance to solve undefined references; \
	$(call latex-color-log,$(FILE))

endif


FORCE:


%.html: %.pdf
	$(HEVEA) $*.hva $*


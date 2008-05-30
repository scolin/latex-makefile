
# Copyright 2008 Samuel Colin

# This file is based off Chris Monson's Makefile:
# http://www.bouncingchairs.net/oss
# As such, parts derived directly from it are licenced under the 
# GPLv2 (http://www.gnu.org/copyleft/gpl.html)
# under the following terms, copied verbatim from the original Makefile
# 
# Parts I added are licenced GPLv2 as well. Things may evolve later on.


#
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
SED		:= sed
SORT		:= sort
TOUCH		:= touch
UNIQ		:= uniq
WHICH		:= which
XARGS		:= xargs
# == LaTeX (tetex-provided) ==
BIBTEX		:= bibtex
DVIPS		:= dvips
LATEX		:= latex
KPSEWHICH	:= kpsewhich
PS2PDF_NORMAL	:= ps2pdf
PS2PDF_EMBED	:= ps2pdf13
# = OPTIONAL PROGRAMS =
# == Makefile Color Output ==
TPUT		:= tput
# == Beamer Enlarged Output ==
PSNUP		?= psnup
# == Viewing Stuff ==
VIEW_POSTSCRIPT	?= gv
VIEW_PDF	?= xpdf
VIEW_GRAPHICS	?= display

#
# BSD SED NOTES
#
# BSD SED is not very nice compared to GNU sed, but it is the most
# commonly-invoked sed on Macs (being based on BSD), so we have to cater to
# it or require people to install GNU sed.  It seems like the GNU
# requirement isn't too bad since this makefile is really a GNU makefile,
# but apparently GNU sed is much less common than GNU make in general, so
# I'm supporting it here.
#
# Sad experience has taught me the following about BSD sed:
#
# 	* \+ is not understood to mean \{1,\}
# 	* \| is meaningless (does not branch)
# 	* \n cannot be used as a substitution character
# 	* ? does not mean \{0,1\}, but is literal
# 	* a\ works, but only reliably for a single line if subsequent lines
# 		have forward slashes in them (as is the case in postscript)
#
# For more info (on the Mac) you can consult
#
# man -M /usr/share/man re_format
#
# And look for the word "Obsolete" near the bottom.

#
# EXTERNAL PROGRAM DOCUMENTATION SCRIPT
#

# $(call output-all-programs,[<output file>])
define output-all-programs
	[ -f '$(this_file)' ] && \
	$(SED) \
		-e '/^[[:space:]]*#[[:space:]]*EXTERNAL PROGRAMS:/,/^$$/!d' \
		-e '/EXTERNAL PROGRAMS/d' \
		-e '/^$$/d' \
		-e '/^[[:space:]]*#/i\ '\
		-e 's/^[[:space:]]*#[[:space:]][^=]*//' \
		$(this_file) $(if $1,> '$1',) || \
	$(ECHO) "Cannot determine the name of this makefile."
endef

# If they misspell gray, it should still work.
GRAY	?= $(call get-default,$(GREY),)

#
# Utility Functions and Definitions
#

# While not exactly a make function, this vim macro is useful.  It takes a
# verbatim sed script and converts each line to something suitable in a command
# context.  Just paste the script's contents into the editor, yank this into a
# register (starting at '0') and run the macro once for each line of the
# original script:
#
# 0i	-e :s/\$/$$/eg:s/'/'"'"'/eg^Ela'A' \:nohj

# Test that a file exists
# $(call test-exists,file)
test-exists		= [ -e '$1' ]

# Copy file1 to file2 only if file2 doesn't exist or they are different
# $(call copy-if-different,sfile,dfile)
copy-if-different	= $(call test-different,$1,$2) && $(CP) '$1' '$2'
copy-if-exists		= $(call test-exists,$1) && $(CP) '$1' '$2'
move-if-different	= $(call test-different,$1,$2) && $(MV) '$1' '$2'
replace-if-different-and-remove	= \
	$(call test-different,$1,$2) && $(MV) '$1' '$2' || $(RM) '$1'

# Note that $(DIFF) returns success when the files are the SAME....
# $(call test-different,sfile,dfile)
test-different		= ! $(DIFF) -q '$1' '$2' &>/dev/null
test-exists-and-different	= \
	$(call test-exists,$2) && $(call test-different,$1,$2)

# Return value 1, or value 2 if value 1 is empty
# $(call get-default,<possibly empty arg>,<default value if empty>)
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
# Characters that are hard to specify in certain places
space		:= $(empty) $(empty)
colon		:= \:
comma		:= ,

# Useful shell definitions
sh_true		:= :
sh_false	:= ! :

# Clear out the standard interfering make suffixes
.SUFFIXES:

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
LATEX_COLOR_WARNING	?= magenta
LATEX_COLOR_ERROR	?= red
LATEX_COLOR_INFO	?= green
LATEX_COLOR_UNDERFULL	?= magenta
LATEX_COLOR_OVERFULL	?= red bold
LATEX_COLOR_PAGES	?= bold
LATEX_COLOR_BUILD	?= blue
LATEX_COLOR_GRAPHIC	?= yellow
LATEX_COLOR_DEP		?= green
LATEX_COLOR_SUCCESS	?= green bold
LATEX_COLOR_FAILURE	?= red bold

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
C_UNDERFULL	:= $(call get-color,UNDERFULL)
C_OVERFULL	:= $(call get-color,OVERFULL)
C_PAGES		:= $(call get-color,PAGES)
C_BUILD		:= $(call get-color,BUILD)
C_GRAPHIC	:= $(call get-color,GRAPHIC)
C_DEP		:= $(call get-color,DEP)
C_SUCCESS	:= $(call get-color,SUCCESS)
C_FAILURE	:= $(call get-color,FAILURE)
C_RESET		:= $(reset)

#
# PRE-BUILD TESTS
#

# Check that clean targets are not combined with other targets (weird things
# happen, and it's not easy to fix them)
hascleangoals	:= $(if $(sort $(filter clean clean-%,$(MAKECMDGOALS))),1)
hasbuildgoals	:= $(if $(sort $(filter-out clean clean-%,$(MAKECMDGOALS))),1)
ifneq "$(hasbuildgoals)" ""
ifneq "$(hascleangoals)" ""
$(error $(C_ERROR)Clean and build targets specified together$(C_RESET)))
endif
endif

#
# VARIABLE DECLARATIONS
#


# Files of interest
all_files.tex		:= $(wildcard *.tex)

# Patterns to never be allowed as source targets
ignore_patterns	:= %._include_

# Patterns allowed as source targets but not included in 'all' builds
nodefault_patterns := %._nobuild_ $(ignore_patterns)

# Utility function for getting targets suitable building
# $(call filter-buildable,suffix)
filter-buildable	= \
	$(filter-out $(addsuffix .$1,$(ignore_patterns)),$(all_files.$1))

# Utility function for getting targets suitable for 'all' builds
# $(call filter-default,suffix)
filter-default		= \
	$(filter-out $(addsuffix .$1,$(nodefault_patterns)),$(all_files.$1))

# Top level sources that can be built even when they are not by default
files.tex	:= $(filter-out %._gray_.tex,$(call filter-buildable,tex))

# Top level sources that are built by default targets
default_files.tex	:= $(filter-out %._gray_.tex,$(call filter-default,tex))

# Utility function for creating larger lists of files
# $(call concat-files,suffixes,[prefix])
concat-files	= $(foreach s,$1,$($(if $2,$2_,)files.$s))

# Useful file groupings
all_files_source	:= $(call concat-files,tex,all)

default_files_source	:= $(call concat-files,tex,default)

files_source	:= $(call concat-files,tex)

# Utility function for obtaining stems
# $(call get-stems,suffix,[prefix])
get-stems	= $(sort $($(if $2,$2_,)files.$1:%.$1=%))

# List of all stems (including ._include_ and ._nobuild_ file stems)
all_stems.tex		:= $(call get-stems,tex,all)

# List of all default stems (all default PDF targets):
default_stems.tex		:= $(call get-stems,tex,default)

# List of all stems (all possible bare PDF targets created here):
stems.tex		:= $(call get-stems,tex)

# Utility function for creating larger lists of stems
# $(call concat-stems,suffixes,[prefix])
concat-stems	= $(sort $(foreach s,$1,$($(if $2,$2_,)stems.$s)))

all_stems_source	:= $(call concat-stems,tex,all)

default_stems_source	:= $(call concat-stems,tex,default)

stems_source		:= $(call concat-stems,tex)

# Calculate names that can generate the need for an include file.  We can't
# really do this with patterns because it's too easy to screw up, so we create
# an exhaustive list.
allowed_source_suffixes	:= \
	pdf \
	ps \
	dvi \
	bbl \
	aux \
	aux.make \
	d \
	tex \
	auxbbl.make \
	_graphics \
	_show
allowed_source_patterns		:= $(addprefix %.,$(allowed_source_suffixes))


# All targets allowed to build documents
allowed_source_targets	:= \
	$(foreach suff,$(allowed_source_suffixes),\
	$(addsuffix .$(suff),$(stems_ssg)))

# All targets allowed to build graphics
allowed_graphic_targets	:= \
	$(foreach suff,$(allowed_graphic_suffixes),\
	$(addsuffix .$(suff),$(stems_gg)))

# All targets that build multiple documents (like 'all')
allowed_batch_source_targets	:= \
	all \
	all-pdf \
	all-ps \
	all-dvi \
	all-bbl \
	show

# Now we figure out which stuff is available as a make target for THIS RUN.
real_goals	:= $(call get-default,$(filter-out _includes,$(MAKECMDGOALS)),\
			all)

specified_source_targets	:= $(strip \
	$(filter $(allowed_source_targets) $(stems_ssg),$(real_goals)) \
	)

specified_batch_source_targets	:= $(strip \
	$(filter $(allowed_batch_source_targets),$(real_goals)) \
	)

# Determine which .d files need including from the information gained above.
# This is done by first checking whether a batch target exists.  If it does,
# then all *default* stems are used to create possible includes (nobuild need
# not apply for batch status).  If no batch targets exist, then the individual
# targets are considered and appropriate includes are taken from them.
source_stems_to_include	:= \
	$(sort\
	$(if $(specified_batch_source_targets),\
		$(default_stems_ss),\
		$(foreach t,$(specified_source_targets),\
		$(foreach p,$(allowed_source_patterns),\
			$(patsubst $p,%,$(filter $p $(stems_ssg),$t)) \
		)) \
	))

# All dependencies for the 'all' targets
all_pdf_targets		:= $(addsuffix .pdf,$(stems_ssg))
all_ps_targets		:= $(addsuffix .ps,$(stems_ssg))
all_dvi_targets		:= $(addsuffix .dvi,$(stems_ssg))
all_tex_targets		:= $(addsuffix .tex,$(stems_sg))
all_d_targets		:= $(addsuffix .d,$(stems_ssg))

default_pdf_targets	:= $(addsuffix .pdf,$(default_stems_ss))
default_ps_targets	:= $(addsuffix .ps,$(default_stems_ss))
default_dvi_targets	:= $(addsuffix .dvi,$(default_stems_ss))

# Extensions generated by LaTeX invocation that can be removed when complete
rm_ext		:= log,aux,dvi,ps,pdf,blg,bbl,out,nav,snm,toc,lof,lot,pfg,fls,vrb
backup_patterns	:= *.bak

# All LaTeX-generated files that can be safely removed
rm_tex		:= \
	$(addsuffix .{$(rm_ext)},$(all_stems_source)) \
	$(addsuffix .{$(rm_ext)$(comma)tex},$(all_stems_sg)) \
	$(addsuffix .log,$(all_ps_targets) $(all_pdf_targets)) \
	$(addsuffix .*.log,$(stems_graphic)) \

#
# Functions used in generating output
#

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

# Outputs all bibliography files to stdout.  Arg 1 is the source stem, arg 2 is
# a list of targets for each dependency found.
#
# The script kills all lines that do not contain bibdata.  Remaining lines have
# the \bibdata macro and delimiters removed to create a dependency list.  A
# trailing comma is added, then all adjacent commas are collapsed into a single
# comma.  Then commas are replaced with the string .bib[space], and the
# trailing space is killed off.  This produces a list of space-delimited .bib
# filenames, which is what the make dep file expects to see.
#
# $(call get-bibs,<aux file>,<targets>)
define get-bibs
$(SED) \
-e '/^\\bibdata/!d' \
-e 's/\\bibdata{\([^}]*\)}/\1,/' \
-e 's/,\{2,\}/,/g' \
-e 's/,/.bib /g' \
-e 's/ \{1,\}$$//' \
$1 | $(XARGS) $(KPSEWHICH) - | \
$(SED) \
-e 's/^/$2: /' | \
\$(SORT) | $(UNIQ)
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
-e     's![.:]!\\&!g' \
-e     'h' \
-e     's!.*!\\:\\\\@input{&}:{!' \
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
	-e ' s/[[:digit:]]\{1,\}/$(C_PAGES)&$(C_RESET)/g' \
	-e ' s/Success!/$(C_SUCCESS)&$(C_RESET)/g' \
	-e '}' \
	-e 's/^! *LaTeX Error:.*/$(C_ERROR)&$(C_RESET)/' -e 't' \
	-e 's/^LaTeX Warning:.*/$(C_WARNING)&$(C_RESET)/' -e 't' \
	-e 's/^Underfull.*/$(C_UNDERFULL)&$(C_RESET)/' -e 't' \
	-e 's/^Overfull.*/$(C_OVERFULL)&$(C_RESET)/' -e 't' \
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

# LaTeX invocations
#
# $(call latex,<tex file>,[<extra LaTeX args>])
run-latex	= $(LATEX) --interaction=batchmode $(if $2,$2,) $1 > /dev/null

# $(call latex-color-log,<LaTeX stem>)
latex-color-log	= $(color_tex) $1.log

# BibTeX invocations
#
# $(call run-bibtex,<tex stem>)
run-bibtex	= $(BIBTEX) $1 | $(color_bib)



# Converts graphviz .dot files into .eps files
# Grayscale is not directly supported by dot, so we pipe it through fig2dev in
# that case.
# $(call convert-dot,<dot file>,<eps file>,<log file>,[gray])
define convert-dot
$(DOT) -Tps '$1' 2>'$3' $(if $4,| $(call kill-ps-color)) > '$2'; \
$(call colorize-dot-errors,$3)
endef

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
echo-build	= $(ECHO) "$(C_BUILD)= $1 --> $2$(if $3, ($3),) =$(C_RESET)"
echo-dep	= $(ECHO) "$(C_DEP)= $1 --> $2 =$(C_RESET)"

# Display a list of something
# $(call echo-list,<values>)
echo-list	= for x in $1; do $(ECHO) "$$x"; done

#
# DEFAULT TARGET
#

.PHONY: all
all: $(default_pdf_targets) ;

.PHONY: all-pdf
all-pdf: $(default_pdf_targets) ;

.PHONY: all-ps
all-ps: $(default_ps_targets) ;

.PHONY: all-dvi
all-dvi: $(default_dvi_targets) ;

#
# VIEWING TARGET
#
.PHONY: show
show: all
	$(QUIET)for x in $(default_pdf_targets); do \
		[ -e "$$x" ] && $(VIEW_PDF) $$x & \
	done

#
# INCLUDES
#
source_includes	:= $(addsuffix .d,$(source_stems_to_include))

# Include only the dependencies used
ifneq "" "$(source_includes)"
include $(source_includes)$(call include-message,$(source_includes))
endif

#
# MAIN TARGETS
#

%: %.pdf ;

# This builds and displays the wanted file.
.PHONY: $(addsuffix ._show,$(stems_ssg))
%._show: %.pdf
	$(QUIET)$(VIEW_PDF) $< &

.SECONDARY: $(all_dvi_targets)
%.dvi: %.bbl %.aux
	$(QUIET)\
	fatal=`$(call colorize-latex-errors,$*.log)`; \
	if [ "$$fatal" != "" ]; then \
		$(ECHO) "$$fatal"; \
		exit 1; \
	fi; \
	$(call make-auxdvi-file,$*.aux.make,$*.auxdvi.cookie); \
	run=0; \
	for i in 1; do \
		if $(call test-exists,$*.bbl.cookie); then \
			run=1; \
			break; \
		fi; \
		if $(call test-exists,$*.run.cookie); then \
			run=1; \
		    	break; \
		fi; \
		if $(call \
		test-exists-and-different,$*.auxdvi.cookie,$*.auxdvi.make);\
		then \
			run=1; \
			break; \
		fi; \
		if $(call test-log-for-need-to-run,$*); then \
			run=1; \
			break; \
		fi; \
	done; \
	$(RM) $*.bbl.cookie $*.run.cookie; \
	$(MV) $*.auxdvi.cookie $*.auxdvi.make; \
	if [ "$$run" = "1" ]; then \
		$(RM) $@.1st.make; \
		for i in 2 3 4 5; do \
			$(if $(findstring 3.79,$(MAKE_VERSION)),\
				$(call echo-build,$*.tex,$@,$$$$i),\
				$(call echo-build,$*.tex,$@,$$i)\
			); \
			$(call run-latex,$*); \
			$(call test-run-again,$*) || break; \
		done; \
	else \
		$(MV) $@.1st.make $@; \
	fi; \
	$(call latex-color-log,$*)

# Build the .bbl file.  When dependencies are included, this will (or will
# not!) depend on something.bib, which we detect, acting accordingly.  The
# dependency creation also produces the %.aux.make file.  BibTeX is a bit
# finicky about what you call the actual files, but we can rest assured that if
# a .aux.make file exists, then the .aux file does, as well.  The .aux.make
# file is a cookie indicating whether the .bbl needs to be rewritten.  It only
# changes if the .aux file changes in ways relevant to .bbl creation.
#
# Note that we do NOT touch the .bbl file if there is no need to
# create/recreate it.  We would like to leave existing files alone if they
# don't need to be changed, thus possibly avoiding a rebuild trigger on the
# .dvi side.
%.bbl: %.auxbbl.make
	$(QUIET)\
	$(if $(filter %.bib,$^),\
		$(call echo-build,$(filter %.bib,$?) $*.aux,$@); \
		$(call run-bibtex,$*); \
		$(TOUCH) $@.cookie; \
	)


#
# DEPENDENCY-RELATED TARGETS.
#

# Generate all of the information needed to get dependencies
# As a side effect, this creates a .dvi file.  We need to be sure to remove it
# if there are errors.  Errors can take several forms and all of them are found
# within the log file:
#	* There was a LaTeX error
#	* A needed file was not found
#	* Cross references need adjustment
#
# Behavior:
#	This rule is responsible for generating the following:
#	%.aux
#	%.d
#	%.aux.make
#	%.dvi.1st.make (the .dvi file, moved)
#
#	Steps:
#
#	Run latex
#	Move .dvi somewhere else (make no judgements about success)
#	Flatten the .aux file into another file
#	Add source dependencies
#	Add graphic dependencies
#	Add bib dependencies
#
#	Create cookies for various suffixes that may represent files that
#	need to be read by LaTeX in order for it to function properly.
#
%.d %.aux %.aux.make: %.tex
	$(QUIET)$(call echo-build,$<,$*.d $*.dvi,1)
	$(QUIET)\
	$(call run-latex,$<,--recorder) || $(sh_true); \
	$(MV) $*.dvi $*.dvi.1st.make; \
	$(call flatten-aux,$*.aux,$*.aux.make); \
	$(ECHO) "# vim: ft=make" > $*.d; \
	$(ECHO) ".PHONY: $*._graphics" >> $*.d; \
	$(call get-inputs,$*.fls,$(addprefix $*.,aux aux.make d dvi)) \
		>> $*.d; \
	$(call get-graphics,$*.log,$(addprefix $*.,d dvi _graphics)) \
		>> $*.d; \
	$(call get-bibs,$*.aux.make,$(addprefix $*.,bbl aux aux.make)) \
		>> $*.d; \
	for s in toc out lot lof nav; do \
		if [ -e "$*.$$s" ]; then \
			if ! $(DIFF) -q $*.$$s $*.$$s.make 2>/dev/null; then \
				$(TOUCH) $*.run.cookie; \
			fi; \
			$(CP) $*.$$s $*.$$s.make; \
		fi; \
	done

# This is a cookie that is updated if the flattened aux file has changed in a
# way that affects the bibliography generation.
.SECONDARY: $(addsuffix .auxbbl.make,$(stems_ssg))
%.auxbbl.make: %.aux.make
	$(QUIET)\
	$(call make-auxbbl-file,$<,$@.temp); \
	$(call replace-if-different-and-remove,$@.temp,$@)


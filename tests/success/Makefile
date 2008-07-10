
TESTS= \
  article \
  article-include \
  beamer \
  bibunits \
  glossary-bib \
  graphics-latex \
  graphics-pdftex \
  index \
  minitoc-partchap \
  minitoc-sect \
  multiple-documents \
  natbib \
  thumbpdf \
  thumbpdf-dvi
# beamer_handouts

UNPURGETESTS:=$(foreach test,$(TESTS),$(test).unpurge)
PURGETESTS:=$(foreach test,$(TESTS),$(test).purge)
CLEANTESTS:=$(foreach test,$(TESTS),$(test).clean)

DVITESTS:=$(foreach test,$(TESTS),$(test).dvi)
PSTESTS:=$(foreach test,$(TESTS),$(test).ps)
PDFTESTS:=$(foreach test,$(TESTS),$(test).pdf)

.PHONY: all purge clean $(TESTS) $(PURGETESTS) $(CLEANTESTS)
.PHONY: dvi ps pdf $(DVITESTS) $(PSTESTS) $(PDFTESTS)

all: $(TESTS)

unsafe-purge: $(UNPURGETESTS)

purge: $(PURGETESTS)

clean: $(CLEANTESTS)

dvi: $(DVITESTS)

ps: $(PSTESTS)

pdf: $(PDFTESTS)


$(UNPURGETESTS): %.unpurge:
	cd $* && $(MAKE) -s unsafe-purge

$(PURGETESTS): %.purge:
	cd $* && $(MAKE) -s purge

$(CLEANTESTS): %.clean:
	cd $* && $(MAKE) -s clean

$(DVITESTS): %.dvi:
	cd $* && $(MAKE) -s dvi

$(PSTESTS): %.ps:
	cd $* && $(MAKE) -s ps

$(PDFTESTS): %.pdf:
	cd $* && $(MAKE) -s pdf

$(TESTS):
	cd $@ && $(MAKE) -s

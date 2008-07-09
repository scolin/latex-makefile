
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

.PHONY: all purge clean $(TESTS) $(PURGETESTS) $(CLEANTESTS)

all: $(TESTS)

unsafe-purge: $(UNPURGETESTS)

purge: $(PURGETESTS)

clean: $(CLEANTESTS)


$(UNPURGETESTS): %.unpurge:
	cd $* && $(MAKE) -s unsafe-purge

$(PURGETESTS): %.purge:
	cd $* && $(MAKE) -s purge

$(CLEANTESTS): %.clean:
	cd $* && $(MAKE) -s clean

$(TESTS):
	cd $@ && $(MAKE) -s

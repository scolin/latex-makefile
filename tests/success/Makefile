
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
  thumbpdf
# beamer_handouts

PURGETESTS:=$(foreach test,$(TESTS),$(test).purge)

.PHONY: all purge $(TESTS) $(PURGETESTS)

all: $(TESTS)

purge: $(PURGETESTS)

$(PURGETESTS): %.purge:
	cd $* && make -s purge

$(TESTS):
	cd $@ && make -s

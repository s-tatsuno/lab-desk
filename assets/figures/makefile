TEX   := $(wildcard *.tex)
SVG   := $(TEX:.tex=.svg)
BUILD := .build

# エディタ等がカレントに出力する中間生成物
AUX_EXT := aux log fls fdb_latexmk synctex.gz out toc nav snm xdv dvi pdf
AUX     := $(foreach e,$(AUX_EXT),$(TEX:.tex=.$(e)))

.PHONY: all tidy clean
.DELETE_ON_ERROR:

# SVG を生成した後、中間生成物をすべて削除する
all: $(SVG)
	@$(MAKE) --no-print-directory tidy

%.svg: %.tex
	latexmk -lualatex -g -silent -halt-on-error -outdir=$(BUILD) $<
	dvisvgm --pdf --no-fonts --exact-bbox -o $@ $(BUILD)/$*.pdf

# 中間生成物のみ削除（SVG は残す）
tidy:
	@echo "rm -rf $(BUILD) and intermediate files"
	@rm -rf $(BUILD) $(AUX)

# SVG も含めてすべて削除
clean: tidy
	rm -f $(SVG)

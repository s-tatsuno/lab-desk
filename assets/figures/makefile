TEX := $(wildcard *.tex)
SVG := $(TEX:.tex=.svg)

all: $(SVG)

%.svg: %.tex
	latexmk -lualatex -silent -outdir=.build $*.tex
	dvisvgm --pdf --no-fonts --exact-bbox -o $@ .build/$*.pdf

clean:
	rm -rf .build *.svg
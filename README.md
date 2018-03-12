## Modified version from Kieran Healy Makefile, with the addition of RTF, nothing else changed
## What extension (e.g. md, markdown, mdown) is being used 
## for markdown files MEXT = md

## Expands to a list of all markdown files in the working directory
SRC = $(wildcard *.$(MEXT))

## Location of Pandoc support files.
PREFIX = /Users/kjhealy/.pandoc

## Location of your working bibliography file
BIB = /Users/kjhealy/Documents/bibs/socbib-pandoc.bib

## CSL stylesheet (located in the csl folder of the PREFIX directory).
CSL = apsa

## x.pdf depends on x.md, x.html depends on x.md, etc
PDFS=$(SRC:.md=.pdf)
HTML=$(SRC:.md=.html)
TEX=$(SRC:.md=.tex)
DOCX=$(SRC:.md=.docx)

## Rules -- make all, make pdf, make html. The `clean` rule is below.
all:    $(PDFS) $(HTML) $(TEX) $(DOCX)
pdf:    clean $(PDFS)
html:   clean $(HTML)
tex:    clean $(TEX)
docx:   clean $(DOCX)

## The commands associated with each rule. The first one is run with 
## `make html`. Read the rule as "When making the html file, run 
## this pandoc command if the .md file has changed."
%.html: %.md
    pandoc -r markdown+simple_tables+table_captions+yaml_metadata_block \ 
    -w html -S --template=$(PREFIX)/templates/html.template \
    --css=$(PREFIX)/marked/kultiad-serif.css --filter pandoc-crossref \     
    --filter pandoc-citeproc --csl=$(PREFIX)/csl/$(CSL).csl \ 
    --bibliography=$(BIB) -o $@ $<

## Same goes for the other file types. 
## Watch out for the TAB before 'pandoc'
%.tex:  %.md
    pandoc -r markdown+simple_tables+table_captions+yaml_metadata_block \ 
    --listings -w latex -s -S --latex-engine=pdflatex \ 
    --template=$(PREFIX)/templates/latex.template \ 
    --filter pandoc-crossref --filter pandoc-citeproc \ 
    --csl=$(PREFIX)/csl/ajps.csl --filter pandoc-citeproc-preamble \ 
    --bibliography=$(BIB) -o $@ $<

%.pdf:  %.md
    pandoc -r markdown+simple_tables+table_captions+yaml_metadata_block \ 
    --listings -s -S --latex-engine=pdflatex \ 
    --template=$(PREFIX)/templates/latex.template \ 
    --filter pandoc-crossref --filter pandoc-citeproc \ 
    --csl=$(PREFIX)/csl/$(CSL).csl --filter pandoc-citeproc-preamble \ 
    --bibliography=$(BIB) -o $@ $<

%.docx: %.md
    pandoc -r markdown+simple_tables+table_captions+yaml_metadata_block \ 
    -s -S --filter pandoc-crossref --csl=$(PREFIX)/csl/$(CSL).csl \ 
    --bibliography=$(BIB) -o $@ $<

clean:
    rm -f *.html *.pdf *.tex *.aux *.log *.docx

.PHONY: clean

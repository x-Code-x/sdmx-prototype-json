TARGET := sdmx-json-field-guide
STATUS := src/status.md
TITLE  := src/title.txt
BODY   := src/sdmx-json-field-guide.md src/full-example-with-comments.md src/references.md

src/status.tex: $(STATUS)
		pandoc -o src/status.tex $(STATUS)

$(TARGET).pdf: src/status.tex src/options.sty $(TITLE) $(BODY)
#		pandoc -B status.tex -N -s -S -H options.sty --default-image-extension=pdf -V geometry:margin=1in -V documentclass:report --toc --toc-depth=4 --variable fontsize=11pt -o $@ $(TITLE) $(BODY)
		pandoc -B src/status.tex -N -s -S -H src/options.sty --default-image-extension=pdf -V documentclass:scrreprt --toc --toc-depth=4 -o $@ $(TITLE) $(BODY)

src/status.html: $(STATUS)
		pandoc -t html5 -o src/status.html $(STATUS)

$(TARGET).html: src/status.html src/doc-style.html $(TITLE) $(BODY)
		pandoc -B src/status.html -N -s --default-image-extension=svg -H src/doc-style.html --toc --toc-depth=4 -t html5 -o $@ $(TITLE) $(BODY)

pdf: $(TARGET).pdf

html: $(TARGET).html

clean:
		rm -f $(TARGET).pdf $(TARGET).html

.PHONY: clean pdf html

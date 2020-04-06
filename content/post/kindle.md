+++
author = "Cillian Berragan"
title = "Scripting in Bash to Export pdfs to a Kindle"
date = "2019-11-15"
tags = [
    "bash",
    "kindle"
]
categories = [
    "Blog"
]
+++

In an effort to convert two column pdfs to the Kindle format, I found that an accepted solution did not at present exist. Converting a single column pdf is simple enough, but with two columns most programs are unable to correctly order the text.

<!--more-->

However, one program was able to order columns into images: `k2pdfopt`. Unfortunately as this was no longer a pdf I had to use a program to read text from an image: `tesseract`. Finally the text could now be converted to the Kindle `EPUB` format using `calibre`.

Requirements:

```
k2pdfopt
tesseract
cpdf
calibre
```

Here is the script:

```bash
#!/bin/sh
INPUT=*.pdf

mkdir out
cp $INPUT out/
cd out/

for pdf in $INPUT; do

    TITLE=$(pdfinfo $pdf | grep -Po 'Title:\s*\K.*')
    AUTHOR=$(pdfinfo $pdf | grep -Po 'Author:\s*K.*')
    DATE=$(pdfinfo $pdf | grep -Po 'CreationDate:\s*\K.*')

    echo | k2pdfopt $pdf -o out_$pdf

    OUTPUT=out.txt # set to the final output file
    ENDPAGE=$(cpdf -pages out_$pdf)  # set to pagenumber of the last page of PDF you wish to convert

for i in $(seq -w 000 $ENDPAGE); do
    pdfimages out_$pdf temp
    echo processing page $i
    tesseract temp-$i.ppm tempoutput
    touch out.txt
    cat tempoutput.txt >> $OUTPUT
done

rm *.ppm
awk ' /^$/ { print "\n"; } /./ { printf("%s ", $0); } END { print ""; } ' $OUTPUT > $pdf.txt

ebook-convert $pdf.txt $pdf.mobi --authors="$AUTHOR" --title="$TITLE" --title-sort="$TITLE" --pubdate="$DATE"

rm temp*
rm *.txt
rm *.html

done

rm *.pdf
```

**Ensure a copy of the original pdf is made, at present this script will delete any pdf in the directory.**

To use this script, simply place it in a directory and copy in pdfs to convert, first run `chmod +x script.sh` to make it executible, then run `bash script.sh` and it should work automatically.

# Introduction to epubr

The `epubr` package provides functions supporting the reading and
parsing of internal e-book content from EPUB files. E-book metadata and
text content are parsed separately and joined together in a tidy, nested
tibble data frame.

E-book formatting is not completely standardized across all literature.
It can be challenging to curate parsed e-book content across an
arbitrary collection of e-books perfectly and in completely general
form, to yield a singular, consistently formatted output. Many EPUB
files do not even contain all the same pieces of information in their
respective metadata.

EPUB file parsing functionality in this package is intended for
relatively general application to arbitrary EPUB e-books. However,
poorly formatted e-books or e-books with highly uncommon formatting may
not work with this package. There may even be cases where an EPUB file
has DRM or some other property that makes it impossible to read with
`epubr`.

Text is read ‘as is’ for the most part. The only nominal changes are
minor substitutions, for example curly quotes changed to straight
quotes. Substantive changes are expected to be performed subsequently by
the user as part of their text analysis. Additional text cleaning can be
performed at the user’s discretion, such as with functions from packages
like `tm` or `qdap`.

## Read EPUB files

Bram Stoker’s Dracula novel sourced from Project Gutenberg is a good
example of an EPUB file with unfortunate formatting. The first thing
that stands out is the naming convention using `item` followed by some
ordered digits does not differentiate sections like the book preamble
from the chapters. The numbering also starts in a weird place. But it is
actually worse than this. Notice that sections are not broken into
chapters; they can begin and end in the middle of chapters!

These annoyances aside, the metadata and contents can still be read into
a convenient table. Text mining analyses can still be performed on the
overall book, if not so easily on individual chapters. See the section
below on restructuring for examples of `epubr` functions that help get
around these issues.

Here a single file is read with
[`epub()`](https://docs.ropensci.org/epubr/reference/epub.md). The
output of the returned primary data frame and the book text data frame
that is nested within its `data` column are shown.

``` r

library(epubr)
file <- system.file("dracula.epub", package = "epubr")
(x <- epub(file))
#> # A tibble: 1 × 9
#>   rights         identifier creator title language subject date  source data    
#>   <chr>          <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <list>  
#> 1 Public domain… http://ww… Bram S… Drac… en       Horror… 1995… http:… <tibble>

x$data[[1]]
#> # A tibble: 15 × 4
#>    section           text                                            nword nchar
#>    <chr>             <chr>                                           <int> <int>
#>  1 item6             "The Project Gutenberg EBook of Dracula, by Br… 11446 60972
#>  2 item7             "But I am not in heart to describe beauty, for… 13879 71798
#>  3 item8             "\" 'Lucy, you are an honest-hearted girl, I k… 12474 65522
#>  4 item9             "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, … 12177 62724
#>  5 item10            "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur … 12806 66678
#>  6 item11            "Once again we went through that ghastly opera… 12103 62949
#>  7 item12            "CHAPTER XIVMINA HARKER'S JOURNAL\n23 Septembe… 12214 62234
#>  8 item13            "CHAPTER XVIDR. SEWARD'S DIARY-continued\nIT w… 13990 72903
#>  9 item14            "\"Thus when we find the habitation of this ma… 13356 69779
#> 10 item15            "\"I see,\" I said. \"You want big things that… 12866 66921
#> 11 item16            "CHAPTER XXIIIDR. SEWARD'S DIARY\n3 October.-T… 11928 61550
#> 12 item17            "CHAPTER XXVDR. SEWARD'S DIARY\n11 October, Ev… 13119 68564
#> 13 item18            " \nLater.-Dr. Van Helsing has returned. He ha…  8435 43464
#> 14 item19            "End of the Project Gutenberg EBook of Dracula…  2665 18541
#> 15 coverpage-wrapper ""                                                  0     0
```

The `file` argument may be a vector of EPUB files. There is one row for
each book.

## EPUB metadata

The above examples jump right in, but it can be helpful to inspect file
metadata before reading a large number of books into memory. Formatting
may differ across books. It can be helpful to know what fields to
expect, the degree of consistency, and what content you may want to drop
during the file reading process.
[`epub_meta()`](https://docs.ropensci.org/epubr/reference/epub.md)
strictly parses file metadata and does not read the e-book text.

``` r

epub_meta(file)
#> # A tibble: 1 × 8
#>   rights                  identifier creator title language subject date  source
#>   <chr>                   <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr> 
#> 1 Public domain in the U… http://ww… Bram S… Drac… en       Horror… 1995… http:…
```

This provides the big picture, though it will not reveal the internal
breakdown of book section naming conventions that were seen in the first
[`epub()`](https://docs.ropensci.org/epubr/reference/epub.md) example.

`file` can also be a vector for
[`epub_meta()`](https://docs.ropensci.org/epubr/reference/epub.md).
Whenever `file` is a vector, the fields (columns) returned are the union
of all fields detected across all EPUB files. Any books (rows) that do
not have a field found in another book return `NA` for that row and
column.

## Additional arguments

There are three optional arguments that can be provided to
[`epub()`](https://docs.ropensci.org/epubr/reference/epub.md) to:

- select fields, or columns of the primary data frame.
- filter sections, or rows of the nested data frame.
- attempt to detect which rows or sections in the nested data frame
  identify book chapters.

Unless you have a collection of well-formatted and similarly formatted
EPUB files, these arguments may not be helpful and can be ignored,
especially chapter detection.

### Select fields

Selecting fields is straightforward. All fields found are returned
unless a vector of fields is provided.

``` r

epub(file, fields = c("title", "creator", "file"))
#> # A tibble: 1 × 4
#>   title   creator     file         data             
#>   <chr>   <chr>       <chr>        <list>           
#> 1 Dracula Bram Stoker dracula.epub <tibble [15 × 4]>
```

Note that `file` was not a field identified in the metadata. This is a
special case. Including `file` will include the `basename` of the input
file. This is helpful when you want to retain file names and `source` is
included in the metadata but may represent something else. Some fields
like `data` and `title` are always returned and do not need to be
specified in `fields`.

Also, if your e-book does not have a metadata field named `title`, you
can pass an additional argument to `...` to map a different, known
metadata field to `title`. E.g., `title = "BookTitle"`. The resulting
table always has a `title` field, but in this case `title` would be
populated with information from the `BookTitle` metadata field. If the
default `title` field or any other field name passed to the additional
`title` argument does not exist in the file metadata, the output `title`
column falls back on filling in with the same unique file names obtained
when requesting the `file` field.

### Drop sections

Filtering out unwanted sections, or rows of the nested data frame, uses
a regular expression pattern. Matched rows are dropped. This is where
knowing the naming conventions used in the e-books in `file`, or at
least knowing they are satisfactorily consistent and predictable for a
collection, helps with removing extraneous clutter.

One section that can be discarded is the cover. For many books it can be
helpful to use a pattern like `"^(C|c)ov"` to drop any sections whose
IDs begin with `Cov`, `cov`, and may be that abbreviation or the full
word. For this book, `cov` suffices. The nested data frame has one less
row than before.

``` r

epub(file, drop_sections = "cov")$data[[1]]
#> # A tibble: 14 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 item6   "The Project Gutenberg EBook of Dracula, by Bram StokerT… 11446 60972
#>  2 item7   "But I am not in heart to describe beauty, for when I ha… 13879 71798
#>  3 item8   "\" 'Lucy, you are an honest-hearted girl, I know. I sho… 12474 65522
#>  4 item9   "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, 11 o'clock… 12177 62724
#>  5 item10  "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur Holmwood.\… 12806 66678
#>  6 item11  "Once again we went through that ghastly operation. I ha… 12103 62949
#>  7 item12  "CHAPTER XIVMINA HARKER'S JOURNAL\n23 September.-Jonatha… 12214 62234
#>  8 item13  "CHAPTER XVIDR. SEWARD'S DIARY-continued\nIT was just a … 13990 72903
#>  9 item14  "\"Thus when we find the habitation of this man-that-was… 13356 69779
#> 10 item15  "\"I see,\" I said. \"You want big things that you can m… 12866 66921
#> 11 item16  "CHAPTER XXIIIDR. SEWARD'S DIARY\n3 October.-The time se… 11928 61550
#> 12 item17  "CHAPTER XXVDR. SEWARD'S DIARY\n11 October, Evening.-Jon… 13119 68564
#> 13 item18  " \nLater.-Dr. Van Helsing has returned. He has got the …  8435 43464
#> 14 item19  "End of the Project Gutenberg EBook of Dracula, by Bram …  2665 18541
```

### Guess chapters

This e-book unfortunately does not have great formatting. For the sake
of example, pretend that chapters are known to be sections beginning
with `item` and followed by *two* digits, using the pattern
`^item\\d\\d`. This does two things. It adds a new metadata column to
the primary data frame called `nchap` giving the estimated number of
chapters in the book. In the nested data frame containing the parsed
e-book text, the `section` column is conditionally mutated to reflect a
new, consistent chapter naming convention for the identified chapters
and a logical `is_chapter` column is added.

``` r

x <- epub(file, drop_sections = "cov", chapter_pattern = "^item\\d\\d")
x
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    10 <tibble>

x$data[[1]]
#> # A tibble: 14 × 5
#>    section text                                           is_chapter nword nchar
#>    <chr>   <chr>                                          <lgl>      <int> <int>
#>  1 item6   "The Project Gutenberg EBook of Dracula, by B… FALSE      11446 60972
#>  2 item7   "But I am not in heart to describe beauty, fo… FALSE      13879 71798
#>  3 item8   "\" 'Lucy, you are an honest-hearted girl, I … FALSE      12474 65522
#>  4 item9   "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day,… FALSE      12177 62724
#>  5 ch01    "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur… TRUE       12806 66678
#>  6 ch02    "Once again we went through that ghastly oper… TRUE       12103 62949
#>  7 ch03    "CHAPTER XIVMINA HARKER'S JOURNAL\n23 Septemb… TRUE       12214 62234
#>  8 ch04    "CHAPTER XVIDR. SEWARD'S DIARY-continued\nIT … TRUE       13990 72903
#>  9 ch05    "\"Thus when we find the habitation of this m… TRUE       13356 69779
#> 10 ch06    "\"I see,\" I said. \"You want big things tha… TRUE       12866 66921
#> 11 ch07    "CHAPTER XXIIIDR. SEWARD'S DIARY\n3 October.-… TRUE       11928 61550
#> 12 ch08    "CHAPTER XXVDR. SEWARD'S DIARY\n11 October, E… TRUE       13119 68564
#> 13 ch09    " \nLater.-Dr. Van Helsing has returned. He h… TRUE        8435 43464
#> 14 ch10    "End of the Project Gutenberg EBook of Dracul… TRUE        2665 18541
```

This renaming of sections is helpful if the e-book content is split into
meaningful sections, but the sections are not named in a similarly
meaningful way. It is not helpful in cases where the sections are poorly
or arbitrarily defined as in the above example where they begin with
`item*` regardless of content. See the examples below on restructuring
parsed content for an approach to dealing with this more problematic
case.

Another problematic formatting that you may encounter is the random
order of book sections as defined in the metadata, even if the sections
themselves are well-defined. For example, there may be clear breaks
between chapters, but the chapters may occur out of order such as
`ch05, ch11, ch02, ...` and so on. Merely identifying which sections are
chapters does not help in this case. They are still be labeled
incorrectly since they are already out of order. Properly reordering
sections is also addressed below.

Also note that not all books have chapters or something like them. Make
sure an optional argument like `chapter_pattern` makes sense to use with
a given e-book in the first place.

Ultimately, everything depends on the quality of the EPUB file. Some
publishers are better than others. Formatting standards may also change
over time.

## Restructure parsed content

When reading EPUB files it is ideal to be able to identify meaningful
sections to retain via a regular expression pattern, as well as to drop
extraneous sections in a similar manner. Using pattern matching as shown
above is a convenient way to filter rows of the nested text content data
frame.

For e-books with poor metadata formatting this is not always possible,
or may be possible only after some other pre-processing. `epubr`
provides other functions to assist in restructuring the text table. The
Dracula EPUB file included in `epubr` is a good example to continue with
here.

### Split and recombine into new sections

This book is internally broken into sections at arbitrary break points,
hence why several sections begin in the middle of chapters, as seen
above. Other chapters begin in the middle of sections. Use
[`epub_recombine()`](https://docs.ropensci.org/epubr/reference/epub_recombine.md)
along with a regular expression that can match the true section breaks.
This function collapses the full text and then rebuilds the text table
using new sections with proper break points. In the process it also
recalculates the numbers of words and characters and relabels the
sections with chapter notation.

Fortunately, a reliable pattern exists, which consists of `CHAPTER` in
capital letters followed by a space and some Roman numerals. Recombine
the text into a new object.

``` r

pat <- "CHAPTER [IVX]+"
x2 <- epub_recombine(x, pat)
x2
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    54 <tibble>

x2$data[[1]]
#> # A tibble: 55 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 prior   "The Project Gutenberg EBook of Dracula, by Bram StokerT…   159  1110
#>  2 ch01    "CHAPTER I\nPage\nJonathan Harker's Journal\n1\n"             7    43
#>  3 ch02    "CHAPTER II\nJonathan Harker's Journal\n14\n"                 6    40
#>  4 ch03    "CHAPTER III\nJonathan Harker's Journal\n26\n"                6    41
#>  5 ch04    "CHAPTER IV\nJonathan Harker's Journal\n38\n"                 6    40
#>  6 ch05    "CHAPTER V\nLetters-Lucy and Mina\n51\n"                      6    35
#>  7 ch06    "CHAPTER VI\nMina Murray's Journal\n59\n"                     6    36
#>  8 ch07    "CHAPTER VII\nCutting from \"The Dailygraph,\" 8 August\…     9    55
#>  9 ch08    "CHAPTER VIII\nMina Murray's Journal\n84\n"                   6    38
#> 10 ch09    "CHAPTER IX\nMina Murray's Journal\n98\n"                     6    36
#> # ℹ 45 more rows
```

But this is not quite as expected. There should be 27 chapters, not 54.
What was not initially apparent was that the same pattern matching each
chapter name also appears in the first section where every chapter is
listed in the table of contents. The new section breaks were successful
in keeping chapter text in single, unique sections, but there are now
twice as many as needed. Unintentionally, the first 27 “chapters”
represent the table of contents being split on each chapter ID. These
should be removed.

An easy way to do this is with
[`epub_sift()`](https://docs.ropensci.org/epubr/reference/epub_sift.md),
which sifts, or filters out, small word- or character-count sections
from the nested data frame. It’s a simple sieve and you can control the
size of the holes with `n`. You can choose `type = "word"` (default) or
`type = "character"`. This is somewhat of a blunt instrument, but is
useful in a circumstance like this one where it is clear it will work as
desired.

``` r

library(dplyr)
x2 <- epub_recombine(x, pat) %>% epub_sift(n = 200)
x2
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    54 <tibble>

x2$data[[1]]
#> # A tibble: 27 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 ch28    "CHAPTER IJONATHAN HARKER'S JOURNAL\n(Kept in shorthand.…  5694 30602
#>  2 ch29    "CHAPTER IIJONATHAN HARKER'S JOURNAL-continued\n5 May.-I…  5476 28462
#>  3 ch30    "CHAPTER IIIJONATHAN HARKER'S JOURNAL-continued\nWHEN I …  5703 29778
#>  4 ch31    "CHAPTER IVJONATHAN HARKER'S JOURNAL-continued\nI AWOKE …  5828 30195
#>  5 ch32    "CHAPTER V\nLetter from Miss Mina Murray to Miss Lucy We…  3546 18004
#>  6 ch33    "CHAPTER VIMINA MURRAY'S JOURNAL\n24 July. Whitby.-Lucy …  5654 29145
#>  7 ch34    "CHAPTER VIICUTTING FROM \"THE DAILYGRAPH,\" 8 AUGUST\n(…  5567 29912
#>  8 ch35    "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, 11 o'clock…  6267 32596
#>  9 ch36    "CHAPTER IX\nLetter, Mina Harker to Lucy Westenra.\n\"Bu…  5910 30129
#> 10 ch37    "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur Holmwood.\…  5932 30730
#> # ℹ 17 more rows
```

This removes the unwanted rows, but one problem remains. Note that
sifting the table sections in this case results in a need to re-apply
[`epub_recombine()`](https://docs.ropensci.org/epubr/reference/epub_recombine.md)
because the sections we removed had nevertheless offset the chapter
indexing. Another call to
[`epub_recombine()`](https://docs.ropensci.org/epubr/reference/epub_recombine.md)
can be chained, but it may be more convenient to use the `sift` argument
to
[`epub_recombine()`](https://docs.ropensci.org/epubr/reference/epub_recombine.md),
which is applied recursively.

``` r

#epub_recombine(x, pat) %>% epub_sift(n = 200) %>% epub_recombine(pat)
x2 <- epub_recombine(x, pat, sift = list(n = 200))
x2
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    27 <tibble>

x2$data[[1]]
#> # A tibble: 27 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 ch01    "CHAPTER IJONATHAN HARKER'S JOURNAL\n(Kept in shorthand.…  5694 30602
#>  2 ch02    "CHAPTER IIJONATHAN HARKER'S JOURNAL-continued\n5 May.-I…  5476 28462
#>  3 ch03    "CHAPTER IIIJONATHAN HARKER'S JOURNAL-continued\nWHEN I …  5703 29778
#>  4 ch04    "CHAPTER IVJONATHAN HARKER'S JOURNAL-continued\nI AWOKE …  5828 30195
#>  5 ch05    "CHAPTER V\nLetter from Miss Mina Murray to Miss Lucy We…  3546 18005
#>  6 ch06    "CHAPTER VIMINA MURRAY'S JOURNAL\n24 July. Whitby.-Lucy …  5654 29145
#>  7 ch07    "CHAPTER VIICUTTING FROM \"THE DAILYGRAPH,\" 8 AUGUST\n(…  5567 29912
#>  8 ch08    "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, 11 o'clock…  6267 32596
#>  9 ch09    "CHAPTER IX\nLetter, Mina Harker to Lucy Westenra.\n\"Bu…  5910 30129
#> 10 ch10    "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur Holmwood.\…  5932 30730
#> # ℹ 17 more rows
```

### Reorder sections based on pattern in text

Some poorly formatted e-books have their internal sections occur in an
arbitrary order. This can be frustrating to work with when doing text
analysis on each section and where order matters. Just like recombining
into new sections based on a pattern, sections that are out of order can
be reordered based on a pattern. This requires a bit more work. In this
case the user must provide a function that will map something in the
matched pattern to an integer representing the desired row index.

Continue with the Dracula example, but with one difference. Even though
the sections were originally broken at arbitrary points, they were in
chronological order. To demonstrate the utility of
[`epub_reorder()`](https://docs.ropensci.org/epubr/reference/epub_reorder.md),
first randomize the rows so that chronological order can be recovered.

``` r

set.seed(1)
x2$data[[1]] <- sample_frac(x2$data[[1]]) # randomize rows for example
x2$data[[1]]
#> # A tibble: 27 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 ch25    "CHAPTER XXVDR. SEWARD'S DIARY\n11 October, Evening.-Jon…  6214 32544
#>  2 ch04    "CHAPTER IVJONATHAN HARKER'S JOURNAL-continued\nI AWOKE …  5828 30195
#>  3 ch07    "CHAPTER VIICUTTING FROM \"THE DAILYGRAPH,\" 8 AUGUST\n(…  5567 29912
#>  4 ch01    "CHAPTER IJONATHAN HARKER'S JOURNAL\n(Kept in shorthand.…  5694 30602
#>  5 ch02    "CHAPTER IIJONATHAN HARKER'S JOURNAL-continued\n5 May.-I…  5476 28462
#>  6 ch11    "CHAPTER XI\nLucy Westenra's Diary.\n12 September.-How g…  5126 26926
#>  7 ch14    "CHAPTER XIVMINA HARKER'S JOURNAL\n23 September.-Jonatha…  6411 32530
#>  8 ch18    "CHAPTER XVIIIDR. SEWARD'S DIARY\n30 September.-I got ho…  6911 35881
#>  9 ch19    "CHAPTER XIXJONATHAN HARKER'S JOURNAL\n1 October, 5 a. m…  5670 29431
#> 10 ch24    "CHAPTER XXIVDR. SEWARD'S PHONOGRAPH DIARY, SPOKEN BY VA…  6272 32065
#> # ℹ 17 more rows
```

It is clear above that sections are now out of order. It is common
enough to load poorly formatted EPUB files and yield this type of
result. If all you care about is the text in its entirely, this does not
matter, but if your analysis involves trends over the course of a book,
this is problematic.

For this book, you need a function that will convert an annoying Roman
numeral to an integer. You already have the pattern for finding the
relevant information in each text section. You only need to tweak it for
proper substitution. Here is an example:

``` r

f <- function(x, pattern) as.numeric(as.roman(gsub(pattern, "\\1", x)))
```

This function is passed to
[`epub_reorder()`](https://docs.ropensci.org/epubr/reference/epub_reorder.md).
It takes and returns scalars. It must take two arguments: the first is a
text string. The second is the regular expression. It must return a
single number representing the index of that row. For example, if the
pattern matches `CHAPTER IV`, the function should return a `4`.

[`epub_reorder()`](https://docs.ropensci.org/epubr/reference/epub_reorder.md)
takes care of the rest. It applies your function to every row in the the
nested data frame and then reorders the rows based on the full set of
indices. Note that it also repeats this for every row (book) in the
primary data frame, i.e., for every nested table. This means that the
same function will be applied to every book. Therefore, you should only
use this in bulk on a collection of e-books if you know the pattern does
not change and the function will work correctly in each case.

The pattern has changed slightly. Parentheses are used to retain the
important part of the matched pattern, the Roman numeral. The function
`f` here substitutes the entire string (because now it begins with `^`
and ends with `.*`) with only the part stored in parentheses (In `f`,
this is the `\\1` substitution).
[`epub_reorder()`](https://docs.ropensci.org/epubr/reference/epub_reorder.md)
applies this to all rows in the nested data frame:

``` r

x2 <- epub_reorder(x2, f, "^CHAPTER ([IVX]+).*")
x2$data[[1]]
#> # A tibble: 27 × 4
#>    section text                                                      nword nchar
#>    <chr>   <chr>                                                     <int> <int>
#>  1 ch01    "CHAPTER IJONATHAN HARKER'S JOURNAL\n(Kept in shorthand.…  5694 30602
#>  2 ch02    "CHAPTER IIJONATHAN HARKER'S JOURNAL-continued\n5 May.-I…  5476 28462
#>  3 ch03    "CHAPTER IIIJONATHAN HARKER'S JOURNAL-continued\nWHEN I …  5703 29778
#>  4 ch04    "CHAPTER IVJONATHAN HARKER'S JOURNAL-continued\nI AWOKE …  5828 30195
#>  5 ch05    "CHAPTER V\nLetter from Miss Mina Murray to Miss Lucy We…  3546 18005
#>  6 ch06    "CHAPTER VIMINA MURRAY'S JOURNAL\n24 July. Whitby.-Lucy …  5654 29145
#>  7 ch07    "CHAPTER VIICUTTING FROM \"THE DAILYGRAPH,\" 8 AUGUST\n(…  5567 29912
#>  8 ch08    "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, 11 o'clock…  6267 32596
#>  9 ch09    "CHAPTER IX\nLetter, Mina Harker to Lucy Westenra.\n\"Bu…  5910 30129
#> 10 ch10    "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur Holmwood.\…  5932 30730
#> # ℹ 17 more rows
```

It is important that this is done on a nested data frame that has
already been cleaned to the point of not containing extraneous rows that
cannot be matched by the desired pattern. If they cannot be matched,
then it is unknown where those rows should be placed relative to the
others.

If sections are both out of order and use arbitrary break points, it
would be necessary to reorder them before you split and recombine. If
you split and recombine first, this would yield new sections that
contain text from different parts of the e-book. However, the two are
not likely to occur together; in fact it may be impossible for an EPUB
file to be structured this way. In developing `epubr`, no such examples
have been encountered. In any event, reordering out of order sections
essentially requires a human-identifiable pattern near the beginning of
each section text string, so it does not make sense to perform this
operation unless the sections have meaningful break points.

## Unzip EPUB file

Separate from using
[`epub_meta()`](https://docs.ropensci.org/epubr/reference/epub.md) and
[`epub()`](https://docs.ropensci.org/epubr/reference/epub.md), you can
call [`epub_unzip()`](https://docs.ropensci.org/epubr/reference/epub.md)
directly if all you want to do is extract the files from the `.epub`
file archive. By default the archive files are extracted to
[`tempdir()`](https://rdrr.io/r/base/tempfile.html) so you may want to
change this with the `exdir` argument.

``` r

bookdir <- file.path(tempdir(), "dracula")
epub_unzip(file, exdir = bookdir)
list.files(bookdir, recursive = TRUE)
#>  [1] "META-INF/container.xml"                                                  
#>  [2] "mimetype"                                                                
#>  [3] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-0.htm.html"   
#>  [4] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-1.htm.html"   
#>  [5] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-10.htm.html"  
#>  [6] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-11.htm.html"  
#>  [7] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-12.htm.html"  
#>  [8] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-13.htm.html"  
#>  [9] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-2.htm.html"   
#> [10] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-3.htm.html"   
#> [11] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-4.htm.html"   
#> [12] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-5.htm.html"   
#> [13] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-6.htm.html"   
#> [14] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-7.htm.html"   
#> [15] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-8.htm.html"   
#> [16] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-9.htm.html"   
#> [17] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@images@colophon.png"
#> [18] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@images@cover.jpg"   
#> [19] "OEBPS/0.css"                                                             
#> [20] "OEBPS/1.css"                                                             
#> [21] "OEBPS/content.opf"                                                       
#> [22] "OEBPS/pgepub.css"                                                        
#> [23] "OEBPS/toc.ncx"                                                           
#> [24] "OEBPS/wrap0000.html"
```

## Other functions

### Word count

The helper function
[`count_words()`](https://docs.ropensci.org/epubr/reference/count_words.md)
provides word counts for strings, but allows you to control the regular
expression patterns used for both splitting the string and conditionally
counting the resulting character elements. This is the same function
used internally by
[`epub()`](https://docs.ropensci.org/epubr/reference/epub.md) and
[`epub_recombine()`](https://docs.ropensci.org/epubr/reference/epub_recombine.md).
It is exported so that it can be used directly.

By default,
[`count_words()`](https://docs.ropensci.org/epubr/reference/count_words.md)
splits on spaces and new line characters. It counts as a word any
element containing at least one alphanumeric character or the ampersand.
It ignores everything else as noise, such as extra spaces, empty strings
and isolated bits of punctuation.

``` r

x <- " This   sentence will be counted to have:\n\n10 (ten) words."
count_words(x)
#> [1] 10
```

### Inspection

Helper functions for inspecting the text in the R console include
[`epub_head()`](https://docs.ropensci.org/epubr/reference/epub_head.md)
and
[`epub_cat()`](https://docs.ropensci.org/epubr/reference/epub_cat.md).

[`epub_head()`](https://docs.ropensci.org/epubr/reference/epub_head.md)
provides an overview of the text by section for each book in the primary
data frame. The nested data frames are unnested and row bound to one
another and returned as a single data frame. The text is shortened to
only the first few characters (defaults to `n = 50`).

[`epub_cat()`](https://docs.ropensci.org/epubr/reference/epub_cat.md)
can be used to `cat` the text of an e-book to the console for quick
inspection in a more readable form. It can take several arguments that
help slice out a section of the text and customize how it is printed.

Both functions can take an EPUB filename or a data frame of an already
loaded EPUB file as their first argument.

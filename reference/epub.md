# Extract and read EPUB e-books

Read EPUB format e-books into a data frame using `epub` or extract EPUB
archive files for direct use with `epub_unzip`.

## Usage

``` r
epub(
  file,
  fields = NULL,
  drop_sections = NULL,
  chapter_pattern = NULL,
  encoding = "UTF-8",
  ...
)

epub_meta(file)

epub_unzip(file, exdir = tempdir())
```

## Arguments

- file:

  character, input EPUB filename. May be a vector for `epub` and
  `epub_meta`. Always a single file for `epub_unzip`.

- fields:

  character, vector of metadata fields (data frame columns) to parse
  from metadata, if they exist. See details.

- drop_sections:

  character, a regular expression pattern string to identify text
  sections (rows of nested text data frame) to drop.

- chapter_pattern:

  character, a regular expression pattern string to attempt
  distinguishing nested data frame rows of chapter text entries from
  other types of entries.

- encoding:

  character, defaults to `"UTF-8"`.

- ...:

  additional arguments. With the exception of passing `title` (see
  details), currently developmental/unsupported.

- exdir:

  for `epub_unzip`, extraction directory to place archive contents
  (files). It will be created if necessary.

## Value

`epub` returns a data frame. `epub_unzip` returns nothing but extracts
files from an EPUB file archive.

## Details

The primary function here is `epub`. It parses EPUB file metadata and
textual content into a data frame. The output data frame has one row for
each file in `file`. It has metadata in all columns except the `data`
column, which is a column of nested data frames containing e-book text
by book section (e.g., chapters). Both the primary and nested data
frames are tibbles and safe to print to the console "as is".

Be careful if `file` is a long vector of many EPUB files. This could
take a long time to process as well as could potentially use up all of
your system RAM if you have far too many large books in one call to
`epub`.

On a case by case basis, you can always select columns and filter rows
of a resulting data frame for a single e-book subsequent to visual
inspection. However, the optional arguments `fields`, `drop_sections`
and `chapter_pattern` allow you to do some of this as part of the EPUB
file reading process. You can ignore these arguments and do all your own
post-processing of the resulting data frame, but if using these
arguments, they are most likely to be useful for bulk e-book processing
where `file` is a vector of like-formatted files.

### Main columns

The `fields` argument can be used to limit the columns returned in the
primary data frame. E.g.,
`fields = c("title", "creator", "date", "identifier", "publisher", "file")`.
Some fields will be returned even if not in `fields`, such as `data` and
`title`.  
  
Ideally, you should already know what metadata fields are in the EPUB
file. This is not possible for large collections with possibly different
formatting. Note that when `"file"` is included in `fields`, the output
will include a column of the original file names, in case this is
different from the content of a `source` field that may be present in
the metadata. So this field is always available even if not part of the
file metadata.  
  
Additionally, if there is no `title` field in the metadata, the output
data frame will include a `title` column filled in with the same file
names, unless you pass the additional optional title argument, e.g.
`title = "TitleFieldID"` so that another field can me mapped to `title`.
If supplying a `title` argument that also does not match an existing
field in the e-book, the output `title` column will again default to
file names. File names are the fallback option because unlike e-book
metadata fields, file names always exist and should also always be
unique when performing vectorized reads over multiple books, ensuring
that `title` can be a column in the output data frame that uniquely
identifies different e-books even if the books did not have a `title`
field in their metadata.  
  
Columns of the nested data frames in `data` are fixed. Select from these
in subsequent data frame manipulations.

### Nested rows

The `chapter_pattern` argument may be helpful for bulk processing of
similarly formatted EPUB files. This should be ignored for poorly
formatted EPUB files or where there is inconsistent naming across an
e-book collection. Like with `fields`, you should explore file metadata
in advance or this argument will not be useful. If provided, a column
`nchap` is added to the output data frame giving the guessed number of
chapters. In the `data` column, the `section` column of the nested data
frames will also be updated to reflect guessed chapters with new,
consistent chapter IDs, always beginning with `ch` and ending with
digits.  
  
The `drop_sections` argument also uses regular expression pattern
matching like `chapter_pattern` and operates on the same `section`
column. It simply filters out any matched rows. This is useful for
dropping rows that may pertain to book cover, copyright and
acknowledgements pages, and other similar, clearly non-chapter text,
e-book sections. An example that might work for many books could be
`drop_sections = "^co(v|p)|^ack"`  
  
Rows of the primary data frame are fixed. Filter or otherwise manipulate
these in subsequent data frame manipulations. There is one row per file
so filtering does not make sense to do as part of the initial file
reading.

### EPUB metadata

Use `epub_meta` to return a data frame of only the metadata for each
file in `file`. This skips the reading of each file's text contents,
strictly parsing the metadata. It returns a data frame with one row for
each file and `n` columns where `n` is equal to the union of all fields
identified across all files in `file`. Fields available for at least one
e-book in `file` will return `NA` in that column for any row pertaining
to an e-book that does not have that field in its metadata. If the
metadata contains multiple entries for a field, such as multiple
subjects or publication dates, they are collapsed using the pipe
character.

### Unzipping EPUB files

If using `epub_unzip` directly on individual EPUB files, this gives you
control over where to extract archive files to and what to do with them
subsequently. `epub` and `epub_meta` use `epub_unzip` internally to
extract EPUB archive files to the R session temp directory (with
[`tempdir()`](https://rdrr.io/r/base/tempfile.html)). You do not need to
use `epub_unzip` directly prior to using these other functions. It is
only needed if you want the internal files for some other purpose in or
out of R.

## Examples

``` r
# Use a local example EPUB file included in the package
file <- system.file("dracula.epub", package = "epubr")
bookdir <- file.path(tempdir(), "dracula")
epub_unzip(file, exdir = bookdir) # unzip to directly inspect archive files
list.files(bookdir, recursive = TRUE)
#>  [1] "META-INF/container.xml"                                                  
#>  [2] "OEBPS/0.css"                                                             
#>  [3] "OEBPS/1.css"                                                             
#>  [4] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-0.htm.html"   
#>  [5] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-1.htm.html"   
#>  [6] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-10.htm.html"  
#>  [7] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-11.htm.html"  
#>  [8] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-12.htm.html"  
#>  [9] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-13.htm.html"  
#> [10] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-2.htm.html"   
#> [11] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-3.htm.html"   
#> [12] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-4.htm.html"   
#> [13] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-5.htm.html"   
#> [14] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-6.htm.html"   
#> [15] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-7.htm.html"   
#> [16] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-8.htm.html"   
#> [17] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@345-h-9.htm.html"   
#> [18] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@images@colophon.png"
#> [19] "OEBPS/@public@vhost@g@gutenberg@html@files@345@345-h@images@cover.jpg"   
#> [20] "OEBPS/content.opf"                                                       
#> [21] "OEBPS/pgepub.css"                                                        
#> [22] "OEBPS/toc.ncx"                                                           
#> [23] "OEBPS/wrap0000.html"                                                     
#> [24] "mimetype"                                                                

# \donttest{
epub_meta(file) # parse EPUB file metadata only
#> # A tibble: 1 × 8
#>   rights                  identifier creator title language subject date  source
#>   <chr>                   <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr> 
#> 1 Public domain in the U… http://ww… Bram S… Drac… en       Horror… 1995… http:…

x <- epub(file) # parse entire e-book
x
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

epub(file, fields = c("title", "creator"), drop_sections = "^cov")
#> # A tibble: 1 × 3
#>   title   creator     data             
#>   <chr>   <chr>       <list>           
#> 1 Dracula Bram Stoker <tibble [14 × 4]>
# }
```

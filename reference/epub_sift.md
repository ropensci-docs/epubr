# Sift EPUB sections

Sift out EPUB sections that have suspiciously low word or character
count.

## Usage

``` r
epub_sift(data, n, type = c("word", "char"))
```

## Arguments

- data:

  a data frame created by `epub`.

- n:

  integer, minimum number of words or characters to retain a section.

- type:

  character, `"word"` or `"character"`.

## Value

a data frame

## Details

This function is like a sieve that lets small section rows fall through.
Choose the minimum number of words or characters to accept as a
meaningful section in the e-book worth retaining in the nested data
frame, e.g., book chapters. Data frame rows pertaining to smaller
sections are dropped.

This function is helpful for isolating meaningful content by removing
extraneous e-book sections that may be difficult to remove by other
methods when working with poorly formatted e-books. The EPUB file
included in `epubr` is a good example of this. It does not contain
meaningful section identifiers in its metadata. This creates a need to
restructure the text table after reading it with `epub` by subsequently
calling `epub_recombine`. However, some unavoidable ambiguity in this
leads to many small sections appearing from the table of contents. These
can then be dropped with `epub_sift`. See a more comprehensive in the
[`epub_recombine`](https://docs.ropensci.org/epubr/reference/epub_recombine.md)
documentation. A simpler example is shown below.

## See also

[`epub_recombine`](https://docs.ropensci.org/epubr/reference/epub_recombine.md)

## Examples

``` r
# \donttest{
file <- system.file("dracula.epub", package = "epubr")
x <- epub(file) # parse entire e-book
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

x <- epub_sift(x, n = 3000) # drops last two sections
x$data[[1]]
#> # A tibble: 13 × 4
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
# }
```

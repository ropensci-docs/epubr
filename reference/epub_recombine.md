# Recombine text sections

Split and recombine EPUB text sections based on regular expression
pattern matching.

## Usage

``` r
epub_recombine(data, pattern, sift = NULL)
```

## Arguments

- data:

  a data frame created by `epub`.

- pattern:

  character, a regular expression.

- sift:

  `NULL` or a named list of parameters passed to
  [`epub_sift`](https://docs.ropensci.org/epubr/reference/epub_sift.md).
  See details.

## Value

a data frame

## Details

This function takes a regular expression and uses it to determine new
break points for the full e-book text. This is particularly useful when
sections pulled from EPUB metadata have arbitrary breaks and the text
contains meaningful breaks at random locations in various sections.
`epub_recombine` collapses the text and then creates a new nested data
frame containing new chapter/section labels, word counts and character
counts, associated with the text based on the new break points.

Usefulness depends on the quality of the e-book. While this function
exists to improve the data structure of e-book content parsed from
e-books with poor metadata formatting, it still requires original
formatting that will at least allow such an operation to be successful,
specifically a consistent, non-ambiguous regular expression pattern. See
examples below using the built in e-book dataset.

When used in conjunction with `epub_sift` via the `sift` argument,
recombining and resifting is done recursively. This is because it is
possible that sifting can create a need to rerun the recombine step in
order to regenerate proper chapter indexing for the section column.
However, recombining a second time does not lead to a need to resift, so
recursion ends after one round regardless.

This is a convenient way to avoid the syntax:

`epub_recombine([args]) %>% epub_sift([args]) %>% epub_recombine([args])`.

## See also

[`epub_sift`](https://docs.ropensci.org/epubr/reference/epub_sift.md)

## Examples

``` r
# \donttest{
file <- system.file("dracula.epub", package = "epubr")
x <- epub(file) # parse entire e-book
x$data[[1]] # note arbitrary section breaks (not between chapters)
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

pat <- "CHAPTER [IVX]+" # but a reliable pattern exists for new breaks
epub_recombine(x, pat) # not as expected; pattern also in table of contents
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    54 <tibble>

epub_recombine(x, pat, sift = list(n = 1000)) # sift low word-count sections
#> # A tibble: 1 × 10
#>   rights   identifier creator title language subject date  source nchap data    
#>   <chr>    <chr>      <chr>   <chr> <chr>    <chr>   <chr> <chr>  <int> <list>  
#> 1 Public … http://ww… Bram S… Drac… en       Horror… 1995… http:…    27 <tibble>
# }
```

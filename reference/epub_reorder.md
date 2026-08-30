# Reorder sections

Reorder text sections in an e-book based on a user-provided function.

## Usage

``` r
epub_reorder(data, .f, pattern)
```

## Arguments

- data:

  a data frame created by `epub`.

- .f:

  a scalar function to determine a single row index based on a matched
  regular expression. It must take two strings, the text and the
  pattern, and return a single number. See examples.

- pattern:

  regular expression passed to `.f`.

## Value

a data frame

## Details

Many e-books have chronologically ordered sections based on quality
metadata. This results in properly book sections in the nested data
frame. However, some poorly formatted e-books have their internal
sections occur in an arbitrary order. This can be frustrating to work
with when doing text analysis on each section and where order matters.

This function addresses this case by reordering the text sections in the
nested data frame based on a user-provided function that re-indexes the
data frame rows based on their content. In general, the approach is to
find something in the content of each section that describes the section
order. For example, `epub_recombine` can use a regular expression to
identify chapters. Taking this a step further, `epub_reorder` can use a
function that works with the same information to reorder the rows.

It is enough in the former case to identify where in the text the
pattern occurs. There is no need to extract numeric ordering from it.
The latter takes more effort. In the example EPUB file included in
`epubr`, chapters can be identified using a pattern of the word CHAPTER
in capital letters followed by a space and then some Roman numerals. The
user must provide a function that would parse the Roman numerals in this
pattern so that the rows of the data frame can be reordered properly.

## Examples

``` r
# \donttest{
file <- system.file("dracula.epub", package = "epubr")
x <- epub(file) # parse entire e-book
x <- epub_recombine(x, "CHAPTER [IVX]+", sift = list(n = 1000)) # clean up

library(dplyr)
#> 
#> Attaching package: ‘dplyr’
#> The following objects are masked from ‘package:stats’:
#> 
#>     filter, lag
#> The following objects are masked from ‘package:base’:
#> 
#>     intersect, setdiff, setequal, union
set.seed(1)
x$data[[1]] <- sample_frac(x$data[[1]]) # randomize rows for example
x$data[[1]]
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

f <- function(x, pattern) as.numeric(as.roman(gsub(pattern, "\\1", x)))
x <- epub_reorder(x, f, "^CHAPTER ([IVX]+).*")
x$data[[1]]
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
# }
```

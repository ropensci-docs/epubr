# Preview the first n characters

Preview the first n characters of each EPUB e-book section.

## Usage

``` r
epub_head(x, n = 50)
```

## Arguments

- x:

  a data frame returned by
  [`epub`](https://docs.ropensci.org/epubr/reference/epub.md) or a
  character string giving the EPUB filename(s).

- n:

  integer, first n characters to retain from each e-book section.

## Value

a data frame.

## Details

This function returns a simplified data frame of only the unnested
`section` and `text` columns of a data frame returned by
[`epub`](https://docs.ropensci.org/epubr/reference/epub.md), with the
text included only up to the first `n` characters. This is useful for
previewing the opening text of each e-book section to inspect for
possible useful regular expression patterns to use for text-based
section identification. For example, an e-book may not have meaningful
section IDs that distinguish one type of book section from another, such
as chapters from non-chapter sections, but the text itself may contain
this information at or near the start of a section.

## See also

[`epub_cat`](https://docs.ropensci.org/epubr/reference/epub_cat.md)

## Examples

``` r
# \donttest{
file <- system.file("dracula.epub", package = "epubr")
epub_head(file)
#> # A tibble: 15 × 2
#>    section           text                                                   
#>    <chr>             <chr>                                                  
#>  1 item6             "The Project Gutenberg EBook of Dracula, by Bram St"   
#>  2 item7             "But I am not in heart to describe beauty, for when"   
#>  3 item8             "\" 'Lucy, you are an honest-hearted girl, I know. I"  
#>  4 item9             "CHAPTER VIIIMINA MURRAY'S JOURNAL\nSame day, 11 o'c"  
#>  5 item10            "CHAPTER X\nLetter, Dr. Seward to Hon. Arthur Holmwo"  
#>  6 item11            "Once again we went through that ghastly operation."   
#>  7 item12            "CHAPTER XIVMINA HARKER'S JOURNAL\n23 September.-Jon"  
#>  8 item13            "CHAPTER XVIDR. SEWARD'S DIARY-continued\nIT was jus"  
#>  9 item14            "\"Thus when we find the habitation of this man-that"  
#> 10 item15            "\"I see,\" I said. \"You want big things that you can"
#> 11 item16            "CHAPTER XXIIIDR. SEWARD'S DIARY\n3 October.-The tim"  
#> 12 item17            "CHAPTER XXVDR. SEWARD'S DIARY\n11 October, Evening."  
#> 13 item18            " \nLater.-Dr. Van Helsing has returned. He has got "  
#> 14 item19            "End of the Project Gutenberg EBook of Dracula, by "   
#> 15 coverpage-wrapper ""                                                     
# }
```

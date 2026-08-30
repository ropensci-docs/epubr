# Pretty printing of EPUB text

Print EPUB text to the console in a more readable format.

## Usage

``` r
epub_cat(
  x,
  max_paragraphs = 10,
  skip = 0,
  paragraph_spacing = 1,
  paragraph_indent = 2,
  section_sep = "====",
  book_sep = "====\n===="
)
```

## Arguments

- x:

  a data frame returned by
  [`epub`](https://docs.ropensci.org/epubr/reference/epub.md) or a
  character string giving the EPUB filename(s).

- max_paragraphs:

  integer, maximum number of paragraphs (non-empty lines) to `cat` to
  console.

- skip:

  integer, number of paragraphs to skip.

- paragraph_spacing:

  integer, number of empty lines between paragraphs.

- paragraph_indent:

  integer, number of spaces to indent paragraphs.

- section_sep:

  character, a string to indicate section breaks.

- book_sep:

  character, separator shown between books when `x` has multiple rows
  (books).

## Value

nothing is returned but a more readable format of the text content for
books in `x` is printed to the console.

## Details

This function prints text from EPUB files to the console using `cat`.
This is useful for quickly obtaining an overview of the book text parsed
by [`epub`](https://docs.ropensci.org/epubr/reference/epub.md) that is
easier to read that looking at strings in the table. `max_paragraphs` is
set low by default to prevent accidentally printing entire books to the
console. To print everything in `x`, set `max_paragraphs = NULL`.

## See also

[`epub_head`](https://docs.ropensci.org/epubr/reference/epub_head.md)

## Examples

``` r
# \donttest{
file <- system.file("dracula.epub", package = "epubr")
d <- epub(file)
epub_cat(d, max_paragraphs = 2, skip = 147)
#>   When the calèche stopped, the driver jumped down and held out his hand to assist me to alight. Again I could not but notice his prodigious strength. His hand actually seemed like a steel vice that could have crushed mine if he had chosen. Then he took out my traps, and placed them on the ground beside me as I stood close to a great door, old and studded with large iron nails, and set in a projecting doorway of massive stone. I could see even in the dim light that the stone was massively carved, but that the carving had been much worn by time and weather. As I stood, the driver jumped again into his seat and shook the reins; the horses started forward, and trap and all disappeared down one of the dark openings.
#> 
#>   I stood in silence where I was, for I did not know what to do. Of bell or knocker there was no sign; through these frowning walls and dark window openings it was not likely that my voice could penetrate. The time I waited seemed endless, and I felt doubts and fears crowding upon me. What sort of place had I come to, and among what kind of people? What sort of grim adventure was it on which I had embarked? Was this a customary incident in the life of a solicitor's clerk sent out to explain the purchase of a London estate to a foreigner? Solicitor's clerk! Mina would not like that. Solicitor-for just before leaving London I got word that my examination was successful; and I am now a full-blown solicitor! I began to rub my eyes and pinch myself to see if I were awake. It all seemed like a horrible nightmare to me, and I expected that I should suddenly awake, and find myself at home, with the dawn struggling in through the windows, as I had now and again felt in the morning after a day of overwork. But my flesh answered the pinching test, and my eyes were not to be deceived. I was indeed awake and among the Carpathians. All I could do now was to be patient, and to wait the coming of the morning.
# }
```

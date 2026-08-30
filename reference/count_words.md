# Word count

Count the number of words in a string.

## Usage

``` r
count_words(x, word_pattern = "[A-Za-z0-9&]", break_pattern = " |\n")
```

## Arguments

- x:

  character, a string containing words to be counted. May be a vector.

- word_pattern:

  character, regular expression to match words. Elements not matched are
  not counted.

- break_pattern:

  character, regular expression to split a string between words.

## Value

an integer

## Details

This function estimates the number of words in strings. Words are first
separated using `break_pattern`. Then the resulting character vector
elements are counted, including only those that are matched by
`word_pattern`. The approach taken is meant to be simple and flexible.

`epub` uses this function internally to estimate the number of words for
each e-book section alongside the use of `nchar` for counting individual
characters. It can be used directly on character strings and is
convenient for applying with different regular expression pattern
arguments as needed.

These two arguments are provided for control, but the defaults are
likely good enough. By default, strings are split only on spaces and new
line characters. The "words" that are counted in the resulting vector
are those that contain any alphanumeric characters or the ampersand.
This means for example that hyphenated words, acronyms and numbers
displayed with digits, are all counted as words. The presence of any
other characters does not negate that a word has been found.

## Examples

``` r
x <- " This   sentence will be counted to have:\n\n10 (ten) words."
count_words(x)
#> [1] 10
```

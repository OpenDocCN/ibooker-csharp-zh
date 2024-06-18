# Chapter 25\. Regular Expressions

The regular expressions language identifies character patterns. The .NET types supporting regular expressions are based on Perl 5 regular expressions and support both search and search/replace functionality.

Regular expressions are used for tasks such as:

*   Validating text input such as passwords and phone numbers

*   Parsing textual data into more structured forms (e.g., a NuGet version string)

*   Replacing patterns of text in a document (e.g., whole words only)

This chapter is split into both conceptual sections teaching the basics of regular expressions in .NET, and reference sections describing the regular expressions language.

All regular expression types are defined in `System.Text.RegularExpressions`.

###### Note

The samples in this chapter are all preloaded into LINQPad, which also includes an interactive RegEx tool (press Ctrl+Shift+F1). An online tool is available at [*http://regexstorm.net/tester*](http://regexstorm.net/tester).

# Regular Expression Basics

One of the most common regular expression operators is a *quantifier*. `?` is a quantifier that matches the preceding item 0 or 1 time. In other words, `?` means *optional*. An item is either a single character or a complex structure of characters in square brackets. For example, the regular expression `"colou?r"` matches `color` and `colour`, but not `colouur`:

[PRE0]

`Regex.Match` searches within a larger string. The object that it returns has properties for the `Index` and `Length` of the match as well as the actual `Value` matched:

[PRE1]

You can think of `Regex.Match` as a more powerful version of the `string`’s `IndexOf` method. The difference is that it searches for a *pattern* rather than a literal string.

The `IsMatch` method is a shortcut for calling `Match` and then testing the `Success` property.

The regular expressions engine works from left to right by default, so only the leftmost match is returned. You can use the `NextMatch` method to return more matches:

[PRE2]

The `Matches` method returns all matches in an array. We can rewrite the preceding example, as follows:

[PRE3]

Another common regular expressions operator is the *alternator*, expressed with a vertical bar, `|`. An alternator expresses alternatives. The following matches “Jen”, “Jenny”, and “Jennifer”:

[PRE4]

The brackets around an alternator separate the alternatives from the rest of the expression.

###### Note

You can specify a timeout when matching regular expressions. If a match operation takes longer than the specified `TimeSpan`, a `RegexMatchTimeoutException` is thrown. This can be useful if your program processes user-supplied regular expressions because it prevents malformed regular expressions from infinitely spinning.

## Compiled Regular Expressions

In some of the preceding examples, we called a static `RegEx` method repeatedly with the same pattern. An alternative approach in these cases is to instantiate a `Regex` object with the pattern and `RegexOptions.Compiled` and then call instance methods:

[PRE5]

`RegexOptions.Compiled` instructs the `RegEx` instance to use lightweight code generation (`DynamicMethod` in `Reflection.Emit`) to dynamically build and compile code tailored to that particular regular expression. This results in faster matching, at the expense of an initial compilation cost.

You can also instantiate a `Regex` object without using `RegexOptions.Compiled`. A `Regex` instance is immutable.

###### Note

The regular expressions engine is fast. Even without compilation, a simple match typically takes less than a microsecond.

## RegexOptions

The `RegexOptions` flags enum lets you tweak matching behavior. A common use for `RegexOptions` is to perform a case-insensitive search:

[PRE6]

This applies the current culture’s rules for case equivalence. The `CultureInvariant` flag lets you request the invariant culture instead:

[PRE7]

You can activate most of the `RegexOptions` flags within a regular expression itself, using a single-letter code, as follows:

[PRE8]

You can turn options on and off throughout an expression:

[PRE9]

Another useful option is `IgnorePatternWhitespace` or `(?x)`. This allows you to insert whitespace to make a regular expression more readable—without the whitespace being taken literally.

The `NonBacktracking` option (from .NET 7) instructs the regex engine to use a forwards-only matching algorithm. This usually results in slower performance and disables some advanced features such as lookahead or lookbehind. However, it also prevents malformed or maliciously constructed expressions from taking near-infinite time, mitigating a potential denial-of-service attack when processing user-supplied regular expressions (a *ReDOS* attack). Specifying a timeout is also useful in this scenario.

[Table 25-1](#regular_expression_options-id00102) lists all `RegExOptions` values along with their single-letter codes.

Table 25-1\. Regular expression options

| Enum value | Regular expressions code | Description |
| --- | --- | --- |
| `None` |   |   |
| `IgnoreCase` | `i` | Ignores case (by default, regular expressions are case sensitive) |
| `Multiline` | `m` | Changes `^` and `$` so that they match the start/end of a line instead of start/end of the string |
| `ExplicitCapture` | `n` | Captures only explicitly named or explicitly numbered groups (see [“Groups”](#groups)) |
| `Compiled` |   | Forces compilation to IL (see [“Compiled Regular Expressions”](#compiled_regular_expressions)) |
| `Singleline` | `s` | Makes `.` match every character (instead of matching every character except `\n`) |
| `IgnorePatternWhitespace` | `x` | Eliminates unescaped whitespace from the pattern |
| `RightToLeft` | `r` | Searches from right to left; can’t be specified midstream |
| `ECMAScript` |   | Forces ECMA compliance (by default, the implementation is not ECMA compliant) |
| `CultureInvariant` |   | Turns off culture-specific behavior for string comparisons |
| `NonBacktracking` |   | Disables backtracking to ensure predictable (albeit slower) performance |

## Character Escapes

Regular expressions have the following metacharacters, which have a special rather than literal meaning:

> \ * + ? | { [ () ^ $ . #

To use a metacharacter literally, you must prefix, or *escape*, the character with a backslash. In the following example, we escape the `?` character to match the string `"what?"`:

[PRE10]

###### Note

If the character is inside a *set* (square brackets), this rule does not apply, and the metacharacters are interpreted literally. We discuss sets in the following section.

The `Regex`’s `Escape` and `Unescape` methods convert a string containing regular expression metacharacters by replacing them with escaped equivalents, and vice versa:

[PRE11]

All the regular expression strings in this chapter are expressed with the C# `@` literal. This is to bypass C#’s escape mechanism, which also uses the backslash. Without the `@`, a literal backslash would require four backslashes:

[PRE12]

Unless you include the `(?x)` option, spaces are treated literally in regular expressions:

[PRE13]

## Character Sets

Character sets act as wildcards for a particular set of characters.

| Expression | Meaning | Inverse (“not”) |
| --- | --- | --- |
| `[abcdef]` | Matches a single character in the list. | `[^abcdef]` |
| `[a-f]` | Matches a single character in a *range.* | `[^a-f]` |
| `\d` | Matches anything in the Unicode *digits* category. In ECMAScript mode, `[0-9]`. | `\D` |
| `\w` | Matches a *word* character (by default, varies according to `CultureInfo.CurrentCulture`; for example, in English, same as `[a-zA-Z_0-9]`). | `\W` |
| `\s` | Matches a whitespace character; that is, anything for which `char.IsWhiteSpace` returns true (including Unicode spaces). In ECMAScript mode, `[\n\r\t\f\v ]`. | `\S` |
| `\p{`*category*`}` | Matches a character in a specified *category*. | `\P` |
| `.` | (Default mode) Matches any character except `\n`. | `\n` |
| `.` | (`SingleLine` mode) Matches any character. | `\n` |

To match exactly one of a set of characters, put the character set in square brackets:

[PRE14]

To match any character *except* those in a set, put the set in square brackets with a `^` symbol before the first character:

[PRE15]

You can specify a range of characters by using a hyphen. The following regular expression matches a chess move:

[PRE16]

`\d` indicates a digit character, so `\d` will match any digit. `\D` matches any nondigit character.

`\w` indicates a word character, which includes letters, numbers, and the underscore. `\W` matches any nonword character. These work as expected for non-English letters, too, such as Cyrillic.

`.` matches any character except `\n` (but allows `\r`).

`\p` matches a character in a specified category, such as `{Lu}` for uppercase letter or `{P}` for punctuation (we list the categories in the reference section later in the chapter):

[PRE17]

We will find more uses for `\d`, `\w`, and `.` when we combine them with *quantifiers*.

# Quantifiers

Quantifiers match an item a specified number of times.

| Quantifier | Meaning |
| --- | --- |
| `*` | Zero or more matches |
| `+` | One or more matches |
| `?` | Zero or one match |
| `{*n*}` | Exactly `*n*` matches |
| `{*n*,}` | At least `*n*` matches |
| `{*n*,*m*}` | Between `*n*` and `*m*` matches |

The `*` quantifier matches the preceding character or group zero or more times. The following matches *cv.docx*, along with any numbered versions of the same file (e.g., *cv2.docx*, *cv15.docx*):

[PRE18]

Notice that we must escape the period in the file extension using a backslash.

The following allows anything between *cv* and *.docx* and is equivalent to `dir cv*.docx`:

[PRE19]

The `+` quantifier matches the preceding character or group one or more times. For example:

[PRE20]

The `{}` quantifier matches a specified number (or range) of repetitions. The following matches a blood pressure reading:

[PRE21]

## Greedy Versus Lazy Quantifiers

By default, quantifiers are *greedy*, as opposed to *lazy*. A greedy quantifier repeats as *many* times as it can before advancing. A lazy quantifier repeats as *few* times as it can before advancing. You can make any quantifier lazy by suffixing it with the `?` symbol. To illustrate the difference, consider the following HTML fragment:

[PRE22]

Suppose that we want to extract the two phrases in italics. If we execute the following

[PRE23]

the result is not two matches, but a *single* match:

[PRE24]

The problem is that our `*` quantifier greedily repeats as many times as it can before matching `</i>`. So, it passes right by the first `</i>`, stopping only at the final `</i>` (the *last point* at which the rest of the expression can still match).

If we make the quantifier lazy, the `*` bails out at the *first* point at which the rest of the expression can match:

[PRE25]

Here’s the result:

[PRE26]

# Zero-Width Assertions

The regular expressions language lets you place conditions on what should occur *before* or *after* a match, through *lookbehind*, *lookahead*, *anchors*, and *word boundaries*. These are called *zero-width* assertions because they don’t increase the width (or length) of the match itself.

## Lookahead and Lookbehind

The `(?=*expr*)` construct checks whether the text that follows matches `*expr*`, without including `expr` in the result. This is called *positive lookahead*. In the following example, we look for a number followed by the word “miles”:

[PRE27]

Notice that the word “miles” was not returned in the result, even though it was required to *satisfy* the match.

After a successful *lookahead*, matching continues as though the sneak preview never took place. So, if we append `.*` to our expression like this:

[PRE28]

the result is `25 miles more`.

Lookahead can be useful in enforcing rules for a strong password. Suppose that a password must be at least six characters and contain at least one digit. With a lookup, we could achieve this, as follows:

[PRE29]

This first performs a *lookahead* to ensure that a digit occurs somewhere in the string. If satisfied, it returns to its position before the sneak preview began and matches six or more characters. (In [“Cookbook Regular Expressions”](#cookbook_regular_expressions), we include a more substantial password validation example.)

The opposite is the *negative lookahead* construct, `(?!*expr*)`. This requires that the match *not* be followed by `*expr*`. The following expression matches “good”—unless “however” or “but” appears later in the string:

[PRE30]

The `(?<=*expr*)` construct denotes *positive lookbehind* and requires that a match be *preceded* by a specified expression. The opposite construct, `(?<!*expr*)`, denotes *negative lookbehind* and requires that a match *not be preceded* by a specified expression. For example, the following matches “good”—unless “however” appears *earlier* in the string:

[PRE31]

We could improve these examples by adding *word boundary assertions*, which we introduce shortly.

## Anchors

The anchors `^` and `$` match a particular *position*. By default:

`^`

Matches the *start* of the string

`$`

Matches the *end* of the string

###### Note

`^` has two context-dependent meanings: an *anchor* and a *character class negator*.

`$` has two context-dependent meanings: an *anchor* and a *replacement group denoter*.

For example:

[PRE32]

When you specify `RegexOptions.Multiline` or include `(?m)` in the expression:

*   `^` matches the start of the string or *line* (directly after a `\n`).

*   `$` matches the end of the string or *line* (directly before a `\n`).

There’s a catch to using `$` in multiline mode: a new line in Windows is nearly always denoted with `\r\n` rather than just `\n`. This means that for `$` to be useful for Windows files, you must usually match the `\r`, as well, with a *positive lookahead*:

[PRE33]

The *positive lookahead* ensures that `\r` doesn’t become part of the result. The following matches lines that end in `".txt"`:

[PRE34]

The following matches all empty lines in string `s`:

[PRE35]

The following matches all lines that are either empty or contain only whitespace:

[PRE36]

###### Note

Because an anchor matches a position rather than a character, specifying an anchor on its own matches an empty string:

[PRE37]

## Word Boundaries

The word boundary assertion `\b` matches where word characters (`\w`) adjoin either:

*   Nonword characters (`\W`)

*   The beginning/end of the string (`^` and `$`)

`\b` is often used to match whole words:

[PRE38]

The following statements highlight the effect of a word boundary:

[PRE39]

The next query uses *positive lookahead* to return words followed by “(sic)”:

[PRE40]

# Groups

Sometimes, it’s useful to separate a regular expression into a series of subexpressions, or *groups*. For instance, consider the following regular expression that represents a US phone number such as 206-465-1918:

[PRE41]

Suppose that we want to separate this into two groups: area code and local number. We can achieve this by using parentheses to *capture* each group:

[PRE42]

We then retrieve the groups programmatically:

[PRE43]

The zeroth group represents the entire match. In other words, it has the same value as the match’s `Value`:

[PRE44]

Groups are part of the regular expressions language itself. This means that you can refer to a group within a regular expression. The `\n` syntax lets you index the group by group number `n` within the expression. For example, the expression `(\w)ee\1` matches `deed` and `peep`. In the following example, we find all words in a string starting and ending in the same letter:

[PRE45]

The brackets around the `\w` instruct the regular expressions engine to store the submatch in a group (in this case, a single letter) so that it can be used later. We refer to that group later using `\1`, meaning the first group in the expression.

## Named Groups

In a long or complex expression, it can be easier to work with groups by *name* rather than index. Here’s a rewrite of the previous example, using a group that we name `'letter'`:

[PRE46]

Here’s how to name a captured group:

[PRE47]

And here’s how to refer to a group:

[PRE48]

The following example matches a simple (non-nested) XML/HTML element by looking for start and end nodes with a matching name:

[PRE49]

Allowing for all possible variations in XML structure, such as nested elements, is more complex. The .NET regular expressions engine has a sophisticated extension called “matched balanced constructs” that can assist with nested tags—information on this is available on the internet and in [*Mastering Regular Expressions*](https://learning.oreilly.com/library/view/mastering-regular-expressions/0596528124/) (O’Reilly) by Jeffrey E. F. Friedl.

# Replacing and Splitting Text

The `RegEx.Replace` method works like `string.Replace` except that it uses a regular expression.

The following replaces “cat” with “dog”. Unlike with `string.Replace`, “catapult” won’t change into “dogapult”, because we match on word boundaries:

[PRE50]

The replacement string can reference the original match with the `$0` substitution construct. The following example wraps numbers within a string in angle brackets:

[PRE51]

You can access any captured groups with `$1`, `$2`, `$3`, and so on, or `${*name*}` for a named group. To illustrate how this can be useful, consider the regular expression in the previous section that matched a simple XML element. By rearranging the groups, we can form a replacement expression that moves the element’s content into an XML attribute:

[PRE52]

Here’s the result:

[PRE53]

## MatchEvaluator Delegate

`Replace` has an overload that takes a `MatchEvaluator` delegate, which is invoked per match. This allows you to delegate the content of the replacement string to C# code when the regular expressions language isn’t expressive enough:

[PRE54]

In [“Cookbook Regular Expressions”](#cookbook_regular_expressions), we show how to use a `Match​Eva⁠luator` to escape Unicode characters appropriately for HTML.

## Splitting Text

The static `Regex.Split` method is a more powerful version of the `string.Split` method, with a regular expression denoting the separator pattern. In this example, we split a string, where any digit counts as a separator:

[PRE55]

The result, here, doesn’t include the separators themselves. You can include the separators, however, by wrapping the expression in a *positive lookahead*. The following splits a camel-case string into separate words:

[PRE56]

# Cookbook Regular Expressions

## Recipes

### Matching US Social Security number/phone number

[PRE57]

### Extracting “name = value” pairs (one per line)

Note that this starts with the *multiline* directive `(?m)`:

[PRE58]

### Strong password validation

The following checks whether a password has at least six characters and whether it contains a digit, symbol, or punctuation mark:

[PRE59]

### Lines of at least 80 characters

[PRE60]

### Parsing dates/times (N/N/N H:M:S AM/PM)

This expression handles a variety of numeric date formats—and works whether the year comes first or last. The `(?x)` directive improves readability by allowing whitespace; the `(?i)` switches off case sensitivity (for the optional AM/PM designator). You can then access each component of the match through the `Groups` collection:

[PRE61]

(Of course, this doesn’t verify that the date/time is correct.)

### Matching Roman numerals

[PRE62]

### Removing repeated words

Here, we capture a named group called `dupe`:

[PRE63]

### Word count

[PRE64]

### Matching a GUID

[PRE65]

### Parsing an XML/HTML tag

Regex is useful for parsing HTML fragments—particularly when the document might be imperfectly formed:

[PRE66]

### Splitting a camel-cased word

This requires a *positive lookahead* to include the uppercase separators:

[PRE67]

### Obtaining a legal filename

[PRE68]

### Escaping Unicode characters for HTML

[PRE69]

### Unescaping characters in an HTTP query string

[PRE70]

### Parsing Google search terms from a web stats log

You should use this in conjunction with the previous example to unescape characters in the query string:

[PRE71]

# Regular Expressions Language Reference

Tables [25-2](#character_escapes) through [25-12](#regular_expression_option) summarize the regular expressions grammar and syntax supported in the .NET implementation.

Table 25-2\. Character escapes

| Escape code sequence | Meaning | Hexadecimal equivalent |
| --- | --- | --- |
| `\a` | Bell | `\u0007` |
| `\b` | Backspace | `\u0008` |
| `\t` | Tab | `\u0009` |
| `\r` | Carriage return | `\u000A` |
| `\v` | Vertical tab | `\u000B` |
| `\f` | Form feed | `\u000C` |
| `\n` | Newline | `\u000D` |
| `\e` | Escape | `\u001B` |
| `\*nnn*` | ASCII character *nnn* as octal (e.g., `\n052`) |   |
| `\x*nn*` | ASCII character *nn* as hex (e.g., `\x3F`) |   |
| `\c*l*` | ASCII control character *l* (e.g., `\cG` for Ctrl-G) |   |
| `\u*nnnn*` | Unicode character *nnnn* as hex (e.g., `\u07DE`) |   |
| `\*symbol*` | A nonescaped symbol |   |

Special case: within a regular expression, `\b` means word boundary, except in a `[ ]` set, in which `\b` means the backspace character.

Table 25-3\. Character sets

| Expression | Meaning | Inverse (“not”) |
| --- | --- | --- |
| `[abcdef]` | Matches a single character in the list | `[^abcdef]` |
| `[a-f]` | Matches a single character in a *range* | `[^a-f]` |
| `\d` | Matches a decimal digit Same as `[0-9]` | `\D` |
| `\w` | Matches a *word* character (by default, varies according to `CultureInfo.CurrentCulture`; for example, in English, same as `[a-zA-Z_0-9]`) | `\W` |
| `\s` | Matches a whitespace character Same as `[\n\r\t\f\v ]` | `\S` |
| `\p{*category*}` | Matches a character in a specified *category* (see [Table 25-4](#character_categories)) | `\P` |
| `.` | (Default mode) Matches any character except `\n` | `\n` |
| `.` | (`SingleLine` mode) Matches any character | `\n` |

Table 25-4\. Character categories

| Quantifier | Meaning |
| --- | --- |
| `\p{L}` | Letters |
| `\p{Lu}` | Uppercase letters |
| `\p{Ll}` | Lowercase letters |
| `\p{N}` | Numbers |
| `\p{P}` | Punctuation |
| `\p{M}` | Diacritic marks |
| `\p{S}` | Symbols |
| `\p{Z}` | Separators |
| `\p{C}` | Control characters |

Table 25-5\. Quantifiers

| Quantifier | Meaning |
| --- | --- |
| `*` | Zero or more matches |
| `+` | One or more matches |
| `?` | Zero or one match |
| `{*n*}` | Exactly `*n*` matches |
| `{*n*,}` | At least `*n*` matches |
| `{*n,m*}` | Between `*n*` and `*m*` matches |

The `?` suffix can be applied to any of the quantifiers to make them *lazy* rather than *greedy*.

Table 25-6\. Substitutions

| Expression | Meaning |
| --- | --- |
| `$0` | Substitutes the matched text |
| `$*group-number*` | Substitutes an indexed `*group-number*` within the matched text |
| `${*group-name*}` | Substitutes a text `*group-name*` within the matched text |

Substitutions are specified only within a replacement pattern.

Table 25-7\. Zero-width assertions

| Expression | Meaning |
| --- | --- |
| `^` | Start of string (or line in *multiline* mode) |
| `$` | End of string (or line in *multiline* mode) |
| `\A` | Start of string (ignores *multiline* mode) |
| `\z` | End of string (ignores *multiline* mode) |
| `\Z` | End of line or string |
| `\G` | Where search started |
| `\b` | On a word boundary |
| `\B` | Not on a word boundary |
| `(?=*expr*)` | Continue matching only if expression `*expr*` matches on right (*positive lookahead*) |
| `(?!*expr*)` | Continue matching only if expression `*expr*` doesn’t match on right (*negative lookahead*) |
| `(?<=*expr*)` | Continue matching only if expression `*expr*` matches on left (*positive lookbehind*) |
| `(?<!*expr*)` | Continue matching only if expression `*expr*` doesn’t match on left (*negative lookbehind*) |
| `(?>*expr*)` | Subexpression `*expr*` is matched once and not backtracked |

Table 25-8\. Grouping constructs

| Syntax | Meaning |
| --- | --- |
| `(*expr*)` | Capture matched expression `*expr*` into indexed group |
| `(?*number*)` | Capture matched substring into a specified group `*number*` |
| `(?'*name*')` | Capture matched substring into group `*name*` |
| `(?'*name1-name2*')` | Undefine `*name2*` and store interval and current group into `*name1*`; if `*name2*` is undefined, matching backtracks |
| `(?:*expr*)` | Noncapturing group |

Table 25-9\. Back references

| Parameter syntax | Meaning |
| --- | --- |
| `\*index*` | Reference a previously captured group by `*index*` |
| `\k<*name*>` | Reference a previously captured group by `*name*` |

Table 25-10\. Alternation

| Expression syntax | Meaning |
| --- | --- |
| `&#124;` | Logical *or* |
| `(?(*expr*)*yes*&#124;*no*)` | Matches `*yes*` if expression matches; otherwise, matches `*no*` (`*no*` is optional) |
| `(?(*name*)*yes*&#124;*no*)` | Matches `*yes*` if named group has a match; otherwise, matches `*no*` (`*no*` is optional) |

Table 25-11\. Miscellaneous constructs

| Expression syntax | Meaning |
| --- | --- |
| `(?#*comment*)` | Inline comment |
| `#*comment*` | Comment to end of line (works only in `IgnorePatternWhitespace` mode) |

Table 25-12\. Regular expression options

| Option | Meaning |
| --- | --- |
| `(?i)` | Case-insensitive match (“ignore” case) |
| `(?m)` | Multiline mode; changes `^` and `$` so that they match beginning and end of any line |
| `(?n)` | Captures only explicitly named or numbered groups |
| `(?c)` | Compiles to Intermediate Language |
| `(?s)` | Single-line mode; changes meaning of “.” so that it matches every character |
| `(?x)` | Eliminates unescaped whitespace from the pattern |
| `(?r)` | Searches from right to left; can’t be specified midstream |
--- 
layout: post
title: RE in Python
comments: true
---

* generate table
{:toc}
---
<br><br>

It is important to note that most regular expression operations are available as module-level functions and `RegexObject` methods. The functions are shortcuts that don't require you to compile a regex object first, but miss some fine-tuning parameters.

##Some Special charaters

For more details about special characters, see [here](https://docs.python.org/2/library/re.html)
<dt markdown="1">`$`</dt>
<dd markdown="1">Matches the end of the string or just before the newline at the end of the string, and in `MULTILINE` mode also matches before a new line. `foo` matches both 'foo' and 'foobar', while the regular expression 'foo$' matches only 'foo'. More interestingly, searching for `foo.$` in 'foo1\nfoo2\n' matches 'foo2' normally, but 'foo1' in `MULTILINE` mode; searching for a single `$` in 'foo\n' will find two (empty) matches: one just before the newline, and one at the end of the string.

***Note that `^$` matches the empty string `""`, but not `"\n"`.***
</dd>

<dt markdown="1">`*?`,`+?`,`??`</dt>
<dd markdown="1"> The `*`,`+`, and `?` qualifiers are all ***greedy***, they match as much text as possible. Sometimes this behaviour isn't desired; if the RE `<.*>` is matched against `<H1>title</H1>`, it will match the entire string, and not just `<H1>`. Adding `?` after the qualifier makes it perform the match in ***non-greedy*** or ***minimal*** fashion; as ***few*** characters as possible will be matched. Using `.*?` in the previous expression will match only `<H1>`.
</dd>

<dt markdown="1">`\`</dt>
<dd markdown="1">Either escapes special characters (permitting you to match characters like `*`, `?`, and so forth), or signals a special sequence; special sequences are discussed below.

If you're not using a raw string to express the pattern, remember that Python also uses the backslash as an escape sequence in string literals; if the escape sequence isn't recognized by Python's parser, the backslash and subsequent character are included in the resulting string. However, if Python would recognize the resulting sequence, the backslash should be repeated twice. This is complicated and hard to understand, so it's highly recommended that you use raw strings for all but the simplest expressions.
</dd>

<dt markdown="1">`[]`</dt>
<dd markdown="1">Used to indicate a set of characters. In a set:
- Characters can be listed individually, e.g., `[amk]` will match `a`, `m`, or `k`.
- Ranges of characters can be indicated by giving two characters and separating them by a `-`, for example `[a-z]` will match any lowercase ASCII letter, `[0-5][0-9]` will match all the two-digits numbers from `00` to `59`, and `[0-9A-Fa-z]` will match any hexadecimal digit. If `-` is escaped (e.g. `[a\-z]`) or if it's placed as the first or last character (e.g. [a-]), it will match a literal `-`.
- Special characters lose their special meaning inside sets. For example, `[(+*)]` will match any of the literal characters `(`, `+`, `*`, or `)`.
- Character classes such as `\w` or `\s` (defined below) are also accepted inside a set, although the characters they match depends on thether `LOCALE` or `UNICODE` mode is in force.
- Characters that are not within a range can be matched by *complementing* the set. If the first character of the set is `^`, all the characters that are *not* in the set will be matched. For example, `[^5]` will match any character except `5`, and `[^^]` will match any character except `^`. `^` has no special meaning if it's not the first character in the set.
- To match a literal `]` inside a set, precede it with a backslash, or place it at the beginning ot the set. For example, both `[()[\]{}]` and `[]()[{}]` will both match a parenthesis.
</dd>

<dt markdown="1">`(...)`</dt>
<dd markdown="1">Matches whatever retular expression is indide the parentheses, and indicated the start and end of a group; cone contents of a group can be retrieved after a match has been performed, and can be matched later in the string with the `\number` special sequence, desribed below. To match the literals `(` or `)`, use `\(` or `\)`, or enclose them indise a character class: `[(][)]`.
</dd>

<dt markdown="1">`(?...)`</dt>
<dd markdown="1">This is an extension notation (a `?` following a `(` is not meaningful otherwise). The first character after the `?` determines what the meaning and further syntax of the construct is. Extensions usually fo not create a new group; `(?P<name>...)` is the only exception to this rule. Following are the currently soported extensions.
</dd>

##Some module contents

<dt markdown="1">`re.compile(pattern, flags=0)`</dt>
<dd markdown="1">
Using `re.compile()` and saving the resulting regular expression object for reuse if more efficient when the expression will be used several times in a single program. The compiled versions of the most recent patterns passed to `re.match()`, `re.search()` or `re.compile()` are cached, so programs that used only a few regular expressions at a time needn't worry about compiling regular expressions.
</dd>

<dt markdown="1">`re.S`</dt>
<dt markdown="1">`re.DOTALL`</dt>
<dd markdown="1">
Make the '.' special character match any character at all, including a newline; without this flag, '.' will match anything except a newline.
</dd>

<dt markdown="1">`re.U`</dt>
<dt markdown="1">`re.UNICODE`</dt>
<dd markdown="1">
Make `\w`, `\W`, `\b`, `\B`, `\d`, `\D`, `\s` and `\S` dependent on the Unicode character properties database.

*New in version 2.0*    
</dd>

<dt markdown="1">`re.X`</dt>
<dt markdown="1">`re.VERBOSE`</dt>
<dd markdown="1">
This flag allows you to write regular expressions that look nicer and are more readable by allowing you to visually seperate logical sections of the pattern and add comments. Whitespace within the pattern is ignored, except when in a character class or when preceded bu an unescaped backslash. When a line contains a `#` that is not in a character class and is not preceded by an unescaped backslash, all characters from the leftmost such `#` through the end of the line are ignored.

This means that the two following regular expression objects that match a decimal number are functionally equal:
{% highlight python linenos %}
a = re.compile(r"""\d + # the integral part
                \.      # the decimal point
                \d *    # some fractional digits""", re.X)
b = re.complie(r"\d+\.\d*")
{% endhighlight %}
</dd>

<dt markdown="1">`re.split(pattern, string, maxsplit=0, flags=0)`</dt>
<dd markdown="1">
If there are capturing groups in the separator and it matches at the start of the string, the result will start with an empty string. The same holds for the end of the string:

{% highlight python linenos %}
>>> re.split('(\W+)', '...words, words...')
['', '...', 'words', ', ', 'words', '...', '']
{% endhighlight %}
</dd>

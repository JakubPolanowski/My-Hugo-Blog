---
title: "Sed is a Stream Text Editor not a Text Extractor"
description: "While sed can ultimately be used to extract text from a string, the process has to be approached as editing the text as opposed to filtering."
date: 2023-03-03T02:14:07-05:00
categories:
    - Programming
tags:
    - Shell
    - Scripting
    - sed
    - Linux
    - Bash
    - Text
    - Regex
---

## Overview

[Sed](https://www.gnu.org/software/sed/manual/sed.html) is a standard Linux utility that can be immensely helpful when working with text. One such use case is to extract sub-strings from string, that being said `sed` is **not** a text extractor, it is a text editor.

That means that rather than telling `sed` what sub-string/pattern you want, you need to tell `sed` how to reconstruct the string according to your desired pattern.

Let's say you have a string `time of completion: 14:24` and you want to extract the time. The `sed` command for this would be...

```bash
sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"
```

So...

```bash
$ echo "time of completion: 14:24" | sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"

14:24
```

## Breakdown

### The Input String

```bash
echo "time of completion: 14:24" | CMD
```

Using `echo` we create a text stream that is then redirect/piped to the next command `CMD` via the `|` pipe. In this `CMD` could be any other command/utility that accepts a text stream such as `cat` or `less`.

### Sed Flags

```bash
sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"
```

This is where the magic actually happens. The `-r` flag is used to enable extended regular expressions, which is often useful but this can also be done without it.

```bash
sed -n "s/.*\([0-9][0-9]:[0-9][0-9]\).*/\1/p"
```

This accomplishes the same task but is less readable so when possible (any modern Linux system) the `-r` flag is preferable.

The the other flag `-n` prevents the automatic printing of the pattern space. The reason why the flag is useful is in the event that the string doesn't have the desired pattern. 

For example...

```bash
$ echo "time of completion: q4:24" | sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"

<no output>
```

We get no output because the time is invalid/doesn't match out format --- `q4:24`.

Now let's see what happens if we don't used the `-n` flag...

```bash
$ echo "time of completion: q4:24" | sed -r "s/.*([0-9]{2}:[0-9]{2}).*/\1/"

time of completion: q4:24
```

Now when there is not a matching pattern, the entire input is printed out, which in most use cases is **not** the desired outcome.

*Note: You may have noticed that in the example with the `-n` flag, the `p` at the end of the `sed` expression is missing, the `p` tells `sed` to print out the output of the expression, if there is an output (as in `sed` is able to match the pattern). If we exclude `-n` but keep the `p` and the pattern is in the input string, the pattern match will be printed out twice, which is why it is excluded.*

```bash
$ echo "time of completion: 14:24" | sed -r "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"

14:24
14:24
```

### Regex

The [regex](https://www.rexegg.com/regex-quickstart.html) (regular expression) is the pattern we're trying to get `sed` to extract from our string. Let's start off with the contents of the capture group.

```re
[0-9]{2}:[0-9]{2}
```

* `[0-9]` specifies that we want a character that is between `0` and `9`, so a numeric digit. 
* `{2}` specifies how many
  * `[0-9]{2}` equates to `[0-9][0-9]`
* Then the `:` is a literal character, since the sub string `14:24` has the literal `:` in it. 

Then that regex is wrapped in parenthesis to specify it as a capture group...

```re
([0-9]{2}:[0-9]{2})
```

So what about the rest?

```re
.*([0-9]{2}:[0-9]{2}).*
```

This goes back to what `sed` is and what `sed` isn't, that being that `sed` is an editor but is **not** an extractor. So you need to account for the entire string, not just the pattern that you want to capture. 

* The `.` means any character, it is a wildcard symbol
  * You can have capture periods explicitly via `\.`
* The `*` specifies how many, that being any (zero or more)
* `.*` essentially means capture everything
* `.*(REGEX).*` will capture the entire string which contains the capture group `(REGEX)`
  * `.*([0-9]{2}:[0-9]{2}).*` captures all strings that have an instance of the `[0-9]{2}:[0-9]{2}` pattern

### Sed Expression

```bash
sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"
```

In generic terms this is...

```bash
sed -r -n "s/REGEX/OUTPUT PATTERN/p"
```

* 's' is the command we are issuing to `sed`, that being substitute command
  * This is by far the most important command and is used 99% of the time
* `REGEX` is our regex
* `OUTPUT PATTERN` is pattern for our output
  * `\1` refers to the first capture group
  * `\2` for the second, `\3` for the third, and so on
* Then after the last `/` we have any additional operations
  * `p` prints the result of the `OUTPUT PATTERN`
  * If the result is empty (say the capture group was not in the input) then it will not print anything


Putting it all together we get...

So...

```bash
$ echo "time of completion: 14:24" | sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1/p"

14:24
```

We could modify the `OUTPUT PATTERN` to print the time twice, separated by a space...

```bash
$ echo "time of completion: 14:24" | sed -r -n "s/.*([0-9]{2}:[0-9]{2}).*/\1 \1/p"

14:24 14:24
```

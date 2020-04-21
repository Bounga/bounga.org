---
title: Ruby one-liners for file manipulations
category: Ruby
tags: ['tips', 'sysadmin', 'ruby']
header:
  overlay_image: /assets/images/headers/ruby.jpg
  overlay_filter: 0.2
excerpt: "Most people using Ruby nowdays are using it for Rails.
    They often defined themselves has 'Rails developers', not 'Ruby
    developers'. I'm an early adopter of Ruby and at the begining it
    was a scripting language intended to manipulate files and so on.
    Let's try to remembrer good old times and see how we can use Ruby
    to write one-liners to manipulate files."
---

When I started using Ruby back in 2003 it was for scripting my Linux
box. I didn't even thought of using it for something else than
scripting and system programming.

At this time I evaluated Perl, Python and Ruby. After a while I chose
Ruby as I thought it was the cleanest and more enjoyable language out
of the three to write scripts. Did I thought that I would use it for web
development for the next fifteen years? Not even a second. I chose it
to ease my daily job on my Linux system.

Today I'd like to share some tips I learned by using Ruby everyday for
scripting. It's going to be a bunch of one-liners that can be useful
to manipulate files and get info out of it.

Sure you can use Perl, Awk / Sed, Python to do the same thing, but I
like to do it using Ruby.

Ready? Fasten your belt, we're going to take off.

In the following example I'm going to use `/usr/share/dict/words` as
an input example since most of us have it but you can use the same
one-liners on any file.

## Working with newlines in files ##

In this section, we're going to see how you can handle spaces in your
text files.

### Double newlines in a file ###

The idea is to add an empty line between each line so if your file is
composed of one word per line, it will end up with a word, an empty
line, a word and so on:

```sh
$ cat /usr/share/dict/words | ruby -pe 'puts'
```

The `p` switch tells Ruby to iterate through all lines and the `e`
switch tells what to do on each one.

So here we print the line then add an empty one.

### Triple newlines in a file ###

Now let's say we want to add two empty lines between each word:

```sh
$ cat /usr/share/dict/words | ruby -pe '2.times { puts }'
```

### Remove double newlines in a file ###

If you have a file with two newlines after each line, you'll maybe
want to strip the extra newline. There's more than one way to do it:

```sh
$ cat double_newlines.txt | ruby -lne 'BEGIN{$/="\n\n"}; puts $_'
```

The `l` switch will use the value of `$/` to `chop!` it (understand
remove it) on each line for us.

If we don't provide the `l` flag, we have to remove the double
newlines by ourselves:

```sh
$ cat double.txt | ruby -ne 'BEGIN{$/="\n\n"}; puts $_.chop!'
```

## Add a blank line every five lines ##

Now we're going to add an empty line every five lines:

```sh
$ cat /usr/share/dict/words | /usr/bin/ruby -pe 'puts if $. % 6 == 0'
```

Here `$.` is the number of the current line we're are processing. So
if the line number modulo six is equal to zero then we add a new blank
line.

## Numbering and counting lines ##

A thing you'll often want to do when writing scripts dealing with text
files is to print the lines numbers or count the number of lines in
the file. Here is how to do it using Ruby.

### Number each line (left justified) ###

```sh
$ cat /usr/shar/dict/words | ruby -ne 'printf("%-6s %s", $., $_)'
```

This will add the line number on the left side, a space then the word.
The line number will be left justified with a six characters pad.

### Number each line (right justified) ###

```sh
$ cat /usr/shar/dict/words | ruby -ne 'printf("%6s %s", $., $_)'
```

This will add the line number on the left side, a space then the word.
The line number will be right justified with a six characters pad.

### Count lines ###

Not that this is not very effective, there are other (non one-liners
to be used in the term) that will be much more faster.

```sh
$ cat /usr/share/dict/words | ruby -ne 'END { puts $. }' 
```

For this file that is near 250k lines it takes 140ms which is still
pretty fast to me.


## Converting newlines format (DOS / Unix) ##

This is something I used to do a lot when I still handled files coming
from Microsoft world. I had to change the text file so that the
Microsoft newline format (`\r\n`) was converted to Unix newline format
(`\n`):

### Convert DOS newlines (CR/LF) to Unix format (LF) ###

```sh
$ cat /usr/share/dict/words | ruby -lne 'BEGIN{$\="\n"}; print $_'
```

### Convert Unix newlines (LF) to DOS format (CR/LF) ###

```sh
$ cat /usr/share/dict/words | ruby -lne 'BEGIN{$\="\r\n"}; print $_'
```

## Deleting white spaces ##

Now we're going to deal with deleting unwanted spaces.

### Leading white spaces (space, tab, ...) ###

You'll sometimes have text files with lines beginning with spaces that
you want to remove. Here how to do it:

```sh
$ cat leadings_whitespaces.txt | ruby -pe 'gsub(/^\s+/, "")'
```

We're substituting everything that is understand as a white space by
nothing so there are gone.

### Trailing white spaces ###

You should also want to be able to do the same substitution for
trailing white spaces:

```sh
$ cat trailing_whitespaces.txt | ruby -pe 'gsub(/\s+$/, $/)'
```

### Leading and trailing white spaces ###

At some point you'll maybe want to remove leading **and** trailing
whither spaces from each line:

```sh
$ cat leading_and_trailing_whitespaces.txt | ruby -pe 'gsub(/^\s+/, "").gsub(/\s+$/, $/)'
```

## Handling indentation ##

If you're dealing with a lot of text files you'll probably want to fix
some indentation issues. Here are some tips.

### Insert 4 spaces at the beginning of each line ###

Don't know why you would want to do this but still 😆

```sh
$ cat /usr/share/dict/words | ruby -pe 'gsub($_, "    #{$_}")'
```

### Align all text flush right on a 79 columns width ###

This one is more useful:

```sh
$ cat /usr/share/dict/words | ruby -ne 'printf("%79s", $_)'
```

### Center all text in middle of 79 columns width ###

```sh
$ cat /usr/share/dict/words | ruby -lne 'puts $_.center(79)'
```

And now every line of text is centered!

## Substitution ##

A common need when it comes to text handling is to change something bu
something else, let's see how to do it in one line.

### Find and replace ###

Let's say we want to change "foo" by "bar" in a text file:

```sh
$ cat foo_file.txt | ruby -pe 'tr("foo", "bar")'
```

### Replace only for some lines ###

Maybe you don't want to replace every occurrences but only the ones
that are on a line that includes "baz":

```sh
$ cat file | ruby -pe 'tr("foo", "bar") if $_ =~ /baz/'
```

### Replace except for some lines ###

Maybe now you want to replace every occurrences for lines that doesn't
include "baz":

```sh
$ cat file | ruby -pe 'tr("foo", "bar") unless $_ =~ /baz/'
```

### Replace some words by another one ###

Let's say you're a demanding one and you want to be able to change
"foo", "bar" or "baz" by "Bounga":

```sh
$ cat file | ruby -pe 'gsub(/(foo|bar|baz)/, "Bounga")'
```

## Reverse things ##

Sometimes you'll have to reverse input, here are some examples.

### Reverse order of lines ###

This is a classic one, for some reason you would want to read the file
in reverse order:

```sh
$ cat /usr/share/dict/words | ruby -ne 'BEGIN{@arr=Array.new}; @arr.push($_); END{puts @arr.reverse}'
```

### Reverse character ###

Maybe you'll want to reverse character of words in every lines:

```sh
$ cat /usr/share/dict/words | ruby -lne 'puts $_.reverse'
```


## Joining ##

### Pairs lines side by side ###

Let's say you have a file full of words just like
`/usr/share/dict/words` and you want to pair words by 2. Here is a way
to do it with a Ruby one-liner:

```sh
$ cat /usr/share/dict/words | ruby -pe '$_ = $_.chomp + " " + gets if $. % 2'
```

### Interpret backslash as an append operator ###

If you're used to shell, you maybe know that you can split your command
on multiple lines by using a backslash. Here's how to interpret such
splitting using Ruby:

```sh
$ cat file | ruby -pe 'while $_.match(/\\$/); $_ = $_.chomp.chop + gets; end'
```

### Appending to previous line ###

Now you want to level up your game by allowing your user to use an
equal sign in the beginning of a line to happen the statement to the
previous line:

```sh
$ cat file | ruby -e 'puts STDIN.readlines.to_s.gsub(/\n=/, "")'
```

## Selective printing ##

### Emulate head behavior ###

Let's print the first line of a file:

```sh
$ cat file | ruby -pe 'puts $_; exit'
```

Now we'll print the first ten lines:

```sh
$ cat file | ruby -pe 'exit if $. > 10'
```

### Emulate tail behavior ###

Now we're gonna print the last line of a file:

```sh
$ cat file | ruby -ne 'line = $_; END {puts line}'
```

Now we'll print the first ten lines:

```sh
$ cat file | ruby -e 'puts STDIN.readlines.reverse!.slice(0,10).reverse!'
```

Once again this one isn't very effective. It's ok for small files
(hundred thousand of lines) but we're parsing the whole file only to
display latest lines.

## Match regexp ##

### Print lines that match a regexp only ###

```sh
$ cat file | ruby -pe 'next unless $_ =~ /regexp/'
```

### Print lines that **do not** match a regexp ###

```sh
$ cat file | ruby -pe 'next if $_ =~ /regexp/'
```

### Print the line immediately before a regexp ###

```sh
$ cat file | ruby -ne 'puts @prev if $_ =~ /regex/; @prev = $_;'
```

### Print the line immediately after a regexp ###

```sh
$ cat file | ruby -ne 'puts $_ if @prev =~ /regex/; @prev = $_;'
```

## Emulating grep ##

### Grep lines with matching terms in any order ###

Here's how to print lines that match `foo`, `bar` and `baz`:

```sh 
$ cat file | ruby -pe 'next unless $_ =~ /foo/ && $_ =~ /bar/ && $_ =~ /baz/'
```

### Grep lines with matching terms in order ###

Now let's do the same but respecting the order:

```sh
$ cat file | ruby -pe 'next unless $_ =~ /foo.*bar.*baz/'
```

### Grep lines with any term matching ###

Now we want print each line matching any of the terms specified/

```sh
$ cat file | ruby -pe 'next unless $_ =~ /(foo|bar|baz)/'
```

## Printing paragraphs ##

### Print paragraph if it contains regexp ###

```sh
$ cat file | ruby -ne 'BEGIN{$/="\n\n"}; print $_ if $_ =~ /regexp/'
```

### Print paragraph if it contains `foo` **and** `bar` **and** `baz` in any order ###

```sh
$ cat file | ruby -ne 'BEGIN{$/="\n\n"}; print $_ if $_ =~ /foo/ && $_ =~ /bar/ && $_ =~ /baz/'
```

### Print paragraph if it contains `foo` **and** `bar` **and** `baz` in order ###

```sh
$ cat file | ruby -ne 'BEGIN{$/="\n\n"}; print $_ if $_ =~ /(foo.*bar.*baz)/'
```

### Print paragraph if it contains `foo` **or** `bar` **or** `baz` ###

```sh
$ cat file | ruby -ne 'BEGIN{$/="\n\n"}; print $_ if $_ =~ /(foo|bar|baz)/'
```

## Print based on line length ##

### Print only lines of 65 characters or greater ###

```sh
$ cat file | ruby -lpe 'next unless $_.length >= 65'
```

### Print only lines of 65 characters or less ###

```sh
$ cat file | ruby -lpe 'next unless $_.length < 65'
```

## Print based on line numbers ##

### Print section of file based on line numbers (eg. lines 2-7 inclusive) ###

```sh
$ cat file | ruby -pe 'next unless $. >= 2 && $. <= 7'
```

### Print line number 52 ###

```sh
$ cat file | ruby -pe 'next unless $. == 52'
```

### Print every 3rd line starting at line 4 ###

```sh
$ cat file | ruby -pe 'next unless $. >= 4 && $. % 3 == 0'
```

## Print line based on regexp ##

### Print section of file from regex to end of file ###

```sh
$ cat file | ruby -pe '@found=true if $_ =~ /regex/; next unless @found'
```

### Print section of file between two regular expressions, /foo/ and /bar/ ###

```sh
$ cat file | ruby -ne '@found=true if $_ =~ /foo/; next unless @found; puts $_; exit if $_ =~ /bar/'
```

### Print all the file except between two regular expressions, /foo/ and /bar/ ###

```sh
$ cat file | ruby -ne '@found = true if $_ =~ /foo/; puts $_ unless @found; @found = false if $_ =~ /bar/'
```

## Removing duplicates ##

### Print file and remove duplicate, consecutive lines from a file ###

```sh
$ cat file | ruby -ne 'puts $_ unless $_ == @prev; @prev = $_'
```

### Print file and remove duplicate, non-consecutive lines from a file ###

```sh
$ cat file | ruby -e 'puts STDIN.readlines.sort.uniq!.to_s'
```

### Delete all consecutive blank lines from a file except the first ###

```sh
$ cat file | ruby -e 'BEGIN{$/=nil}; puts STDIN.readlines.to_s.gsub(/\n(\n)+/, "\n\n")'
```

### Delete all leading blank lines at top of file ###

```sh
$ cat file | ruby -pe '@lineFound = true if $_ !~ /^\s*$/; next if !@lineFound'
```

## Selective deleting ##

### Print file except for first 10 lines ###

```sh
$ cat file | ruby -pe 'next if $. <= 10'
```

### Print file except for last 10 lines ###

```sh
$ cat file | ruby -e 'lines=STDIN.readlines; puts lines [0,lines.size-10]'
```

### Print file except for every 8th line ###

```sh
$ cat file | ruby -pe 'next if $. % 8 == 0'
```

### Print file except for blank lines ###

```sh
$ cat file | ruby -pe 'next if $_ =~ /^\s*$/'
```

## Conclusion ##

Who said that Perl was the only language to deal with file content manipulation?!

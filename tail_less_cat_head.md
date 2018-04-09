# <a name="cat-less-tail-and-head"></a>Cat, Less, Tail and Head

**Table of Contents**

* [cat](#cat)
    * [Concatenate files](#concatenate-files)
    * [Accepting input from stdin](#accepting-input-from-stdin)
    * [Squeeze consecutive empty lines](#squeeze-consecutive-empty-lines)
    * [Prefix line numbers](#prefix-line-numbers)
    * [Viewing special characters](#viewing-special-characters)
    * [Writing text to file](#writing-text-to-file)
    * [tac](#tac)
    * [Useless use of cat](#useless-use-of-cat)
    * [Further Reading for cat](#further-reading-for-cat)
* [less](#less)
    * [Navigation commands](#navigation-commands)
    * [Further Reading for less](#further-reading-for-less)
* [tail](#tail)
    * [linewise tail](#linewise-tail)
    * [characterwise tail](#characterwise-tail)
    * [multiple file input for tail](#multiple-file-input-for-tail)
    * [Further Reading for tail](#further-reading-for-tail)
* [head](#head)
    * [linewise head](#linewise-head)
    * [characterwise head](#characterwise-head)
    * [multiple file input for head](#multiple-file-input-for-head)
    * [combining head and tail](#combining-head-and-tail)
    * [Further Reading for head](#further-reading-for-head)
* [Text Editors](#text-editors)

<br>

## <a name="cat"></a>cat

```bash
$ cat --version | head -n1
cat (GNU coreutils) 8.25

$ man cat
CAT(1)                           User Commands                          CAT(1)

NAME
       cat - concatenate files and print on the standard output

SYNOPSIS
       cat [OPTION]... [FILE]...

DESCRIPTION
       Concatenate FILE(s) to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

* For below examples, `marks_201*` files contain 3 fields delimited by TAB
* To avoid formatting issues, TAB has been converted to spaces using `col -x` while pasting the output here

<br>

#### <a name="concatenate-files"></a>Concatenate files

* One or more files can be given as input and hence a lot of times, `cat` is used to quickly see contents of small single file on terminal
* To save the output of concatenation, just redirect stdout

```bash
$ ls
marks_2015.txt  marks_2016.txt  marks_2017.txt

$ cat marks_201*
Name    Maths   Science
foo     67      78
bar     87      85
Name    Maths   Science
foo     70      75
bar     85      88
Name    Maths   Science
foo     68      76
bar     90      90

$ # save stdout to a file
$ cat marks_201* > all_marks.txt
```

<br>

#### <a name="accepting-input-from-stdin"></a>Accepting input from stdin

```bash
$ # combining input from stdin and other files
$ printf 'Name\tMaths\tScience \nbaz\t56\t63\nbak\t71\t65\n' | cat - marks_2015.txt
Name    Maths   Science
baz     56      63
bak     71      65
Name    Maths   Science
foo     67      78
bar     87      85

$ # - can be placed in whatever order is required
$ printf 'Name\tMaths\tScience \nbaz\t56\t63\nbak\t71\t65\n' | cat marks_2015.txt -
Name    Maths   Science
foo     67      78
bar     87      85
Name    Maths   Science
baz     56      63
bak     71      65
```

<br>

#### <a name="squeeze-consecutive-empty-lines"></a>Squeeze consecutive empty lines

```bash
$ printf 'hello\n\n\nworld\n\nhave a nice day\n'
hello


world

have a nice day
$ printf 'hello\n\n\nworld\n\nhave a nice day\n' | cat -s
hello

world

have a nice day
```

<br>

#### <a name="prefix-line-numbers"></a>Prefix line numbers

```bash
$ # number all lines
$ cat -n marks_201*
     1  Name    Maths   Science
     2  foo     67      78
     3  bar     87      85
     4  Name    Maths   Science
     5  foo     70      75
     6  bar     85      88
     7  Name    Maths   Science
     8  foo     68      76
     9  bar     90      90

$ # number only non-empty lines
$ printf 'hello\n\n\nworld\n\nhave a nice day\n' | cat -sb
     1  hello

     2  world

     3  have a nice day
```

* For more numbering options, check out the command `nl`

```bash
$ whatis nl
nl (1)               - number lines of files
```

<br>

#### <a name="viewing-special-characters"></a>Viewing special characters

* End of line identified by `$`
* Useful for example to see trailing spaces

```bash
$ cat -E marks_2015.txt
Name    Maths   Science $
foo     67      78$
bar     87      85$
```

* TAB identified by `^I`

```bash
$ cat -T marks_2015.txt
Name^IMaths^IScience 
foo^I67^I78
bar^I87^I85
```

* Non-printing characters
* See [Show Non-Printing Characters](http://docstore.mik.ua/orelly/unix/upt/ch25_07.htm) for more detailed info

```bash
$ # NUL character
$ printf 'foo\0bar\0baz\n' | cat -v
foo^@bar^@baz

$ # to check for dos-style line endings
$ printf 'Hello World!\r\n' | cat -v
Hello World!^M

$ printf 'Hello World!\r\n' | dos2unix | cat -v
Hello World!
```

* the `-A` option is equivalent to `-vET`
* the `-e` option is equivalent to `-vE`
* If `dos2unix` and `unix2dos` are not available, see [How to convert DOS/Windows newline (CRLF) to Unix newline (\n)](https://stackoverflow.com/questions/2613800/how-to-convert-dos-windows-newline-crlf-to-unix-newline-n-in-a-bash-script)

<br>

#### <a name="writing-text-to-file"></a>Writing text to file

```bash
$ cat > sample.txt
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.

$ cat sample.txt
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.
```

* See also how to use [heredoc](http://mywiki.wooledge.org/HereDocument)
    * [How can I write a here doc to a file](https://stackoverflow.com/questions/2953081/how-can-i-write-a-here-doc-to-a-file-in-bash-script)
* See also [difference between Ctrl+c and Ctrl+d to signal end of stdin input in bash](https://unix.stackexchange.com/questions/16333/how-to-signal-the-end-of-stdin-input-in-bash)

<br>

#### <a name="tac"></a>tac

```bash
$ whatis tac
tac (1)              - concatenate and print files in reverse
$ tac --version | head -n1
tac (GNU coreutils) 8.25

$ seq 3 | tac
3
2
1

$ tac marks_2015.txt
bar     87      85
foo     67      78
Name    Maths   Science
```

* Useful in cases where logic is easier to write when working on reversed file
* Consider this made up log file, many **Warning** lines but need to extract only from last such **Warning** upto **Error** line
    * See [GNU sed chapter](./gnu_sed.md#lines-between-two-regexps) for details on the `sed` command used below

```bash
$ cat report.log
blah blah
Warning: something went wrong
more blah
whatever
Warning: something else went wrong
some text
some more text
Error: something seriously went wrong
blah blah blah

$ tac report.log | sed -n '/Error:/,/Warning:/p' | tac
Warning: something else went wrong
some text
some more text
Error: something seriously went wrong
```

* Similarly, if characters in lines have to be reversed, use the `rev` command

```bash
$ whatis rev
rev (1)              - reverse lines characterwise
```

<br>

#### <a name="useless-use-of-cat"></a>Useless use of cat

* `cat` is used so frequently to view contents of a file that somehow users think other commands cannot handle file input
* [UUOC](https://en.wikipedia.org/wiki/Cat_(Unix)#Useless_use_of_cat)
* [Useless Use of Cat Award](http://porkmail.org/era/unix/award.html)

```bash
$ cat report.log | grep -E 'Warning|Error'
Warning: something went wrong
Warning: something else went wrong
Error: something seriously went wrong
$ grep -E 'Warning|Error' report.log
Warning: something went wrong
Warning: something else went wrong
Error: something seriously went wrong
```

* Use [input redirection](http://wiki.bash-hackers.org/howto/redirection_tutorial) if a command doesn't accept file input

```bash
$ cat marks_2015.txt | tr 'A-Z' 'a-z'
name    maths   science
foo     67      78
bar     87      85
$ tr 'A-Z' 'a-z' < marks_2015.txt
name    maths   science
foo     67      78
bar     87      85
```

* However, `cat` should definitely be used where **concatenation** is needed

```bash
$ grep -c 'foo' marks_201*
marks_2015.txt:1
marks_2016.txt:1
marks_2017.txt:1

$ # concatenation allows to get overall count in one-shot in this case
$ cat marks_201* | grep -c 'foo'
3
```

<br>

#### <a name="further-reading-for-cat"></a>Further Reading for cat

* [cat Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/cat?sort=votes&pageSize=15)
* [cat Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/cat?sort=votes&pageSize=15)

<br>

## <a name="less"></a>less

```bash
$ less --version | head -n1
less 481 (GNU regular expressions)

$ # By default, pager is used to display the man pages
$ # and usually, pager is linked to less command
$ type pager less
pager is /usr/bin/pager
less is /usr/bin/less

$ realpath /usr/bin/pager
/bin/less
$ realpath /usr/bin/less
/bin/less
$ diff -s /usr/bin/pager /usr/bin/less
Files /usr/bin/pager and /usr/bin/less are identical
```

* `cat` command is NOT suitable for viewing contents of large files on the Terminal
* `less` displays contents of a file, automatically fits to size of Terminal, allows scrolling in either direction and other options for effective viewing
* Usually, `man` command uses `less` command to display the help page
* The navigation commands are similar to `vi` editor

<br>

#### <a name="navigation-commands"></a>Navigation commands

Commonly used commands are given below, press `h` for summary of options

* `g` go to start of file
* `G` go to end of file
* `q` quit
* `/pattern` search for the given pattern in forward direction
* `?pattern` search for the given pattern in backward direction
* `n` go to next pattern
* `N` go to previous pattern

<br>

#### <a name="further-reading-for-less"></a>Further Reading for less

* See `man less` for detailed info on commands and options. For example:
    * `-s` option to squeeze consecutive blank lines
    * `-N` option to prefix line number
* `less` command is an [improved version](https://unix.stackexchange.com/questions/604/isnt-less-just-more) of `more` command
* [differences between most, more and less](https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less)
* [less Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/less?sort=votes&pageSize=15)

<br>

## <a name="tail"></a>tail

```bash
$ tail --version | head -n1
tail (GNU coreutils) 8.25

$ man tail
TAIL(1)                          User Commands                         TAIL(1)

NAME
       tail - output the last part of files

SYNOPSIS
       tail [OPTION]... [FILE]...

DESCRIPTION
       Print  the  last  10  lines of each FILE to standard output.  With more
       than one FILE, precede each with a header giving the file name.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="linewise-tail"></a>linewise tail

Consider this sample file, with line numbers prefixed

```bash
$ cat sample.txt
 1) Hello World
 2) 
 3) Good day
 4) How are you
 5) 
 6) Just do-it
 7) Believe it
 8) 
 9) Today is sunny
10) Not a bit funny
11) No doubt you like it too
12) 
13) Much ado about nothing
14) He he he
15) Adios amigo
```

* default behavior - display last 10 lines

```bash
$ tail sample.txt
 6) Just do-it
 7) Believe it
 8) 
 9) Today is sunny
10) Not a bit funny
11) No doubt you like it too
12) 
13) Much ado about nothing
14) He he he
15) Adios amigo
```

* Use `-n` option to control number of lines to filter

```bash
$ tail -n3 sample.txt
13) Much ado about nothing
14) He he he
15) Adios amigo

$ # some versions of tail allow to skip explicit n character
$ tail -5 sample.txt
11) No doubt you like it too
12) 
13) Much ado about nothing
14) He he he
15) Adios amigo
```

* when number is prefixed with `+` sign, all lines are fetched from that particular line number to end of file

```bash
$ tail -n +10 sample.txt
10) Not a bit funny
11) No doubt you like it too
12) 
13) Much ado about nothing
14) He he he
15) Adios amigo

$ seq 13 17 | tail -n +3
15
16
17
```

<br>

#### <a name="characterwise-tail"></a>characterwise tail

* Note that this works byte wise and not suitable for multi-byte character encodings

```bash
$ # last three characters including the newline character
$ echo 'Hi there!' | tail -c3
e!

$ # excluding the first character
$ echo 'Hi there!' | tail -c +2
i there!
```

<br>

#### <a name="multiple-file-input-for-tail"></a>multiple file input for tail

```bash
$ tail -n2 report.log sample.txt
==> report.log <==
Error: something seriously went wrong
blah blah blah

==> sample.txt <==
14) He he he
15) Adios amigo

$ # -q option to avoid filename in output
$ tail -q -n2 report.log sample.txt
Error: something seriously went wrong
blah blah blah
14) He he he
15) Adios amigo
```

<br>

#### <a name="further-reading-for-tail"></a>Further Reading for tail

* `tail -f` and related options are beyond the scope of this tutorial. Below links might be useful
    * [look out for buffering](http://mywiki.wooledge.org/BashFAQ/009)
    * [Piping tail -f output though grep twice](https://stackoverflow.com/questions/13858912/piping-tail-output-though-grep-twice)
    * [tail and less](https://unix.stackexchange.com/questions/196168/does-less-have-a-feature-like-tail-follow-name-f)
* [tail Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/tail?sort=votes&pageSize=15)
* [tail Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/tail?sort=votes&pageSize=15)

<br>

## <a name="head"></a>head

```bash
$ head --version | head -n1
head (GNU coreutils) 8.25

$ man head
HEAD(1)                          User Commands                         HEAD(1)

NAME
       head - output the first part of files

SYNOPSIS
       head [OPTION]... [FILE]...

DESCRIPTION
       Print  the  first  10 lines of each FILE to standard output.  With more
       than one FILE, precede each with a header giving the file name.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="linewise-head"></a>linewise head

* default behavior - display starting 10 lines

```bash
$ head sample.txt
 1) Hello World
 2) 
 3) Good day
 4) How are you
 5) 
 6) Just do-it
 7) Believe it
 8) 
 9) Today is sunny
10) Not a bit funny
```

* Use `-n` option to control number of lines to filter

```bash
$ head -n3 sample.txt
 1) Hello World
 2) 
 3) Good day

$ # some versions of head allow to skip explicit n character
$ head -4 sample.txt
 1) Hello World
 2) 
 3) Good day
 4) How are you
```

* when number is prefixed with `-` sign, all lines are fetched except those many lines to end of file

```bash
$ # except last 9 lines of file
$ head -n -9 sample.txt
 1) Hello World
 2) 
 3) Good day
 4) How are you
 5) 
 6) Just do-it

$ # except last 2 lines
$ seq 13 17 | head -n -2
13
14
15
```

<br>

#### <a name="characterwise-head"></a>characterwise head

* Note that this works byte wise and not suitable for multi-byte character encodings

```bash
$ # if output of command doesn't end with newline, prompt will be on same line
$ # to highlight working of command, the prompt for such cases is not shown here

$ # first two characters
$ echo 'Hi there!' | head -c2
Hi

$ # excluding last four characters
$ echo 'Hi there!' | head -c -4
Hi the
```

<br>

#### <a name="multiple-file-input-for-head"></a>multiple file input for head

```bash
$ head -n3 report.log sample.txt
==> report.log <==
blah blah
Warning: something went wrong
more blah

==> sample.txt <==
 1) Hello World
 2) 
 3) Good day

$ # -q option to avoid filename in output
$ head -q -n3 report.log sample.txt
blah blah
Warning: something went wrong
more blah
 1) Hello World
 2) 
 3) Good day
```

<br>

#### <a name="combining-head-and-tail"></a>combining head and tail

* Despite involving two commands, often this combination is faster than equivalent sed/awk versions

```bash
$ head -n11 sample.txt | tail -n3
 9) Today is sunny
10) Not a bit funny
11) No doubt you like it too

$ tail sample.txt | head -n2
 6) Just do-it
 7) Believe it
```

<br>

#### <a name="further-reading-for-head"></a>Further Reading for head

* [head Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/head?sort=votes&pageSize=15)

<br>

## <a name="text-editors"></a>Text Editors

For editing text files, the following applications can be used. Of these, `gedit`, `nano`, `vi` and/or `vim` are available in most distros by default

Easy to use

* [gedit](https://wiki.gnome.org/Apps/Gedit)
* [geany](http://www.geany.org/)
* [nano](http://nano-editor.org/)

Powerful text editors

* [vim](https://github.com/vim/vim)
    * [vim learning resources](https://github.com/learnbyexample/scripting_course/blob/master/Vim_curated_resources.md) and [vim reference](https://github.com/learnbyexample/vim_reference) for further info
* [emacs](https://www.gnu.org/software/emacs/)
* [atom](https://atom.io/)
* [sublime](https://www.sublimetext.com/)

Check out [this analysis](https://github.com/jhallen/joes-sandbox/tree/master/editor-perf) for some performance/feature comparisons of various text editors

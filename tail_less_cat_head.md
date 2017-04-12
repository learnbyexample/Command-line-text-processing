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
* [tail](#tail)
* [head](#head)
* [Text Editors](#text-editors)

<br>

## <a name="cat"></a>cat

```bash
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
     1	hello

     2	world

     3	have a nice day
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

$ seq 3 | tac
3
2
1

$ tac marks_2015.txt 
bar	87	85
foo	67	78
Name	Maths	Science 
```

* Useful in cases where logic is easier to write when working on reversed file
* Consider this made up log file, many **Warning** lines but need to extract only from last such **Warning** upto **Error** line

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

>opposite of more

`cat` command is not suitable for viewing contents of large files on the Terminal. `less` displays contents of a file, automatically fits to size of Terminal, allows scrolling in either direction and other options for effective viewing. Usually, `man` command uses `less` command to display the help page. The navigation options are similar to `vi` editor

**Navigation options**

* `g` go to start of file
* `G` go to end of file
* `q` quit
* `/pattern` search for the given pattern in forward direction
* `?pattern` search for the given pattern in backward direction
* `n` go to next pattern
* `N` go to previous pattern
* `h` help

**Example and Further Reading**

* `less -s large_filename` display contents of file large_filename using less command, consecutive blank lines are squeezed as single blank line
* `less` command is an [improved version](https://unix.stackexchange.com/questions/604/isnt-less-just-more) of `more` command
* [differences between most, more and less](https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less)
* [less Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/less?sort=votes&pageSize=15)

<br>

## <a name="tail"></a>tail

>output the last part of files

**Examples**

* `tail report.log` by default display last 10 lines
* `tail -20 report.log` display last 20 lines
* `tail -5 report.log` display last 5 lines
* `tail power.log timing.log` display last 10 lines of both files preceded by filename header
* `tail -q power.log timing.log > result.log` save last 10 lines of both files without filename header to result.log
* `tail -n +3 report.log` display all lines starting from third line (i.e all lines except first two lines)
* [tail Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/tail?sort=votes&pageSize=15)
* [tail Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/tail?sort=votes&pageSize=15)

<br>

## <a name="head"></a>head

>output the first part of files

**Examples**

* `head report.log` display first 10 lines
* `head -20 report.log | tail report.log` display lines 11 to 20
* `tail -30 report.log | head -5 report.log` display 5 lines prior to last 25 lines of file
* `head -n -2 report.log` display all but last 2 lines
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


# <a name="miscellaneous"></a>Miscellaneous

**Table of Contents**

* [cut](#cut)
    * [select specific fields](#select-specific-fields)
    * [suppressing lines without delimiter](#suppressing-lines-without-delimiter)
    * [specifying delimiters](#specifying-delimiters)
    * [complement](#complement)
    * [select specific characters](#select-specific-characters)
    * [Further reading for cut](#further-reading-for-cut)
* [tr](#tr)
* [basename](#basename)
* [dirname](#dirname)

<br>

## <a name="cut"></a>cut

```bash
$ cut --version | head -n1
cut (GNU coreutils) 8.25

$ man cut
CUT(1)                           User Commands                          CUT(1)

NAME
       cut - remove sections from each line of files

SYNOPSIS
       cut OPTION... [FILE]...

DESCRIPTION
       Print selected parts of lines from each FILE to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="select-specific-fields"></a>select specific fields

* Default delimiter is **tab** character
* `-f` option allows to print specific field(s) from each input line

```bash
$ printf 'foo\tbar\t123\tbaz\n'
foo     bar     123     baz

$ # single field
$ printf 'foo\tbar\t123\tbaz\n' | cut -f2
bar

$ # multiple fields can be specified by using ,
$ printf 'foo\tbar\t123\tbaz\n' | cut -f2,4
bar     baz

$ # output is always ascending order of field numbers
$ printf 'foo\tbar\t123\tbaz\n' | cut -f3,1
foo     123

$ # range can be specified using -
$ printf 'foo\tbar\t123\tbaz\n' | cut -f1-3
foo     bar     123
$ # if ending number is omitted, select till last field
$ printf 'foo\tbar\t123\tbaz\n' | cut -f3-
123     baz
```

<br>

#### <a name="suppressing-lines-without-delimiter"></a>suppressing lines without delimiter

```bash
$ cat marks.txt
jan 2017
foobar  12      45      23
feb 2017
foobar  18      38      19

$ # by default lines without delimiter will be printed
$ cut -f2- marks.txt
jan 2017
12      45      23
feb 2017
18      38      19

$ # use -s option to suppress such lines
$ cut -s -f2- marks.txt
12      45      23
18      38      19
```

<br>

#### <a name="specifying-delimiters"></a>specifying delimiters

* use `-d` option to specify input delimiter other than default **tab** character
* only single character can be used, for multi-character/regex based delimiter use `awk` or `perl`

```bash
$ echo 'foo:bar:123:baz' | cut -d: -f3
123

$ # by default output delimiter is same as input
$ echo 'foo:bar:123:baz' | cut -d: -f1,4
foo:baz

$ # quote the delimiter character if it clashes with shell special characters
$ echo 'one;two;three;four' | cut -d; -f3
cut: option requires an argument -- 'd'
Try 'cut --help' for more information.
-f3: command not found
$ echo 'one;two;three;four' | cut -d';' -f3
three
```

* use `--output-delimiter` option to specify different output delimiter
* since this option accepts a string, more than one character can be specified
* See also [using $ prefixed string](https://unix.stackexchange.com/questions/48106/what-does-it-mean-to-have-a-dollarsign-prefixed-string-in-a-script)

```bash
$ printf 'foo\tbar\t123\tbaz\n' | cut --output-delimiter=: -f1-3
foo:bar:123

$ echo 'one;two;three;four' | cut -d';' --output-delimiter=' ' -f1,3-
one three four

$ # tested on bash, might differ with other shells
$ echo 'one;two;three;four' | cut -d';' --output-delimiter=$'\t' -f1,3-
one     three   four

$ echo 'one;two;three;four' | cut -d';' --output-delimiter=' - ' -f1,3-
one - three - four
```

<br>

#### <a name="complement"></a>complement

```bash
$ echo 'one;two;three;four' | cut -d';' -f1,3-
one;three;four

$ # to print other than specified fields
$ echo 'one;two;three;four' | cut -d';' --complement -f2
one;three;four
```

<br>

#### <a name="select-specific-characters"></a>select specific characters

* similar to `-f` for field selection, use `-c` for character selection
* See manual for what defines a character and differences between `-b` and `-c`

```bash
$ echo 'foo:bar:123:baz' | cut -c4
:

$ printf 'foo\tbar\t123\tbaz\n' | cut -c1,4,7
f       r

$ echo 'foo:bar:123:baz' | cut -c8-
:123:baz

$ echo 'foo:bar:123:baz' | cut --complement -c8-
foo:bar

$ echo 'foo:bar:123:baz' | cut -c1,6,7 --output-delimiter=' '
f a r

$ cut -c1-3 marks.txt
jan
foo
feb
foo
```

<br>

#### <a name="further-reading-for-cut"></a>Further reading for cut

* `man cut` and `info cut` for more options and detailed documentation
* [cut Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/cut?sort=votes&pageSize=15)

<br>

## <a name="tr"></a>tr

```bash
$ tr --version | head -n1
tr (GNU coreutils) 8.25

$ man tr
TR(1)                            User Commands                           TR(1)

NAME
       tr - translate or delete characters

SYNOPSIS
       tr [OPTION]... SET1 [SET2]

DESCRIPTION
       Translate, squeeze, and/or delete characters from standard input, writ‐
       ing to standard output.
...
```

**Options**

* `-d` delete the specified characters
* `-c` complement set of characters to be replaced

**Examples**

* `tr a-z A-Z < test_list.txt` convert lowercase to uppercase
* `tr -d ._ < test_list.txt` delete the dot and underscore characters
* `tr a-z n-za-m < test_list.txt > encrypted_test_list.txt` Encrypt by replacing every lowercase alphabet with 13th alphabet after it
    * Same command on encrypted text will decrypt it
* [tr Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/tr?sort=votes&pageSize=15)

<br>

## <a name="basename"></a>basename

```bash
$ basename --version | head -n1
basename (GNU coreutils) 8.25

$ man basename
BASENAME(1)                      User Commands                     BASENAME(1)

NAME
       basename - strip directory and suffix from filenames

SYNOPSIS
       basename NAME [SUFFIX]
       basename OPTION... NAME...

DESCRIPTION
       Print  NAME  with  any leading directory components removed.  If speci‐
       fied, also remove a trailing SUFFIX.
...
```

```bash
$ pwd
/home/learnbyexample

$ basename $PWD
learnbyexample

$ basename '/home/learnbyexample/proj_adder/power.log'
power.log

$ #use suffix option -s to strip file extension from filename
$ basename -s '.log' '/home/learnbyexample/proj_adder/power.log'
power
```

<br>

## <a name="dirname"></a>dirname

```bash
$ dirname --version | head -n1
dirname (GNU coreutils) 8.25

$ man dirname
DIRNAME(1)                       User Commands                      DIRNAME(1)

NAME
       dirname - strip last component from file name

SYNOPSIS
       dirname [OPTION] NAME...

DESCRIPTION
       Output each NAME with its last non-slash component and trailing slashes
       removed; if NAME contains no  /'s,  output  '.'  (meaning  the  current
       directory).
...
```

```bash
$ pwd
/home/learnbyexample

$ dirname $PWD
/home

$ dirname '/home/learnbyexample/proj_adder/power.log'
/home/learnbyexample/proj_adder
```


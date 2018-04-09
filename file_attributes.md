# <a name="file-attributes"></a>File attributes

**Table of Contents**

* [wc](#wc)
    * [Various counts](#various-counts)
    * [subtle differences](#subtle-differences)
    * [Further reading for wc](#further-reading-for-wc)
* [du](#du)
    * [Default size](#default-size)
    * [Various size formats](#various-size-formats)
    * [Dereferencing links](#dereferencing-links)
    * [Filtering options](#filtering-options)
    * [Further reading for du](#further-reading-for-du)
* [df](#df)
    * [Examples](#examples)
    * [Further reading for df](#further-reading-for-df)
* [touch](#touch)
    * [Creating empty file](#creating-empty-file)
    * [Updating timestamps](#updating-timestamps)
    * [Preserving timestamp](#preserving-timestamp)
    * [Further reading for touch](#further-reading-for-touch)
* [file](#file)
    * [File type examples](#file-type-examples)
    * [Further reading for file](#further-reading-for-file)

<br>

## <a name="wc"></a>wc

```bash
$ wc --version | head -n1
wc (GNU coreutils) 8.25

$ man wc
WC(1)                            User Commands                           WC(1)

NAME
       wc - print newline, word, and byte counts for each file

SYNOPSIS
       wc [OPTION]... [FILE]...
       wc [OPTION]... --files0-from=F

DESCRIPTION
       Print newline, word, and byte counts for each FILE, and a total line if
       more than one FILE is specified.  A word is a non-zero-length  sequence
       of characters delimited by white space.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="various-counts"></a>Various counts

```bash
$ cat sample.txt
Hello World
Good day
No doubt you like it too
Much ado about nothing
He he he

$ # by default, gives newline/word/byte count (in that order)
$ wc sample.txt
 5 17 78 sample.txt

$ # options to get individual numbers
$ wc -l sample.txt
5 sample.txt
$ wc -w sample.txt
17 sample.txt
$ wc -c sample.txt
78 sample.txt

$ # use shell input redirection if filename is not needed
$ wc -l < sample.txt
5
```

* multiple file input
* automatically displays total at end

```bash
$ cat greeting.txt
Hello there
Have a safe journey
$ cat fruits.txt
Fruit   Price
apple   42
banana  31
fig     90
guava   6

$ wc *.txt
  5  10  57 fruits.txt
  2   6  32 greeting.txt
  5  17  78 sample.txt
 12  33 167 total
```

* use `-L` to get length of longest line

```bash
$ wc -L < sample.txt
24

$ echo 'foo bar baz' | wc -L
11
$ echo 'hi there!' | wc -L
9

$ # last line will show max value, not sum of all input
$ wc -L *.txt
 13 fruits.txt
 19 greeting.txt
 24 sample.txt
 24 total
```

<br>

#### <a name="subtle-differences"></a>subtle differences

* byte count vs character count

```bash
$ # when input is ASCII
$ printf 'hi there' | wc -c
8
$ printf 'hi there' | wc -m
8

$ # when input has multi-byte characters
$ printf 'hi👍' | od -x
0000000 6968 9ff0 8d91
0000006

$ printf 'hi👍' | wc -m
3

$ printf 'hi👍' | wc -c
6
```

* `-l` option gives only the count of number of newline characters

```bash
$ printf 'hi there\ngood day' | wc -l
1
$ printf 'hi there\ngood day\n' | wc -l
2
$ printf 'hi there\n\n\nfoo\n' | wc -l
4
```

* From `man wc` "A word is a non-zero-length sequence of characters delimited by white space"

```bash
$ echo 'foo        bar ;-*' | wc -w
3

$ # use other text processing as needed
$ echo 'foo        bar ;-*' | grep -iowE '[a-z]+'
foo
bar
$ echo 'foo        bar ;-*' | grep -iowE '[a-z]+' | wc -l
2
```

* `-L` won't count non-printable characters and tabs are converted to equivalent spaces

```bash
$ printf 'food\tgood' | wc -L
12
$ printf 'food\tgood' | wc -m
9
$ printf 'food\tgood' | awk '{print length()}'
9

$ printf 'foo\0bar\0baz' | wc -L
9
$ printf 'foo\0bar\0baz' | wc -m
11
$ printf 'foo\0bar\0baz' | awk '{print length()}'
11
```

<br>

#### <a name="further-reading-for-wc"></a>Further reading for wc

* `man wc` and `info wc` for more options and detailed documentation
* [wc Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/wc?sort=votes&pageSize=15)
* [wc Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/wc?sort=votes&pageSize=15)

<br>

## <a name="du"></a>du

```bash
$ du --version | head -n1
du (GNU coreutils) 8.25

$ man du
DU(1)                            User Commands                           DU(1)

NAME
       du - estimate file space usage

SYNOPSIS
       du [OPTION]... [FILE]...
       du [OPTION]... --files0-from=F

DESCRIPTION
       Summarize disk usage of the set of FILEs, recursively for directories.
...
```

<br>

<br>

#### <a name="default-size"></a>Default size

* By default, size is given in size of **1024 bytes**
* Files are ignored, all directories and sub-directories are recursively reported

```bash
$ ls -F
projs/  py_learn@  words.txt

$ du
17920   ./projs/full_addr
14316   ./projs/half_addr
32952   ./projs
33880   .
```

* use `-a` to recursively show both files and directories
* use `-s` to show total directory size without descending into its sub-directories

```bash
$ du -a
712     ./projs/report.log
17916   ./projs/full_addr/faddr.v
17920   ./projs/full_addr
14312   ./projs/half_addr/haddr.v
14316   ./projs/half_addr
32952   ./projs
0       ./py_learn
924     ./words.txt
33880   .

$ du -s
33880   .

$ du -s projs words.txt
32952   projs
924     words.txt
```

* use `-S` to show directory size without taking into account size of its sub-directories

```bash
$ du -S
17920   ./projs/full_addr
14316   ./projs/half_addr
716     ./projs
928     .
```

<br>

<br>

#### <a name="various-size-formats"></a>Various size formats

```bash
$ # number of bytes
$ stat -c %s words.txt
938848
$ du -b words.txt
938848  words.txt

$ # kilobytes = 1024 bytes
$ du -sk projs
32952   projs
$ # megabytes = 1024 kilobytes
$ du -sm projs
33      projs

$ # -B to specify custom byte scale size
$ du -sB 5000 projs
6749    projs
$ du -sB 1048576 projs
33      projs
```

* human readable and si units

```bash
$ # in terms of powers of 1024
$ # M = 1048576 bytes and so on
$ du -sh projs/* words.txt
18M     projs/full_addr
14M     projs/half_addr
712K    projs/report.log
924K    words.txt

$ # in terms of powers of 1000
$ # M = 1000000 bytes and so on
$ du -s --si projs/* words.txt
19M     projs/full_addr
15M     projs/half_addr
730k    projs/report.log
947k    words.txt
```

* sorting

```bash
$ du -sh projs/* words.txt | sort -h
712K    projs/report.log
924K    words.txt
14M     projs/half_addr
18M     projs/full_addr

$ du -sk projs/* | sort -nr
17920   projs/full_addr
14316   projs/half_addr
712     projs/report.log
```

* to get size based on number of characters in file rather than disk space alloted

```bash
$ du -b words.txt
938848  words.txt

$ du -h words.txt
924K    words.txt

$ # 938848/1024 = 916.84
$ du --apparent-size -h words.txt
917K    words.txt
```

<br>

#### <a name="dereferencing-links"></a>Dereferencing links

* See `man` and `info` pages for other related options

```bash
$ # -D to dereference command line argument
$ du py_learn
0       py_learn
$ du -shD py_learn
503M    py_learn

$ # -L to dereference links found by du
$ du -sh
34M     .
$ du -shL
536M    .
```

<br>

#### <a name="filtering-options"></a>Filtering options

* `-d` to specify maximum depth

```bash
$ du -ah projs
712K    projs/report.log
18M     projs/full_addr/faddr.v
18M     projs/full_addr
14M     projs/half_addr/haddr.v
14M     projs/half_addr
33M     projs

$ du -ah -d1 projs
712K    projs/report.log
18M     projs/full_addr
14M     projs/half_addr
33M     projs
```

* `-c` to also show total size at end

```bash
$ du -cshD projs py_learn
33M     projs
503M    py_learn
535M    total
```

* `-t` to provide a threshold comparison

```bash
$ # >= 15M
$ du -Sh -t 15M
18M     ./projs/full_addr

$ # <= 1M
$ du -ah -t -1M
712K    ./projs/report.log
0       ./py_learn
924K    ./words.txt
```

* excluding files/directories based on **glob** pattern
* see also `--exclude-from=FILE` and `--files0-from=FILE` options

```bash
$ # note that excluded files affect directory size reported
$ du -ah --exclude='*addr*' projs
712K    projs/report.log
716K    projs

$ # depending on shell, brace expansion can be used
$ du -ah --exclude='*.'{v,log} projs
4.0K    projs/full_addr
4.0K    projs/half_addr
12K     projs
```

<br>

#### <a name="further-reading-for-du"></a>Further reading for du

* `man du` and `info du` for more options and detailed documentation
* [du Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/disk-usage?sort=votes&pageSize=15)
* [du Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/du?sort=votes&pageSize=15)

<br>

## <a name="df"></a>df

```bash
$ df --version | head -n1
df (GNU coreutils) 8.25

$ man df
DF(1)                            User Commands                           DF(1)

NAME
       df - report file system disk space usage

SYNOPSIS
       df [OPTION]... [FILE]...

DESCRIPTION
       This  manual  page  documents  the  GNU version of df.  df displays the
       amount of disk space available on the file system containing each  file
       name  argument.   If  no file name is given, the space available on all
       currently mounted file systems is shown.
...
```

<br>

#### <a name="examples"></a>Examples

```bash
$ # use df without arguments to get information on all currently mounted file systems
$ df .
Filesystem     1K-blocks     Used Available Use% Mounted on
/dev/sda1       98298500 58563816  34734748  63% /

$ # use -B option for custom size
$ # use --si for size in powers of 1000 instead of 1024
$ df -h .
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        94G   56G   34G  63% /
```

* Use `--output` to report only specific fields of interest

```bash
$ df -h --output=size,used,file / /media/learnbyexample/projs
 Size  Used File
  94G   56G /
  92G   35G /media/learnbyexample/projs

$ df -h --output=pcent .
Use%
 63%

$ df -h --output=pcent,fstype | awk -F'%' 'NR>2 && $1>=40'
 63% ext3
 40% ext4
 51% ext4
```

<br>

#### <a name="further-reading-for-df"></a>Further reading for df

* `man df` and `info df` for more options and detailed documentation
* [df Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/df?sort=votes&pageSize=15)
* [Parsing df command output with awk](https://unix.stackexchange.com/questions/360865/parsing-df-command-output-with-awk)
* [processing df output](https://www.reddit.com/r/bash/comments/68dbml/using_an_array_variable_in_an_awk_command/)

<br>

## <a name="touch"></a>touch

```bash
$ touch --version | head -n1
touch (GNU coreutils) 8.25

$ man touch
TOUCH(1)                         User Commands                        TOUCH(1)

NAME
       touch - change file timestamps

SYNOPSIS
       touch [OPTION]... FILE...

DESCRIPTION
       Update  the  access  and modification times of each FILE to the current
       time.

       A FILE argument that does not exist is created empty, unless -c  or  -h
       is supplied.
...
```

<br>

#### <a name="creating-empty-file"></a>Creating empty file

```bash
$ ls foo.txt
ls: cannot access 'foo.txt': No such file or directory
$ touch foo.txt
$ ls foo.txt
foo.txt

$ # use -c if new file shouldn't be created
$ rm foo.txt
$ touch -c foo.txt
$ ls foo.txt
ls: cannot access 'foo.txt': No such file or directory
```

<br>

#### <a name="updating-timestamps"></a>Updating timestamps

* Updating both access and modification timestamp to current time

```bash
$ # last access time
$ stat -c %x fruits.txt
2017-07-19 17:06:01.523308599 +0530
$ # last modification time
$ stat -c %y fruits.txt
2017-07-13 13:54:03.576055933 +0530

$ touch fruits.txt
$ stat -c %x fruits.txt
2017-07-21 10:11:44.241921229 +0530
$ stat -c %y fruits.txt
2017-07-21 10:11:44.241921229 +0530
```

* Updating only access or modification timestamp

```bash
$ touch -a greeting.txt
$ stat -c %x greeting.txt
2017-07-21 10:14:08.457268564 +0530
$ stat -c %y greeting.txt
2017-07-13 13:54:26.004499660 +0530

$ touch -m sample.txt
$ stat -c %x sample.txt
2017-07-13 13:48:24.945450646 +0530
$ stat -c %y sample.txt
2017-07-21 10:14:40.770006144 +0530
```

* Using timestamp from another file to update

```bash
$ stat -c $'%x\n%y' power.log report.log
2017-07-19 10:48:03.978295434 +0530
2017-07-14 20:50:42.850887578 +0530
2017-06-24 13:00:31.773583923 +0530
2017-06-24 12:59:53.316751651 +0530

$ # copy both access and modification timestamp from power.log to report.log
$ touch -r power.log report.log
$ stat -c $'%x\n%y' report.log
2017-07-19 10:48:03.978295434 +0530
2017-07-14 20:50:42.850887578 +0530

$ # add -a or -m options to limit to only access or modification timestamp
```

* Using date string to update
* See also `-t` option

```bash
$ # add -a or -m as needed
$ touch -d '2010-03-17 17:04:23' report.log
$ stat -c $'%x\n%y' report.log
2010-03-17 17:04:23.000000000 +0530
2010-03-17 17:04:23.000000000 +0530
```

<br>

#### <a name="preserving-timestamp"></a>Preserving timestamp

* Text processing on files would update the timestamps

```bash
$ stat -c $'%x\n%y' power.log
2017-07-21 11:11:42.862874240 +0530
2017-07-13 21:31:53.496323704 +0530

$ sed -i 's/foo/bar/g' power.log
$ stat -c $'%x\n%y' power.log
2017-07-21 11:12:20.303504336 +0530
2017-07-21 11:12:20.303504336 +0530
```

* `touch` can be used to restore timestamps after processing

```bash
$ # first copy the timestamps using touch -r
$ stat -c $'%x\n%y' story.txt
2017-06-24 13:00:31.773583923 +0530
2017-06-24 12:59:53.316751651 +0530
$ # tmp.txt is temporary empty file
$ touch -r story.txt tmp.txt
$ stat -c $'%x\n%y' tmp.txt
2017-06-24 13:00:31.773583923 +0530
2017-06-24 12:59:53.316751651 +0530

$ # after text processing, copy back the timestamps and remove temporary file
$ sed -i 's/cat/dog/g' story.txt
$ touch -r tmp.txt story.txt && rm tmp.txt
$ stat -c $'%x\n%y' story.txt
2017-06-24 13:00:31.773583923 +0530
2017-06-24 12:59:53.316751651 +0530
```

<br>

#### <a name="further-reading-for-touch"></a>Further reading for touch

* `man touch` and `info touch` for more options and detailed documentation
* [touch Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/touch?sort=votes&pageSize=15)

<br>

## <a name="file"></a>file

```bash
$ file --version | head -n1
file-5.25

$ man file
FILE(1)                   BSD General Commands Manual                  FILE(1)

NAME
     file — determine file type

SYNOPSIS
     file [-bcEhiklLNnprsvzZ0] [--apple] [--extension] [--mime-encoding]
          [--mime-type] [-e testname] [-F separator] [-f namefile]
          [-m magicfiles] [-P name=value] file ...
     file -C [-m magicfiles]
     file [--help]

DESCRIPTION
     This manual page documents version 5.25 of the file command.

     file tests each argument in an attempt to classify it.  There are three
     sets of tests, performed in this order: filesystem tests, magic tests,
     and language tests.  The first test that succeeds causes the file type to
     be printed.
...
```

<br>

<br>

#### <a name="file-type-examples"></a>File type examples

```bash
$ file sample.txt
sample.txt: ASCII text
$ # without file name in output
$ file -b sample.txt
ASCII text

$ printf 'hi👍\n' | file -
/dev/stdin: UTF-8 Unicode text
$ printf 'hi👍\n' | file -i -
/dev/stdin: text/plain; charset=utf-8

$ file ch
ch:  Bourne-Again shell script, ASCII text executable

$ file sunset.jpg moon.png
sunset.jpg: JPEG image data
moon.png: PNG image data, 32 x 32, 8-bit/color RGBA, non-interlaced
```

* different line terminators

```bash
$ printf 'hi' | file -
/dev/stdin: ASCII text, with no line terminators

$ printf 'hi\r' | file -
/dev/stdin: ASCII text, with CR line terminators

$ printf 'hi\r\n' | file -
/dev/stdin: ASCII text, with CRLF line terminators

$ printf 'hi\n' | file -
/dev/stdin: ASCII text
```

* find all files of particular type in current directory, for example `image` files

```bash
$ find -type f -exec bash -c '(file -b "$0" | grep -wq "image data") && echo "$0"' {} \;
./sunset.jpg
./moon.png

$ # if filenames do not contain : or newline characters
$ find -type f -exec file {} + | awk -F: '/\<image data\>/{print $1}'
./sunset.jpg
./moon.png
```

<br>

#### <a name="further-reading-for-file"></a>Further reading for file

* `man file` and `info file` for more options and detailed documentation
* See also `identify` command which `describes the format and characteristics of one or more image files`

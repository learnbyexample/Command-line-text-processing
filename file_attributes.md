# <a name="file-attributes"></a>File attributes

**Table of Contents**

* [wc](#wc)
    * [Examples](#examples)
    * [subtle differences](#subtle-differences)
    * [Further reading for wc](#further-reading-for-wc)
* [du](#du)
    * [Various size formats](#various-size-formats)
* [df](#df)
* [touch](#touch)
* [file](#file)
* [identify](#identify)

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

#### <a name="examples"></a>Examples

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

#### <a name="various-size-formats"></a>Various size formats

* By default, size is given in size of 1024 bytes

```bash
$ ls -F
projs/  py_learn@  words.txt

$ # by default files are ignored
$ # all directories and sub-directories are recursively reported
$ du
17920   ./projs/full_addr
14316   ./projs/half_addr
32952   ./projs
33880   .

$ # -a to get size for files too
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

$ # -s to get only total without descending inside directories
$ du -s projs words.txt
32952   projs
924     words.txt
```

* changing default size format

```bash
$ # number of bytes
$ stat -c %s words.txt 
938848
$ du -b words.txt
938848  words.txt

$ # number of kilobytes
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

* dereferencing links, see `man` and `info` pages for other related options

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

**Examples**

* `du project_report` display size (default unit is 1024 bytes) of folder project_report as well as it sub-directories
* `du --si project_report` display size (unit is 1000 bytes) of folder project_report as well as it sub-directories
* `du -h project_report` display size in human readable format for folder project_report as well as it sub-directories
* `du -ah project_report` display size in human readable format for folder project_report and all of its files and sub-directories
* `du -sh project_report` display size only for project_report folder in human readable format
* `du -sm * | sort -n` sort files and folders of current directory, numbers displayed are in Megabytes
    * use `sort -nr` to reverse sort order, i.e largest at top
* `du -sh * | sort -h` sort files and folders of current directory, output displayed in human-readable format
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

**Examples**

* `df -h` display usage statistics of all available file systems
* `df -h .` display usage of only the file system where the current working directory is located
* [df Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/df?sort=votes&pageSize=15)

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

Used to change file time stamps. But if file doesn't exist, the command will create an empty file with the name provided. Both features are quite useful  

* Some program may require a particular file to be present to work, empty file might even be a valid argument. In such cases, a pre-processing script can scan the destination directories and create empty file if needed
* Similarly, some programs may behave differently according to the time stamps of two or more files - while debugging in such an environment, user might want to just change the time stamp of files

**Examples**

* `touch new_file.txt` create an empty file if it doesn't exist in current directory
    * use `-c` if new file shouldn't be created
    * use `-a` option to change only access time and `-m` to change only modification time
* `touch report.log` change the time stamp of report.log to current time (assuming report.log already exists in current directory)
* `touch -r power.log report.log` use time stamp of power.log instead of current time to change that of report.log
    * use `-d` to provide time stamp from a string instead of file
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

**Examples**

```bash
$ file sample.txt 
sample.txt: ASCII text

$ file sunset.jpg 
sunset.jpg: JPEG image data

$ mv sunset.jpg xyz
$ file xyz 
xyz: JPEG image data

$ file perl.png 
perl.png: PNG image data, 32 x 32, 8-bit/color RGBA, non-interlaced
```

<br>

## <a name="identify"></a>identify

```bash
$ identify --version | head -n1
Version: ImageMagick 6.8.9-9 Q16 x86_64 2017-05-26 http://www.imagemagick.org

$ man identify
identify(1)                 General Commands Manual                identify(1)

NAME
       identify  -  describes  the  format  and characteristics of one or more
       image files.

SYNOPSIS
       identify [options] input-file

OVERVIEW
       The identify program is a member of the ImageMagick(1) suite of  tools.
       It describes the format and characteristics of one or more image files.
       It also reports if an image is incomplete or corrupt.  The  information
       returned includes the image number, the file name, the width and height
       of the image, whether the image is colormapped or not,  the  number  of
       colors  in  the  image (by default off use -define unique=true option),
       the number of bytes in the image, the format of the image  (JPEG,  PNM,
       etc.),  and  finally  the number of seconds it took to read and process
       the image. Many more attributes are available with the verbose option.
...
```

Although file command can also give information like pixel dimensions and image type, identify is more reliable command for images and gives complete format information

```bash
$ identify sunset.jpg
sunset.jpg JPEG 740x740 740x740+0+0 8-bit DirectClass 110KB 0.000u 0:00.030

$ identify perl.png 
perl.png PNG 32x32 32x32+0+0 8-bit DirectClass 838B 0.000u 0:00.000
```

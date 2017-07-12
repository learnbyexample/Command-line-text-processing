# <a name="file-attributes"></a>File attributes

**Table of Contents**

* [wc](#wc)
* [du](#du)
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

**Examples**

* `wc power.log` outputs no. of lines, words and characters separated by white-space and followed by filename
* `wc -l power.log` outputs no. of lines followed by filename
* `wc -w power.log` outputs no. of words followed by filename
* `wc -c power.log` outputs no. of characters followed by filename
* `wc -l < power.log` output only the number of lines
* `wc -L power.log` length of longest line followed by filename
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

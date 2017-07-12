# <a name="whats-the-difference"></a>What's the difference

**Table of Contents**

* [cmp](#cmp)
* [diff](#diff)

<br>

## <a name="cmp"></a>cmp

```bash
$ cmp --version | head -n1
cmp (GNU diffutils) 3.3

$ man cmp
CMP(1)                           User Commands                          CMP(1)

NAME
       cmp - compare two files byte by byte

SYNOPSIS
       cmp [OPTION]... FILE1 [FILE2 [SKIP1 [SKIP2]]]

DESCRIPTION
       Compare two files byte by byte.

       The optional SKIP1 and SKIP2 specify the number of bytes to skip at the
       beginning of each file (zero by default).
...
```

Useful to compare binary files. If the two files are same, no output is displayed (exit status 0)  
If there is a difference, it prints the first difference - line number and byte location (exit status 1)  
Option `-s` allows to suppress the output, useful in scripts

```bash
$ cmp /bin/grep /bin/fgrep
/bin/grep /bin/fgrep differ: byte 25, line 1
```

* More examples [here](http://www.sanfoundry.com/5-cmp-command-usage-examples-linux/)

<br>

## <a name="diff"></a>diff

```bash
$ diff --version | head -n1
diff (GNU diffutils) 3.3

$ man diff
DIFF(1)                          User Commands                         DIFF(1)

NAME
       diff - compare files line by line

SYNOPSIS
       diff [OPTION]... FILES

DESCRIPTION
       Compare FILES line by line.
...
```

Useful to compare old and new versions of text files  
All the differences are printed, which might not be desirable if files are too long

**Options**

* `-s` convey message when two files are same
* `-y` two column output
* `-i` ignore case while comparing
* `-w` ignore white-spaces
* `-r` recursively compare files between the two directories specified
* `-q` report if files differ, not the details of difference

**Examples**

* `diff -s test_list_mar2.txt test_list_mar3.txt` compare two files
* `diff -s report.log bkp/mar10/` no need to specify second filename if names are same
* `diff -qr report/ bkp/mar10/report/` recursively compare files between report and bkp/mar10/report directories, filenames not matching are also specified in output
    * see [this link](https://stackoverflow.com/questions/6217628/diff-to-output-only-the-file-names) for detailed analysis and corner cases
* `diff report/ bkp/mar10/report/ | grep -w '^diff'` useful trick to get only names of mismatching files (provided no mismatches contain the whole word diff at start of line)

**Further Reading**

* [diff Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/diff?sort=votes&pageSize=15)
* `gvimdiff` edit two, three or four versions of a file with Vim and show differences
* [GUI diff and merge tools](http://askubuntu.com/questions/2946/what-are-some-good-gui-diff-and-merge-applications-available-for-ubuntu)


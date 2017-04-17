# <a name="sorting-stuff"></a>Sorting stuff

**Table of Contents**

* [sort](#sort)
    * [Default sort](#default-sort)
    * [Reverse sort](#reverse-sort)
    * [Various number sorting](#various-number-sorting)
* [uniq](#uniq)
* [comm](#comm)

<br>

## <a name="sort"></a>sort

```bash
$ sort --version | head -1
sort (GNU coreutils) 8.25

$ man sort
SORT(1)                          User Commands                         SORT(1)

NAME
       sort - sort lines of text files

SYNOPSIS
       sort [OPTION]... [FILE]...
       sort [OPTION]... --files0-from=F

DESCRIPTION
       Write sorted concatenation of all FILE(s) to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

**Note**: All examples shown here assumes ASCII encoded input file


<br>

#### <a name="default-sort"></a>Default sort

```bash
$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ sort poem.txt 
And so are you.
Roses are red,
Sugar is sweet,
Violets are blue,
```

* Well, that was easy. The lines were sorted alphabetically (ascending order by default) and it so happened that first letter alone was enough to decide the order
* For next example, let's extract all the words and sort them
    * also allows to showcase `sort` accepting stdin
    * See [GNU grep](./gnu_grep.md) chapter if the `grep` command used below looks alien

```bash
$ # output might differ depending on locale settings
$ # note the case-insensitiveness of output
$ grep -oi '[a-z]*' poem.txt | sort
And
are
are
are
blue
is
red
Roses
so
Sugar
sweet
Violets
you
```

* heed hereunto

```bash
$ info sort | tail

   (1) If you use a non-POSIX locale (e.g., by setting ‘LC_ALL’ to
‘en_US’), then ‘sort’ may produce output that is sorted differently than
you’re accustomed to.  In that case, set the ‘LC_ALL’ environment
variable to ‘C’.  Note that setting only ‘LC_COLLATE’ has two problems.
First, it is ineffective if ‘LC_ALL’ is also set.  Second, it has
undefined behavior if ‘LC_CTYPE’ (or ‘LANG’, if ‘LC_CTYPE’ is unset) is
set to an incompatible value.  For example, you get undefined behavior
if ‘LC_CTYPE’ is ‘ja_JP.PCK’ but ‘LC_COLLATE’ is ‘en_US.UTF-8’.
```

* Example to help show effect of locale setting

```bash
$ # note how uppercase is sorted before lowercase
$ grep -oi '[a-z]*' poem.txt | LC_ALL=C sort
And
Roses
Sugar
Violets
are
are
are
blue
is
red
so
sweet
you
```

<br>

#### <a name="reverse-sort"></a>Reverse sort

* This is simply reversing from default ascending order to descending order

```bash
$ sort -r poem.txt 
Violets are blue,
Sugar is sweet,
Roses are red,
And so are you.
```

<br>

#### <a name="various-number-sorting"></a>Various number sorting

```bash
$ cat numbers.txt 
20
53
3
101

$ sort numbers.txt 
101
20
3
53
```

* Whoops, what happened there? `sort` won't know to treat them as numbers unless specified
* Depending on format of numbers, different options have to be used
* First up is `-n` option, which sorts based on numerical value

```bash
$ sort -n numbers.txt 
3
20
53
101

$ sort -nr numbers.txt 
101
53
20
3
```

* The `-n` option can handle negative numbers
* As well as thousands separator and decimal point (depends on locale)

```bash
$ # multiple files are merged as single input by default
$ sort -n numbers.txt <(echo '-4')
-4
3
20
53
101

$ sort -n numbers.txt <(echo '1,234')
3
20
53
101
1,234

$ sort -n numbers.txt <(echo '31.24')
3
20
31.24
53
101
```

* Use `-g` if input contains numbers prefixed by `+` or [E scientific notation](https://en.wikipedia.org/wiki/Scientific_notation#E_notation)

```bash
$ cat generic_numbers.txt 
+120
-1.53
3.14e+4
42.1e-2

$ sort -g generic_numbers.txt 
-1.53
42.1e-2
+120
3.14e+4
```

* Commands like `du` have options to display numbers in human readable formats
* `sort` supports sorting such numbers using the `-h` option

```bash
$ du -sh *
104K    power.log
746M    projects
316K    report.log
20K     sample.txt
$ du -sh * | sort -h
20K     sample.txt
104K    power.log
316K    report.log
746M    projects

$ # --si uses powers of 1000 instead of 1024
$ du -s --si *
107k    power.log
782M    projects
324k    report.log
21k     sample.txt
$ du -s --si * | sort -h
21k     sample.txt
107k    power.log
324k    report.log
782M    projects
```

* Version sort - dealing with numbers mixed with other characters
* If this sorting is needed simply while displaying directory contents, use `ls -v` instead of piping to `sort -V`

```bash
$ cat versions.txt 
foo_v1.2
bar_v2.1.3
foobar_v2
foo_v1.2.1
foo_v1.3

$ sort -V versions.txt 
bar_v2.1.3
foobar_v2
foo_v1.2
foo_v1.2.1
foo_v1.3
```

* Another common use case is when there are multiple filenames differentiated by numbers

```bash
$ cat files.txt 
file0
file10
file3
file4

$ sort -V files.txt 
file0
file3
file4
file10
```

* Can be used when dealing with numbers reported by `time` command as well

```bash
$ # different solving durations
$ cat rubik_time.txt 
5m35.363s
3m20.058s
4m5.099s
4m1.130s
3m42.833s
4m33.083s

$ # assuming consistent min/sec format
$ sort -V rubik_time.txt 
3m20.058s
3m42.833s
4m1.130s
4m5.099s
4m33.083s
5m35.363s
```

More to follow...

<br>

## <a name="uniq"></a>uniq

>report or omit repeated lines

This command is more specific to recognizing duplicates. Usually requires a sorted input as the comparison is made on adjacent lines only

**Options**

* `-d` print only duplicate lines
* `-c` prefix count to occurrences
* `-u` print only unique lines

**Examples**

* `sort test_list.txt | uniq` outputs lines of test_list.txt in sorted order with duplicate lines removed
	* `uniq <(sort test_list.txt)` same command using process substitution
	* `sort -u test_list.txt` equivalent command
* `uniq -d sorted_list.txt` print only duplicate lines
* `uniq -cd sorted_list.txt` print only duplicate lines and prefix the line with number of times it is repeated
* `uniq -u sorted_list.txt` print only unique lines, repeated lines are ignored
* [uniq Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/uniq?sort=votes&pageSize=15)

```bash
$ echo -e 'Blue\nRed\nGreen\nBlue\nRed\nBlack\nRed' > colors.txt 
$ uniq colors.txt 
Blue
Red
Green
Blue
Red
Black
Red

$ echo -e 'Blue\nRed\nGreen\nBlue\nRed\nBlack\nRed' | sort > sorted_colors.txt 
$ uniq sorted_colors.txt
Black
Blue
Green
Red

$ uniq -d sorted_colors.txt 
Blue
Red

$ uniq -cd sorted_colors.txt 
      2 Blue
      3 Red
      
$ uniq -u sorted_colors.txt 
Black
Green
```

<br>

## <a name="comm"></a>comm

>compare two sorted files line by line

Without any options, it prints output in three columns - lines unique to file1, line unique to file2 and lines common to both files

**Options**

* `-1` suppress lines unique to file1
* `-2` suppress lines unique to file2
* `-3` suppress lines common to both files

**Examples**

* `comm -23 sorted_file1.txt sorted_file2.txt` print lines unique to sorted_file1.txt
    * `comm -23 <(sort file1.txt) <(sort file2.txt)'` same command using process substitution, if sorted input files are not available
* `comm -13 sorted_file1.txt sorted_file2.txt` print lines unique to sorted_file2.txt
* `comm -12 sorted_file1.txt sorted_file2.txt` print lines common to both files
* [comm Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/comm?sort=votes&pageSize=15)

```bash
$ echo -e 'Brown\nRed\nPurple\nBlue\nTeal\nYellow' | sort > colors_1.txt 
$ echo -e 'Red\nGreen\nBlue\nBlack\nWhite' | sort > colors_2.txt 

$ # the input files viewed side by side
$ paste colors_1.txt colors_2.txt
Blue    Black
Brown   Blue
Purple  Green
Red     Red
Teal    White
Yellow  
```

* examples

```bash
$ # 3 column output - unique to file1, file2 and common
$ comm colors_1.txt colors_2.txt
        Black
                Blue
Brown
        Green
Purple
                Red
Teal
        White
Yellow 

$ # suppress 1 and 2 column, gives only common lines
$ comm -12 colors_1.txt colors_2.txt
Blue
Red

$ # suppress 1 and 3 column, gives lines unique to file2
$ comm -13 colors_1.txt colors_2.txt
Black
Green
White

$ # suppress 2 and 3 column, gives lines unique to file1
$ comm -23 colors_1.txt colors_2.txt
Brown
Purple
Teal
Yellow
```


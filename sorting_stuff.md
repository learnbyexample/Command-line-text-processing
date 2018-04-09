# <a name="sorting-stuff"></a>Sorting stuff

**Table of Contents**

* [sort](#sort)
    * [Default sort](#default-sort)
    * [Reverse sort](#reverse-sort)
    * [Various number sorting](#various-number-sorting)
    * [Random sort](#random-sort)
    * [Specifying output file](#specifying-output-file)
    * [Unique sort](#unique-sort)
    * [Column based sorting](#column-based-sorting)
    * [Further reading for sort](#further-reading-for-sort)
* [uniq](#uniq)
    * [Default uniq](#default-uniq)
    * [Only duplicates](#only-duplicates)
    * [Only unique](#only-unique)
    * [Prefix count](#prefix-count)
    * [Ignoring case](#ignoring-case)
    * [Combining multiple files](#combining-multiple-files)
    * [Column options](#column-options)
    * [Further reading for uniq](#further-reading-for-uniq)
* [comm](#comm)
    * [Default three column output](#default-three-column-output)
    * [Suppressing columns](#suppressing-columns)
    * [Files with duplicates](#files-with-duplicates)
    * [Further reading for comm](#further-reading-for-comm)
* [shuf](#shuf)
    * [Random lines](#random-lines)
    * [Random integer numbers](#random-integer-numbers)
    * [Further reading for shuf](#further-reading-for-shuf)

<br>

## <a name="sort"></a>sort

```bash
$ sort --version | head -n1
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
* See also
    * [arch wiki - locale](https://wiki.archlinux.org/index.php/locale)
    * [Linux: Define Locale and Language Settings](https://www.shellhacks.com/linux-define-locale-language-settings/)

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
* The `<()` syntax is [Process Substitution](http://mywiki.wooledge.org/ProcessSubstitution)
    * to put it simply - allows output of command to be passed as input file to another command without needing to manually create a temporary file

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

<br>

#### <a name="random-sort"></a>Random sort

* Note that duplicate lines will always end up next to each other
    * might be useful as a feature for some cases ;)
    * Use `shuf` if this is not desirable
* See also [How can I shuffle the lines of a text file on the Unix command line or in a shell script?](https://stackoverflow.com/questions/2153882/how-can-i-shuffle-the-lines-of-a-text-file-on-the-unix-command-line-or-in-a-shel)

```bash
$ cat nums.txt
1
10
10
12
23
563

$ # the two 10s will always be next to each other
$ sort -R nums.txt
563
12
1
10
10
23

$ # duplicates can end up anywhere
$ shuf nums.txt
10
23
1
10
563
12
```

<br>

#### <a name="specifying-output-file"></a>Specifying output file

* The `-o` option can be used to specify output file
* Useful for in place editing

```bash
$ sort -R nums.txt -o rand_nums.txt
$ cat rand_nums.txt
23
1
10
10
563
12

$ sort -R nums.txt -o nums.txt
$ cat nums.txt
563
23
10
10
1
12
```

* Use shell script looping if there multiple files to be sorted in place
* Below snippet is for `bash` shell

```bash
$ for f in *.txt; do echo sort -V "$f" -o "$f"; done
sort -V files.txt -o files.txt
sort -V rubik_time.txt -o rubik_time.txt
sort -V versions.txt -o versions.txt

$ # remove echo once commands look fine
$ for f in *.txt; do sort -V "$f" -o "$f"; done
```

<br>

#### <a name="unique-sort"></a>Unique sort

* Keep only first copy of lines that are deemed to be same according to `sort` option used

```bash
$ cat duplicates.txt
foo
12 carrots
foo
12 apples
5 guavas

$ # only one copy of foo in output
$ sort -u duplicates.txt
12 apples
12 carrots
5 guavas
foo
```

* According to option used, definition of duplicate will vary
* For example, when `-n` is used, matching numbers are deemed same even if rest of line differs
    * Pipe the output to `uniq` if this is not desirable

```bash
$ # note how first copy of line starting with 12 is retained
$ sort -nu duplicates.txt
foo
5 guavas
12 carrots

$ # use uniq when entire line should be compared to find duplicates
$ sort -n duplicates.txt | uniq
foo
5 guavas
12 apples
12 carrots
```

* Use `-f` option to ignore case of alphabets while determining duplicates

```bash
$ cat words.txt
CAR
are
car
Are
foot
are

$ # only the two 'are' were considered duplicates
$ sort -u words.txt
are
Are
car
CAR
foot

$ # note again that first copy of duplicate is retained
$ sort -fu words.txt
are
CAR
foot
```

<br>

#### <a name="column-based-sorting"></a>Column based sorting

From `info sort`

```
‘-k POS1[,POS2]’
‘--key=POS1[,POS2]’
     Specify a sort field that consists of the part of the line between
     POS1 and POS2 (or the end of the line, if POS2 is omitted),
     _inclusive_.

     Each POS has the form ‘F[.C][OPTS]’, where F is the number of the
     field to use, and C is the number of the first character from the
     beginning of the field.  Fields and character positions are
     numbered starting with 1; a character position of zero in POS2
     indicates the field’s last character.  If ‘.C’ is omitted from
     POS1, it defaults to 1 (the beginning of the field); if omitted
     from POS2, it defaults to 0 (the end of the field).  OPTS are
     ordering options, allowing individual keys to be sorted according
     to different rules; see below for details.  Keys can span multiple
     fields.
```

* By default, blank characters (space and tab) serve as field separators

```bash
$ cat fruits.txt
apple   42
guava   6
fig     90
banana  31

$ sort fruits.txt
apple   42
banana  31
fig     90
guava   6

$ # sort based on 2nd column numbers
$ sort -k2,2n fruits.txt
guava   6
banana  31
apple   42
fig     90
```

* Using a different field separator
* Consider the following sample input file having fields separated by `:`

```bash
$ # name:pet_name:no_of_pets
$ cat pets.txt
foo:dog:2
xyz:cat:1
baz:parrot:5
abcd:cat:3
joe:dog:1
bar:fox:1
temp_var:squirrel:4
boss:dog:10
```

* Sorting based on particular column or column to end of line
* In case of multiple entries, by default `sort` would use content of remaining parts of line to resolve

```bash
$ # only 2nd column
$ # -k2,4 would mean 2nd column to 4th column
$ sort -t: -k2,2 pets.txt
abcd:cat:3
xyz:cat:1
boss:dog:10
foo:dog:2
joe:dog:1
bar:fox:1
baz:parrot:5
temp_var:squirrel:4

$ # from 2nd column to end of line
$ sort -t: -k2 pets.txt
xyz:cat:1
abcd:cat:3
joe:dog:1
boss:dog:10
foo:dog:2
bar:fox:1
baz:parrot:5
temp_var:squirrel:4
```

* Multiple keys can be specified to resolve ties
* Note that if there are still multiple entries with specified keys, remaining parts of lines would be used

```bash
$ # default sort for 2nd column, numeric sort on 3rd column to resolve ties
$ sort -t: -k2,2 -k3,3n pets.txt
xyz:cat:1
abcd:cat:3
joe:dog:1
foo:dog:2
boss:dog:10
bar:fox:1
baz:parrot:5
temp_var:squirrel:4

$ # numeric sort on 3rd column, default sort for 2nd column to resolve ties
$ sort -t: -k3,3n -k2,2 pets.txt
xyz:cat:1
joe:dog:1
bar:fox:1
foo:dog:2
abcd:cat:3
temp_var:squirrel:4
baz:parrot:5
boss:dog:10
```

* Use `-s` option to retain original order of lines in case of tie

```bash
$ sort -s -t: -k2,2 pets.txt
xyz:cat:1
abcd:cat:3
foo:dog:2
joe:dog:1
boss:dog:10
bar:fox:1
baz:parrot:5
temp_var:squirrel:4
```

* The `-u` option, as seen earlier, will retain only first match

```bash
$ sort -u -t: -k2,2 pets.txt
xyz:cat:1
foo:dog:2
bar:fox:1
baz:parrot:5
temp_var:squirrel:4

$ sort -u -t: -k3,3n pets.txt
xyz:cat:1
foo:dog:2
abcd:cat:3
temp_var:squirrel:4
baz:parrot:5
boss:dog:10
```

* Sometimes, the input has to be sorted first and then `-u` used on the sorted output
* See also [remove duplicates based on the value of another column](https://unix.stackexchange.com/questions/379835/remove-duplicates-based-on-the-value-of-another-column)

```bash
$ # sort by number in 3rd column
$ sort -t: -k3,3n pets.txt
bar:fox:1
joe:dog:1
xyz:cat:1
foo:dog:2
abcd:cat:3
temp_var:squirrel:4
baz:parrot:5
boss:dog:10

$ # then get unique entry based on 2nd column
$ sort -t: -k3,3n pets.txt | sort -t: -u -k2,2
xyz:cat:1
joe:dog:1
bar:fox:1
baz:parrot:5
temp_var:squirrel:4
```

* Specifying particular characters within fields
* If character position is not specified, defaults to `1` for starting column and `0` (last character) for ending column

```bash
$ cat marks.txt
fork,ap_12,54
flat,up_342,1.2
fold,tn_48,211
more,ap_93,7
rest,up_5,63

$ # for 2nd column, sort numerically only from 4th character to end
$ sort -t, -k2.4,2n marks.txt
rest,up_5,63
fork,ap_12,54
fold,tn_48,211
more,ap_93,7
flat,up_342,1.2

$ # sort uniquely based on first two characters of line
$ sort -u -k1.1,1.2 marks.txt
flat,up_342,1.2
fork,ap_12,54
more,ap_93,7
rest,up_5,63
```

* If there are headers

```bash
$ cat header.txt
fruit   qty
apple   42
guava   6
fig     90
banana  31

$ # separate and combine header and content to be sorted
$ cat <(head -n1 header.txt) <(tail -n +2 header.txt | sort -k2nr)
fruit   qty
fig     90
apple   42
banana  31
guava   6
```

* See also [sort by last field value when number of fields varies](https://stackoverflow.com/questions/3832068/bash-sort-text-file-by-last-field-value)

<br>

#### <a name="further-reading-for-sort"></a>Further reading for sort

* There are many other options apart from handful presented above. See `man sort` and `info sort` for detailed documentation and more examples
* [sort like a master](http://www.skorks.com/2010/05/sort-files-like-a-master-with-the-linux-sort-command-bash/)
* [When -b to ignore leading blanks is needed](https://unix.stackexchange.com/a/104527/109046)
* [sort Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/sort?sort=votes&pageSize=15)
* [sort on multiple columns using -k option](https://unix.stackexchange.com/questions/249452/unix-multiple-column-sort-issue)
* [sort a string character wise](https://stackoverflow.com/questions/2373874/how-to-sort-characters-in-a-string)
* [Scalability of 'sort -u' for gigantic files](https://unix.stackexchange.com/questions/279096/scalability-of-sort-u-for-gigantic-files)

<br>

## <a name="uniq"></a>uniq

```bash
$ uniq --version | head -n1
uniq (GNU coreutils) 8.25

$ man uniq
UNIQ(1)                          User Commands                         UNIQ(1)

NAME
       uniq - report or omit repeated lines

SYNOPSIS
       uniq [OPTION]... [INPUT [OUTPUT]]

DESCRIPTION
       Filter  adjacent matching lines from INPUT (or standard input), writing
       to OUTPUT (or standard output).

       With no options, matching lines are merged to the first occurrence.
...
```

<br>

#### <a name="default-uniq"></a>Default uniq

```bash
$ cat word_list.txt
are
are
to
good
bad
bad
bad
good
are
bad

$ # adjacent duplicate lines are removed, leaving one copy
$ uniq word_list.txt
are
to
good
bad
good
are
bad

$ # To remove duplicates from entire file, input has to be sorted first
$ # also showcases that uniq accepts stdin as input
$ sort word_list.txt | uniq
are
bad
good
to
```

<br>

#### <a name="only-duplicates"></a>Only duplicates

```bash
$ # duplicates adjacent to each other
$ uniq -d word_list.txt
are
bad

$ # duplicates in entire file
$ sort word_list.txt | uniq -d
are
bad
good
```

* To get only duplicates as well as show all duplicates

```bash
$ uniq -D word_list.txt
are
are
bad
bad
bad

$ sort word_list.txt | uniq -D
are
are
are
bad
bad
bad
bad
good
good
```

* To distinguish the different groups

```bash
$ # using --all-repeated=prepend will add a newline before the first group as well
$ sort word_list.txt | uniq --all-repeated=separate
are
are
are

bad
bad
bad
bad

good
good
```

<br>

#### <a name="only-unique"></a>Only unique

```bash
$ # lines with no adjacent duplicates
$ uniq -u word_list.txt
to
good
good
are
bad

$ # unique lines in entire file
$ sort word_list.txt | uniq -u
to
```

<br>

#### <a name="prefix-count"></a>Prefix count

```bash
$ # adjacent lines
$ uniq -c word_list.txt
      2 are
      1 to
      1 good
      3 bad
      1 good
      1 are
      1 bad

$ # entire file
$ sort word_list.txt | uniq -c
      3 are
      4 bad
      2 good
      1 to

$ # entire file, only duplicates
$ sort word_list.txt | uniq -cd
      3 are
      4 bad
      2 good
```

* Sorting by count

```bash
$ # sort by count
$ sort word_list.txt | uniq -c | sort -n
      1 to
      2 good
      3 are
      4 bad

$ # reverse the order, highest count first
$ sort word_list.txt | uniq -c | sort -nr
      4 bad
      3 are
      2 good
      1 to
```

* To get only entries with min/max count, bit of [awk](./gnu_awk.md) magic would help

```bash
$ # consider this result
$ sort colors.txt | uniq -c | sort -nr
      3 Red
      3 Blue
      2 Yellow
      1 Green
      1 Black

$ # to get all max count
$ # save 1st line 1st column value to c and then print if 1st column equals c
$ sort colors.txt | uniq -c | sort -nr | awk 'NR==1{c=$1} $1==c'
      3 Red
      3 Blue
$ # to get all min count
$ sort colors.txt | uniq -c | sort -n | awk 'NR==1{c=$1} $1==c'
      1 Black
      1 Green
```

* Get rough count of most used commands from `history` file

```bash
$ # awk '{print $1}' will get the 1st column alone
$ awk '{print $1}' "$HISTFILE" | sort | uniq -c | sort -nr | head
   1465 echo
   1180 grep
    552 cd
    531 awk
    451 sed
    423 vi
    418 cat
    392 perl
    325 printf
    320 sort

$ # extract command name from start of line or preceded by 'spaces|spaces'
$ # won't catch commands in other places like command substitution though
$ grep -oP '(^| +\| +)\K[^ ]+' "$HISTFILE" | sort | uniq -c | sort -nr | head
   2006 grep
   1469 echo
    933 sed
    698 awk
    552 cd
    513 perl
    510 cat
    453 sort
    423 vi
    327 printf
```

<br>

#### <a name="ignoring-case"></a>Ignoring case

```bash
$ cat another_list.txt
food
Food
good
are
bad
Are

$ # note how first copy is retained
$ uniq -i another_list.txt
food
good
are
bad
Are

$ uniq -iD another_list.txt
food
Food
```

<br>

#### <a name="combining-multiple-files"></a>Combining multiple files

```bash
$ sort -f word_list.txt another_list.txt | uniq -i
are
bad
food
good
to

$ sort -f word_list.txt another_list.txt | uniq -c
      4 are
      1 Are
      5 bad
      1 food
      1 Food
      3 good
      1 to

$ sort -f word_list.txt another_list.txt | uniq -ic
      5 are
      5 bad
      2 food
      3 good
      1 to
```

* If only adjacent lines (not sorted) is required, need to concatenate files using another command

```bash
$ uniq -id word_list.txt
are
bad

$ uniq -id another_list.txt
food

$ cat word_list.txt another_list.txt | uniq -id
are
bad
food
```

<br>

#### <a name="column-options"></a>Column options

* `uniq` has few options dealing with column manipulations. Not extensive as `sort -k` but handy for some cases
* First up, skipping fields
    * No option to specify different delimiter
    * From `info uniq`: Fields are sequences of non-space non-tab characters that are separated from each other by at least one space or tab
    * Number of spaces/tabs between fields should be same

```bash
$ cat shopping.txt
lemon 5
mango 5
banana 8
bread 1
orange 5

$ # skips first field
$ uniq -f1 shopping.txt
lemon 5
banana 8
bread 1
orange 5

$ # use -f3 to skip first three fields and so on
```

* Skipping characters

```bash
$ cat text
glue
blue
black
stack
stuck

$ # don't consider first 2 characters
$ uniq -s2 text
glue
black
stuck

$ # to visualize the above example
$ # assume there are two fields and uniq is applied on 2nd column
$ sed 's/^../& /' text
gl ue
bl ue
bl ack
st ack
st uck
```

* Upto specified characters

```bash
$ # consider only first 2 characters
$ uniq -w2 text
glue
blue
stack

$ # to visualize the above example
$ # assume there are two fields and uniq is applied on 1st column
$ sed 's/^../& /' text
gl ue
bl ue
bl ack
st ack
st uck
```

* Combining `-s` and `-w`
* Can be combined with `-f` as well

```bash
$ # skip first 3 characters and then use next 2 characters
$ uniq -s3 -w2 text
glue
black
```


<br>

#### <a name="further-reading-for-uniq"></a>Further reading for uniq

* Do check out `man uniq` and `info uniq` for other options and more detailed documentation
* [uniq Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/uniq?sort=votes&pageSize=15)
* [process duplicate lines only based on certain fields](https://unix.stackexchange.com/questions/387590/print-the-duplicate-lines-only-on-fields-1-2-from-csv-file)

<br>

## <a name="comm"></a>comm

```bash
$ comm --version | head -n1
comm (GNU coreutils) 8.25

$ man comm
COMM(1)                          User Commands                         COMM(1)

NAME
       comm - compare two sorted files line by line

SYNOPSIS
       comm [OPTION]... FILE1 FILE2

DESCRIPTION
       Compare sorted files FILE1 and FILE2 line by line.

       When FILE1 or FILE2 (not both) is -, read standard input.

       With  no  options,  produce  three-column  output.  Column one contains
       lines unique to FILE1, column two contains lines unique to  FILE2,  and
       column three contains lines common to both files.
...
```

<br>

#### <a name="default-three-column-output"></a>Default three column output

Consider below sample input files

```bash
$ # sorted input files viewed side by side
$ paste colors_1.txt colors_2.txt
Blue    Black
Brown   Blue
Purple  Green
Red     Red
Teal    White
Yellow
```

* Without any option, `comm` gives 3 column output
    * lines unique to first file
    * lines unique to second file
    * lines common to both files

```bash
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
```

<br>

#### <a name="suppressing-columns"></a>Suppressing columns

* `-1` suppress lines unique to first file
* `-2` suppress lines unique to second file
* `-3` suppress lines common to both files

```bash
$ # suppressing column 3
$ comm -3 colors_1.txt colors_2.txt
        Black
Brown
        Green
Purple
Teal
        White
Yellow
```

* Combining options gives three distinct and useful constructs
* First, getting only common lines to both files

```bash
$ comm -12 colors_1.txt colors_2.txt
Blue
Red
```

* Second, lines unique to first file

```bash
$ comm -23 colors_1.txt colors_2.txt
Brown
Purple
Teal
Yellow
```

* And the third, lines unique to second file

```bash
$ comm -13 colors_1.txt colors_2.txt
Black
Green
White
```

* See also how the above three cases can be done [using grep alone](./gnu_grep.md#search-strings-from-file)
    * **Note** input files do not need to be sorted for `grep` solution

If different `sort` order than default is required, use `--nocheck-order` to ignore error message

```bash
$ comm -23 <(sort -n numbers.txt) <(sort -n nums.txt)
3
comm: file 1 is not in sorted order
20
53
101

$ comm --nocheck-order -23 <(sort -n numbers.txt) <(sort -n nums.txt)
3
20
53
101
```

<br>

#### <a name="files-with-duplicates"></a>Files with duplicates

* As many duplicate lines match in both files, they'll be considered as common
* Rest will be unique to respective files
* This is useful for cases like finding lines present in first but not in second taking in to consideration count of duplicates as well
    * This solution won't be possible with `grep`

```bash
$ paste list1 list2
a       a
a       b
a       c
b       c
b       d
c

$ comm list1 list2
                a
a
a
                b
b
                c
        c
        d

$ comm -23 list1 list2
a
a
b
```

<br>

#### <a name="further-reading-for-comm"></a>Further reading for comm

* `man comm` and `info comm` for more options and detailed documentation
* [comm Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/comm?sort=votes&pageSize=15)

<br>

## <a name="shuf"></a>shuf

```bash
$ shuf --version | head -n1
shuf (GNU coreutils) 8.25

$ man shuf
SHUF(1)                          User Commands                         SHUF(1)

NAME
       shuf - generate random permutations

SYNOPSIS
       shuf [OPTION]... [FILE]
       shuf -e [OPTION]... [ARG]...
       shuf -i LO-HI [OPTION]...

DESCRIPTION
       Write a random permutation of the input lines to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="random-lines"></a>Random lines

* Without repeating input lines

```bash
$ cat nums.txt
1
10
10
12
23
563

$ # duplicates can end up anywhere
$ # all lines are part of output
$ shuf nums.txt
10
23
1
10
563
12

$ # limit max number of output lines
$ shuf -n2 nums.txt
563
23
```

* Use `-o` option to specify output file name instead of displaying on stdout
* Helpful for inplace editing

```bash
$ shuf nums.txt -o nums.txt
$ cat nums.txt
10
12
23
10
563
1
```

* With repeated input lines

```bash
$ # -n3 for max 3 lines, -r allows input lines to be repeated
$ shuf -n3 -r nums.txt
1
1
563

$ seq 3 | shuf -n5 -r
2
1
2
1
2

$ # if a limit using -n is not specified, shuf will output lines indefinitely
```

* use `-e` option to specify multiple input lines from command line itself

```bash
$ shuf -e red blue green
green
blue
red

$ shuf -e 'hi there' 'hello world' foo bar
bar
hi there
foo
hello world

$ shuf -n2 -e 'hi there' 'hello world' foo bar
foo
hi there

$ shuf -r -n4 -e foo bar
foo
foo
bar
foo
```

<br>

#### <a name="random-integer-numbers"></a>Random integer numbers

* The `-i` option accepts integer range as input to be shuffled

```bash
$ shuf -i 3-8
3
7
6
4
8
5
```

* Combine with other options as needed

```bash
$ shuf -n3 -i 3-8
5
4
7

$ shuf -r -n4 -i 3-8
5
5
7
8

$ shuf -r -n5 -i 0-1
1
0
0
1
1
```

* Use [seq](./miscellaneous.md#seq) input if negative numbers, floating point, etc are needed

```bash
$ seq 2 -1 -2 | shuf
2
-1
-2
0
1

$ seq 0.3 0.1 0.7 | shuf -n3
0.4
0.5
0.7
```


<br>

#### <a name="further-reading-for-shuf"></a>Further reading for shuf

* `man shuf` and `info shuf` for more options and detailed documentation
* [Generate random numbers in specific range](https://unix.stackexchange.com/questions/140750/generate-random-numbers-in-specific-range)
* [Variable - randomly choose among three numbers](https://unix.stackexchange.com/questions/330689/variable-randomly-chosen-among-three-numbers-10-100-and-1000)
* Related to 'random' stuff:
    * [How to generate a random string?](https://unix.stackexchange.com/questions/230673/how-to-generate-a-random-string)
    * [How can I populate a file with random data?](https://unix.stackexchange.com/questions/33629/how-can-i-populate-a-file-with-random-data)
    * [Run commands at random](https://unix.stackexchange.com/questions/81566/run-commands-at-random)


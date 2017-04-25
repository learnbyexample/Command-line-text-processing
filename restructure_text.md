# <a name="restructure-text"></a>Restructure text

**Table of Contents**

* [paste](#paste)
    * [Concatenating files column wise](#concatenating-files-column-wise)
    * [Interleaving lines](#interleaving-lines)
    * [Lines to multiple columns](#lines-to-multiple-columns)
    * [Different delimiters between columns](#different-delimiters-between-columns)
    * [Multiple lines to single row](#multiple-lines-to-single-row)
    * [Further reading for paste](#further-reading-for-paste)
* [column](#column)
    * [Pretty printing tables](#pretty-printing-tables)
    * [Specifying different input delimiter](#specifying-different-input-delimiter)
    * [Further reading for column](#further-reading-for-column)
* [pr](#pr)
    * [Converting lines to columns](#converting-lines-to-columns)
    * [Changing PAGE_WIDTH](#changing-page_width)
    * [Combining multiple input files](#combining-multiple-input-files)
    * [Transposing a table](#transposing-a-table)
    * [Further reading for pr](#further-reading-for-pr)
* [fold](#fold)
    * [Examples](#examples)
    * [Further reading for fold](#further-reading-for-fold)

<br>

## <a name="paste"></a>paste

```bash
$ paste --version | head -n1
paste (GNU coreutils) 8.25

$ man paste 
PASTE(1)                         User Commands                        PASTE(1)

NAME
       paste - merge lines of files

SYNOPSIS
       paste [OPTION]... [FILE]...

DESCRIPTION
       Write  lines  consisting  of  the sequentially corresponding lines from
       each FILE, separated by TABs, to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="concatenating-files-column-wise"></a>Concatenating files column wise

* By default, `paste` adds a TAB between corresponding lines of input files

```bash
$ paste colors_1.txt colors_2.txt
Blue    Black
Brown   Blue
Purple  Green
Red     Red
Teal    White
```

* Specifying a different delimiter using `-d`
* The `<()` syntax is [Process Substitution](http://mywiki.wooledge.org/ProcessSubstitution)
    * to put it simply - allows output of command to be passed as input file to another command without needing to manually create a temporary file

```bash
$ paste -d, <(seq 5) <(seq 6 10)
1,6
2,7
3,8
4,9
5,10

$ # empty cells if number of lines is not same for all input files
$ # -d\| can also be used
$ paste -d'|' <(seq 3) <(seq 4 6) <(seq 7 10)
1|4|7
2|5|8
3|6|9
||10
```

<br>

#### <a name="interleaving-lines"></a>Interleaving lines

* Interleave lines by using newline as delimiter

```bash
$ paste -d'\n' <(seq 11 13) <(seq 101 103)
11
101
12
102
13
103
```

<br>

#### <a name="lines-to-multiple-columns"></a>Lines to multiple columns

* Number of `-` specified determines number of output columns
* Input lines can be passed only as stdin

```bash
$ # single column to two columns
$ seq 10 | paste -d, - -
1,2
3,4
5,6
7,8
9,10

$ # single column to five columns
$ seq 10 | paste -d: - - - - -
1:2:3:4:5
6:7:8:9:10

$ # input redirection for file input
$ paste -d, - - < colors_1.txt 
Blue,Brown
Purple,Red
Teal,
```

* Use `printf` trick if number of columns to specify is too large

```bash
$ # prompt at end of line not shown for simplicity
$ printf -- "- %.s" {1..5}
- - - - - 

$ seq 10 | paste -d, $(printf -- "- %.s" {1..5})
1,2,3,4,5
6,7,8,9,10
```

<br>

#### <a name="different-delimiters-between-columns"></a>Different delimiters between columns

* For more than 2 columns, different delimiter character can be specified - passed as list to `-d` option

```bash
$ # , is used between 1st and 2nd column
$ # - is used between 2nd and 3rd column
$ paste -d',-' <(seq 3) <(seq 4 6) <(seq 7 9)
1,4-7
2,5-8
3,6-9

$ # re-use list from beginning if not specified for all columns
$ paste -d',-' <(seq 3) <(seq 4 6) <(seq 7 9) <(seq 10 12)
1,4-7,10
2,5-8,11
3,6-9,12
$ # another example
$ seq 10 | paste -d':,' - - - - -
1:2,3:4,5
6:7,8:9,10

$ # so, with single delimiter, it is just re-used for all columns
$ paste -d, <(seq 3) <(seq 4 6) <(seq 7 9) <(seq 10 12)
1,4,7,10
2,5,8,11
3,6,9,12
```

* combination of `-d` and `/dev/null` (empty file) can give multi-character separation between columns
* If this is too confusing to use, consider [pr](#pr) instead

```bash
$ paste -d' : ' <(seq 3) /dev/null /dev/null <(seq 4 6) /dev/null /dev/null <(seq 7 9)
1 : 4 : 7
2 : 5 : 8
3 : 6 : 9

$ # or just use pr instead
$ pr -mts' : ' <(seq 3) <(seq 4 6) <(seq 7 9)
1 : 4 : 7
2 : 5 : 8
3 : 6 : 9

$ # but paste would allow different delimiters ;)
$ paste -d' :  - ' <(seq 3) /dev/null /dev/null <(seq 4 6) /dev/null /dev/null <(seq 7 9)
1 : 4 - 7
2 : 5 - 8
3 : 6 - 9

$ # pr would need two invocations
$ pr -mts' : ' <(seq 3) <(seq 4 6) | pr -mts' - ' - <(seq 7 9)
1 : 4 - 7
2 : 5 - 8
3 : 6 - 9
```

* example to show using empty file instead of `/dev/null`

```bash
$ # assuming file named e doesn't exist
$ touch e
$ # or use this, will empty contents even if file named e already exists :P
$ > e

$ paste -d' :  - ' <(seq 3) e e <(seq 4 6) e e <(seq 7 9)
1 : 4 - 7
2 : 5 - 8
3 : 6 - 9
```

<br>

#### <a name="multiple-lines-to-single-row"></a>Multiple lines to single row

```bash
$ paste -sd, colors_1.txt
Blue,Brown,Purple,Red,Teal

$ # multiple files each gets a row
$ paste -sd: colors_1.txt colors_2.txt 
Blue:Brown:Purple:Red:Teal
Black:Blue:Green:Red:White

$ # multiple input files need not have same number of lines
$ paste -sd, <(seq 3) <(seq 5 9)
1,2,3
5,6,7,8,9
```

* Often used to serialize multiple line output from another command

```bash
$ sort -u colors_1.txt colors_2.txt | paste -sd,
Black,Blue,Brown,Green,Purple,Red,Teal,White
```

* For multiple character delimiter, post-process if separator is unique or use another tool like `perl`

```bash
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

$ # post-process
$ seq 10 | paste -sd, | sed 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10

$ # using perl alone
$ seq 10 | perl -pe 's/\n/ : / if(!eof)'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

<br>

#### <a name="further-reading-for-paste"></a>Further reading for paste

* `man paste` and `info paste` for more options and detailed documentation
* [paste Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/paste?sort=votes&pageSize=15)

<br>

## <a name="column"></a>column

```bash
COLUMN(1)                 BSD General Commands Manual                COLUMN(1)

NAME
     column — columnate lists

SYNOPSIS
     column [-entx] [-c columns] [-s sep] [file ...]

DESCRIPTION
     The column utility formats its input into multiple columns.  Rows are
     filled before columns.  Input is taken from file operands, or, by
     default, from the standard input.  Empty lines are ignored unless the -e
     option is used.
...
```

<br>

#### <a name="pretty-printing-tables"></a>Pretty printing tables

* by default whitespace is input delimiter

```bash
$ cat dishes.txt 
North alootikki baati khichdi makkiroti poha
South appam bisibelebath dosa koottu sevai
West dhokla khakhra modak shiro vadapav
East handoguri litti momo rosgulla shondesh

$ column -t dishes.txt 
North  alootikki  baati         khichdi  makkiroti  poha
South  appam      bisibelebath  dosa     koottu     sevai
West   dhokla     khakhra       modak    shiro      vadapav
East   handoguri  litti         momo     rosgulla   shondesh
```

* often useful to get neatly aligned columns from output of another command

```bash
$ paste fruits.txt price.txt
Fruits  Price
apple   182
guava   90
watermelon      35
banana  72
pomegranate     280

$ paste fruits.txt price.txt | column -t
Fruits       Price
apple        182
guava        90
watermelon   35
banana       72
pomegranate  280
```

<br>

#### <a name="specifying-different-input-delimiter"></a>Specifying different input delimiter

* Use `-s` to specify input delimiter
* Use `-n` to prevent merging empty cells
    * From `man column` "This option is a Debian GNU/Linux extension"

```bash
$ paste -d, <(seq 3) <(seq 5 9) <(seq 11 13)
1,5,11
2,6,12
3,7,13
,8,
,9,

$ paste -d, <(seq 3) <(seq 5 9) <(seq 11 13) | column -s, -t
1  5  11
2  6  12
3  7  13
8
9

$ paste -d, <(seq 3) <(seq 5 9) <(seq 11 13) | column -s, -nt
1  5  11
2  6  12
3  7  13
   8  
   9  
```

<br>

#### <a name="further-reading-for-column"></a>Further reading for column

* `man column` for more options and detailed documentation
* [column Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/columns?sort=votes&pageSize=15)
* More examples [here](http://www.commandlinefu.com/commands/using/column/sort-by-votes)

<br>

## <a name="pr"></a>pr

```bash
$ pr --version | head -n1
pr (GNU coreutils) 8.25

$ man pr
PR(1)                            User Commands                           PR(1)

NAME
       pr - convert text files for printing

SYNOPSIS
       pr [OPTION]... [FILE]...

DESCRIPTION
       Paginate or columnate FILE(s) for printing.

       With no FILE, or when FILE is -, read standard input.
...
```

* `Paginate` is not covered, examples related only to `columnate`
* For example, default invocation on a file would add a header, etc

```bash
$ # truncated output shown
$ pr fruits.txt 


2017-04-21 17:49                    fruits.txt                    Page 1


Fruits
apple
guava
watermelon
banana
pomegranate

```

* Following sections will use `-t` to omit page headers and trailers

<br>

#### <a name="converting-lines-to-columns"></a>Converting lines to columns

* With [paste](#lines-to-multiple-columns), changing input file rows to column(s) is possible only with consecutive lines
* `pr` can do that as well as split entire file itself according to number of columns needed
* And `-s` option in `pr` allows multi-character output delimiter
* As usual, examples to better show the functionalities

```bash
$ # note how the input got split into two and resulting splits joined by ,
$ seq 6 | pr -2ts,
1,4
2,5
3,6

$ # note how two consecutive lines gets joined by ,
$ seq 6 | paste -d, - -
1,2
3,4
5,6
```

* Default **PAGE_WIDTH** is 72 characters, so each column gets 72 divided by number of columns unless `-s` is used

```bash
$ # 3 columns, so each column width is 24 characters
$ seq 9 | pr -3t
1                       4                       7
2                       5                       8
3                       6                       9

$ # using -s, desired delimiter can be specified
$ seq 9 | pr -3ts' '
1 4 7
2 5 8
3 6 9

$ seq 9 | pr -3ts' : '
1 : 4 : 7
2 : 5 : 8
3 : 6 : 9

$ # default is TAB when using -s option with no arguments
$ seq 9 | pr -3ts
1       4       7
2       5       8
3       6       9
```

* Using `-a` to change consecutive rows, similar to `paste`

```bash
$ seq 8 | pr -4ats:
1:2:3:4
5:6:7:8

$ # no output delimiter for empty cells
$ seq 22 | pr -5ats,
1,2,3,4,5
6,7,8,9,10
11,12,13,14,15
16,17,18,19,20
21,22

$ # note output delimiter even for empty cells
$ seq 22 | paste -d, - - - - -
1,2,3,4,5
6,7,8,9,10
11,12,13,14,15
16,17,18,19,20
21,22,,,
```

<br>

#### <a name="changing-page_width"></a>Changing PAGE_WIDTH

* The default PAGE_WIDTH is 72
* The formula `(col-1)*len(delimiter) + col` seems to work in determining minimum PAGE_WIDTH required for multiple column output
    * `col` is number of columns required

```bash
$ # (36-1)*1 + 36 = 71, so within PAGE_WIDTH limit
$ seq 74 | pr -36ats,
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36
37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72
73,74
$ # (37-1)*1 + 37 = 73, more than default PAGE_WIDTH limit
$ seq 74 | pr -37ats,
pr: page width too narrow
```

* Use `-w` to specify a different PAGE_WIDTH
* The `-J` option turns off truncation

```bash
$ # (37-1)*1 + 37 = 73
$ seq 74 | pr -J -w73 -37ats,
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37
38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74

$ # (3-1)*4 + 3 = 11
$ seq 6 | pr -J -w10 -3ats'::::'
pr: page width too narrow
$ seq 6 | pr -J -w11 -3ats'::::'
1::::2::::3
4::::5::::6

$ # if calculating is difficult, simply use a large number
$ seq 6 | pr -J -w500 -3ats'::::'
1::::2::::3
4::::5::::6
```

<br>

#### <a name="combining-multiple-input-files"></a>Combining multiple input files

* Use `-m` option to combine multiple files in parallel, similar to `paste`

```bash
$ # 2 columns, so each column width is 36 characters
$ pr -mt fruits.txt price.txt
Fruits                              Price
apple                               182
guava                               90
watermelon                          35
banana                              72
pomegranate                         280

$ # default is TAB when using -s option with no arguments
$ pr -mts <(seq 3) <(seq 4 6) <(seq 7 10)
1       4       7
2       5       8
3       6       9
                10

$ # double TAB as separator
$ # shell expands $'\t\t' before command is executed
$ pr -mts$'\t\t' colors_1.txt colors_2.txt
Blue            Black
Brown           Blue
Purple          Green
Red             Red
Teal            White
```

* For interleaving, specify newline as separator

```bash
$ pr -mts$'\n' fruits.txt price.txt
Fruits
Price
apple
182
guava
90
watermelon
35
banana
72
pomegranate
280
```

<br>

#### <a name="transposing-a-table"></a>Transposing a table

```bash
$ # delimiter is single character, so easy to use tr to change it to newline
$ cat dishes.txt 
North alootikki baati khichdi makkiroti poha
South appam bisibelebath dosa koottu sevai
West dhokla khakhra modak shiro vadapav
East handoguri litti momo rosgulla shondesh

$ # 4 columns, so each column width is 18 characters
$ # $(wc -l < dishes.txt) gives number of columns required
$ tr ' ' '\n' < dishes.txt | pr -$(wc -l < dishes.txt)t
North             South             West              East
alootikki         appam             dhokla            handoguri
baati             bisibelebath      khakhra           litti
khichdi           dosa              modak             momo
makkiroti         koottu            shiro             rosgulla
poha              sevai             vadapav           shondesh
```

* Pipe the output to `column` if spacing is too much

```bash
$ tr ' ' '\n' < dishes.txt | pr -$(wc -l < dishes.txt)t | column -t
North      South         West     East
alootikki  appam         dhokla   handoguri
baati      bisibelebath  khakhra  litti
khichdi    dosa          modak    momo
makkiroti  koottu        shiro    rosgulla
poha       sevai         vadapav  shondesh
```

<br>

#### <a name="further-reading-for-pr"></a>Further reading for pr

* `man pr` and `info pr` for more options and detailed documentation
* More examples [here](http://docstore.mik.ua/orelly/unix3/upt/ch21_15.htm)

<br>

## <a name="fold"></a>fold

```bash
$ fold --version | head -n1
fold (GNU coreutils) 8.25

$ man fold
FOLD(1)                          User Commands                         FOLD(1)

NAME
       fold - wrap each input line to fit in specified width

SYNOPSIS
       fold [OPTION]... [FILE]...

DESCRIPTION
       Wrap input lines in each FILE, writing to standard output.

       With no FILE, or when FILE is -, read standard input.
...
```

<br>

#### <a name="examples"></a>Examples

```bash
$ nl story.txt
     1	The princess of a far away land fought bravely to rescue a travelling group from bandits. And the happy story ends here. Have a nice day.
     2	Still here? okay, read on: The prince of Happalakkahuhu wished he could be as brave as his sister and vowed to train harder

$ # default folding width is 80
$ fold story.txt 
The princess of a far away land fought bravely to rescue a travelling group from
 bandits. And the happy story ends here. Have a nice day.
Still here? okay, read on: The prince of Happalakkahuhu wished he could be as br
ave as his sister and vowed to train harder

$ fold story.txt | nl
     1	The princess of a far away land fought bravely to rescue a travelling group from
     2	 bandits. And the happy story ends here. Have a nice day.
     3	Still here? okay, read on: The prince of Happalakkahuhu wished he could be as br
     4	ave as his sister and vowed to train harder
```

* `-s` option breaks at spaces to avoid word splitting

```bash
$ fold -s story.txt 
The princess of a far away land fought bravely to rescue a travelling group 
from bandits. And the happy story ends here. Have a nice day.
Still here? okay, read on: The prince of Happalakkahuhu wished he could be as 
brave as his sister and vowed to train harder
```

* Use `-w` to change default width

```bash
$ fold -s -w60 story.txt 
The princess of a far away land fought bravely to rescue a 
travelling group from bandits. And the happy story ends 
here. Have a nice day.
Still here? okay, read on: The prince of Happalakkahuhu 
wished he could be as brave as his sister and vowed to 
train harder
```

<br>

#### <a name="further-reading-for-fold"></a>Further reading for fold

* `man fold` and `info fold` for more options and detailed documentation


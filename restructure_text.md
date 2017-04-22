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

>convert text files for printing

```bash
$ pr sample.txt 


2016-05-29 11:00                    sample.txt                    Page 1


This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.
Adding a line of text at end of file
```

* Options include converting text files for printing with header, footer, page numbers, double space a file, combine multiple files column wise, etc
* More examples [here](http://docstore.mik.ua/orelly/unix3/upt/ch21_15.htm)

```bash
$ # single column to multiple column, split vertically
$ # for example, in command below, output of seq is split into two
$ seq 5 | pr -2t
1				    4
2				    5
3

$ # different output delimiter can be used by passing string to -s option
$ seq 5 | pr -2ts' '
1 4
2 5
3

$ seq 15 | pr -5ts,
1,4,7,10,13
2,5,8,11,14
3,6,9,12,15
```

* Use `-a` option to split across

```bash
$ seq 5 | pr -2ats' : '
1 : 2
3 : 4
5

$ seq 15 | pr -5ats,
1,2,3,4,5
6,7,8,9,10
11,12,13,14,15

$ # use $ to expand characters denoted by escape characters like \t for tab
$ seq 5 | pr -3ts$'\t'
1	3	5
2	4

$ # or leave the argument to -s empty as tab is default
$ seq 5 | pr -3ts
1	3	5
2	4
```

* The default PAGE_WIDTH is 72
* The formula `(col-1)*len(delimiter) + col` seems to work in determining minimum PAGE_WIDTH required for multiple column output
* The `-J` option will help in turning off line truncation

```bash
$ seq 74 | pr -36ats,
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36
37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72
73,74
$ seq 74 | pr -37ats,
pr: page width too narrow

$ # (37-1)*1 + 37 = 73
$ seq 74 | pr -Jw 73 -37ats,
1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37
38,39,40,41,42,43,44,45,46,47,48,49,50,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,68,69,70,71,72,73,74

$ # (3-1)*4 + 3 = 11
$ seq 6 | pr -Jw 10 -3ats'::::'
pr: page width too narrow
$ seq 6 | pr -Jw 11 -3ats'::::'
1::::2::::3
4::::5::::6
```

* Use `-m` option to combine multiple files in parallel

```bash
$ pr -mts', ' <(seq 3) <(seq 4 6) <(seq 7 9)
1, 4, 7
2, 5, 8
3, 6, 9
```

<br>

We can use a combination of different commands for complicated operations. For example, transposing a table

```bash
$ tr ' ' '\n' < dishes.txt | pr -$(wc -l < dishes.txt)t
North               South               West                East
alootikki           appam               dhokla              handoguri
baati               bisibelebath        khakhra             litti
khichdi             dosa                modak               momo
makkiroti           koottu              shiro               rosgulla
poha                sevai               vadapav             shondesh
```

Notice how `pr` neatly arranges the columns. If spacing is too much, we can use `column`

```bash
$ tr ' ' '\n' < dishes.txt | pr -$(wc -l < dishes.txt)ts | column -t
North      South         West     East
alootikki  appam         dhokla   handoguri
baati      bisibelebath  khakhra  litti
khichdi    dosa          modak    momo
makkiroti  koottu        shiro    rosgulla
poha       sevai         vadapav  shondesh
```

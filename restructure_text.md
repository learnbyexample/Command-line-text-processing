# <a name="restructure-text"></a>Restructure text

**Table of Contents**

* [paste](#paste)
* [column](#column)
* [pr](#pr)

<br>

## <a name="paste"></a>paste

>merge lines of files

**Examples**

* `paste list1.txt list2.txt list3.txt > combined_list.txt` combines the three files column-wise into single file, the entries separated by TAB character
* `paste -d':' list1.txt list2.txt list3.txt > combined_list.txt` the entries are separated by : character instead of TAB
    * See [pr](#pr) command for multiple character delimiter
* [paste Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/paste?sort=votes&pageSize=15)

```bash
$ # joining multiple files
$ paste -d, <(seq 5) <(seq 6 10)
1,6
2,7
3,8
4,9
5,10

$ paste -d, <(seq 3) <(seq 4 6) <(seq 7 10)
1,4,7
2,5,8
3,6,9
,,10
```

* Single column to multiple columns

```bash
$ seq 5 | paste - -
1	2
3	4
5	

$ # specifying different output delimiter, default is tab
$ seq 5 | paste -d, - -
1,2
3,4
5,

$ # if number of columns to specify is large, use the printf trick
$ seq 5 | paste $(printf -- "- %.s" {1..3})
1	2	3
4	5	
```

* Combine all lines to single line

```bash
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

$ # for multiple character delimiter, perl can be used
$ seq 10 | perl -pe 's/\n/ : / if(!eof)'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

<br>

## <a name="column"></a>column

>columnate lists

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

# <a name="sorting-stuff"></a>Sorting stuff

**Table of Contents**

* [sort](#sort)
* [uniq](#uniq)
* [comm](#comm)

<br>

## <a name="sort"></a>sort

>sort lines of text files

As the name implies, this command is used to sort files. How about alphabetic sort and numeric sort? Possible. How about sorting a particular column? Possible. Prioritized multiple sorting order? Possible. Randomize? Unique? Just about any sorting need is catered by this powerful command

**Options**

* `-R` random sort
* `-r` reverse the sort order
* `-o` redirect sorted result to specified filename, very useful to sort a file inplace
* `-n` sort numerically
* `-V` version sort, aware of numbers within text
* `-h` sort human readable numbers like 4K, 3M, etc
* `-k` sort via key
* `-u` sort uniquely
* `-b` ignore leading white-spaces of a line while sorting
* `-t` use SEP instead of non-blank to blank transition

**Examples**

* `sort dir_list.txt` display sorted file on standard output
* `sort -bn numbers.txt -o numbers.txt` sort numbers.txt numerically (ignoring leading white-spaces) and overwrite the file with sorted output
* `sort -R crypto_keys.txt -o crypto_keys_random.txt` sort randomly and write to new file
	* `shuf crypto_keys.txt -o crypto_keys_random.txt` can also be used
* `du -sh * | sort -h` sort file/directory sizes in current directory in human readable format

<br>

```bash
$ cat ip.txt 
6.2  : 897 : bar
3.1  : 32  : foo
2.3  : 012 : bar
1.2  : 123 : xyz

$ # -k3,3 means from 3rd column onwards to 3rd column
$ # for ex: to sort from 2nd column till end, use -k2
$ sort -t: -k3,3 ip.txt 
2.3  : 012 : bar
6.2  : 897 : bar
3.1  : 32  : foo
1.2  : 123 : xyz

$ # -n option for numeric sort, check out what happens when -n is not used
$ sort -t: -k2,2n ip.txt 
2.3  : 012 : bar
3.1  : 32  : foo
1.2  : 123 : xyz
6.2  : 897 : bar

$ # more than one rule can be specified to resolve same values
$ sort -t: -k3,3 -k1,1rn ip.txt 
6.2  : 897 : bar
2.3  : 012 : bar
3.1  : 32  : foo
1.2  : 123 : xyz
```

**Further Reading**

* [sort like a master](http://www.skorks.com/2010/05/sort-files-like-a-master-with-the-linux-sort-command-bash/)
* [sort Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/sort?sort=votes&pageSize=15)
* [sort on multiple columns using -k option](https://unix.stackexchange.com/questions/249452/unix-multiple-column-sort-issue)

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


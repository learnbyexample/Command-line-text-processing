# <a name="sed"></a>sed

**Table of Contents**

* [Simple search and replace](#simple-search-and-replace)
    * [editing stdin](#editing-stdin)
    * [editing file input](#editing-file-input)
* [Inplace file editing](#inplace-file-editing)
    * [With backup](#with-backup)
    * [Without backup](#without-backup)
    * [Multiple files](#multiple-files)
* [Line filtering options](#line-filtering-options)
    * [Print command](#print-command)
    * [Delete command](#delete-command)
    * [Negating REGEXP address](#negating-regexp-address)
    * [Combining multiple REGEXP](#combining-multiple-regexp)
    * [Filtering by line number](#filtering-by-line-number)
    * [Print only line number](#print-only-line-number)
    * [Address range](#address-range)
    * [Relative addressing](#relative-addressing)
* [Using different delimiter for REGEXP](#using-different-delimiter-for-regexp)
* [Regular Expressions](#regular-expressions)
    * [Line Anchors](#line-anchors)
    * [Word Anchors](#word-anchors)
    * [Matching the meta characters](#matching-the-meta-characters)

<br>

```bash
$ sed --version | head -n1
sed (GNU sed) 4.2.2

$ man sed
SED(1)                           User Commands                          SED(1)

NAME
       sed - stream editor for filtering and transforming text

SYNOPSIS
       sed [OPTION]... {script-only-if-no-other-script} [input-file]...

DESCRIPTION
       Sed  is a stream editor.  A stream editor is used to perform basic text
       transformations on an input stream (a file or input from  a  pipeline).
       While  in  some  ways similar to an editor which permits scripted edits
       (such as ed), sed works by making only one pass over the input(s),  and
       is consequently more efficient.  But it is sed's ability to filter text
       in a pipeline which particularly distinguishes it from other  types  of
       editors.
...
```

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

Detailed examples for **substitute** command will be convered in later section, syntax is

```
s/REGEXP/REPLACEMENT/FLAGS
```

<br>

#### <a name="editing-stdin"></a>editing stdin

```bash
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

$ # change only first ',' to ' : '
$ seq 10 | paste -sd, | sed 's/,/ : /'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : ' by using 'g' FLAG
$ seq 10 | paste -sd, | sed 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

<br>

#### <a name="editing-file-input"></a>editing file input

```bash
$ cat greeting.txt 
Hi, how are you?
What are you upto these days?
Hope you are atleast keeping in touch with Prof.
As usual, am upto no good :P

$ # change all 'atleast' to 'at least'
$ sed 's/atleast/at least/g' greeting.txt 
Hi, how are you?
What are you upto these days?
Hope you are at least keeping in touch with Prof.
As usual, am upto no good :P
```

<br>

## <a name="inplace-file-editing"></a>Inplace file editing

* In previous section, the output from `sed` was displayed on stdout
* To save stdout to another file, shell redirection can be used
    * For ex: `sed 's/atleast/at least/g' greeting.txt > out.txt`
* However, to write the changes back to original file, use `-i` option

**Note**

Refer to `man sed` for details of how to use the `-i` option. It varies with different `sed` implementations. As mentioned at start of this chapter, `sed (GNU sed) 4.2.2` is being used here

<br>

#### <a name="with-backup"></a>With backup

* When extension is given, the original input file is preserved with name changed according to extension provided

```bash
$ # '.bkp' is extension provided
$ sed -i.bkp 's/atleast/at least/g' greeting.txt

$ # original file gets preserved in 'greeting.txt.bkp'
$ cat greeting.txt.bkp 
Hi, how are you?
What are you upto these days?
Hope you are atleast keeping in touch with Prof.
As usual, am upto no good :P

$ # output from sed gets written to 'greeting.txt'
$ cat greeting.txt
Hi, how are you?
What are you upto these days?
Hope you are at least keeping in touch with Prof.
As usual, am upto no good :P
```

<br>

#### <a name="without-backup"></a>Without backup

* Use this option with caution, changes made cannot be undone

```bash
$ # note that 'uptown' would also get changed to 'up town'
$ # Use regular expressions (covered later) to specify exact string to be replaced
$ sed -i 's/upto/up to/g' greeting.txt

$ # note, 'atleast' was already changed to 'at least' in previous example
$ cat greeting.txt
Hi, how are you?
What are you up to these days?
Hope you are at least keeping in touch with Prof.
As usual, am up to no good :P
```

<br>

#### <a name="multiple-files"></a>Multiple files

* Multiple input files are treated individually and changes are written back to respective files

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ # -i can be used with or without backup
$ sed -i 's/3/three/' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

<br>

## <a name="line-filtering-options"></a>Line filtering options

* By default, `sed` acts on entire file. Often, one needs to print or change only specific lines based on text search, line numbers, lines between two patterns, etc
* This filtering is much like using `grep`, `head` and `tail` commands in many ways and there are even more features
    * Use `sed` for inplace editing, the filtered lines to be transformed etc. Not as substitute for `grep`, `head` and `tail`

<br>

#### <a name="print-command"></a>Print command

* It is usually used in conjunction with `-n` option
* By default, `sed` prints every input line, including any changes like substitution
* Using `-n` option and `p` command together, only specific lines needed can be filtered
* Examples below use the `/REGEXP/` addressing, other forms will be seen in sections to follow

```bash
$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # all lines containing the string 'are'
$ # same as: grep 'are' poem.txt 
$ sed -n '/are/p' poem.txt 
Roses are red,
Violets are blue,
And so are you.

$ # all lines containing the string 'so are'
$ # same as: grep 'so are' poem.txt 
$ sed -n '/so are/p' poem.txt 
And so are you.
```

* Using print and substitution together

```bash
$ # print only lines on which substitution happens
$ sed -n 's/are/ARE/p' poem.txt 
Roses ARE red,
Violets ARE blue,
And so ARE you.

$ # only lines containing 'are' as well as 'so'
$ sed -n '/are/ s/so/SO/p' poem.txt 
And SO are you.
```

* Duplicating every input line

```bash
$ # note, -n is not used and no filtering applied
$ seq 3 | sed 'p'
1
1
2
2
3
3
```

<br>

#### <a name="delete-command"></a>Delete command

* By default, `sed` prints every input line, including any changes like substitution
* Using the `d` command, those specific lines will NOT be printed

```bash
$ # same as: grep -v 'are' poem.txt 
$ sed '/are/d' poem.txt 
Sugar is sweet,

$ # same as: seq 5 | grep -v '3'
$ seq 5 | sed '/3/d'
1
2
4
5
```

* Regular expressions will be covered later, but modifier `I` allows to filter lines in case-insensitive way

```bash
$ # /rose/I means match the string 'rose' irrespective of case
$ sed '/rose/Id' poem.txt 
Violets are blue,
Sugar is sweet,
And so are you.
```

<br>

#### <a name="negating-regexp-address"></a>Negating REGEXP address

* Use `!` to invert the specified address

```bash
$ # same as: sed -n '/so are/p' poem.txt
$ sed '/so are/!d' poem.txt
And so are you.

$ # same as: sed '/are/d' poem.txt
$ sed -n '/are/!p' poem.txt 
Sugar is sweet,
```

<br>

#### <a name="combining-multiple-regexp"></a>Combining multiple REGEXP

* Use `-e` option to logically OR different REGEXPs

```bash
$ # same as: grep -e 'blue' -e 'you' poem.txt
$ sed -n -e '/blue/p' -e '/you/p' poem.txt 
Violets are blue,
And so are you.
```

* Use `{}` command grouping for logical AND

```bash
$ # same as: grep 'are' poem.txt | grep 'And'
$ sed -n '/are/ {/And/p}' poem.txt 
And so are you.

$ # same as: grep 'are' poem.txt | grep -v 'so'
$ sed -n '/are/ {/so/!p}' poem.txt 
Roses are red,
Violets are blue,

$ # space between /REGEXP/ and {} is optional
$ # same as: grep -v 'red' poem.txt | grep -v 'blue'
$ sed -n '/red/!{/blue/!p}' poem.txt 
Sugar is sweet,
And so are you.
$ # many ways to do it, use whatever feels easier to construct
$ # sed -e '/red/d' -e '/blue/d' poem.txt 
$ # grep -v -e 'red' -e 'blue' poem.txt
```

<br>

#### <a name="filtering-by-line-number"></a>Filtering by line number

* Exact line number can be specified to be acted upon
* As a special case, `$` indicates last line of file

```bash
$ # here, 2 represents the address for print command, similar to /REGEXP/p
$ # same as: head -n2 poem.txt | tail -n1
$ sed -n '2p' poem.txt 
Violets are blue,

$ # print 2nd and 4th line
$ # for `p`, `d`, `s` etc multiple commands can be specified separated by ;
$ sed -n '2p; 4p' poem.txt 
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
$ sed -n '$p' poem.txt 
And so are you.

$ # delete only 3rd line
$ sed '3d' poem.txt 
Roses are red,
Violets are blue,
And so are you.
```

* For large input files, combine `p` with `q` for speedy exit
* `sed` would immediately quit without processing further input lines when `q` is used

```bash
$ seq 3542 4623452 | sed -n '2452{p;q}'
5993

$ seq 3542 4623452 | sed -n '250p; 2452{p;q}'
3791
5993

$ # here is a sample time comparison
$ time seq 3542 4623452 | sed -n '2452{p;q}' > /dev/null 

real    0m0.003s
user    0m0.000s
sys     0m0.000s
$ time seq 3542 4623452 | sed -n '2452p' > /dev/null 

real    0m0.334s
user    0m0.396s
sys     0m0.024s
```

* mimicking `head` command using `q`

```bash
$ # same as: seq 23 45 | head -n5
$ # remember that printing is default action if -n is not used
$ seq 23 45 | sed '5q'
23
24
25
26
27
```

<br>

#### <a name="print-only-line-number"></a>Print only line number

```bash
$ # gives both line number and matching line
$ grep -n 'blue' poem.txt 
2:Violets are blue,

$ # gives only line number of matching line
$ sed -n '/blue/=' poem.txt 
2

$ sed -n '/are/=' poem.txt 
1
2
4
```

* If needed, matching line can also be printed. But there will be newline separation

```bash
$ sed -n '/blue/{=;p}' poem.txt 
2
Violets are blue,

$ # or
$ sed -n '/blue/{p;=}' poem.txt 
Violets are blue,
2
```

<br>

#### <a name="address-range"></a>Address range

* So far, we've seen how to filter specific line based on REGEXP and line numbers
* `sed` also allows to combine them to enable selecting a range of lines
* Consider the sample input file for this section

```bash
$ cat sample.txt 
Hello World!

Good day
How do you do?

Just do it
Believe it!

Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* Range defined by start and end REGEXP
* Other cases like getting lines without the line matching start and/or end, unbalanced start/end, when end REGEXP doesn't match, etc will be covered separately later

```bash
$ sed -n '/is/,/like/p' sample.txt 
Today is sunny
Not a bit funny
No doubt you like it too

$ sed -n '/just/I,/believe/Ip' sample.txt 
Just do it
Believe it!

$ # the second REGEXP will always be checked after the line matching first address
$ sed -n '/No/,/No/p' sample.txt 
Not a bit funny
No doubt you like it too

$ # all the matching ranges will be printed
$ sed -n '/you/,/do/p' sample.txt 
How do you do?

Just do it
No doubt you like it too

Much ado about nothing
```

* Range defined by start and end line numbers

```bash
$ sed -n '3,7p' sample.txt 
Good day
How do you do?

Just do it
Believe it!

$ sed -n '13,$p' sample.txt 
Much ado about nothing
He he he

$ sed '2,13d' sample.txt 
Hello World!
He he he
```

* Range defined by mix of line number and REGEXP

```bash
$ sed -n '3,/do/p' sample.txt 
Good day
How do you do?

$ sed -n '/Today/,$p' sample.txt 
Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* Negating address range, just add `!` to end of address range

```bash
$ # same as: seq 10 | sed '3,7d'
$ seq 10 | sed -n '3,7!p'
1
2
8
9
10

$ # same as: sed '/Today/,$d' sample.txt
$ sed -n '/Today/,$!p' sample.txt
Hello World!

Good day
How do you do?

Just do it
Believe it!

```

<br>

#### <a name="relative-addressing"></a>Relative addressing

* Prefixing `+` to a number for second address gives relative filtering

```bash
$ # line matching 'is' and 2 lines after
$ sed -n '/is/,+2p' sample.txt 
Today is sunny
Not a bit funny
No doubt you like it too

$ # note that all matching ranges will be filtered
$ sed -n '/do/,+2p' sample.txt 
How do you do?

Just do it
No doubt you like it too

Much ado about nothing
```

* The first address could be number too
* Useful when using shell variables (covered later) instead of constant

```bash
$ sed -n '3,+4p' sample.txt 
Good day
How do you do?

Just do it
Believe it!
```

* Another relative format is `i~j` which acts on ith line and i+j, i+2j, i+3j, etc
    * `1~2` means 1st, 3rd, 5th, 7th, etc (i.e odd numbered lines)
    * `5~3` means 5th, 8th, 11th, etc 

```bash
$ # match odd numbered lines
$ # for even, use 2~2
$ seq 10 | sed -n '1~2p'
1
3
5
7
9

$ # match line numbers: 2, 2+2*2, 2+3*2, etc
$ seq 10 | sed -n '2~4p'
2
6
10
```

* If `~j` is specified after `,` then meaning changes completely
* After the matching line based on number or REGEXP of start address, the closest line number multiple of `j` will mark end address

```bash
$ # 2nd line is start address
$ # closest multiple of 4 is 4th line
$ seq 10 | sed -n '2,~4p'
2
3
4
$ # closest multiple of 4 is 8th line
$ seq 10 | sed -n '5,~4p'
5
6
7
8

$ # line matching on `Just` is 6th line, so ending is 10th line
$ sed -n '/Just/,~5p' sample.txt 
Just do it
Believe it!

Today is sunny
Not a bit funny
```

<br>

## <a name="using-different-delimiter-for-regexp"></a>Using different delimiter for REGEXP

* `/` is idiomatically used as the REGEXP delimiter
* But any character other than `\` and newline character can be used instead
* This helps to avoid/reduce use of `\`

```bash
$ # instead of this
$ echo '/home/learnbyexample/reports' | sed 's/\/home\/learnbyexample\//~\//'
~/reports

$ # use a different delimiter
$ echo '/home/learnbyexample/reports' | sed 's#/home/learnbyexample/#~/#'
~/reports
```

* For REGEXP used in address matching, syntax is a bit different `\<char>REGEXP<char>`

```bash
$ printf '/foo/bar\n/food/good\n'
/foo/bar
/food/good

$ printf '/foo/bar\n/food/good\n' | sed -n '\;/foo/;p'
/foo/bar
```

<br>

## <a name="regular-expressions"></a>Regular Expressions

* By default, `sed` treats REGEXP as BRE (Basic Regular Expression)
* The `-E` option enables ERE (Extended Regular Expression) which in GNU sed's case only differs in how meta characters are used, no difference in functionalities
    * Initially GNU sed only had `-r` option to enable ERE and `man sed` doesn't even mention `-E`
    * Other `sed` versions use `-E` and `grep` uses `-E` as well. So `-r` won't be used in examples in this tutorial

<br>

#### <a name="line-anchors"></a>Line Anchors

* Often, search must match from beginning of line or towards end of line
* For example, an integer variable declaration in `C` will start with optional white-space, the keyword `int`, white-space and then variable(s)
    * This way one can avoid matching declarations inside single line comments as well
* Similarly, one might want to match a variable at end of statement

Consider the input file and sample substitution without using any anchoring

```bash
$ cat anchors.txt 
cat and dog
too many cats around here
to concatenate, use the cmd cat
catapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety muscat

$ # without anchors, substitution will replace whereever the string is found
$ sed 's/cat/XXX/g' anchors.txt 
XXX and dog
too many XXXs around here
to conXXXenate, use the cmd XXX
XXXapults laid waste to the village
just sXXX and quit bothering me
that is quite a fabriXXXed tale
try the grape variety musXXX
```

* The meta character `^` forces REGEXP to match only at start of line

```bash
$ # filtering lines starting with 'cat'
$ sed -n '/^cat/p' anchors.txt 
cat and dog
catapults laid waste to the village

$ # replace only at start of line
$ # g modifier not needed as there can only be single match at start of line
$ sed 's/^cat/XXX/' anchors.txt
XXX and dog
too many cats around here
to concatenate, use the cmd cat
XXXapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety muscat

$ # add something to start of line
$ echo 'Have a good day' | sed 's/^/Hi! /'
Hi! Have a good day
```

* The meta character `$` forces REGEXP to match only at end of line

```bash
$ # filtering lines ending with 'cat'
$ sed -n '/cat$/p' anchors.txt 
to concatenate, use the cmd cat
try the grape variety muscat

$ # replace only at end of line
$ sed 's/cat$/YYY/' anchors.txt 
cat and dog
too many cats around here
to concatenate, use the cmd YYY
catapults laid waste to the village
just scat and quit bothering me
that is quite a fabricated tale
try the grape variety musYYY

$ # add something to end of line
$ echo 'Have a good day' | sed 's/$/. Cya later/'
Have a good day. Cya later
```

<br>

#### <a name="word-anchors"></a>Word Anchors

* A **word** character is any alphabet (irrespective of case) or any digit or the underscore character
* The word anchors help in matching or not matching boundaries of a word
    * For example, to distinguish between `par`, `spar` and `apparent`
* `\b` matches word boundary
    * `\` is meta character and certain combinations like `\b` and `\B` have special meaning
* One can also use these alternatives for `\b`
    * `\<` for start of word
    * `\>` for end of word

```bash
$ # words ending with 'cat'
$ sed -n 's/cat\b/XXX/p' anchors.txt 
XXX and dog
to concatenate, use the cmd XXX
just sXXX and quit bothering me
try the grape variety musXXX

$ # words starting with 'cat'
$ sed -n 's/\bcat/YYY/p' anchors.txt 
YYY and dog
too many YYYs around here
to concatenate, use the cmd YYY
YYYapults laid waste to the village

$ # only whole words
$ sed -n 's/\bcat\b/ZZZ/p' anchors.txt 
ZZZ and dog
to concatenate, use the cmd ZZZ

$ # word is made up of alphabets, numbers and _
$ echo 'foo, foo_bar and foo1' | sed 's/\bfoo\b/baz/g'
baz, foo_bar and foo1
```

* `\B` is opposite of `\b`, i.e it doesn't match word boundaries

```bash
$ # substitute only if 'cat' is surrounded by word characters
$ sed -n 's/\Bcat\B/QQQ/p' anchors.txt 
to conQQQenate, use the cmd cat
that is quite a fabriQQQed tale

$ # substitute only if 'cat' is not start of word
$ sed -n 's/\Bcat/RRR/p' anchors.txt 
to conRRRenate, use the cmd cat
just sRRR and quit bothering me
that is quite a fabriRRRed tale
try the grape variety musRRR

$ # substitute only if 'cat' is not end of word
$ sed -n 's/cat\B/SSS/p' anchors.txt 
too many SSSs around here
to conSSSenate, use the cmd cat
SSSapults laid waste to the village
that is quite a fabriSSSed tale
```

<br>

#### <a name="matching-the-meta-characters"></a>Matching the meta characters

* Since meta characters like `^`, `$`, `\` etc have special meaning in REGEXP, they have to be escaped using `\` to match them literally
* Certain characters like `&` have special meaning in REPLACEMENT section of substitute as well. They too have to be escaped using `\`

```bash
$ # here, '^' will match only start of line
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/^/**/g'
**(a+b)^2 = a^2 + b^2 + 2ab

$ # '\` before '^' will match '^' literally
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/\^/**/g'
(a+b)**2 = a**2 + b**2 + 2ab

$ # to match '\' use '\\'
$ echo 'foo\bar' | sed 's/\\/ /'
foo bar

$ echo 'pa$$' | sed 's/$/s/g'
pa$$s
$ echo 'pa$$' | sed 's/\$/s/g'
pass

$ # '^' has special meaning only at start of REGEXP
$ # similarly, '$' has special meaning only at end of REGEXP
$ echo '(a+b)^2 = a^2 + b^2 + 2ab' | sed 's/a^2/A^2/g'
(a+b)^2 = A^2 + b^2 + 2ab
```


<br>
<br>
<br>

*More to follow*

# <a name="gnu-sed"></a>GNU sed

**Table of Contents**

* [Simple search and replace](#simple-search-and-replace)
    * [editing stdin](#editing-stdin)
    * [editing file input](#editing-file-input)
* [Inplace file editing](#inplace-file-editing)
    * [With backup](#with-backup)
    * [Without backup](#without-backup)
    * [Multiple files](#multiple-files)
    * [Prefix backup name](#prefix-backup-name)
    * [Place backups in directory](#place-backups-in-directory)
* [Line filtering options](#line-filtering-options)
    * [Print command](#print-command)
    * [Delete command](#delete-command)
    * [Quit commands](#quit-commands)
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
    * [Alternation](#alternation)
    * [The dot meta character](#the-dot-meta-character)
    * [Quantifiers](#quantifiers)
    * [Character classes](#character-classes)
    * [Escape sequences](#escape-sequences)
    * [Grouping](#grouping)
    * [Back reference](#back-reference)
    * [Changing case](#changing-case)
* [Substitute command modifiers](#substitute-command-modifiers)
    * [g modifier](#g-modifier)
    * [Replace specific occurrence](#replace-specific-occurrence)
    * [Ignoring case](#ignoring-case)
    * [p modifier](#p-modifier)
    * [w modifier](#w-modifier)
    * [e modifier](#e-modifier)
    * [m modifier](#m-modifier)
* [Shell substitutions](#shell-substitutions)
    * [Variable substitution](#variable-substitution)
    * [Command substitution](#command-substitution)
* [z and s command line options](#z-and-s-command-line-options)
* [change command](#change-command)
* [insert command](#insert-command)
* [append command](#append-command)
* [adding contents of file](#adding-contents-of-file)
    * [r for entire file](#r-for-entire-file)
    * [R for line by line](#r-for-line-by-line)
* [n and N commands](#n-and-n-commands)
* [Control structures](#control-structures)
    * [if then else](#if-then-else)
    * [replacing in specific column](#replacing-in-specific-column)
    * [overlapping substitutions](#overlapping-substitutions)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [Include or Exclude matching REGEXPs](#include-or-exclude-matching-regexps)
    * [First or Last block](#first-or-last-block)
    * [Broken blocks](#broken-blocks)
* [sed scripts](#sed-scripts)
* [Further Reading](#further-reading)

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

**Note:** [Multiline and manipulating pattern space](https://www.gnu.org/software/sed/manual/sed.html#Multiline-techniques) with h,x,D,G,H,P etc is not covered in this chapter and examples/information is based on ASCII encoded text input only

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

Detailed examples for **substitute** command will be convered in later sections, syntax is

```
s/REGEXP/REPLACEMENT/FLAGS
```

The `/` character is idiomatically used as delimiter character. See also [Using different delimiter for REGEXP](#using-different-delimiter-for-regexp)

<br>

#### <a name="editing-stdin"></a>editing stdin

```bash
$ # sample command output to be edited
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

$ # change only first ',' to ' : '
$ seq 10 | paste -sd, | sed 's/,/ : /'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : ' by using 'g' modifier
$ seq 10 | paste -sd, | sed 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

**Note:** As a good practice, all examples use single quotes around arguments to prevent shell interpretation. See [Shell substitutions](#shell-substitutions) section on use of double quotes

<br>

#### <a name="editing-file-input"></a>editing file input

* By default newline character is the line separator
* See [Regular Expressions](#regular-expressions) section for qualifying search terms, for ex
    * word boundaries to distinguish between 'hi', 'this', 'his', 'history', etc
    * multiple search terms, specific set of character, etc

```bash
$ cat greeting.txt 
Hi there
Have a nice day

$ # change first 'e' in each line to 'E'
$ sed 's/e/E/' greeting.txt
Hi thEre
HavE a nice day

$ # change first 'nice day' in each line to 'safe journey'
$ sed 's/nice day/safe journey/' greeting.txt
Hi there
Have a safe journey

$ # change all 'e' to 'E' and save changed text to another file
$ sed 's/e/E/g' greeting.txt > out.txt
$ cat out.txt 
Hi thErE
HavE a nicE day
```

<br>

## <a name="inplace-file-editing"></a>Inplace file editing

* In previous section, the output from `sed` was displayed on stdout or saved to another file
* To write the changes back to original file, use `-i` option

**Note**:

* Refer to `man sed` for details of how to use the `-i` option. It varies with different `sed` implementations. As mentioned at start of this chapter, `sed (GNU sed) 4.2.2` is being used here
* See [this Q&A](https://unix.stackexchange.com/questions/348693/sed-update-etc-grub-conf-in-spite-this-link-file) when working with symlinks

<br>

#### <a name="with-backup"></a>With backup

* When extension is given, the original input file is preserved with name changed according to extension provided

```bash
$ # '.bkp' is extension provided
$ sed -i.bkp 's/Hi/Hello/' greeting.txt

$ # original file gets preserved in 'greeting.txt.bkp'
Hi there
Have a nice day

$ # output from sed gets written to 'greeting.txt'
$ cat greeting.txt
Hello there
Have a nice day
```

<br>

#### <a name="without-backup"></a>Without backup

* Use this option with caution, changes made cannot be undone

```bash
$ sed -i 's/nice day/safe journey/' greeting.txt

$ # note, 'Hi' was already changed to 'Hello' in previous example
$ cat greeting.txt
Hello there
Have a safe journey
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

#### <a name="prefix-backup-name"></a>Prefix backup name

* A `*` in argument given to `-i` will get expanded to input filename
* This way, one can add prefix instead of suffix for backup

```bash
$ cat var.txt 
foo
bar
baz

$ sed -i'bkp.*' 's/foo/hello/' var.txt 
$ cat var.txt 
hello
bar
baz

$ cat bkp.var.txt 
foo
bar
baz
```

<br>

#### <a name="place-backups-in-directory"></a>Place backups in directory

* `*` also allows to specify an existing directory to place the backups instead of current working directory

```bash
$ mkdir bkp_dir
$ sed -i'bkp_dir/*' 's/bar/hi/' var.txt 
$ cat var.txt 
hello
hi
baz

$ cat bkp_dir/var.txt
hello
bar
baz

$ # extensions can be added as well
$ # bkp_dir/*.bkp for suffix
$ # bkp_dir/bkp.* for prefix
$ # bkp_dir/bkp.*.2017 for both and so on
```

<br>

## <a name="line-filtering-options"></a>Line filtering options

* By default, `sed` acts on entire file. Often, one needs to extract or change only specific lines based on text search, line numbers, lines between two patterns, etc
* This filtering is much like using `grep`, `head` and `tail` commands in many ways and there are even more features
    * Use `sed` for inplace editing, the filtered lines to be transformed etc. Not as substitute for those commands

<br>

#### <a name="print-command"></a>Print command

* It is usually used in conjunction with `-n` option
* By default, `sed` prints every input line, including any changes made by commands like substitution
    * printing here refers to line being part of `sed` output which may be shown on terminal, redirected to file, etc
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

$ # if line contains 'are', perform given command
$ # print only if substitution succeeds
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

* Modifier `I` allows to filter lines in case-insensitive way
* See [Regular Expressions](#regular-expressions) section for more details

```bash
$ # /rose/I means match the string 'rose' irrespective of case
$ sed '/rose/Id' poem.txt 
Violets are blue,
Sugar is sweet,
And so are you.
```

<br>

#### <a name="quit-commands"></a>Quit commands

* Exit `sed` without processing further input

```bash
$ # same as: seq 23 45 | head -n5
$ # remember that printing is default action if -n is not used
$ # here, 5 is line number based addressing
$ seq 23 45 | sed '5q'
23
24
25
26
27
```

* `Q` is similar to `q` but won't print the matching line

```bash
$ seq 23 45 | sed '5Q'
23
24
25
26

$ # useful to print from beginning of file up to but not including line matching REGEXP
$ sed '/is/Q' poem.txt 
Roses are red,
Violets are blue,
```

* Use `tac` to get all lines starting from last occurrence of search string

```bash
$ # all lines from last occurrence of '7'
$ seq 50 | tac | sed '/7/q' | tac
47
48
49
50

$ # all lines from last occurrence of '7' excluding line with '7'
$ seq 50 | tac | sed '/7/Q' | tac
48
49
50
```

**Note**

* This way of using quit commands won't work for inplace editing with multiple file input
* See [this Q&A](https://unix.stackexchange.com/questions/309514/sed-apply-changes-in-multiple-files) for alternate solution, also has solutions using `gawk` and `perl`

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

* See also [sed manual - Multiple commands syntax](https://www.gnu.org/software/sed/manual/sed.html#Multiple-commands-syntax) for more details
* See also [sed scripts](#sed-scripts) section for an alternate way

```bash
$ # each command as argument to -e option
$ sed -n -e '/blue/p' -e '/you/p' poem.txt 
Violets are blue,
And so are you.

$ # each command separated by ;
$ # not all commands can be specified so
$ sed -n '/blue/p; /you/p' poem.txt 
Violets are blue,
And so are you.

$ # each command separated by literal newline character
$ # might depend on whether the shell allows such multiline command
$ sed -n '
/blue/p
/you/p
' poem.txt
Violets are blue,
And so are you.
```

* Use `{}` command grouping for logical AND

```bash
$ # same as: grep 'are' poem.txt | grep 'And'
$ # space between /REGEXP/ and {} is optional
$ sed -n '/are/ {/And/p}' poem.txt 
And so are you.

$ # same as: grep 'are' poem.txt | grep -v 'so'
$ sed -n '/are/ {/so/!p}' poem.txt 
Roses are red,
Violets are blue,

$ # same as: grep -v 'red' poem.txt | grep -v 'blue'
$ sed -n '/red/!{/blue/!p}' poem.txt 
Sugar is sweet,
And so are you.
$ # many ways to do it, use whatever feels easier to construct
$ # sed -e '/red/d' -e '/blue/d' poem.txt 
$ # grep -v -e 'red' -e 'blue' poem.txt
```

* Different ways to do same things. See also [Alternation](#alternation) and [Control structures](#control-structures)

```bash
$ # multiple commands can lead to duplicatation
$ sed -n '/blue/p; /t/p' poem.txt 
Violets are blue,
Violets are blue,
Sugar is sweet,
$ # in such cases, use regular expressions instead
$ sed -nE '/blue|t/p;' poem.txt 
Violets are blue,
Sugar is sweet,

$ sed -nE '/red|blue/!p' poem.txt 
Sugar is sweet,
And so are you.

$ sed -n '/so/b; /are/p' poem.txt
Roses are red,
Violets are blue,
```

<br>

#### <a name="filtering-by-line-number"></a>Filtering by line number

* Exact line number can be specified to be acted upon
* As a special case, `$` indicates last line of file
* See also [sed manual - Multiple commands syntax](https://www.gnu.org/software/sed/manual/sed.html#Multiple-commands-syntax)

```bash
$ # here, 2 represents the address for print command, similar to /REGEXP/p
$ # same as: head -n2 poem.txt | tail -n1
$ sed -n '2p' poem.txt 
Violets are blue,

$ # print 2nd and 4th line
$ sed -n '2p; 4p' poem.txt 
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
$ sed -n '$p' poem.txt 
And so are you.

$ # delete except 3rd line
$ sed '3!d' poem.txt
Sugar is sweet,

$ # substitution only on 2nd line
$ sed '2 s/are/ARE/' poem.txt
Roses are red,
Violets ARE blue,
Sugar is sweet,
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

* So far, we've seen how to filter specific line based on *REGEXP* and line numbers
* `sed` also allows to combine them to enable selecting a range of lines
* Consider the sample input file for this section

```bash
$ cat addr_range.txt 
Hello World

Good day
How are you

Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

Much ado about nothing
He he he
```

* Range defined by start and end *REGEXP*
* For other cases like getting lines without the line matching start and/or end, unbalanced start/end, when end *REGEXP* doesn't match, etc see [Lines between two REGEXPs](#lines-between-two-regexps) section

```bash
$ sed -n '/is/,/like/p' addr_range.txt 
Today is sunny
Not a bit funny
No doubt you like it too

$ sed -n '/just/I,/believe/Ip' addr_range.txt 
Just do-it
Believe it

$ # the second REGEXP will always be checked after the line matching first address
$ sed -n '/No/,/No/p' addr_range.txt 
Not a bit funny
No doubt you like it too

$ # all the matching ranges will be printed
$ sed -n '/you/,/do/p' addr_range.txt 
How are you

Just do-it
No doubt you like it too

Much ado about nothing
```

* Range defined by start and end line numbers

```bash
$ # print lines numbered 3 to 7
$ sed -n '3,7p' addr_range.txt 
Good day
How are you

Just do-it
Believe it

$ # print lines from line number 13 to last line
$ sed -n '13,$p' addr_range.txt 
Much ado about nothing
He he he

$ # delete lines numbered 2 to 13
$ sed '2,13d' addr_range.txt 
Hello World
He he he
```

* Range defined by mix of line number and *REGEXP*

```bash
$ sed -n '3,/do/p' addr_range.txt 
Good day
How are you

Just do-it

$ sed -n '/Today/,$p' addr_range.txt 
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

$ # same as: sed '/Today/,$d' addr_range.txt
$ sed -n '/Today/,$!p' addr_range.txt
Hello World

Good day
How are you

Just do-it
Believe it

```

<br>

#### <a name="relative-addressing"></a>Relative addressing

* Prefixing `+` to a number for second address gives relative filtering
* Similar to using `grep -A<num> --no-group-separator 'REGEXP'` but `grep` merges adjacent groups while `sed` does not

```bash
$ # line matching 'is' and 2 lines after
$ sed -n '/is/,+2p' addr_range.txt 
Today is sunny
Not a bit funny
No doubt you like it too

$ # note that all matching ranges will be filtered
$ sed -n '/do/,+2p' addr_range.txt 
Just do-it
Believe it

No doubt you like it too

Much ado about nothing
```

* The first address could be number too
* Useful when using [Shell substitutions](#shell-substitutions)

```bash
$ sed -n '3,+4p' addr_range.txt 
Good day
How are you

Just do-it
Believe it
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
* After the matching line based on number or *REGEXP* of start address, the closest line number multiple of `j` will mark end address

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
$ sed -n '/Just/,~5p' addr_range.txt 
Just do-it
Believe it

Today is sunny
Not a bit funny
```

<br>

## <a name="using-different-delimiter-for-regexp"></a>Using different delimiter for REGEXP

* `/` is idiomatically used as the *REGEXP* delimiter
    * See also [a bit of history on why / is commonly used as delimiter](https://www.reddit.com/r/commandline/comments/3lhgwh/why_did_people_standardize_on_using_forward/cvgie7j/)
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

* For *REGEXP* used in address matching, syntax is a bit different `\<char>REGEXP<char>`

```bash
$ printf '/foo/bar/1\n/foo/baz/1\n'
/foo/bar/1
/foo/baz/1

$ printf '/foo/bar/1\n/foo/baz/1\n' | sed -n '\;/foo/bar/;p'
/foo/bar/1
```

<br>

## <a name="regular-expressions"></a>Regular Expressions

* By default, `sed` treats *REGEXP* as BRE (Basic Regular Expression)
* The `-E` option enables ERE (Extended Regular Expression) which in GNU sed's case only differs in how meta characters are used, no difference in functionalities
    * Initially GNU sed only had `-r` option to enable ERE and `man sed` doesn't even mention `-E`
    * Other `sed` versions use `-E` and `grep` uses `-E` as well. So `-r` won't be used in examples in this tutorial
    * See also [sed manual - BRE-vs-ERE](https://www.gnu.org/software/sed/manual/sed.html#BRE-vs-ERE)
* See [sed manual - Regular Expressions](https://www.gnu.org/software/sed/manual/sed.html#sed-regular-expressions) for more details

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

* The meta character `^` forces *REGEXP* to match only at start of line

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

* The meta character `$` forces *REGEXP* to match only at end of line

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

* Since meta characters like `^`, `$`, `\` etc have special meaning in *REGEXP*, they have to be escaped using `\` to match them literally

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

* Certain characters like `&` and `\` have special meaning in *REPLACEMENT* section of substitute as well. They too have to be escaped using `\`
* And the delimiter character has to be escaped of course
* See [back reference](#back-reference) section for use of `&` in *REPLACEMENT* section

```bash
$ # & will refer to entire matched string of REGEXP section
$ echo 'foo and bar' | sed 's/and/"&"/'
foo "and" bar
$ echo 'foo and bar' | sed 's/and/"\&"/'
foo "&" bar

$ # use different delimiter where required
$ echo 'a b' | sed 's/ /\//'
a/b
$ echo 'a b' | sed 's# #/#'
a/b

$ # use \\ to represent literal \
$ echo '/foo/bar/baz' | sed 's#/#\\#g'
\foo\bar\baz
```

<br>

#### <a name="alternation"></a>Alternation

* Two or more *REGEXP* can be combined as logical OR using the `|` meta character
    * syntax is `\|` for BRE and `|` for ERE
* Each side of `|` is complete regular expression with their own start/end anchors
* How each part of alternation is handled and order of evaluation/output is beyond the scope of this tutorial
    * See [this](http://www.regular-expressions.info/alternation.html) for more info on this topic.

```bash
$ # BRE
$ sed -n '/red\|blue/p' poem.txt 
Roses are red,
Violets are blue,

$ # ERE
$ sed -nE '/red|blue/p' poem.txt 
Roses are red,
Violets are blue,

$ # filter lines starting or ending with 'cat'
$ sed -nE '/^cat|cat$/p' anchors.txt 
cat and dog
to concatenate, use the cmd cat
catapults laid waste to the village
try the grape variety muscat

$ # g modifier is needed for more than one replacement
$ echo 'foo and temp and baz' | sed -E 's/foo|temp|baz/XYZ/'
XYZ and temp and baz
$ echo 'foo and temp and baz' | sed -E 's/foo|temp|baz/XYZ/g'
XYZ and XYZ and XYZ
```

<br>

#### <a name="the-dot-meta-character"></a>The dot meta character

* The `.` meta character matches any character once, including newline

```bash
$ # replace all sequence of 3 characters starting with 'c' and ending with 't'
$ echo 'coat cut fit c#t' | sed 's/c.t/XYZ/g'
coat XYZ fit XYZ

$ # replace all sequence of 4 characters starting with 'c' and ending with 't'
$ echo 'coat cut fit c#t' | sed 's/c..t/ABCD/g'
ABCD cut fit c#t

$ # space, tab etc are also characters which will be matched by '.' 
$ echo 'coat cut fit c#t' | sed 's/t.f/IJK/g'
coat cuIJKit c#t
```

<br>

#### <a name="quantifiers"></a>Quantifiers

All quantifiers in `sed` are greedy, i.e longest match wins as long as overall *REGEXP* is satisfied and precedence is left to right. In this section, we'll cover usage of quantifiers on characters

* `?` will try to match 0 or 1 time
* For BRE, use `\?`

```bash
$ printf 'late\npale\nfactor\nrare\nact\n'
late
pale
factor
rare
act

$ # same as using: sed -nE '/at|act/p'
$ printf 'late\npale\nfactor\nrare\nact\n' | sed -nE '/ac?t/p'
late
factor
act

$ # greediness comes in handy in some cases
$ # problem: '<' has to be replaced with '\<' only if not preceded by '\'
$ echo 'blah \< foo bar < blah baz <'
blah \< foo bar < blah baz <
$ # this won't work as '\<' gets replaced with '\\<'
$ echo 'blah \< foo bar < blah baz <' | sed -E 's/</\\</g'
blah \\< foo bar \< blah baz \<
$ # by using '\\?<' both '\<' and '<' gets replaced by '\<'
$ echo 'blah \< foo bar < blah baz <' | sed -E 's/\\?</\\</g'
blah \< foo bar \< blah baz \<
```

* `*` will try to match 0 or more times

```bash
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n'
abc
ac
adc
abbc
bbb
bc
abbbbbc

$ # match 'a' and 'c' with any number of 'b' in between
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab*c/p'
abc
ac
abbc
abbbbbc

$ # delete from start of line to 'te'
$ echo 'that is quite a fabricated tale' | sed 's/.*te//'
d tale
$ # delete from start of line to 'te '
$ echo 'that is quite a fabricated tale' | sed 's/.*te //'
a fabricated tale
$ # delete from first 'f' in the line to end of line
$ echo 'that is quite a fabricated tale' | sed 's/f.*//'
that is quite a 
```

* `+` will try to match 1 or more times
* For BRE, use `\+`

```bash
$ # match 'a' and 'c' with at least one 'b' in between
$ # BRE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab\+c/p'
abc
abbc
abbbbbc

$ # ERE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab+c/p'
abc
abbc
abbbbbc
```

* For more precise control on number of times to match, use `{}`

```bash
$ # exactly 5 times
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{5}c/p'
abbbbbc

$ # between 1 to 3 times, inclusive of 1 and 3
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{1,3}c/p'
abc
abbc

$ # maximum of 2 times, including 0 times
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{,2}c/p'
abc
ac
abbc

$ # minimum of 2 times
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -nE '/ab{2,}c/p'
abbc
abbbbbc

$ # BRE
$ printf 'abc\nac\nadc\nabbc\nbbb\nbc\nabbbbbc\n' | sed -n '/ab\{2,\}c/p'
abbc
abbbbbc
```

<br>

#### <a name="character-classes"></a>Character classes

* The `.` meta character provides a way to match any character
* Character class provides a way to match any character among a specified set of characters enclosed within `[]`

```bash
$ # same as: sed -nE '/lane|late/p'
$ printf 'late\nlane\nfate\nfete\n' | sed -n '/la[nt]e/p'
late
lane

$ printf 'late\nlane\nfate\nfete\n' | sed -n '/[fl]a[nt]e/p'
late
lane
fate

$ # quantifiers can be added similar to using for any other character
$ # filter lines made up entirely of digits, containing at least one digit
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0123456789]+$/p'
123
42
$ # filter lines made up entirely of digits, containing at least three digits
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0123456789]{3,}$/p'
123
```

Character ranges

* Matching any alphabet, number, hexadecimal number etc becomes cumbersome if every character has to be individually specified
* So, there's a shortcut, using `-` to construct a range (has to be specified in ascending order)
* See [ascii codes table](http://ascii.cl/) for reference
    * Note that behavior of range will depend on locale settings
    * [arch wiki - locale](https://wiki.archlinux.org/index.php/locale)
    * [Linux: Define Locale and Language Settings](https://www.shellhacks.com/linux-define-locale-language-settings/)

```bash
$ # filter lines made up entirely of digits, at least one
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[0-9]+$/p'
123
42

$ # filter lines made up entirely of lower case alphabets, at least one
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[a-z]+$/p'
foo

$ # filter lines made up entirely of lower case alphabets and digits, at least one
$ printf 'cat5\nfoo\n123\n42\n' | sed -nE '/^[a-z0-9]+$/p'
cat5
foo
123
42
```

* Numeric ranges, easy for certain cases but not suitable always. Use `awk` or `perl` for arithmetic computation
* See also [Matching Numeric Ranges with a Regular Expression](http://www.regular-expressions.info/numericranges.html)

```bash
$ # numbers between 10 to 29
$ printf '23\n154\n12\n26\n98234\n' | sed -n '/^[12][0-9]$/p'
23
12
26

$ # numbers >= 100
$ printf '23\n154\n12\n26\n98234\n' | sed -nE '/^[0-9]{3,}$/p'
154
98234

$ # numbers >= 100 if there are leading zeros
$ printf '0501\n035\n154\n12\n26\n98234\n' | sed -nE '/^0*[1-9][0-9]{2,}$/p'
0501
154
98234
```

Negating character class

* Meta characters inside and outside of `[]` are completely different
* For example, `^` as first character inside `[]` matches characters other than those specified inside character class

```bash
$ # delete zero or more characters before first =
$ echo 'foo=bar; baz=123' | sed 's/^[^=]*//'
=bar; baz=123

$ # delete zero or more characters after last =
$ echo 'foo=bar; baz=123' | sed 's/[^=]*$//'
foo=bar; baz=

$ # same as: sed -n '/[aeiou]/!p'
$ printf 'tryst\nglyph\npity\nwhy\n' | sed -n '/^[^aeiou]*$/p'
tryst
glyph
why
```

Matching meta characters inside `[]`

* Characters like `^`, `]`, `-`, etc need special attention to be part of list
* Also, sequences like `[.` or `=]` have special meaning within `[]`
    * See [sed manual - Character-Classes-and-Bracket-Expressions](https://www.gnu.org/software/sed/manual/sed.html#Character-Classes-and-Bracket-Expressions) for complete list

```bash
$ # to match - it should be first or last character within []
$ printf 'Foo-bar\nabc-456\n42\nCo-operate\n' | sed -nE '/^[a-z-]+$/Ip'
Foo-bar
Co-operate

$ # to match ] it should be first character within []
$ printf 'int foo\nint a[5]\nfoo=bar\n' | sed -n '/[]=]/p'
int a[5]
foo=bar

$ # to match [ use [ anywhere in the character list
$ # [][] will match both [ and ]
$ printf 'int foo\nint a[5]\nfoo=bar\n' | sed -n '/[[]/p'
int a[5]

$ # to match ^ it should be other than first in the list
$ printf 'c=a^b\nd=f*h+e\nz=x-y\n' | sed -n '/[*^]/p'
c=a^b
d=f*h+e
```

Named character classes

* Equivalent class shown is for C locale and ASCII character encoding
    * See [ascii codes table](http://ascii.cl/) for reference
* See [sed manual - Character Classes and Bracket Expressions](https://www.gnu.org/software/sed/manual/sed.html#Character-Classes-and-Bracket-Expressions) for more details

| Character classes | Description |
| ------------- | ----------- |
| [:digit:] | Same as [0-9] |
| [:lower:] | Same as [a-z] |
| [:upper:] | Same as [A-Z] |
| [:alpha:] | Same as [a-zA-Z] |
| [:alnum:] | Same as [0-9a-zA-Z] |
| [:xdigit:] | Same as [0-9a-fA-F] |
| [:cntrl:] | Control characters - first 32 ASCII characters and 127th (DEL) |
| [:punct:] | All the punctuation characters |
| [:graph:] | [:alnum:] and [:punct:] |
| [:print:] | [:alnum:], [:punct:] and space |
| [:blank:] | Space and tab characters |
| [:space:] | white-space characters: tab, newline, vertical tab, form feed, carriage return and space |

```bash
$ # lines containing only hexadecimal characters
$ printf '128\n34\nfe32\nfoo1\nbar\n' | sed -nE '/^[[:xdigit:]]+$/p'
128
34
fe32

$ # lines containing at least one non-hexadecimal character
$ printf '128\n34\nfe32\nfoo1\nbar\n' | sed -n '/[^[:xdigit:]]/p'
foo1
bar

$ # same as: sed -nE '/^[a-z-]+$/Ip'
$ printf 'Foo-bar\nabc-456\n42\nCo-operate\n' | sed -nE '/^[[:alpha:]-]+$/p'
Foo-bar
Co-operate

$ # remove all punctuation characters
$ sed 's/[[:punct:]]//g' poem.txt 
Roses are red
Violets are blue
Sugar is sweet
And so are you
```

Backslash character classes

* Equivalent class shown is for C locale and ASCII character encoding
    * See [ascii codes table](http://ascii.cl/) for reference
* See [sed manual - regular expression extensions](https://www.gnu.org/software/sed/manual/sed.html#regexp-extensions) for more details

| Character classes | Description |
| ------------- | ----------- |
| \w | Same as [0-9a-zA-Z_] or [[:alnum:]_] |
| \W | Same as [^0-9a-zA-Z_] or [^[:alnum:]_] |
| \s | Same as [[:space:]] |
| \S | Same as [^[:space:]] |

```bash
$ # lines containing only word characters
$ printf '123\na=b+c\ncmp_str\nFoo_bar\n' | sed -nE '/^\w+$/p'
123
cmp_str
Foo_bar

$ # backslash character classes cannot be used inside [] unlike perl
$ # \w would simply match w
$ echo 'w=y-x+9*3' | sed 's/[\w=]//g'
y-x+9*3
$ echo 'w=y-x+9*3' | perl -pe 's/[\w=]//g'
-+*
```

<br>

#### <a name="escape-sequences"></a>Escape sequences

* Certain ASCII characters like tab, carriage return, newline, etc have escape sequence to represent them
    * Unlike backslash character classes, these can be used within `[]` as well
* Any ASCII character can be also represented using their decimal or octal or hexadecimal value
    * See [ascii codes table](http://ascii.cl/) for reference
* See [sed manual - Escapes](https://www.gnu.org/software/sed/manual/sed.html#Escapes) for more details

```bash
$ # example for representing tab character
$ printf 'foo\tbar\tbaz\n'
foo     bar     baz
$ printf 'foo\tbar\tbaz\n' | sed 's/\t/ /g'
foo bar baz
$ echo 'a b c' | sed 's/ /\t/g'
a       b       c

$ # using escape sequence inside character class
$ printf 'a\tb\vc\n'
a       b
         c
$ printf 'a\tb\vc\n' | cat -vT
a^Ib^Kc
$ printf 'a\tb\vc\n' | sed 's/[\t\v]/ /g'
a b c

$ # most common use case for hex escape sequence is to represent single quotes
$ # equivalent is '\d039' and '\o047' for decimal and octal respectively
$ echo "foo: '34'"
foo: '34'
$ echo "foo: '34'" | sed 's/\x27/"/g'
foo: "34"
$ echo 'foo: "34"' | sed 's/"/\x27/g'
foo: '34'
```

<br>

#### <a name="grouping"></a>Grouping

* Character classes allow matching against a choice of multiple character list and then quantifier added if needed
* One of the uses of grouping is analogous to character classes for whole regular expressions, instead of just list of characters
* The meta characters `()` are used for grouping
    * requires `\(\)` for BRE
* Similar to maths `ab + ac = a(b+c)`, think of regular expression `a(b|c) = ab|ac`

```bash
$ # four letter words with 'on' or 'no' in middle
$ printf 'known\nmood\nknow\npony\ninns\n' | sed -nE '/\b[a-z](on|no)[a-z]\b/p'
know
pony
$ # common mistake to use character class, will match 'oo' and 'nn' as well
$ printf 'known\nmood\nknow\npony\ninns\n' | sed -nE '/\b[a-z][on]{2}[a-z]\b/p'
mood
know
pony
inns

$ # quantifier example
$ printf 'handed\nhand\nhandy\nhands\nhandle\n' | sed -nE '/^hand([sy]|le)?$/p'
hand
handy
hands
handle

$ # remove first two columns where : is delimiter
$ echo 'foo:123:bar:baz' | sed -E 's/^([^:]+:){2}//'
bar:baz

$ # can be nested as required
$ printf 'spade\nscore\nscare\nspare\nsphere\n' | sed -nE '/^s([cp](he|a)[rd])e$/p'
spade
scare
spare
sphere
```

<br>

#### <a name="back-reference"></a>Back reference

* The matched string within `()` can also be used to be matched again by back referencing the captured groups
* `\1` denotes the first matched group, `\2` the second one and so on
    * Order is leftmost `(` is `\1`, next one is `\2` and so on
    * Can be used both in *REGEXP* as well as in *REPLACEMENT* sections
* `&` or `\0` represents entire matched string in *REPLACEMENT* section
* Note that the matched string, not the regular expression itself is referenced
    * for ex: if `([0-9][a-f])` matches `3b`, then back referencing will be `3b` not any other valid match of the regular expression like `8f`, `0a` etc
* As `\` and `&` are special characters in *REPLACEMENT* section, use `\\` and `\&` respectively for literal representation

```bash
$ # filter lines with consecutive repeated alphabets
$ printf 'eel\nflee\nall\npat\nilk\nseen\n' | sed -nE '/([a-z])\1/p'
eel
flee
all
seen

$ # reduce \\ to single \ and delete if only single \
$ echo '\[\] and \\w and \[a-zA-Z0-9\_\]' | sed -E 's/(\\?)\\/\1/g'
[] and \w and [a-zA-Z0-9_]

$ # remove two or more duplicate words separated by space
$ # word boundaries prevent false matches like 'the theatre' 'sand and stone' etc
$ echo 'a a a walking for for a cause' | sed -E 's/\b(\w+)( \1)+\b/\1/g'
a walking for a cause

$ # surround only third column with double quotes
$ # note the nested capture groups and numbers used in REPLACEMENT section
$ echo 'foo:123:bar:baz' | sed -E 's/^(([^:]+:){2})([^:]+)/\1"\3"/'
foo:123:"bar":baz

$ # add first column data to end of line as well
$ echo 'foo:123:bar:baz' | sed -E 's/^([^:]+).*/& \1/'
foo:123:bar:baz foo

$ # surround entire line with double quotes
$ echo 'hello world' | sed 's/.*/"&"/'
"hello world"
$ # add something at start as well as end of line
$ echo 'hello world' | sed 's/.*/Hi. &. Have a nice day/'
Hi. hello world. Have a nice day
```

<br>

#### <a name="changing-case"></a>Changing case

* Applies only to *REPLACEMENT* section, unlike `perl` where these can be used in *REGEXP* portion as well
* See [sed manual - The s Command](https://www.gnu.org/software/sed/manual/sed.html#The-_0022s_0022-Command) for more details and corner cases

```bash
$ # UPPERCASE all alphabets, will be stopped on \L or \E
$ echo 'HeLlO WoRLD' | sed 's/.*/\U&/'
HELLO WORLD

$ # lowercase all alphabets, will be stopped on \U or \E
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/'
hello world

$ # Uppercase only next character
$ echo 'foo bar' | sed 's/\w*/\u&/g'
Foo Bar
$ echo 'foo_bar next_line' | sed -E 's/_([a-z])/\u\1/g'
fooBar nextLine

$ # lowercase only next character
$ echo 'FOO BAR' | sed 's/\w*/\l&/g'
fOO bAR
$ echo 'fooBar nextLine Baz' | sed -E 's/([a-z])([A-Z])/\1_\l\2/g'
foo_bar next_line Baz

$ # titlecase if input has mixed case
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/; s/\w*/\u&/g'
Hello World
$ # sed 's/.*/\L\u&/' also works, but not sure if it is defined behavior
$ echo 'HeLlO WoRLD' | sed 's/.*/\L&/; s/./\u&/'
Hello world

$ # \E will stop conversion started by \U or \L
$ echo 'foo_bar next_line baz' | sed -E 's/([a-z]+)(_[a-z]+)/\U\1\E\2/g'
FOO_bar NEXT_line baz
```

<br>

## <a name="substitute-command-modifiers"></a>Substitute command modifiers

The `s` command syntax:

```
s/REGEXP/REPLACEMENT/FLAGS
```

* Modifiers (or FLAGS) like `g`, `p` and `I` have been already seen. For completeness, they will be discussed again along with rest of the modifiers
* See [sed manual - The s Command](https://www.gnu.org/software/sed/manual/sed.html#The-_0022s_0022-Command) for more details and corner cases

<br>

#### <a name="g-modifier"></a>g modifier

By default, substitute command will replace only first occurrence of match. `g` modifier is needed to replace all occurrences

```bash
$ # replace only first : with -
$ echo 'foo:123:bar:baz' | sed 's/:/-/'
foo-123:bar:baz

$ # replace all : with -
$ echo 'foo:123:bar:baz' | sed 's/:/-/g'
foo-123-bar-baz
```

<br>

#### <a name="replace-specific-occurrence"></a>Replace specific occurrence

* A number can be used to specify *N*th match to be replaced

```bash
$ # replace first occurrence
$ echo 'foo:123:bar:baz' | sed 's/:/-/'
foo-123:bar:baz
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/'
XYZ:123:bar:baz

$ # replace second occurrence
$ echo 'foo:123:bar:baz' | sed 's/:/-/2'
foo:123-bar:baz
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/2'
foo:XYZ:bar:baz

$ # replace third occurrence
$ echo 'foo:123:bar:baz' | sed 's/:/-/3'
foo:123:bar-baz
$ echo 'foo:123:bar:baz' | sed -E 's/[^:]+/XYZ/3'
foo:123:XYZ:baz

$ # choice of quantifier depends on knowing input
$ echo ':123:bar:baz' | sed 's/[^:]*/XYZ/2'
:XYZ:bar:baz
$ echo ':123:bar:baz' | sed -E 's/[^:]+/XYZ/2'
:123:XYZ:baz
```

* Replacing *N*th match from end of line when number of matches is unknown
* Makes use of greediness of quantifiers

```bash
$ # replacing last occurrence
$ # can also use sed -E 's/:([^:]*)$/-\1/'
$ echo 'foo:123:bar:baz' | sed -E 's/(.*):/\1-/'
foo:123:bar-baz
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):/\1-/'
456:foo:123:bar:789-baz
$ echo 'foo and bar and baz and good' | sed -E 's/(.*)and/\1XYZ/'
foo and bar and baz XYZ good
$ # use word boundaries as necessary
$ echo 'foo and bar and baz land good' | sed -E 's/(.*)\band\b/\1XYZ/'
foo and bar XYZ baz land good

$ # replacing last but one
$ echo 'foo:123:bar:baz' | sed -E 's/(.*):(.*:)/\1-\2/'
foo:123-bar:baz
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):(.*:)/\1-\2/'
456:foo:123:bar-789:baz

$ # replacing last but two
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):((.*:){2})/\1-\2/'
456:foo:123-bar:789:baz
$ # replacing last but three
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(.*):((.*:){3})/\1-\2/'
456:foo-123:bar:789:baz
```

* Replacing all but first *N* occurrences by combining with `g` modifier

```bash
$ # replace all : with - except first two
$ echo '456:foo:123:bar:789:baz' | sed -E 's/:/-/3g'
456:foo:123-bar-789-baz

$ # replace all : with - except first three
$ echo '456:foo:123:bar:789:baz' | sed -E 's/:/-/4g'
456:foo:123:bar-789-baz
```

* Replacing multiple *N*th occurrences

```bash
$ # replace first two occurrences of : with -
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/; s/:/-/'
456-foo-123:bar:789:baz

$ # replace second and third occurrences of : with -
$ # note the changes in number to be used for subsequent replacement
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/2; s/:/-/2'
456:foo-123-bar:789:baz

$ # better way is to use descending order
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/3; s/:/-/2'
456:foo-123-bar:789:baz
$ # replace second, third and fifth occurrences of : with -
$ echo '456:foo:123:bar:789:baz' | sed 's/:/-/5; s/:/-/3; s/:/-/2'
456:foo-123-bar:789-baz
```

<br>

#### <a name="ignoring-case"></a>Ignoring case

* Either `i` or `I` can be used for replacing in case-insensitive manner
* Since only `I` can be used for address filtering (for ex: `sed '/rose/Id' poem.txt`), use `I` for substitute command as well for consistency

```bash
$ echo 'hello Hello HELLO HeLlO' | sed 's/hello/hi/g'
hi Hello HELLO HeLlO

$ echo 'hello Hello HELLO HeLlO' | sed 's/hello/hi/Ig'
hi hi hi hi
```

<br>

#### <a name="p-modifier"></a>p modifier

* Usually used in conjunction with `-n` option to output only modified lines

```bash
$ # no output if no substitution
$ echo 'hi there. have a nice day' | sed -n 's/xyz/XYZ/p'
$ # modified line if there is substitution
$ echo 'hi there. have a nice day' | sed -n 's/\bh/H/pg'
Hi there. Have a nice day

$ # only lines containing 'are'
$ sed -n 's/are/ARE/p' poem.txt 
Roses ARE red,
Violets ARE blue,
And so ARE you.

$ # only lines containing 'are' as well as 'so'
$ sed -n '/are/ s/so/SO/p' poem.txt 
And SO are you.
```

<br>

#### <a name="w-modifier"></a>w modifier

* Allows to write only the changes to specified file name instead of default **stdout**

```bash
$ # space between w and filename is optional
$ # same as: sed -n 's/3/three/p' > 3.txt
$ seq 20 | sed -n 's/3/three/w 3.txt'
$ cat 3.txt 
three
1three

$ # do not use -n if output should be displayed as well as written to file
$ echo '456:foo:123:bar:789:baz' | sed -E 's/(:[^:]*){2}$//w col.txt'
456:foo:123:bar
$ cat col.txt 
456:foo:123:bar
```

* For multiple output files, use `-e` for each file

```bash
$ seq 20 | sed -n -e 's/5/five/w 5.txt' -e 's/7/seven/w 7.txt'
$ cat 5.txt 
five
1five
$ cat 7.txt 
seven
1seven
```

* There are two predefined filenames
    * `/dev/stdout` to write to **stdout**
    * `/dev/stderr` to write to **stderr**

```bash
$ # inplace editing as well as display changes on terminal
$ sed -i 's/three/3/w /dev/stdout' 3.txt 
3
13
$ cat 3.txt 
3
13
```

<br>

#### <a name="e-modifier"></a>e modifier

* Allows to use shell command output in *REPLACEMENT* section
* Trailing newline from command output is suppressed

```bash
$ # replacing a line with output of shell command
$ printf 'Date:\nreplace this line\n'
Date:
replace this line
$ printf 'Date:\nreplace this line\n' | sed 's/^replace.*/date/e'
Date:
Thu May 25 10:19:46 IST 2017

$ # when using p modifier with e, order is important
$ printf 'Date:\nreplace this line\n' | sed -n 's/^replace.*/date/ep'
Thu May 25 10:19:46 IST 2017
$ printf 'Date:\nreplace this line\n' | sed -n 's/^replace.*/date/pe'
date

$ # entire modified line is executed as shell command
$ echo 'xyz 5' | sed 's/xyz/seq/e'
1
2
3
4
5
```

<br>

#### <a name="m-modifier"></a>m modifier

* Either `m` or `M` can be used
* So far, we've seen only line based operations (newline character being used to distinguish lines)
* There are various ways (see [sed manual - How sed Works](https://www.gnu.org/software/sed/manual/sed.html#Execution-Cycle)) by which more than one line is there in pattern space and in such cases `m` modifier can be used
* See also [usage of multi-line modifier](https://unix.stackexchange.com/questions/298670/simple-significant-usage-of-m-multi-line-address-suffix) for more examples

Before seeing example with `m` modifier, let's see a simple example to get two lines in pattern space

```bash
$ # line matching 'blue' and next line in pattern space
$ sed -n '/blue/{N;p}' poem.txt 
Violets are blue,
Sugar is sweet,

$ # applying substitution, remember that . matches newline as well
$ sed -n '/blue/{N;s/are.*is//p}' poem.txt 
Violets  sweet,
```

* When `m` modifier is used, it affects the behavior of `^`, `$` and `.` meta characters

```bash
$ # without m modifier, ^ will anchor only beginning of entire pattern space
$ sed -n '/blue/{N;s/^/:: /pg}' poem.txt 
:: Violets are blue,
Sugar is sweet,
$ # with m modifier, ^ will anchor each individual line within pattern space
$ sed -n '/blue/{N;s/^/:: /pgm}' poem.txt 
:: Violets are blue,
:: Sugar is sweet,

$ # same applies to $ as well
$ sed -n '/blue/{N;s/$/ ::/pg}' poem.txt 
Violets are blue,
Sugar is sweet, ::
$ sed -n '/blue/{N;s/$/ ::/pgm}' poem.txt 
Violets are blue, ::
Sugar is sweet, ::

$ # with m modifier, . will not match newline character
$ sed -n '/blue/{N;s/are.*//p}' poem.txt 
Violets 
$ sed -n '/blue/{N;s/are.*//pm}' poem.txt 
Violets 
Sugar is sweet,
```

<br>

## <a name="shell-substitutions"></a>Shell substitutions

* Examples presented works with `bash` shell, might differ for other shells
* See also [Difference between single and double quotes in Bash](https://stackoverflow.com/questions/6697753/difference-between-single-and-double-quotes-in-bash)
* For robust substitutions taking care of meta characters in *REGEXP* and *REPLACEMENT* sections, see
    * [How to ensure that string interpolated into sed substitution escapes all metachars](https://unix.stackexchange.com/questions/129059/how-to-ensure-that-string-interpolated-into-sed-substitution-escapes-all-metac)
    * [What characters do I need to escape when using sed in a sh script?](https://unix.stackexchange.com/questions/32907/what-characters-do-i-need-to-escape-when-using-sed-in-a-sh-script)
    * [Is it possible to escape regex metacharacters reliably with sed](https://stackoverflow.com/questions/29613304/is-it-possible-to-escape-regex-metacharacters-reliably-with-sed)

<br>

#### <a name="variable-substitution"></a>Variable substitution

* Entire command in double quotes can be used for simple use cases

```bash
$ word='are'
$ sed -n "/$word/p" poem.txt 
Roses are red,
Violets are blue,
And so are you.

$ replace='ARE'
$ sed "s/$word/$replace/g" poem.txt 
Roses ARE red,
Violets ARE blue,
Sugar is sweet,
And so ARE you.

$ # need to use delimiter as suitable
$ echo 'home path is:' | sed "s/$/ $HOME/"
sed: -e expression #1, char 7: unknown option to `s'
$ echo 'home path is:' | sed "s|$| $HOME|"
home path is: /home/learnbyexample
```

* If command has characters like `\`, backtick, `!` etc, double quote only the variable

```bash
$ # if history expansion is enabled, ! is special
$ word='are'
$ sed "/$word/!d" poem.txt 
sed "/$word/date +%A" poem.txt 
sed: -e expression #1, char 7: extra characters after command

$ # so double quote only the variable
$ # the command is concatenation of '/' and "$word" and '/!d'
$ sed '/'"$word"'/!d' poem.txt 
Roses are red,
Violets are blue,
And so are you.
```

<br>

#### <a name="command-substitution"></a>Command substitution

* Much more flexible than using `e` modifier as part of line can be modified as well

```bash
$ echo 'today is date' | sed 's/date/'"$(date +%A)"'/'
today is Tuesday

$ # need to use delimiter as suitable
$ echo 'current working dir is: ' | sed 's/$/'"$(pwd)"'/'
sed: -e expression #1, char 6: unknown option to `s'
$ echo 'current working dir is: ' | sed 's|$|'"$(pwd)"'|'
current working dir is: /home/learnbyexample/command_line_text_processing

$ # multiline output cannot be substituted in this manner
$ echo 'foo' | sed 's/foo/'"$(seq 5)"'/'
sed: -e expression #1, char 7: unterminated `s' command
```

<br>

## <a name="z-and-s-command-line-options"></a>z and s command line options

* We have already seen a few options like `-n`, `-e`, `-i` and `-E`
* This section will cover `-z` and `-s` options
* See [sed manual - Command line options](https://www.gnu.org/software/sed/manual/sed.html#Command_002dLine-Options) for other options and more details

The `-z` option will cause `sed` to separate input based on ASCII NUL character instead of newlines

```bash
$ # useful to process null separated data
$ # for ex: output of grep -Z, find -print0, etc
$ printf 'teal\0red\nblue\n\0green\n' | sed -nz '/red/p' | cat -A
red$
blue$
^@

$ # also useful to process whole file(not having NUL characters) as a single string
$ # adds ; to previous line if current line starts with c
$ printf 'cat\ndog\ncoat\ncut\nmat\n' | sed -z 's/\nc/;&/g'
cat
dog;
coat;
cut
mat
```

The `-s` option will cause `sed` to treat multiple input files separately instead of treating them as single concatenated input. If `-i` is being used, `-s` is implied

```bash
$ # without -s, there is only one first line
$ # F command prints file name of current file
$ sed '1F' f1 f2
f1
I ate three apples
I bought two bananas and three mangoes

$ # with -s, each file has its own address
$ sed -s '1F' f1 f2
f1
I ate three apples
f2
I bought two bananas and three mangoes
```

<br>

<br>

## <a name="change-command"></a>change command

The change command `c` will delete line(s) represented by address or address range and replace it with given string

**Note** the string used cannot have literal newline character, use escape sequence instead

```bash
$ # white-space between c and replacement string is ignored
$ seq 3 | sed '2c foo bar'
1
foo bar
3

$ # note how all lines in address range are replaced
$ seq 8 | sed '3,7cfoo bar'
1
2
foo bar
8

$ # escape sequences are allowed in string to be replaced
$ sed '/red/,/is/chello\nhi there' poem.txt 
hello
hi there
And so are you.
```

* command will apply for all matching addresses

```bash
$ seq 5 | sed '/[24]/cfoo'
1
foo
3
foo
5
```

* `\` is special immediately after `c`, see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* If escape sequence is needed at beginning of replacement string, use an additional `\`

```bash
$ # \ helps to add leading spaces
$ seq 3 | sed '2c  a'
1
a
3
$ seq 3 | sed '2c\ a'
1
 a
3

$ seq 3 | sed '2c\tgood day'
1
tgood day
3
$ seq 3 | sed '2c\\tgood day'
1
        good day
3
```

* Since `;` cannot be used to distinguish between string and end of command, use `-e` for multiple commands

```bash
$ sed -e '/are/cHi;s/is/IS/' poem.txt 
Hi;s/is/IS/
Hi;s/is/IS/
Sugar is sweet,
Hi;s/is/IS/

$ sed -e '/are/cHi' -e 's/is/IS/' poem.txt 
Hi
Hi
Sugar IS sweet,
Hi
```

* Using shell substitution

```bash
$ text='good day'
$ seq 3 | sed '2c'"$text"
1
good day
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2c'"$text"
1
good day
foo bar
3

$ seq 3 | sed '2c'"$(date +%A)"
1
Thursday
3

$ # multiline command output will lead to error
$ seq 3 | sed '2c'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

<br>

## <a name="insert-command"></a>insert command

The insert command allows to add string before a line matching given address

**Note** the string used cannot have literal newline character, use escape sequence instead

```bash
$ # white-space between i and string is ignored
$ # same as: sed '2s/^/hello\n/'
$ seq 3 | sed '2i hello'
1
hello
2
3

$ # escape sequences can be used
$ seq 3 | sed '2ihello\nhi'
1
hello
hi
2
3
```

* command will apply for all matching addresses

```bash
$ seq 5 | sed '/[24]/ifoo'
1
foo
2
3
foo
4
5
```

* `\` is special immediately after `i`, see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* If escape sequence is needed at beginning of replacement string, use an additional `\`

```bash
$ seq 3 | sed '2i  foo'
1
foo
2
3
$ seq 3 | sed '2i\ foo'
1
 foo
2
3

$ seq 3 | sed '2i\tbar'
1
tbar
2
3
$ seq 3 | sed '2i\\tbar'
1
        bar
2
3
```

* Since `;` cannot be used to distinguish between string and end of command, use `-e` for multiple commands

```bash
$ sed -e '/is/ifoobar;s/are/ARE/' poem.txt 
Roses are red,
Violets are blue,
foobar;s/are/ARE/
Sugar is sweet,
And so are you.

$ sed -e '/is/ifoobar' -e 's/are/ARE/' poem.txt 
Roses ARE red,
Violets ARE blue,
foobar
Sugar is sweet,
And so ARE you.
```

* Using shell substitution

```bash
$ text='good day'
$ seq 3 | sed '2i'"$text"
1
good day
2
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2i'"$text"
1
good day
foo bar
2
3

$ seq 3 | sed '2iToday is '"$(date +%A)"
1
Today is Thursday
2
3

$ # multiline command output will lead to error
$ seq 3 | sed '2i'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

<br>

## <a name="append-command"></a>append command

The append command allows to add string after a line matching given address

**Note** the string used cannot have literal newline character, use escape sequence instead

```bash
$ # white-space between a and string is ignored
$ # same as: sed '2s/$/\nhello/'
$ seq 3 | sed '2a hello'
1
2
hello
3

$ # escape sequences can be used
$ seq 3 | sed '2ahello\nhi'
1
2
hello
hi
3
```

* command will apply for all matching addresses

```bash
$ seq 5 | sed '/[24]/afoo'
1
2
foo
3
4
foo
5
```

* `\` is special immediately after `a`, see [sed manual - other commands](https://www.gnu.org/software/sed/manual/sed.html#Other-Commands) for details
* If escape sequence is needed at beginning of replacement string, use an additional `\`

```bash
$ seq 3 | sed '2a  foo'
1
2
foo
3
$ seq 3 | sed '2a\ foo'
1
2
 foo
3

$ seq 3 | sed '2a\tbar'
1
2
tbar
3
$ seq 3 | sed '2a\\tbar'
1
2
        bar
3
```

* Since `;` cannot be used to distinguish between string and end of command, use `-e` for multiple commands

```bash
$ sed -e '/is/afoobar;s/are/ARE/' poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
foobar;s/are/ARE/
And so are you.

$ sed -e '/is/afoobar' -e 's/are/ARE/' poem.txt 
Roses ARE red,
Violets ARE blue,
Sugar is sweet,
foobar
And so ARE you.
```

* Using shell substitution

```bash
$ text='good day'
$ seq 3 | sed '2a'"$text"
1
2
good day
3

$ text='good day\nfoo bar'
$ seq 3 | sed '2a'"$text"
1
2
good day
foo bar
3

$ seq 3 | sed '2aToday is '"$(date +%A)"
1
2
Today is Thursday
3

$ # multiline command output will lead to error
$ seq 3 | sed '2a'"$(seq 2)"
sed: -e expression #1, char 5: missing command
```

* See [this Q&A](https://stackoverflow.com/questions/41343062/what-does-this-mean-in-linux-sed-a-a-txt) for using `a` command to make sure last line of input has a newline character

<br>

## <a name="adding-contents-of-file"></a>adding contents of file

<br>

#### <a name="r-for-entire-file"></a>r for entire file

* The `r` command allows to add contents of file after a line matching given address
* It is a robust way to add multiline content or if content can have characters that may be interpreted
* Special name `/dev/stdin` allows to read from **stdin** instead of file input
* First, a simple example to add contents of one file into another at specified address

```bash
$ cat 5.txt 
five
1five

$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # space between r and filename is optional
$ sed '2r 5.txt' poem.txt 
Roses are red,
Violets are blue,
five
1five
Sugar is sweet,
And so are you.

$ # content cannot be added before first line
$ sed '0r 5.txt' poem.txt 
sed: -e expression #1, char 2: invalid usage of line address 0
$ # but that is trivial to solve: cat 5.txt poem.txt
```

* command will apply for all matching addresses

```bash
$ seq 5 | sed '/[24]/r 5.txt'
1
2
five
1five
3
4
five
1five
5
```

* adding content of variable as it is without any interpretation
* also shows example for using `/dev/stdin`

```bash
$ text='Good day\nfoo bar baz\n'
$ # escape sequence like \n will be interpreted when 'a' command is used
$ sed '/is/a'"$text" poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
Good day
foo bar baz

And so are you.

$ # \ is just another character, won't be treated as special with 'r' command
$ echo "$text" | sed '/is/r /dev/stdin' poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
Good day\nfoo bar baz\n
And so are you.
```

* adding multiline command output is simple as well

```bash
$ seq 3 | sed '/is/r /dev/stdin' poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
1
2
3
And so are you.
```

* replacing a line or range of lines with contents of file
* See also [various ways to replace line M in file1 with line N in file2](https://unix.stackexchange.com/a/396450)

```bash
$ # replacing range of lines
$ # order is important, first 'r' and then 'd'
$ sed -e '/is/r 5.txt' -e '1,/is/d' poem.txt 
five
1five
And so are you.

$ # replacing a line
$ seq 3 | sed -e '3r /dev/stdin' -e '3d' poem.txt
Roses are red,
Violets are blue,
1
2
3
And so are you.

$ # can also use {} grouping to avoid repeating the address
$ seq 3 | sed -e '/blue/{r /dev/stdin' -e 'd}' poem.txt
Roses are red,
1
2
3
Sugar is sweet,
And so are you.
```

<br>

#### <a name="r-for-line-by-line"></a>R for line by line

* add a line for every address match
* Special name `/dev/stdin` allows to read from **stdin** instead of file input

```bash
$ # space between R and filename is optional
$ seq 3 | sed '/are/R /dev/stdin' poem.txt 
Roses are red,
1
Violets are blue,
2
Sugar is sweet,
And so are you.
3

$ sed '2,3R 5.txt' poem.txt 
Roses are red,
Violets are blue,
five
Sugar is sweet,
1five
And so are you.
```

* number of lines from file to be read different from number of matching address lines

```bash
$ # file has more lines than matching address
$ # 2 lines in 5.txt but only 1 line matching 'is'
$ sed '/is/R 5.txt' poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
five
And so are you.

$ # lines matching address is more than file to be read
$ # 3 lines matching 'are' but only 2 lines from stdin
$ seq 2 | sed '/are/R /dev/stdin' poem.txt
Roses are red,
1
Violets are blue,
2
Sugar is sweet,
And so are you.
```

<br>

## <a name="n-and-n-commands"></a>n and N commands

* These two commands will fetch next line (newline or NUL character separated, depending on options)
* `n` will fetch the next line and replace whatever is already there in pattern space

```bash
$ # if line contains 'blue', replace 'e' with 'E' only for following line
$ sed '/blue/{n;s/e/E/g}' poem.txt 
Roses are red,
Violets are blue,
Sugar is swEEt,
And so are you.

$ # better illustrated with -n option
$ sed -n '/blue/{n;s/e/E/pg}' poem.txt 
Sugar is swEEt,

$ # if line contains 'blue', replace 'e' with 'E' only for next to next line
$ sed -n '/blue/{n;n;s/e/E/pg}' poem.txt 
And so arE you.
```

* `N` will fetch the next line and append to pattern space
* See [this Q&A](https://stackoverflow.com/questions/40229578/how-to-insert-a-line-feed-into-a-sed-line-concatenation) for an interesting case of applying substitution every 4 lines but excluding the 4th line

```bash
$ # if line contains 'blue', replace 'e' with 'E' both in current line and next
$ sed '/blue/{N;s/e/E/g}' poem.txt 
Roses are red,
ViolEts arE bluE,
Sugar is swEEt,
And so are you.

$ # better illustrated with -n option
$ sed -n '/blue/{N;s/e/E/pg}' poem.txt 
ViolEts arE bluE,
Sugar is swEEt,

$ sed -n '/blue/{N;N;s/e/E/pg}' poem.txt 
ViolEts arE bluE,
Sugar is swEEt,
And so arE you.
```

* Combination

```bash
$ # n will fetch next line, current line is out of pattern space
$ # N will then add another line
$ sed -n '/blue/{n;N;s/e/E/pg}' poem.txt 
Sugar is swEEt,
And so arE you.
```

* not necessary to qualify with an address

```bash
$ seq 6 | sed 'n;cXYZ'
1
XYZ
3
XYZ
5
XYZ

$ seq 6 | sed 'N;s/\n/ /'
1 2
3 4
5 6
```

<br>

## <a name="control-structures"></a>Control structures

* Using `:label` one can mark a command location to branch to conditionally or unconditionally
* See [sed manual - Commands for sed gurus](https://www.gnu.org/software/sed/manual/sed.html#Programming-Commands) for more details

<br>

#### <a name="if-then-else"></a>if then else

* Simple if-then-else can be simulated using `b` command
* `b` command will unconditionally branch to specified label
* Without label, `b` will skip rest of commands and start next cycle
* See [processing only lines between REGEXPs](https://unix.stackexchange.com/questions/292819/remove-commented-lines-except-one-comment-using-sed) for interesting use case

```bash
$ # changing -ve to +ve and vice versa
$ cat nums.txt 
42
-2
10101
-3.14
-75
$ # same as: perl -pe '/^-/ ? s/// : s/^/-/'
$ # empty REGEXP section will reuse previous REGEXP, in this case /^-/
$ sed '/^-/{s///;b}; s/^/-/' nums.txt 
-42
2
-10101
3.14
75

$ # same as: perl -pe '/are/ ? s/e/*/g : s/e/#/g'
$ # if line contains 'are' replace 'e' with '*' else replace 'e' with '#'
$ sed '/are/{s/e/*/g;b}; s/e/#/g' poem.txt 
Ros*s ar* r*d,
Viol*ts ar* blu*,
Sugar is sw##t,
And so ar* you.
```

<br>

#### <a name="replacing-in-specific-column"></a>replacing in specific column

* `t` command will branch to specified label on successful substitution
* Without label, `t` will skip rest of commands and start next cycle
* More examples
    * [replace data after last delimiter](https://stackoverflow.com/questions/39907133/replace-data-after-last-delimiter-of-every-line-using-sed-or-awk/39908523#39908523)
    * [replace multiple occurrences in specific column](https://stackoverflow.com/questions/42886531/replace-mutliple-occurances-in-delimited-columns/42886919#42886919)

```bash
$ # replace space with underscore only in 3rd column
$ # ^(([^|]+\|){2} captures first two columns
$ # [^|]* zero or more non-column separator characters
$ # as long as match is found, command will be repeated on same input line
$ echo 'foo bar|a b c|1 2 3|xyz abc' | sed -E ':a s/^(([^|]+\|){2}[^|]*) /\1_/; ta'
foo bar|a b c|1_2_3|xyz abc

$ # use awk/perl for simpler syntax
$ # for ex: awk 'BEGIN{FS=OFS="|"} {gsub(/ /,"_",$3); print}'
```

* example to show difference between `b` and `t`

```bash
$ # whether or not 'R' is found on lines containing 'are', branch will happen
$ sed '/are/{s/R/*/g;b}; s/e/#/g' poem.txt 
*oses are red,
Violets are blue,
Sugar is sw##t,
And so are you.

$ # branch only if line contains 'are' and substitution of 'R' succeeds
$ sed '/are/{s/R/*/g;t}; s/e/#/g' poem.txt 
*oses are red,
Viol#ts ar# blu#,
Sugar is sw##t,
And so ar# you.
```

<br>

#### <a name="overlapping-substitutions"></a>overlapping substitutions

* `t` command looping with label comes in handy for overlapping substitutions as well
* Note that in general this method will work recursively, see [substitute recursively](https://stackoverflow.com/questions/9983646/sed-substitute-recursively) for example

```bash
$ # consider the problem of replacing empty columns with something
$ # case1: no consecutive empty columns - no problem
$ echo 'foo::bar::baz' | sed 's/::/:0:/g'
foo:0:bar:0:baz
$ # case2: consecutive empty columns are present - problematic
$ echo 'foo:::bar::baz' | sed 's/::/:0:/g'
foo:0::bar:0:baz

$ # t command looping will handle both cases
$ echo 'foo::bar::baz' | sed ':a s/::/:0:/; ta'
foo:0:bar:0:baz
$ echo 'foo:::bar::baz' | sed ':a s/::/:0:/; ta'
foo:0:0:bar:0:baz
```

<br>

## <a name="lines-between-two-regexps"></a>Lines between two REGEXPs

* Simple cases were seen in [address range](#address-range) section
* This section will deal with more cases and some corner cases

<br>

#### <a name="include-or-exclude-matching-regexps"></a>Include or Exclude matching REGEXPs

Consider the sample input file, for simplicity the two REGEXPs are **BEGIN** and **END** strings instead of regular expressions

```bash
$ cat range.txt 
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
END
baz
```

First, lines between the two *REGEXP*s are to be printed

* Case 1: both starting and ending *REGEXP* part of output

```bash
$ sed -n '/BEGIN/,/END/p' range.txt 
BEGIN
1234
6789
END
BEGIN
a
b
c
END
```

* Case 2: both starting and ending *REGEXP* not part of ouput

```bash
$ # remember that empty REGEXP section will reuse previously matched REGEXP
$ sed -n '/BEGIN/,/END/{//!p}' range.txt 
1234
6789
a
b
c
```

* Case 3: only starting *REGEXP* part of output

```bash
$ sed -n '/BEGIN/,/END/{/END/!p}' range.txt 
BEGIN
1234
6789
BEGIN
a
b
c
```

* Case 4: only ending *REGEXP* part of output

```bash
$ sed -n '/BEGIN/,/END/{/BEGIN/!p}' range.txt 
1234
6789
END
a
b
c
END
```

Second, lines between the two *REGEXP*s are to be deleted

* Case 5: both starting and ending *REGEXP* not part of output

```bash
$ sed '/BEGIN/,/END/d' range.txt 
foo
bar
baz
```

* Case 6: both starting and ending *REGEXP* part of output

```bash
$ # remember that empty REGEXP section will reuse previously matched REGEXP
$ sed '/BEGIN/,/END/{//!d}' range.txt 
foo
BEGIN
END
bar
BEGIN
END
baz
```

* Case 7: only starting *REGEXP* part of output

```bash
$ sed '/BEGIN/,/END/{/BEGIN/!d}' range.txt 
foo
BEGIN
bar
BEGIN
baz
```

* Case 8: only ending *REGEXP* part of output

```bash
$ sed '/BEGIN/,/END/{/END/!d}' range.txt 
foo
END
bar
END
baz
```

<br>

#### <a name="first-or-last-block"></a>First or Last block

* Getting first block is very simple by using `q` command

```bash
$ sed -n '/BEGIN/,/END/{p;/END/q}' range.txt 
BEGIN
1234
6789
END

$ # use other tricks discussed in previous section as needed
$ sed -n '/BEGIN/,/END/{//!p;/END/q}' range.txt 
1234
6789
```

* To get last block, reverse the input linewise, the order of *REGEXP*s and finally reverse again

```bash
$ tac range.txt | sed -n '/END/,/BEGIN/{p;/BEGIN/q}' | tac
BEGIN
a
b
c
END

$ # use other tricks discussed in previous section as needed
$ tac range.txt | sed -n '/END/,/BEGIN/{//!p;/BEGIN/q}' | tac
a
b
c
```

* To get a specific block, say 3rd one, `awk` or `perl` would be a better choice
    * See [Specific blocks](./gnu_awk.md#specific-blocks) for `awk` examples

<br>

#### <a name="broken-blocks"></a>Broken blocks

* If there are blocks with ending *REGEXP* but without corresponding starting *REGEXP*, `sed -n '/BEGIN/,/END/p'` will suffice
* Consider the modified input file where final starting *REGEXP* doesn't have corresponding ending

```bash
$ cat broken_range.txt 
foo
BEGIN
1234
6789
END
bar
BEGIN
a
b
c
baz
```

* All lines till end of file gets printed with simple use of `sed -n '/BEGIN/,/END/p'`
* The file reversing trick comes in handy here as well
* But if both kinds of broken blocks are present, further processing will be required. Better to use `awk` or `perl` in such cases
    * See [Broken blocks](./gnu_awk.md#broken-blocks) for `awk` examples

```bash
$ sed -n '/BEGIN/,/END/p' broken_range.txt 
BEGIN
1234
6789
END
BEGIN
a
b
c
baz

$ tac broken_range.txt | sed -n '/END/,/BEGIN/p' | tac
BEGIN
1234
6789
END
```

* If there are multiple starting *REGEXP* but single ending *REGEXP*, the reversing trick comes handy again

```bash
$ cat uneven_range.txt 
foo
BEGIN
1234
BEGIN
42
6789
END
bar
BEGIN
a
BEGIN
b
BEGIN
c
BEGIN
d
BEGIN
e
END
baz

$ tac uneven_range.txt | sed -n '/END/,/BEGIN/p' | tac
BEGIN
42
6789
END
BEGIN
e
END
```

<br>

## <a name="sed-scripts"></a>sed scripts

* `sed` commands can be placed in a file and called using `-f` option or directly executed using [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix))
* See [sed manual - Some Sample Scripts](https://www.gnu.org/software/sed/manual/sed.html#Examples) for more examples
* See [sed manual - Often-Used Commands](https://www.gnu.org/software/sed/manual/sed.html#Common-Commands) for more details on using comments

```bash
$ cat script.sed 
# each line is a command
/is/cfoo bar
/you/r 3.txt
/you/d
# single quotes can be used freely
s/are/'are'/g

$ sed -f script.sed poem.txt 
Roses 'are' red,
Violets 'are' blue,
foo bar
3
13

$ # command line options are specified as usual
$ sed -nf script.sed poem.txt 
foo bar
3
13
```

* command line options can be specified along with shebang as well as added at time of invocation
* **Note** [usage of options along with shebang depends on lot of factors](https://stackoverflow.com/questions/4303128/how-to-use-multiple-arguments-with-a-shebang-i-e)

```bash
$ type sed
sed is /bin/sed

$ cat executable.sed 
#!/bin/sed -f
/is/cfoo bar
/you/r 3.txt
/you/d
s/are/'are'/g

$ chmod +x executable.sed 

$ ./executable.sed poem.txt 
Roses 'are' red,
Violets 'are' blue,
foo bar
3
13

$ ./executable.sed -n poem.txt 
foo bar
3
13
```

<br>

## <a name="further-reading"></a>Further Reading

* Manual and related
    * `man sed` and `info sed` for more details, known issues/limitations as well as options/commands not covered in this tutorial
    * [GNU sed manual](https://www.gnu.org/software/sed/manual/sed.html) has even more detailed information and examples
    * [sed FAQ](http://sed.sourceforge.net/sedfaq.html), but last modified '10 March 2003'
    * [BSD/macOS Sed vs GNU Sed vs the POSIX Sed specification](https://stackoverflow.com/questions/24275070/sed-not-giving-me-correct-substitute-operation-for-newline-with-mac-difference/24276470#24276470)
    * [Differences between sed on Mac OSX and other standard sed](https://unix.stackexchange.com/questions/13711/differences-between-sed-on-mac-osx-and-other-standard-sed)
* Tutorials and Q&A
    * [sed basics](https://code.snipcademy.com/tutorials/shell-scripting/sed/introduction)
    * [sed detailed tutorial](http://www.grymoire.com/Unix/Sed.html) - has details on differences between various `sed` versions as well
    * [sed one-liners explained](http://www.catonmat.net/series/sed-one-liners-explained)
    * [cheat sheet](http://www.catonmat.net/download/sed.stream.editor.cheat.sheet.txt)
    * [common search and replace examples](https://unix.stackexchange.com/questions/112023/how-can-i-replace-a-string-in-a-files)
    * [sed Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/sed?sort=votes&pageSize=15)
    * [sed Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/sed?sort=votes&pageSize=15)
* Selected examples - portable solutions, commands not covered in this tutorial, same problem solved using different tools, etc
    * [replace multiline string](http://unix.stackexchange.com/questions/26284/how-can-i-use-sed-to-replace-a-multi-line-string)
    * [deleting empty lines with optional white spaces](https://stackoverflow.com/questions/16414410/delete-empty-lines-using-sed)
    * [print only line above the matching line](https://unix.stackexchange.com/questions/264489/find-each-line-matching-a-pattern-but-print-only-the-line-above-it)
    * [How to select lines between two patterns?](https://stackoverflow.com/questions/38972736/how-to-select-lines-between-two-patterns)
    * [get lines between two patterns only if there is third pattern between them](https://stackoverflow.com/questions/39960075/bash-how-to-get-lines-between-patterns-only-if-there-is-pattern2-between-them)
        * [similar example](https://unix.stackexchange.com/questions/228699/sed-print-lines-matched-by-a-pattern-range-if-one-line-matches-a-condition)
* Learn Regular Expressions (has information on flavors other than BRE/ERE too)
    * [Regular Expressions Tutorial](http://www.regular-expressions.info/tutorial.html)
    * [regexcrossword](https://regexcrossword.com/)
    * [What does this regex mean?](https://stackoverflow.com/questions/22937618/reference-what-does-this-regex-mean)
* Related tools
    * [sedsed - Debugger, indenter and HTMLizer for sed scripts](https://github.com/aureliojargas/sedsed)
    * [xo - composes regular expression match groups](https://github.com/ezekg/xo)
* [unix.stackexchange - When to use grep, sed, awk, perl, etc](https://unix.stackexchange.com/questions/303044/when-to-use-grep-less-awk-sed)

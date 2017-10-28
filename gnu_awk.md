## <a name="gnu-awk"></a>GNU awk

**Table of Contents**

* [Field processing](#field-processing)
    * [Default field separation](#default-field-separation)
    * [Specifying different input field separator](#specifying-different-input-field-separator)
    * [Specifying different output field separator](#specifying-different-output-field-separator)
* [Filtering](#filtering)
    * [Idiomatic print usage](#idiomatic-print-usage)
    * [Field comparison](#field-comparison)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)
    * [Line number based filtering](#line-number-based-filtering)
* [Case Insensitive filtering](#case-insensitive-filtering)
* [Changing record separators](#changing-record-separators)
    * [Paragraph mode](#paragraph-mode)
    * [Multicharacter RS](#multicharacter-rs)
* [Substitute functions](#substitute-functions)
* [Inplace file editing](#inplace-file-editing)
* [Using shell variables](#using-shell-variables)
* [Multiple file input](#multiple-file-input)
* [Control Structures](#control-structures)
    * [if-else and loops](#if-else-and-loops)
    * [next and nextfile](#next-and-nextfile)
* [Multiline processing](#multiline-processing)
* [Two file processing](#two-file-processing)
    * [Comparing whole lines](#comparing-whole-lines)
    * [Comparing specific fields](#comparing-specific-fields)
    * [getline](#getline)
* [Creating new fields](#creating-new-fields)
* [Dealing with duplicates](#dealing-with-duplicates)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [All unbroken blocks](#all-unbroken-blocks)
    * [Specific blocks](#specific-blocks)
    * [Broken blocks](#broken-blocks)
* [Arrays](#arrays)
* [awk scripts](#awk-scripts)
* [Miscellaneous](#miscellaneous)
    * [FPAT and FIELDWIDTHS](#fpat-and-fieldwidths)
    * [String functions](#string-functions)
    * [Executing external commands](#executing-external-commands)
    * [printf formatting](#printf-formatting)
    * [Redirecting print output](#redirecting-print-output)
* [Gotchas and Tips](#gotchas-and-tips)
* [Further Reading](#further-reading)

<br>

```bash
$ awk --version | head -n1
GNU Awk 4.1.3, API: 1.1 (GNU MPFR 3.1.4, GNU MP 6.1.0)

$ man awk
GAWK(1)                        Utility Commands                        GAWK(1)

NAME
       gawk - pattern scanning and processing language

SYNOPSIS
       gawk [ POSIX or GNU style options ] -f program-file [ -- ] file ...
       gawk [ POSIX or GNU style options ] [ -- ] program-text file ...

DESCRIPTION
       Gawk  is  the  GNU Project's implementation of the AWK programming lan‐
       guage.  It conforms to the definition of  the  language  in  the  POSIX
       1003.1  Standard.   This version in turn is based on the description in
       The AWK Programming Language, by Aho, Kernighan, and Weinberger.   Gawk
       provides  the additional features found in the current version of Brian
       Kernighan's awk and a number of GNU-specific extensions.
...
```

**Prerequisites and notes**

* familiarity with programming concepts like variables, printing, control structures, arrays, etc
* familiarity with regular expressions
    * if not, check out **ERE** portion of [GNU sed regular expressions](./gnu_sed.md#regular-expressions) which is close enough to features available in `gawk`
* this tutorial is primarily focussed on short programs that are easily usable from command line, similar to using `grep`, `sed`, etc
* see [Gawk: Effective AWK Programming](https://www.gnu.org/software/gawk/manual/) manual for complete reference, has information on other `awk` versions as well as notes on POSIX standard

<br>

## <a name="field-processing"></a>Field processing

<br>

#### <a name="default-field-separation"></a>Default field separation

* `$0` contains the entire input record
    * default input record separator is newline character
* `$1` contains the first field text
    * default input field separator is one or more of continuous space, tab or newline characters
* `$2` contains the second field text and so on
* `$(2+3)` result of expressions can be used, this one evaluates to `$5` and hence gives fifth field
    * similarly if variable `i` has value `2`, then `$(i+3)` will give fifth field
    * See also [gawk manual - Expressions](https://www.gnu.org/software/gawk/manual/html_node/Expressions.html)
* `NF` is a built-in variable which contains number of fields in the current record
    * so, `$NF` will give last field
    * `$(NF-1)` will give second last field and so on

```bash
$ cat fruits.txt 
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field
$ awk '{print $1}' fruits.txt 
fruit
apple
banana
fig
guava

$ # print only second field
$ awk '{print $2}' fruits.txt 
qty
42
31
90
6
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option
* by setting `FS` variable
* See [FPAT and FIELDWIDTHS](#fpat-and-fieldwidths) section for other ways of defining input fields

```bash
$ # second field where input field separator is :
$ echo 'foo:123:bar:789' | awk -F: '{print $2}'
123

$ # last field
$ echo 'foo:123:bar:789' | awk -F: '{print $NF}'
789

$ # first and last field
$ # note the use of , and space between output fields
$ echo 'foo:123:bar:789' | awk -F: '{print $1, $NF}'
foo 789

$ # second last field
$ echo 'foo:123:bar:789' | awk -F: '{print $(NF-1)}'
bar

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | awk -F';' '{print $3}'
three
```

* Regular expressions based input field separator

```bash
$ echo 'Sample123string54with908numbers' | awk -F'[0-9]+' '{print $2}'
string

$ # first field will be empty as there is nothing before '{'
$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $1}'

$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $2}'
foo
$ echo '{foo}   bar=baz' | awk -F'[{}= ]+' '{print $3}'
bar
```

* default input field separator is one or more of continuous space, tab or newline characters (will be termed as whitespace here on)
    * exact same behavior if `FS` is assigned single space character
* in addition, leading and trailing whitespaces won't be considered when splitting the input record

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b     c
$ printf ' a    ate b\tc   \n' | awk '{print $1}'
a
$ printf ' a    ate b\tc   \n' | awk '{print NF}'
4
$ # same behavior if FS is assigned to single space character
$ printf ' a    ate b\tc   \n' | awk -F' ' '{print $1}'
a
$ printf ' a    ate b\tc   \n' | awk -F' ' '{print NF}'
4

$ # for anything else, leading/trailing whitespaces will be considered
$ printf ' a    ate b\tc   \n' | awk -F'[ \t]+' '{print $2}'
a
$ printf ' a    ate b\tc   \n' | awk -F'[ \t]+' '{print NF}'
6
```

* assigning empty string to FS will split the input record character wise
* note the use of command line option `-v` to set FS

```bash
$ echo 'apple' | awk -v FS= '{print $1}'
a
$ echo 'apple' | awk -v FS= '{print $2}'
p
$ echo 'apple' | awk -v FS= '{print $NF}'
e

$ # detecting multibyte characters depends on locale
$ printf 'hi👍 how are you?' | awk -v FS= '{print $3}'
👍
```

**Further Reading**

* [gawk manual - Field Splitting Summary](https://www.gnu.org/software/gawk/manual/html_node/Field-Splitting-Summary.html#Field-Splitting-Summary)
* [stackoverflow - explanation on default FS](https://stackoverflow.com/questions/30405694/default-field-separator-for-awk)
* [unix.stackexchange - filter lines if it contains a particular character only once](https://unix.stackexchange.com/questions/362550/how-to-remove-line-if-it-contains-a-character-exactly-once)

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* by setting `OFS` variable
* also gets added between every argument to `print` statement
    * use [printf](#printf-formatting) to avoid this
* default is single space

```bash
$ # statements inside BEGIN are executed before processing any input text
$ echo 'foo:123:bar:789' | awk 'BEGIN{FS=OFS=":"} {print $1, $NF}'
foo:789
$ # can also be set using command line option -v
$ echo 'foo:123:bar:789' | awk -F: -v OFS=':' '{print $1, $NF}'
foo:789

$ # changing a field will re-build contents of $0
$ echo ' a      ate b   ' | awk '{$2 = "foo"; print $0}' | cat -A
a foo b$

$ # $1=$1 is an idiomatic way to re-build when there is nothing else to change
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{print $0}'
foo:123:bar:789
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{$1=$1; print $0}'
foo-123-bar-789

$ # OFS is used to separate different arguments given to print
$ echo 'foo:123:bar:789' | awk -F: -v OFS='\t' '{print $1, $3}'
foo     bar

$ echo 'Sample123string54with908numbers' | awk -F'[0-9]+' '{$1=$1; print $0}'
Sample string with numbers
```

<br>

## <a name="filtering"></a>Filtering

<br>

#### <a name="idiomatic-print-usage"></a>Idiomatic print usage

* `print` statement with no arguments will print contents of `$0`
* if condition is specified without corresponding statements, contents of `$0` is printed if condition evaluates to true
* `1` is typically used to represent always true condition and thus print contents of `$0`

```bash
$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # displaying contents of input file(s) similar to 'cat' command
$ # equivalent to using awk '{print $0}' and awk '1'
$ awk '{print}' poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.
```

<br>

#### <a name="field-comparison"></a>Field comparison

* Each block of statements within `{}` can be prefixed by an optional condition so that those statements will execute only if condition evaluates to true
* Condition specified without corresponding statements will lead to printing contents of `$0` if condition evaluates to true

```bash
$ # if first field exactly matches the string 'apple'
$ awk '$1=="apple"{print $2}' fruits.txt 
42

$ # print first field if second field > 35
$ # NR>1 to avoid the header line
$ # NR built-in variable contains record number
$ awk 'NR>1 && $2>35{print $1}' fruits.txt 
apple
fig

$ # print header and lines with qty < 35
$ awk 'NR==1 || $2<35' fruits.txt
fruit   qty
banana  31
guava   6
```

* If the above examples are too confusing, think of it as syntactical sugar
* Statements are grouped within `{}`
    * inside `{}`, we have a `if` control structure
    * Like `C` language, braces not needed for single statements within `if`, but consider that `{}` is used for clarity
    * From this explicit syntax, remove the outer `{}`, `if` and `()` used for `if`
* As we'll see later, this allows to mash up few lines of program compactly on command line itself
    * Of course, for medium to large programs, it is better to put the code in separate file. See [awk scripts](#awk-scripts) section

```bash
$ # awk '$1=="apple"{print $2}' fruits.txt 
$ awk '{
         if($1 == "apple"){
            print $2
         }
       }' fruits.txt
42

$ # awk 'NR==1 || $2<35' fruits.txt
$ awk '{
         if(NR==1 || $2<35){
            print $0
         }
       }' fruits.txt
fruit   qty
banana  31
guava   6
```

**Further Reading**

* [gawk manual - Truth Values and Conditions](https://www.gnu.org/software/gawk/manual/html_node/Truth-Values-and-Conditions.html)
* [gawk manual - Operator Precedence](https://www.gnu.org/software/gawk/manual/html_node/Precedence.html)
* [unix.stackexchange - filtering columns by header name](https://unix.stackexchange.com/questions/359697/print-columns-in-awk-by-header-name)

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* the *REGEXP* is specified within `//` and by default acts upon `$0`
* See also [stackoverflow - lines around matching regexp](https://stackoverflow.com/questions/17908555/printing-with-sed-or-awk-a-line-following-a-matching-pattern)

```bash
$ # all lines containing the string 'are'
$ # same as: grep 'are' poem.txt
$ awk '/are/' poem.txt
Roses are red,
Violets are blue,
And so are you.

$ # negating REGEXP, same as: grep -v 'are' poem.txt
$ awk '!/are/' poem.txt
Sugar is sweet,

$ # same as: grep 'are' poem.txt | grep -v 'so'
$ awk '/are/ && !/so/' poem.txt
Roses are red,
Violets are blue,

$ # lines starting with 'a' or 'b'
$ awk '/^[ab]/' fruits.txt
apple   42
banana  31

$ # print last field of all lines containing 'are'
$ awk '/are/{print $NF}' poem.txt
red,
blue,
you.
```

* strings can be used as well, which will be interpreted as *REGEXP* if necessary
* Allows [using shell variables](#using-shell-variables) instead of hardcoded *REGEXP*
    * that section also notes difference between using `//` and string

```bash
$ awk '$0 !~ "are"' poem.txt
Sugar is sweet,

$ awk '$0 ~ "^[ab]"' fruits.txt
apple   42
banana  31

$ # also helpful if search strings have the / delimiter character
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log
$ awk '/\/foo\/a\//' paths.txt
/foo/a/report.log
$ awk '$0 ~ "/foo/a/"' paths.txt
/foo/a/report.log
```

* *REGEXP* matching against specific field

```bash
$ # if first field contains 'a'
$ awk '$1 ~ /a/' fruits.txt 
apple   42
banana  31
guava   6

$ # if first field contains 'a' and qty > 20
$ awk '$1 ~ /a/ && $2 > 20' fruits.txt 
apple   42
banana  31

$ # if first field does NOT contain 'a'
$ awk '$1 !~ /a/' fruits.txt 
fruit   qty
fig     90
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* to search a string literally, `index` function can be used instead of *REGEXP*
    * similar to `grep -F`
* the function returns the starting position and `0` if no match found

```bash
$ cat eqns.txt
a=b,a+b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # no output since '+' is meta character, would need '/a\+b/'
$ awk '/a+b/' eqns.txt
$ # same as: grep -F 'a+b' eqns.txt
$ awk 'index($0,"a+b")' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # much easier than '/i\*\(t\+9-g\)/'
$ awk 'index($0,"i*(t+9-g)")' eqns.txt
i*(t+9-g)/8,4-a+b

$ # check only last field
$ awk -F, 'index($NF,"a+b")' eqns.txt
i*(t+9-g)/8,4-a+b
$ # index not needed if entire field/line is being compared
$ awk -F, '$1=="a+b"' eqns.txt
a+b,pi=3.14,5e12
```

* return value is useful to match at specific position
* for ex: at start/end of line

```bash
$ # start of line
$ awk 'index($0,"a+b")==1' eqns.txt
a+b,pi=3.14,5e12

$ # end of line
$ # length function returns number of characters, by default acts on $0
$ awk 'index($0,"a+b")==length()-length("a+b")+1' eqns.txt
i*(t+9-g)/8,4-a+b
$ # to avoid repetitions, save the search string in variable
$ awk -v s="a+b" 'index($0,s)==length()-length(s)+1' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

#### <a name="line-number-based-filtering"></a>Line number based filtering

* Built-in variable `NR` contains total records read so far
* Use `FNR` if you need line numbers separately for [multiple file processing](#multiple-file-processing)

```bash
$ # same as: head -n2 poem.txt | tail -n1
$ awk 'NR==2' poem.txt 
Violets are blue,

$ # print 2nd and 4th line
$ awk 'NR==2 || NR==4' poem.txt 
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
$ # statements inside END are executed after processing all input text
$ awk 'END{print}' poem.txt 
And so are you.

$ awk 'NR==4{print $2}' fruits.txt 
90
```

* for large input, use `exit` to avoid unnecessary record processing

```bash
$ seq 14323 14563435 | awk 'NR==234{print; exit}'
14556

$ # sample time comparison
$ time seq 14323 14563435 | awk 'NR==234{print; exit}'
14556

real    0m0.004s
user    0m0.004s
sys     0m0.000s
$ time seq 14323 14563435 | awk 'NR==234{print}'
14556

real    0m2.167s
user    0m2.280s
sys     0m0.092s
```

* See also [unix.stackexchange - filtering list of lines from every X number of lines](https://unix.stackexchange.com/questions/325985/how-to-print-lines-number-15-and-25-out-of-each-50-lines)

<br>

## <a name="case-insensitive-filtering"></a>Case Insensitive filtering

```bash
$ # same as: grep -i 'rose' poem.txt 
$ awk -v IGNORECASE=1 '/rose/' poem.txt 
Roses are red,

$ # for small enough set, can also use REGEXP character class
$ awk '/[rR]ose/' poem.txt 
Roses are red,

$ # another way is to use built-in string function 'tolower'
$ awk 'tolower($0) ~ /rose/' poem.txt 
Roses are red,
```

<br>

## <a name="changing-record-separators"></a>Changing record separators

* `RS` to change input record separator
* default is newline character

```bash
$ s='this is a sample string'

$ # space as input record separator, printing all records
$ printf "$s" | awk -v RS=' ' '{print NR, $0}'
1 this
2 is
3 a
4 sample
5 string

$ # print all records containing 'a'
$ printf "$s" | awk -v RS=' ' '/a/'
a
sample
```

* `ORS` to change output record separator
* gets added to every `print` statement
    * use [printf](#printf-formatting) to avoid this
* default is newline character

```bash
$ seq 3 | awk '{print $0}'
1
2
3
$ # note that there is empty line after last record
$ seq 3 | awk -v ORS='\n\n' '{print $0}'
1

2

3

$ # dynamically changing ORS
$ # can also use: seq 6 | awk '{ORS = NR%2 ? " " : RS} 1'
$ seq 6 | awk '{ORS = NR%2 ? " " : "\n"} 1'
1 2
3 4
5 6
$ seq 6 | awk '{ORS = NR%3 ? "-" : "\n"} 1'
1-2-3
4-5-6
```

<br>

#### <a name="paragraph-mode"></a>Paragraph mode

* When `RS` is set to empty string, one or more consecutive empty lines is used as input record separator
* Can also use regular expression `RS=\n\n+` but there are subtle differences, see [gawk manual - multiline records](https://www.gnu.org/software/gawk/manual/html_node/Multiple-Line.html). Important points from that link quoted below

>However, there is an important difference between ‘RS = ""’ and ‘RS = "\n\n+"’. In the first case, leading newlines in the input data file are ignored, and if a file ends without extra blank lines after the last record, the final newline is removed from the record. In the second case, this special processing is not done

>Now that the input is separated into records, the second step is to separate the fields in the records. One way to do this is to divide each of the lines into fields in the normal manner. This happens by default as the result of a special feature. When RS is set to the empty string and FS is set to a single character, the newline character always acts as a field separator. This is in addition to whatever field separations result from FS

>When FS is the null string ("") or a regexp, this special feature of RS does not apply. It does apply to the default field separator of a single space: ‘FS = " "’

Consider the below sample file

```bash
$ cat sample.txt
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

* Filtering paragraphs

```bash
$ # print all paragraphs containing 'it'
$ # if extra newline at end is undesirable, can use
$ # awk -v RS= '/it/{print c++ ? "\n" $0 : $0}' sample.txt
$ awk -v RS= -v ORS='\n\n' '/it/' sample.txt
Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

$ # based on number of lines in each paragraph
$ awk -F'\n' -v RS= -v ORS='\n\n' 'NF==1' sample.txt
Hello World

$ awk -F'\n' -v RS= -v ORS='\n\n' 'NF==2 && /do/' sample.txt
Just do-it
Believe it

Much ado about nothing
He he he

```

* Re-structuring paragraphs

```bash
$ # default FS is one or more of continuous space, tab or newline characters
$ # default OFS is single space
$ # so, $1=$1 will change it uniformly to single space between fields
$ awk -v RS= '{$1=$1} 1' sample.txt
Hello World
Good day How are you
Just do-it Believe it
Today is sunny Not a bit funny No doubt you like it too
Much ado about nothing He he he

$ # a better usecase
$ awk 'BEGIN{FS="\n"; OFS=". "; RS=""; ORS="\n\n"} {$1=$1} 1' sample.txt
Hello World

Good day. How are you

Just do-it. Believe it

Today is sunny. Not a bit funny. No doubt you like it too

Much ado about nothing. He he he

```

**Further Reading**

* [unix.stackexchange - filtering line surrounded by empty lines](https://unix.stackexchange.com/questions/359717/select-line-with-empty-line-above-and-under)
* [stackoverflow - excellent example and explanation of RS and FS](https://stackoverflow.com/questions/46142118/converting-regex-to-sed-or-grep-regex)

<br>

#### <a name="multicharacter-rs"></a>Multicharacter RS

* Some marker like `Error` or `Warning` etc

```bash
$ cat report.log 
blah blah
Error: something went wrong
more blah
whatever
Error: something surely went wrong
some text
some more text
blah blah blah

$ awk -v RS='Error:' 'END{print NR-1}' report.log
2
$ awk -v RS='Error:' 'NR==1' report.log
blah blah

$ # filter 'Error:' block matching particular string
$ # to preserve formatting, use: '/whatever/{print RS $0}'
$ awk -v RS='Error:' '/whatever/' report.log
 something went wrong
more blah
whatever

$ # blocks with more than 3 lines
$ # splitting string with 3 newlines will yield 4 fields
$ awk -F'\n' -v RS='Error:' 'NF>4{print RS $0}' report.log
Error: something surely went wrong
some text
some more text
blah blah blah

```

* Regular expression based `RS`
    * the `RT` variable will contain string matched by `RS`
* Note that entire input is treated as single string, so `^` and `$` anchors will apply only once - not every line

```bash
$ s='Sample123string54with908numbers'
$ printf "$s" | awk -v RS='[0-9]+' 'NR==1'
Sample

$ # note the relationship between record and separators
$ printf "$s" | awk -v RS='[0-9]+' '{print NR " : " $0 " - " RT}'
1 : Sample - 123
2 : string - 54
3 : with - 908
4 : numbers - 

$ # need to be careful of empty records
$ printf '123string54with908' | awk -v RS='[0-9]+' '{print NR " : " $0}'
1 : 
2 : string
3 : with
$ # and newline at end of input
$ printf '123string54with908\n' | awk -v RS='[0-9]+' '{print NR " : " $0}'
1 : 
2 : string
3 : with
4 : 

```

* Joining lines based on specific end of line condition

```bash
$ cat msg.txt
Hello there.
It will rain to-
day. Have a safe
and pleasant jou-
rney.

$ # join lines ending with - to next line
$ # by manipulating RS and ORS
$ awk -v RS='-\n' -v ORS= '1' msg.txt
Hello there.
It will rain today. Have a safe
and pleasant journey.

$ # by manipulating ORS alone, sub function covered in later sections
$ awk '{ORS = sub(/-$/,"") ? "" : "\n"} 1' msg.txt
Hello there.
It will rain today. Have a safe
and pleasant journey.
$ # easier: perl -pe 's/-\n//' msg.txt as newline is still part of input line
```

* processing null terminated input

```bash
$ printf 'foo\0bar\0' | cat -A
foo^@bar^@$ 
$ printf 'foo\0bar\0' | awk -v RS='\0' '{print}'
foo
bar
```

**Further Reading**

* [gawk manual - Records](https://www.gnu.org/software/gawk/manual/html_node/Records.html#Records)
* [unix.stackexchange - Slurp-mode in awk](https://unix.stackexchange.com/questions/304457/slurp-mode-in-awk)
* [stackoverflow - using RS to count number of occurrences of a given string](https://stackoverflow.com/questions/45102651/how-to-grep-double-quote-followed-by-a-string-at-same-time/45102962#45102962)

<br>

## <a name="substitute-functions"></a>Substitute functions

* Use `sub` string function for replacing first occurrence
* Use `gsub` for replacing all occurrences
* By default, `$0` which contains input record is modified, can specify any other field or variable as needed

```bash
$ # replacing first occurrence
$ echo '1-2-3-4-5' | awk '{sub("-", ":")} 1'
1:2-3-4-5

$ # replacing all occurrences
$ echo '1-2-3-4-5' | awk '{gsub("-", ":")} 1'
1:2:3:4:5

$ # return value for sub/gsub is number of replacements made
$ echo '1-2-3-4-5' | awk '{n=gsub("-", ":"); print n} 1'
4
1:2:3:4:5

$ # // format is better suited to specify search REGEXP
$ echo '1-2-3-4-5' | awk '{gsub(/[^-]+/, "abc")} 1'
abc-abc-abc-abc-abc

$ # replacing all occurrences only for third field
$ echo 'one;two;three;four' | awk -F';' '{gsub("e", "E", $3)} 1'
one two thrEE four
```

* Use `gensub` to return the modified string unlike `sub` or `gsub` which modifies inplace
* it also supports back-references and ability to modify specific match
* acts upon `$0` if target is not specified

```bash
$ # replace second occurrence
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(":", "-", 2)} 1'
foo:123-bar:baz
$ # use REGEXP as needed
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "XYZ", 2)} 1'
foo:XYZ:bar:baz

$ # or print the returned string directly
$ echo 'foo:123:bar:baz' | awk '{print gensub(":", "-", 2)}'
foo:123-bar:baz

$ # replace third occurrence
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "XYZ", 3)} 1'
foo:123:XYZ:baz

$ # replace all occurrences, similar to gsub
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "XYZ", "g")} 1'
XYZ:XYZ:XYZ:XYZ

$ # target other than $0
$ echo 'foo:123:bar:baz' | awk -F: -v OFS=: '{$1=gensub(/o/, "b", 2, $1)} 1'
fob:123:bar:baz
```

* back-reference examples
* use `\"` within double-quotes to represent `"` character in replacement string
* use `\\1` to represent `\1` - the first captured group and so on
* `&` or `\0` will back-reference entire matched string

```bash
$ # replacing last occurrence without knowing how many occurrences are there
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/(.*):/, "\\1-", 1)} 1'
foo:123:bar-baz
$ echo 'foo and bar and baz land good' | awk '{$0=gensub(/(.*)and/, "\\1XYZ", 1)} 1'
foo and bar and baz lXYZ good

$ # use word boundaries as necessary
$ echo 'foo and bar and baz land good' | awk '{$0=gensub(/(.*)\<and\>/, "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good

$ # replacing last but one
$ echo '456:foo:123:bar:789:baz' | awk '{$0=gensub(/(.*):(.*:)/, "\\1-\\2", 1)} 1'
456:foo:123:bar-789:baz

$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "\"&\"", "g")} 1'
"foo":"123":"bar":"baz"
```

* saving quotes in variables - to avoid escaping double quotes or having to use octal code for single quotes

```bash
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "\047&\047", "g")} 1'
'foo':'123':'bar':'baz'
$ echo 'foo:123:bar:baz' | awk -v sq="'" '{$0=gensub(/[^:]+/, sq"&"sq, "g")} 1'
'foo':'123':'bar':'baz'

$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "\"&\"", "g")} 1'
"foo":"123":"bar":"baz"
$ echo 'foo:123:bar:baz' | awk -v dq='"' '{$0=gensub(/[^:]+/, dq"&"dq, "g")} 1'
"foo":"123":"bar":"baz"
```

**Further Reading**

* [gawk manual - String-Manipulation Functions](https://www.gnu.org/software/gawk/manual/html_node/String-Functions.html)
* [gawk manual - escape processing](https://www.gnu.org/software/gawk/manual/html_node/Gory-Details.html)

<br>

## <a name="inplace-file-editing"></a>Inplace file editing

* Use this option with caution, preferably after testing that the `awk` code is working as intended

```bash
$ cat greeting.txt 
Hi there
Have a nice day

$ awk -i inplace '{gsub("e", "E")} 1' greeting.txt 
$ cat greeting.txt 
Hi thErE
HavE a nicE day
```

* Multiple input files are treated individually and changes are written back to respective files

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ awk -i inplace '{gsub("3", "three")} 1' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

* to create backups of original file, set `INPLACE_SUFFIX` variable

```bash
$ awk -i inplace -v INPLACE_SUFFIX='.bkp' '{gsub("three", "3")} 1' f1
$ cat f1
I ate 3 apples
$ cat f1.bkp
I ate three apples
```

* See [gawk manual - Enabling In-Place File Editing](https://www.gnu.org/software/gawk/manual/html_node/Extension-Sample-Inplace.html) for implementation details

<br>

## <a name="using-shell-variables"></a>Using shell variables

* when `awk` code is part of shell program and shell variable needs to be passed as input to `awk` code
* for example:
    * command line argument passed to shell script, which is in turn passed on to `awk`
    * control structures in shell script calling `awk` with different search strings
* See also [stackoverflow - How do I use shell variables in an awk script?](https://stackoverflow.com/questions/19075671/how-do-i-use-shell-variables-in-an-awk-script)

```bash
$ # examples tested with bash shell

$ f='apple'
$ awk -v word="$f" '$1==word' fruits.txt
apple   42
$ f='fig'
$ awk -v word="$f" '$1==word' fruits.txt
fig     90

$ q='20'
$ awk -v threshold="$q" 'NR==1 || $2>threshold' fruits.txt
fruit   qty
apple   42
banana  31
fig     90
```

* accessing shell environment variables

```bash
$ # existing environment variable
$ awk 'BEGIN{print ENVIRON["PWD"]}'
/home/learnbyexample
$ awk 'BEGIN{print ENVIRON["SHELL"]}'
/bin/bash

$ # defined along with awk code
$ word='hello world' awk 'BEGIN{print ENVIRON["word"]}'
hello world

$ # using ENVIRON also prevents awk's interpretation of escape sequences
$ s='a\n=c'
$ foo="$s" awk 'BEGIN{print ENVIRON["foo"]}'
a\n=c
$ awk -v foo="$s" 'BEGIN{print foo}'
a
=c
```

* passing *REGEXP*
* See also [gawk manual - Using Dynamic Regexps](https://www.gnu.org/software/gawk/manual/html_node/Computed-Regexps.html)

```bash
$ s='are'
$ # for: awk '!/are/' poem.txt
$ awk -v s="$s" '$0 !~ s' poem.txt
Sugar is sweet,
$ # for: awk '/are/ && !/so/' poem.txt
$ awk -v s="$s" '$0 ~ s && !/so/' poem.txt
Roses are red,
Violets are blue,

$ r='[^-]+'
$ echo '1-2-3-4-5' | awk -v r="$r" '{gsub(r, "abc")} 1'
abc-abc-abc-abc-abc

$ # escape sequence has to be doubled when string is interpreted as REGEXP
$ s='foo and bar and baz land good'
$ echo "$s" | awk '{$0=gensub("(.*)\\<and\\>", "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good
$ # hence passing as variable should be
$ r='(.*)\\<and\\>'
$ echo "$s" | awk -v r="$r" '{$0=gensub(r, "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good

$ # or use ENVIRON
$ r='(.*)\<and\>'
$ echo "$s" | r="$r" awk '{$0=gensub(ENVIRON["r"], "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good
```

<br>

## <a name="multiple-file-input"></a>Multiple file input

* Example to show difference between `NR` and `FNR`

```bash
$ # NR for overall record number
$ awk 'NR==1' poem.txt greeting.txt 
Roses are red,

$ # FNR for individual file's record number
$ # same as: head -q -n1 poem.txt greeting.txt
$ awk 'FNR==1' poem.txt greeting.txt 
Roses are red,
Hi thErE
```

* Constructs to do some processing before starting each file as well as at the end
* `BEGINFILE` - to add code to be executed before start of each input file
* `ENDFILE` - to add code to be executed after processing each input file
* `FILENAME` - file name of current input file being processed

```bash
$ # similar to: tail -n1 poem.txt greeting.txt
$ awk 'BEGINFILE{print "file: "FILENAME}
       ENDFILE{print $0"\n------"}' poem.txt greeting.txt
file: poem.txt
And so are you.
------
file: greeting.txt
HavE a nicE day
------
```

* And of course, there can be usual `awk` code

```bash
$ awk 'BEGINFILE{print "file: "FILENAME}
       FNR==1;
       ENDFILE{print "------"}' poem.txt greeting.txt
file: poem.txt
Roses are red,
------
file: greeting.txt
Hi thErE
------

$ awk 'BEGINFILE{c++; print "file: "FILENAME}
       FNR==2;
       END{print "\nTotal input files: "c}' poem.txt greeting.txt
file: poem.txt
Violets are blue,
file: greeting.txt
HavE a nicE day

Total input files: 2
```

**Further Reading**

* [gawk manual - Using ARGC and ARGV](https://www.gnu.org/software/gawk/manual/html_node/ARGC-and-ARGV.html)
* [gawk manual - ARGIND](https://www.gnu.org/software/gawk/manual/html_node/Auto_002dset.html#index-ARGIND-variable)
* [gawk manual - ERRNO](https://www.gnu.org/software/gawk/manual/html_node/Auto_002dset.html#index-ERRNO-variable)
* [stackoverflow - Finding common value across multiple files](https://stackoverflow.com/a/43473385/4082052)

<br>

## <a name="control-structures"></a>Control Structures

* Syntax is similar to `C` language and single statements inside control structures don't require to be grouped within `{}`
* See [gawk manual - Control Statements](https://www.gnu.org/software/gawk/manual/html_node/Statements.html) for details

Remember that by default there is a loop that goes over all input records and constructs like `BEGIN` and `END` fall outside that loop

```bash
$ cat nums.txt 
42
-2
10101
-3.14
-75
$ awk '{sum += $1} END{print sum}' nums.txt
10062.9

$ # uninitialized variables will have empty string
$ printf '' | awk '{sum += $1} END{print sum}'

$ # so either add '0' or use unary '+' operator to convert to number
$ printf '' | awk '{sum += $1} END{print +sum}'
0
```

<br>

#### <a name="if-else-and-loops"></a>if-else and loops

* We have already seen simple `if` examples in [Filtering](#filtering) section
* See also [gawk manual - Switch](https://www.gnu.org/software/gawk/manual/html_node/Switch-Statement.html)

```bash
$ # same as: sed -n '/are/ s/so/SO/p' poem.txt 
$ # remember that sub/gsub returns number of substitutions made
$ awk '/are/{if(sub("so", "SO")) print}' poem.txt
And SO are you.
$ # of course, can also use
$ awk '/are/ && sub("so", "SO")' poem.txt
And SO are you.

$ # if-else example
$ awk 'NR>1{if($2>40) $0="+"$0; else $0="-"$0} 1' fruits.txt
fruit   qty
+apple   42
-banana  31
+fig     90
-guava   6
```

* conditional operator
* See also [stackoverflow - finding min and max value of a column](https://stackoverflow.com/a/29784278/4082052)

```bash
$ cat nums.txt 
42
-2
10101
-3.14
-75

$ # changing -ve to +ve and vice versa
$ # same as: awk '{if($0 ~ /^-/) sub(/^-/,""); else sub(/^/,"-")} 1' nums.txt
$ awk '{$0 ~ /^-/ ? sub(/^-/,"") : sub(/^/,"-")} 1' nums.txt
-42
2
-10101
3.14
75
$ # can also use: awk '!sub(/^-/,""){sub(/^/,"-")} 1' nums.txt
```

* for loop
* similar to `C` language, `break` and `continue` statements are also available
* See also [stackoverflow - find missing numbers from sequential list](https://stackoverflow.com/questions/38491676/how-can-i-find-the-missing-integers-in-a-unique-and-sequential-list-one-per-lin)

```bash
$ awk 'BEGIN{for(i=2; i<11; i+=2) print i}'
2
4
6
8
10

$ # looping each field
$ s='scat:cat:no cat:abdicate:cater'
$ echo "$s" | awk -F: -v OFS=: '{for(i=1;i<=NF;i++) if($i=="cat") $i="CAT"} 1'
scat:CAT:no cat:abdicate:cater
$ # can also use sub function
$ echo "$s" | awk -F: -v OFS=: '{for(i=1;i<=NF;i++) sub(/^cat$/,"CAT",$i)} 1'
scat:CAT:no cat:abdicate:cater
```

* while loop
* do-while is also available

```bash
$ awk 'BEGIN{i=2; while(i<11){print i; i+=2}}'
2
4
6
8
10

$ # recursive substitution
$ # here again return value of sub/gsub is useful
$ echo 'titillate' | awk '{while( gsub(/til/, "") ) print}'
tilate
ate
```

<br>

#### <a name="next-and-nextfile"></a>next and nextfile

* `next` will skip rest of statements and start processing next line of current file being processed
    * there is a loop by default which goes over all input records, `next` is applicable for that
    * it is similar to `continue` statement within loops
* it is often used in [Two file processing](#two-file-processing)

```bash
$ # here 'next' is used to skip processing header line
$ awk 'NR==1{print; next} /a.*a/{$0="*"$0} /[eiou]/{$0="-"$0} 1' fruits.txt
fruit   qty
-apple   42
*banana  31
-fig     90
-*guava   6
```

* `nextfile` is useful to skip remaining lines from current file being processed and move on to next file

```bash
$ # same as: head -q -n1 poem.txt greeting.txt fruits.txt
$ awk 'FNR>1{nextfile} 1' poem.txt greeting.txt fruits.txt
Roses are red,
Hi thErE
fruit   qty

$ # specific field
$ awk 'FNR>2{nextfile} {print $1}' poem.txt greeting.txt fruits.txt
Roses
Violets
Hi
HavE
fruit
apple

$ # similar to 'grep -il'
$ awk -v IGNORECASE=1 '/red/{print FILENAME; nextfile}' *
colors_1.txt
colors_2.txt
poem.txt
$ awk -v IGNORECASE=1 '$1 ~ /red/{print FILENAME; nextfile}' *
colors_1.txt
colors_2.txt
```

<br>

## <a name="multiline-processing"></a>Multiline processing

* Processing consecutive lines

```bash
$ cat poem.txt 
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # match two consecutive lines
$ awk 'p~/are/ && /is/{print p ORS $0} {p=$0}' poem.txt 
Violets are blue,
Sugar is sweet,
$ # if only the second line is needed
$ awk 'p~/are/ && /is/; {p=$0}' poem.txt 
Sugar is sweet,

$ # match three consecutive lines
$ awk 'p2~/red/ && p1~/blue/ && /is/{print p2} {p2=p1; p1=$0}' poem.txt
Roses are red,

$ # common mistake
$ sed -n '/are/{N;/is/p}' poem.txt 
$ # would need something like this and not practical to extend for other cases
$ sed '$!N; /are.*\n.*is/p; D' poem.txt 
Violets are blue,
Sugar is sweet,
```

Consider this sample input file

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

* extracting lines around matching line
* See also [stackoverflow - lines around matching regexp](https://stackoverflow.com/questions/17908555/printing-with-sed-or-awk-a-line-following-a-matching-pattern)
* how `n && n--` works:
    * need to note that right hand side of `&&` is processed only if left hand side is `true`
    * so for example, if initially `n=2`, then we get
        * `2 && 2; n=1` - evaluates to `true`
        * `1 && 1; n=0` - evaluates to `true`
        * `0 && ` - evaluates to `false` ... no decrementing `n` and hence will be `false` until `n` is re-assigned non-zero value

```bash
$ # similar to: grep --no-group-separator -A1 'BEGIN' range.txt 
$ awk '/BEGIN/{n=2} n && n--' range.txt
BEGIN
1234
BEGIN
a

$ # only print the line after matching line
$ # can also use: awk '/BEGIN/{n=1; next} n && n--' range.txt
$ awk 'n && n--; /BEGIN/{n=1}' range.txt 
1234
a
$ # generic case: print nth line after match
$ awk 'n && !--n; /BEGIN/{n=3}' range.txt
END
c

$ # print second line prior to matched line
$ awk '/END/{print p2} {p2=p1; p1=$0}' range.txt
1234
b
$ # save all lines in an array for generic case
$ awk '/END/{print a[NR-3]} {a[NR]=$0}' range.txt
BEGIN
a
$ # or use the reversing trick
$ tac range.txt | awk 'n && !--n; /END/{n=3}' | tac
BEGIN
a
```

* Checking if multiple strings are present at least once in entire input file
* If there are lots of strings to check, use arrays

```bash
$ # can also use BEGINFILE instead of FNR==1
$ awk 'FNR==1{s1=s2=0} /is/{s1=1} /are/{s2=1} s1&&s2{print FILENAME; nextfile}' *
poem.txt
sample.txt

$ awk 'FNR==1{s1=s2=0} /foo/{s1=1} /report/{s2=1} s1&&s2{print FILENAME; nextfile}' *
paths.txt
```

<br>

## <a name="two-file-processing"></a>Two file processing

* We'll use awk's associative arrays (key-value pairs) here
    * key can be number or string
    * See also [gawk manual - Arrays](https://www.gnu.org/software/gawk/manual/html_node/Arrays.html)
* Unlike [comm](./sorting_stuff.md#comm) the input files need not be sorted and comparison can be done based on certain field(s) as well

<br>

#### <a name="comparing-whole-lines"></a>Comparing whole lines

Consider the following test files

```bash
$ cat colors_1.txt
Blue
Brown
Purple
Red
Teal
Yellow

$ cat colors_2.txt
Black
Blue
Green
Red
White
```

* common lines and lines unique to one of the files
* For two files as input, `NR==FNR` will be true only when first file is being processed
* Using `next` will skip rest of code when first file is processed
* `a[$0]` will create unique keys (here entire line content is used as key) in array `a`
    * just referencing a key will create it if it doesn't already exist, with value as empty string (will also act as zero in numeric context)
* `$0 in a` will be true if key already exists in array `a`

```bash
$ # common lines
$ # same as: grep -Fxf colors_1.txt colors_2.txt
$ awk 'NR==FNR{a[$0]; next} $0 in a' colors_1.txt colors_2.txt
Blue
Red

$ # lines from colors_2.txt not present in colors_1.txt
$ # same as: grep -vFxf colors_1.txt colors_2.txt
$ awk 'NR==FNR{a[$0]; next} !($0 in a)' colors_1.txt colors_2.txt
Black
Green
White

$ # reversing the order of input files gives
$ # lines from colors_1.txt not present in colors_2.txt
$ awk 'NR==FNR{a[$0]; next} !($0 in a)' colors_2.txt colors_1.txt
Brown
Purple
Teal
Yellow
```

<br>

#### <a name="comparing-specific-fields"></a>Comparing specific fields

Consider the sample input file

```bash
$ cat marks.txt
Dept    Name    Marks
ECE     Raj     53
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
EEE     Tia     59
ECE     Om      92
CSE     Amy     67
```

* single field
* For ex: only first field comparison by using `$1` instead of `$0` as key

```bash
$ cat list1
ECE
CSE

$ # extract only lines matching first field specified in list1
$ awk 'NR==FNR{a[$1]; next} $1 in a' list1 marks.txt
ECE     Raj     53
ECE     Joel    72
CSE     Surya   81
ECE     Om      92
CSE     Amy     67

$ # if header is needed as well
$ awk 'NR==FNR{a[$1]; next} FNR==1 || $1 in a' list1 marks.txt
Dept    Name    Marks
ECE     Raj     53
ECE     Joel    72
CSE     Surya   81
ECE     Om      92
CSE     Amy     67
```

* multiple fields
* create a string by adding some character between the fields to act as key
    * for ex: to avoid matching two field values `abc` and `123` to match with two other field values `ab` and `c123`
    * by adding character, say `_`, the key would be `abc_123` for first case and `ab_c123` for second case
    * this can still lead to false match if input data has `_`
    * there is also a built-in way to do this using [gawk manual - Multidimensional Arrays](https://www.gnu.org/software/gawk/manual/html_node/Multidimensional.html#Multidimensional)

```bash
$ cat list2
EEE Moi
CSE Amy
ECE Raj

$ # extract only lines matching both fields specified in list2
$ awk 'NR==FNR{a[$1"_"$2]; next} $1"_"$2 in a' list2 marks.txt
ECE     Raj     53
EEE     Moi     68
CSE     Amy     67

$ # uses SUBSEP as separator, whose default value is non-printing character \034
$ awk 'NR==FNR{a[$1,$2]; next} ($1,$2) in a' list2 marks.txt
ECE     Raj     53
EEE     Moi     68
CSE     Amy     67
```

* field and value comparison

```bash
$ cat list3
ECE 70
EEE 65
CSE 80

$ # extract line matching Dept and minimum marks specified in list3
$ awk 'NR==FNR{d[$1]; m[$1]=$2; next} $1 in d && $3 >= m[$1]' list3 marks.txt
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
ECE     Om      92
```

<br>

#### <a name="getline"></a>getline

* If entire line (instead of fields) from one file is needed to change the other file, using `getline` would be faster
* But use it with caution
    * [gawk manual - getline](https://www.gnu.org/software/gawk/manual/html_node/Getline.html) for details, especially about corner cases, errors, etc
    * [gawk manual - Closing Input and Output Redirections](https://www.gnu.org/software/gawk/manual/html_node/Close-Files-And-Pipes.html) if you have to start from beginning of file again

```bash
$ # replace mth line in poem.txt with nth line from nums.txt
$ awk -v m=3 -v n=2 'BEGIN{while(n-- > 0) getline s < "nums.txt"}
                     FNR==m{$0=s} 1' poem.txt
Roses are red,
Violets are blue,
-2
And so are you.

$ # without getline, but slower due to NR==FNR check for every line processed
$ awk -v m=3 -v n=2 'NR==FNR{if(FNR==n){s=$0; nextfile} next}
                     FNR==m{$0=s} 1' nums.txt poem.txt
Roses are red,
Violets are blue,
-2
And so are you.
```

* Another use case is if two files are to be processed exactly for same line numbers

```bash

$ # print line from fruits.txt if corresponding line from nums.txt is +ve number
$ awk -v file='nums.txt' '{getline num < file; if(num>0) print}' fruits.txt
fruit   qty
banana  31

$ # without getline, but has to save entire file in array
$ awk 'NR==FNR{n[FNR]=$0; next} n[FNR]>0' nums.txt fruits.txt
fruit   qty
banana  31
```

**Further Reading**

* [stackoverflow - Fastest way to find lines of a text file from another larger text file](https://stackoverflow.com/questions/42239179/fastest-way-to-find-lines-of-a-text-file-from-another-larger-text-file-in-bash)
* [unix.stackexchange - filter lines based on line numbers specified in another file](https://unix.stackexchange.com/questions/320651/read-numbers-from-control-file-and-extract-matching-line-numbers-from-the-data-f)
* [stackoverflow - three file processing to extract a matrix subset](https://stackoverflow.com/questions/45036019/how-to-filter-the-values-from-selected-columns-and-rows)
* [unix.stackexchange - column wise merging](https://unix.stackexchange.com/questions/294145/merging-two-files-one-column-at-a-time)
* [stackoverflow - extract specific rows from a text file using an index file](https://stackoverflow.com/questions/40595990/print-many-specific-rows-from-a-text-file-using-an-index-file)

<br>

## <a name="creating-new-fields"></a>Creating new fields

* Number of fields in input record can be changed by simply manipulating `NF`

```bash
$ # reducing fields
$ echo 'foo,bar,123,baz' | awk -F, -v OFS=, '{NF=2} 1'
foo,bar

$ # creating new empty field(s)
$ echo 'foo,bar,123,baz' | awk -F, -v OFS=, '{NF=5} 1'
foo,bar,123,baz,

$ # assigning to field greater than NF will create empty fields as needed
$ echo 'foo,bar,123,baz' | awk -F, -v OFS=, '{$7=42} 1'
foo,bar,123,baz,,,42

$ # adding a new 'Grade' field
$ awk 'BEGIN{OFS="\t"; g[9]="S"; g[8]="A"; g[7]="B"; g[6]="C"; g[5]="D"}
      {NF++; if(NR==1)$NF="Grade"; else $NF=g[int($(NF-1)/10)]} 1' marks.txt
Dept    Name    Marks   Grade
ECE     Raj     53      D
ECE     Joel    72      B
EEE     Moi     68      C
CSE     Surya   81      A
EEE     Tia     59      D
ECE     Om      92      S
CSE     Amy     67      C
```

* two file example

```bash
$ cat list4
Raj class_rep
Amy sports_rep
Tia placement_rep

$ awk -v OFS='\t' 'NR==FNR{r[$1]=$2; next}
         {NF++; if(FNR==1)$NF="Role"; else $NF=r[$2]} 1' list4 marks.txt
Dept    Name    Marks   Role
ECE     Raj     53      class_rep
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
EEE     Tia     59      placement_rep
ECE     Om      92
CSE     Amy     67      sports_rep
```

<br>

## <a name="dealing-with-duplicates"></a>Dealing with duplicates

* default value of uninitialized variable is `0` in numeric context and empty string in text context
    * and evaluates to `false` when used conditionally

*Illustration to show default numeric value and array in action*

```bash
$ printf 'mad\n42\n42\ndam\n42\n'
mad
42
42
dam
42

$ printf 'mad\n42\n42\ndam\n42\n' | awk '{print $0 "\t" int(a[$0]); a[$0]++}'
mad     0
42      0
42      1
dam     0
42      2
$ # only those entries with second column value zero will be retained
$ printf 'mad\n42\n42\ndam\n42\n' | awk '!a[$0]++'
mad
42
dam
```

* first, examples that retain only first copy of duplicates

```bash
$ cat duplicates.txt
abc  7   4
food toy ****
abc  7   4
test toy 123
good toy ****

$ # whole line
$ awk '!seen[$0]++' duplicates.txt
abc  7   4
food toy ****
test toy 123
good toy ****

$ # particular column
$ awk '!seen[$2]++' duplicates.txt
abc  7   4
food toy ****

$ # total count
$ awk '!seen[$2]++{c++} END{print +c}' duplicates.txt
2
```

* if input is so large that integer numbers can overflow
* See also [gawk manual - Arbitrary-Precision Integer Arithmetic](https://www.gnu.org/software/gawk/manual/html_node/Arbitrary-Precision-Integers.html)

```bash
$ # avoid unnecessary counting altogether
$ awk '!($2 in seen); {seen[$2]}' duplicates.txt
abc  7   4
food toy ****

$ # use arbitrary-precision integers, limited only by available memory
$ awk -M '!($2 in seen){c++} {seen[$2]} END{print +c}' duplicates.txt
2
```

* For multiple fields, separate them using `,` or form a string with some character in between
    * choose a character unlikely to appear in input data, else there can be false matches

```bash
$ awk '!seen[$2"_"$3]++' duplicates.txt
abc  7   4
food toy ****
test toy 123

$ # can also use simulated multidimensional array
$ # SUBSEP, whose default is \034 non-printing character, is used as separator
$ awk '!seen[$2,$3]++' duplicates.txt
abc  7   4
food toy ****
test toy 123
```

* retaining specific numbered copy

```bash
$ # second occurrence of duplicate
$ awk '++seen[$2]==2' duplicates.txt
abc  7   4
test toy 123

$ # third occurrence of duplicate
$ awk '++seen[$2]==3' duplicates.txt
good toy ****
```

* retaining only last copy of duplicate

```bash
$ # reverse the input line-wise, retain first copy and then reverse again
$ tac duplicates.txt | awk '!seen[$2]++' | tac
abc  7   4
good toy ****
```

* filtering based on duplicate count
* allows to emulate [uniq](./sorting_stuff.md#uniq) command for specific fields
* See also [unix.stackexchange - retain only parent directory paths](https://unix.stackexchange.com/questions/362571/filter-out-paths-from-a-text-file-that-are-deeper-than-their-immediate-predecces)

```bash
$ # all duplicates based on 1st column
$ awk 'NR==FNR{a[$1]++; next} a[$1]>1' duplicates.txt duplicates.txt
abc  7   4
abc  7   4
$ # all duplicates based on 3rd column
$ awk 'NR==FNR{a[$3]++; next} a[$3]>1' duplicates.txt duplicates.txt
abc  7   4
food toy ****
abc  7   4
good toy ****

$ # more than 2 duplicates based on 2nd column
$ awk 'NR==FNR{a[$2]++; next} a[$2]>2' duplicates.txt duplicates.txt
food toy ****
test toy 123
good toy ****

$ # only unique lines based on 3rd column
$ awk 'NR==FNR{a[$3]++; next} a[$3]==1' duplicates.txt duplicates.txt
test toy 123
```

<br>

## <a name="lines-between-two-regexps"></a>Lines between two REGEXPs

* This section deals with filtering lines bound by two *REGEXP*s (referred to as blocks)
* For simplicity the two *REGEXP*s usually used in below examples are the strings **BEGIN** and **END**

<br>

#### <a name="all-unbroken-blocks"></a>All unbroken blocks

Consider the below sample input file, which doesn't have any unbroken blocks (i.e **BEGIN** and **END** are always present in pairs)

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

* Extracting lines between starting and ending *REGEXP*

```bash
$ # include both starting/ending REGEXP
$ # can also use: awk '/BEGIN/,/END/' range.txt
$ # which is similar to sed -n '/BEGIN/,/END/p'
$ # but not suitable to extend for other cases
$ awk '/BEGIN/{f=1} f; /END/{f=0}' range.txt
BEGIN
1234
6789
END
BEGIN
a
b
c
END

$ # exclude both starting/ending REGEXP
$ # can also use: awk '/BEGIN/{f=1; next} /END/{f=0} f' range.txt
$ awk '/END/{f=0} f; /BEGIN/{f=1}' range.txt
1234
6789
a
b
c
```

* Include only start or end *REGEXP*

```bash
$ # include only starting REGEXP
$ awk '/BEGIN/{f=1} /END/{f=0} f' range.txt
BEGIN
1234
6789
BEGIN
a
b
c

$ # include only ending REGEXP
$ awk 'f; /END/{f=0} /BEGIN/{f=1}' range.txt
1234
6789
END
a
b
c
END
```

* Extracting lines other than lines between the two *REGEXP*s

```bash
$ awk '/BEGIN/{f=1} !f; /END/{f=0}' range.txt
foo
bar
baz

$ # the other three cases would be
$ awk '/END/{f=0} !f; /BEGIN/{f=1}' range.txt
$ awk '!f; /BEGIN/{f=1} /END/{f=0}' range.txt 
$ awk '/BEGIN/{f=1} /END/{f=0} !f' range.txt
```

<br>

#### <a name="specific-blocks"></a>Specific blocks

* Getting first block

```bash
$ awk '/BEGIN/{f=1} f; /END/{exit}' range.txt 
BEGIN
1234
6789
END

$ # use other tricks discussed in previous section as needed
$ awk '/END/{exit} f; /BEGIN/{f=1}' range.txt
1234
6789
```

* Getting last block

```bash
$ # reverse input linewise, change the order of REGEXPs, finally reverse again
$ tac range.txt | awk '/END/{f=1} f; /BEGIN/{exit}' | tac
BEGIN
a
b
c
END

$ # or, save the blocks in a buffer and print the last one alone
$ # ORS contains output record separator, which is newline by default
$ seq 30 | awk '/4/{f=1; b=$0; next} f{b=b ORS $0} /6/{f=0} END{print b}'
24
25
26
```

* Getting blocks based on a counter

```bash
$ # all blocks
$ seq 30 | sed -n '/4/,/6/p'
4
5
6
14
15
16
24
25
26

$ # get only 2nd block
$ # can also use: seq 30 | awk -v b=2 '/4/{c++} c==b{print; if(/6/) exit}'
$ seq 30 | awk -v b=2 '/4/{c++} c==b; /6/ && c==b{exit}'
14
15
16

$ # to get all blocks greater than 'b' blocks
$ seq 30 | awk -v b=1 '/4/{f=1; c++} f && c>b; /6/{f=0}'
14
15
16
24
25
26
```

* excluding a particular block

```bash
$ # excludes 2nd block
$ seq 30 | awk -v b=2 '/4/{f=1; c++} f && c!=b; /6/{f=0}'
4
5
6
24
25
26
```

<br>

#### <a name="broken-blocks"></a>Broken blocks

* If there are blocks with ending *REGEXP* but without corresponding start, `awk '/BEGIN/{f=1} f; /END/{f=0}'` will suffice
* Consider the modified input file where starting *REGEXP* doesn't have corresponding ending

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

$ # the file reversing trick comes in handy here as well
$ tac broken_range.txt | awk '/END/{f=1} f; /BEGIN/{f=0}' | tac
BEGIN
1234
6789
END
```

* But if both kinds of broken blocks are present, accumulate the records and print accordingly

```bash
$ cat multiple_broken.txt 
qqqqqqq
BEGIN
foo
BEGIN
1234
6789
END
bar
END
0-42-1
BEGIN
a
BEGIN
b
END
;as;s;sd;

$ awk '/BEGIN/{f=1; buf=$0; next}
       f{buf=buf ORS $0}
       /END/{f=0; if(buf) print buf; buf=""}' multiple_broken.txt
BEGIN
1234
6789
END
BEGIN
b
END
```

**Further Reading**

* [stackoverflow - select lines between two regexps](https://stackoverflow.com/questions/38972736/how-to-select-lines-between-two-patterns)
* [unix.stackexchange - print only blocks with lines > n](https://unix.stackexchange.com/questions/295600/deleting-lines-between-rows-in-a-text-file-using-awk-or-sed)
* [unix.stackexchange - print a block only if it contains matching string](https://unix.stackexchange.com/a/335523/109046)
* [unix.stackexchange - print a block matching two different strings](https://unix.stackexchange.com/questions/347368/grep-with-range-and-pass-three-filters)

<br>

## <a name="arrays"></a>Arrays

We've already seen examples using arrays, some more examples discussed in this section

* array looping

```bash
$ # average marks for each department
$ awk 'NR>1{d[$1]+=$3; c[$1]++} END{for(i in d)print i, d[i]/c[i]}' marks.txt
ECE 72.3333
EEE 63.5
CSE 74
```

* Sorting
* See [gawk manual - Predefined Array Scanning Orders](https://www.gnu.org/software/gawk/manual/html_node/Controlling-Scanning.html#Controlling-Scanning) for more details

```bash
$ # by default, keys are traversed in random order
$ awk 'BEGIN{a["z"]=1; a["x"]=12; a["b"]=42; for(i in a)print i, a[i]}'
x 12
z 1
b 42

$ # index sorted ascending order as strings
$ awk 'BEGIN{PROCINFO["sorted_in"] = "@ind_str_asc";
       a["z"]=1; a["x"]=12; a["b"]=42; for(i in a)print i, a[i]}'
b 42
x 12
z 1

$ # value sorted ascending order as numbers
$ awk 'BEGIN{PROCINFO["sorted_in"] = "@val_num_asc";
       a["z"]=1; a["x"]=12; a["b"]=42; for(i in a)print i, a[i]}'
z 1
x 12
b 42
```

* deleting array elements

```bash
$ cat list5
CSE     Surya   75
EEE     Jai     69
ECE     Kal     83

$ # update entry if a match is found
$ # else append the new entries
$ awk '{ky=$1"_"$2} NR==FNR{upd[ky]=$0; next}
        ky in upd{$0=upd[ky]; delete upd[ky]} 1;
        END{for(i in upd)print upd[i]}' list5 marks.txt
Dept    Name    Marks
ECE     Raj     53
ECE     Joel    72
EEE     Moi     68
CSE     Surya   75
EEE     Tia     59
ECE     Om      92
CSE     Amy     67
ECE     Kal     83
EEE     Jai     69
```

* true multidimensional arrays
* length of sub-arrays need not be same. See [gawk manual - Arrays of Arrays](https://www.gnu.org/software/gawk/manual/html_node/Arrays-of-Arrays.html#Arrays-of-Arrays) for details

```bash
$ awk 'NR>1{d[$1][$2]=$3} END{for(i in d["ECE"])print i}' marks.txt
Joel
Raj
Om

$ awk -v f='CSE' 'NR>1{d[$1][$2]=$3} END{for(i in d[f])print i, d[f][i]}' marks.txt
Surya 81
Amy 67
```

**Further Reading**

* [gawk manual - all array topics](https://www.gnu.org/software/gawk/manual/html_node/Arrays.html)
* [unix.stackexchange - count words based on length](https://unix.stackexchange.com/questions/396855/is-there-an-easy-way-to-count-characters-in-words-in-file-from-terminal)
* [unix.stackexchange - filtering specific lines](https://unix.stackexchange.com/a/326215/109046)

<br>

## <a name="awk-scripts"></a>awk scripts

* For larger programs, save the code in a file and use `-f` command line option
* `;` is not needed to terminate a statement
* See also [gawk manual - Command-Line Options](https://www.gnu.org/software/gawk/manual/html_node/Options.html#Options) for other related options

```bash
$ cat buf.awk
/BEGIN/{
    f=1
    buf=$0
    next
}

f{
    buf=buf ORS $0
}

/END/{
    f=0
    if(buf)
        print buf
    buf=""
}

$ awk -f buf.awk multiple_broken.txt 
BEGIN
1234
6789
END
BEGIN
b
END
```

* Another advantage is that single quotes can be freely used

```bash
$ echo 'foo:123:bar:baz' | awk '{$0=gensub(/[^:]+/, "\047&\047", "g")} 1'
'foo':'123':'bar':'baz'

$ cat quotes.awk
{
    $0 = gensub(/[^:]+/, "'&'", "g")
}

1

$ echo 'foo:123:bar:baz' | awk -f quotes.awk
'foo':'123':'bar':'baz'
```

* If the code has been first tried out on command line, add `-o` option to get a pretty printed version

```bash
$ awk -o -v OFS='\t' 'NR==FNR{r[$1]=$2; next}
         {NF++; if(FNR==1)$NF="Role"; else $NF=r[$2]} 1' list4 marks.txt
Dept    Name    Marks   Role
ECE     Raj     53      class_rep
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
EEE     Tia     59      placement_rep
ECE     Om      92
CSE     Amy     67      sports_rep
```

File name can be passed along `-o` option, otherwise by default `awkprof.out` will be used

```bash
$ cat awkprof.out
        # gawk profile, created Tue Oct 24 15:10:02 2017

        # Rule(s)

        NR == FNR {
                r[$1] = $2
                next
        }

        {
                NF++
                if (FNR == 1) {
                        $NF = "Role"
                } else {
                        $NF = r[$2]
                }
        }

        1 {
                print $0
        }

$ # note that other command line options have to be provided as usual
$ # for ex: awk -v OFS='\t' -f awkprof.out list4 marks.txt
```

<br>

## <a name="miscellaneous"></a>Miscellaneous

<br>

#### <a name="fpat-and-fieldwidths"></a>FPAT and FIELDWIDTHS

* `FS` allows to define field separator
* In contrast, `FPAT` allows to define what should the fields be made up of
* See also [gawk manual - Defining Fields by Content](https://www.gnu.org/software/gawk/manual/html_node/Splitting-By-Content.html)

```bash
$ s='Sample123string54with908numbers'
$ # define fields to be one or more consecutive digits
$ echo "$s" | awk -v FPAT='[0-9]+' '{print $1, $2, $3}'
123 54 908
$ # define fields to be one or more consecutive alphabets
$ echo "$s" | awk -v FPAT='[a-zA-Z]+' '{print $1, $2, $3, $4}'
Sample string with numbers
```

* For simpler **csv** input having quoted strings if fields themselves have `,` in them, using `FPAT` is reasonable approach
* Use a proper parser if input can have other cases like newlines in fields
    * See [unix.stackexchange - using csv parser](https://unix.stackexchange.com/a/238192) for a sample program in `perl`

```bash
$ s='foo,"bar,123",baz,abc'
$ echo "$s" | awk -F, '{print $2}'
"bar
$ echo "$s" | awk -v FPAT='"[^"]*"|[^,]*' '{print $2}'
"bar,123"
```

* if input has well defined fields based on number of characters, `FIELDWIDTHS` can be used to specify width of each field

```bash
$ awk -v FIELDWIDTHS='8 3' -v OFS= '/fig/{$2=35} 1' fruits.txt
fruit   qty
apple   42
banana  31
fig     35
guava   6

$ # without FIELDWIDTHS
$ awk '/fig/{$2=35} 1' fruits.txt
fruit   qty
apple   42
banana  31
fig 35
guava   6
```

**Further Reading**

* [gawk manual - Processing Fixed-Width Data](https://www.gnu.org/software/gawk/manual/html_node/Fixed-width-data.html)
* [unix.stackexchange - Modify records in fixed-width files](https://unix.stackexchange.com/questions/368574/modify-records-in-fixed-width-files)
* [unix.stackexchange - detecting empty fields in fixed width files](https://unix.stackexchange.com/questions/321559/extracting-data-with-awk-when-some-lines-have-empty-missing-values)
* [stackoverflow - count number of times value is repeated each line](https://stackoverflow.com/questions/37450880/how-do-i-filter-tab-separated-input-by-the-count-of-fields-with-a-given-value)

<br>

#### <a name="string-functions"></a>String functions

* `length` function - returns length of string, by default acts on `$0`

```bash
$ seq 8 13 | awk 'length()==1'
8
9

$ awk 'NR==1 || length($1)>4' fruits.txt
fruit   qty
apple   42
banana  31
guava   6

$ # character count and not byte count is calculated, similar to 'wc -m'
$ printf 'hi👍' | awk '{print length()}'
3

$ # use -b option if number of bytes are needed
$ printf 'hi👍' | awk -b '{print length()}'
6
```

* `split` function - similar to `FS` splitting input record into fields
* use `patsplit` function to get results similar to `FPAT`
* See also [gawk manual - Split function](https://www.gnu.org/software/gawk/manual/gawk.html#index-split_0028_0029-function)
* See also [unix.stackexchange - delimit second column](https://unix.stackexchange.com/questions/372253/awk-command-to-delimit-the-second-column)

```bash
$ # 1st argument is string to be split
$ # 2nd argument is array to save results, indexed from 1
$ # 3rd argument is separator, default is FS
$ s='foo,1996-10-25,hello,good'
$ echo "$s" | awk -F, '{split($2,d,"-"); print "Month is: " d[2]}'
Month is: 10

$ # using regular expression to define separator
$ # return value is number of fields after splitting
$ s='Sample123string54with908numbers'
$ echo "$s" | awk '{n=split($0,s,/[0-9]+/); for(i=1;i<=n;i++)print s[i]}'
Sample
string
with
numbers
$ # use 4th argument if separators are needed as well
$ echo "$s" | awk '{n=split($0,s,/[0-9]+/,seps); for(i=1;i<n;i++)print seps[i]}'
123
54
908

$ # single row to multiple rows based on splitting last field
$ s='foo,baz,12:42:3'
$ echo "$s" | awk -F, '{n=split($NF,a,":"); NF--; for(i=1;i<=n;i++) print $0,a[i]}'
foo baz 12
foo baz 42
foo baz 3
```

* `substr` function allows to extract specified number of characters from given string
    * indexing starts with `1`
* See [gawk manual - substr function](https://www.gnu.org/software/gawk/manual/gawk.html#index-substr_0028_0029-function) for corner cases and details

```bash
$ # 1st argument is string to be worked on
$ # 2nd argument is starting position
$ # 3rd argument is number of characters to be extracted
$ echo 'abcdefghij' | awk '{print substr($0,1,5)}'
abcde
$ echo 'abcdefghij' | awk '{print substr($0,4,3)}'
def
$ # if 3rd argument is not given, string is extracted until end
$ echo 'abcdefghij' | awk '{print substr($0,6)}'
fghij

$ echo 'abcdefghij' | awk -v OFS=':' '{print substr($0,2,3), substr($0,6,3)}'
bcd:fgh

$ # if only few characters are needed from input line, can use empty FS
$ echo 'abcdefghij' | awk -v FS= '{print $3}'
c
$ echo 'abcdefghij' | awk -v FS= '{print $3, $5}'
c e
```

<br>

#### <a name="executing-external-commands"></a>Executing external commands

* External commands can be issued using `system` function
* Output would be as usual on `stdout` unless redirected while calling the command
* Return value of `system` depends on `exit` status of executed command, see [gawk manual - Input/Output Functions](https://www.gnu.org/software/gawk/manual/html_node/I_002fO-Functions.html) for details

```bash
$ awk 'BEGIN{system("echo Hello World")}'
Hello World

$ wc poem.txt
 4 13 65 poem.txt
$ awk 'BEGIN{system("wc poem.txt")}'
 4 13 65 poem.txt

$ awk 'BEGIN{system("seq 10 | paste -sd, > out.txt")}'
$ cat out.txt
1,2,3,4,5,6,7,8,9,10

$ ls xyz.txt
ls: cannot access 'xyz.txt': No such file or directory
$ echo $?
2
$ awk 'BEGIN{s=system("ls xyz.txt"); print "Status: " s}'
ls: cannot access 'xyz.txt': No such file or directory
Status: 2

$ cat f2
I bought two bananas and three mangoes
$ echo 'f1,f2,odd.txt' | awk -F, '{system("cat " $2)}'
I bought two bananas and three mangoes
```

<br>

#### <a name="printf-formatting"></a>printf formatting

* Similar to `printf` function in `C` and shell built-in command
* use `sprintf` function to save result in variable instead of printing
* See also [gawk manual - printf](https://www.gnu.org/software/gawk/manual/html_node/Printf.html)

```bash
$ awk '{sum += $1} END{print sum}' nums.txt
10062.9

$ # note that ORS is not appended and has to be added manually
$ awk '{sum += $1} END{printf "%.2f\n", sum}' nums.txt
10062.86

$ awk '{sum += $1} END{printf "%10.2f\n", sum}' nums.txt
  10062.86

$ awk '{sum += $1} END{printf "%010.2f\n", sum}' nums.txt
0010062.86

$ awk '{sum += $1} END{printf "%d\n", sum}' nums.txt
10062

$ awk '{sum += $1} END{printf "%+d\n", sum}' nums.txt
+10062

$ awk '{sum += $1} END{printf "%e\n", sum}' nums.txt
1.006286e+04
```

* to refer argument by positional number (starts with 1), use `<num>$`

```bash
$ # can also use: awk 'BEGIN{printf "hex=%x\noct=%o\ndec=%d\n", 15, 15, 15}'
$ awk 'BEGIN{printf "hex=%1$x\noct=%1$o\ndec=%1$d\n", 15}'
hex=f
oct=17
dec=15

$ # adding prefix to hex/oct numbers
$ awk 'BEGIN{printf "hex=%1$#x\noct=%1$#o\ndec=%1$d\n", 15}'
hex=0xf
oct=017
dec=15
```

* strings

```bash
$ # prefix remaining width with spaces
$ awk 'BEGIN{printf "%6s:%5s\n", "foo", "bar"}'
   foo:  bar

$ # suffix remaining width with spaces
$ awk 'BEGIN{printf "%-6s:%-5s\n", "foo", "bar"}'
foo   :bar  

$ # truncate
$ awk 'BEGIN{printf "%.2s\n", "foobar"}'
fo
```

* avoid using `printf` without format specifier

```bash
$ awk 'BEGIN{s="solve: 5 % x = 1"; printf s}'
awk: cmd. line:1: fatal: not enough arguments to satisfy format string
    `solve: 5 % x = 1'
               ^ ran out for this one

$ awk 'BEGIN{s="solve: 5 % x = 1"; printf "%s\n", s}'
solve: 5 % x = 1
```

<br>

#### <a name="redirecting-print-output"></a>Redirecting print output

* redirecting to file instead of stdout using `>`
* similar to behavior in shell, if file already exists it is overwritten
    * use `>>` to append to an existing file without deleting content
* however, unlike shell, subsequent redirections to same file will append to it
* See also [gawk manual - Closing Input and Output Redirections](https://www.gnu.org/software/gawk/manual/html_node/Close-Files-And-Pipes.html) if you have too many redirections

```bash
$ seq 6 | awk 'NR%2{print > "odd.txt"; next} {print > "even.txt"}'
$ cat odd.txt
1
3
5
$ cat even.txt
2
4
6

$ awk 'NR==1{col1=$1".txt"; col2=$2".txt"; next}
       {print $1 > col1; print $2 > col2}' fruits.txt
$ cat fruit.txt
apple
banana
fig
guava
$ cat qty.txt
42
31
90
6
```

* redirecting to shell command
* this is useful if you have different things to redirect to different commands, otherwise it can be done as usual in shell acting on `awk`'s output
* all redirections to same command gets combined as single input to that command

```bash
$ # same as: echo 'foo good 123' | awk '{print $2}' | wc -c
$ echo 'foo good 123' | awk '{print $2 | "wc -c"}'
5
$ # to avoid newline character being added to print
$ echo 'foo good 123' | awk -v ORS= '{print $2 | "wc -c"}'
4
$ # assuming no format specifiers in input
$ echo 'foo good 123' | awk '{printf $2 | "wc -c"}'
4

$ # same as: echo 'foo good 123' | awk '{printf $2 $3 | "wc -c"}'
$ echo 'foo good 123' | awk '{printf $2 | "wc -c"; printf $3 | "wc -c"}'
7
```

**Further Reading**

* [gawk manual - Input/Output Functions](https://www.gnu.org/software/gawk/manual/html_node/I_002fO-Functions.html)
* [gawk manual - Redirecting Output of print and printf](https://www.gnu.org/software/gawk/manual/html_node/Redirection.html)
* [gawk manual - Two-Way Communications with Another Process](https://www.gnu.org/software/gawk/manual/html_node/Two_002dway-I_002fO.html)
* [unix.stackexchange - inplace editing as well as stdout](https://unix.stackexchange.com/questions/321679/gawk-inplace-and-stdout)
* [stackoverflow - redirect blocks to separate files](https://stackoverflow.com/questions/45098279/write-blocks-in-a-text-file-to-multiple-new-files)

<br>

## <a name="gotchas-and-tips"></a>Gotchas and Tips

* using `$` for variables
* only input record `$0` and field contents `$1`, `$2` etc need `$`
* See also [unix.stackexchange - Why does awk print the whole line when I want it to print a variable?](https://unix.stackexchange.com/questions/291126/why-does-awk-print-the-whole-line-when-i-want-it-to-print-a-variable)

```bash
$ # wrong
$ awk -v word="apple" '$1==$word' fruits.txt

$ # right
$ awk -v word="apple" '$1==word' fruits.txt
apple   42
```

* dos style line endings
* See also [unix.stackexchange - filtering when last column has \r](https://unix.stackexchange.com/questions/399560/using-awk-to-select-rows-with-specific-value-in-specific-column)

```bash
$ # no issue with unix style line ending
$ printf 'foo bar\n123 789\n' | awk '{print $2, $1}'
bar foo
789 123

$ # dos style line ending causes trouble
$ printf 'foo bar\r\n123 789\r\n' | awk '{print $2, $1}'
 foo
 123

$ # easy to deal by simply setting appropriate RS
$ # note that ORS would still be newline character only
$ printf 'foo bar\r\n123 789\r\n' | awk -v RS='\r\n' '{print $2, $1}'
bar foo
789 123
```

* relying on default intial value

```bash
$ # step 1 - works for single file
$ awk '{sum += $1} END{print sum}' nums.txt
10062.9

$ # step 2 - change to work for multiple file
$ awk '{sum += $1} ENDFILE{print FILENAME, sum}' nums.txt
nums.txt 10062.9

$ # step 3 - check with multiple file input
$ # oops, default numerical value '0' for sum works only once
$ awk '{sum += $1} ENDFILE{print FILENAME, sum}' nums.txt <(seq 3)
nums.txt 10062.9
/dev/fd/63 10068.9

$ # step 4 - correctly initialize variables
$ awk 'BEGINFILE{sum=0} {sum += $1} ENDFILE{print FILENAME, sum}' nums.txt <(seq 3)
nums.txt 10062.9
/dev/fd/63 6
```

* use unary operator `+` to force numeric conversion

```bash
$ awk '{sum += $1} END{print FILENAME, sum}' nums.txt
nums.txt 10062.9

$ awk '{sum += $1} END{print FILENAME, sum}' /dev/null
/dev/null 

$ awk '{sum += $1} END{print FILENAME, +sum}' /dev/null
/dev/null 0
```

* concatenate empty string to force string comparison

```bash
$ echo '5 5.0' | awk '{print $1==$2 ? "same" : "different", "string"}'
same string

$ echo '5 5.0' | awk '{print $1""==$2 ? "same" : "different", "string"}'
different string
```

* beware of expressions going -ve for field calculations

```bash
$ cat misc.txt
foo
good bad ugly
123 xyz
a b c d

$ # trying to delete last two fields
$ awk '{NF -= 2} 1' misc.txt
awk: cmd. line:1: (FILENAME=misc.txt FNR=1) fatal: NF set to negative value
$ # dynamically change it depending on number of fields
$ awk '{NF = (NF<=2) ? 0 : NF-2} 1' misc.txt

good

a b

$ # similarly, trying to access 3rd field from end
$ awk '{print $(NF-2)}' misc.txt
awk: cmd. line:1: (FILENAME=misc.txt FNR=1) fatal: attempt to access field -1
$ awk 'NF>2{print $(NF-2)}' misc.txt
good
b
```

* If input is ASCII alone, simple trick to improve speed
* For simple non-regex based column filtering, using [cut](./miscellaneous.md#cut) command might give faster results
    * See [stackoverflow - how to split columns faster](https://stackoverflow.com/questions/46882557/how-to-split-columns-faster-in-python/46883120#46883120) for example

```bash
$ # all words containing exactly 3 lowercase a
$ time awk -F'a' 'NF==4{cnt++} END{print +cnt}' /usr/share/dict/words
1019

real    0m0.075s

$ time LC_ALL=C awk -F'a' 'NF==4{cnt++} END{print +cnt}' /usr/share/dict/words
1019

real    0m0.045s
```

<br>

## <a name="further-reading"></a>Further Reading

* Manual and related
    * `man awk` and `info awk` for quick reference from command line
    * [gawk manual](https://www.gnu.org/software/gawk/manual/gawk.html#SEC_Contents) for complete reference, extensions and more
    * [awk FAQ](http://www.faqs.org/faqs/computer-lang/awk/faq/) - from 2002, but plenty of information, especially about all the various `awk` implementations
* What's up with different `awk` versions?
    * [unix.stackexchange - brief explanation](https://unix.stackexchange.com/questions/29576/difference-between-gawk-vs-awk)
    * [Differences between gawk, nawk, mawk, and POSIX awk](https://www.reddit.com/r/awk/comments/4omosp/differences_between_gawk_nawk_mawk_and_posix_awk/)
    * [cheat sheet for awk/nawk/gawk](http://www.catonmat.net/download/awk.cheat.sheet.txt)
* Tutorials and Q&A
    * [code.snipcademy - gentle intro](https://code.snipcademy.com/tutorials/shell-scripting/awk/introduction)
    * [funtoo - using examples](https://www.funtoo.org/Awk_by_Example,_Part_1)
    * [grymoire - detailed tutorial](http://www.grymoire.com/Unix/Awk.html) - covers information about different `awk` versions as well
    * [catonmat - one liners explained](http://www.catonmat.net/series/awk-one-liners-explained)
    * [Why Learn AWK?](http://blog.jpalardy.com/posts/why-learn-awk/)
    * [awk Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/awk?sort=votes&pageSize=15)
    * [awk Q&A on unix.stackexchange](https://unix.stackexchange.com/questions/tagged/awk?sort=votes&pageSize=15)
* Alternatives
    * [GNU datamash](https://www.gnu.org/software/datamash/alternatives/)
    * [bioawk](https://github.com/lh3/bioawk)
    * [hawk](https://github.com/gelisam/hawk/blob/master/doc/README.md) - based on Haskell
    * [miller](https://github.com/johnkerl/miller) - similar to awk/sed/cut/join/sort for name-indexed data such as CSV, TSV, and tabular JSON
        * See this [ycombinator news](https://news.ycombinator.com/item?id=10066742) for other tools like this
* miscellaneous
    * [unix.stackexchange - When to use grep, sed, awk, perl, etc](https://unix.stackexchange.com/questions/303044/when-to-use-grep-less-awk-sed)
    * [awk-libs](https://github.com/e36freak/awk-libs) - lots of useful functions
    * [awkaster](https://github.com/TheMozg/awk-raycaster) - Pseudo-3D shooter written completely in awk using raycasting technique
    * [awk REPL](http://awk.js.org/) - live editor on browser
* examples for some of the stuff not covered in this tutorial
    * [unix.stackexchange - rand/srand](https://unix.stackexchange.com/questions/372816/awk-get-random-lines-of-file-satisfying-a-condition)
    * [unix.stackexchange - strftime](https://unix.stackexchange.com/questions/224969/current-date-in-awk)
    * [unix.stackexchange - ARGC and ARGV](https://unix.stackexchange.com/questions/222146/awk-does-not-end/222150#222150)
    * [stackoverflow - arbitrary precision integer extension](https://stackoverflow.com/questions/46904447/strange-output-while-comparing-engineering-numbers-in-awk)
    * [unix.stackexchange - sprintf and close](https://unix.stackexchange.com/questions/223727/splitting-file-for-every-10000-numbers-not-lines/223739#223739)
    * [unix.stackexchange - user defined functions and array passing](https://unix.stackexchange.com/questions/72469/gawk-passing-arrays-to-functions)

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
    * [Line number based filtering](#line-number-based-filtering)
* [Case Insensitive filtering](#case-insensitive-filtering)
* [Substitute functions](#substitute-functions)
* [Inplace file editing](#inplace-file-editing)
* [Using shell variables](#using-shell-variables)
* [Multiple file processing](#multiple-file-processing)
* [Control Structures](#control-structures)
    * [if-else and loops](#if-else-and-loops)
    * [next and nextfile](#next-and-nextfile)
* [Multiline processing](#multiline-processing)
* [Two file processing](#two-file-processing)
    * [Comparing whole lines](#comparing-whole-lines)
    * [Comparing specific fields](#comparing-specific-fields)
* [Dealing with duplicates](#dealing-with-duplicates)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [All unbroken blocks](#all-unbroken-blocks)
    * [Specific blocks](#specific-blocks)

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

* Assumes that you are familiar with programming concepts like variables, printing, control structures, arrays, etc
* Assumes that you are familiar with regular expressions
    * if not, check out **ERE** portion of [GNU sed regular expressions](./gnu_sed.md#regular-expressions) which is close enough to features available in `gawk`
* Refer to [Gawk: Effective AWK Programming](https://www.gnu.org/software/gawk/manual/) manual for complete reference, has information on other `awk` versions as well as POSIX capabilities

<br>

## <a name="field-processing"></a>Field processing

<br>

#### <a name="default-field-separation"></a>Default field separation

* `$0` contains the entire input record
    * default input record separator is newline character
* `$1` contains the first field text
    * default input field separator is one or more of continuous space, tab or newline characters
* `$2` contains the second field text and so on
* `$NF` points to last field
    * `NF` is built-in variable and can be used in expressions
* `$(NF-1)` points to second last field and so on

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

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* by setting `OFS` variable
* default is single space

```bash
$ # statements inside BEGIN are executed before processing any input text
$ echo 'foo:123:bar:789' | awk 'BEGIN{FS=OFS=":"} {print $1, $NF}'
foo:789
$ # can also be set using command line option -v
$ echo 'foo:123:bar:789' | awk -F: -v OFS=':' '{print $1, $NF}'
foo:789

$ # to change field separator, need to re-build contents of $0
$ # $1=$1 is an idiomatic way to do it
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{print $0}'
foo:123:bar:789
$ echo 'foo:123:bar:789' | awk -F: -v OFS='-' '{$1=$1; print $0}'
foo-123-bar-789

$ # OFS is used to separate different arguments given to print
$ echo 'foo:123:bar:789' | awk -F: -v OFS='\t' '{print $1, $3}'
foo     bar
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
* See also [gawk manual - Truth Values and Conditions](https://www.gnu.org/software/gawk/manual/html_node/Truth-Values-and-Conditions.html) and [gawk manual - Operator Precedence](https://www.gnu.org/software/gawk/manual/html_node/Precedence.html)

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
    * Of course, for medium to large programs, it is better to put the code in separate file

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

$ # print last field of all lines containing 'are'
$ awk '/are/{print $NF}' poem.txt
red,
blue,
you.
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

$ # for large input, use exit to avoid unnecessary record processing
$ seq 14323 14563435 | awk 'NR==234{print; exit}'
14556
```

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

$ # // form can also be used to specify search REGEXP
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

<br>

## <a name="using-shell-variables"></a>Using shell variables

* when `awk` code is part of shell program and shell variable needs to be passed as input to `awk` code
* for example:
    * command line argument passed to shell script, which is in turn passed on to `awk`
    * control structures in shell script calling `awk` with different search strings

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

$ # when string has to be interpreted as REGEXP, the escape sequence has to be doubled
$ echo 'foo and bar and baz land good' | awk '{$0=gensub("(.*)\\<and\\>", "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good
$ # hence passing as variable should be
$ r='(.*)\\<and\\>'
$ echo 'foo and bar and baz land good' | awk -v r="$r" '{$0=gensub(r, "\\1XYZ", 1)} 1'
foo and bar XYZ baz land good
```

<br>

## <a name="multiple-file-processing"></a>Multiple file processing

* Example to show difference between `NR` and `FNR`

```bash
$ # NR for overall record number
$ awk 'NR==1' poem.txt greeting.txt 
Roses are red,

$ # FNR for individual file's record number
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

* [gawk manual - Using ARGC and ARGV](https://www.gnu.org/software/gawk/manual/html_node/ARGC-and-ARGV.html) and [gawk manual - ARGIND](https://www.gnu.org/software/gawk/manual/html_node/Auto_002dset.html#index-ARGIND-variable)
* [stackoverflow - Finding common value across multiple files](https://stackoverflow.com/a/43473385/4082052)

<br>

## <a name="control-structures"></a>Control Structures

* Syntax is similar to `C` language and single statements inside control structures don't require to be grouped within `{}`
* See [gawk manual - Control Statements](https://www.gnu.org/software/gawk/manual/html_node/Statements.html) for details

<br>

#### <a name="if-else-and-loops"></a>if-else and loops

* We have already seen simple `if` examples in [Filtering](#filtering) section
* See also [gawk manual - Switch](https://www.gnu.org/software/gawk/manual/html_node/Switch-Statement.html)

```bash
$ # same as: sed -n '/are/ s/so/SO/p' poem.txt 
$ awk '/are/{if(sub("so", "SO")) print}' poem.txt
And SO are you.
$ # of course, can also use
$ awk '/are/ && sub("so", "SO")' poem.txt

$ # if-else example
$ awk 'NR>1{if($2>40) $0="+"$0; else $0="-"$0} 1' fruits.txt
fruit   qty
+apple   42
-banana  31
+fig     90
-guava   6
```

* conditional operator

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
```

* for loop
* similar to `C` language, `break` and `continue` statements are also available

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
$ echo 'titillate' | awk '{while( gsub(/til/, "") ) print}'
tilate
ate
```

<br>

#### <a name="next-and-nextfile"></a>next and nextfile

* `next` will skip rest of statements and start processing next line of current file being processed
    * there is a loop by default which goes over all input records, `next` is applicable for that
    * it is similar to `continue` statement within loops
* it is often used in two file processing (examples in later sections)

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

* See also [stackoverflow - Fastest way to find lines of a text file from another larger text file](https://stackoverflow.com/questions/42239179/fastest-way-to-find-lines-of-a-text-file-from-another-larger-text-file-in-bash)

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
```

* For multiple fields, separate them using `,` or form a string with some character in between

```bash
$ # can also use: awk '!seen[$2,$3]++' duplicates.txt
$ awk '!seen[$2"_"$3]++' duplicates.txt
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
$ can also use: awk '/BEGIN/{f=1; next} /END/{f=0} f' range.txt
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
$ # except 'b' block: seq 30 | awk -v b=2 '/4/{f=1; c++} f && c!=b; /6/{f=0}'
```

<br>

<br>

<br>

*More to follow*

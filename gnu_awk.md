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
$ # statements inside BEGIN are executed before processing input
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
* See also [gawk manual - Truth Values and Conditions](https://www.gnu.org/software/gawk/manual/gawk.html#Truth-Values-and-Conditions) and [gawk manual - Operator Precedence](https://www.gnu.org/software/gawk/manual/gawk.html#Precedence)

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
* Use `FNR` if there are multliple input files and you need line numbers separately for each file

```bash
$ # same as: head -n2 poem.txt | tail -n1
$ awk 'NR==2' poem.txt 
Violets are blue,

$ # print 2nd and 4th line
$ awk 'NR==2 || NR==4' poem.txt 
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
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
$ echo 'foo:123:bar:baz' | awk -F: -v OFS=: '{$1=gensub(/o/, "g", 2, $1)} 1'
fog:123:bar:baz
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

**Further Reading**

* [gawk manual - String-Manipulation Functions](https://www.gnu.org/software/gawk/manual/gawk.html#String-Functions)
* [gawk manual - escape processing](https://www.gnu.org/software/gawk/manual/gawk.html#index-sub_0028_0029-function_002c-escape-processing)



<br>

<br>

<br>

*More to follow*

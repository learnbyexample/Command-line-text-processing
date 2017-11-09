# <a name="perl-one-liners"></a>Perl one liners

**Table of Contents**

* [Executing Perl code](#executing-perl-code)
* [Simple search and replace](#simple-search-and-replace)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)
    * [Line number based filtering](#line-number-based-filtering)
* [Field processing](#field-processing)
    * [Field comparison](#field-comparison)
    * [Specifying different input field separator](#specifying-different-input-field-separator)
    * [Specifying different output field separator](#specifying-different-output-field-separator)
* [Changing record separators](#changing-record-separators)
    * [Input record separator](#input-record-separator)
    * [Output record separator](#output-record-separator)
* [Multiline processing](#multiline-processing)

<br>

```bash
$ perl -le 'print $^V'
v5.22.1

$ man perl
PERL(1)                Perl Programmers Reference Guide                PERL(1)

NAME
       perl - The Perl 5 language interpreter

SYNOPSIS
       perl [ -sTtuUWX ]      [ -hv ] [ -V[:configvar] ]
            [ -cw ] [ -d[t][:debugger] ] [ -D[number/list] ]
            [ -pna ] [ -Fpattern ] [ -l[octal] ] [ -0[octal/hexadecimal] ]
            [ -Idir ] [ -m[-]module ] [ -M[-]'module...' ] [ -f ]
            [ -C [number/list] ]      [ -S ]      [ -x[dir] ]
            [ -i[extension] ]
            [ [-e|-E] 'command' ] [ -- ] [ programfile ] [ argument ]...

       For more information on these options, you can run "perldoc perlrun".
...
```

**Prerequisites and notes**

* familiarity with programming concepts like variables, printing, control structures, arrays, etc
* Perl borrows syntax/features from **C, shell scripting, awk, sed** etc. Prior experience working with them would help a lot
* familiarity with regular expression basics
    * if not, check out **ERE** portion of [GNU sed regular expressions](./gnu_sed.md#regular-expressions)
    * examples for non-greedy, lookarounds, etc will be covered here
* this tutorial is primarily focussed on short programs that are easily usable from command line, similar to using `grep`, `sed`, `awk` etc
    * do NOT use style/syntax presented here when writing full fledged Perl programs which should use **strict, warnings** etc
* links to Perl documentation will be added as necessary
* unless otherwise specified, consider input as ASCII encoded text only

<br>

## <a name="executing-perl-code"></a>Executing Perl code

* One way is to put code in a file and use `perl` command with filename as argument
* Another is to use [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) at beginning of script, make the file executable and directly run it

```bash
$ cat code.pl
print "Hello Perl\n"
$ perl code.pl
Hello Perl

$ # similar to bash
$ cat code.sh
echo 'Hello Bash'
$ bash code.sh
Hello Bash
```

* For short programs, one can use `-e` commandline option to provide code from command line itself
* This entire chapter is about using `perl` this way from commandline

```bash
$ perl -e 'print "Hello Perl\n"'
Hello Perl

$ # similar to
$ bash -c 'echo "Hello Bash"'
Hello Bash

$ # multiple commands can be issued separated by ;
$ # -l will be covered later, here used to append newline to print
$ perl -le '$a=25; $b=12; print $a**$b'
59604644775390625
```

**Further Reading**

* `perl -h` for summary of options
* [perldoc - Command Switches](https://perldoc.perl.org/perlrun.html#Command-Switches)
* [explainshell](https://explainshell.com/explain?cmd=perl+-F+-l+-anpeE+-i+-0+-M) - to quickly get information without having to traverse through the docs
* See [Changing record separators](#changing-record-separators) section for more details on `-l` option

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

* **substitution** command syntax is very similar to `sed` for search and replace
    * syntax is `variable =~ s/REGEXP/REPLACEMENT/FLAGS` and by default acts on `$_` if variable is not specified
    * see [perldoc - SPECIAL VARIABLES](https://perldoc.perl.org/perlvar.html#SPECIAL-VARIABLES) for explanation on `$_` and other such special variables
    * more detailed examples will be covered in later sections
* Just like other text processing commands, `perl` will automatically loop over input line by line when `-n` or `-p` option is used
    * like `sed`, the `-n` option won't print the record
    * `-p` will print the record, including any changes made
    * newline character being default record separator
    * `$_` will contain the input record content, including the record separator (unlike `sed` and `awk`)
* and similar to other commands, `perl` will work with both stdin and file input

```bash
$ # change only first ',' to ' : '
$ # same as: sed 's/,/ : /'
$ seq 10 | paste -sd, | perl -pe 's/,/ : /'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : ' by using 'g' modifier
$ # same as: sed 's/,/ : /g'
$ seq 10 | paste -sd, | perl -pe 's/,/ : /g'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10

$ cat greeting.txt
Hi there
Have a nice day
$ # same as: sed 's/nice day/safe journey/' greeting.txt
$ perl -pe 's/nice day/safe journey/' greeting.txt
Hi there
Have a safe journey
```

* inplace editing
* similar to [GNU sed - using * with inplace option](./gnu_sed.md#prefix-backup-name), one can also use `*` to either prefix the backup name or place the backup files in another existing directory

```bash
$ # same as: sed -i.bkp 's/Hi/Hello/' greeting.txt
$ perl -i.bkp -pe 's/Hi/Hello/' greeting.txt
$ # original file gets preserved in 'greeting.txt.bkp'
$ cat greeting.txt
Hello there
Have a nice day

$ # use this with caution, changes made cannot be undone
$ perl -i -pe 's/nice day/safe journey/' greeting.txt
$ cat greeting.txt
Hello there
Have a safe journey
```

* Multiple input files are treated individually and changes are written back to respective files

```bash
$ cat f1
I ate 3 apples
$ cat f2
I bought two bananas and 3 mangoes

$ # -i can be used with or without backup
$ perl -i -pe 's/3/three/' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

<br>

## <a name="line-filtering"></a>Line filtering

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* syntax is `variable =~ m/REGEXP/FLAGS` to check for a match
    * `variable !~ m/REGEXP/FLAGS` for negated match
    * by default acts on `$_` if variable is not specified
* as we need to print only selective lines, use `-n` option
    * by default, contents of `$_` will be printed if no argument is passed to `print`

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # same as: grep '^[RS]' or sed -n '/^[RS]/p' or awk '/^[RS]/'
$ # /^[RS]/ is shortcut for $_ =~ m/^[RS]/
$ perl -ne 'print if /^[RS]/' poem.txt
Roses are red,
Sugar is sweet,

$ # same as: grep -i 'and' poem.txt
$ perl -ne 'print if /and/i' poem.txt
And so are you.

$ # same as: grep -v 'are' poem.txt
$ # !/are/ is shortcut for $_ !~ m/are/
$ perl -ne 'print if !/are/' poem.txt
Sugar is sweet,

$ # same as: awk '/are/ && !/so/' poem.txt
$ perl -ne 'print if /are/ && !/so/' poem.txt
Roses are red,
Violets are blue,
```

* using different delimiter
* quoting from [perldoc - Regexp Quote-Like Operators](https://perldoc.perl.org/perlop.html#Regexp-Quote-Like-Operators)

> With the m you can use any pair of non-alphanumeric, non-whitespace characters as delimiters

```bash
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log
/foo/abc/errors.log

$ perl -ne 'print if /\/foo\/a\//' paths.txt
/foo/a/report.log

$ perl -ne 'print if m#/foo/a/#' paths.txt
/foo/a/report.log

$ perl -ne 'print if !m#/foo/a/#' paths.txt
/foo/y/power.log
/foo/abc/errors.log
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* similar to `grep -F` and `awk index`
* See also
    * [perldoc - index function](https://perldoc.perl.org/functions/index.html)
    * [perldoc - Quote and Quote-like Operators](https://perldoc.perl.org/5.8.8/perlop.html#Quote-and-Quote-like-Operators)

```bash
$ # same as: grep -F 'a[5]' or awk 'index($0, "a[5]")'
$ # index returns matching position(starts at 0) and -1 if not found
$ echo 'int a[5]' | perl -ne 'print if index($_, "a[5]") != -1'
int a[5]

$ # however, string within double quotes gets interpolated, for ex
$ a='123'; echo "$a"
123
$ perl -e '$a=123; print "$a\n"'
123

$ # so, for commandline usage, better to pass string as environment variable
$ # they are accessible via the %ENV hash variable
$ perl -le 'print $ENV{PWD}'
/home/learnbyexample
$ perl -le 'print $ENV{SHELL}'
/bin/bash

$ echo 'a#$%d' | perl -ne 'print if index($_, "#$%") != -1'
$ echo 'a#$%d' | s='#$%' perl -ne 'print if index($_, $ENV{s}) != -1'
a#$%d
```

* return value is useful to match at specific position
* for ex: at start/end of line

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # start of line
$ # same as: s='a+b' awk 'index($0, ENVIRON["s"])==1' eqns.txt
$ s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
a+b,pi=3.14,5e12

$ # end of line
$ # length function returns number of characters, by default acts on $_
$ s='a+b' perl -ne '$pos = length() - length($ENV{s}) - 1;
                    print if index($_, $ENV{s}) == $pos' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

#### <a name="line-number-based-filtering"></a>Line number based filtering

* special variable `$.` contains total records read so far, similar to `NR` in `awk`
    * But no equivalent of awk's `FNR`, [see this stackoverflow Q&A for workaround](https://stackoverflow.com/questions/12384692/line-number-of-a-file-in-perl)
* See also [perldoc - eof](https://perldoc.perl.org/perlfunc.html#eof)

```bash
$ # same as: head -n2 poem.txt | tail -n1
$ # or sed -n '2p' or awk 'NR==2'
$ perl -ne 'print if $.==2' poem.txt
Violets are blue,

$ # print 2nd and 4th line
$ # same as: sed -n '2p; 4p' or awk 'NR==2 || NR==4'
$ perl -ne 'print if $.==2 || $.==4' poem.txt
Violets are blue,
And so are you.

$ # same as: tail -n1 poem.txt
$ # or sed -n '$p' or awk 'END{print}'
$ perl -ne 'print if eof' poem.txt
And so are you.
```

* for large input, use `exit` to avoid unnecessary record processing

```bash
$ seq 14323 14563435 | perl -ne 'if($.==234){print; exit}'
14556

$ # sample time comparison
$ time seq 14323 14563435 | perl -ne 'if($.==234){print; exit}' > /dev/null
real    0m0.005s
$ time seq 14323 14563435 | perl -ne 'print if $.==234' > /dev/null
real    0m2.439s

$ # mimicking head command, same as: head -n3 or sed '3q'
$ seq 14 25 | perl -pe 'exit if $.>3'
14
15
16

$ # same as: sed '3Q'
$ seq 14 25 | perl -pe 'exit if $.==3'
14
15
```

* selecting range of lines
* `..` is [perldoc - range operator](https://perldoc.perl.org/perlop.html#Range-Operators)

```bash
$ # same as: sed -n '3,5p' or awk 'NR>=3 && NR<=5'
$ # in this context, the range is compared against $.
$ seq 14 25 | perl -ne 'print if 3..5'
16
17
18

$ # selecting from particular line number to end of input
$ # same as: sed -n '10,$p' or awk 'NR>=10'
$ seq 14 25 | perl -ne 'print if $.>=10'
23
24
25
```

<br>

## <a name="field-processing"></a>Field processing

* `-a` option will auto-split each input record based on one or more continuous white-space, similar to default behavior in `awk`
* Special variable array `@F` will contain all the elements, index starting from `0`
* See also [perldoc - split function](https://perldoc.perl.org/functions/split.html)

```bash
$ cat fruits.txt
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field, index starting from 0
$ # same as: awk '{print $1}' fruits.txt 
$ perl -lane 'print $F[0]' fruits.txt
fruit
apple
banana
fig
guava

$ # print only second field
$ # same as: awk '{print $2}' fruits.txt 
$ perl -lane 'print $F[1]' fruits.txt
qty
42
31
90
6
```

* by default, leading and trailing whitespaces won't be considered when splitting the input record
    * mimicking `awk`'s default behavior

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b	c   
$ printf ' a    ate b\tc   \n' | perl -lane 'print $F[0]'
a
$ printf ' a    ate b\tc   \n' | perl -lane 'print $F[-1]'
c

$ # number of fields, $#F gives index of last element - so add 1
$ echo '1 a 7' | perl -lane 'print $#F+1'
3
$ printf ' a    ate b\tc   \n' | perl -lane 'print $#F+1'
4
```

<br>

#### <a name="field-comparison"></a>Field comparison

* for numeric context, Perl automatically tries to convert the string to number, ignoring white-space
* for string comparison, use `eq` for `==`, `ne` for `!=` and so on

```bash
$ # if first field exactly matches the string 'apple'
$ # same as: awk '$1=="apple"{print $2}' fruits.txt
$ perl -lane 'print $F[1] if $F[0] eq "apple"' fruits.txt
42

$ # print first field if second field > 35 (excluding header)
$ # same as: awk 'NR>1 && $2>35{print $1}' fruits.txt 
$ perl -lane 'print $F[0] if $F[1]>35 && $.>1' fruits.txt
apple
fig

$ # print header and lines with qty < 35
$ # same as: awk 'NR==1 || $2<35' fruits.txt
$ perl -ane 'print if $F[1]<35 || $.==1' fruits.txt
fruit   qty
banana  31
guava   6

$ # if first field does NOT contain 'a'
$ # same as: awk '$1 !~ /a/' fruits.txt 
$ perl -ane 'print if $F[0] !~ /a/' fruits.txt
fruit   qty
fig     90
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option

```bash
$ # second field where input field separator is :
$ # same as: awk -F: '{print $2}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[1]'
123

$ # last field, same as: awk -F: '{print $NF}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[-1]'
789
$ # second last field, same as: awk -F: '{print $(NF-1)}'
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[-2]'
bar

$ # second and last field
$ # other ways to print more than 1 element will be covered later
$ echo 'foo:123:bar:789' | perl -F: -lane 'print "$F[1] $F[-1]"'
123 789

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | perl -F';' -lane 'print $F[2]'
three
```

* Regular expressions based input field separator

```bash
$ # same as: awk -F'[0-9]+' '{print $2}'
$ echo 'Sample123string54with908numbers' | perl -F'\d+' -lane 'print $F[1]'
string

$ # first field will be empty as there is nothing before '{'
$ # same as: awk -F'[{}= ]+' '{print $1}'
$ # \x20 is space character, can't use literal space within [] when using -F
$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[0]'

$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[1]'
foo
$ echo '{foo}   bar=baz' | perl -F'[{}=\x20]+' -lane 'print $F[2]'
bar
```

* empty argument to `-F` will split the input record character wise

```bash
$ # same as: gawk -v FS= '{print $1}'
$ echo 'apple' | perl -F -lane 'print $F[0]'
a
$ echo 'apple' | perl -F -lane 'print $F[1]'
p
$ echo 'apple' | perl -F -lane 'print $F[-1]'
e

$ # use -C option when dealing with unicode characters
$ # S will turn on UTF-8 for stdin/stdout/stderr streams
$ printf 'hi👍 how are you?' | perl -CS -F -lane 'print $F[2]'
👍
```

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* Method 1: use `$,` to change separator between `print` arguments
    * could be remembered easily by noting that `,` is used to separate `print` arguments

```bash
$ # by default, the various arguments are concatenated
$ echo 'foo:123:bar:789' | perl -F: -lane 'print $F[1], $F[-1]'
123789

$ # change $, if different separator is needed
$ echo 'foo:123:bar:789' | perl -F: -lane '$,=" "; print $F[1], $F[-1]'
123 789
$ echo 'foo:123:bar:789' | perl -F: -lane '$,="-"; print $F[1], $F[-1]'
123-789

$ # argument can be array too
$ echo 'foo:123:bar:789' | perl -F: -lane '$,="-"; print @F[1,-1]'
123-789
$ echo 'foo:123:bar:789' | perl -F: -lane '$,=" - "; print @F'
foo - 123 - bar - 789
```

* Method 2: use `join`

```bash
$ echo 'foo:123:bar:789' | perl -F: -lane 'print join "-", $F[1], $F[-1]'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane 'print join "-", @F[1,-1]'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane 'print join " - ", @F'
foo - 123 - bar - 789
```

* Method 3: use `$"` to change separator when array is interpolated, default is space character
    * could be remembered easily by noting that interpolation happens within double quotes

```bash
$ # default is space
$ echo 'foo:123:bar:789' | perl -F: -lane 'print "@F[1,-1]"'
123 789

$ echo 'foo:123:bar:789' | perl -F: -lane '$"="-"; print "@F[1,-1]"'
123-789

$ echo 'foo:123:bar:789' | perl -F: -lane '$"=","; print "@F"'
foo,123,bar,789
```

* use `BEGIN` if same separator is to be used for all lines
    * statements inside `BEGIN` are executed before processing any input text

```bash
$ # can also use: perl -lane 'BEGIN{$"=","} print "@F"' fruits.txt
$ perl -lane 'BEGIN{$,=","} print @F' fruits.txt
fruit,qty
apple,42
banana,31
fig,90
guava,6
```

## <a name="changing-record-separators"></a>Changing record separators

* Before seeing examples for changing record separators, let's cover a detail about contents of input record and use of `-l` option
* See also [perldoc - chomp](https://perldoc.perl.org/functions/chomp.html)

```bash
$ # input record includes the record separator as well
$ # can also use: perl -pe 's/$/ 123/'
$ echo 'foo' | perl -pe 's/\n/ 123\n/'
foo 123

$ # this example shows better use case
$ # similar to paste -sd but with ability to use multi-character delimiter
$ seq 5 | perl -pe 's/\n/ : / if !eof'
1 : 2 : 3 : 4 : 5

$ # -l option will chomp off the record separator (among other things)
$ echo 'foo' | perl -l -pe 's/\n/ 123\n/'
foo

$ # -l also sets output record separator which gets added to print statements
$ # ORS gets input record separator value if no argument is passed to -l
$ # hence the newline automatically getting added for print in this example
$ perl -lane 'print $F[0] if $F[1]<35 && $.>1' fruits.txt
banana
guava
```

<br>

#### <a name="input-record-separator"></a>Input record separator

* by default, newline character is used as input record separator
* use `$/` to specify a different input record separator
    * unlike `awk`, only string can be used, no regular expressions
* for single character separator, can also use `-0` command line option which accepts octal/hexadecimal value as argument
* if `-l` option is also used
    * input record separator will be chomped from input record
    * in addition, if argument is not passed to `-l`, output record separator will get whatever is current value of input record separator
    * so, order of `-l`, `-0` and/or `$/` usage becomes important

```bash
$ s='this is a sample string'

$ # space as input record separator, printing all records
$ # same as: awk -v RS=' ' '{print NR, $0}'
$ # ORS is newline as -l is used before $/ gets changed
$ printf "$s" | perl -lne 'BEGIN{$/=" "} print "$. $_"'
1 this
2 is
3 a
4 sample
5 string

$ # print all records containing 'a'
$ # same as: awk -v RS=' ' '/a/'
$ printf "$s" | perl -l -0040 -ne 'print if /a/'
a
sample

$ # if the order is changed, ORS will be space, not newline
$ printf "$s" | perl -0040 -l -ne 'print if /a/'
a sample 
```

* `-0` option used without argument will use the ASCII NUL character as input record separator 

```bash
$ printf 'foo\0bar\0' | cat -A
foo^@bar^@$ 
$ printf 'foo\0bar\0' | perl -l -0 -ne 'print'
foo
bar

$ # could be golfed to: perl -l -0pe ''
$ # but dont use `-l0` as `0` will be treated as argument to `-l`
```

* values `-0400` to `-0777` will cause entire file to be slurped
    * idiomatically, `-0777` is used

```bash
$ # s modifier allows . to match newline as well
$ perl -0777 -pe 's/red.*are //s' poem.txt
Roses are you.

$ # replace first newline with '. '
$ perl -0777 -pe 's/\n/. /' greeting.txt
Hello there. Have a safe journey
```

* for paragraph mode (two more more consecutive newline characters), use `-00` or assign empty string to `$/`

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

* again, input record will have the separator too and using `-l` will chomp it
* however, if more than two consecutive newline characters separate the paragraphs, only two newlines will be preserved and the rest discarded
    * use `$/="\n\n"` to avoid this behavior

```bash
$ # print all paragraphs containing 'it'
$ # same as: awk -v RS= -v ORS='\n\n' '/it/' sample.txt
$ perl -00 -ne 'print if /it/' sample.txt
Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

$ # based on number of lines in each paragraph
$ perl -F'\n' -00 -ane 'print if $#F==0' sample.txt
Hello World

$ # unlike awk -F'\n' -v RS= -v ORS='\n\n' 'NF==2 && /do/' sample.txt
$ # there wont be empty line at end because input file didn't have it
$ perl -F'\n' -00 -ane 'print if $#F==1 && /do/' sample.txt
Just do-it
Believe it

Much ado about nothing
He he he
```

* Re-structuring paragraphs

```bash
$ # same as: awk 'BEGIN{FS="\n"; OFS=". "; RS=""; ORS="\n\n"} {$1=$1} 1'
$ perl -F'\n' -00 -ane 'print join ". ", @F; print "\n\n"' sample.txt
Hello World

Good day. How are you

Just do-it. Believe it

Today is sunny. Not a bit funny. No doubt you like it too

Much ado about nothing. He he he

```

* multi-character separator

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

$ # number of records, same as: awk -v RS='Error:' 'END{print NR}'
$ perl -lne 'BEGIN{$/="Error:"} print $. if eof' report.log
3
$ # print first record
$ perl -lne 'BEGIN{$/="Error:"} print if $.==1' report.log
blah blah

$ # same as: awk -F'\n' -v RS='Error:' '/surely/{print RS $0}' report.log
$ perl -F'\n' -lane 'BEGIN{$/="Error:"} print "$/$_" if /surely/' report.log
Error: something surely went wrong
some text
some more text
blah blah blah

```

* Joining lines based on specific end of line condition

```bash
$ cat msg.txt
Hello there.
It will rain to-
day. Have a safe
and pleasant jou-
rney.

$ # same as: awk -v RS='-\n' -v ORS= '1' msg.txt
$ # can also use: perl -pe 's/-\n//' msg.txt
$ perl -pe 'BEGIN{$/="-\n"} chomp' msg.txt
Hello there.
It will rain today. Have a safe
and pleasant journey.
```

<br>

#### <a name="output-record-separator"></a>Output record separator

* one way is to use `$\` to specify a different output record separator
    * by default it doesn't have a value

```bash
$ # note that despite $\ not having a value, output has newlines
$ # because the input record still has the input record separator
$ seq 3 | perl -ne 'print'
1
2
3
$ # same as: awk -v ORS='\n\n' '{print $0}'
$ seq 3 | perl -ne 'BEGIN{$\="\n"} print'
1

2

3

```

* dynamically changing output record separator

```bash
$ # same as: awk '{ORS = NR%2 ? " " : "\n"} 1'
$ # note the use of -l to chomp the input record separator
$ seq 6 | perl -lpe '$\ = $.%2 ? " " : "\n"'
1 2
3 4
5 6
$ # -l also sets the output record separator
$ # but gets overridden by $\
$ seq 6 | perl -lpe '$\ = $.%3 ? "-" : "\n"'
1-2-3
4-5-6
```

* passing argument to `-l` to set output record separator

```bash
$ seq 8 | perl -ne 'print if /[24]/'
2
4

$ # null separator, note how -l also chomps input record separator
$ seq 8 | perl -l0 -ne 'print if /[24]/' | cat -A
2^@4^@

$ # comma separator
$ seq 8 | perl -l054 -ne 'print if /[24]/'
2,4, 

$ # to add a final newline to output, use END and printf
$ seq 8 | perl -l054 -ne 'print if /[24]/; END{printf "\n"}'
2,4,
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
$ # same as: awk 'p~/are/ && /is/{print p ORS $0} {p=$0}' poem.txt
$ perl -ne 'print "$p$_" if /is/ && $p=~/are/; $p=$_' poem.txt
Violets are blue,
Sugar is sweet,
$ # if only the second line is needed, same as: awk 'p~/are/ && /is/; {p=$0}'
$ perl -ne 'print if /is/ && $p=~/are/; $p=$_' poem.txt
Sugar is sweet,

$ # print if line matches a condition as well as condition for next 2 lines
$ # same as: awk 'p2~/red/ && p1~/blue/ && /is/{print p2} {p2=p1; p1=$0}'
$ perl -ne 'print $p2 if /is/ && $p1=~/blue/ && $p2=~/red/;
            $p2=$p1; $p1=$_' poem.txt
Roses are red,
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
* how `$n && $n--` works:
    * need to note that right hand side of `&&` is processed only if left hand side is `true`
    * so for example, if initially `$n=2`, then we get
        * `2 && 2; $n=1` - evaluates to `true`
        * `1 && 1; $n=0` - evaluates to `true`
        * `0 && ` - evaluates to `false` ... no decrementing `$n` and hence will be `false` until `$n` is re-assigned non-zero value

```bash
$ # similar to: grep --no-group-separator -A1 'BEGIN' range.txt 
$ # same as: awk '/BEGIN/{n=2} n && n--' range.txt
$ perl -ne '$n=2 if /BEGIN/; print if $n && $n--' range.txt
BEGIN
1234
BEGIN
a

$ # print only line after matching line, same as: awk 'n && n--; /BEGIN/{n=1}'
$ perl -ne 'print if $n && $n--; $n=1 if /BEGIN/' range.txt
1234
a

$ # generic case: print nth line after match, awk 'n && !--n; /BEGIN/{n=3}'
$ perl -ne 'print if $n && !--$n; $n=3 if /BEGIN/' range.txt
END
c

$ # print second line prior to matched line
$ # same as: awk '/END/{print p2} {p2=p1; p1=$0}' range.txt
$ perl -ne 'print $p2 if /END/; $p2=$p1; $p1=$_' range.txt
1234
b

$ # use reversing trick for generic case of nth line before match
$ # same as: tac range.txt | awk 'n && !--n; /END/{n=3}' | tac
$ tac range.txt | perl -ne 'print if $n && !--$n; $n=3 if /END/' | tac
BEGIN
a
```




<br>

<br>

<br>

*More to follow*

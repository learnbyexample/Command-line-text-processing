# <a name="perl-one-liners"></a>Perl one liners

**Table of Contents**

* [Executing Perl code](#executing-perl-code)
* [Simple search and replace](#simple-search-and-replace)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)
    * [Line number based filtering](#line-number-based-filtering)
* [Field processing](#field-processing)
    * [Specifying different input field separator](#specifying-different-input-field-separator)

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
    * `$_` will contain the input record content, including the record separator
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

<br>

<br>

*More to follow*

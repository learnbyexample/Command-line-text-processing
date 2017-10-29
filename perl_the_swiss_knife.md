# <a name="perl-one-liners"></a>Perl one liners

**Table of Contents**

* [Executing Perl code](#executing-perl-code)
* [Simple search and replace](#simple-search-and-replace)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)

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
```

* using different delimiter
* quoting from [perldoc - Regexp Quote-Like Operators](https://perldoc.perl.org/perlop.html#Regexp-Quote-Like-Operators)

> With the m you can use any pair of non-alphanumeric, non-whitespace characters as delimiters

```bash
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log

$ perl -ne 'print if /\/foo\/a\//' paths.txt
/foo/a/report.log

$ perl -ne 'print if m#/foo/a/#' paths.txt
/foo/a/report.log
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* like `grep -F` and `awk index`
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

$ # so, for command line usage, better to pass string as environment variable
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
$ s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
a+b,pi=3.14,5e12

$ # end of line
$ # length function returns number of characters, by default acts on $_
$ s='a+b' perl -ne '$pos = length() - length($ENV{s}) - 1;
                    print if index($_, $ENV{s}) == $pos' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

<br>

<br>

*More to follow*

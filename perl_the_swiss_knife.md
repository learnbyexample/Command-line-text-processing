## <a name="perl"></a>perl

>The Perl 5 language interpreter

[Larry Wall](https://en.wikipedia.org/wiki/Larry_Wall) wrote Perl as a **general purpose scripting language**, borrowing features from **C, shell scripting, awk, sed, grep, cut, sort** etc

Reference tables given below for frequently used constructs with **perl one-liners**. Resource links given at end for further reading.

<br>

Descriptions adapted from [perldoc - command switches](http://perldoc.perl.org/perlrun.html#Command-Switches)

| Option | Description |
| ------------- | ----------- |
| -e | execute perl code |
| -n | iterate over input files in a loop, lines are NOT printed by default |
| -p | iterate over input files in a loop, lines are printed by default |
| -l | chomp input line, $\ gets value of $/ if no argument given |
| -a | autosplit input lines on space, implicitly sets -n for Perl version 5.20.0 and above |
| -F | specifies the pattern to split input lines, implicitly sets -a and -n for Perl version 5.20.0 and above |
| -i | edit files inplace, if extension provided make a backup copy |
| -0777 | slurp entire file as single string, not advisable for large input files |

<br>

Descriptions adapted from [perldoc - Special Variables](http://perldoc.perl.org/perlvar.html#SPECIAL-VARIABLES)

| Variable | Description |
| ------------- | ----------- |
| $_ | The default input and pattern-searching space |
| $. | Current line number |
| $/ | input record separator, newline by default |
| $\ | output record separator, empty string by default |
| @F | contains the fields of each line read, applicable with -a or -F option |
| %ENV | contains current environment variables |
| $ARGV | contains the name of the current file |

<br>

| Function | Description |
| ------------- | ----------- |
| length | Returns the length in characters of the value of EXPR. If EXPR is omitted, returns the length of $_ |
| eof | Returns 1 if the next read on FILEHANDLE will return end of file |

<br>

**Simple Perl program**

```bash
$ perl -e 'print "Hello!\nTesting Perl one-liner\n"'
Hello!
Testing Perl one-liner
```

<br>

**Example input file**

```bash
$ cat test.txt 
abc  : 123 : xyz
3    : 32  : foo
-2.3 : bar : bar
```

<br>

* Search and replace

```bash
$ perl -pe 's/3/%/' test.txt
abc  : 12% : xyz
%    : 32  : foo
-2.% : bar : bar

$ # use g flag to replace all occurrences, not just first match in line
$ perl -pe 's/3/%/g' test.txt
abc  : 12% : xyz
%    : %2  : foo
-2.% : bar : bar

$ # conditional replacement
$ perl -pe 's/3/@/g if /foo/' test.txt 
abc  : 123 : xyz
@    : @2  : foo
-2.3 : bar : bar

$ # using shell variables
$ r="@"
$ perl -pe "s/3/$r/" test.txt 
abc  : 12@ : xyz
@    : 32  : foo
-2.@ : bar : bar

$ # preferred approach is to use ENV hash variable
$ export s="%"
$ perl -pe 's/3/$ENV{s}/' test.txt 
abc  : 12% : xyz
%    : 32  : foo
-2.% : bar : bar
```

<br>

* Search and replace special characters

The `\Q` and `q()` constructs are helpful to nullify regex meta characters

```bash
$ # if not properly escaped or quoted, it can lead to errors
$ echo '*.^[}' | perl -pe 's/*.^[}/abc/'
Quantifier follows nothing in regex; marked by <-- HERE in m/* <-- HERE .^[}/ at -e line 1.

$ echo '*.^[}' | perl -pe 's/\*\.\^\[}/abc/'
abc

$ echo '*.^[}' | perl -pe 's/\Q*.^[}/abc/'
abc

$ echo '*.^[}' | perl -pe 's/\Q*.^[}/\$abc\$/'
$abc$

$ echo '*.^[}' | perl -pe 's/\Q*.^[}/q($abc$)/e'
$abc$
```

<br>

* Print lines based on line number or pattern

```bash
$ perl -ne 'print if /a/' test.txt 
abc  : 123 : xyz
-2.3 : bar : bar

$ perl -ne 'print if !/abc/' test.txt 
3    : 32  : foo
-2.3 : bar : bar

$ seq 123 135 | perl -ne 'print if $. == 7'
129

$ seq 1 30 | perl -ne 'print if eof'
30

$ # Use exit to save time on large input files
$ seq 14323 14563435 | perl -ne 'if($. == 234){print; exit}'
14556

$ # length() can also be used instead of length $_
$ seq 8 13 | perl -lne 'print if length $_ == 1'
8
9
```

<br>

* Print range of lines based on line number or pattern

```bash
$ seq 123 135 | perl -ne 'print if $. >= 3 && $. <= 5'
125
126
127

$ # $. is default variable compared against when using ..
$ seq 123 135 | perl -ne 'print if 3..5'
125
126
127

$ # can use many alternatives, eof looks more readable
$ seq 5 | perl -ne 'print if 3..eof'
3
4
5

$ # matching regex specified by /pattern/ is checked against $_
$ seq 5 | perl -ne 'print if 3../4/'
3
4

$ seq 1 30 | perl -ne 'print if /4/../6/'
4
5
6
14
15
16
24
25
26

$ seq 2 8 | perl -ne 'print if !(/4/../6/)'
2
3
7
8
```

<br>

* `..` vs `...`

```bash
$ echo -e '10\n11\n10' | perl -ne 'print if /10/../10/'
10
10

$ echo -e '10\n11\n10' | perl -ne 'print if /10/.../10/'
10
11
10
```

<br>

* Column manipulations

```bash
$ echo -e "1 3 4\na b c" | perl -nale 'print $F[1]'
3
b

$ echo -e "1,3,4,8\na,b,c,d" | perl -F, -lane 'print $F[$#F]'
8
d

$ perl -F: -lane 'print "$F[0] $F[2]"' test.txt 
abc    xyz
3      foo
-2.3   bar

$ perl -F: -lane '$sum+=$F[1]; END{print $sum}' test.txt 
155

$ perl -F: -lane '$F[2] =~ s/\w(?=\w)/$&,/g; print join ":", @F' test.txt 
abc  : 123 : x,y,z
3    : 32  : f,o,o
-2.3 : bar : b,a,r

$ perl -F'/:\s*[a-z]+/i' -lane 'print $F[0]' test.txt 
abc  : 123 
3    : 32  
-2.3 

$ perl -F'\s*:\s*' -lane 'print join ",", grep {/[a-z]/i} @F' test.txt 
abc,xyz
foo
bar,bar

$ perl -F: -ane 'print if (grep {/\d/} @F) < 2' test.txt 
abc  : 123 : xyz
-2.3 : bar : bar
```

<br>

* Dealing with duplicates

```bash
$ cat duplicates.txt 
abc 123 ijk
foo 567 xyz
abc 123 ijk
bar 090 pqr
tst 567 zzz

$ # whole line
$ perl -ne 'print if !$seen{$_}++' duplicates.txt 
abc 123 ijk
foo 567 xyz
bar 090 pqr
tst 567 zzz

$ # particular column
$ perl -ane 'print if !$seen{$F[1]}++' duplicates.txt 
abc 123 ijk
foo 567 xyz
bar 090 pqr
```

<br>

* Multiline processing

```bash
$ # save previous lines to make it easier for multiline matching
$ perl -ne 'print if /3/ && $p =~ /abc/; $p = $_' test.txt 
3    : 32  : foo

$ perl -ne 'print "$p$_" if /3/ && $p =~ /abc/; $p = $_' test.txt 
abc  : 123 : xyz
3    : 32  : foo

$ # with multiline matching, -0777 slurping not advisable for very large files
$ perl -0777 -ne 'print $1 if /.*abc.*\n(.*3.*\n)/' test.txt 
3    : 32  : foo
$ perl -0777 -ne 'print $1 if /(.*abc.*\n.*3.*\n)/' test.txt 
abc  : 123 : xyz
3    : 32  : foo

$ # use s flag to allow .* to match across lines
$ perl -0777 -pe 's/(.*abc.*32)/ABC/s' test.txt 
ABC  : foo
-2.3 : bar : bar

$ # use m flag if ^$ anchors are needed to match individual lines
$ perl -0777 -pe 's/(.*abc.*3)/ABC/s' test.txt 
ABC : bar : bar
$ perl -0777 -pe 's/(.*abc.*^3)/ABC/sm' test.txt 
ABC    : 32  : foo
-2.3 : bar : bar

$ # print multiple lines after matching line
$ perl -ne 'if(/abc/){ print; foreach (1..2){$n = <>; print $n} }' test.txt 
abc  : 123 : xyz
3    : 32  : foo
-2.3 : bar : bar
```

<br>

* Using modules

```bash
$ echo 'a,b,a,c,d,1,d,c,2,3,1,b' | perl -MList::MoreUtils=uniq -F, -lane 'print join ",",uniq(@F)'
a,b,c,d,1,2,3

$ base64 test.txt 
YWJjICA6IDEyMyA6IHh5egozICAgIDogMzIgIDogZm9vCi0yLjMgOiBiYXIgOiBiYXIK
$ base64 test.txt | base64 -d
abc  : 123 : xyz
3    : 32  : foo
-2.3 : bar : bar
$ base64 test.txt | perl -MMIME::Base64 -ne 'print decode_base64($_)' 
abc  : 123 : xyz
3    : 32  : foo
-2.3 : bar : bar

$ perl -MList::MoreUtils=indexes -nale '@i = indexes { /[a-z]/i } @F if $. == 1; print join ",", @F[@i]' test.txt 
abc,xyz
3,foo
-2.3,bar
```

<br>

* In place editing

```bash
$ perl -i -pe 's/\d/*/g' test.txt 
$ cat test.txt 
abc  : *** : xyz
*    : **  : foo
-*.* : bar : bar

$ perl -i.bak -pe 's/\*/^/g' test.txt 
$ cat test.txt 
abc  : ^^^ : xyz
^    : ^^  : foo
-^.^ : bar : bar
$ cat test.txt.bak 
abc  : *** : xyz
*    : **  : foo
-*.* : bar : bar
```

<br>

**Further Reading**

* [Perl Introduction](https://github.com/learnbyexample/Perl_intro) - Introductory course for Perl 5 through examples
* [Perl curated resources](https://github.com/learnbyexample/scripting_course/blob/master/Perl_curated_resources.md)
* [Handy Perl regular expressions](http://www.catonmat.net/blog/perl-one-liners-explained-part-seven/)
* [What does this regex mean?](http://stackoverflow.com/questions/22937618/reference-what-does-this-regex-mean)
* [Perl one-liners](http://www.catonmat.net/series/perl-one-liners-explained) 
* [Perl command line switches](http://perl101.org/command-line-switches.html)
* [Env](http://perldoc.perl.org/Env.html)


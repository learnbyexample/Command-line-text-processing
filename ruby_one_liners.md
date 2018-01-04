# <a name="ruby-one-liners"></a>Ruby one liners

**Table of Contents**

* [Executing Ruby code](#executing-ruby-code)
* [Simple search and replace](#simple-search-and-replace)
    * [inplace editing](#inplace-editing)
* [Line filtering](#line-filtering)
    * [Regular expressions based filtering](#regular-expressions-based-filtering)
    * [Fixed string matching](#fixed-string-matching)

<br>

```
$ ruby --version
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]

$ man ruby
RUBY(1)                Ruby Programmers Reference Guide                RUBY(1)

NAME
     ruby — Interpreted object-oriented scripting language

SYNOPSIS
     ruby [--copyright] [--version] [-SUacdlnpswvy] [-0[octal]] [-C directory]
          [-E external[:internal]] [-F[pattern]] [-I directory] [-K[c]]
          [-T[level]] [-W[level]] [-e command] [-i[extension]] [-r library]
          [-x[directory]] [--{enable|disable}-FEATURE] [--dump=target]
          [--verbose] [--] [program_file] [argument ...]

DESCRIPTION
     Ruby is an interpreted scripting language for quick and easy object-ori‐
     ented programming.  It has many features to process text files and to do
     system management tasks (like in Perl).  It is simple, straight-forward,
     and extensible.

     If you want a language for easy object-oriented programming, or you don't
     like the Perl ugliness, or you do like the concept of LISP, but don't
     like too many parentheses, Ruby might be your language of choice.
...
```

**Prerequisites and notes**

* familiarity with programming concepts like variables, printing, control structures, arrays, etc
* familiarity with regular expression
* this tutorial is primarily focussed on short programs that are easily usable from command line, similar to using `grep`, `sed`, `awk`, `perl` etc
* unless otherwise specified, consider input as ASCII encoded text only
* this is an attempt to translate [Perl chapter](./perl_the_swiss_knife.md) to `ruby`, I don't have prior experience of using `ruby`

<br>

## <a name="executing-ruby-code"></a>Executing Ruby code

* One way is to put code in a file and use `ruby` command with filename as argument
    * another is to use [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) at beginning of script, make the file executable and directly run it
* For short programs, one can use `-e` commandline option to provide code from command line itself
    * this entire chapter is about using `ruby` this way from commandline

```bash
$ cat code.rb
print "Hello Ruby\n"
$ ruby code.rb
Hello Ruby

$ # same as: perl -e 'print "Hello Perl\n"'
$ ruby -e 'print "Hello Ruby\n"'
Hello Ruby

$ # multiple commands can be issued separated by ;
$ # use puts instead of print to automatically add newline character
$ # same as: perl -E '$x=25; $y=12; say $x**$y'
$ ruby -e 'x=25; y=12; puts x**y'
59604644775390625
```

**Further Reading**

* `ruby -h` for summary of options
    * [explainshell](https://explainshell.com/explain?cmd=ruby+-F+-l+-anpe+-i+-0) - to quickly get information without having to traverse through the docs
* [ruby-lang documentation](https://www.ruby-lang.org/en/documentation/) - manuals, tutorials and references

<br>

## <a name="simple-search-and-replace"></a>Simple search and replace

* More detailed examples with regular expressions will be covered in later sections
* Just like other text processing commands, `ruby` will automatically loop over input line by line when `-n` or `-p` option is used
    * like `sed`, the `-n` option won't print the record
    * `-p` will print the record, including any changes made
    * default record separator is newline character
    * `$_` will contain the input record content, including the record separator (like `perl` and unlike `sed/awk`)
* and similar to other commands, `ruby` will work with both stdin and file input
    * See other chapters for examples of [seq](./miscellaneous.md#seq), [paste](./restructure_text.md#paste), etc

```bash
$ # change only first ',' to ' : '
$ # same as: perl -pe 's/,/ : /'
$ seq 10 | paste -sd, | ruby -pe 'sub(/,/, " : ")'
1 : 2,3,4,5,6,7,8,9,10

$ # change all ',' to ' : '
$ # same as: perl -pe 's/,/ : /g'
$ seq 10 | paste -sd, | ruby -pe 'gsub(/,/, " : ")'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10

$ # sub(/,/, " : ") is shortcut for $_.sub!(/,/, " : ")
$ # gsub(/,/, " : ") is shortcut for $_.gsub!(/,/, " : ")
$ # sub! and gsub! do inplace changing
$ # sub and gsub returns the result, similar to perl's s///r modifier
$ # () is optional, sub /,/, " : " can be used instead of sub(/,/, " : ")
```

<br>

#### <a name="inplace-editing"></a>inplace editing

```bash
$ cat greeting.txt
Hi there
Have a nice day

$ # original file gets preserved in 'greeting.txt.bkp'
$ # same as: perl -i.bkp -pe 's/Hi/Hello/' greeting.txt
$ ruby -i.bkp -pe 'sub(/Hi/, "Hello")' greeting.txt
$ cat greeting.txt
Hello there
Have a nice day

$ # use empty argument to -i with caution, changes made cannot be undone
$ ruby -i -pe 'sub(/nice day/, "safe journey")' greeting.txt
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

$ # same as: perl -i.bkp -pe 's/3/three/' f1 f2
$ ruby -i.bkp -pe 'sub(/3/, "three")' f1 f2
$ cat f1
I ate three apples
$ cat f2
I bought two bananas and three mangoes
```

**Further Reading**

* [ruby-doc Pre-defined variables](https://ruby-doc.org/core-2.5.0/doc/globals_rdoc.html#label-Pre-defined+variables) for explanation on `$_` and other such special variables
* [ruby-doc gsub](https://ruby-doc.org/core-2.5.0/String.html#method-i-gsub) for `gsub` syntax details

<br>

## <a name="line-filtering"></a>Line filtering

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* one way is to use `variable =~ /REGEXP/FLAGS` to check for a match
    * `variable !~ /REGEXP/FLAGS` for negated match
    * by default acts on `$_` if variable is not specified
    * see [ruby-doc Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) for regular expression details
* as we need to print only selective lines, use `-n` option
    * by default, contents of `$_` will be printed if no argument is passed to `print`

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # same as: perl -ne 'print if /^[RS]/' poem.txt
$ # /^[RS]/ is shortcut for $_ =~ /^[RS]/
$ ruby -ne 'print if /^[RS]/' poem.txt
Roses are red,
Sugar is sweet,

$ # same as: perl -ne 'print if /and/i' poem.txt
$ ruby -ne 'print if /and/i' poem.txt
And so are you.

$ # same as: perl -ne 'print if !/are/' poem.txt
$ # !/are/ is shortcut for $_ !~ /are/
$ ruby -ne 'print if !/are/' poem.txt
Sugar is sweet,

$ # same as: perl -ne 'print if /are/ && !/so/' poem.txt
$ ruby -ne 'print if /are/ && !/so/' poem.txt
Roses are red,
Violets are blue,
```

* using different delimiter
* quoting from [ruby-doc Percent Strings](https://ruby-doc.org/core-2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings)

> If you are using “(”, “[”, “{”, “<” you must close it with “)”, “]”, “}”, “>” respectively. You may use most other non-alphanumeric characters for percent string delimiters such as “%”, “|”, “^”, etc.

```bash
$ cat paths.txt
/foo/a/report.log
/foo/y/power.log
/foo/abc/errors.log

$ # same as: perl -ne 'print if /\/foo\/a\//' paths.txt
$ ruby -ne 'print if /\/foo\/a\//' paths.txt
/foo/a/report.log

$ # same as: perl -ne 'print if m#/foo/a/#' paths.txt
$ ruby -ne 'print if %r#/foo/a/#' paths.txt
/foo/a/report.log

$ # same as: perl -ne 'print if !m#/foo/a/#' paths.txt
$ ruby -ne 'print if !%r#/foo/a/#' paths.txt
/foo/y/power.log
/foo/abc/errors.log
```

<br>

#### <a name="fixed-string-matching"></a>Fixed string matching

* To match strings literally, use `index`
* See [ruby-doc index](https://ruby-doc.org/core-2.5.0/String.html#method-i-index) for details

```bash
$ # index returns matching position(starts at 0) and nil if not found
$ # same as: perl -ne 'print if index($_, "a[5]") != -1'
$ echo 'int a[5]' | ruby -ne 'print if /a[5]/'
$ echo 'int a[5]' | ruby -ne 'print if $_.index("a[5]")'
int a[5]

$ # however, string within double quotes gets interpolated, for ex
$ ruby -e 'a=5; puts "value of a: #{a}"'
value of a: 5

$ # so, for commandline usage, better to pass string as environment variable
$ # they are accessible via the ENV hash variable
$ # same as: perl -le 'print $ENV{SHELL}'
$ ruby -e 'puts ENV["SHELL"]'
/bin/bash

$ echo 'int #{a}' | ruby -ne 'print if $_.index("#{a}")'
-e:1:in `<main>': undefined local variable or method `a' for main:Object (NameError)
$ echo 'int #{a}' | s='#{a}' ruby -ne 'print if $_.index(ENV["s"])'
int #{a}
```

* `index` allows to use regex as well

```bash
$ # passing string
$ ruby -ne 'print if $_.index("a+b")' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # passing regex
$ ruby -ne 'print if $_.index(/a+b/)' eqns.txt
$ ruby -ne 'print if $_.index(/a\+b/)' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b
```

* return value is useful to match at specific position
* for ex: at start/end of line

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # start of line
$ # same as: s='a+b' perl -ne 'print if index($_, $ENV{s})==0' eqns.txt
$ s='a+b' ruby -ne 'print if $_.index(ENV["s"])==0' eqns.txt
a+b,pi=3.14,5e12

$ # optional 2nd argument allows to specify offset to start searching
$ # similar to: s='a+b' perl -ne 'print if index($_, $ENV{s})>0' eqns.txt
$ s='a+b' ruby -ne 'print if $_.index(ENV["s"], 1)' eqns.txt
i*(t+9-g)/8,4-a+b

$ # end of line
$ # same as: s='a+b' perl -ne '$pos = length() - length($ENV{s}) - 1;
$ #                  print if index($_, $ENV{s}) == $pos' eqns.txt
$ s='a+b' ruby -ne 'pos = $_.length() - ENV["s"].length() - 1;
                    print if $_.index(ENV["s"]) == pos' eqns.txt
i*(t+9-g)/8,4-a+b
```












<br>

<br>

<br>

*More to come*

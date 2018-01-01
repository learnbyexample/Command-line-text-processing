# <a name="ruby-one-liners"></a>Ruby one liners

**Table of Contents**

* [Executing Ruby code](#executing-ruby-code)
* [Simple search and replace](#simple-search-and-replace)
    * [inplace editing](#inplace-editing)

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
* [ruby-doc Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) for regular expression details

<br>

<br>

<br>

*More to come*

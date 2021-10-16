<br> <br> <br>

---

:information_source: :information_source: This chapter has been converted into a better formatted ebook - https://learnbyexample.github.io/learn_ruby_oneliners/. The ebook also has content updated for newer version of `ruby`, extra chapter for parsing json/csv/xml, includes exercises, solutions, etc.

For markdown source and links to buy pdf/epub versions, see: https://github.com/learnbyexample/learn_ruby_oneliners

---

<br> <br> <br>

# <a name="ruby-one-liners"></a>Ruby one liners

**Table of Contents**

* [Executing Ruby code](#executing-ruby-code)
* [Simple search and replace](#simple-search-and-replace)
    * [inplace editing](#inplace-editing)
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
* [Ruby regular expressions](#ruby-regular-expressions)
    * [gotchas and tricks](#gotchas-and-tricks)
    * [Backslash sequences](#backslash-sequences)
    * [Non-greedy quantifier](#non-greedy-quantifier)
    * [Lookarounds](#lookarounds)
    * [Special capture groups](#special-capture-groups)
    * [Modifiers](#modifiers)
    * [Code in replacement section](#code-in-replacement-section)
    * [Quoting metacharacters](#quoting-metacharacters)
* [Two file processing](#two-file-processing)
    * [Comparing whole lines](#comparing-whole-lines)
    * [Comparing specific fields](#comparing-specific-fields)
    * [Line number matching](#line-number-matching)
* [Creating new fields](#creating-new-fields)
* [Multiple file input](#multiple-file-input)
* [Dealing with duplicates](#dealing-with-duplicates)
    * [using uniq method](#using-uniq-method)
* [Lines between two REGEXPs](#lines-between-two-regexps)
    * [All unbroken blocks](#all-unbroken-blocks)
    * [Specific blocks](#specific-blocks)
    * [Broken blocks](#broken-blocks)
* [Array operations](#array-operations)
    * [Filtering](#filtering)
    * [Sorting](#sorting)
    * [Transforming](#transforming)
* [Miscellaneous](#miscellaneous)
    * [split](#split)
    * [Fixed width processing](#fixed-width-processing)
    * [String and file replication](#string-and-file-replication)
    * [transliteration](#transliteration)
    * [Executing external commands](#executing-external-commands)
* [Further Reading](#further-reading)

<br>

```
$ ruby --version
ruby 2.5.0p0 (2017-12-25 revision 61468) [x86_64-linux]

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
* familiarity with regular expressions
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

$ # multiple statements can be issued separated by ;
$ # puts adds newline character if input doesn't end with a newline
$ # similar to: perl -E '$x=25; $y=12; say $x**$y'
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
$ # sample stdin data
$ seq 10 | paste -sd,
1,2,3,4,5,6,7,8,9,10

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

* [ruby-doc: Pre-defined variables](https://ruby-doc.org/core-2.5.0/doc/globals_rdoc.html#label-Pre-defined+variables) for explanation on `$_` and other such special variables
* [ruby-doc: gsub](https://ruby-doc.org/core-2.5.0/String.html#method-i-gsub) for `gsub` syntax details

<br>

## <a name="line-filtering"></a>Line filtering

<br>

#### <a name="regular-expressions-based-filtering"></a>Regular expressions based filtering

* one way is to use `variable =~ /REGEXP/FLAGS` to check for a match
    * use `variable !~ /REGEXP/FLAGS` for negated match
    * by default acts on `$_` if variable is not specified
    * see [ruby-doc: Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) for documentation
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
* quoting from [ruby-doc: Percent Strings](https://ruby-doc.org/core-2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings)

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

* To match strings literally, use `include?` method

```bash
$ echo 'int a[5]' | ruby -ne 'print if /a[5]/'
$ echo 'int a[5]' | ruby -ne 'print if $_.include?("a[5]")'
int a[5]

$ # however, string within double quotes gets interpolated
$ ruby -e 'a=5; puts "value of a:\t#{a}"'
value of a:     5
$ # use %q (covered later) to specify single quoted string
$ echo 'int #{a}' | ruby -ne 'print if $_.include?(%q/#{a}/)'
int #{a}
$ # or pass the string as environment variable
$ echo 'int #{a}' | s='#{a}' ruby -ne 'print if $_.include?(ENV["s"])'
int #{a}
```

* restricting match to start/end of line

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # start of line
$ s='a+b' ruby -ne 'print if $_.start_with?(ENV["s"])' eqns.txt
a+b,pi=3.14,5e12

$ # end of line
$ # -l option is needed to remove record separator (covered later)
$ s='a+b' ruby -lne 'print if $_.end_with?(ENV["s"])' eqns.txt
i*(t+9-g)/8,4-a+b
```

* `index` method returns matching position (starts at 0) and nil if not found
    * supports both string and regexp
    * optional 2nd argument allows to specify offset to start searching
* See [ruby-doc: index](https://ruby-doc.org/core-2.5.0/String.html#method-i-index) for details

```bash
$ # passing string
$ ruby -ne 'print if $_.index("a+b")' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b
$ ruby -ne 'print if $_.index("a+b")==0' eqns.txt
a+b,pi=3.14,5e12

$ # passing regexp
$ ruby -ne 'print if $_.index(/[+*]/)<5' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ s='a+b' ruby -ne 'print if $_.index(ENV["s"], 1)' eqns.txt
i*(t+9-g)/8,4-a+b
```

<br>

#### <a name="line-number-based-filtering"></a>Line number based filtering

* special variable `$.` contains total records read so far, similar to `NR` in `awk`
    * as far as I've checked the docs, there's no equivalent of awk's `FNR`
* See also [ruby-doc: eof](https://ruby-doc.org/core-2.5.0/IO.html#method-i-eof)

```bash
$ # print 2nd line
$ # same as: perl -ne 'print if $.==2' poem.txt
$ ruby -ne 'print if $.==2' poem.txt
Violets are blue,

$ # print 2nd and 4th line
$ # same as: perl -ne 'print if $.==2 || $.==4' poem.txt
$ # can also use: ruby -ne 'print if [2, 4].include?($.)' poem.txt
$ ruby -ne 'print if $.==2 || $.==4' poem.txt
Violets are blue,
And so are you.

$ # print last line
$ # same as: perl -ne 'print if eof' poem.txt
$ # $< is like filehandle for input files/stdin given from commandline
$ ruby -ne 'print if $<.eof' poem.txt
And so are you.
```

* for large input, use `exit` to avoid unnecessary record processing
* See [ruby-doc: Control Expressions](https://ruby-doc.org/core-2.5.0/doc/syntax/control_expressions_rdoc.html) for syntax details

```bash
$ # same as: perl -ne 'if($.==234){print; exit}'
$ seq 14323 14563435 | ruby -ne 'if $.==234 then print; exit end'
14556
$ # can also group the statements in ()
$ seq 14323 14563435 | ruby -ne '(print; exit) if $.==234'
14556

$ # mimicking head command
$ # same as: head -n3 and sed '3q' or perl -pe 'exit if $.>3'
$ seq 14 25 | ruby -pe 'exit if $.>3'
14
15
16

$ # same as: sed '3Q' and perl -pe 'exit if $.==3'
$ seq 14 25 | ruby -pe 'exit if $.==3'
14
15
```

* selecting range of lines
* See [ruby-doc: Range](https://ruby-doc.org/core-2.5.0/Range.html) for syntax details

```bash
$ # in this context, the range is compared against $.
$ # same as: perl -ne 'print if 3..5'
$ seq 14 25 | ruby -ne 'print if 3..5'
16
17
18

$ # selecting from particular line number to end of input
$ # same as: perl -ne 'print if $.>=10'
$ seq 14 25 | ruby -ne 'print if $.>=10'
23
24
25
```

<br>

## <a name="field-processing"></a>Field processing

* `-a` option will auto-split each input record based on one or more continuous white-space
    * similar to default behavior in `awk` and same as `perl -a`
    * See also [split](#split) section
* Special variable array `$F` will contain all the elements, indexing starts from 0
    * negative indexing is also supported, `-1` gives last element, `-2` gives last-but-one and so on
    * see [Array operations](#array-operations) section for examples on array usage

```bash
$ cat fruits.txt
fruit   qty
apple   42
banana  31
fig     90
guava   6

$ # print only first field, indexing starts from 0
$ # same as: perl -lane 'print $F[0]' fruits.txt
$ ruby -ane 'puts $F[0]' fruits.txt
fruit
apple
banana
fig
guava

$ # print only second field
$ # same as: perl -lane 'print $F[1]' fruits.txt
$ ruby -ane 'puts $F[1]' fruits.txt
qty
42
31
90
6
```

* by default, leading and trailing whitespaces won't be considered when splitting the input record
    * same as `awk`'s default behavior and `perl -a`

```bash
$ printf ' a    ate b\tc   \n'
 a    ate b     c
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F[0]'
a
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F[-1]'
c

$ # number of elements
$ printf ' a    ate b\tc   \n' | ruby -ane 'puts $F.length'
4
```

<br>

#### <a name="field-comparison"></a>Field comparison

* operators `=`, `!=`, `<`, etc will work for both string/numeric comparison
* unlike `perl`, numeric comparison for text requires converting to appropriate numeric format
    * See [ruby-doc: string methods](https://ruby-doc.org/core-2.5.0/String.html#method-i-to_c) for details

```bash
$ # if first field exactly matches the string 'apple'
$ # same as: perl -lane 'print $F[1] if $F[0] eq "apple"' fruits.txt
$ ruby -ane 'puts $F[1] if $F[0] == "apple"' fruits.txt
42

$ # print first field if second field > 35 (excluding header)
$ # same as: perl -lane 'print $F[0] if $F[1]>35 && $.>1' fruits.txt
$ ruby -ane 'puts $F[0] if $F[1].to_i > 35 && $.>1' fruits.txt
apple
fig

$ # print header and lines with qty < 35
$ # same as: perl -ane 'print if $F[1]<35 || $.==1' fruits.txt
$ ruby -ane 'print if $F[1].to_i < 35 || $.==1' fruits.txt
fruit   qty
banana  31
guava   6

$ # if first field does NOT contain 'a'
$ # same as: perl -ane 'print if $F[0] !~ /a/' fruits.txt
$ ruby -ane 'print if $F[0] !~ /a/' fruits.txt
fruit   qty
fig     90
```

<br>

#### <a name="specifying-different-input-field-separator"></a>Specifying different input field separator

* by using `-F` command line option

```bash
$ # second field where input field separator is :
$ # same as: perl -F: -lane 'print $F[1]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[1]'
123

$ # last field, same as: perl -F: -lane 'print $F[-1]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[-1]'
789
$ # second last field, perl -F: -lane 'print $F[-2]'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts $F[-2]'
bar

$ # second and last field, same as: perl -F: -lane 'print "$F[1] $F[-1]"'
$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts "#{$F[1]} #{$F[-1]}"'
123 789

$ # use quotes to avoid clashes with shell special characters
$ echo 'one;two;three;four' | ruby -F';' -ane 'puts $F[2]'
three
```

* last element of `$F` array will contain the record separator as well
    * note that default `-a` option without `-F` won't have this issue as whitespaces at start/end are stripped
* it doesn't make visual difference when `puts` is used as it adds newline only if not already present
* if the record separator is not desired, use `-l` option to remove the record separator from input

```bash
$ echo 'foo 123' | ruby -ane 'puts "#{$F[-1]}xyz"'
123xyz

$ echo 'foo:123:bar:789' | ruby -F: -ane 'puts "#{$F[-1]}a"'
789
a
$ echo 'foo:123:bar:789' | ruby -F: -lane 'puts "#{$F[-1]}a"'
789a
```

* Regular expressions based input field separator

```bash
$ # same as: perl -F'\d+' -lane 'print $F[1]'
$ echo 'Sample123string54with908numbers' | ruby -F'\d+' -ane 'puts $F[1]'
string

$ # first field will be empty as there is nothing before '{'
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[0]'

$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[1]'
foo
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[2]'
bar
$ echo '{foo}   bar=baz' | ruby -F'[{}= ]+' -ane 'puts $F[-1]'
baz
```

* to process individual characters, simply use indexing on input string
* See [ruby-doc: Encoding](https://ruby-doc.org/core-2.5.0/Encoding.html) for details on handling different string encodings

```bash
$ # same as: perl -F -lane 'print $F[0]'
$ echo 'apple' | ruby -ne 'puts $_[0]'
a

$ # if needed, chomp the record separator using -l
$ # same as: perl -F -lane 'print $F[-1]'
$ echo 'apple' | ruby -lne 'puts $_[-1]'
e

$ ruby -e 'puts Encoding.default_external'
UTF-8
$ printf 'hi👍 how are you?' | ruby -ne 'puts $_[2]'
👍
$ # use -E option to explicitly specify external/internal encodings
$ printf 'hi👍 how are you?' | ruby -E UTF-8:UTF-8 -ne 'puts $_[2]'
👍
```

<br>

#### <a name="specifying-different-output-field-separator"></a>Specifying different output field separator

* use `$,` to change separator between `print` arguments
    * could be remembered easily by noting that `,` is used to separate `print` arguments
    * note that `$,` doesn't affect `puts` which always uses newline as separator
* the `-l` option is useful here in more than one way
    * it removes input record separator
    * and appends the record separator to `print` output

```bash
$ # by default, the various arguments are concatenated
$ echo 'foo:123:bar:789' | ruby -F: -lane 'print $F[1], $F[-1]'
123789

$ # change $, if different separator is needed
$ # same as: perl -F: -lane '$,=" "; print $F[1], $F[-1]'
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,=" "; print $F[1], $F[-1]'
123 789
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,="-"; print $F[1], $F[-1]'
123-789

$ # array's join method also uses $,
$ # same as: perl -F: -lane '$,=" - "; print @F'
$ echo 'foo:123:bar:789' | ruby -F: -lane '$,=" - "; print $F.join'
foo - 123 - bar - 789
$ # or pass the separator as argument to join method
$ echo 'foo:123:bar:789' | ruby -F: -lane 'print $F.join(" - ")'
foo - 123 - bar - 789
$ # or the equivalent
$ echo 'foo:123:bar:789' | ruby -F: -lane 'print $F * " - "'
foo - 123 - bar - 789
```

* use `BEGIN` if same separator is to be used for all lines
    * statements inside `BEGIN` are executed before processing any input text

```bash
$ # same as: perl -lane 'BEGIN{$,=","} print @F' fruits.txt
$ ruby -lane 'BEGIN{$,=","}; print $F.join' fruits.txt
fruit,qty
apple,42
banana,31
fig,90
guava,6
```

<br>

## <a name="changing-record-separators"></a>Changing record separators

<br>

#### <a name="input-record-separator"></a>Input record separator

* by default, newline character is used as input record separator
* use `$/` to specify a different input record separator
    * unlike `gawk`, only string can be used, no regular expressions
* for single character separator, can also use `-0` command line option which accepts octal value as argument
* if `-l` option is also used
    * input record separator will be chomped from input record
        * earlier versions used `chop` instead of `chomp`. See [bugs.ruby-lang.org 12926](https://bugs.ruby-lang.org/issues/12926)
    * in addition, output record separator(ORS) will get whatever is current value of input record separator
    * so, order of `-l`, `-0` and/or `$/` usage becomes important

```bash
$ s='this is a sample string'

$ # space as input record separator, printing all records
$ # ORS is newline as -l is used before $/ gets changed
$ # same as: perl -lne 'BEGIN{$/=" "} print "$. $_"'
$ printf "$s" | ruby -lne 'BEGIN{$/=" "}; print "#{$.} #{$_}"'
1 this
2 is
3 a
4 sample
5 string

$ # print all records containing 'a'
$ # same as: perl -l -0040 -ne 'print if /a/'
$ printf "$s" | ruby -l -0040 -ne 'print if /a/'
a
sample

$ # if the order is changed, ORS will be space, not newline
$ printf "$s" | ruby -0040 -l -ne 'print if /a/'
a sample 
```

* `-0` option used without argument will use the ASCII NUL character as input record separator
* `-0777` will cause entire file to be slurped

```bash
$ printf 'foo\0bar\0' | cat -A
foo^@bar^@$
$ # same as: perl -l -0 -ne 'print'
$ # could be golfed to: ruby -l0pe ''
$ printf 'foo\0bar\0' | ruby -l -0 -ne 'print'
foo
bar

$ # replace first newline with '. '
$ # same as: perl -0777 -pe 's/\n/. /' greeting.txt
$ ruby -0777 -pe 'sub(/\n/, ". ")' greeting.txt
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
$ # same as: perl -00 -ne 'print if /it/' sample.txt
$ ruby -00 -ne 'print if /it/' sample.txt
Just do-it
Believe it

Today is sunny
Not a bit funny
No doubt you like it too

$ # based on number of lines in each paragraph
$ # same as: perl -F'\n' -00 -ane 'print if $#F==0' sample.txt
$ ruby -F'\n' -00 -ane 'print if $F.length==1' sample.txt
Hello World

```

* Re-structuring paragraphs

```bash
$ # same as: perl -F'\n' -l -00 -ane 'print join ". ", @F' sample.txt
$ ruby -F'\n' -l -00 -ane 'print $F.join(". ")' sample.txt
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

$ # number of records, same as: perl -lne 'BEGIN{$/="Error:"} print $. if eof'
$ ruby -ne 'BEGIN{$/="Error:"}; puts $. if $<.eof' report.log
3
$ # print first record, same as: perl -lne 'BEGIN{$/="Error:"} print if $.==1'
$ ruby -lne 'BEGIN{$/="Error:"}; print if $.==1' report.log
blah blah

$ # print a record if it contains given string
$ # same as: perl -lne 'BEGIN{$/="Error:"} print "$/$_" if /surely/'
$ ruby -lne 'BEGIN{$/="Error:"}; print $/,$_ if /surely/' report.log
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

$ # same as: perl -pe 'BEGIN{$/="-\n"} chomp' msg.txt
$ ruby -pe 'BEGIN{$/="-\n"}; chomp' msg.txt
Hello there.
It will rain today. Have a safe
and pleasant journey.
```

<br>

#### <a name="output-record-separator"></a>Output record separator

* use `$\` to specify a different output record separator
    * applies to `print` but not `puts`

```bash
$ # note that despite not setting $\, output has newlines
$ # because the input record still has the input record separator
$ seq 3 | ruby -ne 'print'
1
2
3
$ # same as: perl -ne 'BEGIN{$\="\n"} print'
$ seq 3 | ruby -ne 'BEGIN{$\="\n"}; print'
1

2

3

$ seq 2 | ruby -ne 'BEGIN{$\="---\n"}; print'
1
---
2
---
```

* dynamically changing output record separator
* **Note:** except `nil` and `false`, all other values evaluate to `true`
    * `0`, empty string/array/etc evaluate to `true`

```bash
$ # note the use of -l to chomp the input record separator
$ # same as: perl -lpe '$\ = $.%2 ? " " : "\n"'
$ seq 6 | ruby -lpe '$\ = $.%2!=0 ? " " : "\n"'
1 2
3 4
5 6

$ # -l also sets the output record separator
$ # but gets overridden by $\
$ # same as: perl -lpe '$\ = $.%3 ? "-" : "\n"'
$ seq 6 | ruby -lpe '$\ = $.%3!=0 ? "-" : "\n"'
1-2-3
4-5-6
```

<br>

## <a name="multiline-processing"></a>Multiline processing

* Processing consecutive lines
* to keep the one-liner short, global variables(`$` prefix) are used here
    * See [ruby-doc: Global variables](https://ruby-doc.org/core-2.5.0/doc/syntax/assignment_rdoc.html#label-Global+Variables) for syntax details

```bash
$ cat poem.txt
Roses are red,
Violets are blue,
Sugar is sweet,
And so are you.

$ # match two consecutive lines
$ # same as: perl -ne 'print $p,$_ if /is/ && $p=~/are/; $p=$_' poem.txt
$ ruby -ne 'print $p,$_ if /is/ && $p=~/are/; $p=$_' poem.txt
Violets are blue,
Sugar is sweet,
$ # if only the second line is needed
$ ruby -ne 'print if /is/ && $p=~/are/; $p=$_' poem.txt
Sugar is sweet,

$ # print if line matches a condition as well as condition for next 2 lines
$ ruby -ne 'print $p2 if /is/ && $p1=~/blue/ && $p2=~/red/;
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
* **Note**
    * default uninitialized value is `nil`, has to be explicitly converted for comparison
    * no auto increment/decrement operators, can use `+=1` and `-=1`


```bash
$ ruby -le 'print $a'

$ ruby -le 'print $a.to_i'
0

$ # print matching line and n-1 lines following the matched line
$ # same as: perl -ne '$n=2 if /BEGIN/; print if $n && $n--' range.txt
$ # can also use: ruby -ne 'BEGIN{n=0}; n=2 if /BEGIN/; print if n>0 && n-=1'
$ ruby -ne '$n=2 if /BEGIN/; print if $n.to_i>0 && $n-=1' range.txt
BEGIN
1234
BEGIN
a

$ # print nth line after match
$ # same as: perl -ne 'print if $n && !--$n; $n=3 if /BEGIN/' range.txt
$ ruby -ne '$n.to_i>0 && (print if $n==1; $n-=1); $n=3 if /BEGIN/' range.txt
END
c

$ # use reversing trick for nth line before match
$ tac range.txt | ruby -ne '$n.to_i>0 && (print if $n==1; $n-=1); $n=3 if /END/' | tac
BEGIN
a
```

**Further Reading**

* [softwareengineering - FSM examples](https://softwareengineering.stackexchange.com/questions/47806/examples-of-finite-state-machines)
* [wikipedia - FSM](https://en.wikipedia.org/wiki/Finite-state_machine)

<br>

## <a name="ruby-regular-expressions"></a>Ruby regular expressions

* assuming that you are already familiar with basics of regular expressions
    * if not, check out [Ruby Regexp](https://leanpub.com/rubyregexp) ebook - step by step guide from beginner to advanced levels
* examples/descriptions are for string containing ASCII characters only
* See [ruby-doc: Regexp](https://ruby-doc.org/core-2.5.0/Regexp.html) for documentation
* See [rexegg ruby](https://www.rexegg.com/regex-ruby.html) for a bit of ruby regexp history and differences with other regexp engines

<br>

#### <a name="gotchas-and-tricks"></a>gotchas and tricks

* input record separator being part of input record

```bash
$ # newline character gets replaced too as shown by shell prompt
$ echo 'foo:123:bar:789' | ruby -pe 'sub(/[^:]+$/, "xyz")'
foo:123:bar:xyz$
$ # simple workaround is to use -l option
$ echo 'foo:123:bar:789' | ruby -lpe 'sub(/[^:]+$/, "xyz")'
foo:123:bar:xyz

$ # of course it is useful too
$ # same as: perl -pe 's/\n/ : / if !eof'
$ seq 10 | ruby -pe 'sub(/\n/, " : ") if !$<.eof'
1 : 2 : 3 : 4 : 5 : 6 : 7 : 8 : 9 : 10
```

* how much does `*` match?

```bash
$ # both empty and non-empty strings are matched
$ # even though * is a greedy quantifier
$ echo ',baz,,xyz,,,' | ruby -lpe 'gsub(/[^,]*/, "A")'
A,AA,A,AA,A,A,A
$ echo 'foo,baz,,xyz,,,123' | ruby -lpe 'gsub(/[^,]*/, "A")'
AA,AA,A,AA,A,A,AA

$ # one workaround is to use lookarounds(covered later)
$ echo ',baz,,xyz,,,' | ruby -lpe 'gsub(/(?<=^|,)[^,]*/, "A")'
A,A,A,A,A,A,A
$ echo 'foo,baz,,xyz,,,123' | ruby -lpe 'gsub(/(?<=^|,)[^,]*/, "A")'
A,A,A,A,A,A,A
```

* difference between `^` and `\A`

```bash
$ # ^ matches start of line, not start of string
$ # same as: perl -00 -ne 'print if /^Believe/m' sample.txt
$ ruby -00 -ne 'print if /^Believe/' sample.txt
Just do-it
Believe it

$ ruby -00 -ne 'print if /^he/i' sample.txt
Hello World

Much ado about nothing
He he he

$ # \A matches start of string
$ # without m modifier, both ^ and \A will match start of string in perl
$ ruby -00 -ne 'print if /\Ahe/i' sample.txt
Hello World

$ # similarly, $ matches end of line
$ ruby -00 -ne 'print if /funny$/' sample.txt
Today is sunny
Not a bit funny
No doubt you like it too
```

* difference between `\z` and `\Z`

```bash
$ # \Z matches just before newline
$ seq 14 | ruby -ne 'print if /2\Z/'
2
12

$ # \z matches end of string
$ seq 14 | ruby -ne 'print if /2\z/'
$ seq 14 | ruby -ne 'print if /2\n\z/'
2
12

$ # without newline at end of line, both \z and \Z will behave same
$ seq 14 | ruby -lne 'print if /2\z/'
2
12
```

* delimiters and quoting
* from [ruby-doc: Percent Strings](https://ruby-doc.org/core-2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings)

> If you are using “(”, “[”, “{”, “<” you must close it with “)”, “]”, “}”, “>” respectively. You may use most other non-alphanumeric characters for percent string delimiters such as “%”, “|”, “^”, etc.

```bash
$ # %r allows to use delimiter other than /
$ echo 'a/b' | ruby -pe 'sub(/a\/b/, "foo")'
foo
$ echo 'a/b' | ruby -pe 'sub(%r{a/b}, "foo")'
foo

$ # use %q (single quoting) to avoid variable interpolation
$ echo 'foo123' | ruby -pe 'a="huh?"; sub(/12/, "#{a}")'
foohuh?3
$ echo 'foo123' | ruby -pe 'a="huh?"; sub(/12/, %q/#{a}/)'
foo#{a}3

$ # %q also useful for backreferences, as \ is special inside double quotes
$ echo 'a a a 2 be be' | ruby -pe 'gsub(/\b(\w+)( \1)+\b/, "\\1")'
a 2 be
$ echo 'a a a 2 be be' | ruby -pe 'gsub(/\b(\w+)( \1)+\b/, %q/\1/)'
a 2 be
$ # and when double quotes is part of replacement string
$ echo '42,789' | ruby -lpe 'gsub(/\d+/, "\"\\0\"")'
"42","789"
$ echo '42,789' | ruby -lpe 'gsub(/\d+/, %q/"\0"/)'
"42","789"
$ # \& can also be used instead of \0
```

<br>

#### <a name="backslash-sequences"></a>Backslash sequences

* `\w` for `[A-Za-z0-9_]`
* `\d` for `[0-9]`
* `\s` for `[ \t\r\n\f\v]`
* `\h` for `[0-9a-fA-F]` or `[[:xdigit:]]`
* `\W`, `\D`, `\S`, `\H`, respectively for their opposites
* See also [ruby-doc: scan](https://ruby-doc.org/core-2.5.0/String.html#method-i-scan)

```bash
$ # same as: perl -ne 'print if /^[[:xdigit:]]+$/'
$ # can also use: ruby -lne 'print if !/\H/'
$ printf '128A\n34\nfe32\nfoo1\nbar\n' | ruby -ne 'print if /^\h+$/'
128A
34
fe32

$ # same as: perl -pe 's/\d+/xxx/g'
$ echo 'like 42 and 37' | ruby -pe 'gsub(/\d+/, "xxx")'
like xxx and xxx

$ # note again the use of -l because of newline in input record
$ # same as: perl -lpe 's/\D+/xxx/g'
$ echo 'like 42 and 37' | ruby -lpe 'gsub(/\D+/, "xxx")'
xxx42xxx37

$ # get all matches as an array
$ echo 'tea sea-pit sit' | ruby -ne 'puts $_.scan(/[\w\s]+/)'
tea sea
pit sit
```

<br>

#### <a name="non-greedy-quantifier"></a>Non-greedy quantifier

* adding a `?` to `?` or `*` or `+` or `{}` quantifiers will change matching from greedy to non-greedy. In other words, to match as minimally as possible
    * also known as lazy quantifier

```bash
$ # greedy matching
$ echo 'foo and bar and baz land good' | ruby -lne 'print $_.scan(/.*and/)'
["foo and bar and baz land"]
$ # non-greedy matching
$ echo 'foo and bar and baz land good' | ruby -lne 'print $_.scan(/.*?and/)'
["foo and", " bar and", " baz land"]

$ echo '12342789' | ruby -pe 'sub(/\d{2,5}/, "")'
789
$ echo '12342789' | ruby -pe 'sub(/\d{2,5}?/, "")'
342789

$ # for single character, non-greedy is not always needed
$ echo '123:42:789:good:5:bad' | ruby -pe 'sub(/:.*?:/, ":")'
123:789:good:5:bad
$ echo '123:42:789:good:5:bad' | ruby -pe 'sub(/:[^:]*:/, ":")'
123:789:good:5:bad

$ # just like greedy, overall matching is considered, as minimal as possible
$ echo '123:42:789:good:5:bad' | ruby -pe 'sub(/:.*?:[a-z]/, ":")'
123:ood:5:bad
$ echo '123:42:789:good:5:bad' | ruby -pe 'sub(/:.*:[a-z]/, ":")'
123:ad
```

<br>

#### <a name="lookarounds"></a>Lookarounds

* Ability to add if conditions to match before/after required pattern
* There are four types
    * positive lookahead `(?=`
    * negative lookahead `(?!`
    * positive lookbehind `(?<=`
    * negative lookbehind `(?<!`
* One way to remember is that **behind** uses `<` and **negative** uses `!` instead of `=`

The string matched by lookarounds are like word boundaries and anchors, do not constitute as part of matched string. They are termed as **zero-width patterns**

* positive lookbehind `(?<=`

```bash
$ s='foo=5, bar=3; x=83, y=120'

$ # extract all digit sequences, same as: perl -lne 'print join " ", /\d+/g'
$ echo "$s" | ruby -lne 'print $_.scan(/\d+/).join(" ")'
5 3 83 120

$ # extract digits only if preceded by two lowercase alphabets and =
$ # note how the characters matched by lookbehind isn't part of output
$ # same as: perl -lne 'print join " ", /(?<=[a-z]{2}=)\d+/g'
$ echo "$s" | ruby -lne 'print $_.scan(/(?<=[a-z]{2}=)\d+/).join(" ")'
5 3
$ # this can be done without lookbehind too
$ echo "$s" | ruby -lne 'print $_.scan(/[a-z]{2}=(\d+)/).join(" ")'
5 3

$ # change all digits preceded by single lowercase alphabet and =
$ # same as: perl -pe 's/(?<=\b[a-z]=)\d+/42/g'
$ echo "$s" | ruby -pe 'gsub(/(?<=\b[a-z]=)\d+/, "42")'
foo=5, bar=3; x=42, y=42
```

* positive lookahead `(?=`

```bash
$ s='foo=5, bar=3; x=83, y=120'

$ # extract digits that end with ,
$ # same as: perl -lne 'print join ":", /\d+(?=,)/g'
$ echo "$s" | ruby -lne 'print $_.scan(/\d+(?=,)/).join(":")'
5:83

$ # change all digits ending with ,
$ # same as: perl -pe 's/\d+(?=,)/42/g'
$ echo "$s" | ruby -pe 'gsub(/\d+(?=,)/, "42")'
foo=42, bar=3; x=42, y=120

$ # both lookbehind and lookahead
$ echo 'foo,,baz,,,xyz' | ruby -pe 'gsub(/,,/, ",NA,")'
foo,NA,baz,NA,,xyz
$ echo 'foo,,baz,,,xyz' | ruby -pe 'gsub(/(?<=,)(?=,)/, "NA")'
foo,NA,baz,NA,NA,xyz
```

* negative lookbehind `(?<!` and negative lookahead `(?!`

```bash
$ # change foo if not preceded by _
$ # note how 'foo' at start of line is matched as well
$ # same as: perl -pe 's/(?<!_)foo/baz/g'
$ echo 'foo _foo 1foo' | ruby -pe 'gsub(/(?<!_)foo/, "baz")'
baz _foo 1baz

$ # join each line in paragraph by replacing newline character
$ # except the one at end of paragraph
$ # same as: perl -00 -pe 's/\n(?!$)/. /g' sample.txt
$ ruby -00 -pe 'gsub(/\n(?!$)/, ". ")' sample.txt
Hello World

Good day. How are you

Just do-it. Believe it

Today is sunny. Not a bit funny. No doubt you like it too

Much ado about nothing. He he he
```

* capture groups can also be used inside lookarounds

```bash
$ # same as: perl -pe 's/(\H+\h+)(?=(\H+)\h)/$1$2\n/g'
$ # %q cannot be used here as \n is not meaningful inside single quotes
$ echo 'a b c d e' | ruby -lpe 'gsub(/(\S+\s+)(?=(\S+)\s)/, "\\1\\2\n")'
a b
b c
c d
d e
```

* `\K` helps as a workaround for some of the variable-length lookbehind cases
* See also [stackoverflow - Variable-length lookbehind-assertion alternatives](https://stackoverflow.com/questions/11640447/variable-length-lookbehind-assertion-alternatives-for-regular-expressions)

```bash
$ echo '1 and 2 and 3 land 4' | ruby -pe 'sub(/(?<=(and.*?){2})and/, "-")'
-e:1: invalid pattern in look-behind: /(?<=(and.*?){2})and/

$ # \K helps in such cases
$ # same as: sed 's/and/-/3' and perl -pe 's/(and.*?){2}\Kand/-/'
$ echo '1 and 2 and 3 land 4' | ruby -pe 'sub(/(and.*?){2}\Kand/, "-")'
1 and 2 and 3 l- 4
```

* don't use `\K` if there are consecutive matches
* this is because of how the regexp engine has been implemented, `perl` or `vim`'s `\zs` don't have this limitation

```bash
$ echo ',,' | perl -pe 's/,\K/foo/g'
,foo,foo
$ echo ',,' | ruby -pe 'gsub(/,\K/, "foo")'
,foo,
$ echo ',,' | ruby -pe 'gsub(/(?<=,)/, "foo")'
,foo,foo

$ # another example
$ echo '"foo","12,34","good"' | perl -F'/"\K,(?=")/' -lane 'print $F[1]'
"12,34"
$ echo '"foo","12,34","good"' | ruby -F'"\K,(?=")' -lane 'print $F[1]'
"12,34
$ echo '"foo","12,34","good"' | ruby -F'(?<="),(?=")' -lane 'print $F[1]'
"12,34"
```

<br>

#### <a name="special-capture-groups"></a>Special capture groups

* `\1`, `\2` etc only matches exact string
* `\g<1>`, `\g<2>` etc re-uses the regular expression itself

```bash
$ s='baz 2008-03-24 and 2012-08-12 foo 2016-03-25'
$ # same as: perl -pe 's/(\d{4}-\d{2}-\d{2}) and (?1)/XYZ/'
$ echo "$s" | ruby -pe 'sub(/(\d{4}-\d{2}-\d{2}) and \g<1>/, "XYZ")'
baz XYZ foo 2016-03-25

$ # using \1 won't work as the two dates are different
$ echo "$s" | ruby -pe 'sub(/(\d{4}-\d{2}-\d{2}) and \1/, "")'
baz 2008-03-24 and 2012-08-12 foo 2016-03-25
```

* use `(?:` to group regular expressions without capturing it, so this won't be counted for backreference
* See also [stackoverflow - what is non-capturing group](https://stackoverflow.com/questions/3512471/what-is-a-non-capturing-group-what-does-do)

```bash
$ # using ?: helps to focus only on required capture groups
$ # same as: perl -pe 's/(?:co|fo)\K(\w)(\w)/$2$1/g'
$ echo 'cod1 foo_bar' | ruby -pe 'gsub(/(?:co|fo)\K(\w)(\w)/, %q/\2\1/)'
co1d fo_obar

$ # without ?: you'd need to remember all the other groups as well
$ echo 'cod1 foo_bar' | ruby -pe 'gsub(/(co|fo)\K(\w)(\w)/, %q/\3\2/)'
co1d fo_obar
```

* named capture groups `(?<name>` or `(?'name'`
* for backreference, use `\k<name>`
* both named capture groups and normal capture groups cannot be used at the same time

```bash
$ # same as: perl -pe 's/(?<fw>\w+) (?<sw>\w+)/$+{sw} $+{fw}/'
$ echo 'foo 123' | ruby -pe 'sub(/(?<fw>\w+) (?<sw>\w+)/, %q/\k<sw> \k<fw>/)'
123 foo

$ # also useful to transform different capture groups
$ s='"foo,bar",123,"x,y,z",42'
$ # same as: perl -lpe 's/"(?<a>[^"]+)",|(?<a>[^,]+),/$+{a}|/g'
$ echo "$s" | ruby -lpe 'gsub(/"(?<a>[^"]+)",|(?<a>[^,]+),/, %q/\k<a>|/)'
foo,bar|123|x,y,z|42
```

**Further Reading**

* [rexegg - all the (? usages](https://www.rexegg.com/regex-disambiguation.html)
* [regular-expressions - recursion](https://www.regular-expressions.info/recurse.html#balanced)
* [stackoverflow - Recursive nested matching pairs of curly braces](https://stackoverflow.com/questions/19486686/recursive-nested-matching-pairs-of-curly-braces-in-ruby-regex)

<br>

#### <a name="modifiers"></a>Modifiers

* use `i` modifier to ignore case while matching

```bash
$ ruby -ne 'print if /rose/i' poem.txt
Roses are red,

$ echo 'foo 123 FoO' | ruby -pe 'gsub(/foo/i, "good")'
good 123 good
```

* by default, `.` doesn't match the newline character
* `m` modifier allows `.` metacharacter to match newline character as well

```bash
$ # searching for a match which can span across multiple lines

$ # no output as . doesn't match newline
$ ruby -00 -ne 'print if /do.*he/' sample.txt

$ # same as: perl -00 -ne 'print if /do.*he/s' sample.txt
$ ruby -00 -ne 'print if /do.*he/m' sample.txt
Much ado about nothing
He he he
```

<br>

#### <a name="code-in-replacement-section"></a>Code in replacement section

* block form allows to use `ruby` code for replacement section

quoting from [ruby-doc: gsub](https://ruby-doc.org/core-2.5.0/String.html#method-i-gsub)

>In the block form, the current match string is passed in as a parameter, and variables such as $1, $2, $`, $&, and $' will be set appropriately. The value returned by the block will be substituted for the match on each call.

* `$1`, `$2`, etc are equivalent of `\1`, `\2`, etc
* `$&` is equivalent of `\&`(or `\0`) - i.e the entire matched string


```bash
$ # replace numbers with their squares, same as: perl -pe 's/\d+/$&**2/ge'
$ echo '4 and 10' | ruby -pe 'gsub(/\d+/){$&.to_i ** 2}'
16 and 100

$ # replace matched string with incremental value
$ # same as: perl -pe 's/\d+/++$c/ge'
$ echo '4 and 10 foo 57' | ruby -pe 'BEGIN{c=0}; gsub(/\d+/){c+=1}'
1 and 2 foo 3

$ # replace with string length, same as: perl -pe 's/\w+/length($&)/ge'
$ echo 'food:12:explain:789' | ruby -pe 'gsub(/\w+/){$&.length}'
4:2:7:3

$ # formatting string, same as: perl -lpe 's/[^-]+/sprintf "%04s", $&/ge'
$ echo 'a1-2-deed' | ruby -lpe 'gsub(/[^-]+/){ $&.rjust(4, "0") }'
00a1-0002-deed

$ # applying another substitution to matched string
$ # same as: perl -pe 's/"[^"]+"/$&=~s|a|A|gr/ge'
$ echo '"mango" and "guava"' | ruby -pe 'gsub(/"[^"]+"/){$&.gsub(/a/, "A")}'
"mAngo" and "guAvA"
```

* replacing specific occurrence

```bash
$ # replacing 2nd occurrence, same as: sed 's/:/-/2'
$ # same as: perl -pe '$c=0; s/:/++$c==2 ? "-" : $&/ge'
$ echo 'foo:123:bar:baz' | ruby -pe 'c=0; gsub(/:/){(c+=1)==2 ? "-" : $&}'
foo:123-bar:baz
$ # or use non-greedy matching, same as: sed 's/and/-/3'
$ echo 'foo and bar and baz land good' | ruby -pe 'sub(/(and.*?){2}\Kand/, "-")'
foo and bar and baz l- good

$ # emulating GNU sed's number+g modifier
$ a='456:foo:123:bar:789:baz
x:y:z:a:v:xc:gf'
$ echo "$a" | sed 's/:/-/3g'
456:foo:123-bar-789-baz
x:y:z-a-v-xc-gf
$ # same as: perl -pe '$c=0; s/:/++$c<3 ? $& : "-"/ge'
$ echo "$a" | ruby -pe 'c=0; gsub(/:/){(c+=1)<3 ? $& : "-"}'
456:foo:123-bar-789-baz
x:y:z-a-v-xc-gf
```

<br>

#### <a name="quoting-metacharacters"></a>Quoting metacharacters

* to match contents of string variable exactly, all metacharacters need to be escaped
* See [ruby-doc: Regexp.escape](https://ruby-doc.org/core-2.5.0/Regexp.html#method-c-escape) for syntax details

```bash
$ cat eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # since + is a metacharacter, no match found
$ # note that #{} allows interpolation
$ s='a+b' ruby -ne 'print if /#{ENV["s"]}/' eqns.txt

$ # same as: s='a+b' perl -ne 'print if /\Q$ENV{s}/' eqns.txt
$ s='a+b' ruby -ne 'print if /#{Regexp.escape(ENV["s"])}/' eqns.txt
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a+b

$ # use regexp as needed around variable content, for ex: end of line anchor
$ ruby -pe 'BEGIN{s="a+b"}; sub(/#{Regexp.escape(s)}$/, "a**b")' eqns.txt
a=b,a-b=c,c*d
a+b,pi=3.14,5e12
i*(t+9-g)/8,4-a**b
```

<br>

## <a name="two-file-processing"></a>Two file processing

First, a bit about `ARGV` which allows to keep track of which file is being processed

```bash
$ # similar to: perl -lne 'print $#ARGV' <(seq 2) <(seq 3) <(seq 1)
$ ruby -ne 'puts ARGV.length' <(seq 2) <(seq 3) <(seq 1)
2
2
1
1
1
0
```

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

* `-r` command line option allows to specify library required
    * the `include?` method allows to check if `set` already contains the element
    * See [ruby-doc: include?](https://ruby-doc.org/stdlib-2.5.0/libdoc/set/rdoc/Set.html#method-i-include-3F) for syntax details

```bash
$ # common lines
$ # note that all duplicates matching in second file would get printed
$ # same as: perl -ne 'if(!$#ARGV){$h{$_}=1; next}
$ #            print if $h{$_}' colors_1.txt colors_2.txt
$ ruby -rset -ne 'BEGIN{s=Set.new}; s.add($_) && next if ARGV.length==1;
                  print if s.include?($_)' colors_1.txt colors_2.txt
Blue
Red

$ # lines from colors_2.txt not present in colors_1.txt
$ ruby -rset -ne 'BEGIN{s=Set.new}; s.add($_) && next if ARGV.length==1;
                  print if !s.include?($_)' colors_1.txt colors_2.txt
Black
Green
White

$ # next - to skip rest of code and process next input line
$ # here used to skip rest of code as long as first file is being processed
$ # alternate: ARGV.length==1 ? s.add($_) : s.include?($_) && print
```

alternate solution by using set operations available for arrays

* [ruby-doc: ARGF](https://ruby-doc.org/core-2.5.0/ARGF.html) filehandle allows to read from filename arguments supplied to script
    * if filename arguments are not present, it would act upon stdin
* `STDIN` filehandle allows to read from stdin
* [ruby-doc: readlines](https://ruby-doc.org/core-2.5.0/IO.html#method-c-readlines) method allows to read all the lines as an array
    * if filehandle is not specified, default is ARGF
* some comparison notes
    * both files will get saved as array in memory here, while previous solution would save only first file
    * duplicates would get removed here
    * likely to be faster compared to previous solution

```bash
$ # note that -n/-p options are not used
$ # and puts is helpful here as record separator is newline character

$ # common lines, output order is based on array to left of & operator
$ ruby -e 'f1=STDIN.readlines; f2=readlines;
           puts f1 & f2' <colors_1.txt colors_2.txt
Blue
Red

$ # lines from colors_2.txt not present in colors_1.txt
$ ruby -e 'f1=STDIN.readlines; f2=readlines;
           puts f2 - f1' <colors_1.txt colors_2.txt
Black
Green
White

$ # for union, use either of these
$ # ruby -e 'f1=STDIN.readlines; f2=readlines;
$ #          puts f1 | f2' <colors_1.txt colors_2.txt
$ # ruby -e 'puts readlines.uniq' colors_1.txt colors_2.txt
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
* For ex: only first field comparison instead of entire line as key

```bash
$ cat list1
ECE
CSE

$ # extract only lines matching first field specified in list1
$ ruby -rset -ane 'BEGIN{s=Set.new}; s.add($F[0]) && next if ARGV.length==1;
                   print if s.include?($F[0])' list1 marks.txt
ECE     Raj     53
ECE     Joel    72
CSE     Surya   81
ECE     Om      92
CSE     Amy     67
```

* multiple field comparison

```bash
$ cat list2
EEE Moi
CSE Amy
ECE Raj

$ # $F[0..1] will return array with elements specified by range (0 to 1 here)
$ ruby -rset -ane 'BEGIN{s=Set.new}; s.add($F[0..1]) && next if ARGV.length==1;
                   print if s.include?($F[0..1])' list2 marks.txt
ECE     Raj     53
EEE     Moi     68
CSE     Amy     67
```

* field and value comparison
* here, we use [hash](https://ruby-doc.org/core-2.5.0/Hash.html) as well to save values based on a key

```bash
$ cat list3
ECE 70
EEE 65
CSE 80

$ # extract line matching Dept and minimum marks specified in list3
$ ruby -rset -ane 'BEGIN{d=Set.new; m={}};
                   (d.add($F[0]); m[$F[0]]=$F[1]) && next if ARGV.length==1;
                   print if d.include?($F[0]) && $F[2]>=m[$F[0]]' list3 marks.txt
ECE     Joel    72
EEE     Moi     68
CSE     Surya   81
ECE     Om      92
```

<br>

#### <a name="line-number-matching"></a>Line number matching

```bash
$ # replace mth line in poem.txt with nth line from list1
$ # same as: m=3 n=2 perl -pe 'BEGIN{ $s=<> while $ENV{n}-- > 0; close ARGV}
$ #                    $_=$s if $.==$ENV{m}' list1 poem.txt
$ m=3 n=2 ruby -pe 'BEGIN{ENV["n"].to_i.times { $s=gets }; ARGF.close };
                    $_=$s if $.==ENV["m"].to_i' list1 poem.txt
Roses are red,
Violets are blue,
CSE
And so are you.

$ # print line from fruits.txt if corresponding line from nums.txt is +ve number
$ # same as: <nums.txt perl -ne 'print if <STDIN> > 0' fruits.txt
$ # line from fruits.txt is saved first as STDIN.gets will also set $_
$ <nums.txt ruby -ne 'ln=$_; print ln if STDIN.gets.to_i>0' fruits.txt
fruit   qty
banana  31
$ # can also use:
$ # ruby -e 'STDIN.readlines.zip(readlines).each {|a| puts a[1] if a[0].to_i>0}'
```

For syntax and implementation details, see

* [ruby-doc: ARGF](https://ruby-doc.org/core-2.5.0/ARGF.html)
* [ruby-doc: times](https://ruby-doc.org/core-2.5.0/Integer.html#method-i-times)
* [ruby-doc: gets](https://ruby-doc.org/core-2.5.0/IO.html#method-i-gets)

<br>

## <a name="creating-new-fields"></a>Creating new fields

* See [ruby-doc: slice](https://ruby-doc.org/core-2.5.0/Array.html#method-i-slice) for syntax details

```bash
$ s='foo,bar,123,baz'

$ # to reduce fields, use slice method
$ # same as: echo "$s" | perl -F, -lane '$,=","; $#F=1; print @F'
$ # 1st arg - starting index, 2nd arg - number of elements
$ echo "$s" | ruby -F, -lane '$F.slice!(-2,2); print $F * ","'
foo,bar

$ # assigning to field greater than length will create empty fields as needed
$ # same as: echo "$s" | perl -F, -lane '$,=","; $F[6]=42; print @F'
$ echo "$s" | ruby -F, -lane '$F[6]=42; print $F * ","'
foo,bar,123,baz,,,42
```

* adding a field based on existing fields
* See [ruby-doc: Percent Strings](https://ruby-doc.org/core-2.5.0/doc/syntax/literals_rdoc.html#label-Percent+Strings) for details on `%w`

```bash
$ # adding a new 'Grade' field
$ # same as: perl -lane 'BEGIN{$,="\t"; @g = qw(D C B A S)}
$ #          push @F, $.==1 ? "Grade" : $g[$F[-1]/10 - 5]; print @F' marks.txt
$ ruby -lane 'BEGIN{g = %w[D C B A S]};
              $F.push($.==1 ? "Grade" : g[$F[-1].to_i/10 - 5]);
              print $F * "\t"' marks.txt
Dept    Name    Marks   Grade
ECE     Raj     53      D
ECE     Joel    72      B
EEE     Moi     68      C
CSE     Surya   81      A
EEE     Tia     59      D
ECE     Om      92      S
CSE     Amy     67      C
```

<br>

## <a name="multiple-file-input"></a>Multiple file input

* processing based on line-number/begin/end of each input file

```bash
$ # same as: perl -ne 'print if $.==2; close ARGV if eof'
$ # ARGF.close will reset $. to 0
$ ruby -ne 'print if $.==2; ARGF.close if $<.eof' poem.txt greeting.txt
Violets are blue,
Have a safe journey

$ # same as: perl -lne 'print "file: $ARGV" if $.==1;
$ #            print "$_\n------" and close ARGV if eof' poem.txt greeting.txt
$ ruby -lne 'print "file: #{ARGF.filename}" if $.==1;
             (print "#{$_}\n------"; ARGF.close) if $<.eof' poem.txt greeting.txt
file: poem.txt
And so are you.
------
file: greeting.txt
Have a safe journey
------
```

* to skip remaining lines from current file being processed and move on to next file

```bash
$ # same as: perl -pe 'close ARGV if $.>=1' poem.txt greeting.txt fruits.txt
$ ruby -pe 'ARGF.close if $.>=1' poem.txt greeting.txt fruits.txt
Roses are red,
Hello there
fruit   qty

$ # same as: perl -lane 'print $ARGV and close ARGV if $F[0] =~ /red/i' *
$ ruby -ane '(puts ARGF.filename; ARGF.close) if $F[0] =~ /red/i' *
colors_1.txt
colors_2.txt
```

<br>

## <a name="dealing-with-duplicates"></a>Dealing with duplicates

* retain only first copy of duplicates
* `-r` command line option allows to specify library required
* here, `set` data type is used to keep track of unique values - be it whole line or a particular field
    * the `add?` method will add element to `set` and returns `nil` if element already exists
    * See [ruby-doc: add?](https://ruby-doc.org/stdlib-2.5.0/libdoc/set/rdoc/Set.html#method-i-add-3F) for syntax details

```bash
$ cat duplicates.txt
abc  7   4
food toy ****
abc  7   4
test toy 123
good toy ****

$ # whole line, same as: perl -ne 'print if !$seen{$_}++' duplicates.txt
$ ruby -rset -ne 'BEGIN{s=Set.new}; print if s.add?($_)' duplicates.txt
abc  7   4
food toy ****
test toy 123
good toy ****

$ # particular column, same as: perl -ane 'print if !$seen{$F[1]}++'
$ ruby -rset -ane 'BEGIN{s=Set.new}; print if s.add?($F[1])' duplicates.txt
abc  7   4
food toy ****

$ # total count, same as: perl -lane '$c++ if !$seen{$F[1]}++; END{print $c}'
$ ruby -rset -ane 'BEGIN{s=Set.new}; s.add($F[1]);
                   END{puts s.length}' duplicates.txt
2
```

* multiple fields

```bash
$ # same as: perl -ane 'print if !$seen{$F[1],$F[2]}++' duplicates.txt
$ # $F[1..2] will return an array with fields 2 and 3 as elements
$ ruby -rset -ane 'BEGIN{s=Set.new}; print if s.add?($F[1..2])' duplicates.txt
abc  7   4
food toy ****
test toy 123
```

* retaining only last copy of duplicate

```bash
$ # reverse the input line-wise, retain first copy and then reverse again
$ # same as: tac duplicates.txt | perl -ane 'print if !$seen{$F[1]}++' | tac
$ tac duplicates.txt | ruby -rset -ane 'BEGIN{s=Set.new};
                       print if s.add?($F[1])' | tac
abc  7   4
good toy ****
```

* for count based filtering (other than first/last count), use a `hash`
* `Hash.new(0)` will initialize value of new key to `0`

```bash
$ # second occurrence of duplicate
$ # same as: perl -ane 'print if ++$h{$F[1]}==2' duplicates.txt
$ ruby -ane 'BEGIN{h=Hash.new(0)}; print if (h[$F[1]]+=1)==2' duplicates.txt
abc  7   4
test toy 123

$ # third occurrence of duplicate
$ # same as: perl -ane 'print if ++$h{$F[1]}==3' duplicates.txt
$ ruby -ane 'BEGIN{h=Hash.new(0)}; print if (h[$F[1]]+=1)==3' duplicates.txt
good toy ****
```

* filtering based on duplicate count
* allows to emulate [uniq](./sorting_stuff.md#uniq) command for specific fields

```bash
$ # all duplicates based on 1st column
$ # same as: perl -ane '!$#ARGV ? $x{$F[0]}++ : $x{$F[0]}>1 && print'
$ ruby -ane 'BEGIN{h=Hash.new(0)}; ARGV.length==1 ? h[$F[0]]+=1 :
              h[$F[0]]>1 && print' duplicates.txt duplicates.txt
abc  7   4
abc  7   4

$ # more than 2 duplicates based on 2nd column
$ ruby -ane 'BEGIN{h=Hash.new(0)}; ARGV.length==1 ? h[$F[1]]+=1 :
              h[$F[1]]>2 && print' duplicates.txt duplicates.txt
food toy ****
test toy 123
good toy ****

$ # only unique lines based on 3rd column
$ ruby -ane 'BEGIN{h=Hash.new(0)}; ARGV.length==1 ? h[$F[2]]+=1 :
              h[$F[2]]==1 && print' duplicates.txt duplicates.txt
test toy 123
```

<br>

#### <a name="using-uniq-method"></a>using uniq method

* [ruby-doc: uniq](https://ruby-doc.org/core-2.5.0/Array.html#method-i-uniq)
* original order is maintained

```bash
$ # same as: ruby -rset -ne 'BEGIN{s=Set.new}; print if s.add?($_)'
$ ruby -e 'puts readlines.uniq' duplicates.txt
abc  7   4
food toy ****
test toy 123
good toy ****

$ # same as: ruby -rset -ane 'BEGIN{s=Set.new}; print if s.add?($F[1])'
$ ruby -e 'puts readlines.uniq {|s| s.split[1]}' duplicates.txt
abc  7   4
food toy ****

$ # same as: ruby -rset -ane 'BEGIN{s=Set.new}; print if s.add?($F[1..2])'
$ ruby -e 'puts readlines.uniq {|s| s.split[1..2]}' duplicates.txt
abc  7   4
food toy ****
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
$ # same as: perl -ne '$f=1 if /BEGIN/; print if $f; $f=0 if /END/'
$ ruby -ne '$f=1 if /BEGIN/; print if $f==1; $f=0 if /END/' range.txt
BEGIN
1234
6789
END
BEGIN
a
b
c
END

$ # can also use: ruby -ne 'print if /BEGIN/../END/' range.txt
$ # which is similar to sed -n '/BEGIN/,/END/p'
$ # but not suitable to extend for other cases
```

* other variations

```bash
$ # exclude both starting/ending REGEXP
$ # same as: perl -ne '$f=0 if /END/; print if $f; $f=1 if /BEGIN/'
$ ruby -ne '$f=0 if /END/; print if $f==1; $f=1 if /BEGIN/' range.txt
1234
6789
a
b
c

$ # check out what these do:
$ ruby -ne '$f=1 if /BEGIN/; $f=0 if /END/; print if $f==1' range.txt
$ ruby -ne 'print if $f==1; $f=0 if /END/; $f=1 if /BEGIN/' range.txt
```

* Extracting lines other than lines between the two *REGEXP*s

```bash
$ # same as: perl -ne '$f=1 if /BEGIN/; print if !$f; $f=0 if /END/'
$ # can also use: ruby -ne 'print if !(/BEGIN/../END/)' range.txt
$ ruby -ne '$f=1 if /BEGIN/; print if $f!=1; $f=0 if /END/' range.txt
foo
bar
baz

$ # the other three cases would be
$ ruby -ne '$f=0 if /END/; print if $f!=1; $f=1 if /BEGIN/' range.txt
$ ruby -ne 'print if $f!=1; $f=1 if /BEGIN/; $f=0 if /END/' range.txt
$ ruby -ne '$f=1 if /BEGIN/; $f=0 if /END/; print if $f!=1' range.txt
```

<br>

#### <a name="specific-blocks"></a>Specific blocks

* Getting first block

```bash
$ # same as: perl -ne '$f=1 if /BEGIN/; print if $f; exit if /END/'
$ ruby -ne '$f=1 if /BEGIN/; print if $f==1; exit if /END/' range.txt
BEGIN
1234
6789
END

$ # use other tricks discussed in previous section as needed
$ ruby -ne 'exit if /END/; print if $f==1; $f=1 if /BEGIN/' range.txt
1234
6789
```

* Getting last block

```bash
$ # reverse input linewise, change the order of REGEXPs, finally reverse again
$ tac range.txt | ruby -ne '$f=1 if /END/; print if $f==1; exit if /BEGIN/' | tac
BEGIN
a
b
c
END

$ # or, save the blocks in a buffer and print the last one alone
$ # same as: seq 30 | perl -ne 'if(/4/){$f=1; $b=$_; next}
$ #                     $b.=$_ if $f; $f=0 if /6/; END{print $b}'
$ # << operator concatenates given string to the variable in-place
$ seq 30 | ruby -ne '($f=1; $b=$_) && next if /4/;
                     $b << $_ if $f==1; $f=0 if /6/; END{print $b}'
24
25
26
```

* Getting blocks based on a counter

```bash
$ # get only 2nd block
$ # same as: b=2 perl -ne '$c++ if /4/; if($c==$ENV{b}){print; exit if /6/}'
$ seq 30 | b=2 ruby -ne 'BEGIN{c=0}; c+=1 if /4/;
                         c==ENV["b"].to_i && (print; exit if /6/)'
14
15
16

$ # to get all blocks greater than 'b' blocks
$ seq 30 | b=1 ruby -ne 'BEGIN{c=0}; ($f=1; c+=1) if /4/;
                         print if $f==1 && c>ENV["b"].to_i; $f=0 if /6/'
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
$ seq 30 | b=2 ruby -ne 'BEGIN{c=0}; ($f=1; c+=1) if /4/;
                         print if $f==1 && c!=ENV["b"].to_i; $f=0 if /6/'
4
5
6
24
25
26
```

* extract block only if it matches another string as well

```bash
$ # string to match inside block: 23
$ # same as: perl -ne 'if(/BEGIN/){$f=1; $m=0; $b=""}; $m=1 if $f && /23/;
$ #            $b.=$_ if $f; if(/END/){print $b if $m; $f=0}' range.txt
$ ruby -ne '($f=1; $m=0; $b="") if /BEGIN/; $m=1 if $f==1 && /23/;
            $b<<$_ if $f==1; (print $b if $m==1; $f=0) if /END/' range.txt
BEGIN
1234
6789
END

$ # line to match inside block: 5 or 25
$ seq 30 | ruby -ne '($f=1; $m=0; $b="") if /4/; $m=1 if $f==1 && /^2?5$/;
                     $b<<$_ if $f==1; (print $b if $m==1; $f=0) if /6/'
4
5
6
24
25
26
```

<br>

#### <a name="broken-blocks"></a>Broken blocks

* If there are blocks with ending *REGEXP* but without corresponding start, earlier techniques used will suffice
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
$ tac broken_range.txt | ruby -ne '$f=1 if /END/;
                         print if $f==1; $f=0 if /BEGIN/' | tac
BEGIN
1234
6789
END
```

* But if both kinds of broken blocks are present, for ex:

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
xyzabc
```

then use buffers to accumulate the records and print accordingly

```bash
$ # same as: perl -ne 'if(/BEGIN/){$f=1; $b=$_; next} $b.=$_ if $f;
$ #            if(/END/){$f=0; print $b if $b; $b=""}' multiple_broken.txt
$ ruby -ne '($f=1; $b=$_) && next if /BEGIN/; $b << $_ if $f==1;
            ($f=0; print $b if $b!=""; $b="") if /END/' multiple_broken.txt
BEGIN
1234
6789
END
BEGIN
b
END

$ # note how buffer is initialized as well as cleared
$ # on matching beginning/end REGEXPs respectively
```

<br>

## <a name="array-operations"></a>Array operations

See [ruby-doc: Array](https://ruby-doc.org/core-2.5.0/Array.html) for various ways to initialize and methods available

* initialization

```bash
$ # as comma separated values, indexing starts at 0
$ ruby -le 'sq = [1, 4, 9, 16]; print sq[2]'
9
$ ruby -le 'a = [123, "foo", "baz789"]; print a[1]'
foo
$ # -ve indexing, -1 for last element, -2 for second last, etc
$ ruby -le 'foo = [2, "baz", ["a", "b"]]; print foo[-1]'
["a", "b"]

$ # variables can be used, double quoted string will interpolate
$ ruby -le 'a=5; b=["a", "b"]; c=[a, 789, b]; print c'
[5, 789, ["a", "b"]]
$ ruby -le 'c=[89, "a\nb"]; print c[-1]'
a
b

$ # %w allows space separated string values, no interpolation
$ ruby -le 'b = %w[123 foo baz789]; print b[1]'
foo
$ ruby -le 's = %w[foo "baz" "a\nb"]; print s[-1]'
"a\nb"
```

* array slices
* See also [ruby-doc: Array to Arguments Conversion](https://ruby-doc.org/core-2.5.0/doc/syntax/calling_methods_rdoc.html#label-Array+to+Arguments+Conversion)

```bash
$ # accessing more than one element in random order
$ echo 'a b c d' | ruby -lane 'print $F.values_at(0,-1,2) * " "'
a d c
$ echo 'a b c d' | ruby -lane 'i=[0, -1, 2]; print $F.values_at(*i) * " "'
a d c

$ # starting index and number of elements needed from that index
$ echo 'a b c d' | ruby -lane 'print $F[0,3] * " "'
a b c
$ # range operator, arguments are start/end indexes
$ echo 'a b c d' | ruby -lane 'print $F[1..3] * " "'
b c d

$ # n elements from start, can also use 'first' method instead of 'take'
$ echo 'a b c d' | ruby -lane 'print $F.take(2) * " "'
a b
$ # remaining elements after ignoring n elements from start
$ echo 'a b c d' | ruby -lane 'print $F.drop(3) * " "'
d
$ # n elements from end
$ echo 'a b c d' | ruby -lane 'print $F.last(3) * " "'
b c d
```

* looping

```bash
$ # by element value, use 'reverse_each' to iterate in reversed order
$ # can also use range here: ruby -e '(1..4).each {|n| puts n*2}'
$ ruby -e 'nums=[1, 2, 3, 4]; nums.each {|n| puts n*2}'
2
4
6
8

$ # by index
$ ruby -e 'books=%w[Elantris Martian Dune Alchemist]
           books.each_index {|i| puts "#{i+1}) #{books[i]}"}'
1) Elantris
2) Martian
3) Dune
4) Alchemist
```

<br>

#### <a name="filtering"></a>Filtering

* based on regexp

```ruby
$ s='foo:123:bar:baz'
$ echo "$s" | ruby -F: -lane 'print $F.grep(/[a-z]/) * ":"'
foo:bar:baz

$ words='tryst fun glyph pity why'
$ echo "$words" | ruby -lane 'puts $F.grep(/[a-g]/)'
fun
glyph

$ # grep_v inverts the selection
$ echo "$words" | ruby -lane 'puts $F.grep_v(/[aeiou]/)'
tryst
glyph
why
```

* use `select` or `reject` for generic conditions

```bash
$ # to get index instead of matches
$ s='foo:123:bar:baz'
$ echo "$s" | ruby -F: -lane 'print $F.each_index.select{|i| $F[i] =~ /[a-z]/}'
[0, 2, 3]

$ # based on numeric value
$ s='23 756 -983 5'
$ echo "$s" | ruby -lane 'print $F.select { |s| s.to_i < 100 } * " "'
23 -983 5

$ # filters only those elements with successful substitution
$ # for opposite, either use negated condition or use reject instead of select
$ echo "$s" | ruby -lane 'print $F.select { |s| s.sub!(/3/, "E") } * " "'
2E -98E
```

* random element(s)

```bash
$ s='65 23 756 -983 5'
$ echo "$s" | ruby -lane 'print $F.sample'
23
$ echo "$s" | ruby -lane 'print $F.sample'
5

$ echo "$s" | ruby -lane 'print $F.sample(2)'
["-983", "756"]
```

<br>

#### <a name="sorting"></a>Sorting

* [ruby-doc: sort](https://ruby-doc.org/core-2.5.0/Array.html#method-i-sort)
* See also [stackoverflow What does map(&:name) mean in Ruby?](https://stackoverflow.com/questions/1217088/what-does-mapname-mean-in-ruby) for explanation on `&:`

```bash
$ s='foo baz v22 aimed'
$ # same as: perl -lane 'print join " ", sort @F'
$ echo "$s" | ruby -lane 'print $F.sort * " "'
aimed baz foo v22

$ # demonstrating the <=> operator
$ ruby -e 'puts 4 <=> 2'
1
$ ruby -e 'puts 4 <=> 20'
-1
$ ruby -e 'puts 4 <=> 4'
0

$ # descending order
$ # same as: perl -lane 'print join " ", sort {$b cmp $a} @F'
$ echo "$s" | ruby -lane 'print $F.sort { |a,b| b <=> a } * " "'
v22 foo baz aimed
$ # can also reverse the array after default sorting
$ echo "$s" | ruby -lane 'print $F.sort.reverse * " "'
v22 foo baz aimed
```

* using `sort_by` to sort based on a key

```bash
$ s='floor bat to dubious four'
$ # can also use: ruby -lane 'print $F.sort_by(&:length) * ":"'
$ echo "$s" | ruby -lane 'print $F.sort_by {|a| a.length} * ":"'
to:bat:four:floor:dubious

$ # for descending order, simply negate the key
$ echo "$s" | ruby -lane 'print $F.sort_by {|a| -a.length} * ":"'
dubious:floor:four:bat:to

$ # need to explicitly convert from string to number for numeric input
$ s='23 756 -983 5'
$ echo "$s" | ruby -lane 'print $F.sort_by(&:to_i) * " "'
-983 5 23 756
$ s='5.33:2.2e3:42'
$ echo "$s" | ruby -F: -lane 'print $F.sort_by{|n| -n.to_f} * ":"'
2.2e3:42:5.33
```

* sorting characters within word
* `chars` method returns array with individual characters

```bash
$ echo 'foobar' | ruby -lne 'print $_.chars.sort * ""'
abfoor

$ cat words.txt
bot
art
are
boat
toe
flee
reed

$ # words with characters in ascending order
$ # can also use: ruby -lne 'print if $_.chars == $_.chars.sort' words.txt
$ ruby -lne 'print if $_ == $_.chars.sort * ""' words.txt
bot
art

$ # words with characters in descending order
$ # can also use: ruby -lne 'print if $_.chars == $_.chars.sort.reverse'
$ ruby -lne 'print if $_ == $_.chars.sort {|a,b| b <=> a} * ""' words.txt
toe
reed
```

* sorting columns based on header

```bash
$ # need to get indexes of order required for header, then use it for all lines
$ # same as: perl -lane '@i = sort {$F[$a] cmp $F[$b]} 0..$#F if $.==1;
$ #              print join "\t", @F[@i]' marks.txt
$ ruby -lane 'idx = $F.each_index.sort {|i,j| $F[i] <=> $F[j]} if $.==1;
              print $F.values_at(*idx) * "\t"' marks.txt
Dept    Marks   Name
ECE     53      Raj
ECE     72      Joel
EEE     68      Moi
CSE     81      Surya
EEE     59      Tia
ECE     92      Om
CSE     67      Amy
```

* [ruby-doc: uniq](https://ruby-doc.org/core-2.5.0/Array.html#method-i-uniq)
* order is preserved

```bash
$ s='3,b,a,c,d,1,d,c,2,3,1,b'
$ # same as: perl -MList::MoreUtils=uniq -F, -lane 'print join ",",uniq @F'
$ echo "$s" | ruby -F, -lane 'print $F.uniq * ","'
3,b,a,c,d,1,2

$ # same as: ruby -rset -ane 'BEGIN{s=Set.new}; print if s.add?($F[1])'
$ # note that -n/-p option is not used
$ ruby -e 'puts readlines.uniq {|s| s.split[1]}' duplicates.txt
abc  7   4
food toy ****
```

* max/min values

```bash
$ # if numeric array is constructed from string input
$ echo '34,17,6' | ruby -F, -lane 'print $F.max {|a,b| a.to_i <=> b.to_i}'
34
$ # or convert numeric array first, 'map' is covered in next section
$ echo '34,17,6' | ruby -F, -lane 'print $F.map(&:to_i).max'
34
$ echo '23.5,42,-36' | ruby -F, -lane 'puts $F.map(&:to_f).max'
42.0

$ # string comparison is default
$ s='floor bat to dubious four'
$ echo "$s" | ruby -lane 'print $F.min'
bat

$ # can also get max/min 'n' elements
$ echo "$s" | ruby -lane 'print $F.max(2)'
["to", "four"]
$ echo "$s" | ruby -lane 'print $F.min(3) {|a,b| a.size <=> b.size}'
["to", "bat", "four"]
```

<br>

#### <a name="transforming"></a>Transforming

* shuffling elements

```bash
$ s='23 756 -983 5'
$ echo "$s" | ruby -lane 'print $F.shuffle * " "'
5 756 -983 23
$ echo "$s" | ruby -lane 'print $F.shuffle * " "'
756 5 23 -983

$ # randomizing file contents
$ # note that -n/-p option is not used
$ ruby -e 'puts readlines.shuffle' poem.txt
And so are you.
Violets are blue,
Roses are red,
Sugar is sweet,

$ # or if shuffle order is known
$ seq 5 | ruby -e 'puts readlines.values_at(3,1,0,2,4)'
4
2
1
3
5
```

* use `map` to transform every element
* See also [stackoverflow What does map(&:name) mean in Ruby?](https://stackoverflow.com/questions/1217088/what-does-mapname-mean-in-ruby) for explanation on `&:`

```bash
$ echo '23 756 -983 5' | ruby -lane 'print $F.map {|n| n.to_i ** 2} * " "'
529 571536 966289 25
$ echo 'a b c' | ruby -lane 'print $F.map {|s| %Q/"#{s}"/} * ","'
"a","b","c"
$ echo 'a b c' | ruby -lane 'print $F.map {|s| %Q/"#{s}"/.upcase} * ","'
"A","B","C"

$ # ASCII int values for each character
$ echo 'AaBbCc' | ruby -lne 'print $_.chars.map(&:ord) * " "'
65 97 66 98 67 99

$ echo '34,17,6' | ruby -F, -lane 'puts $F.map(&:to_i).sum'
57

$ # shuffle each field character wise
$ s='this is a sample sentence'
$ echo "$s" | ruby -lane 'print $F.map {|s| s.chars.shuffle * ""} * " "'
hsti si a mlepas esencnet
```

* reverse array/string

```bash
$ s='23 756 -983 5'
$ echo "$s" | ruby -lane 'print $F.reverse * " "'
5 -983 756 23

$ echo 'foobar' | ruby -lne 'print $_.reverse'
raboof
$ # or inplace reverse
$ echo 'foobar' | ruby -lpe '$_.reverse!'
raboof
```

* See also [ruby-doc: Enumerable](https://ruby-doc.org/core-2.5.0/Enumerable.html) for more methods like `inject`

<br>

## <a name="miscellaneous"></a>Miscellaneous

<br>

#### <a name="split"></a>split

* the `-a` command line option uses `split` and automatically saves the results in `$F` array
* default separator is `\s+` and also strips whitespace from start/end of string
* See also [ruby-doc: split](https://ruby-doc.org/core-2.5.0/String.html#method-i-split)

```bash
$ # specifying maximum number of splits
$ # same as: perl -lne 'print join ":", split /\s+/,$_,2'
$ echo 'a 1 b 2 c' | ruby -lne 'print $_.split(/\s+/, 2) * ":"'
a:1 b 2 c

$ # by default, trailing empty fields are stripped
$ echo ':123::' | ruby -lne 'print $_.split(/:/) * ","'
,123
$ # specify a negative count to preserve trailing empty fields
$ echo ':123::' | ruby -lne 'print $_.split(/:/, -1) * ","'
,123,,

$ # use string argument for fixed-string split instead of regexp
$ echo 'foo**123**baz' | ruby -lne 'print $_.split("**") * ":"'
foo:123:baz

$ # to save the separators as well, use capture groups
$ s='Sample123string54with908numbers'
$ echo "$s" | ruby -lne 'print $_.split(/(\d+)/) * ":"'
Sample:123:string:54:with:908:numbers
```

* single line to multiple line by splitting a column

```bash
$ cat split.txt
foo,1:2:5,baz
wry,4,look
free,3:8,oh

$ # same as: perl -F, -ane 'print join ",", $F[0],$_,$F[2] for split /:/,$F[1]'
$ ruby -F, -ane '$F[1].split(/:/).each {|x| print [$F[0],x,$F[2]]*","}' split.txt
foo,1,baz
foo,2,baz
foo,5,baz
wry,4,look
free,3,oh
free,8,oh
$ # can also use scan here:
$ # ruby -F, -ane '$F[1].scan(/[^:]+/) {|x| print [$F[0],x,$F[2]]*","}'
```

<br>

#### <a name="fixed-width-processing"></a>Fixed width processing

* [ruby-doc: unpack](https://ruby-doc.org/core-2.5.0/String.html#method-i-unpack)

```bash
$ # same as: perl -lne '@x = unpack("a1xa3xa4", $_); print $x[0]'
$ # here 'a' indicates arbitrary binary string
$ # the number that follows indicates length
$ # the 'x' indicates characters to ignore, use length after 'x' if needed
$ # and there are many other formats, see ruby-doc for details
$ echo 'b 123 good' | ruby -lne 'print $_.unpack("a1xa3xa4")[0]'
b
$ echo 'b 123 good' | ruby -lne 'print $_.unpack("a1xa3xa4")[1]'
123
$ echo 'b 123 good' | ruby -lne 'print $_.unpack("a1xa3xa4")[2]'
good

$ # unpack not always needed, simple slicing might help
$ echo 'b 123 good' | ruby -ne 'puts $_[2,3]'
123
$ echo 'b 123 good' | ruby -ne 'puts $_[6,4]'
good

$ # replacing arbitrary slice
$ # same as: perl -lpe 'substr $_, 2, 3, "gleam"'
$ echo 'b 123 good' | ruby -lpe '$_[2,3] = "gleam"'
b gleam good
```

<br>

#### <a name="string-and-file-replication"></a>String and file replication

```bash
$ # replicate each line, same as: perl -ne 'print $_ x 2'
$ seq 2 | ruby -ne 'print $_ * 2'
1
1
2
2

$ # replicate a string, same as: perl -le 'print "abc" x 5'
$ ruby -e 'puts "abc" * 5'
abcabcabcabcabc

$ # works for array too, but be careful with mutable elements
$ ruby -le 'x = [3, 2, 1] * 2; print x'
[3, 2, 1, 3, 2, 1]
$ ruby -le 'x = [3, 2, [1, 7]] * 2; x[2][0]="a"; print x'
[3, 2, ["a", 7], 3, 2, ["a", 7]]

$ # replicating file, same as: perl -0777 -ne 'print $_ x 100'
$ wc -c poem.txt
65 poem.txt
$ ruby -0777 -ne 'print $_ * 100' poem.txt | wc -c
6500
```

<br>

#### <a name="transliteration"></a>transliteration

* [ruby-doc: tr](https://ruby-doc.org/core-2.5.0/String.html#method-i-tr)

```bash
$ echo 'Uryyb Jbeyq' | ruby -pe '$_.tr!("a-zA-Z", "n-za-mN-ZA-M")'
Hello World
$ echo 'hi there!' | ruby -pe '$_.tr!("a-z", "\u{1d5ee}-\u{1d607}")'
𝗵𝗶 𝘁𝗵𝗲𝗿𝗲!

$ # when first argument is longer
$ # the last character of second argument is padded
$ echo 'foo bar cat baz' | ruby -pe '$_.tr!("a-z", "123")'
333 213 313 213

$ # use ^ at start of first argument to complement specified characters
$ echo 'foo:123:baz' | ruby -lpe '$_.tr!("^0-9", "-")'
----123----

$ # use empty second argument to delete specified characters
$ echo '"Foo1!", "Bar.", ":Baz:"' | ruby -lpe '$_.tr!("^A-Za-z,", "")'
Foo,Bar,Baz

$ # use - at start/end and ^ other than start to match themselves
$ echo 'a^3-b*d' | ruby -lpe '$_.tr!("-^*", "*/+")'
a/3*b+d
```

<br>

#### <a name="executing-external-commands"></a>Executing external commands

* External commands can be issued using `system` function
* Output would be as usual on `stdout` unless redirected while calling the command

```bash
$ # same as: perl -e 'system("echo Hello World")'
$ ruby -e 'system("echo Hello World")'
Hello World

$ ruby -e 'system("wc poem.txt")'
 4 13 65 poem.txt

$ ruby -e 'system("seq 10 | paste -sd, > out.txt")'
$ cat out.txt
1,2,3,4,5,6,7,8,9,10

$ cat f2
I bought two bananas and three mangoes
$ # same as: perl -F, -lane 'system "cat $F[1]"'
$ echo 'f1,f2,odd.txt' | ruby -F, -lane 'system("cat #{$F[1]}")'
I bought two bananas and three mangoes
```

* return value of `system` or global variable `$?` can be used to act upon exit status of command issued
* see [ruby-doc: system](https://ruby-doc.org/core-2.5.0/Kernel.html#method-i-system) for details

```bash
$ ruby -e 'es=system("ls poem.txt"); puts es'
poem.txt
true
$ ruby -e 'system("ls poem.txt"); puts $?'
poem.txt
pid 17005 exit 0

$ ruby -e 'system("ls xyz.txt"); puts $?'
ls: cannot access 'xyz.txt': No such file or directory
pid 17059 exit 2
```

* to save result of external command, use backticks or `%x`

```bash
$ ruby -e 'lines = `wc -l < poem.txt`; print lines'
4

$ ruby -e 'nums = %x/seq 3/; print nums'
1
2
3
```

* See also [stackoverflow - difference between exec, system and %x() or backticks](https://stackoverflow.com/questions/6338908/ruby-difference-between-exec-system-and-x-or-backticks)

<br>

## <a name="further-reading"></a>Further Reading

* Manual and related
    * [ruby-lang documentation](https://www.ruby-lang.org/en/documentation/) - manuals, tutorials and references
    * [ruby-lang - faqs](https://www.ruby-lang.org/en/documentation/faq/)
    * [ruby-lang - quickstart](https://www.ruby-lang.org/en/documentation/quickstart/)
    * [ruby-lang - To Ruby From Perl](https://www.ruby-lang.org/en/documentation/ruby-from-other-languages/to-ruby-from-perl/)
    * [rubular - Ruby regular expression editor](http://rubular.com/)
* Tutorials and Q&A
    * [Smooth Ruby One-Liners](https://dev.to/rpalo/smooth-ruby-one-liners-154) - simple intro to ruby one-liners
    * [Ruby one-liners](http://benoithamelin.tumblr.com/ruby1line) based on [awk one-liners](http://www.pement.org/awk/awk1line.txt)
    * [Ruby Tricks, Idiomatic Ruby, Refactorings and Best Practices](https://franzejr.github.io/best-ruby/index.html)
    * [freecodecamp - learning Ruby](https://medium.freecodecamp.org/learning-ruby-from-zero-to-hero-90ad4eecc82d)
    * [Ruby Regexp](https://leanpub.com/rubyregexp) ebook - step by step guide from beginner to advanced levels
    * [regex FAQ on SO](https://stackoverflow.com/questions/22937618/reference-what-does-this-regex-mean)
* Alternatives
    * [bioruby](https://github.com/bioruby/bioruby)
    * [perl](https://perldoc.perl.org/)
    * [unix.stackexchange - When to use grep, sed, awk, perl, etc](https://unix.stackexchange.com/questions/303044/when-to-use-grep-less-awk-sed)


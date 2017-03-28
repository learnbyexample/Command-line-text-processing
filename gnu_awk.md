## <a name="awk"></a>awk

>pattern scanning and text processing language

`awk` derives its name from authors Alfred Aho, Peter Weinberger and Brian Kernighan.

**syntax**

* `awk 'BEGIN {initialize} condition1 {stmts} condition2 {stmts}... END {finish}'`
	* `BEGIN {initialize}` used to initialize variables (could be user defined or awk variables or both), executed once - optional block
	* `condition1 {stmts} condition2 {stmts}...` action performed for every line of input, condition is optional, more than one block {} can be used with/without condition
	* `END {finish}` perform action once at end of program - optional block
* commands can be written in a file and passed using the `-f` option instead of writing it all on command line
    * for examples and details, refer to links given below

<br>

**Example input file**

```bash
$ cat test.txt 
abc  : 123 : xyz
3    : 32  : foo
-2.3 : bar : bar
```

* Just printing something, no input

```bash
$ awk 'BEGIN{print "Hello!\nTesting awk one-liner"}'
Hello!
Testing awk one-liner
```

* search and replace
* when the `{stmts}` portion of `condition {stmts}` is not specified, by default `{print $0}` is executed if the `condition` evaluates to true
    * `1` is a generally used `awk` idiom to print contents of `$0` after performing some processing
    * `print` statement without argument will print the content of `$0`

```bash
$ # sub will replace only first occurrence
$ # third argument to sub specifies variable to change, defaults to $0
$ awk '{sub("3", "%")} 1' test.txt 
abc  : 12% : xyz
%    : 32  : foo
-2.% : bar : bar

$ # gsub will replace all occurrences
$ awk '{gsub("3", "%")} 1' test.txt 
abc  : 12% : xyz
%    : %2  : foo
-2.% : bar : bar

$ # add a condition to restrict processing only to those records
$ awk '/foo/{gsub("3", "%")} 1' test.txt 
abc  : 123 : xyz
%    : %2  : foo
-2.3 : bar : bar

$ # using shell variables
$ r="@"
$ awk -v r_str="$r" '{sub("3", r_str)} 1' test.txt 
abc  : 12@ : xyz
@    : 32  : foo
-2.@ : bar : bar

$ # bash environment variables like PWD, HOME is also accessible via ENVIRON
$ s="%" awk '{sub("3", ENVIRON["s"])} 1' test.txt 
abc  : 12% : xyz
%    : 32  : foo
-2.% : bar : bar
```

* filtering content

```bash
$ # regex pattern, by default tested against $0
$ awk '/a/' test.txt 
abc  : 123 : xyz
-2.3 : bar : bar

$ # use ! to invert condition
$ awk '!/abc/' test.txt 
3    : 32  : foo
-2.3 : bar : bar

$ seq 30 | awk 'END{print}'
30

$ # generic, length(var) - default is $0
$ seq 8 13 | awk 'length==1'
8
9
```

* selecting based on line numbers
* `NR` is record number

```bash
$ seq 123 135 | awk 'NR==7'
129

$ seq 123 135 | awk 'NR>=3 && NR<=5'
125
126
127

$ seq 5 | awk 'NR>=3'
3
4
5

$ # for large input, use exit to avoid unnecessary record processing
$ seq 14323 14563435 | awk 'NR==234{print; exit}'
14556
```

* selecting based on start and end condition
* for following examples
    * numbers 1 to 20 is input
    * regex pattern `/4/` is start condition
    * regex pattern `/6/` is end condition
* `f` is idiomatically used to represent a flag variable

```bash
$ # records between start and end
$ seq 20 | awk '/4/{f=1; next} /6/{f=0} f'
5
15

$ # records between start and end and also includes start
$ seq 20 | awk '/4/{f=1} /6/{f=0} f'
4
5
14
15

$ # records between start and end and also includes end
$ seq 20 | awk '/4/{f=1; next} f; /6/{f=0}'
5
6
15
16

$ # records from start to end
$ seq 20 | awk '/4/{f=1} f{print} /6/{f=0}'
4
5
6
14
15
16

$ # records excluding start to end
$ seq 10 | awk '/4/{f=1} !f; /6/{f=0}'
1
2
3
7
8
9
10
```

* column manipulations
* by default, one or more consecutive spaces/tabs are considered as field separators

```bash
$ echo -e "1 3 4\na b c"
1 3 4
a b c

$ # second column
$ echo -e "1 3 4\na b c" | awk '{print $2}'
3
b

$ # last column
$ echo -e "1 3 4\na b c" | awk '{print $NF}'
4
c

$ # default output field separator is single space character
$ echo -e "1 3 4\na b c" | awk '{print $1, $3}'
1 4
a c

$ # condition for specific field
$ echo -e "1 3 4\na b c" | awk '$2 ~ /[0-9]/'
1 3 4
```

* specifying a different input/output field separator
* can be string alone or regex, multiple separators can be specified using `|` in regex pattern

```bash
$ awk -F' *: *' '$1 == "3"' test.txt 
3    : 32  : foo

$ awk -F' *: *' '{print $1 "," $2}' test.txt 
abc,123
3,32
-2.3,bar

$ awk -F' *: *' -v OFS="::" '{print $1, $2}' test.txt 
abc::123
3::32
-2.3::bar

$ awk -F: -v OFS="\t" '{print $1 OFS $2}' test.txt 
abc  	 123 
3    	 32  
-2.3 	 bar 
```

* dealing with duplicates, line/field wise

```bash
$ cat duplicates.txt 
abc 123 ijk
foo 567 xyz
abc 123 ijk
bar 090 pqr
tst 567 zzz

$ # whole line
$ awk '!seen[$0]++' duplicates.txt 
abc 123 ijk
foo 567 xyz
bar 090 pqr
tst 567 zzz

$ # particular column
$ awk '!seen[$2]++' duplicates.txt 
abc 123 ijk
foo 567 xyz
bar 090 pqr
```

* inplace editing

```bash
$ awk -i inplace '{print NR ") " $0}' test.txt
$ cat test.txt
1) abc  : 123 : xyz
2) 3    : 32  : foo
3) -2.3 : bar : bar
```

**Further Reading**

* [awk basics](http://code.snipcademy.com/tutorials/shell-scripting/awk/introduction)
* [Gawk: Effective AWK Programming](https://www.gnu.org/software/gawk/manual/)
* [awk detailed tutorial](http://www.grymoire.com/Unix/Awk.html)
* [basic tutorials for grep, awk, sed](https://unix.stackexchange.com/questions/2434/is-there-a-basic-tutorial-for-grep-awk-and-sed)
* [awk one-liners explained](http://www.catonmat.net/series/awk-one-liners-explained)
* [awk book](http://www.catonmat.net/blog/awk-book/)
* [awk cheat-sheet](http://www.catonmat.net/download/awk.cheat.sheet.txt) for awk variables, statements, functions, etc
* [awk examples](http://www.thegeekstuff.com/tag/unix-awk-examples/)
* [awk Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/awk?sort=votes&pageSize=15)
* [awk Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/awk?sort=votes&pageSize=15)


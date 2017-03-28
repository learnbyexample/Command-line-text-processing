## <a name="sed"></a>sed

>stream editor for filtering and transforming text

**Options**

* `-n` suppress automatic printing of pattern space
* `-i` edit files inplace (makes backup if SUFFIX supplied)
* `-r` use extended regular expressions
* `-e` add the script to the commands to be executed
* `-f` add the contents of script-file to the commands to be executed
    * for examples and details, refer to links given below

**commands**

We'll be seeing examples only for three commonly used commands

* `d` Delete the pattern space
* `p` Print out the pattern space
* `s` search and replace
* check out 'Often-Used Commands' and 'Less Frequently-Used Commands' sections in `info sed` for complete list of commands

**range**

By default, `sed` acts on all of input contents. This can be refined to specific line number or a range defined by line numbers, search pattern or mix of the two

* `n,m` range between nth line to mth line, including n and m
* `i~j` act on ith line and i+j, i+2j, i+3j, etc
    * `1~2` means 1st, 3rd, 5th, 7th, etc lines i.e odd numbered lines
    * `5~3` means 5th, 8th, 11th, etc 
* `n` only nth line
* `$` only last line
* `/pattern/` lines matching pattern
* `n,/pattern/` nth line to line matching pattern
* `n,+x` nth line and x lines after
* `/pattern/,m` line matching pattern to mth line
* `/pattern/,+x` line matching pattern and x lines after
* `/pattern1/,/pattern2/` line matching pattern1 to line matching pattern2
* `/pattern/I` lines matching pattern, pattern is case insensitive
* for more details, see section 'Selecting lines with sed' in `info sed`
* see 'Regular Expressions' in [grep command](https://github.com/learnbyexample/Linux_command_line/blob/master/Working_with_Files_and_Directories.md#grep) for extended regular expressions reference
* also check out 'Overview of Regular Expression Syntax' section in `info sed`

**Examples for selective deletion(d)**

* `sed '/cat/d' story.txt` delete every line containing cat
* `sed '/cat/!d' story.txt` delete every line NOT containing cat
* `sed '$d' story.txt` delete last line of the file
* `sed '2,5d' story.txt` delete lines 2,3,4,5 of the file
* `sed '1,/test/d' dir_list.txt` delete all lines from beginning of file to first occurrence of line containing test (the matched line is also deleted)
* `sed '/test/,$d' dir_list.txt` delete all lines from line containing test to end of file

**Examples for selective printing(p)**

* `sed -n '5p' story.txt` print 5th line, `-n` option overrides default print behavior of sed
	* use `sed '5q;d' story.txt` on large files. [Read more](https://stackoverflow.com/questions/191364/quick-unix-command-to-display-specific-lines-in-the-middle-of-a-file/17367226#17367226)
* `sed -n '/cat/p' story.txt` print every line containing the text cat
	* equivalent to `sed '/cat/!d' story.txt`
* `sed -n '4,8!p' story.txt` print all lines except lines 4 to 8
* `man grep | sed -n '/^\s*exit status/I,/^$/p'` extract exit status information of a command from manual
    * `/^\s*exit status/I` checks for line starting with 'exit status' in case insensitive way, white-space may be present at start of line
    * `/^$/` empty line
* `man ls | sed -n '/^\s*-F/,/^$/p'` extract information on command option from manual
    * `/^\s*-F/` line starting with option '-F', white-space may be present at start of line

**Examples for search and replace(s)**

* `sed -i 's/cat/dog/g' story.txt` search and replace every occurrence of cat with dog in story.txt
* `sed -i.bkp 's/cat/dog/g' story.txt` in addition to inplace file editing, create backup file story.txt.bkp, so that if a mistake happens, original file can be restored
    * `sed -i.bkp 's/cat/dog/g' *.txt` to perform operation on all files ending with .txt in current directory
* `sed -i '5,10s/cat/dog/gI' story.txt` search and replace every occurrence of cat (case insensitive due to modifier I) with dog in story.txt only in line numbers 5 to 10
* `sed '/cat/ s/animal/mammal/g' story.txt` replace animal with mammal in all lines containing cat
    * Since `-i` option is not used, output is displayed on standard output and story.txt is not changed
    * spacing between range and command is optional, `sed '/cat/s/animal/mammal/g' story.txt` can also be used
* `sed -i -e 's/cat/dog/g' -e 's/lion/tiger/g' story.txt` search and replace every occurrence of cat with dog and lion with tiger
    * any number of `-e` option can be used
	* `sed -i 's/cat/dog/g ; s/lion/tiger/g' story.txt` alternative syntax, spacing around ; is optional
* `sed -r 's/(.*)/abc: \1 :xyz/' list.txt` add prefix 'abc: ' and suffix ' :xyz' to every line of list.txt
* `sed -i -r "s/(.*)/$(basename $PWD)\/\1/" dir_list.txt` add current directory name and forward-slash character at the start of every line
    * Note the use of double quotes to perform command substitution
* `sed -i -r "s|.*|$HOME/\0|" dir_list.txt` add home directory and forward-slash at the start of every line
    * Since the value of '$HOME' itself contains forward-slash characters, we cannot use `/` as delimiter
    * Any character other than backslash or newline can be used as delimiter, for example `| # ^` [see this link for more info](https://stackoverflow.com/questions/33914360/what-delimiters-can-you-use-in-sed)
    * `\0` back-reference contains entire matched string

<br>

**Example input file**

```bash
$ cat mem_test.txt 
mreg2 = 1200 # starting address
mreg4 = 2180 # ending address

dreg5 = get(mreg2) + get(mreg4)
print dreg5
```

* replace all **reg** with **register**

```bash
$ sed 's/reg/register/g' mem_test.txt 
mregister2 = 1200 # starting address
mregister4 = 2180 # ending address

dregister5 = get(mregister2) + get(mregister4)
print dregister5
```

* change start and end address

```bash
$ sed 's/1200/1530/; s/2180/1870/' mem_test.txt 
mreg2 = 1530 # starting address
mreg4 = 1870 # ending address

dreg5 = get(mreg2) + get(mreg4)
print dreg5

$ # to make changes only on mreg initializations, use
$ # sed '/mreg[0-9] *=/ s/1200/1530/; s/2180/1870/' mem_test.txt
```

* Using `bash` variables

```bash
$ s_add='1760'; e_add='2500'
$ sed "s/1200/$s_add/; s/2180/$e_add/" mem_test.txt 
mreg2 = 1760 # starting address
mreg4 = 2500 # ending address

dreg5 = get(mreg2) + get(mreg4)
print dreg5
```

* split inline commented code to comment + code

```bash
$ sed -E 's/^([^#]+)(#.*)/\2\n\1/' mem_test.txt 
# starting address
mreg2 = 1200 
# ending address
mreg4 = 2180 

dreg5 = get(mreg2) + get(mreg4)
print dreg5
```

* range of lines matching pattern

```bash
$ seq 20 | sed -n '/3/,/5/p'
3
4
5
13
14
15

```

* inplace editing

```bash
$ sed -i -E 's/([md]r)eg/\1/g' mem_test.txt
$ cat mem_test.txt
mr2 = 1200 # starting address
mr4 = 2180 # ending address

dr5 = get(mr2) + get(mr4)
print dr5

$ # more than one input files can be given
$ # use glob pattern if files share commonality, ex: *.txt
```

**Further Reading**

* [sed basics](http://code.snipcademy.com/tutorials/shell-scripting/sed/introduction)
* [sed detailed tutorial](http://www.grymoire.com/Unix/Sed.html)
* [sed-book](http://www.catonmat.net/blog/sed-book/)
* [cheat sheet](http://www.catonmat.net/download/sed.stream.editor.cheat.sheet.txt)
* [sed examples](http://how-to.linuxcareer.com/learning-linux-commands-sed)
* [sed one-liners explained](http://www.catonmat.net/series/sed-one-liners-explained)
* [common search and replace examples with sed](https://unix.stackexchange.com/questions/112023/how-can-i-replace-a-string-in-a-files)
* [sed Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/sed?sort=votes&pageSize=15)
* [sed Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/sed?sort=votes&pageSize=15)


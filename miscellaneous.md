# <a name="miscellaneous"></a>Miscellaneous

**Table of Contents**

* [cut](#cut)
* [tr](#tr)
* [basename](#basename)
* [dirname](#dirname)

<br>

## <a name="cut"></a>cut

>remove sections from each line of files

For columns operations with well defined delimiters, `cut` command is handy

**Examples**

* `ls -l | cut -d' ' -f1` first column of `ls -l`
    * `-d` option specifies delimiter character, in this case it is single space character (Default delimiter is TAB character)
    * `-f` option specifies which fields to print separated by commas, in this case field 1
* `cut -d':' -f1 /etc/passwd` prints first column of /etc/passwd file
* `cut -d':' -f1,7 /etc/passwd` prints 1st and 7th column of /etc/passwd file with : character in between
* `cut -d':' --output-delimiter=' ' -f1,7 /etc/passwd` use space as delimiter between 1st and 7th column while printing
* [cut Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/cut?sort=votes&pageSize=15)

<br>

## <a name="tr"></a>tr

>translate or delete characters

**Options**

* `-d` delete the specified characters
* `-c` complement set of characters to be replaced

**Examples**

* `tr a-z A-Z < test_list.txt` convert lowercase to uppercase
* `tr -d ._ < test_list.txt` delete the dot and underscore characters
* `tr a-z n-za-m < test_list.txt > encrypted_test_list.txt` Encrypt by replacing every lowercase alphabet with 13th alphabet after it
    * Same command on encrypted text will decrypt it
* [tr Q&A on unix stackexchange](http://unix.stackexchange.com/questions/tagged/tr?sort=votes&pageSize=15)

<br>

## <a name="basename"></a>basename

>strip directory and suffix from filenames

```bash
$ pwd
/home/learnbyexample

$ basename $PWD
learnbyexample

$ basename '/home/learnbyexample/proj_adder/power.log'
power.log

$ #use suffix option -s to strip file extension from filename
$ basename -s '.log' '/home/learnbyexample/proj_adder/power.log'
power
```

<br>

## <a name="dirname"></a>dirname

>strip last component from file name

```bash
$ pwd
/home/learnbyexample

$ dirname $PWD
/home

$ dirname '/home/learnbyexample/proj_adder/power.log'
/home/learnbyexample/proj_adder
```


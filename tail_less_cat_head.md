# <a name="cat-less-tail-and-head"></a>Cat, Less, Tail and Head

**Table of Contents**

* [cat](#cat)
* [less](#less)
* [tail](#tail)
* [head](#head)
* [Text Editors](#text-editors)

<br>

## <a name="cat"></a>cat

>concatenate files and print on the standard output

**Options**

* `-n` number output lines
* `-s` squeeze repeated empty lines into single empty line
* `-e` show non-printing characters and end of line
* `-A` in addition to `-e`, also shows tab characters

**Examples**

* `cat > sample.txt` create a new file for writing, use `Ctrl+d` on a newline to save the file and quit
    * [difference between Ctrl+c and Ctrl+d to signal end of stdin input in bash](https://unix.stackexchange.com/questions/16333/how-to-signal-the-end-of-stdin-input-in-bash)
* `cat sample.txt` display the contents of file sample.txt
* `cat power.log timing.log area.log > report.log` concatenate the contents of all three files and save to report.log
* `tac sample.txt` display the lines of file sample.txt in reversed order
* [cat Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/cat?sort=votes&pageSize=15)
* [cat Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/cat?sort=votes&pageSize=15)

```bash
$ cat > sample.txt
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.

$ cat sample.txt 
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.

$ wc sample.txt 
  2  23 111 sample.txt

$ tac sample.txt 
Press Ctrl+d on a newline to save and quit.
This is an example of adding text to a new file using cat command.
```

<br>

## <a name="less"></a>less

>opposite of more

`cat` command is not suitable for viewing contents of large files on the Terminal. `less` displays contents of a file, automatically fits to size of Terminal, allows scrolling in either direction and other options for effective viewing. Usually, `man` command uses `less` command to display the help page. The navigation options are similar to `vi` editor

**Navigation options**

* `g` go to start of file
* `G` go to end of file
* `q` quit
* `/pattern` search for the given pattern in forward direction
* `?pattern` search for the given pattern in backward direction
* `n` go to next pattern
* `N` go to previous pattern
* `h` help

**Example and Further Reading**

* `less -s large_filename` display contents of file large_filename using less command, consecutive blank lines are squeezed as single blank line
* `less` command is an [improved version](https://unix.stackexchange.com/questions/604/isnt-less-just-more) of `more` command
* [differences between most, more and less](https://unix.stackexchange.com/questions/81129/what-are-the-differences-between-most-more-and-less)
* [less Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/less?sort=votes&pageSize=15)

<br>

## <a name="tail"></a>tail

>output the last part of files

**Examples**

* `tail report.log` by default display last 10 lines
* `tail -20 report.log` display last 20 lines
* `tail -5 report.log` display last 5 lines
* `tail power.log timing.log` display last 10 lines of both files preceded by filename header
* `tail -q power.log timing.log > result.log` save last 10 lines of both files without filename header to result.log
* `tail -n +3 report.log` display all lines starting from third line (i.e all lines except first two lines)
* [tail Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/tail?sort=votes&pageSize=15)
* [tail Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/tail?sort=votes&pageSize=15)

<br>

## <a name="head"></a>head

>output the first part of files

**Examples**

* `head report.log` display first 10 lines
* `head -20 report.log | tail report.log` display lines 11 to 20
* `tail -30 report.log | head -5 report.log` display 5 lines prior to last 25 lines of file
* `head -n -2 report.log` display all but last 2 lines
* [head Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/head?sort=votes&pageSize=15)

<br>

## <a name="text-editors"></a>Text Editors

For editing text files, the following applications can be used. Of these, `gedit`, `nano`, `vi` and/or `vim` are available in most distros by default

Easy to use

* [gedit](https://wiki.gnome.org/Apps/Gedit)
* [geany](http://www.geany.org/)
* [nano](http://nano-editor.org/)

Powerful text editors

* [vim](https://github.com/vim/vim)
    * [vim learning resources](https://github.com/learnbyexample/scripting_course/blob/master/Vim_curated_resources.md) and [vim reference](https://github.com/learnbyexample/vim_reference) for further info
* [emacs](https://www.gnu.org/software/emacs/)
* [atom](https://atom.io/)
* [sublime](https://www.sublimetext.com/)


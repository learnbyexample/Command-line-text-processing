# <a name="file-attributes"></a>File attributes

**Table of Contents**

* [wc](#wc)
* [du](#du)
* [df](#df)
* [touch](#touch)
* [file](#file)
* [identify](#identify)
* [chmod](#chmod)

<br>

## <a name="wc"></a>wc

>print newline, word, and byte counts for each file

**Examples**

* `wc power.log` outputs no. of lines, words and characters separated by white-space and followed by filename
* `wc -l power.log` outputs no. of lines followed by filename
* `wc -w power.log` outputs no. of words followed by filename
* `wc -c power.log` outputs no. of characters followed by filename
* `wc -l < power.log` output only the number of lines
* `wc -L power.log` length of longest line followed by filename
* [wc Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/wc?sort=votes&pageSize=15)
* [wc Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/wc?sort=votes&pageSize=15)

<br>

## <a name="du"></a>du

>estimate file space usage

* du command is useful to get size of files and folders, not for file systems

**Examples**

* `du project_report` display size (default unit is 1024 bytes) of folder project_report as well as it sub-directories
* `du --si project_report` display size (unit is 1000 bytes) of folder project_report as well as it sub-directories
* `du -h project_report` display size in human readable format for folder project_report as well as it sub-directories
* `du -ah project_report` display size in human readable format for folder project_report and all of its files and sub-directories
* `du -sh project_report` display size only for project_report folder in human readable format
* `du -sm * | sort -n` sort files and folders of current directory, numbers displayed are in Megabytes
    * use `sort -nr` to reverse sort order, i.e largest at top
* `du -sh * | sort -h` sort files and folders of current directory, output displayed in human-readable format
* [du Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/disk-usage?sort=votes&pageSize=15)
* [du Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/du?sort=votes&pageSize=15)

<br>

## <a name="df"></a>df

>report file system disk space usage

**Examples**

* `df -h` display usage statistics of all available file systems
* `df -h .` display usage of only the file system where the current working directory is located
* [df Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/df?sort=votes&pageSize=15)

<br>

## <a name="touch"></a>touch

>change file timestamps

Used to change file time stamps. But if file doesn't exist, the command will create an empty file with the name provided. Both features are quite useful  

* Some program may require a particular file to be present to work, empty file might even be a valid argument. In such cases, a pre-processing script can scan the destination directories and create empty file if needed
* Similarly, some programs may behave differently according to the time stamps of two or more files - while debugging in such an environment, user might want to just change the time stamp of files

**Examples**

* `touch new_file.txt` create an empty file if it doesn't exist in current directory
    * use `-c` if new file shouldn't be created
    * use `-a` option to change only access time and `-m` to change only modification time
* `touch report.log` change the time stamp of report.log to current time (assuming report.log already exists in current directory)
* `touch -r power.log report.log` use time stamp of power.log instead of current time to change that of report.log
    * use `-d` to provide time stamp from a string instead of file
* [touch Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/touch?sort=votes&pageSize=15)

<br>

## <a name="file"></a>file

>determine file type

**Examples**

```bash
$ file sample.txt 
sample.txt: ASCII text

$ file sunset.jpg 
sunset.jpg: JPEG image data

$ mv sunset.jpg xyz
$ file xyz 
xyz: JPEG image data

$ file perl.png 
perl.png: PNG image data, 32 x 32, 8-bit/color RGBA, non-interlaced
```

<br>

## <a name="identify"></a>identify

>describes the format and characteristics of one or more image files

Although file command can also give information like pixel dimensions and image type, identify is more reliable command for images and gives complete format information

```bash
$ identify sunset.jpg
sunset.jpg JPEG 740x740 740x740+0+0 8-bit DirectClass 110KB 0.000u 0:00.030

$ identify perl.png 
perl.png PNG 32x32 32x32+0+0 8-bit DirectClass 838B 0.000u 0:00.000
```

<br>

## <a name="chmod"></a>chmod

>change file mode bits

```bash
$ ls -l sample.txt 
-rw-rw-r-- 1 learnbyexample learnbyexample 111 May 25 14:47 sample.txt
```

In the output of `ls -l` command, the first 10 characters displayed are related to type of file and its permissions.

First character indicates the **file type** The most common are

* `-` regular file
* `d` directory
* `l` symbolic link
* for complete list, see `-l` option in `info ls`

The other 9 characters represent three sets of **file permissions** for 'user', 'group' and 'others' - in that order

* `user`  file properties for owner of file - `u`
* `group` file properties for the group the file belongs to - `g`
* `others` file properties for everyone else - `o`

We'll be seeing only `rwx` file properties in this section, for other types of properties, [refer this detailed doc](https://www.mkssoftware.com/docs/man1/ls.1.asp#Long_Output_Format)

**Permission characters and values**

| Character | Meaning | Value | File | Directory |
| ------- | ------- | ------- | ------- | ------ |
| r | read | 4 | file can be read | can see contents of directory |
| w | write | 2 | file can be modified | can add/remove files in directory |
| x | execute | 1 | file can be run as a program | can access contents of directory |
| - | no permission | 0 | permission is disabled | permission is disabled |

**File Permissions**

```bash
$ pwd
/home/learnbyexample/linux_tutorial
$ ls -lF
total 8
-rw-rw-r-- 1 learnbyexample learnbyexample  40 May 28 13:25 hello_world.pl
-rw-rw-r-- 1 learnbyexample learnbyexample 111 May 25 14:47 sample.txt

$ #Files need execute permission to be run as program
$ ./hello_world.pl
bash: ./hello_world.pl: Permission denied
$ chmod +x hello_world.pl 
$ ls -lF hello_world.pl
-rwxrwxr-x 1 learnbyexample learnbyexample  40 May 28 13:25 hello_world.pl*
$ ./hello_world.pl
Hello World

$ #Read permission
$ cat sample.txt 
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.
$ chmod -r sample.txt
$ ls -lF sample.txt 
--w--w---- 1 learnbyexample learnbyexample 111 May 25 14:47 sample.txt
$ cat sample.txt 
cat: sample.txt: Permission denied

$ #Files need write permission to modify its content
$ cat >> sample.txt 
Adding a line of text at end of file
^C
$ cat sample.txt 
cat: sample.txt: Permission denied
$ chmod +r sample.txt 
$ ls -lF sample.txt 
-rw-rw-r-- 1 learnbyexample learnbyexample 148 May 29 11:00 sample.txt
$ cat sample.txt 
This is an example of adding text to a new file using cat command.
Press Ctrl+d on a newline to save and quit.
Adding a line of text at end of file
$ chmod -w sample.txt 
$ ls -lF sample.txt 
-r--r--r-- 1 learnbyexample learnbyexample 148 May 29 11:00 sample.txt
$ cat >> sample.txt 
bash: sample.txt: Permission denied
```

**Directory Permissions**

```bash
$ ls -ld linux_tutorial/
drwxrwxr-x 2 learnbyexample learnbyexample 4096 May 29 10:59 linux_tutorial/

$ #Read Permission
$ ls linux_tutorial/
hello_world.pl  sample.txt
$ chmod -r linux_tutorial/
$ ls -ld linux_tutorial/
d-wx-wx--x 2 learnbyexample learnbyexample 4096 May 29 10:59 linux_tutorial/
$ ls linux_tutorial/
ls: cannot open directory linux_tutorial/: Permission denied
$ chmod +r linux_tutorial/

$ #Execute Permission
$ chmod -x linux_tutorial/
$ ls -ld linux_tutorial/
drw-rw-r-- 2 learnbyexample learnbyexample 4096 May 29 10:59 linux_tutorial/
$ ls linux_tutorial/
ls: cannot access linux_tutorial/hello_world.pl: Permission denied
ls: cannot access linux_tutorial/sample.txt: Permission denied
hello_world.pl  sample.txt
$ chmod +x linux_tutorial/

$ #Write Permission
$ chmod -w linux_tutorial/
$ ls -ld linux_tutorial/
dr-xr-xr-x 2 learnbyexample learnbyexample 4096 May 29 10:59 linux_tutorial/
$ touch linux_tutorial/new_file.txt
touch: cannot touch ‘linux_tutorial/new_file.txt’: Permission denied
$ chmod +w linux_tutorial/
$ ls -ld linux_tutorial/
drwxrwxr-x 2 learnbyexample learnbyexample 4096 May 29 10:59 linux_tutorial/
$ touch linux_tutorial/new_file.txt
$ ls linux_tutorial/
hello_world.pl  new_file.txt  sample.txt
```

**Changing multiple permissions at once**

```bash
$ # r(4) + w(2) + 0 = 6
$ # r(4) + 0 + 0 = 4
$ chmod 664 sample.txt 
$ ls -lF sample.txt 
-rw-rw-r-- 1 learnbyexample learnbyexample 148 May 29 11:00 sample.txt

$ # r(4) + w(2) + x(1) = 7
$ # r(4) + 0 + x(1) = 5
$ chmod 755 hello_world.pl 
$ ls -lF hello_world.pl 
-rwxr-xr-x 1 learnbyexample learnbyexample 40 May 28 13:25 hello_world.pl*

$ chmod 775 report/
$ ls -ld report/
drwxrwxr-x 2 learnbyexample learnbyexample 4096 May 29 14:01 report/
```

**Changing single permission selectively**

```bash
$ chmod o-r sample.txt 
$ ls -lF sample.txt 
-rw-rw---- 1 learnbyexample learnbyexample 148 May 29 11:00 sample.txt

$ chmod go-x hello_world.pl 
$ ls -lF hello_world.pl 
-rwxr--r-- 1 learnbyexample learnbyexample 40 May 28 13:25 hello_world.pl*

$ chmod go+x hello_world.pl 
$ ls -lF hello_world.pl 
-rwxr-xr-x 1 learnbyexample learnbyexample 40 May 28 13:25 hello_world.pl*
```

**Recursively changing permission for directory**

```bash
$ ls -lR linux_tutorial/
linux_tutorial/:
total 12
-rwxr-xr-x 1 learnbyexample learnbyexample   40 May 28 13:25 hello_world.pl
drwxrwxr-x 2 learnbyexample learnbyexample 4096 May 29 14:32 report
-rw-rw---- 1 learnbyexample learnbyexample  148 May 29 11:00 sample.txt

linux_tutorial/report:
total 0
-rw-rw-r-- 1 learnbyexample learnbyexample 0 May 29 11:46 new_file.txt
$ ls -ld linux_tutorial/
drwxrwxr-x 3 learnbyexample learnbyexample 4096 May 29 14:32 linux_tutorial/

$ #adding/removing files to a directory depends only on parent directory permissions
$ chmod -w linux_tutorial/
$ ls -ld linux_tutorial/
dr-xr-xr-x 3 learnbyexample learnbyexample 4096 May 29 14:32 linux_tutorial/
$ ls -ld linux_tutorial/report/
drwxrwxr-x 2 learnbyexample learnbyexample 4096 May 29 14:32 linux_tutorial/report/
$ rm linux_tutorial/sample.txt 
rm: cannot remove ‘linux_tutorial/sample.txt’: Permission denied
$ touch linux_tutorial/report/power.log
$ ls linux_tutorial/report/
new_file.txt  power.log
$ rm linux_tutorial/report/new_file.txt
$ ls linux_tutorial/report/
power.log

$ chmod +w linux_tutorial/
$ ls -ld linux_tutorial/
drwxrwxr-x 3 learnbyexample learnbyexample 4096 May 29 14:32 linux_tutorial/
$ chmod -w -R linux_tutorial/
$ ls -lR linux_tutorial/
linux_tutorial/:
total 12
-r-xr-xr-x 1 learnbyexample learnbyexample   40 May 28 13:25 hello_world.pl
dr-xr-xr-x 2 learnbyexample learnbyexample 4096 May 29 14:40 report
-r--r----- 1 learnbyexample learnbyexample  148 May 29 11:00 sample.txt

linux_tutorial/report:
total 0
-r--r--r-- 1 learnbyexample learnbyexample 0 May 29 14:39 power.log
$ rm linux_tutorial/report/power.log 
rm: remove write-protected regular empty file ‘linux_tutorial/report/power.log’? y
rm: cannot remove ‘linux_tutorial/report/power.log’: Permission denied
```

* `+r -r +x -x` without `u g o` qualifier affects all the three categories
* `+w -w` without `u g o` qualifier affects only user and group categories

**Further Reading**

* [Linux File Permissions](https://www.linux.com/learn/getting-know-linux-file-permissions)
* [Linux Permissions Primer](https://danielmiessler.com/study/unixlinux_permissions/)
* [chmod Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/chmod?sort=votes&pageSize=15)
* [chmod Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/chmod?sort=votes&pageSize=15)

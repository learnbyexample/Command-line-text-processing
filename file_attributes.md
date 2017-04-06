# <a name="file-attributes"></a>File attributes

**Table of Contents**

* [wc](#wc)
* [du](#du)
* [df](#df)
* [touch](#touch)
* [file](#file)
* [identify](#identify)

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

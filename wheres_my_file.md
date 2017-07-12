# <a name="where's-my-file"></a>Where's my file

**Table of Contents**

* [find](#find)
* [locate](#locate)

<br>

## <a name="find"></a>find

```bash
$ find --version | head -n1
find (GNU findutils) 4.7.0-git

$ man find
FIND(1)                     General Commands Manual                    FIND(1)

NAME
       find - search for files in a directory hierarchy

SYNOPSIS
       find  [-H]  [-L]  [-P]  [-D  debugopts]  [-Olevel]  [starting-point...]
       [expression]

DESCRIPTION
       This manual page documents the GNU version of find.  GNU find  searches
       the  directory  tree  rooted at each given starting-point by evaluating
       the given expression from left to right,  according  to  the  rules  of
       precedence  (see  section  OPERATORS),  until the outcome is known (the
       left hand side is false for and operations,  true  for  or),  at  which
       point  find  moves  on  to the next file name.  If no starting-point is
       specified, `.' is assumed.
...
```

**Examples**

Filtering based on file name

* `find . -iname 'power.log'` search and print path of file named power.log (ignoring case) in current directory and its sub-directories
* `find -name '*log'` search and print path of all files whose name ends with log in current directory - using `.` is optional when searching in current directory
* `find -not -name '*log'` print path of all files whose name does NOT end with log in current directory
* `find -regextype egrep -regex '.*/\w+'` use extended regular expression to match filename containing only `[a-zA-Z_]` characters
    * `.*/` is needed to match initial part of file path

Filtering based on file type

* `find /home/guest1/proj -type f` print path of all regular files found in specified directory
* `find /home/guest1/proj -type d` print path of all directories found in specified directory
* `find /home/guest1/proj -type f -name '.*'` print path of all hidden files

Filtering based on depth

The relative path `.` is considered as depth 0 directory, files and folders immediately contained in a directory are at depth 1 and so on

* `find -maxdepth 1 -type f` all regular files (including hidden ones) from current directory (without going to sub-directories)
* `find -maxdepth 1 -type f -name '[!.]*'` all regular files (but not hidden ones) from current directory (without going to sub-directories)
    * `-not -name '.*'` can be also used
* `find -mindepth 1 -maxdepth 1 -type d` all directories (including hidden ones) in current directory (without going to sub-directories)

Filtering based on file properties

* `find -mtime -2` print files that were modified within last two days in current directory
    * Note that day here means 24 hours
* `find -mtime +7` print files that were modified more than seven days back in current directory
* `find -daystart -type f -mtime -1` files that were modified from beginning of day (not past 24 hours)
* `find -size +10k` print files with size greater than 10 kilobytes in current directory
* `find -size -1M` print files with size less than 1 megabytes in current directory
* `find -size 2G` print files of size 2 gigabytes in current directory

Passing filtered files as input to other commands

* `find report -name '*log*' -exec rm {} \;` delete all filenames containing log in report folder and its sub-folders
    * here `rm` command is called for every file matching the search conditions
    * since `;` is a special character for shell, it needs to be escaped using `\`
* `find report -name '*log*' -delete` delete all filenames containing log in report folder and its sub-folders
* `find -name '*.txt' -exec wc {} +` list of files ending with txt are all passed together as argument to `wc` command instead of executing wc command for every file
    * no need to use escape the `+` character in this case
    * also note that number of invocations of command specified is not necessarily once if number of files found is too large
* `find -name '*.log' -exec mv {} ../log/ \;` move files ending with .log to log directory present in one hierarchy above. `mv` is executed once per each filtered file
* `find -name '*.log' -exec mv -t ../log/ {} +` the `-t` option allows to specify target directory and then provide multiple files to be moved as argument
    * Similarly, one can use `-t` for `cp` command

**Further Reading**

* [using find](http://mywiki.wooledge.org/UsingFind)
* [find examples on SO](https://stackoverflow.com/documentation/bash/566/find#t=201612140534548263961)
* [Collection of find examples](http://alvinalexander.com/unix/edu/examples/find.shtml)
* [find Q&A on unix stackexchange](https://unix.stackexchange.com/questions/tagged/find?sort=votes&pageSize=15)
* [find and tar example](https://unix.stackexchange.com/questions/282762/find-mtime-1-print-xargs-tar-archives-all-files-from-directory-ignoring-t/282885#282885)
* [find Q&A on stackoverflow](https://stackoverflow.com/questions/tagged/find?sort=votes&pageSize=15)
* [Why is looping over find's output bad practice?](https://unix.stackexchange.com/questions/321697/why-is-looping-over-finds-output-bad-practice)


<br>

## <a name="locate"></a>locate

```bash
$ locate --version | head -n1
mlocate 0.26

$ man locate
locate(1)                   General Commands Manual                  locate(1)

NAME
       locate - find files by name

SYNOPSIS
       locate [OPTION]... PATTERN...

DESCRIPTION
       locate  reads  one or more databases prepared by updatedb(8) and writes
       file names matching at least one of the PATTERNs  to  standard  output,
       one per line.

       If  --regex is not specified, PATTERNs can contain globbing characters.
       If any PATTERN contains no globbing characters, locate  behaves  as  if
       the pattern were *PATTERN*.
...
```

Faster alternative to `find` command when searching for a file by its name. It is based on a database, which gets updated by a `cron` job. So, newer files may be not present in results. Use this command if it is available in your distro and you remember some part of filename. Very useful if one has to search entire filesystem in which case `find` command might take a very long time compared to `locate`

**Examples**

* `locate 'power'` print path of files containing power in the whole filesystem
    * matches anywhere in path, ex: '/home/learnbyexample/lowpower_adder/result.log' and '/home/learnbyexample/power.log' are both a valid match
    * implicitly, `locate` would change the string to `*power*` as no globbing characters are present in the string specified
* `locate -b '\power.log'` print path matching the string power.log exactly at end of path
    * '/home/learnbyexample/power.log' matches but not '/home/learnbyexample/lowpower.log'
    * since globbing character '\' is used while specifying search string, it doesn't get implicitly replaced by `*power.log*`
* `locate -b '\proj_adder'` the `-b` option also comes in handy to print only the path of directory name, otherwise every file under that folder would also be displayed
* [find vs locate - pros and cons](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)



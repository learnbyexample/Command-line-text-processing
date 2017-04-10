Note: Every file in this directory and sub-directories is input for grep, unless otherwise specified

1) Match all lines containing the string: you
Solution: grep -r 'you'

2) Show only filenames matching the string: Hello
    filenames should only end with .txt 
Solution: grep -rl --include='*.txt' 'Hello'

3) Show only filenames matching the string: Hello
    filenames should NOT end with .txt 
Solution: grep -rl --exclude='*.txt' 'Hello'

4) Show only filenames matching the string: are
    should not include the directory: progs
Solution: grep -rl --exclude-dir='progs' 'are'

5) Show only filenames matching the string: are
    should NOT include these directories
            dir1: progs
            dir2: msg
Solution: grep -rl --exclude-dir='progs' --exclude-dir='msg' 'are'

6) Show only filenames matching the string: are
    should include files only from sub-directories
    hint: use shell glob pattern to specify directories to search
Solution: grep -rl 'are' */

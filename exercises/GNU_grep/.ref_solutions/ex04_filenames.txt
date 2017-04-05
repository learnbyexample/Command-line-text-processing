Note: All files present in the directory should be given as file inputs to grep

1) Show only filenames containing the string: are
Solution: grep -l 'are' *

2) Show only filenames NOT containing the string: two
Solution: grep -L 'two' *

3) Match all lines containing the string: are
Solution: grep 'are' *

4) Match maximum of two matching lines along with filenames containing the character: a
Solution: grep -m2 'a' *

5) Match all lines without prefixing filename containing the string: to
Solution: grep -h 'to' *

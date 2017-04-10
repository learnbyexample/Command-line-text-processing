1) Extract all 3 character strings surrounded by word boundaries
Solution: grep -ow '...' garbled.txt

2) Extract largest string from each line
        starting with character: d
        ending with character  : g
Solution: grep -o 'd.*g' garbled.txt

3) Extract all strings from each line
        starting with character: d
        followed by zero or one: o
        ending with character  : g
Solution: grep -oE 'do?g' garbled.txt

4) Extract all strings from each line
        starting with character: d
        followed by zero or one of any character
        ending with character  : g
Solution: grep -oE 'd.?g' garbled.txt

5) Extract all strings from each line
        starting with character: g
        followed by atleast one: o
        ending with character  : d
Solution: grep -oE 'go+d' garbled.txt

6) Extract all strings from each line
        starting with character : g
        followed by extactly six: o
        ending with character   : d
Solution: grep -oE 'go{6}d' garbled.txt

7) Extract all strings from each line
        starting with character         : g
        followed by min two and max four: o
        ending with character           : d
Solution: grep -oE 'go{2,4}d' garbled.txt

8) Extract all strings from each line
        starting with character: d
        followed by max of two : o
        ending with character  : g
Solution: grep -oE 'do{,2}g' garbled.txt

9) Extract all strings from each line
        starting with character : g
        followed by min of three: o
        ending with character   : d
Solution: grep -oE 'go{3,}d' garbled.txt

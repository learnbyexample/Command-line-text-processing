1) Extract all characters before first occurrence of =
Solution: grep -o '^[^=]*' sample.txt

2) Extract all characters from start of line made up of these characters
        upper or lower case alphabets
        all digits
        the underscore character
Solution: grep -o '^\w*' sample.txt

3) Match all lines containing the sequence
        String1: there
        any number of whitespace
        String2: have
Solution: grep 'there\s*have' sample.txt

4) Extract all characters from start of line made up of these characters
        upper or lower case alphabets
        all digits
        the characters [ and ]
        ending with ]
Solution: grep -oi '^[]a-z0-9[]*]' sample.txt

5) Extract all punctuation characters from first line
Solution: grep -om1 '[[:punct:]]' sample.txt

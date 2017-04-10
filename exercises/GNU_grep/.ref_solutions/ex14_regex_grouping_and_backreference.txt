1) Match lines containing these strings
        String1: scare
        String2: spore
Solution: grep -E 's(po|ca)re' sample.txt

2) Extract these words
        Word1: handy
        Word2: hand
        Word3: hands
        Word4: handful
Solution: grep -oE 'hand([sy]|ful)?' sample.txt

3) Extract all whole words with at least one letter occurring twice in the word
    ignore case
    only alphabets
    the letter occurring twice need not be placed next to each other
Solution: grep -ioE '[a-z]*([a-z])[a-z]*\1[a-z]*' sample.txt

4) Match lines where same sequence of three consecutive alphabets is matched another time in the same line
    ignore case
Solution: grep -iE '([a-z]{3}).*\1' sample.txt

1) Match lines containing either of these three strings
        String1: Not
        String2: he
        String3: sun
Solution: grep -e 'Not' -e 'he' -e 'sun' sample.txt

2) Match lines containing both these strings
        String1: He
        String2: or
Solution: grep 'He' sample.txt | grep 'or'

3) Match lines containing either of these two strings
        String1: a
        String2: i
   and contains this as well
        String3: do
Solution: grep -e 'a' -e 'i' sample.txt | grep 'do'

4) Match lines containing the string
        String1: it
   but not these strings
        String2: No
        String3: no
Solution: grep 'it' sample.txt | grep -vi 'no'

1) Match all lines containing any of these strings:
        String1: day
        String2: not
Solution: grep -E 'day|not' sample.txt

2) Match all lines containing any of these whole words:
        String1: he
        String2: in
Solution: grep -wE 'he|in' sample.txt

3) Match all lines containing any of these strings:
        String1: you
        String2: be
        String3: to
        String4: he
Solution: grep -E 'he|be|to|you' sample.txt

4) Match all lines containing any of these strings:
        String1: you
        String2: be
        String3: to
        String4: he
    but NOT these strings:
        String1: it
        String2: do
Solution: grep -E 'he|be|to|you' sample.txt | grep -vE 'do|it'

5) Match all lines starting with any of these strings:
        String1: no
        String2: to
Solution: grep -E '^no|^to' sample.txt

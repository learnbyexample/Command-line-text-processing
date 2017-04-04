1) Match lines containing the string irrespective of lower/upper case: no
Solution: grep -i 'no' sample.txt

2) Match lines not containing the string: o
Solution: grep -v 'o' sample.txt

3) Match lines with line numbers containing the string: it
Solution: grep -n 'it' sample.txt

4) Output only number of matching lines containing the string: a
Solution: grep -c 'a' sample.txt

5) Match first two lines containing the string: do
Solution: grep -m2 'do' sample.txt

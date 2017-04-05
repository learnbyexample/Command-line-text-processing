1) Get lines and 3 following it containing the string: you
Solution: grep -A3 'you' sample.txt

2) Get lines and 2 preceding it containing the string: is
Solution: grep -B2 'is' sample.txt

3) Get lines and 1 following/preceding containing the string: Not
Solution: grep -C1 'Not' sample.txt

4) Get lines and 1 following and 4 preceding containing the string: Not
Solution: grep -A1 -B4 'Not' sample.txt

5) Get lines and 1 preceding it containing the string: you
        there should be no separator between the matches
Solution: grep --no-group-separator -B1 'you' sample.txt

6) Get lines and 1 preceding it containing the string: you
        the separator between the matches should be: #####
Solution: grep --group-separator='#####' -B1 'you' sample.txt

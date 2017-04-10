1) Match all lines starting with: no
Solution: grep '^no' sample.txt

2) Match all lines ending with: it
Solution: grep 'it$' sample.txt

3) Match all lines containing whole word: do
Solution: grep -w 'do' sample.txt

4) Match all lines containing words starting with: do
Solution: grep '\<do' sample.txt

5) Match all lines containing words ending with: do
Solution: grep 'do\>' sample.txt

6) Match all lines starting with: ^
Solution: grep '^^' sample.txt

7) Match all lines ending with: $
Solution: grep '$$' sample.txt

8) Match all lines containing the string: in
    not surrounded by word boundaries, for ex: mint but not tin or ink
Solution: grep '\Bin\B' sample.txt

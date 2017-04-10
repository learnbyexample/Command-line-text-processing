Note: All files present in the directory should be given as file inputs to grep

1) Match lines containing whole word: do
Solution: grep -w 'do' *

2) Match whole lines containing the string: Hello World
Solution: grep -x 'Hello World' *

3) Match lines containing these whole words:
        Word1: He
        Word2: far
Solution: grep -w -e 'far' -e 'He' *

4) Match lines containing the whole word: you
    and NOT containing the case insensitive string: How
Solution: grep -w 'you' * | grep -vi 'how'

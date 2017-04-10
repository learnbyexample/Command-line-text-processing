1) Match all lines containing any of these characters:
        character1: q
        character2: x
        character3: z
Solution: grep '[qzx]' sample_words.txt

2) Match all lines containing any of these characters:
        character1: c
        character2: f
    followed by any character
    followed by   : t
Solution: grep '[cf].t' sample_words.txt

3) Extract all words starting with character: s
    ignore case
    should contain only alphabets
    minimum two letters
    should be surrounded by word boundaries
Solution: grep -iowE 's[a-z]+' sample_words.txt

4) Extract all words made up of these characters:
        character1: a
        character2: c
        character3: e
        character4: r
        character5: s
    ignore case
    should contain only alphabets
    should be surrounded by word boundaries
Solution: grep -iowE '[acers]+' sample_words.txt

5) Extract all numbers surrounded by word boundaries
Solution: grep -ow '[0-9]*' sample_words.txt

6) Extract all numbers surrounded by word boundaries matching the condition
    30 <= number <= 70
Solution: grep -owE '[3-6][0-9]|70' sample_words.txt

7) Extract all words made up of non-vowel characters
    ignore case
    should contain only alphabets and at least two
    should be surrounded by word boundaries
Solution: grep -iowE '[b-df-hj-np-tv-z]{2,}' sample_words.txt

8) Extract all sequence of strings consisting of character: -
    surrounded on either side by zero or more case insensitive alphabets    
Solution: grep -io '[a-z]*-[a-z]*' sample_words.txt

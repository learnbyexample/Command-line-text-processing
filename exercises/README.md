# <a name="exercises"></a>Exercises

Instructions and shell script here assumes `bash` shell. Tested on *GNU bash, version 4.3.46*

<br>

* For example, the first exercise for **GNU_grep**
    * directory: `ex01_basic_match`
    * question file: `ex01_basic_match.txt`
    * solution reference: `.ref_solutions/ex01_basic_match.txt`
* Each exercise contains one or more question to be solved
* The script `solve` will assist in checking solutions

```bash
$ git clone https://github.com/learnbyexample/Command-line-text-processing.git
$ cd Command-line-text-processing/exercises/GNU_grep/
$ ls
ex01_basic_match      ex02_basic_options      ex03_multiple_string_match      solve
ex01_basic_match.txt  ex02_basic_options.txt  ex03_multiple_string_match.txt

$ find -name 'ex01*'
./.ref_solutions/ex01_basic_match.txt
./ex01_basic_match
./ex01_basic_match.txt
```

<br>

* Solving the questions
    * Go to the exercise folder
    * Use `ls` to see input file(s)
    * To see the problems for that exercise, follow the steps below

```bash
$ cd ex01_basic_match
$ ls
sample.txt

$ # to see the questions
$ source ../solve -q
1) Match lines containing the string: day


2) Match lines containing the string: it


3) Match lines containing the string: do you


$ # or open the questions file with your fav editor
$ gvim ../$(basename "$PWD").txt
$ # create an alias to use from any ex* directory
$ alias oq='gvim ../$(basename "$PWD").txt'
$ oq
```

<br>

* Submitting solutions one by one
    * immediately after executing command that answers a question, call the `solve` script

```bash
$ grep 'day' sample.txt 
Good day
Today is sunny
$ source ../solve -s
---------------------------------------------
Match for question 1:
Submitted solution: grep 'day' sample.txt 
Reference solution: grep 'day' sample.txt
---------------------------------------------
```

<br>

* Submit all at once
    * by editing the `../$(basename "$PWD").txt` file directly
    * the answer should replace the empty line immediately following the question
* **Note**
    * there are different ways to solve the same question
    * but for specific exercise like **GNU_grep** try to solve using `grep` only
    * also, remember that `eval` is used to check equivalence. So be sure of commands submitted

```bash
$ cat ../$(basename "$PWD").txt
1) Match lines containing the string: day
grep 'day' sample.txt

2) Match lines containing the string: it
sed -n '/it/p' sample.txt

3) Match lines containing the string: do you
echo 'How do you do?'

$ source ../solve
---------------------------------------------
Match for question 1:
Submitted solution: grep 'day' sample.txt
Reference solution: grep 'day' sample.txt
---------------------------------------------
---------------------------------------------
Match for question 2:
Submitted solution: sed -n '/it/p' sample.txt
Reference solution: grep 'it' sample.txt
---------------------------------------------
---------------------------------------------
Match for question 3:
Submitted solution: echo 'How do you do?'
Reference solution: grep 'do you' sample.txt
---------------------------------------------
		All Pass		
```

<br>

* Then move on to next exercise directory
* Create aliases for different commands for easy use, after checking that the aliases are available of course

```bash
$ type cs cq ca nq pq
bash: type: cs: not found
bash: type: cq: not found
bash: type: ca: not found
bash: type: nq: not found
bash: type: pq: not found

$ alias cs='source ../solve -s'
$ alias cq='source ../solve -q'
$ alias ca='source ../solve'
$ # to go to directory of next question
$ nq() { d=$(basename "$PWD"); nd=$(printf "../ex%02d*/" $((${d:2:2}+1))); cd $nd ; }
$ # to go to directory of previous question
$ pq() { d=$(basename "$PWD"); pd=$(printf "../ex%02d*/" $((${d:2:2}-1))); cd $pd ; }
```

<br>

If wrong solution is submitted, the expected output is shown. This also helps to better understand the question as I found it difficult to convey the intent of question clearly with words alone...

```bash
$ source ../solve -q
1) Match lines containing the string: day


2) Match lines containing the string: it


3) Match lines containing the string: do you

$ grep 'do' sample.txt 
How do you do?
Just do it
No doubt you like it too
Much ado about nothing
$ source ../solve -s
---------------------------------------------
Mismatch for question 1:
Expected output is:
Good day
Today is sunny
---------------------------------------------
```

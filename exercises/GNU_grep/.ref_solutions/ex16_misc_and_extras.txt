Note: all files in directory are input to grep, unless otherwise specified

1) Extract all negative numbers
    starts with - followed by one or more digits
    do not output filenames
Solution: grep -hoE -- '-[0-9]+' *

2) Display only filenames containing these two strings anywhere in the file
        String1: day
        String2: and
Solution: grep -zlE 'day.*and|and.*day' *

3) The below command
        grep -c '^Solution:' ../.ref_solutions/*
    will give number of questions in each exercise. Change it, using another command and pipe if needed, so that only overall total is printed
Solution: cat ../.ref_solutions/* | grep -c '^Solution:'


# Basic Python Questions & Solutions

## Table of Contents
1. [Swap Two Numbers](#1-swap-two-numbers-without-temp-variable)
2. [Reverse a String](#2-reverse-a-string)
3. [Even or Odd](#3-even-or-odd)
4. [Print Even Numbers Upto N](#4-print-even-numbers-upto-n)
5. [Palindrome Using Two-Pointers](#5-palindrome-using-two-pointers)
6. [Multiplication Table](#6-multiplication-table-of-n)
7. [Factorial (Recursion)](#7-factorial-recursion)
8. [Fibonacci Series](#8-fibonacci-series)
9. [Second Largest & Smallest](#9-second-largest-smallest-second-smallest)
10. [Remove Duplicates from List](#10-remove-duplicates-from-list)
11. [Rotate List by k](#11-rotate-list-by-k)
12. [Frequency Dictionary](#12-frequency-dictionary)
13. [Merge Two Dictionaries](#13-merge-two-dictionaries-by-adding-values)
14. [Reverse Dictionary](#14-reverse-dictionary-values--list-of-keys)
15. [Check Anagram](#15-check-anagram)
16. [Group Anagrams](#16-group-anagrams)
17. [Prime Number Check](#17-prime-number-check)
18. [List of Primes Upto N](#18-list-of-primes-upto-n)
19. [Word Frequency](#19-word-frequency-case-insensitive)
20. [Sorting Words Alphabetically](#20-sorting-words-alphabetically)
21. [Rotate List (Left & Right)](#21-rotate-list-left--right)
22. [Reverse Integer + Digits Sum](#22-reverse-integer--digits-sum--palindrome)
23. [Merge Two Sorted Arrays](#23-merge-two-sorted-arrays)

---

## 1. Swap Two Numbers (Without Temp Variable)

### ✔ Solution

```python
a = 20
b = 5

a = a + b
b = a - b
a = a - b

print(a, b)
```

**Output:**
```
5 20
```

---

## 2. Reverse a String

### ✔ Solution 1: Using Loop

```python
s = "sanjay"
rs = ""

for i in range(len(s)-1, -1, -1):
    rs += s[i]

print(rs)
```

### ✔ Solution 2: Using String Concatenation

```python
s = "abc"
rs = ""

for e in s:
    rs = e + rs 
    
print(rs)
```

### ➤ Pythonic Way

```python
s = "sanjay"
print(s[::-1])
```

**Output:**
```
yajnas
```

---

## 3. Even or Odd

```python
n = 34
if n % 2 == 0:
    print("even")
else:
    print("odd")
```

**Output:**
```
even
```

---

## 4. Print Even Numbers Upto N

```python
n = 10
for i in range(n+1):
    if i % 2 == 0:
        print(i)
```

**Output:**
```
0
2
4
6
8
10
```

---

## 5. Palindrome Using Two-Pointers

```python
def pal(s):
    l = 0
    r = len(s)-1

    while l < r:
        if s[l] != s[r]:
            return "not palindrome"
        l += 1
        r -= 1

    return "palindrome"

print(pal("sanjay"))
print(pal("madam"))
```

**Output:**
```
not palindrome
palindrome
```

---

## 6. Multiplication Table of n

```python
n = 5
for i in range(1, 11):
    print(n * i)
```

**Output:**
```
5
10
15
20
25
30
35
40
45
50
```

---

## 7. Factorial (Recursion)

```python
def fact(f):
    if f == 0 or f == 1:
        return 1
    return f * fact(f-1)

print(fact(5))
```

**Output:**
```
120
```

---

## 8. Fibonacci Series

```python
a = 0
b = 1
print(a)
print(b)

for i in range(5):
    c = a + b
    a = b
    b = c
    print(c)
```

**Output:**
```
0
1
1
2
3
5
8
```

---

## 9. Second Largest, Smallest, Second Smallest

### ✔ Solution

```python
nums = [10, 4, 6, 2, 9]
maxl = float("-inf")
sl = float("-inf")
minl = float("inf")
sminl = float("inf")

for n in nums:
    if n > maxl:
        sl = maxl
        maxl = n 
    elif n > sl and n != maxl:
        sl = n 
        
    if n < minl:
        sminl = minl
        minl = n 
        
    elif n < sminl and n != minl:
        sminl = n 

print(maxl, sl, minl, sminl)
```

**Output:**
```
10 9 2 4
```

---

## 10. Remove Duplicates from List

```python
l = [1, 2, 1, 1, 4, 3, 4, 5, 6, 7, 7, 5, 8, 9, 9]

def dup(l):
    d = []
    for i in l:
        if i not in d:
            d.append(i)
    return d 

print(dup(l))
```

**Output:**
```
[1, 2, 4, 3, 5, 6, 7, 8, 9]
```

---

## 11. Rotate List by k

```python
def rotate(l, k):
    return l[k:] + l[:k]

l = [1, 2, 4, 3, 5, 6, 7, 8, 9]
print(rotate(l, 5))
```

**Output:**
```
[6, 7, 8, 9, 1, 2, 4, 3, 5]
```

---

## 12. Frequency Dictionary

```python
l = [1, 2, 1, 1, 4, 3, 4, 5, 6, 7, 7, 5, 8, 9, 9]
d = {}

for i in l:
    d[i] = d.get(i, 0) + 1 

print(d)
```

**Output:**
```
{1: 3, 2: 1, 4: 2, 3: 1, 5: 2, 6: 1, 7: 2, 8: 1, 9: 2}
```

---

## 13. Merge Two Dictionaries by Adding Values

```python
d1 = {"a": 1, "b": 2, "c": 3}
d2 = {"b": 5, "c": 1, "d": 3}

def merge(d1, d2):
    for k, v in d2.items():
        if k in d1:
            d1[k] += v
        else:
            d1[k] = v
    return d1

print(merge(d1, d2))
```

**Output:**
```
{'a': 1, 'b': 7, 'c': 4, 'd': 3}
```

---

## 14. Reverse Dictionary: Values → List of Keys

```python
d3 = {"ramesh": 50, "sachin": 100, "raju": 20, "virat": 70, "sanju": 20}
d4 = {}

for k, v in d3.items():
    if v in d4:
        d4[v].append(k)
    else:
        d4[v] = [k]

print(d4)
print(dict(sorted(d4.items())))
```

**Output:**
```
{50: ['ramesh'], 100: ['sachin'], 20: ['raju', 'sanju'], 70: ['virat']}
{20: ['raju', 'sanju'], 50: ['ramesh'], 70: ['virat'], 100: ['sachin']}
```

---

## 15. Check Anagram

```python
def isanag(s1, s2):
    if len(s1) != len(s2):
        return False

    d1 = {}
    d2 = {}

    for i in s1:
        d1[i] = d1.get(i, 0) + 1 
        
    for i in s2:
        d2[i] = d2.get(i, 0) + 1 
        
    for k, v in d1.items():
        if k not in d2 or d2[k] != v:
            return False
            
    return True

print(isanag("silent", "listen"))
print(isanag("silent", "li3ten"))
```

**Output:**
```
True
False
```

---

## 16. Group Anagrams

```python
def grpans(l):
    d = {}
    for i in l:
        key = tuple(sorted(i))
        if key in d:
            d[key].append(i)
        else:
            d[key] = [i]
    return d 

st = ["eat", "tea", "tan", "ate", "nat", "bat"]
print(grpans(st))
```

**Output:**
```
{('a', 'e', 't'): ['eat', 'tea', 'ate'], ('a', 'n', 't'): ['tan', 'nat'], ('a', 'b', 't'): ['bat']}
```

---

## 17. Prime Number Check

```python
def isprime(n):
    if n < 2:
        return "not prime"
    for i in range(2, int(n**0.5)+1):
        if n % i == 0:
            return "not prime"
    return "prime"

print(isprime(16))
print(isprime(17))
```

**Output:**
```
not prime
prime
```

---

## 18. List of Primes Upto N

```python
def primeupton(n):
    l = []
    for i in range(2, n+1):
        isp = True
        for j in range(2, int(i**0.5)+1):
            if i % j == 0:
                isp = False
                break
        if isp:
            l.append(i)
    return l 

print(primeupton(20))
```

**Output:**
```
[2, 3, 5, 7, 11, 13, 17, 19]
```

---

## 19. Word Frequency (Case-Insensitive)

```python
sentence = "python is great and Python is easy to learn"
d = {}
words = []
word = ""

for i in sentence:
    if i == " ":
        words.append(word.lower())
        word = ""
    else:
        word += i

words.append(word)

for i in words:
    d[i] = d.get(i, 0) + 1 

print(d)
```

**Output:**
```
{'python': 2, 'is': 2, 'great': 1, 'and': 1, 'easy': 1, 'to': 1, 'learn': 1}
```

---

## 20. Sorting Words Alphabetically

```python
sentence = "Banana apple Mango orange Kiwi"
words = []
word = ""

for i in sentence:
    if i == " ":
        words.append(word.lower())
        word = ""
    else:
        word += i

words.append(word)
words.sort(key=str.lower)
print(words)
```

**Output:**
```
['apple', 'banana', 'kiwi', 'mango', 'orange']
```

---

## 21. Rotate List (Left & Right)

```python
l = [1, 2, 3, 4, 5, 6, 7, 8]
k = 3 
k = k % len(l)

print(l[-k:] + l[:-k])   # right rotate
print(l[k:] + l[:k])     # left rotate
```

**Output:**
```
[6, 7, 8, 1, 2, 3, 4, 5]
[4, 5, 6, 7, 8, 1, 2, 3]
```

---

## 22. Reverse Integer + Digits Sum + Palindrome

```python
n = 13211
temp = n 
s = 0 
rev = 0 

while n > 0:
    rem = n % 10
    s += rem
    rev = rev * 10 + rem
    n //= 10

print(rev, s)

if rev == temp:
    print("palindrome")
else:
    print("not")
```

**Output:**
```
11231 8
not
```

---

## 23. Merge Two Sorted Arrays

```python
a = [1, 3, 5, 7]
b = [2, 4, 6, 8]

i = j = 0 
res = []

while i < len(a) and j < len(b):
    if a[i] < b[j]:
        res.append(a[i])
        i += 1 
    else:
        res.append(b[j])
        j += 1 
        
res.extend(a[i:])
res.extend(b[j:])

print(res)
```

**Output:**
```
[1, 2, 3, 4, 5, 6, 7, 8]
```

---

## 🎯 End of Basic Python Questions

This guide covers essential Python programming questions commonly asked in interviews. Practice these examples to strengthen your Python fundamentals!

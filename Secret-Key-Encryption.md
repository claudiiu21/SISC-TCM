## Task 1

Create a file, write a message inside of it "Hello World!"  
```echo "Hello World!" > article.txt```

Then, we will use the following 2 commands:
  1) Converts all upper cases to lower cases  
  ```$ tr [:upper:][:lower:] < article.txt > lowercase.txt ```  
  2) Removes all punctuations and numbers  
  ```$ tr-cd '[a-z][\n][:space:]' < lowercase.txt > plaintext.txt ```

Now, create a python file using any editor (ex. ```gedit generate_key.py```) using this code:  
  ```
  #!/bin/envpython3
  import random  
  s = "abcdefghijklmnopqrstuvwxyz"
  list = random.sample(s,len(s))
  key = ''.join(list)  
  print(key)
```  

Then run it
```
$ python3 generate_key.py 
skjfthcviudqrgwmenxlypbzoa
```
Last command which needs to be used, you can replace the key with the one generated for you, but then different outputs will be available.
```
tr 'abcdefghijklmnopqrstuvwxyz' 'skjfthcviudqrgwmenxlypbzoa' \ < plaintext.txt > ciphertext.txt
```
The outputted string will be ```vtqqw bwnqf```  

If we want to decypher /Files/ciphertext.txt we can do it manually, using the provided links for [Bigram frequency](https://en.wikipedia.org/wiki/Bigram) and [Trigram frequency](https://en.wikipedia.org/wiki/Trigram).  
Run and compare the output of the /Files/freq.py to these links, and we can assume that ```ytn``` matches ```the```.  

Or use the following python script:
```
#!/usr/bin/env python3

from collections import Counter
import re
from tkinter.tix import Tree

from sympy import false, true

TOP_K  = 20
N_GRAM = 3

# Generate all the n-grams for value n
def ngrams(n, text):
    for i in range(len(text) -n + 1):
        # Ignore n-grams containing white space
        if not re.search(r'\s', text[i:i+n]):
           yield text[i:i+n]

# Read the data from the ciphertext
with open('ciphertext.txt') as f:
    text = f.read()

# Count, sort, and print out the n-grams

# Most frequent n-grams of size 2 and 3.
n2_gram = ["th", "he", "in", "er", "an"]
n3_gram = ["the", "and", "tha", "ent", "ing"]

ngram_mean_differences = [0, 0]
idx = 0
letters = "abcdefghijklmnopqrstuvwxyz"
alphabet_matches = {}
for let in letters:
    alphabet_matches[let] = ""

for N in range(1, N_GRAM): # We don't consider n-grams of size 1, because they're very misleading
    print("-------------------------------------")
    print("{}-gram (top {}):".format(N+1, TOP_K))
    counts = Counter(ngrams(N+1, text))        # Count
    sorted_counts = counts.most_common(TOP_K)  # Sort
    for ngram, count in sorted_counts:
       print("{}: {}".format(ngram, count))   # Print
    
    for i in range(1, len(sorted_counts[:-1])):
       ngram_mean_differences[idx] += sorted_counts[i - 1][1] - sorted_counts[i][1]

    # Mean frequence analysis difference
    if(len(sorted_counts) - 1 > 0):
        ngram_mean_differences[idx] /= (len(sorted_counts) - 1)

    n_gram_letters_idx = 0
    flag = False
    for i in range(1, len(sorted_counts[:-1])):
        flag = False
        if sorted_counts[i - 1][1] - sorted_counts[i][1] >= ngram_mean_differences[idx]: # Elegible n-gram
            ngram = sorted_counts[i - 1][0]            
            
            if idx == 0: 
                ngram_size = 2 # n-gram (size = 2)
                selected_ngram = n2_gram
            else:
                ngram_size = 3 # n-gram (size = 3)
                selected_ngram = n3_gram

            for j in range(ngram_size):
                ltr = ngram[j]
                if alphabet_matches[ltr] != "" and alphabet_matches[ltr] != selected_ngram[n_gram_letters_idx][j]:
                    print("Collision with letter %s | Current value: %s | New value: %s" % (ltr, alphabet_matches[ltr], selected_ngram[n_gram_letters_idx][j]))
                else:
                    alphabet_matches[ltr] = selected_ngram[n_gram_letters_idx][j]
                    flag = True
        else:
            break
        if flag:
            n_gram_letters_idx += 1
    idx += 1

print(alphabet_matches)
for char in text:
    if char in alphabet_matches.keys() and alphabet_matches[char] != "":
        print(alphabet_matches[char].upper(), end="")
    else:
        print(char, end="")
```

If we use the python script, we are given the following matches (**left side** ciphertext; **right side** plaintext):   
``` { 'm': 'i', 'n': 'e', 'p': 'd', 't': 'h', 'u': 'n', 'v': 'a', 'y': 't'} ```  
We can use the following command: ``` tr 'mnptuvy' 'iedhnat' < ciphertext.txt > decipheredtext.txt ```

The final secret key used for decrypting is: ```tr 'abcdefghijklmnopqrstuvwxyz' 'cfmypvbrlqxwiejdsgkhnazotu' < ciphertext.txt > plaintext_decyphered.txt ```

## Task 2

For the same ```plain.txt``` file use at least 3 different ciphers:

+ **-aes-128-cbc**  
  Encrypt:
  ```
  openssl enc -aes-128-cbc -e -p -in plain.txt -out cipher_aes.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```
  Decrypt:
  ```
  openssl enc -aes-128-cbc -d -p -in cypher_aes.bin -out decipher_aes.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```
+ **-bf-cbc**  
  Encrypt:
  ```
  openssl enc -bf-cbc -e -p -in plain.txt -out cipher_cbc.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```
  Decrypt:
  ```
  openssl enc -bf-cbc -d -p -in cypher_cbc.bin -out decipher_cbc.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```
+ **-aes-128-cfb**  
  Encrypt:
  ```
  openssl enc -aes-128-cfb -e -p -in plain.txt -out cipher_aes_cfb.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```
  Decrypt:
  ```
  openssl enc -aes-128-cfb -d -p -in cypher_aes_cfb.bin -out decipher_aes_cfb.bin  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  ```

## Task 3

For the .bmp file, the first 54 bytes contain the header information about the picture, needs to be set correctly in order for the encrypted file to be treated as a legitimate one.

Need to save the first 

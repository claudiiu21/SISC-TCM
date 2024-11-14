# Task 1

### In the first task, we are introduced to the Random Substitution Cipher, meaning each letter in the original text is replaced by another letter, where the replacement does not vary.

```echo "Hello World!" > article.txt```

``` tr [:upper:][:lower:] < article.txt > lowercase.txt ```  

``` tr-cd '[a-z][\n][:space:]' < lowercase.txt > plaintext.txt ```

``` gedit generate_key.py```
``` 
#!/bin/envpython3
import random  
s = "abcdefghijklmnopqrstuvwxyz"
list = random.sample(s,len(s))
key = ''.join(list)  
print(key)
```

``` python3 generate_key.py ```

``` tr 'abcdefghijklmnopqrstuvwxyz' 'add the key generated from previous py3 command' \ < plaintext.txt > ciphertext_1.txt ```
In the ciphertext_1.txt you should see the hello world message

Next, go to /Files/cipher.txt, in order to decypher this use:  
``` tr 'abcdefghijklmnopqrstuvwxyz' 'cfmypvbrlqxwiejdsgkhnazotu' < ciphertext.txt > plaintext.txt ```


# Task 2

### In this task, we will play with various encryption algorithms and modes. You can use the following openssl enc command to encrypt/decrypt a file. To see the manuals, you can type man openssl and man enc.
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

  # Task 3
  ### The file pic original.bmp is included in the Labsetup.zip file, and it is a simple picture. We would like to encrypt this picture, so people without the encryption keys cannot know what is in the picture. Please encrypt the file using the ECB (Electronic Code Book) and CBC (Cipher Block Chaining) modes, and then do the following:

  ### ECB Encryption
  ```
  ./Labsetup/Files
  openssl enc -aes-128-ecb -e -p -in pic_original.bmp -out pic_ecb_enc.bmp  
  -K 00112233445566778889aabbccddeeff

  ./Labsetup/Files                                                                              
  head -c 54 pic_original.bmp > header

  ./Labsetup/Files                                                                            
  tail -c +55 pic_ecb_enc.bmp > body

  ./Labsetup/Files
  cat header body > pic_ecb.bmp
  ```
  ### CBC Encryption
  ```
  ./Labsetup/Files
  openssl enc -aes-128-cbc -e -p -in pic_original.bmp  -out pic_cbc_enc.bmp  
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708

  ./Labsetup/Files                                                                            
  head -c 54 pic_original.bmp > header_cbc

  ./Labsetup/Files                                                                              
  tail -c +55 pic_cbc_enc.bmp > body_cbc

  ./Labsetup/Files                                                                              
  cat header_cbc body_cbc > pic_cbc.bmp
  ```

  ### ECB vs. CBC
  From the generated pictures it is very easy to see that the ECB mode is much more unsafe than the CBC mode. It's quite easy to see which was the original image in the case of the ECB mode, can't say the same for CBC.
  
  # Task 4: Padding
  ### For block ciphers, when the size of a plaintext is not a multiple of the block size, padding may be required. The PKCS#5 padding scheme is widely used by many block ciphers (see Chapter 21.4 of the SEED book for details). We will conduct the following experiments to understand how this type of padding works:

  ### ECB

  ```
  ./Labsetup/Task4
  openssl enc -aes-128-ecb -e -p -in f1.txt -out f1_enc.txt -K 00112233445566778889aabbccddeeff

  ./Labsetup/Task4
  ll (LL)
  -rw-r--r-- 1 seed seed 16 Nov  14 13:32 f1_enc.txt
  -rw-r--r-- 1 seed seed  5 Nov  14 13:30 f1.txt
  ```
  Now we can decrypt using the -nopad parameter, in order for the padding to not be removed
  ```
  ./Labsetup/Task4
  openssl enc -aes-128-ecb -d -nopad -p -in f1_enc.txt -out f1_dec.txt -K 00112233445566778889aabbccddeeff

  ./Labsetup/Task4
  ll
  -rw-r--r-- 1 seed seed 16 Nov  14 13:39 f1_dec.txt
  -rw-r--r-- 1 seed seed 16 Nov  14 13:32 f1_enc.txt
  -rw-r--r-- 1 seed seed  5 Nov  14 13:30 f1.txt
  ```
  11 bytes are added, we can see by the difference or by using the following command:
  ```
  ./Labsetup/Task4
  hexdump -C f1_dec.txt      
  00000000  31 32 33 34 35 0b 0b 0b  0b 0b 0b 0b 0b 0b 0b 0b  |12345...........|
  00000010
  ```

  #### ==> ECB uses padding
  ### CBC
  Run all the previous commands but CBC instead of ECB
  #### ==> CBC uses padding

  ### CFB
  ```
  openssl enc -aes-128-cfb -e -p -in f1.txt -out f1_cfb_enc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  hex string is too short, padding with zero bytes to length
  salt=4A52B536EE7F0000
  key=00112233445566778889AABBCCDDEEFF
  iv =01020304050607080000000000000000
  
  openssl enc -aes-128-cfb -d -nopad -p -in f1_cfb_enc.txt -out f1_cfb_dec.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  hex string is too short, padding with zero bytes to length
  salt=4A9299684C7F0000
  key=00112233445566778889AABBCCDDEEFF
  iv =01020304050607080000000000000000
  
  hexdump -C f1_cfb_dec.txt                                                     
  00000000  31 32 33 34 35                                    |12345|
  00000005
  ```
  no bytes are added
  #### ==> CFB mode does not need padding (Stream cipher)

  ### OFB
  Run all the previous command but OFB instead of CFB
  #### ==> OFB mode does not need padding (Stream cipher)

  # Task 5
  ### To understand the error propagation property of various encryption modes, we would like to do the following exercise:
  ### 1. Create a text file that is at least 1000 bytes long.
  ### 2. Encrypt the file using the AES-128 cipher.
  ### 3. Unfortunately, a single bit of the 55th byte in the encrypted file got corrupted. You can achieve this corruption using the bless hex editor.
  ### 4. Decrypt the corrupted ciphertext file using the correct key and IV. Please answer the following question: How much information can you recover by decrypting the corrupted file, if the encryption mode is ECB, CBC, CFB, or OFB, respectively? Please answer this question before you conduct this task, and then find out whether your answer is correct or wrong after you finish this task. Please provide justification.

  ### ECB

  ```
  a.  python3 -c "print('A' * 1500, end='')" > test.txt 
  b.  openssl enc -aes-128-ecb -e -p -in test.txt -out test_ecb_enc.txt -K 00112233445566778889aabbccddeeff   
  c.  bless test_ecb_enc.txt  --> change the 55th byte -> check offset:55
   
  openssl enc -aes-128-ecb -d -p -in test_ecb_enc.txt -out test_ecb_dec.txt -K 00112233445566778889aabbccddeeff    

  cat test_ecb_dec.txt   
  d. Only a small portion of the message got screwed up, because ECB is not a chain, no block depends on the previous one.
  ```

  ### CBC
  ```
  a. SAME
  b. openssl enc -aes-128-cbc -e -p -in test.txt -out test_cbc_enc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  c. bless test_cbc_enc.txt
  
  openssl enc -aes-128-cbc -d -p -in test_cbc_enc.txt -out test_cbc_dec.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
 
  cat test_cbc_dec.txt
  
  d. Also only a small portion ==> the next block always depends on the previous one, if we would have modified an entire block, the damage would be higher
  ```

  ### CFB
  ```
  a. SAME
  b. openssl enc -aes-128-cfb -e -p -in test.txt -out test_cfb_enc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  c. bless test_cfb_enc.txt

  openssl enc -aes-128-cfb -d -p -in test_cfb_enc.txt -out test_cfb_dec.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708

  cat test_cfb_dec.txt
  
  d.small part as well, the deciphering of the next part of the stream still depends in N bits on the deciphering of the previous part of the stream
  One bit does not affect the entire plaintext block
  ```

  ### OFB
  ```
  a. SAME
  b. openssl enc -aes-128-ofb -e -p -in test.txt -out test_ofb_enc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
  c. bless_test_ofb_enc.txt

  openssl enc -aes-128-ofb -d -p -in test_ofb_enc.txt -out test_ofb_dec.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708

  cat test_ofb_dec.txt

  d. only one character gets messed up
  ```

  ### Final conclusion
  #### The error propagation is bigger in CBC, due to the high dependency between each block in the deciphering process, then on the CFB and ECB (similar kinda) and finally OFB, where the dependency is minimum.
  

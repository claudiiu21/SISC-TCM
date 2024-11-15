# Task 1 Getting Familiar with Padding

  ```
  ./Labsetup/Task1
  echo -n "12345" > f1.txt
  echo -n "1234512345" > f2.txt
  echo -n "1234512345123456" > f3.txt
  ```

  ```
  ./Labsetup/Task1
  openssl enc -aes-128-cbc -e -p -in f1.txt -out f1_enc.txt
  -K 00112233445566778889aabbccddeeff -iv 0102030405060708

  ./Labsetup/Task1
  ll (LL)
  -rw-r--r-- 1 seed seed 16 Nov  14 13:32 f1_enc.txt
  -rw-r--r-- 1 seed seed  5 Nov  14 13:30 f1.txt
  ```
  Now we can decrypt using the -nopad parameter, in order for the padding to not be removed
  ```
  ./Labsetup/Task1
  openssl enc -aes-128-ecb -d -nopad -p -in f1_enc.txt -out f1_dec.txt -K 00112233445566778889aabbccddeeff 

  ./Labsetup/Task1
  ll
  -rw-r--r-- 1 seed seed 16 Nov  14 13:39 f1_dec.txt
  -rw-r--r-- 1 seed seed 16 Nov  14 13:32 f1_enc.txt
  -rw-r--r-- 1 seed seed  5 Nov  14 13:30 f1.txt
  ```
  11 bytes are added, we can see by the difference or by using the following command:
  ```
  ./Labsetup/Task1
  hexdump -C f1_dec.txt      
  00000000  31 32 33 34 35 0b 0b 0b  0b 0b 0b 0b 0b 0b 0b 0b  |12345...........|
  00000010
  ```

  ### Full block of padding seen for the 16 bytes file because it is a multiple of the block syze, therefore we will get a 32-byte ciphertext, aka a full block of padding has been added
  ### ==> CBC mode uses paddings
  ### SAME for f2.txt, f3.txt

  # Task 2 Padding Oracle Attack (Level 1)

  ## TURN ON DOCKER
  ```
  dcbuild
  dcup
  ```
  ### ERROR: Conflict. Container name "..." is already in use by container "container ID"
  ```
  docker rm <container ID>

  ex. Container name "/oracle-10.9.0.80" is already in use by container "5eaa4ce0061917bfd810e7e1d7063c256c72f50584a9669a6e29634f99f0f3aa".
  docker rm 5eaa

  Now it will work, do dcup again
  ```
  ### OUTPUT:
  ```
  Recreating container ... done
  Attaching to oracle-10.9.0.80
  oracle-10.9.0.80 | Server listening on 5000 for padding_orracle_L1
  oracle-10.9.0.80 | Server listening on 6000 for padding_orracle_L2

  do nc 10.9.0.80 5000 in another terminal and you should see in this terminal:
  oracle-10.9.0.80 | Connect to 5000, launching padding_oracle_L1
  ```

  ## AFTER DOCKER
  ## MANUALLY FINDING THE FIRST 6 BYTES

  ```
  Use manual_attack.py from ./Labsetup

  Valid: i = 0x39
  1) CC1[15] = 0xcc; K=2

  Valid: i = 0xf2
  2) CC1[15] = 0xcd; CC1[14] = 0x38; K=3

  Valid: i = 0xf
  3) CC1[15] = 0xca; CC1[14] = 0x3f; CC1[13] = 0xf5; K=4

  Valid: i = 0x18
  4) CC1[15] = 0xcb; CC1[14] = 0x3e; CC1[13] = 0xf4; CC1[12] = 0x19; K=5

  Valid: i = 0x40
  5) CC1[15] = 0xc8; CC1[14] = 0x3d; CC1[13] = 0xf7; CC1[12] = 0x1a; CC1[11] = 0x43; K=6

  Valid: i = 0xea
  6) CC1[15] = 0xc9; CC1[14] = 0x3c; CC1[13] = 0xf6; CC1[12] = 0x1bb; CC1[11] = 0x42; CC1[10] = 0xeb; K=7
  ```
  ## AUTOMATED WAY OF FINDING ALL BYTES
  ```
    #!/usr/bin/python3
  import socket
  from binascii import hexlify, unhexlify
  
  # XOR two bytearrays
  def xor(first, second):
     return bytearray(x^y for x,y in zip(first, second))
  
  class PaddingOracle:
  
      def __init__(self, host, port) -> None:
          self.s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
          self.s.connect((host, port))
  
          ciphertext = self.s.recv(4096).decode().strip()
          self.ctext = unhexlify(ciphertext)
  
      def decrypt(self, ctext: bytes) -> None:
          self._send(hexlify(ctext))
          return self._recv()
  
      def _recv(self):
          resp = self.s.recv(4096).decode().strip()
          return resp 
  
      def _send(self, hexstr: bytes):
          self.s.send(hexstr + b'\n')
  
      def __del__(self):
          self.s.close()
  
  def decryptBlock(K, current_padding, CC1, D2, C2):
      if K > 16:
          return CC1, D2
  
      for i in range(256):
          CC1[16 - K] = i
          status = oracle.decrypt(IV + CC1 + C2)
          CC1_cp = CC1.copy()
          D2_cp = D2.copy()
  
          if status == "Valid":
              auxI = hex(i)[2:].zfill(2)
              D2[16 - K] = int(xor(bytearray.fromhex(current_padding), bytearray.fromhex(auxI)).hex(), base=16)
              current_padding = hex(int(current_padding, base=16) + 1)[2:].zfill(2)
              for x in range(1, K + 1):
                  auxD2 = hex(D2[16 - x])[2:].zfill(2)
                  CC1[16 - x] = int(xor(bytearray.fromhex(current_padding), bytearray.fromhex(auxD2)).hex(), base=16)
  
              print("Valid: i = 0x{:02x}".format(i))
              print("CC1: " + CC1.hex())
  
              CC1, D2 = decryptBlock(K + 1, current_padding, CC1, D2, C2)
              if CC1 != [] and D2 != []:
                  return CC1, D2
  
          CC1 = CC1_cp
          D2 = D2_cp
  
      return [], []
  
  if __name__ == "__main__":
      oracle = PaddingOracle('10.9.0.80', 5000)
  
      # Get the IV + Ciphertext from the oracle
      iv_and_ctext = bytearray(oracle.ctext)
      IV    = iv_and_ctext[00:16]
      C1    = iv_and_ctext[16:32]  # 1st block of ciphertext
      C2    = iv_and_ctext[32:48]  # 2nd block of ciphertext
      print("C1:  " + C1.hex())
      print("C2:  " + C2.hex())
  
      ###############################################################
      # Here, we initialize D2 with C1, so when they are XOR-ed,
      # The result is 0. This is not required for the attack.
      # Its sole purpose is to make the printout look neat.
      # In the experiment, we will iteratively replace these values.
      D2 = bytearray(16)
  
      D2[0]  = C1[0]
      D2[1]  = C1[1]
      D2[2]  = C1[2]
      D2[3]  = C1[3]
      D2[4]  = C1[4]
      D2[5]  = C1[5]
      D2[6]  = C1[6]
      D2[7]  = C1[7]
      D2[8]  = C1[8]
      D2[9]  = C1[9]
      D2[10] = C1[10]
      D2[11] = C1[11]
      D2[12] = C1[12]
      D2[13] = C1[13]
      D2[14] = C1[14]
      D2[15] = C1[15]
  
      ###############################################################
      # In the experiment, we need to iteratively modify CC1
      # We will send this CC1 to the oracle, and see its response.
      CC1 = bytearray(16)
  
      CC1[0]  = 0x00
      CC1[1]  = 0x00
      CC1[2]  = 0x00
      CC1[3]  = 0x00
      CC1[4]  = 0x00
      CC1[5]  = 0x00
      CC1[6]  = 0x00
      CC1[7]  = 0x00
      CC1[8]  = 0x00
      CC1[9]  = 0x00
      CC1[10] = 0x00
      CC1[11] = 0x00
      CC1[12] = 0x00
      CC1[13] = 0x00
      CC1[14] = 0x00
      CC1[15] = 0x00
  
      ###############################################################
      # In each iteration, we focus on one byte of CC1.  
      # We will try all 256 possible values, and send the constructed
      # ciphertext CC1 + C2 (plus the IV) to the oracle, and see 
      # which value makes the padding valid. 
      # As long as our construction is correct, there will be 
      # one valid value. This value helps us get one byte of D2. 
      # Repeating the method for 16 times, we get all the 16 bytes of D2.
  
      current_padding = "01"
      CC1, D2 = decryptBlock(1, current_padding, CC1, D2, C2)
  
      ###############################################################
  
      # Once you get all the 16 bytes of D2, you can easily get P2
      P2 = xor(C1, D2)
      print("P2:  " + P2.hex())
  ```
  
  # Task 3 Padding Oracle Attack (Level 2)

  ### Prerequisites: type into terminal (required for program to work)
  ```
  pip3 install numpy
  pip3 install tables
  ```
  ### AFTER
  ```
      #!/usr/bin/python3
  import socket
  from binascii import hexlify, unhexlify
  
  from numpy import block
  from tables import Complex128Col
  
  # XOR two bytearrays
  def xor(first, second):
     return bytearray(x^y for x,y in zip(first, second))
  
  class PaddingOracle:
  
      def __init__(self, host, port) -> None:
          self.s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
          self.s.connect((host, port))
  
          ciphertext = self.s.recv(4096).decode().strip()
          self.ctext = unhexlify(ciphertext)
  
      def decrypt(self, ctext: bytes) -> None:
          self._send(hexlify(ctext))
          return self._recv()
  
      def _recv(self):
          resp = self.s.recv(4096).decode().strip()
          return resp 
  
      def _send(self, hexstr: bytes):
          self.s.send(hexstr + b'\n')
  
      def __del__(self):
          self.s.close()
  
  def decryptBlock(K, current_padding, CC1, D2, C2):
      if K > 16:
          return CC1, D2
  
      for i in range(256):
          CC1[16 - K] = i
          status = oracle.decrypt(IV + CC1 + C2)
          CC1_cp = CC1.copy()
          D2_cp = D2.copy()
  
          if status == "Valid":
              auxI = hex(i)[2:].zfill(2)
              D2[16 - K] = int(xor(bytearray.fromhex(current_padding), bytearray.fromhex(auxI)).hex(), base=16)
              current_padding = hex(int(current_padding, base=16) + 1)[2:].zfill(2)
              for x in range(1, K + 1):
                  auxD2 = hex(D2[16 - x])[2:].zfill(2)
                  CC1[16 - x] = int(xor(bytearray.fromhex(current_padding), bytearray.fromhex(auxD2)).hex(), base=16)
  
              print("Valid: i = 0x{:02x}".format(i))
              print("CC1: " + CC1.hex())
  
              CC1, D2 = decryptBlock(K + 1, current_padding, CC1, D2, C2)
              if CC1 != [] and D2 != []:
                  return CC1, D2
  
          CC1 = CC1_cp
          D2 = D2_cp
  
      return [], []
  
  if __name__ == "__main__":
      oracle = PaddingOracle('10.9.0.80', 6000)
      BLOCK_SIZE = 16
  
      # Get the IV + Ciphertext from the oracle
      iv_and_ctext = bytearray(oracle.ctext)
      num_blocks = int((len(iv_and_ctext) - BLOCK_SIZE) / BLOCK_SIZE)
      block_1_ciphertxt_idx = (num_blocks - 1) * BLOCK_SIZE
      block_2_ciphertxt_idx = num_blocks * BLOCK_SIZE
      IV    = iv_and_ctext[00:BLOCK_SIZE]
  
      decipheredText = ""
      current_block = num_blocks
      for i in range(num_blocks - 1):
          C1    = iv_and_ctext[block_1_ciphertxt_idx:(block_1_ciphertxt_idx + BLOCK_SIZE)]  # 1st block of ciphertext
          C2    = iv_and_ctext[block_2_ciphertxt_idx:(block_2_ciphertxt_idx + BLOCK_SIZE)]  # 2nd block of ciphertext
          print("C1:  " + C1.hex())
          print("C2:  " + C2.hex())
  
          D2 = bytearray(16)
  
          D2[0]  = C1[0]
          D2[1]  = C1[1]
          D2[2]  = C1[2]
          D2[3]  = C1[3]
          D2[4]  = C1[4]
          D2[5]  = C1[5]
          D2[6]  = C1[6]
          D2[7]  = C1[7]
          D2[8]  = C1[8]
          D2[9]  = C1[9]
          D2[10] = C1[10]
          D2[11] = C1[11]
          D2[12] = C1[12]
          D2[13] = C1[13]
          D2[14] = C1[14]
          D2[15] = C1[15]
  
          CC1 = bytearray(16)
  
          CC1[0]  = 0x00
          CC1[1]  = 0x00
          CC1[2]  = 0x00
          CC1[3]  = 0x00
          CC1[4]  = 0x00
          CC1[5]  = 0x00
          CC1[6]  = 0x00
          CC1[7]  = 0x00
          CC1[8]  = 0x00
          CC1[9]  = 0x00
          CC1[10] = 0x00
          CC1[11] = 0x00
          CC1[12] = 0x00
          CC1[13] = 0x00
          CC1[14] = 0x00
          CC1[15] = 0x00
  
          current_padding = "01"
          CC1, D2 = decryptBlock(1, current_padding, CC1, D2, C2)
  
          P = xor(C1, D2)
          print("\n\n###############################################################")
          print("Current Block: %d" % current_block)
          print("P:  " + P.hex())
          decipheredText = P.hex() + decipheredText
          print("###############################################################\n\n")
  
          block_1_ciphertxt_idx -= BLOCK_SIZE
          block_2_ciphertxt_idx -= BLOCK_SIZE
          current_block -= 1
      
      # First block (Special Case)
      current_padding = "01"
      CC1, D2 = decryptBlock(1, current_padding, CC1, D2, iv_and_ctext[16:32])
      P = xor(D2, IV)
      print("\n\n###############################################################")
      print("Current Block: %d" % current_block)
      print("P:  " + P.hex())
      decipheredText = P.hex() + decipheredText
      print("###############################################################\n\n")
  
      print("Plaintext (HEX): " + decipheredText)
      print("Plaintext (ASCII): " + bytes.fromhex(decipheredText).decode("unicode_escape"))
  ```

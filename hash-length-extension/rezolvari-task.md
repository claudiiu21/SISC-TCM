# Lab Environment

Set up the container and run it (`www-10.9.0.80`) in the background:

```sh
curl https://seedsecuritylabs.org/Labs_20.04/Files/Crypto_Hash_Length_Ext/Labsetup.zip -o Labsetup.zip
unzip Labsetup.zip
cd Labsetup
dcbuild
dcup -d
```

If necessary, get the running container id by `dockps` and use `docksh <id>` to start a shell on this container.

Add the following entry in `/etc/hosts` *(root privilege required, try `sudo vi /etc/hosts`)*:

```
10.9.0.80 www.seedlab-hashlen.com
```

# Task 1

Construct and send a benign request to the server:

1. Pick up a `uid` with its key value from `Labsetup/image_flask/app/LabHome/key.txt` instead of using a real name, for example, I choose the entry `1001:123456` in this task.
2. Calculate the MAC of the key concatenated with request content `R`, that is

```
Key:R = 123456:myname=clau&uid=1001&lstcmd=1
```

Suppose that the name used here is "clau" and it requests for listing all the files in `LabHome` folder.

So the MAC is calculated as:

```sh
echo -n "123456:myname=clau&uid=1001&lstcmd=1" | sha256sum
#e77ba23ca2f3071eebf615a3eab38b47fb741ece13d8082cc37c62af32661f00  -
```

Thus the complete request is:

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1001&lstcmd=1&mac=e77ba23ca2f3071eebf615a3eab38b47fb741ece13d8082cc37c62af32661f00
```

*Don't use `curl` or `wget`, it doesn't support. Just open a Firefox browser via VNC client and visit the url link above.*

The web looks like:

![](./lstcmd.png)

For a download request, we take a similar strategy to construct:

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1001&lstcmd=1&download=secret.txt&mac=e77ba23ca2f3071eebf615a3eab38b47fb741ece13d8082cc37c62af32661f00
```

![](./download.png)

# Task 2

Construct the padding for 
```
123456:myname=clau&uid=1001&lstcmd=1
```

Use Python REPL to complete this work:

```sh
python
>>> payload = bytearray("123456:myname=clau&uid=1001&lstcmd=1",'utf8')
>>> len(payload)
36
>>> length_field = (len(payload)*8).to_bytes(8,'big')
>>> padding = b'\x80' + b'\x00'*(64-len(payload)-1-8) + length_field
>>> print(''.join('\\x{:02x}'.format(x) for x in padding))
\x80\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x01\x20
# for url-encoding
>>> print(''.join('%{:02x}'.format(x) for x in padding))
%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%20
```

# Task 3

Compile and run [`calculate_mac.c`](./calculate_mac.c), in which [`SHA256_Update`](./calculate_mac.c#L9) takes the padding bytes we obtained in [previous task](#task-2) followed by `&download=secret.txt` as the second argument.


```sh
gcc calculate_mac.c -o calculate_mac -lcrypto
./calculate_mac
```

It gives:

```
64f16899eb5ab2343f29cde7bc949511060e495edb3676fcf6bf6241c6b1bf69
```

---

If it reports an error as:

```
gcc: error: calculate_mac.c: No such file or directory
```

try:

```
sudo apt install libssl-dev
```

---

Then, visit

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1001&lstcmd=1%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%20&download=secret.txt&mac=64f16899eb5ab2343f29cde7bc949511060e495edb3676fcf6bf6241c6b1bf69
```

![](./padding.png)

# Task 4

Alternatively, to distinguish from the existing work, we turn to apply the `1002:983abe` as `mackey-uid` and "MUR" as current username.

A legitimate request to list files without MAC value:

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1002&lstcmd=1
```

can be calculated by

```sh
echo -n "983abe:myname=clau&uid=1002&lstcmd=1" | sha256sum
# 112f6b12cc37fa1b3bfc2ad2d2f8e00640b6924d1456ec5d5c5e486b64b54f6b  -
```

Assume that we have already observed the full request URL as

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1002&lstcmd=1&mac=112f6b12cc37fa1b3bfc2ad2d2f8e00640b6924d1456ec5d5c5e486b64b54f6b
```

But we do not know the mac key of it. So we use [`length_ext.c`](./length_ext.c) to obtain the MAC after appending `"&download=secret.txt"` argument. Compile and run:

```sh
gcc length_ext.c -o length_ext -lcrypto
./length_ext
# a454cff7ac411d226616e71f4be9dce7158263eb5db8b72ff43a376cefb1a43f
```

Then, construct the padding of the original message as [task-2](#task-2), recall that we don't know what the mac key exactly is but we know the length of keys are fixed, so we can easily calculate the padding:

```sh
python
>>> payload = bytearray("******:myname=MUR&uid=1002&lstcmd=1",'utf8')
>>> length_field = (len(payload)*8).to_bytes(8,'big')
>>> padding = b'\x80' + b'\x00'*(64-len(payload)-1-8) + length_field
>>> print(''.join('%{:02x}'.format(x) for x in padding))
%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%20


```

So the full request is:

```
http://www.seedlab-hashlen.com/?myname=clau&uid=1002&lstcmd=1%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%20&download=secret.txt&mac=a454cff7ac411d226616e71f4be9dce7158263eb5db8b72ff43a376cefb1a43f
```

![](./mur.png)

# Task 5

Keyed-hash mesaage authentication code (HMAC) can be used as the follwing example:


```sh
echo -n "myname=clau&uid=1001&lstcmd=1" | openssl dgst -sha256 -hmac "123456"
# (stdin)= 082f4b1b60c84284ced68720fc7968025bd646ee297c8ab035742080fb7cdcfc
```

HMAC works as the figure below shows:


![](./hmac.png)

$H$ is a hash function and $K$ is a secret key, which could be of any length. $B$ denotes the block size for $H$.

For an input message $M$, he inner hash (left part) first computes $H(K \oplus ipad) \mathbin\Vert M$ and its result $h$ is passed to the outer hash in order to perform $H((K \oplus opad) \mathbin\Vert h$, in which `ipad` and `opad` are both constants.

In such an algorithm, the MAC key is required in both 2 hash functions. The MAC of a full message calculated by inner hash is required taking by the outer hash function, so if we don't have the internal result, which is invisible to us, we cannot compute the correct final MAC. Due to the sequential design, if the server applies HMAC instead of ordinary MAC methods we discussed above, the attacker cannot directly construct the MAC of an extended message from the final MAC of a legal request only. Therefore, hash length extension attack will fail.

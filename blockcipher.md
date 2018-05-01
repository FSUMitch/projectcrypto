# Block Cipher Writeup

### Original Question

We of the Illuminati have recovered an AES-encrypted ciphertext block from our mortal enemies the Free Masons! We have also used sup3r l33t methods to gain access to part of the key. Can you recover the original message so we can continue our plans for world domination?

```
key: 60XXddbb8da6deb21eYY547ced9aZZ4d
ciphertext: "f07686754544a221f1f0f4ccf9d2de63"
```

### Solve

For this question, we have a partial key and a ciphertext message. If we knew the entire key, the question would be as simple as using pycrypto's AES module to decrypt the ciphertext.

Unfortunately, the XX, YY, and ZZ indicate that three bytes of the original plaintext are missing. However, three byes are only 2^24 bits, which is easily possible to brute force through each possible key value. Since we know the flag starts with "flag{", we can search for this and print out results with the same beginning. This should only give us 1 (correct) answer. The following code does the trick:

```
from Crypto.Cipher import AES #import pycrypto's AES class
c = 'f07686754544a221f1f0f4ccf9d2de63'.decode('hex')
p = '6000ddbb8da6deb21e01547ced9a024d'.decode('hex') #changed XX, YY, and ZZ to give valid hex decodings

for i in range(256):
    for j in range(256):
        for k in range(256):
            aes = AES.new(p.replace('\x00', chr(i)).replace('\x01', chr(j)).replace('\x02', chr(k)), 1)
                if aes.decrypt(c).startswith('flag{'): print aes.decrypt(c)
```

This code prints out the flag, and we are done.

### Homework Question - Feistel 1

For this question, when we connect to the server we get a message that we are playing a game. In this game, during each round we have access to a special encryption oracle.

* Half of the time, the oracle is a random function - this random function takes in any 64-bit input and outputs a random 64-bit number
* The other half of the time, the oracle is one round of feistel encryption - it performs one round of the feistel structure shown in the lecture notes.

Our job is to differentiate these two cases (many, many times).

How do we differentiate feistel from random? We must look at the feistel structure itself. In short, it transforms the left half of our input while moving the right half of our input to the left half. Here is some pseudocode showing the transformation:

```
newL1  = R1
newR1 = F(R1) XOR L1

L1 = newL1
R1 = newR1
```

For this problem, we need to only look at the first and third lines of code. The new L1 is simply the old R1. So when we input a message for the oracle to encrypt, the left half of the output will be the right half of the input. If we input a string of all `0`s, then that means the right half of the 0s will be converted to the left half of the output message.

We can make this our factor to differentiate the two cases in each round:

* If the oracle is Feistel, the output will be at most 32 bits because the left 32 will all be 0.
* If the oracle is Random, it is overwhelmingly likely that at least one of the left 32 bits will be 1.

So for our solution, all we have to do is check if the result is greater than 32 bits. If yes, guess random, if no, guess Feistel.

Here is pseudocode of our result:

```
c = connect_to_server()
while True:
    c.send('o')
    c.send('0')

    result = c.receive()
    c.send('g')

    if result > 2**32: c.send('0') # '0' indicates random
    else: c.send('1') # '1' indicates real (feistel)
```

Once the code has finished running, it will error out when receiving the flag and print it.

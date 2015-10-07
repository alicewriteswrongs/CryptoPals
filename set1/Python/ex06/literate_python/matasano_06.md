#Matasano exercise 06

This is a literate Python explanation of my solution to the
[sixth](http://cryptopals.com/sets/1/challenges/6/) Matasano problem. Here we
have a message which has been encrypted with repeating-key XOR (which is, more
or less, the Vigenere's cipher). It's encoded in Base64, and we want to figure
out how to read it! Great! How do?

Well, actually the Matasano folks give us a really nice explanation of how to
proceed. Basically I'm going to take their overview, modify it somewhat, and
intersperse the code that performs that step. Cool!

First off, we'll need this later:


~~~~{.python}
from base64 import b64decode
~~~~~~~~~~~~~



##KEYSIZE

Keysize is the length of the key. Since this is the repeating-key XOR
cryptosystem this is basically the number of bytes in a key. A message is
encrypted by XORing character `i` with `key[i % keysize]`, so for a keysize of
4 the fourth, eighth, twelfth, and so on characters will all be XORed with
the fourth byte of the key (and same for the first, fifth, ninth, etc.
with the first byte of the key).

Matasano lets us know that we only have to worry about keysize ranging between
2 and 40 (they're so helpful with the hints sometimes!). So we'll need some way
to figure out the most appropriate keysize - once we know that we can get to
the business of figuring out the key.

##Hamming Distance

Hamming distance is a metric for *string difference*, and in this case we want
to essentially count the number of bits where two strings (C-style bytestrings)
are different.

Here's a little function to do that:


~~~~{.python}
def distance(s1, s2):
    return sum(bin(x^y).count('1') for x,y in zip(s1,s2))
~~~~~~~~~~~~~



Ok, so we zip string one and string two together, then we XOR them (which
will leave ones wherever they differ), using `bin` to get a string
representation of that, and then count the number of ones. If we sum this
across `zip(s1,s2)` we get our difference. Nice!

##Finding the right KEYSIZE

OK, now that we've defined the Hamming distance, we can use that to find
an appropriate keysize with which to move forward. Basically, we expect
that if we have the right keysize, then if we chunk the ciphertext into
blocks of `keysize` length, we should see a lower Hamming distance between
those chunks than we would between chunks of a randomly selected length.
This is because if we have `keysize` correct, then those chunks will have
been XORed against the same block, and so will have that in common. Great!

This is a class I have named `keysieve` which does this for us:


~~~~{.python}
class Keysieve(object):
    def __init__(self, ciphertext, minkey, maxkey):
        self.scores = []
        self.keys = range(minkey, maxkey + 1)
        self.ctext = ciphertext
        self.sieve()

    def sieve(self):
        for ksize in self.keys:
            chunks = [self.ctext[i*ksize:(i+1)*ksize] for i in range(10)]
            scores = [distance(first, i)/ksize for i in rest]
            self.scores.append((ksize, avg(scores)))
        self.scores.sort(key = lambda x: x[1])
~~~~~~~~~~~~~


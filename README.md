# Cracking the coin code with Python

![The tails side of a 50c coinc.](./media/tails.jpg)

My work asked me to have a look at the [Australian Signals Directorate (ASD) 75th anniversary coin challenge](https://www.abc.net.au/news/2022-09-01/act-spy-agency-releases-coin-with-secret-code/101391964).
The agency is using the coin as a novel recruitment/marketing campaign.
See its website for more details, [including higher-resolution images of the coin](https://www.asd.gov.au/75th-anniversary/events/commemorative-coin-challenge).

I'm a cryptography n00b; I barely knew the difference between binary and hexadecimal numbers before this.
But my manager told me: "Surely we should know what this coin says before we bring everyone's attention to it?"

So I stumbled through the puzzles &mdash; and they were fun!
Given that some people have already published how-to guides online, I figure it's worth writing down how I approached this with Python, partly so I don't forget what I learnt.

If you want to decipher the messages yourself, stop reading and throw yourself into it!
But if you lack time, learn from me.
I wasted hours on the wrong ideas (the coin [looks like a Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher) but it's not) and even more hours because of a typo in my code.

In fact, let's start by avoiding typos.
These are error-free strings of the characters we can see on the tails side.

```python
# encrypted messages on the tails side
# (weights are coded as 1=light, 2=striped, 3=dark)

outer_ring_characters = "DVZIVZFWZXRLFHRMXLMXVKGZMWNVGRXFOLFHRMVCVXFGRLM.URMWXOZIRGBRM7DRWGSC5WVKGS."
outer_ring_weights = "311112111132333312113332213323332133323123133213332323132123113231231321312"

inner_ring_characters = "BGOAMVOEIATSIRLNGTTNEOGRERGXNTEAIFCECAIEOALEKFNR5LWEFCHDEEAEEE7NMDRXX5"
inner_ring_weights = "1333331131331113331331333311113331311133133113313311333331133133113313"

hexcode = """
    E3B
    8287D4
    290F723381
    4D7A47A291DC
    0F71B2806D1A53B
    311CC4B97A0E1CC2B9
    3B31068593332F10C6A335
    2F14D1B27A3514D6F7382F1A
    D0B0322955D1B83D3801CDB2
    287D05C0B82A311085A03329
    1D85A3323855D6BC333119D
    6FB7A3C11C4A72E3C17CCB
    B33290C85B6343955CCBA3
    B3A1CCBB62E341ACBF72
    E3255CAA73F2F14D1B27A
    341B85A3323855D6BB33
    3055C4A53F3C55C7B22
    E2A10C0B97A291DC0F
    73E3413C3BE392819
    D1F73B331185A33
    23855CCBA2A3
    206D6BE383
    1108B
"""
hexcode = "".join(hexcode.splitlines()).replace(" ", "")
```

I recorded the _weights_ of the characters in each of the two rings &mdash; each character is either light, striped or dark.
I first thought these weights disguised a code wheel; [some kind of a substitution cipher](https://en.wikipedia.org/wiki/Substitution_cipher).
But if you filter the characters by weight, none of the string lengths are equal, which rules out a simple code wheel.

It wasn't until I turned the coin over that I started to get somewhere:

## Puzzle 1: Braille

![The heads side of a 50c coinc.](./media/heads.jpg)

It's hard to miss the Braille "hidden" on this face of the coin.
While the ASD says you can decrypt most of the coin's messages with just pen and paper, it forgot to mention that you need to know [how to read Braille](https://en.wikipedia.org/wiki/Braille).
Perhaps it should have said pen, paper and Wikipedia?

There are six Braille characters on the coin, each directly under a Roman letter.
Wikipedia tells us that the first 10 Braille characters have dual meanings: they can represent a number or a letter.

| Letter | B | T | H | A | S | A |
| --- | --- | --- | --- | --- | --- | --- |
| Braille | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Braille_C3.svg/40px-Braille_C3.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Braille_B2.svg/40px-Braille_B2.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/Braille_F6.svg/40px-Braille_F6.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Braille_A1.svg/40px-Braille_A1.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Braille_E5.svg/40px-Braille_E5.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Braille_D4.svg/40px-Braille_D4.svg.png"> |
| Meaning 1 | 3 | 2 | 6 | 1 | 5 | 4 |
| Meaning 2 | C | B | F | A | E | D |

As it happens, the Braille characters used are the first six in the alphabet.
It's highly unlikely this is random &mdash; the Braille is probably a numeric code (`326154`) rather than a reference to Roman letters (`CBFAED`).

So what's it mean?
We have `BTHASA` aligned with `326154` &hellip; a sequence?
But if we rearrange the letters in the order of the numbers, we get a nonsense word:

```python
brail_letters = "BTHASA"
brail_numbers = "326154"
brail_clue = "".join([brail_letters[brail_numbers.index(str(i))] for i in range(1, 7)])
print(brail_clue)
```

    ATBASH

Is it really a nonsense word, though? [No, it's a clue](https://en.wikipedia.org/wiki/Atbash)! (Thanks again, Wikipedia.)

And yes, pen and paper would have been much faster than the Python above.

## Puzzle 2: Atbash cipher

The Atbash cipher is an ancient encryption method that was first used to conceal messages written in Hebrew.
But it can be applied to any sequentially ordered alphabet.
The cipher is the alphabet in reverse order.
This is how it's usually used with the Roman alphabet:

| Letter | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Cipher | Z | Y | X | W | V | U | T | S | R | Q | P | O | N | M | L | K | J | I | H | G | F | E | D | C | B | A |

So `MARKUS` would be encrypted as `NZIPFH`.

This is extremely simple &mdash; surely too simple for the ASD?
Let's go back to the tails side of the coin and try it, starting with the outer ring of characters:

```python
import string
letters = string.ascii_uppercase

# NB This solves simple strings of upper-case Roman characters only
def atbash_cipher(code):
    '''A function to decipher Atbash-encoded strings'''    
    decoded_message = "".join([(letters[-letters.index(x) - 1] if x in letters else x) for x in code])
    return decoded_message

# decode the outer ring message
print(atbash_cipher(outer_ring_characters))
```

    WEAREAUDACIOUSINCONCEPTANDMETICULOUSINEXECUTION.FINDCLARITYIN7WIDTHX5DEPTH.

Worked first time! It looks like we get two messages out of this:

> We are audacious in concept and meticulous in execution.

Hmm, very corporate. Oh look: [it's a reference to ASD's values](https://www.asd.gov.au/about/values).
But we also decoded a clue:

> Find clarity in 7 width x 5 depth.

At this point, I checked Google &hellip; and got a bit lost.
I read about matrix encryption and block ciphers &mdash; they were beyond n00bish me.
I wondered whether `CLARITY` was some kind of encryption key.

Then a Google search of *encryption*, *width* and *depth* led me to discover the route cipher, [a kind of transposition cipher](https://en.wikipedia.org/wiki/Transposition_cipher).
It seemed worth a try.

## Puzzle 3: Transposition cipher

This is another simple encryption method.
You write a message into a grid/matrix (we already know it's 7 columns x 5 rows).
However, to encrypt the message, you don't follow the usual left-to-right, top-to-bottom route when you enter the characters &mdash; you take a different path.
And if you want to *read* the message, you need to know which route, or path, to follow.

A 7 x 5 matrix has 35 characters. Will that fit the next message we're trying to decode: the inner ring of characters?
Let's check:

```python
print(len(inner_ring_characters))
```

    70

Not a perfect fit, but 70 characters *does* perfectly fill two matrices. So we're probably on the right track. Let's prepare the message:

```python
import pandas as pd

# transposition cipher using two 7 x 5 matrices
grid_1 = pd.DataFrame(
    columns=[w for w in range(7)],
    index=[d for d in range(5)]
)
grid_2 = grid_1.copy()

# enter encrypted message in the usual (left-to-right, top-to-bottom) manner
for i, character in enumerate(inner_ring_characters):
    if i < 35:
        grid_1.at[i // 7, i % 7] = character
    else:
        grid_2.at[(i - 35) // 7, (i - 35) % 7] = character

# display the matrices
print("Transposition matrix 1")
print(grid_1)
print("Transposition matrix 2")
print(grid_2)
```

#### Transposition matrix 1
| &nbsp; | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | B | G | O | A | M | V | O |
| **1** | E | I | A | T | S | I | R |
| **2** | L | N | G | T | T | N | E |
| **3** | O | G | R | E | R | G | X |
| **4** | N | T | E | A | I | F | C |

#### Transposition matrix 2
| &nbsp; | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **0** | E | C | A | I | E | O | A |
| **1** | L | E | K | F | N | R | 5 |
| **2** | L | W | E | F | C | H | D |
| **3** | E | E | A | E | E | E | 7 |
| **4** | N | M | D | R | X | X | 5 |

This is another puzzle you might have solved more easily with paper and pen.
Can you see the answer?
Remember, look for a different reading route.

In this case, it's a simple matter of top-to-bottom, left-to-right: `BELONGINGTOA` &hellip; this is starting to look like words!
Here's code to display it all:

```python
# decode two 7 x 5 transposition ciphers, reading
# the message top-to-bottom, left-to-right

decoded_message = ""

for i in range(70): # no. of characters in message
    if i < 35: # first matrix
        decoded_message = decoded_message + grid_1.at[i % 5, i // 5]
    else: # second matrix
        decoded_message = decoded_message + grid_2.at[(i - 35) % 5, (i - 35) // 5]

print(decoded_message)
```

    BELONGINGTOAGREATTEAMSTRIVINGFOREXCELLENCEWEMAKEADIFFERENCEXORHEXA5D75

Again, we get  two messages from this:

> Belonging to a great team striving for excellence, we make a difference.

More ASD value statements! My *esprit de corps* is bulging! But I'm more interested in the last bit:

> XORHEXA5D75

Now, if this challenge has taught me anything, it's the value of Wikipedia.
And yes, a quick search tells me there's [such a thing as a XOR cipher](https://en.wikipedia.org/wiki/XOR_cipher).

## Puzzle 4: XOR cipher

XOR stands for eXclusive OR, a logical operation that you can read about elsewhere.
In fact, you'll find dozens of XOR calculators, encoders and decoders online.

The problem? Almost none of them worked for me.

The reason for that is the next part of the clue: `HEX`.
It's clear this means [hexademical code](https://en.wikipedia.org/wiki/Hexadecimal).
Scan the small characters in the centre of the coin: they're mostly numbers, with some letters thrown in.
However, you'll note that only six letters are used: `A` through to `F`.
So, 10 digits plus six letters makes 16 characters: all those characters are almost certainly a base 16 numeric code &mdash; another way of saying hexademical code.

The final part of the clue, `A5D75`, is also hex code &mdash; and a cute way of writing ASD75 (happy birthday, ASD!).

At this point, I'll spare you the several frustrated hours I spent reading about XOR and trying to decrypt code with a typo in it.
This is what matters:

1. XOR encryption usually involves a key (ours is `A5D75`) that is repeated until it matches the length of the encrypted message.
This repeated key (e.g. `A5D75A5D75A5D75...`) is used as a XOR cipher to decode a section of the message of matching length.

2. Every example of a Python XOR cipher that I found was deprecated (Python 2 only), unable to work with hex code, and/or overly complex.
So I wrote my own.

Here's the decryption function (by the way, it's the same as an encryption function):

```python
# XOR cipher for hexademical code
def xorhex_cipher(message, key):
    '''XOR cipher for hex-encoded messages and keys that
    outputs a hex-encoded result (encryption or decryption)'''
    
    code = "" # empty string to store encoded/decoded hex
    
    # interate through message, applying XOR operation
    for i in range(len(message)):
        code = code + "%x" % (int(message[i], 16) ^ int(key[i % len(key)], 16))

    return code

decoded_message = xorhex_cipher(hexcode, "A5D75")

# print as readable text (hex to bytes)
print(bytes.fromhex(decoded_message).decode())
```

    For 75 years the Australian Signals Directorate has brought together people with the skills, adaptability and imagination to operate in the slim area between the difficult and the impossible.

We did it &mdash; our efforts yielded another corporate message!

> For 75 years the Australian Signals Directorate has brought together people with the skills, adaptability and imagination to operate in the slim area between the difficult and the impossible.

## Epilogue: A fifth puzzle?

A day after releasing the coin, the ASD announced that [it contained a bonus message](https://www.abc.net.au/news/2022-09-02/asd-50-cent-code-cracked-by-14yo-tasmanian-boy/101401978).
I'm not particularly keen to read it, as it's almost certainly a LinkedIn-style message like the others.
But cracking it should be fun.

The question is: where to start? One apparent place is the series of strange characters at the base of the coin's tails side:

![A series of strange characters](./media/codelogo.jpg)

However, if you look at [the bottom of the ASD website](https://www.asd.gov.au/), this is simply what the agency calls its "code logo".

Does it mean anything?
Is it perhaps a key to solve the mystery of the differently weighted characters in the outer ring?

What about the three equal sectors that the coin is split into?

I'm entirely out of ideas, but I'll update this page if I find out.

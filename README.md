# Cracking the coin code with Python

![The tails side of a 50c coinc.](./media/tails.jpg)

My work asked me to have a look at the [Australian Signals Directorate (ASD) 75th anniversary coin challenge](https://www.abc.net.au/news/2022-09-01/act-spy-agency-releases-coin-with-secret-code/101391964).
The agency is using the coin as a novel recruitment/marketing campaign.
There are more details on the ASD website, [including high-resolution images of the coin](https://www.asd.gov.au/75th-anniversary/events/commemorative-coin-challenge).

I'm a cryptography n00b and always will be; before this, I barely knew the difference between binary and hexadecimal numbers.
But my manager told me: "Surely we should know what this coin says before we bring everyone's attention to it?"

So I stumbled through the puzzles &mdash; and they were fun!
And given that some people have already published how-to guides online, I figure it's worth writing down how I approached this with Python, in part so I don't forget what I learned.
(Most people seemed to use C++, because they're real programmers, or R, because they're fashionable.)

If you want to decipher the ASD's encrypted messages yourself, stop reading and throw yourself into it!
But if you lack time, learn from me.
I wasted hours on the wrong ideas (the coin [looks like a Caesar cipher](https://en.wikipedia.org/wiki/Caesar_cipher) but it's not) and even more hours because of a typo in my code.

In fact, let's start by avoiding typos.
These are error-free strings of the characters we can see on the tails side.

```python
# encrypted messages on the tails side (weights are coded as 1=light, 2=striped, 3=dark)

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

I recorded the _weights_ of the characters in each of the two rings; each character is either light, striped or dark.
I initially thought these weights disguised a code wheel; [some kind of a substitution cipher](https://en.wikipedia.org/wiki/Substitution_cipher).
But if you filter the characters by weight, none of the string lengths are equal, which rules out a simple code wheel (more on that later).

It wasn't until I turned the coin over that I started to get somewhere:

## Puzzle 1: Braille

![The heads side of a 50c coinc.](./media/heads.jpg)

It's hard not to notice the Braille "hidden" on this face of the coin.
While the ASD says you can decrypt most of the coin's messages with just pen and paper, it forgot to mention you need to [know how to read Braille](https://en.wikipedia.org/wiki/Braille).
Perhaps it should have said pen, paper and Wikipedia?

There are six Braille characters on the coin, each directly under a Roman letter.
Wikipedia tells us that the first 10 Braille characters have dual meanings: they can represent a number or a letter.

| Letter | B | T | H | A | S | A |
| --- | --- | --- | --- | --- | --- | --- |
| Braille | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Braille_C3.svg/40px-Braille_C3.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/f/fb/Braille_B2.svg/40px-Braille_B2.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/9b/Braille_F6.svg/40px-Braille_F6.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Braille_A1.svg/40px-Braille_A1.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/8/85/Braille_E5.svg/40px-Braille_E5.svg.png"> | <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Braille_D4.svg/40px-Braille_D4.svg.png"> |
| Meaning 1 | 3 | 2 | 6 | 1 | 5 | 4 |
| Meaning 2 | C | B | F | A | E | D |

As it happens, the Braille characters used are the first six in the alphabet.
In other words, it's highly likely that the Braille is a numeric code (`326154`) rather than a reference to Roman letters (`CBFAED`).

So, what's it mean?
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

(Yes, pen and paper would have been much faster than the Python above.) 

## Puzzle 2: Atbash cipher

The Atbash cipher is an ancient encryption method that was first used to conceal messages written in Hebrew.
But it can be applied to any sequentially ordered alphabet.
This is how it would normally be used in English messages:

| Letter | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R | S | T | U | V | W | X | Y | Z |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Cipher | Z | Y | X | W | V | U | T | S | R | Q | P | O | N | M | L | K | J | I | H | G | F | E | D | C | B | A |

So `MARKUS` would be encrypted as `NZIPFH`.

This is extremely simple &hellip; too simple for ASD? Let's go back to the tails side of the coin and try it, starting with the outside ring:

```python
# NB This solves simple strings of upper-case Roman characters only

def atbash_cipher(code):
    '''A function to decipher Atbash-encoded strings'''
    letters = string.ascii_uppercase
    decoded_message = "".join([(letters[-letters.index(x) - 1] if x in letters else x) for x in code])
    return decoded_message

# decode the outer ring message
print(atbash_cipher(outer_ring_characters))
```

    WEAREAUDACIOUSINCONCEPTANDMETICULOUSINEXECUTION.FINDCLARITYIN7WIDTHX5DEPTH.

Worked first time! It looks like we got two messages out of this:

> We are audacious in concept and meticulous in execution.

Hmm, very corporate. But we also decoded a clue:

>






asdasdasd

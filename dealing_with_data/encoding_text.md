# Encoding Text

## ASCII

Bits in a computer typically do something useful, like encoding assembly instructions, whole programs, images, text, etc. But we need an agreed-upon standard for encoding these things. Example: the earliest extant text encoding format is `ASCII` which means **A**merican **S**tandard **C**ode for **I**nformation **I**nterchange. It was a standard created to represent texts.

Basically is a table who relates a 7-bit **number** with control signals and printable characters. For example, the number _97 (0b1100001)_ represents the character _a_, while _98 (0b1100010)_ is _b_. Note that these numbers don't exceed 7 bits, but since in computing we almost always talk about bytes, it turns out that an ASCII character has 8 bits but only uses 7. The ASCII table goes from 0 to 127 and can be found in the ASCII Table appendix.

We can use the function `chr()`in python to know that the number _97_ is the character _a_.

```python
>>> chr(97)
'a'
```

When we type _a_, the computer understands the byte 0x61 (97 in decimal). Similarly, when a program that displays text on the screen encounters byte 0x61, it displays the character _a_. This is exactly how ASCII text ends up inside a program or file.

```python
>>> b'hello'.hex(' ')
'68 65 6c 6c 6f'
```

_Obs: the lowercase b creates an object of the `bytes` class instead of `str`._



<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption><p>ASCII Table</p></figcaption></figure>

## Unicode

You are probably thinking how difficult it is for computers to work with different text encodings, which is why we have Unicode.

### UTF-8

The 8-bits UTF standard (_Unicode Transformation Format_) was created to cover all the possible characters in the various language on this planet.

The first 128 characters of the UTF-8 table have exactly the same values as the standard ASCII table and only need 1 byte to be represented. We call these numbers codepoints. The next characters use _**2 bytes**_ and include not only the Latin alphabet (as in the extended ASCII with ISO-8859-1 encoding) but also Greek, Arabic, Hebrew and other characters. To represent the characters of languages such as Chinese and Japanese, _**3 bytes**_ are required. Finally, there are the characters of ancient manuscripts, mathematical symbols and even emojis, which use _**4 bytes**_.

We concluded that UTF-8 characters range from 1 to 4 bytes. Lets see how would be the text "pãodequeijo" in a sequence of bytes.

```python
>>> 'pãodequeijo'.encode('utf-8').hex(' ')
'70 c3 a3 6f 64 65 71 75 65 69 6a 6f'
```

As said before, the codepoints in the ASCII table are the same in UTF-8, but the character _ã_ (which doesn't exist in pure ASCII) uses 2 bytes (in this case, c3 a3) to be represented. This is a valid UTF-8 string. We can say its size is 11 because it contains 11 characters, but in bytes its size is 12.

### UTF-16

Represented in UTF-16, the characters in the ASCII table are 2 bytes long, where the first byte is the same as in the ASCII table and the second is a zero. For example, to write _A_ in UTF-16, we would write: 41 00. Let's understand this better with the help of Python.

```python
>>> b'hello'.hex(' ')
'68 65 6c 6c 6f'
```

So far, nothing new, but let's see how this string would be encoded in UTF-16:

```python
>>> 'hello'.encode('utf-16').hex(' ')
'ff fe 68 00 65 00 6c 00 6c 00 6f 00'
```

Two things happened in this conversion: the first is that a sequence of two bytes, FF FE, was placed at the beginning of the string. This sequence is called the Byte Order Mark (BOM). The second thing is that the bytes have been followed by zeros.

It's also important to note that UTF-16 strings also have the possibility of four-byte characters. For example, an emoji:

```python
>>> '🧙'.encode('utf-16-le').hex(' ')
'3e d8 d9 dd'
```

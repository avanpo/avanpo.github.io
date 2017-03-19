---
title: "Character encoding refresher"
categories: encoding unicode
---

Sometimes I forget the details on different character encodings. Maybe you do too.

### Unicode

Unicode is a standard designed to facilitate consistent encoding and representation of text. Unicode encodes graphemes to code points, where a grapheme is the smallest unit of a writing system of a language. A code point is a number. So Unicode by itself is not enough to store text on a computer, since the way that the code points should be stored in memory is not defined.

The codespace defined by Unicode spans `0x0` to `0x10ffff` (2^20 + 2^16 = 1,114,112 code points in total). This includes seventeen *planes* of `0xffff` codepoints. The zero plane is called the Basic Multilingual Plane, and contains characters for almost all modern languages and a large number of symbols. Plane 1, the Supplementary Multilingual Plane, includes historic scripts along with symbols for a variety of fields. The vast majority of the codespace is unassigned, leaving lots of room for additions. This happens regularly, such as the avocado emoji (&#x1f951;) which was added in 2016 with the Unicode 9.0 release.

As previously mentioned, Unicode does not define how to store text in memory. It leaves that up to one of its implementations, such as UTF-8 or UTF-16.

Code points are typically denoted using a prefix of `U+`, followed by a hexadecimal number.

#### UTF-8

UTF-8 (where UTF stands for Unicode Transformation Format) is a variable-length character encoding that encodes all Unicode code points to a sequence of bytes. These sequences can be anywhere from one to four bytes long.

The first 128 Unicode code points are mapped directly to one byte, easily identified as such by the leading zero. If the byte contains one or more leading ones, it is part of a multi-byte sequence. This is better explained by a table.

| Unicode range      | Byte sequence
|--------------------|--------------------------------------
| `U+0000-U+007f`    | `0xxxxxxx`
| `U+0080-U+07ff`    | `110xxxxx 10xxxxxx`
| `U+0800-U+ffff`    | `1110xxxx 10xxxxxx 10xxxxxx`
| `U+10000-U+10ffff` | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx`

This encoding has a number of advantages. For one, UTF-8 is a proper superset of ASCII -- encoded ASCII text is the same as when the text is UTF-8 encoded. Secondly, it is trivial to distinguish between single byte sequences, the leading byte of a multi-byte sequence, or a non-leading byte. There are also no issues with endianness, since UTF-8 is byte oriented.

UTF-8 is the default character encoding for the web, since the HTML5 standard. It is by far the most widely used encoding, nearing 89% of the web.

For more details on the actual encoding function, see [RFC 3629](https://tools.ietf.org/html/rfc3629). 

#### UTF-16

UTF-16 is another implementation of Unicode, mapping the codespace to two or four bytes. Unlike UTF-8, it is defined in units of two bytes, meaning the byte order depends on the computer architecture. This problem is solved by including a BOM (byte order mark), a special character `U+feff`, combined with a non-character `U+fffe` which can serve as a hint that the byte order needs to be swapped.

The Unicode points `U+0000` to `U+d7ff`, and `U+e000` to `U+ffff` are mapped directly to two bytes. The missing sequence corresponds to Unicode characters allocated specifically for UTF-16, called surrogates. These allow Unicode code points `U+10000` and higher to be encoded as a pair of two surrogates instead, one high and one low.

While you probably won't see UTF-16 on the web, it is still widely used in software internally (particularly Windows). For example, a Java `char` is 16 bits and is equivalent to a UTF-16 code unit. Strings are therefore represented internally as UTF-16.

UTF-16 is an extension of the deprecated **UCS-2** encoding. The earliest versions of Unicode managed to fit into 16 bits, allowing a fixed length implementation: UCS-2. When Unicode was extended, UTF-16 was created to accomodate it.

#### UTF-32

UTF-32 is a fixed length implementation of Unicode, where a code point is represented directly as its numerical value.

While UTF-32 is space inefficient since 32 bits are used for each code point while 21 would suffice, a string encoded in this manner is indexable in constant time. This cannot be said for the more space efficient variable length encodings.

UTF-32 was originally known as **UCS-4**.

### ASCII

The American Standard Code for Information Interchange is a character encoding standard that predates Unicode by more than twenty years. It consists of [128 code points](https://en.wikipedia.org/wiki/ASCII#Code_chart), spanning 7 bits. On modern computers, this leaves one unused bit, and this bit has been used in a number of creative ways.

ASCII is a proper subset of Unicode, which is quite convenient. For example, ASCII encoded text can easily be parsed using UTF-8. The reverse is possible too (provided no non-ASCII characters were used).

It's a great encoding for applications that only require the English language and a very small set of symbols, but for everything else it falls short.

### ISO-8859

The ISO-8859 standards are a series of fixed length, eight bit encodings. There are sixteen of them, each a proper superset of ASCII by making use of the eight bit.

The extended code points initially included a number of unused characters, in positions 0x80 to 0x9f. These ensured that if the high bit were stripped, a printable character would not suddenly turn into a control character. Later, these unused characters were supplemented with control characters from other standards.

The ISO-8859 standards are no longer updated, having been abandoned in favor of Unicode.

#### ISO-8859-1

The most widely used such standard is ISO-8859-1, also named *Latin-1 Western European*. It covers most of the languages in Western Europe (some missing a few characters), along with several African and Asian languages that happen to fit. Interpreted numberically, it is a proper subset of Unicode, but not compatible with any of the UTF encodings.

ISO-8859-1 is still used on more than 5% of the web, although its usage has been steadily declining.

#### ISO-8859-15

ISO-8859-15, known as *Latin-9*, is a revision of ISO-8859-1 that removes some little-used characters and replaces them with others. In particular, it added the euro sign (&#x20ac;) and some letters that were missing from several languages.

This character encoding was meant to replace ISO-8859-1 and become the standard eight bit character encoding, but this attempt was ultimately unsuccessful.

### Windows-1252

The Windows-1252 character encoding (also known as CP-1252) is similar to ISO-8859-1, except that it uses printable characters in the 0x80 to 0x9f range instead of control characters. It added the characters introduced in ISO-8859-15 (albeit in different places), along with several others.

Incorrect labeling of this charset as ISO-8859-1 resulted in a large number of problems, since printable characters (such as curly quotation marks and others) were replaced with boxes or other characters on non-Windows operating systems. To accomodate this, most modern browsers and e-mail clients treat ISO-8859-1 labeled text as Windows-1252. This widespread behavior became standardized in HTML5.

While less than a percent of the web is labeled as Windows-1252, the HTML5 standard means that more than 6% of the web is decoded as Windows-1252. However, UTF-8 is steadily taking over.

## Examples

Here are some examples that highlight the differences between the character encodings. All byte representations are in hexadecimal.

| Unicode              | ASCII | 8859-1 | W-1252 | UTF-8      | UTF-16     | UTF-32
|----------------------|-------|--------|--------|------------|------------|-----------
| `U+000a:` \n         | `0a`  | `0a`   | `0a`   | `0a`       | `000a`     | `0000000a`
| `U+0061:` a          | `61`  | `61`   | `61`   | `61`       | `0061`     | `00000061`
| `U+00e1:` &#x00e1;   |       | `e1`   | `e1`   | `c3a1`     | `00e1`     | `000000e1`
| `U+20ac:` &#x20ac;   |       |        | `80`   | `e282ac`   | `20ac`     | `000020ac`
| `U+1f951:` &#x1f951; |       |        |        | `f09fa591` | `d83edd51` | `0001f951`

Note the difference in UTF-8 and UTF-16 encodings for the euro sign. The UTF-8 encoding uses more space, and this is the case for a large number of code points in the basic multilingual plane. For languages that reside from `U+0800` to `U+ffff`, UTF-16 could theoretically be up to 33% more space efficient. However, the prevalence of whitespace, digits, and embedded markup can easily nullify or even reverse this effect.


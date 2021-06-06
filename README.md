# The Eclog Format

*Draft v0.9.1*

## Abstract

Eclog is a JSON-like, text-based, lightweight, language-independent data interchange format. Its design aims to be simple, easy to use, explicit, and elegant. It is suitable for data interchange, storage, configuration, and description.

## Table of Contents

- [1. Basics](#1-basics)
- [2. Objects](#2-objects)
- [3. Arrays](#3-arrays)
- [4. Strings](#4-strings)
  - [4.1. Quoted Strings](#41-quoted-strings)
  - [4.2. Unquoted Strings](#42-unquoted-strings)
  - [4.3. Raw Strings](#43-raw-strings)
  - [4.4. Heredoc Strings](#44-heredoc-strings)
  - [4.5. Concatenation](#45-concatenation)
- [5. Numbers](#5-numbers)
- [6. Booleans](#6-booleans)
- [7. Nulls](#7-nulls)
- [8. Comments](#8-comments)
- [9. JSON Compatibility](#9-json-compatibility)

## 1. Basics

- An Eclog text is encoded in UTF-8 (without a byte order mark).
- An Eclog text is a sequence of tokens; the set of tokens includes seven structural characters, strings, numbers, Booleans, and nulls.
- The seven structural characters are curly brackets, square brackets, colon, comma, and plus sign.
- An Eclog text is an object.
- Whitespace and comments are allowed before and after any token.
- A whitespace character is any of the following: horizontal tab, space, CR, and LF.
- A line break can take three forms: CR, LF, and CRLF.

## 2. Objects

An object structure is represented as a pair of curly brackets surrounding zero or more key-value pairs; for the root object (the Eclog text), curly brackets can be omitted.

A key is a string; a value can be an object, array, string, number, Boolean, or null.

A colon separates each key from the value, and there is also a comma after each value, separating the value from the next pair's key. The comma can be omitted if the next pair's key starts on a new line. A trailing comma is allowed after the last pair.

Example:

```
# Person.ecl

firstName: John
lastName: Smith

isAlive: true
age: 27

address:
{
    streetAddress: "21 2nd Street"
    city: "New York"
    state: NY
    postalCode: "10021-3100"
}

phoneNumbers:
[
    { type: home, number: "212 555-1234" }
    { type: office, number: "646 555-4567" }
    { type: mobile, number: "123 456-7890" }
]

children: []
spouse: null
```

Keys within an object should be unique. If a key is duplicated, the parser will take only the last of the duplicated key-value pairs.

## 3. Arrays

An array structure is represented as a pair of square brackets surrounding zero or more values. A value can be an object, array, string, number, Boolean, or null.

A comma comes after each value, separating the value from the next one; the comma can be omitted if the next value starts on a new line. A trailing comma is allowed after the last value.

Example:

```
[
    0.1, 2.0
    "ordered"
    false
    null
]
```

## 4. Strings

Eclog offers four kinds of strings: quoted strings, unquoted strings, raw strings, and heredoc strings.

#### 4.1. Quoted Strings

Like conventions used in the C family of programming languages, a quoted string begins and ends with quotation marks. All Unicode characters can be placed within the quotation marks, except for the characters that must be escaped: quotation mark, backslash, and control characters other than horizontal tab.

Example:

```
"Hello,\nWorld!"
```

Any character may be escaped; available escape sequences are:

| Escape sequence | Unicode code point | Character represented |
| ---- | ---- | ---- |
| `\"` | U+0022 | Double quotation mark |
| `\\` | U+005C | Backslash |
| `\/` | U+002F | Forward slash |
| `\b` | U+0008 | Backspace |
| `\f` | U+000C | Form feed |
| `\n` | U+000A | Newline |
| `\r` | U+000D | Carriage Return |
| `\t` | U+0009 | Horizontal Tab |
| `\uhhhh` | U+0000–U+FFFF |  |
| `\u{...}` | Any |  |

A Unicode escape sequence can take two forms: `\u` followed by four hex digits (four-digit form), or `\u` followed by a pair of curly brackets surrounding 1–6 hex digits (variable-length form).

A character outside the Basic Multilingual Plane (U+0000–U+FFFF) is allowed to be encoded as two code units, namely a UTF-16 surrogate pair.

Examples:

```
"\u6587\u5b57"
"\u{a}\u{61}"
"\u{10437}"
"\ud801\udc37"      # U+10437 is encoded as a surrogate pair
```

#### 4.2. Unquoted Strings

An unquoted string has no explicit delimiters. It begins with a letter  or an underscore, and it ends when a character other than allowed characters is reached. Allowed characters are letters, digits, underscore, hyphen-minus, and period.

**Note**: The five keywords (`true`, `false`, `null`, `inf`, and `nan`) are not unquoted strings and cannot be used as keys.

Examples:

```
timeout: 90
ip-address: "127.0.0.1"
config.cipher: aes256-ctr
_length_: 4096
access: allow-from-all
```

#### 4.3. Raw Strings

A raw string begins with `@delimiter"` and ends with `"delimiter`. All Unicode characters can be placed within the delimiters, except for the control characters other than horizontal tab.

The identifier `delimiter` consists of zero to 16 characters; each character can be a letter, a digit, or an underscore.

As long as `"delimiter` is not a substring of the string literal, the raw string is well-formed. Therefore, if the string literal does not contain double quotation marks, the identifier `delimiter` can be empty.

Examples:

```
# A Windows path; '\' characters do not need to be escaped
path: @"C:\Program Files\Microsoft SDKs\Windows"

# A regex that contains a double quotation mark
regex: @ddd"<\s*img[^>]+src\s*=\s*(["'])(.*?)\1[^>]*>"ddd
```

#### 4.4. Heredoc Strings

A heredoc string begins with `|delimiter` followed by a line break, and it ends when a separate line with `delimiter` on it is reached. All Unicode characters can be placed within the delimiters, except for the control characters other than horizontal tab, CR, and LF.

The line breaks in the string literal will remain as they are. N indent characters (horizontal tabs and spaces) will be removed from the beginning of each line, where N is the end delimiter's indent.

The identifier `delimiter` consists of one to 16 characters; each character can be a letter, a digit, or an underscore.

Example:

```
# A "Hello, World!" program in C
prog_c: |EOF
    #include <stdio.h>

    int main(void)
    {
        printf("Hello, World!\n");
    }
    EOF
```

#### 4.5. Concatenation

The plus sign can join two or more strings end-to-end as one; each string could be a quoted string, raw string, or heredoc string.

Examples:

```
str: "Hello" + ", World!"
path: @"C:\" + @"Windows\" + "Fonts"
```

## 5. Numbers

A number can represent a finite numeric value, infinity, or NaN (Not-A-Number).

As with most programming languages, a finite number begins with an optional plus or minus sign, followed by three parts: an integer part, a fraction part, and an exponent part. The integer part is required; it must contain at least one digit and not start with zero. The fraction part and the exponent part are both optional.

A fraction part is a decimal point followed by one or more digits.

An exponent part is indicated by the letter `e` or `E`, followed by an optional plus or minus sign, followed by one or more digits. As with the integer part, leading zeros are not allowed.

The special values `inf` and `nan`, which stand for infinity and NaN, can also have plus or minus signs.

Examples:

```
ii: -1
e: 2.7182818
length: 40075
speed: 3e8
mass: 1.98855e30
distance: +inf
result: -nan
```

## 6. Booleans

A Boolean can only take one of two possible values: `true` or `false`.

Examples:

```
async: true
dev: false
```

## 7. Nulls

A null can only take one possible value: `null`.

Example:

```
list: null
```

## 8. Comments

A comment begins with a number sign and continues until a line break or EOF is reached. All Unicode characters can be placed in the comment, except for the control characters other than horizontal tab.

Examples:

```
# config.ecl

host_name: "127.0.0.1"      # localhost
port: 4502
#timeout: 3600
max_pool_size: 5            # 0 indicates no limit
#connection_debug: false
```

## 9. JSON Compatibility

A valid JSON text is also a valid Eclog text when the following are true:

1. The JSON text is encoded in UTF-8.
2. The JSON text is an object.


# Klingon Keyword Replacements

This document specifies the mapping between standard C keywords and their Klingon equivalents for the kclang compiler dialect.

## Control Flow Keywords

| C Keyword | Klingon Equivalent | Meaning |
|-----------|-------------------|---------|
| if        | HIja'             | "if" (yes condition) |
| else      | ghobe'            | "else" (no, otherwise) |
| for       | vangqa'           | "for" (repeatedly act) |
| while     | tIq               | "while" (be enduring) |
| do        | ta'               | "do" (accomplish) |
| switch    | tam               | "switch" (exchange) |
| case      | chen              | "case" (take form) |
| break     | mev               | "break" (stop) |
| continue  | yIt               | "continue" (walk/proceed) |
| return    | chegh             | "return" (return/retreat) |
| goto      | jaH               | "goto" (go) |

## Type Keywords

| C Keyword | Klingon Equivalent | Meaning |
|-----------|-------------------|---------|
| int       | mI'               | "int" (number) |
| char      | qIt               | "char" (symbol/character) |
| float     | ghurtaH           | "float" (floating) |
| double    | ghurtaH_chorghvI  | "double" (double floating) |
| void      | chIm              | "void" (be empty) |
| long      | nI'               | "long" (be long) |
| short     | poH               | "short" (be short/limited) |
| unsigned  | Hutlh             | "unsigned" (lack) |
| signed    | moj               | "signed" (be included) |

## Storage Class Keywords

| C Keyword | Klingon Equivalent | Meaning |
|-----------|-------------------|---------|
| static    | motlh             | "static" (be permanent) |
| extern    | puS               | "extern" (outside) |
| auto      | peq               | "auto" (self) |
| register  | qaw               | "register" (remember) |
| const     | choH              | "const" (unchanging) |
| volatile  | choH_pagh         | "volatile" (change constantly) |

## Compound Type Keywords

| C Keyword | Klingon Equivalent | Meaning |
|-----------|-------------------|---------|
| struct    | ghom              | "struct" (group/assembly) |
| union     | tu'               | "union" (combine/unite) |
| enum      | pong              | "enum" (name/list) |
| typedef   | pIm_pong          | "typedef" (define name) |

## Other Keywords

| C Keyword | Klingon Equivalent | Meaning |
|-----------|-------------------|---------|
| sizeof    | yIq               | "sizeof" (measure) |

## Usage

To use the Klingon dialect:

```bash
kclang -x klingon myfile.klingon
```

Example Klingon C code:

```c
mI' qoq(mI' a, mI' b) {
  HIja' (a > b) {
    chegh a;
  } ghobe' {
    chegh b;
  }
}
```

This is equivalent to the standard C code:

```c
int max(int a, int b) {
  if (a > b) {
    return a;
  } else {
    return b;
  }
}
```

## String Delimiters

**Important Note:** In Klingon mode, the single quote (apostrophe, ') is NOT a string delimiter. The apostrophe is a letter in the Klingon alphabet representing a glottal stop, and it can appear within identifiers and keywords (e.g., `HIja'`, `ghobe'`, `ta'`, `mI'`).

- **String literals:** Use double quotes (") only for string literals: `"Hello, world!"`
- **Character literals:** Use backticks (`) for character literals (no closing backtick): `` `a ``, `` `\n ``, `` `Z ``
  - Example: `qIt letter = `K;`
  - Example with escape sequences: `qIt newline = `\n;`
  - Space character: Use `\s` escape sequence: `qIt space = `\s;`
  - All standard C escape sequences are supported: `\n`, `\t`, `\\`, `\"`, `\'`, `\s`, etc.
  - The literal is **implicitly closed** after reading one character or escape sequence
  - No closing backtick is required or expected

The backtick syntax was chosen because:
- It's visually distinct from both single and double quotes
- It's available on standard keyboards
- It doesn't conflict with any existing Klingon syntax
- It clearly indicates a single character literal
- The implicit closing makes the syntax clean and unambiguous

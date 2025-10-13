# Klingon Dialect Examples

This directory contains example programs written in the Klingon dialect of C.

## Files

### hello.klingon
A simple "Hello World" program demonstrating basic Klingon syntax.

**Compile and run:**
```bash
kclang hello.klingon -o hello
./hello
```

### fibonacci.klingon
A comprehensive example demonstrating:
- Functions (iterative and recursive)
- Control flow (if/else, for, while, do-while, switch/case)
- Data types (int, char, short, long, float, double)
- Storage classes (static, extern)
- Structures (struct)
- Unions (union)
- Enumerations (enum)
- Typedefs (typedef)
- Break and continue statements
- The sizeof operator

**Compile and run:**
```bash
kclang fibonacci.klingon -o fibonacci
./fibonacci
```

## Quick Reference

Here are the Klingon keywords used in these examples:

| C Keyword | Klingon | Meaning |
|-----------|---------|---------|
| int       | mI      | number |
| void      | chIm    | empty |
| if        | HIja    | yes condition |
| else      | ghobe   | no, otherwise |
| for       | vangqa  | repeatedly act |
| while     | tIq     | endure |
| do        | ta      | accomplish |
| return    | chegh   | return/retreat |
| struct    | ghom    | group |
| union     | tu      | unite |
| enum      | pong    | name |
| typedef   | pIm_pong| define name |
| sizeof    | yIq     | measure |
| static    | motlh   | permanent |
| break     | mev     | stop |
| continue  | yIt     | proceed |

For the complete list of keyword mappings, see [../files/SPEC.md](../files/SPEC.md).

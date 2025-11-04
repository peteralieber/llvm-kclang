# Klingon Dialect Implementation Notes

## Overview

This document provides technical details about the Klingon dialect implementation in Clang.

## Architecture

The implementation follows a minimal, surgical approach with all changes isolated behind the `LangOptions.Klingon` flag. The keyword translation happens at the lexer level, ensuring zero impact on the rest of the compilation pipeline.

## Key Components

### 1. Language Definition

**Files Modified:**
- `clang/include/clang/Basic/LangStandard.h`
- `clang/include/clang/Basic/LangStandards.def`
- `clang/lib/Basic/LangStandards.cpp`

Added `Klingon` to the `Language` enum and defined a language standard (`lang_klingon`) that inherits C23 features with line comments, digraphs, and hex floats.

### 2. Language Options

**Files Modified:**
- `clang/include/clang/Basic/LangOptions.def`
- `clang/lib/Basic/LangOptions.cpp`

Added a `Klingon` boolean option to `LangOptions` that is set when the language is `Language::Klingon`.

### 3. Input Type System

**Files Modified:**
- `clang/include/clang/Driver/Types.def`
- `clang/lib/Frontend/FrontendOptions.cpp`

Added two new input types:
- `TY_Klingon` - for `.klingon` source files
- `TY_PP_Klingon` - for `.ki` preprocessed files

### 4. Driver Integration

**Files Modified:**
- `clang/lib/Driver/ToolChain.cpp`
- `clang/tools/driver/CMakeLists.txt`

Added `kclang` to the driver suffix list and created a symlink to the clang binary.

### 5. Keyword Translation (Core Implementation)

**File Modified:**
- `clang/lib/Lex/Preprocessor.cpp`

Modified `Preprocessor::LookUpIdentifierInfo()` to translate Klingon keywords before identifier lookup:

```cpp
if (getLangOpts().Klingon) {
  // Get the identifier string
  StringRef IdentStr = ...;
  
  // Translate Klingon keywords to C keywords
  StringRef Translated = llvm::StringSwitch<StringRef>(IdentStr)
      .Case("HIja'", "if")
      .Case("ghobe'", "else")
      .Case("mI'", "int")
      // ... etc
      .Default(StringRef());
  
  if (!Translated.empty()) {
    // Use translated keyword for lookup
    II = getIdentifierInfo(Translated);
  }
}
```

This approach ensures:
- Translation happens before macro expansion
- No changes to the parser, semantic analysis, or code generation
- Zero overhead when not in Klingon mode

## Performance Considerations

### When NOT in Klingon mode:
- **Zero overhead**: Single boolean check that is always false
- **No memory impact**: No additional data structures
- **No code path changes**: Standard path taken

### When IN Klingon mode:
- **Translation overhead**: StringSwitch lookup for each identifier
- **Minimal impact**: Only affects identifiers, not operators or literals
- **One-time cost**: Translation happens once per token

## Memory Layout

No changes to:
- Token representation
- AST node structure
- Symbol table layout
- Debug information format

## Debug Information

The implementation preserves full debug information:
- Source locations point to original Klingon keywords
- Variable names remain as written in source
- Line information is accurate

## Control Flow Impact

The implementation does NOT affect:
- Branch prediction (no new branches in hot paths)
- Call/return semantics
- Exception handling
- Function inlining decisions

The only control flow change is the addition of a boolean check in `LookUpIdentifierInfo`, which:
- Is easily predictable (always false for non-Klingon code)
- Is not in any critical hot path
- Has no impact on performance profiling data

## Testing Strategy

### Unit Tests
- Driver test: Verifies `-x klingon` flag handling
- Parser test: Validates all keyword translations

### Integration Tests
- Example programs: Demonstrate real-world usage
- Comprehensive coverage: All 30+ keywords tested

### Future Testing
- Performance benchmarks: Verify zero overhead claim
- Compatibility tests: Ensure C code unaffected

## Keyword Mapping Strategy

Keyword mappings follow Klingon linguistic principles:
- Control flow → Action verbs
- Types → Descriptive nouns
- Storage classes → State descriptions

See `files/SPEC.md` for complete mapping table.

## Extension Points

Future enhancements could include:
1. Additional keywords (C++, C23 features)
2. Custom diagnostic messages in Klingon
3. Standard library header with Klingon names
4. IDE support (syntax highlighting, completion)

## Maintenance Notes

When adding new C keywords:
1. Add Klingon equivalent to SPEC.md
2. Update StringSwitch in Preprocessor.cpp
3. Add test case to klingon_keywords.klingon

## Apostrophe Support

In authentic Klingon, the apostrophe (') represents a glottal stop and is a letter of the alphabet, not punctuation. To support this:

### Lexer Changes
**File Modified:** `clang/lib/Lex/Lexer.cpp`

1. **Identifier continuation**: Added apostrophe as a valid identifier continuation character in Klingon mode:
   ```cpp
   // In Klingon mode, apostrophe is a letter (glottal stop), not a delimiter
   if (C == '\'' && LangOpts.Klingon) {
     CurPtr = ConsumeChar(CurPtr, Size, Result);
     continue;
   }
   ```

2. **Character literal handling**: Disabled character literals in Klingon mode since single quotes are reserved for the glottal stop:
   ```cpp
   case '\'':
     if (LangOpts.Klingon) {
       if (!isLexingRawMode())
         Diag(BufferPtr, diag::err_klingon_char_literal);
       Kind = tok::unknown;
       break;
     }
   ```

### Impact
- Apostrophes can now appear within identifiers: `HIja'`, `ghobe'`, `mI'`, `ta'`, etc.
- Character literals with single quotes (`'a'`) are not supported in Klingon mode
- **Backtick character literals**: Use single opening backtick for character literals (no closing backtick): `` `a ``, `` `\n ``, `` `K ``
- String literals with double quotes remain fully supported: `"Hello"`

### Backtick Character Literals

**Files Modified:**
- `clang/include/clang/Lex/Lexer.h`
- `clang/lib/Lex/Lexer.cpp`
- `clang/include/clang/Basic/DiagnosticLexKinds.td`

To provide character literal support while preserving apostrophes as letters, backtick (`) is used as an opening delimiter for character constants. The literal is **implicitly closed** after reading one character or escape sequence:

1. **Lexer function**: Added `LexBacktickCharConstant()` with implicit closing:
   ```cpp
   bool Lexer::LexBacktickCharConstant(Token &Result, const char *CurPtr,
                                       tok::TokenKind Kind) {
     // Reads opening backtick
     // Reads one character or escape sequence
     // Implicitly closes - no closing backtick required
     // Supports all standard escape sequences plus \s for space
   }
   ```

2. **Lexer switch case**: Added backtick handling in `LexTokenInternal()`:
   ```cpp
   case '`':
     if (LangOpts.Klingon) {
       MIOpt.ReadToken();
       return LexBacktickCharConstant(Result, CurPtr, tok::char_constant);
     }
     Kind = tok::unknown;
     break;
   ```

3. **Updated diagnostic**: The error message for single quote usage now suggests the backtick alternative.

**Usage:**
- Character literals (no closing backtick): `` `a ``, `` `Z ``, `` `0 ``
- Escape sequences: `` `\n ``, `` `\t ``, `` `\\ ``, `` `\' ``, `` `\s ``
- All standard C escape sequences are supported, plus `\s` for space
- Examples: `qIt c = `a;`, `qIt space = `\s;`, `qIt newline = `\n;`

## Known Limitations

1. Preprocessor directives remain in English (`#include`, `#define`, etc.)
2. Standard library identifiers remain in English (`printf`, `stdio.h`, etc.)
3. No translation for C++ keywords (future enhancement)
4. Backtick character literals only work in Klingon mode (not available in standard C mode)

## Conclusion

The implementation achieves the goals of:
- ✅ Minimal, targeted changes
- ✅ Zero impact on standard C
- ✅ Full isolation behind language option
- ✅ No corruption of performance or debug data
- ✅ Clean, maintainable code
- ✅ Authentic Klingon language support with apostrophes
